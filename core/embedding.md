---
url: https://foldkit.dev/core/embedding
title: "Embedding"
description: "Run a Foldkit app inside a host application with a typed lifecycle handle and Ports."
access_date: 2026-08-03T18:23:29.850Z
current_date: 2026-08-03T18:23:29.850Z
---

# Embedding

## Overview

A Foldkit app does not have to own its page. `Runtime.embed` starts a program under a host-controlled lifecycle and returns a handle the host uses to communicate with it: for example, a widget inside a React or Vue app, a checkout flow inside a server-rendered page, or an interactive panel inside a larger dashboard. The host controls when the app starts and stops, pushes data in, and receives values out, all without touching the Model or dispatching Messages. The boundary is a set of Schema-typed Ports, modeled on Elm ports.

The [embedding example](https://foldkit.dev/example-apps/embedding) runs everything on this page: a plain TypeScript host page that mounts a ticking widget, pushes a step value in, mirrors the count the widget emits, and unmounts it on demand.

## Choosing an Entry Point

Embedded apps are usually built with `makeElement`: the view returns `Html` and the runtime stays scoped to its container, never touching the document `<head>`, the URL bar, or anything else the host owns. Use `makeApplication` only when the embedded app should own page-level concerns like the document title. `embed` accepts programs from both.

## Declaring Ports

Ports are declared with `Port.inbound` and `Port.outbound`, grouped in a record, and registered on the program config. The record keys name the ports on the handle:

```
import { Effect, Schema as S } from 'effect'
import { Port, Runtime } from 'foldkit'

// Each Port carries a Schema. The host works with the Schema's Encoded
// side, the app with the decoded Type. Record keys name the ports on the
// handle. Name ports subject-first like DOM event names (stepChanged);
// the app wraps each value into its own verb-first Message (ChangedStep).
export const ports = {
  inbound: { stepChanged: Port.inbound(S.Number) },
  outbound: { countChanged: Port.outbound(S.Number) },
}

export const makeElement = (container: HTMLElement, flags: Flags) =>
  Runtime.makeElement({
    Model,
    Flags,
    flags: Effect.succeed(flags),
    init,
    update,
    view,
    subscriptions,
    // Registering the record makes the ports available to the app and
    // types the EmbedHandle that Runtime.embed returns.
    ports,
    container,
  })
```

## Three Communication Directions

Host interop maps onto primitives the architecture already has. Data crosses the boundary in three ways, and each direction reuses the concept that already handles that shape of input or output.

### Flags: Initial Data In

Data the app needs once, at startup, enters through `Flags`, exactly as in a page-owning app. The host passes values when it constructs the program, and `init` folds them into the initial Model.

### Inbound Ports: a Subscription

Data the host pushes while the app runs arrives on an inbound Port, which the app consumes as a Subscription source. `Port.subscription` wraps every value into a Message, so host input drives `update` the same way any other external event does:

```
import { Port, Subscription } from 'foldkit'

import { ports } from './ports'

// An inbound Port is a Subscription source. Port.subscription wraps every
// decoded value the host sends into a Message, so host input enters update
// the same way any other external event does.
export const subscriptions = Subscription.make<Model, Message>()(_entry => ({
  hostStep: Port.subscription(ports.inbound.stepChanged, step =>
    ChangedStep({ step }),
  ),
}))
```

For a Model-gated entry, build one from `Port.stream` inside `Subscription.make`. Values sent while no Stream for the Port is running are dropped, with one exception: values sent before the first Stream attaches are buffered and delivered to it in order, so sends issued right after `embed` are not lost during startup.

### Outbound Ports: a Command

Values the app announces to the host leave through an outbound Port, written from a Command. `Port.emit` is an Effect that encodes the value and delivers it to every subscribed host listener; it composes into the app’s own Commands like any other Effect:

```
import { Effect, Schema as S } from 'effect'
import { Command, Port } from 'foldkit'
import { evo } from 'foldkit/struct'

import { CompletedReportCount } from './message'
import { ports } from './ports'

// An outbound Port is written from a Command. Port.emit encodes the value
// and delivers it to every host listener; the Command acknowledges with a
// Completed* Message like any other fire-and-forget Command.
export const ReportCount = Command.define('ReportCount', {
  args: { count: S.Number },
  messages: [CompletedReportCount],
  execute: ({ count }) =>
    Port.emit(ports.outbound.countChanged, count).pipe(
      Effect.as(CompletedReportCount()),
    ),
})

// In update, emitting is just returning the Command:
const handleAdvance = (model: Model): UpdateReturn => {
  const count = model.count + model.step
  return [evo(model, { count: () => count }), [ReportCount({ count })]]
}
```

When the program runs without an embed handle (started with `Runtime.run`), emitting is a no-op, so the same app works embedded and standalone.

## The Embed Handle

`Runtime.embed(program)` starts the runtime and returns an `EmbedHandle`. The handle has one entry per declared Port under `ports` (inbound Ports get `send`, outbound Ports get `subscribe`), plus `dispose`:

```
import { Runtime } from 'foldkit'

import { makeElement } from './widget'

const container = document.getElementById('widget-slot')
if (container === null) {
  throw new Error('Missing widget container')
}

// Flags seed the widget's initial state at mount.
const element = makeElement(container, { initialCount: 10 })

// embed starts the runtime and returns the handle. The handle is the whole
// boundary: the host never reads the Model or dispatches Messages.
const handle = Runtime.embed(element)

// Host to widget. The value is validated against the Port's Schema; an
// invalid value never reaches the app and comes back as a typed Exit failure.
handle.ports.stepChanged.send(5)

// Widget to host. subscribe returns an unsubscribe function; multiple
// listeners each receive every emitted value.
const unsubscribe = handle.ports.countChanged.subscribe(count => {
  console.log(`widget count: ${count}`)
})

// On host unmount: dispose interrupts the runtime and runs all cleanup.
// Subscriptions, Mounts, ManagedResources, and in-flight Commands stop, the
// rendered DOM is removed, and the container element is restored empty,
// ready for a fresh embed. dispose is idempotent.
unsubscribe()
handle.dispose()
```

`dispose` ties the runtime to the host’s unmount. It interrupts the runtime and runs all cleanup: Subscriptions, Mounts, and ManagedResources release, in-flight Commands stop, the rendered DOM is removed, and the container element is restored empty in its place, ready for a fresh `embed`. It is idempotent, and sends on a disposed handle are no-ops, so a host that unmounts and remounts in quick succession stays correct. A program can be embedded once at a time; after `dispose`, the same program and container can be embedded again.

## The Schema Boundary

Every value that crosses the boundary passes through its Port’s Schema. The host works with the Schema’s Encoded side, the app with the decoded Type: `send` validates by decoding, and `Port.emit` encodes before delivery. Keep Port Schemas to data that survives encoding, the same discipline as a network payload; functions and DOM references cannot cross. The Model does not cross either: outbound Ports carry facts the app chooses to announce, not state snapshots, so the host never couples to the app’s internal shape.

An invalid inbound value never reaches the app. `send` returns an `Exit` carrying the `SchemaError` and logs the rejection, so a typed host gets compile-time checking and an untyped host gets a clear runtime signal, while the app only ever sees values its Schemas accepted.

## Embedding in React

The handle is framework-agnostic, and its lifecycle maps directly onto effect hooks. In React, `embed` on effect setup and `dispose` on cleanup is the whole integration:

```
import { Runtime } from 'foldkit'
import { useEffect, useRef, useState } from 'react'

import { makeElement } from './widget'

function WidgetPanel() {
  const containerRef = useRef<HTMLDivElement>(null)
  const [count, setCount] = useState(0)

  useEffect(() => {
    if (containerRef.current === null) {
      return
    }

    // Mount on effect setup, dispose on cleanup. dispose restores the
    // container element, so the ref stays valid across remounts (including
    // strict mode's setup/cleanup/setup sequence).
    const element = makeElement(containerRef.current, { initialCount: 10 })
    const handle = Runtime.embed(element)
    const unsubscribe = handle.ports.countChanged.subscribe(setCount)

    return () => {
      unsubscribe()
      handle.dispose()
    }
  }, [])

  return (
    <div>
      <p>Widget count: {count}</p>
      <div ref={containerRef} />
    </div>
  )
}
```
