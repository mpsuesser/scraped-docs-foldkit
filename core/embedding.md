---
url: https://foldkit.dev/core/embedding
title: "Embedding"
description: "Embed a Foldkit widget in another application through a Schema-typed handle. Covers initial Flags, inbound and outbound Ports, disposal, and React integration."
access_date: 2026-10-05T07:06:39.496Z
current_date: 2026-10-05T07:06:39.496Z
---

# Embedding

## Overview

A Foldkit app does not have to own its page. `Runtime.embed` starts a program under a host-controlled lifecycle and returns a handle the host uses to communicate with it: for example, a widget inside a React or Vue app, a checkout flow inside a server-rendered page, or an interactive panel inside a larger dashboard. The host controls when the app starts and stops, pushes data in, and receives values out, all without touching the Model or dispatching Messages. The boundary is a set of Schema-typed Ports, modeled on Elm ports.

The [embedding example](https://foldkit.dev/example-apps/embedding) runs everything on this page: a plain TypeScript host page that mounts a ticking widget, pushes a step value in, mirrors the count the widget emits, and unmounts it on demand.

## Choosing an Entry Point

Embedded apps are usually built with `makeElement`: the view returns `Html` and the runtime stays scoped to its container, never touching the document `<head>`, the URL bar, or anything else the host owns. Use `makeApplication` only when the embedded app should own page-level concerns like the document title. `embed` accepts programs from both.

## Declaring Ports

Ports are declared with `Port.inbound` and `Port.outbound`, grouped in a record, and registered on the program config. The record keys name the ports on the handle:

Port declaration

## Three Communication Directions

Host interop maps onto primitives the architecture already has. Data crosses the boundary in three ways, and each direction reuses the concept that already handles that shape of input or output.

### Flags: Initial Data In

Data the app needs once, at startup, enters through `Flags`, exactly as in a page-owning app. The host passes values when it constructs the program, and `init` folds them into the initial Model.

### Inbound Ports: a Subscription

Data the host pushes while the app runs arrives on an inbound Port, which the app consumes as a Subscription source. `Port.subscription` wraps every value into a Message, so host input drives `update` the same way any other external event does:

Inbound Subscription

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

Outbound Command

When the program runs without an embed handle (started with `Runtime.run`), emitting is a no-op, so the same app works embedded and standalone.

## The Embed Handle

`Runtime.embed(program)` starts the runtime and returns an `EmbedHandle`. The handle has one entry per declared Port under `ports` (inbound Ports get `send`, outbound Ports get `subscribe`), plus `dispose`:

Host wiring

`dispose` ties the runtime to the host’s unmount. It interrupts the runtime and runs all cleanup: Subscriptions, Mounts, and ManagedResources release, in-flight Commands stop, the rendered DOM is removed, and the container element is restored empty in its place, ready for a fresh `embed`. It is idempotent, and sends on a disposed handle are no-ops, so a host that unmounts and remounts in quick succession stays correct. A program can be embedded once at a time; after `dispose`, the same program and container can be embedded again.

## The Schema Boundary

Every value that crosses the boundary passes through its Port’s Schema. The host works with the Schema’s Encoded side, the app with the decoded Type: `send` validates by decoding, and `Port.emit` encodes before delivery. Keep Port Schemas to data that survives encoding, the same discipline as a network payload; functions and DOM references cannot cross. The Model does not cross either: outbound Ports carry facts the app chooses to announce, not state snapshots, so the host never couples to the app’s internal shape.

An invalid inbound value never reaches the app. `send` returns an `Exit` carrying the `SchemaError` and logs the rejection, so a typed host gets compile-time checking and an untyped host gets a clear runtime signal, while the app only ever sees values its Schemas accepted.

## Embedding in React

The handle is framework-agnostic, and its lifecycle maps directly onto effect hooks. In React, `embed` on effect setup and `dispose` on cleanup is the whole integration:

React host
