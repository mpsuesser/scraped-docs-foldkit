---
url: https://foldkit.dev/core/managed-resources
title: "Managed Resources"
description: "Resources that activate and release based on Model state."
access_date: 2026-08-19T19:38:38.072Z
current_date: 2026-08-19T19:38:38.072Z
---

# Managed Resources

## Overview

Resources live for the entire runtime. Some stateful handles should exist only while the Model is in a particular state: a camera stream during a video call, a `WebSocket` while on a chat page, or a Web Worker pool during a computation. Managed Resources give those handles a Model-driven acquire and release lifecycle, using the same dependency-diffing engine as Subscriptions.

The restaurant analogy

Resources are the kitchen equipment available all night. A Managed Resource is a specialty station set up only while the menu needs it. Changing the special tears down the old station and sets up the new one, just as changing camera requirements releases one stream and acquires another. A Command that asks for an inactive station receives `ResourceNotAvailable`.

Define the handle’s identity with `ManagedResource.tag`, then wire its lifecycle with `ManagedResource.make`. The `modelToMaybeRequirements` function returns `Option.some(params)` while the handle should be active and `Option.none()` while it should be absent.

```
import { Effect, Option, Schema as S, pipe } from 'effect'
import { ManagedResource, Runtime } from 'foldkit'

// 1. Define a Managed Resource identity
const CameraStream = ManagedResource.tag<MediaStream>()('CameraStream')

// 2. Wire the lifecycle with make. The requirements schema sits inline next
//    to its config: Option.some = active, Option.none = inactive
const managedResources = ManagedResource.make<Model, Message>()(entry => ({
  camera: entry(S.Option(S.Struct({ facingMode: S.String })), {
    resource: CameraStream,
    modelToMaybeRequirements: model =>
      pipe(
        model.callState,
        Option.liftPredicate(
          (callState): callState is typeof InCall.Type =>
            callState._tag === 'InCall',
        ),
        Option.map(callState => ({
          facingMode: callState.facingMode,
        })),
      ),
    acquire: ({ facingMode }) =>
      Effect.tryPromise(() =>
        navigator.mediaDevices.getUserMedia({
          video: { facingMode },
        }),
      ),
    release: stream =>
      Effect.sync(() => stream.getTracks().forEach(track => track.stop())),
    onAcquired: () => AcquiredCamera(),
    onReleased: () => ReleasedCamera(),
    onAcquireError: error => FailedAcquireCamera({ error: String(error) }),
  }),
}))

// 3. Pass to makeApplication
const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  container: document.getElementById('root'),
  managedResources,
})
```

The runtime compares the requirements after every Model change and performs the corresponding transition.

Requirements transition

Runtime behavior

`Option.none()`

to

`Option.some(params)`

Acquire the handle, then dispatch

`onAcquired`

.

`Option.some(params)`

to

`Option.none()`

Release the handle, then dispatch

`onReleased`

.

`Option.some(a)`

to a different

`Option.some(b)`

Release and dispatch

`onReleased`

, then acquire and dispatch

`onAcquired`

.

Structurally equal requirements

Keep the current handle.

If acquisition fails, the runtime dispatches `onAcquireError` as a Message. The lifecycle keeps watching for the next requirements change, and the failed acquisition does not crash the application.

## Accessing Managed Resources in Commands

Commands access the current handle through `.get`. Because the handle may be inactive, `.get` can fail with `ResourceNotAvailable`. The Command must turn that error into one of its declared result Messages.

```
import { Array, Effect, Option } from 'effect'
import { Command, ManagedResource } from 'foldkit'

const CameraStream = ManagedResource.tag<MediaStream>()('CameraStream')

const TakePhoto = Command.define('TakePhoto', {
  messages: [SucceededTakePhoto, CameraUnavailable],
  execute: Effect.gen(function* () {
    const stream = yield* CameraStream.get

    const maybeTrack = Array.head(stream.getVideoTracks())
    const bitmap = yield* Option.match(maybeTrack, {
      onNone: () => Effect.fail(new Error('No video track available')),
      onSome: track => {
        const imageCapture = new ImageCapture(track)
        return Effect.tryPromise(() => imageCapture.grabFrame())
      },
    })

    return SucceededTakePhoto({ width: bitmap.width, height: bitmap.height })
  }).pipe(Effect.catch(() => Effect.succeed(CameraUnavailable()))),
})
```

This is the usual Command error-to-Message boundary. The Model should gate the operation, for example by enabling `TakePhoto` only after `AcquiredCamera` has been received. The error handler remains a safety net if that Model logic is wrong or the handle disappears before the Command reads it.

## Building a Layer in acquire

