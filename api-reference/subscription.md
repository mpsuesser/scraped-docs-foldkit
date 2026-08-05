---
url: https://foldkit.dev/api-reference/subscription
title: "Subscription"
description: "API documentation for the Subscription module."
access_date: 2026-08-05T01:22:40.949Z
current_date: 2026-08-05T01:22:40.949Z
---

# Subscription

## Functions

### aggregate

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/subscription.ts#L194)

```
/**
 * Combines multiple Subscriptions records into one. Throws on duplicate
 * keys so a misconfigured aggregate fails loudly at startup rather than
 * silently overriding.
 */
<Model, Message, Services = never>(): (records: readonly Array<Readonly<Record<string, Subscription<Model, Message, any, Services>>>>) => Subscriptions<Model, Message, Services>
```

### animationFrame

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/animationFrame.ts#L66)

```
/**
 * Build a Subscription that emits a Message on every
 * `requestAnimationFrame` tick, with the inter-frame delta in milliseconds.
 */
<Model, Message>(config: AnimationFrameConfig<Model, Message>): {
  dependenciesSchema: Struct<{
    isActive: Boolean
  }>
  dependenciesToStream: (__namedParameters: {
    isActive: boolean
  }) => Stream<Message, never, never>
  modelToDependencies: (model: Model) => {
    isActive: boolean
  }
}
```

### fromEvent

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/fromEvent.ts#L170)

```
/**
 * Build a Stream that emits a Message for every dispatch of a DOM event,
 * registering the listener when the Stream's scope opens and removing it when
 * the scope closes.
 * 
 * The listener lifecycle uses `Effect.acquireRelease`. The `addEventListener`
 * call happens inside the acquire Effect, and the matching
 * `removeEventListener` is registered only after acquire completes, so the
 * listener never leaks on interruption.
 * 
 * This is a Stream, not a Subscription entry. Wrap it with
 * `Subscription.persistent` for a listener whose lifetime spans the whole
 * Subscriptions record, or plug it into a `Subscription.make` entry's
 * `dependenciesToStream` (typically behind `Stream.when`) to gate it on a
 * Model condition.
 * 
 * For a listener that reacts to only some events, reach for
 * `fromEventFilterMap`, whose mapper returns `Option<Message>`.
 */
<EventType extends Event, Message>(config: FromEventConfig<EventType, Message>): Stream<Message>
```

### fromEventFilterMap

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/fromEvent.ts#L104)

```
/**
 * Build a Stream that emits a Message for the dispatches of a DOM event the
 * mapper chooses to keep, registering the listener when the Stream's scope
 * opens and removing it when the scope closes.
 * 
 * This is the filtered variant of `fromEvent`. Its `toMessage` returns
 * `Option.some(message)` to emit and `Option.none()` to ignore the event, so a
 * single listener can react to some dispatches while passing on the rest.
 * 
 * Reach for this over a downstream `Stream.filterMap` whenever the decision to
 * keep an event is paired with `event.preventDefault()`. The mapper runs
 * synchronously inside the browser's event dispatch, so `preventDefault()`
 * takes effect, while a downstream filter would run on a later turn after the
 * default action has already happened.
 * 
 * The listener lifecycle uses `Effect.acquireRelease`. The `addEventListener`
 * call happens inside the acquire Effect, and the matching
 * `removeEventListener` is registered only after acquire completes, so the
 * listener never leaks on interruption.
 * 
 * This is a Stream, not a Subscription entry. Wrap it with
 * `Subscription.persistent` for a listener whose lifetime spans the whole
 * Subscriptions record, or plug it into a `Subscription.make` entry's
 * `dependenciesToStream` (typically behind `Stream.when`) to gate it on a
 * Model condition.
 */
<EventType extends Event, Message>(config: FromEventFilterMapConfig<EventType, Message>): Stream<Message>
```

### lift

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/subscription.ts#L266)

```
/**
 * Lifts a record of child Subscriptions into a parent's Model and Message
 * context, applying a Model accessor and a Message wrapper uniformly to
 * every entry. Per-entry dependency types, schemas, and `keepAliveEquivalence`
 * settings are preserved; each lifted entry's variant (with or without
 * `readDependencies`) matches its source entry's.
 */
<Subscriptions extends Readonly<Record<string, Subscription<any, any, any, any>>>>(subscriptions: Subscriptions): (config: {
  toChildModel: (parentModel: ParentModel) => ChildModelOf<Subscriptions>
  toParentMessage: (message: ChildMessageOf<Subscriptions>) => ParentMessage
}) => {
  readonly [K in string | number | symbol]: Subscriptions[K] extends Subscription<any, any, Dependencies, Services>
    ? Subscription<ParentModel, ParentMessage, Dependencies, Services>
    : never
}
```

### make

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/subscription.ts#L166)

```
/**
 * Declares a Subscriptions record. The Model, Message, and optional Services
 * generics are provided up front; the entries record follows, built from
 * calls to the `entry` builder passed into the inner function.
 * 
 * Reach for `Subscription.aggregate` to combine multiple records, and
 * `Subscription.lift` to translate a child Submodel's record into a parent
 * context.
 */
<Model, Message, Services = never>(): (build: (entry: EntryBuilder<Model, Message, Services>) => Entries) => {
  readonly [K in string | number | symbol]: Entries[K] & SubscriptionBrand
}
```

