---
url: https://foldkit.dev/api-reference/port
title: "Port"
description: "API documentation for the Port module."
access_date: 2026-08-07T13:54:51.385Z
current_date: 2026-08-07T13:54:51.385Z
---

# Port

## Functions

### emit

function

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L245)

```
/**
 * Emits a value on an outbound Port. The value is encoded against the Port's
 * Schema and delivered to every host listener subscribed through the
 * `EmbedHandle`. Compose it into the app's own Commands like any other
 * Effect:
 * 
 * ```ts
 * const ReportCount = Command.define('ReportCount', {
 *   args: { count: S.Number },
 *   messages: [CompletedReportCount],
 *   execute: ({ count }) =>
 *     Port.emit(ports.outbound.countChanged, count).pipe(
 *       Effect.as(CompletedReportCount()),
 *     ),
 * })
 * ```
 * 
 * When the program runs without an embed handle (started with `Runtime.run`),
 * emitting is a no-op.
 */
<Value, Encoded>(
  port: Outbound<Value, Encoded>,
  value: Value
): Effect<void>
```

### inbound

function

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L68)

```
/**
 * Declares an inbound Port carrying values of the given Schema. The host
 * sends values in the Schema's Encoded form; they are validated by decoding
 * at the boundary, so only well-formed `Value`s ever reach the app.
 */
<Value, Encoded>(schema: Codec<Value, Encoded>): Inbound<Value, Encoded>
```

### outbound

function

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L88)

```
/**
 * Declares an outbound Port carrying values of the given Schema. The app
 * emits `Value`s with `Port.emit`; they are encoded at the boundary, so host
 * listeners receive the Schema's Encoded form.
 */
<Value, Encoded>(schema: Codec<Value, Encoded>): Outbound<Value, Encoded>
```

### stream

function

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L177)

```
/**
 * The decoded values arriving on an inbound Port, as a Stream. This is the
 * atomic primitive for consuming a Port inside a Subscription entry; reach
 * for `Port.subscription` when you want the common always-on form. Values
 * sent while no Stream for the Port is running are dropped, except for
 * values sent before the first Stream attaches, which are buffered and
 * delivered to it in order (so host sends issued right after `Runtime.embed`
 * are not lost during startup).
 */
<Value, Encoded>(port: Inbound<Value, Encoded>): Stream<Value>
```

### subscription

function

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L220)

```
/**
 * Builds a Subscription entry that wraps every decoded value arriving on an
 * inbound Port into a Message. The entry is persistent: it runs for the
 * runtime's lifetime, independent of the Model. Pass it as an entry value
 * inside `Subscription.make`. For a Model-gated entry, build one yourself
 * from `Port.stream`.
 */
<Value, Encoded, Message>(
  port: Inbound<Value, Encoded>,
  toMessage: (value: Value) => Message
): EntryWithoutKeepAlive<unknown, Message, Record<string, never>, never>
```

## Types

### Ports

type

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L50)

```
/**
 * The Ports record passed to `makeApplication` or `makeElement` via the
 * `ports` config field. Record keys name the ports; the names appear on the
 * `EmbedHandle` and in boundary error messages.
 */
type Ports = Readonly<{
  inbound: Readonly<Record<string, Inbound<any, any>>>
  outbound: Readonly<Record<string, Outbound<any, any>>>
}>
```

## Interfaces

### Inbound

interface

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L29)

```
/**
 * A typed channel for values flowing from the host into the app. The app
 * consumes the decoded values as a Subscription source via `Port.stream` or
 * `Port.subscription`; the host pushes encoded values through the
 * `EmbedHandle` returned by `Runtime.embed`. Create with `Port.inbound`.
 */
interface Inbound {
  [InboundTypeId]: typeof InboundTypeId
  schema: Codec<Value, Encoded>
}
```

### Outbound

interface

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L40)

```
/**
 * A typed channel for values flowing from the app out to the host. The app
 * emits values with `Port.emit` inside its own Commands; the host listens
 * through the `EmbedHandle` returned by `Runtime.embed`. Create with
 * `Port.outbound`.
 */
interface Outbound {
  [OutboundTypeId]: typeof OutboundTypeId
  schema: Codec<Value, Encoded>
}
```

## Constants

### InboundTypeId

const

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L7)

```
/** Type-level brand for inbound Port values. */
const InboundTypeId: unique symbol
```

### OutboundTypeId

const

[source](https://github.com/foldkit/foldkit/blob/b94bf96a2f96d984390df790a8ab63c1618f4776/packages/foldkit/src/port/port.ts#L16)

```
/** Type-level brand for outbound Port values. */
const OutboundTypeId: unique symbol
```
