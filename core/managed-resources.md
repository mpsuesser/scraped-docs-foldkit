---
url: https://foldkit.dev/core/managed-resources
title: "Managed Resources"
description: "Resources that activate and release based on Model state."
access_date: 2026-08-03T19:45:20.723Z
current_date: 2026-08-03T19:45:20.723Z
---

# Managed Resources

## Overview

Resources live for the entire application lifecycle. But some resources are heavy and should only be active while the Model is in a particular state, like a camera stream during a video call, a `WebSocket` connection while on a chat page, or a Web Worker pool during a computation. Managed Resources provide Model-driven acquire/release lifecycle, using the same deps-diffing engine as Subscriptions.

The restaurant analogy

If resources are kitchen equipment (permanent, always on), Managed Resources are specialty ingredients sourced on demand. When the menu shifts to a seafood special (Model state changes), the kitchen orders in fresh lobster and sets up the shellfish station. When the special ends, the lobster goes back to the supplier and the station is broken down. If the chef (Command) tries to plate lobster when it’s not in season, they get a clear signal: `ResourceNotAvailable`. And if the special changes from Maine lobster to king crab (params change), the old stock is returned and new stock is sourced, just like switching camera resolutions triggers release and reacquire.

Define a Managed Resource identity with `ManagedResource.tag`, then wire its lifecycle with `ManagedResource.make`. The `modelToMaybeRequirements` function returns `Option.some(params)` when the resource should be active, and `Option.none()` when it should be released.

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

When requirements change, the runtime handles the lifecycle automatically. If `modelToMaybeRequirements` transitions from `Option.none()` to `Option.some(params)`, the resource is acquired and `onAcquired` is sent. When it goes back to `Option.none()`, the resource is released and `onReleased` is sent. If the params change while active (e.g. switching cameras), the old resource is released and a new one is acquired with the new params.

If acquisition fails, `onAcquireError` is sent as a Message. The resource daemon continues watching for the next deps change. A failed acquisition does not crash the application.

## Accessing Managed Resources in Commands

Commands access the resource value via `.get`. Since the resource might not be active, `.get` can fail with `ResourceNotAvailable`. The type system enforces this: your Command won’t compile unless you handle the error.

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

This is the same `catchTag` pattern you already use for Command errors. If your Model correctly gates Commands (only dispatching `takePhoto` after `AcquiredCamera` has been received), the `catchTag` is a safety net that never fires. But if your Model logic has a bug, you get a graceful error message instead of a crash.

## Building a Layer in acquire

When a resource’s setup and teardown are already packaged as an Effect `Layer`, you do not have to unpack it by hand. `acquire` runs with the resource-lifetime `Scope` in its context, the same scope the runtime closes on release or re-acquire. So `Layer.build` registers the Layer’s finalizers on it, and you map the built context down to the bare service value.

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

The resource tag holds the bare service value, so Commands read it through `.get` with no wrapper to destructure. Any finalizer registered during `acquire`, whether through `Layer.build` or `Effect.addFinalizer`, tears down with the resource, so `release` is simply `() => Effect.void`. The explicit `release` callback still runs first, then the scope finalizers, matching the last-in-first-out order Effect uses for any scope.

## Composing Child Submodels

A child Submodel owns its Managed Resources in its own Model and Message terms, built with `ManagedResource.make` and knowing nothing about any parent. `ManagedResource.lift` translates that record into the parent through a single Model lens and a single Message wrapper, the same shape as update delegation and `Subscription.lift`. `ManagedResource.aggregate` then combines a root-level record with any lifted child records into the single record `makeApplication` expects, throwing at startup on duplicate keys.

Unlike `Subscription.lift`, `toChildModel` returns an `Option`. A Managed Resource already speaks in `Option` (`modelToMaybeRequirements` returns `Option.none()` to release), so a Submodel embedded as `Option` that is not mounted is just another `none`: a missing child releases the resource through the same channel.

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

These verbs compose across Submodel levels the same way their Subscription counterparts do. The [Subscription Organization](https://foldkit.dev/patterns/subscription-organization) page traces the full leaf-to-root walkthrough. It uses Subscriptions for its example, but the shape is identical here: `make` at each level, `lift` each child, `aggregate` the results.

Resources vs Managed Resources

Use `resources` for things that live forever (an `RpcClient`, an analytics client). Use `managedResources` for things tied to a Model state (camera streams, an `AudioContext`, `WebSocket` connections).

With resources and Managed Resources, your app can work with any browser API. But what happens when something goes seriously wrong, like an unrecoverable error in update, view, or a Command? The next page covers crash views.