### persistent

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/subscription.ts#L226)

```
/**
 * Wraps a Stream as a Subscription entry whose lifecycle is independent of
 * the Model. The Stream runs for the lifetime of the Subscriptions record;
 * no Model change tears it down or restarts it. Use for any Stream whose
 * work doesn't depend on Model state, such as system theme listeners,
 * viewport width observers, or route-independent timers.
 * 
 * Returns an entry shape, not a branded Subscription. Pass it into `make`
 * as an entry value.
 */
<Message, Services = never>(stream: Stream<Message, never, Services>): EntryWithoutKeepAlive<unknown, Message, Record<string, never>, Services>
```

## Types

### AnimationFrameConfig

type

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/animationFrame.ts#L12)

```
/**
 * Configuration for the `animationFrame` Subscription helper.
 * 
 * `isActive(model)` controls whether the request-animation-frame loop is
 * scheduled at all. When it returns `false` (e.g. the game is paused, the
 * scene is static, or the canvas is offscreen), no rAF callbacks fire and
 * no Messages are emitted. The Subscription system automatically restarts
 * the loop when `isActive` flips back to `true`.
 */
type AnimationFrameConfig = Readonly<{
  isActive: (model: Model) => boolean
  toMessage: (deltaTime: number) => Message
}>
```

### EntryWithoutKeepAlive

type

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/subscription.ts#L16)

```
/**
 * The entry shape produced by helpers like `Subscription.persistent` and
 * `Port.subscription` before branding. Pass values of this shape into
 * `Subscription.make` as entry values.
 */
type EntryWithoutKeepAlive = {
  dependenciesSchema: DependenciesSchema<Dependencies>
  dependenciesToStream: (dependencies: Dependencies) => Stream.Stream<Message, never, Services>
  keepAliveEquivalence: never
  modelToDependencies: (model: Model) => Dependencies
}
```

### FromEventConfig

type

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/fromEvent.ts#L16)

```
/**
 * Configuration for the `fromEvent` Stream helper.
 * 
 * `target` is read inside the acquire Effect, never before it, so the
 * resolved `EventTarget` is captured at the moment the Subscription's scope
 * opens. Pass a thunk when the target may not exist until the scope opens, or
 * pass the `EventTarget` directly for always-present globals like `window` or
 * `document`.
 * 
 * `toMessage(event)` transforms each dispatched event into a Message. The
 * mapper runs synchronously in the same call stack as the browser's event
 * dispatch, so calling `event.preventDefault()` inside it works as expected.
 */
type FromEventConfig = Readonly<{
  options: AddEventListenerOptions
  target: EventTarget | () => EventTarget
  toMessage: (event: EventType) => Message
  type: string
}>
```

### FromEventFilterMapConfig

type

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/fromEvent.ts#L41)

```
/**
 * Configuration for the `fromEventFilterMap` Stream helper.
 * 
 * `target` is read inside the acquire Effect, never before it, so the
 * resolved `EventTarget` is captured at the moment the Subscription's scope
 * opens. Pass a thunk when the target may not exist until the scope opens, or
 * pass the `EventTarget` directly for always-present globals like `window` or
 * `document`.
 * 
 * `toMessage(event)` returns `Option.some(message)` to emit a Message for the
 * event, or `Option.none()` to ignore it. The mapper runs synchronously in the
 * same call stack as the browser's event dispatch, so calling
 * `event.preventDefault()` inside it works as expected.
 */
type FromEventFilterMapConfig = Readonly<{
  options: AddEventListenerOptions
  target: EventTarget | () => EventTarget
  toMessage: (event: EventType) => Option.Option<Message>
  type: string
}>
```

### Subscription

type

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/subscription.ts#L64)

```
/**
 * A single subscription entry produced by `Subscription.make`,
 * `Subscription.lift`, or `Subscription.aggregate`. The brand field is
 * `never`, so application code cannot manually construct a `Subscription`
 * value: it must go through one of those constructors (or a helper like
 * `Subscription.persistent` that returns an entry shape, then through
 * `make`).
 * 
 * Two variants by `keepAliveEquivalence` presence:
 * 
 * - Without `keepAliveEquivalence` (the common case), every Model change recomputes
 *   the dependencies. Equivalent dependencies leave the Stream alone; any
 *   change tears it down and restarts. `dependenciesToStream` takes a single
 *   argument: the latest dependencies.
 * - With `keepAliveEquivalence` (an escape hatch), Model changes that the
 *   equivalence treats as equal leave the Stream running, but the running
 *   Stream can still read the latest dependencies via the second
 *   `readDependencies` argument. Use this when the Stream needs mid-flight
 *   access to data that changes often but shouldn't trigger restarts
 *   (Foldkit UI's `DragAndDrop.autoScroll` reading the latest pointer
 *   `clientY` each rAF tick is the canonical example).
 * 
 * `dependenciesSchema` must be a `Schema.Struct` so every dependency is
 * explicitly named at the schema level.
 */
type Subscription = Entry<Model, Message, Dependencies, Services> & SubscriptionBrand
```

### Subscriptions

type

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/foldkit/src/subscription/subscription.ts#L72)

```
/** A record of named Subscriptions keyed by dependency field name. */
type Subscriptions = Readonly<Record<string, Subscription<Model, Message, any, Services>>>
```