When setup and teardown are already packaged as an Effect `Layer`, keep that lifecycle intact. `acquire` runs with the Managed Resource’s `Scope` in its context. `Layer.build` registers the Layer’s finalizers on that Scope, and the runtime closes it on release or reacquisition. Map the built Context down to the bare service value that Commands need.

```
import { Context, Effect, Layer, Option, Schema as S } from 'effect'
import { ManagedResource } from 'foldkit'

// A heavy engine whose init and teardown are packaged as an Effect Layer.
interface ChessEngine {
  readonly bestMove: (fen: string) => Effect.Effect<string>
}

class ChessEngineService extends Context.Service<
  ChessEngineService,
  ChessEngine
>()('ChessEngineService') {}

declare const engineLayer: Layer.Layer<ChessEngineService>

// 1. The Managed Resource holds the bare service value, with no wrapper.
const Engine = ManagedResource.tag<ChessEngine>()('ChessEngine')

// 2. acquire runs with the resource-lifetime Scope in its context, so
//    Layer.build registers the Layer's finalizers on it. They tear down when
//    the resource is released or re-acquired.
const managedResources = ManagedResource.make<Model, Message>()(entry => ({
  engine: entry(S.Option(S.Null), {
    resource: Engine,
    modelToMaybeRequirements: model => Option.as(model.maybeAnalysisSlug, null),
    acquire: () =>
      Layer.build(engineLayer).pipe(
        Effect.map(context => Context.get(context, ChessEngineService)),
      ),
    // The scope closes on release, so the Layer finalizers run automatically.
    release: () => Effect.void,
    onAcquired: () => StartedEngine(),
    onReleased: () => StoppedEngine(),
    onAcquireError: error => FailedStartEngine({ error: String(error) }),
  }),
}))
```

The resource tag holds that bare value, so Commands read it through `.get` with no wrapper to destructure. Any finalizer registered during `acquire`, through either `Layer.build` or `Effect.addFinalizer`, runs when the handle is released. In that case the explicit `release` can be `() => Effect.void`. The explicit callback runs first, followed by the Scope finalizers in Effect’s last-in-first-out order.

## Composing Child Submodels

A child Submodel defines its Managed Resources in its own Model and Message terms, with no knowledge of its parent. `ManagedResource.lift` translates the child record through a Model accessor and a Message wrapper, matching the shape of update delegation and `Subscription.lift`. `ManagedResource.aggregate` combines root and lifted child records into the single record the runtime config expects. Duplicate keys throw at startup instead of silently replacing an entry.

Unlike `Subscription.lift`, `toChildModel` returns an `Option`. A Managed Resource already uses `Option.none()` to mean “release”, so an optional child that is not mounted naturally follows the same path. Removing the child releases its handle.

```
// page/call/managedResource.ts
import { Effect, Option, Schema as S } from 'effect'
import { ManagedResource } from 'foldkit'

import {
  ClosedSignaling,
  FailedSignaling,
  GotVideoCallMessage,
  type Message,
  OpenedSignaling,
} from './message'
import type { Model } from './model'
import * as VideoCall from './videoCall'

const SIGNALING_URL = 'wss://example.com/call/signaling'

const SignalingSocket = ManagedResource.tag<WebSocket>()('SignalingSocket')

const videoCallManagedResources = ManagedResource.lift(
  VideoCall.managedResources,
)<Model, Message>({
  toChildModel: model => model.videoCall,
  toParentMessage: message => GotVideoCallMessage({ message }),
})

const localManagedResources = ManagedResource.make<Model, Message>()(entry => ({
  signalingSocket: entry(S.Option(S.Null), {
    resource: SignalingSocket,
    modelToMaybeRequirements: model => Option.as(model.videoCall, null),
    acquire: () => Effect.try(() => new WebSocket(SIGNALING_URL)),
    release: socket => Effect.sync(() => socket.close()),
    onAcquired: () => OpenedSignaling(),
    onReleased: () => ClosedSignaling(),
    onAcquireError: error => FailedSignaling({ error: String(error) }),
  }),
}))

export const managedResources = ManagedResource.aggregate<Model, Message>()(
  videoCallManagedResources,
  localManagedResources,
)
```

The same operations compose across every Submodel level: `make` at the owner, `lift` through each parent, and `aggregate` at the root. [Subscription Organization](https://foldkit.dev/patterns/subscription-organization) traces that leaf-to-root shape with Subscriptions; the Managed Resource structure is identical.

Resources vs Managed Resources

Use `resources` for services that live with the runtime, such as an `RpcClient` or analytics client. Use `managedResources` for handles whose lifetime follows the Model, such as camera streams, an `AudioContext`, or `WebSocket` connections.

Resources and Managed Resources cover long-lived services and Model-scoped handles. Unrecoverable errors in update, view, or a Command follow a different runtime path. The next page covers crash views.
