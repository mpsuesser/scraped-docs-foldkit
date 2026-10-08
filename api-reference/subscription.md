---
url: https://foldkit.dev/api-reference/subscription
title: "Subscription"
description: "API documentation for the Subscription module."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

# Subscription

## Functions

### animationFrameEntry

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/animationFrame.ts#L66)

```
/**
 * Build a Subscription entry that emits a Message on every
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

### lift

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L580)

```
/**
 * Lifts child Subscriptions into a parent's Model and Message context.
 * `read` returns `Some(childModel)` while the child exists and `None` while
 * it is absent, matching `Update.foldChild` and `ManagedResource.lift`.
 * An absent child tears down every entry's Stream without reading the
 * child's dependencies. Use `Option.some` for a child that is always present.
 * 
 * The optional `when` adds conditions from the parent Model. One predicate
 * gates every entry; an EntryGates map adds a gate only to its named
 * entries. An entry runs only while its gate is open and `read` returns
 * `Some`. A closed gate skips `read` as well as the child's dependency
 * projection. Entries omitted from a gate map still stop when the child
 * is absent.
 * 
 * Every lifted entry wraps its dependencies in GatedDependencies.
 * The child's dependency Schema, service requirements, and
 * `keepAliveEquivalence` are preserved inside that wrapper. A running child
 * Stream's `readDependencies` receives current child dependencies while
 * active, with its starting dependencies as a fallback during teardown.
 */
<Subscriptions extends Readonly<Record<string, Subscription<any, any, any, any>>>>(subscriptions: Subscriptions): (config: LiftConfig<ParentModel, ParentMessage, Subscriptions>) => LiftedSubscriptions<ParentModel, ParentMessage, Subscriptions>
```

### make

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L175)

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

### persistentEntry

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L366)

```
/**
 * Wraps a Stream as a Subscription entry with no dependencies on its own
 * Model. Local Model changes do not restart the Stream. A parent can still
 * gate the entry when lifting it, so the Stream starts and stops with that
 * parent condition. Use for work such as system theme listeners, viewport
 * width observers, or route-independent timers.
 * 
 * Returns an entry shape, not a branded Subscription. Pass it into `make`
 * as an entry value.
 */
<Message, Services = never>(stream: Stream<Message, never, Services>): EntryWithoutKeepAlive<unknown, Message, Record<string, never>, Services>
```

## Types

### AnimationFrameConfig

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/animationFrame.ts#L12)

```
/**
 * Configuration for the `animationFrameEntry` Subscription helper.
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

### EntryGates

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L415)

```
/**
 * Additional parent conditions for named entries in a lift's `when` map.
 * Entries omitted from the map run whenever `read` returns a child Model.
 */
type EntryGates = Readonly<Partial<Record<keyof Subscriptions, WhenPredicate<ParentModel>>>>
```

### EntryWithoutKeepAlive

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L25)

```
/**
 * The entry shape produced by helpers like `Subscription.persistentEntry` and
 * `Port.subscriptionEntry` before branding. Pass values of this shape into
 * `Subscription.make` as entry values.
 */
type EntryWithoutKeepAlive = {
  dependenciesSchema: DependenciesSchema<Dependencies>
  dependenciesToStream: (dependencies: Dependencies) => Stream.Stream<Message, never, Services>
  keepAliveEquivalence: never
  modelToDependencies: (model: Model) => Dependencies
}
```

### GatedDependencies

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L405)

```
/**
 * The dependencies of a lifted Subscription. `maybeDependencies` holds the
 * child's dependencies while `read` returns `Some` and its `when` gate is
 * open. Otherwise it holds `None`, which tears down the entry's Stream and
 * skips the child's `modelToDependencies`.
 */
type GatedDependencies = Readonly<{
  maybeDependencies: Option.Option<Dependencies>
}>
```

### Subscription

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L73)

```
/**
 * A single subscription entry produced by `Subscription.make`,
 * `Subscription.lift`, or `Subscription.aggregate`. The brand field is
 * `never`, so application code cannot manually construct a `Subscription`
 * value: it must go through one of those constructors (or a helper like
 * `Subscription.persistentEntry` that returns an entry shape, then through
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

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L81)

```
/** A record of named Subscriptions keyed by dependency field name. */
type Subscriptions = Readonly<Record<string, Subscription<Model, Message, any, Services>>>
```

## Constants

### aggregate

const

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/subscription/subscription.ts#L337)

```
/**
 * Combines multiple Subscriptions records into one. Throws on duplicate
 * keys so a misconfigured aggregate fails loudly at startup rather than
 * silently overriding.
 * 
 * Pass the records directly and the Model, Message, and Services are read
 * off them. The Model of the first record with a Model dependency is the one
 * every later record is checked against, so a record from another Model
 * universe fails at its own argument position. Message and Services widen to
 * the union across all records, which is what lets a record that needs an
 * Effect service sit beside records that need none.
 * 
 * The result keeps each record's keys and each entry's exact dependency type,
 * schema, and `keepAliveEquivalence` variant, so a lifted entry's
 * GatedDependencies survives aggregation.
 * 
 * The curried form remains available for a record that has to be typed before
 * its entries exist, such as a value annotated as
 * `Subscriptions<Model, Message>` at a module boundary. It erases keys and
 * per-entry dependency types, so reach for it only when the explicit contract
 * is the point.
 */
const aggregate: () => (records: readonly Array<Readonly<Record<string, Subscription<Model, Message, any, Services>>>>) => Subscriptions<Model, Message, Services>
```
