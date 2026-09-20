---
url: https://foldkit.dev/api-reference/subscription
title: "Subscription"
description: "API documentation for the Subscription module."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

# Subscription

## Functions

### animationFrame

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/animationFrame.ts#L66)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/fromEvent.ts#L394)

```
/**
 * Build a Stream that emits a value for every dispatch of a DOM event,
 * registering the listener when the Stream's scope opens and removing it when
 * the scope closes.
 * 
 * The target, the event name, and the event the mapper receives are one fact:
 * `type` is constrained to the names the target declares, and the mapper's
 * parameter is what those two resolve to, so annotating it narrows nothing and
 * cannot contradict the name. A target that is neither annotated nor one
 * lib.dom declares a map for accepts any name and reports `Event`; annotate it
 * with TypedEventTarget to resolve its own events.
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
 * Model condition. The mapper's output type is inferred (even a raw Event is
 * accepted here); `Subscription.make` checks the final Stream against the
 * application's Message type.
 * 
 * For a listener that reacts to only some events, reach for
 * `fromEventFilterMap`, whose mapper returns `Option<Output>`. For a
 * listener that also cancels the default action of the events it handles,
 * reach for `fromEventFilterMapPreventDefault`.
 */
<Target extends EventTarget, Type extends string, Output>(config: FromEventConfig<Target, Type, Output>): Stream<Output>
```

### fromEventFilterMap

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/fromEvent.ts#L335)

```
/**
 * Build a Stream that emits a value for the dispatches of a DOM event the
 * mapper chooses to keep, registering the listener when the Stream's scope
 * opens and removing it when the scope closes.
 * 
 * This is the filtered variant of `fromEvent`. Its `filterMapEvent` returns
 * `Option.some(value)` to emit and `Option.none()` to ignore the event, so a
 * single listener can react to some dispatches while passing on the rest. A
 * mapper that never emits produces a `Stream<never>`.
 * 
 * Reach for this over a downstream `Stream.filterMap` whenever the decision to
 * keep an event is paired with `event.preventDefault()`. The mapper runs
 * synchronously inside the browser's event dispatch, so `preventDefault()`
 * takes effect, while a downstream filter would run on a later turn after the
 * default action has already happened. The exception is a passive listener,
 * which ignores `preventDefault()`. Some browsers default wheel and touch
 * listeners on global targets to passive. Pass
 * `options: { passive: false }` explicitly when cancelling those events, or
 * reach for `fromEventFilterMapPreventDefault`, which does so for you.
 * 
 * The target, the event name, and the event the mapper receives are one fact:
 * `type` is constrained to the names the target declares, and the mapper's
 * parameter is what those two resolve to, so annotating it narrows nothing and
 * cannot contradict the name. A target that is neither annotated nor one
 * lib.dom declares a map for accepts any name and reports `Event`; annotate it
 * with TypedEventTarget to resolve its own events.
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
 * Model condition. The mapper's output type is inferred (even a raw Event is
 * accepted here); `Subscription.make` checks the final Stream against the
 * application's Message type.
 */
<Target extends EventTarget, Type extends string, Output>(config: FromEventFilterMapConfig<Target, Type, Output>): Stream<Output>
```

### fromEventFilterMapPreventDefault

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/fromEvent.ts#L465)

```
/**
 * Build a Stream that emits a value for the dispatches of a DOM event the
 * mapper marks handled, calling `event.preventDefault()` on each of them,
 * registering the listener when the Stream's scope opens and removing it when
 * the scope closes.
 * 
 * This is the cancelling variant of `fromEventFilterMap`, mirroring
 * `h.OnKeyDownPreventDefault` from `foldkit/html`. Its `filterMapEvent` returns
 * `Option.some(value)` to mark a dispatch handled. The helper evaluates the
 * mapper, calls `event.preventDefault()`, and queues the value before the
 * native listener returns. `Option.none()` leaves the default behavior intact.
 * The mapper never calls `preventDefault()` itself.
 * 
 * Because cancelling is the point, the listener registers with
 * `passive: false` when the config does not say otherwise. This keeps wheel
 * and touch events cancelable when a browser would otherwise make listeners
 * on a global target passive. The config rejects `passive: true`; the runtime
 * guard also throws for unchecked JavaScript inputs.
 * 
 * The target, event name, and mapper parameter are one fact: `type` is
 * constrained to the names the target declares, and the mapper receives the
 * event those two resolve to. A target with no declared event map accepts any
 * name and reports `Event`; annotate it with TypedEventTarget to
 * resolve its own events.
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
 * Model condition. The mapper's output type is inferred (even a raw Event is
 * accepted here); `Subscription.make` checks the final Stream against the
 * application's Message type.
 */
<Target extends EventTarget, Type extends string, Output>(config: FromEventFilterMapPreventDefaultConfig<Target, Type, Output>): Stream<Output>
```

### keyBindings

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/keyBindings.ts#L846)

```
/**
 * Build a Stream that maps declarative key bindings to values. The output is
 * inferred from each binding's `mapEvent` callback; `Subscription.make`
 * checks that the final Stream emits the application's Message type.
 * 
 * A string describes one key press. Modifiers are joined with `+`:
 * `'Mod+K'`, `'Control+Shift+P'`, or `'Alt+ArrowDown'`. The supported modifiers
 * are `Mod`, `Control`, `Meta`, `Alt`, and `Shift`. `Mod` resolves to Meta on
 * Apple platforms and Control elsewhere; `modKey` can override that choice.
 * Matching uses `KeyboardEvent.key`, case-insensitively, after the active
 * keyboard layout has been applied. Use `Space` and `Plus` for those keys.
 * 
 * An array describes an ordered sequence of two or more presses. Every press
 * uses the same grammar, so `['G', 'Shift+G']` is valid. Sequences reset after
 * one second by default; `sequenceTimeout` accepts any Effect Duration input.
 * Modifier-only events and repeated keydowns do not advance a sequence.
 * 
 * Bindings are suppressed by default when the event's composed path contains
 * an `input`, `textarea`, `select`, or contenteditable element. Set
 * `whileTyping` to `'Allow'` for a binding that must work there. Events emitted
 * during IME composition are always ignored. Repeated keydowns are ignored for
 * one-press bindings unless `whenRepeated` is `'Allow'`. An event another
 * handler already canceled is ignored and clears any sequence in progress.
 * 
 * Matched key presses call `preventDefault()` before dispatching. For a
 * sequence, that policy applies to every matched press. Set `preventDefault`
 * to `false` to opt out. Sequences sharing a prefix must use the same policy.
 * Duplicate bindings and a complete binding that is also a sequence prefix
 * are rejected when the Stream is created.
 * 
 * This helper returns a Stream, not a complete Subscription entry. Use
 * `Subscription.persistent` for a fixed table. When availability depends on
 * the Model that owns the entry, build it inside `dependenciesToStream` and
 * derive each binding's `isEnabled` from the dependency record. A dependency
 * change opens a new Stream scope and resets any sequence in progress. If a
 * parent owns a condition for a lifted child, declare the table at that parent
 * or put bindings with different parent-owned lifetimes in separate child
 * entries so `Subscription.lift` can gate them individually. If the meaning
 * of a key depends on the Model, dispatch a factual key Message and decide
 * what it means in update instead of reading the Model from `mapEvent`.
 */
<Output>(config: KeyBindingsConfig<Output>): Stream<Output>
```

### lift

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L682)

```
/**
 * Lifts a record of child Subscriptions into a parent's Model and Message
 * context, applying a Model accessor and a Message wrapper uniformly to
 * every entry. Per-entry dependency types, schemas, and `keepAliveEquivalence`
 * settings are preserved; each lifted entry's variant (with or without
 * `readDependencies`) matches its source entry's.
 * 
 * The optional `when` is the parent's own gate. The parent writes it here on
 * its `lift` call and answers it from the parent Model, which is what makes
 * it useful: it carries the half of a condition the child cannot see, such as
 * the route a page Submodel sits behind. The child neither declares nor sees
 * the gate, and keeps holding its own half in `modelToDependencies`. A gated
 * entry runs only while its gate returns `true`, and a closed gate tears it
 * down.
 * 
 * `when` takes either shape:
 * 
 * - One predicate gates every entry in the record, for the common case where
 *   the whole child answers to one parent condition.
 * - An EntryGates map gates entries by name, for a child whose
 *   Subscriptions answer to different parent conditions. Entries the map
 *   omits are lifted ungated. A child never has to organize its records
 *   around its parent's gating.
 * 
 * Gating rewrites a gated entry's dependencies to GatedDependencies,
 * so its `readDependencies` returns the last dependencies seen through an
 * open gate. Ungated entries keep the child's dependencies untouched. Passing
 * `ParentModel` and `ParentMessage` explicitly suppresses inference on the
 * gate map, which leaves each named entry's dependencies as either shape; let
 * both infer from an annotated `toChildModel` when you want the exact per
 * entry types.
 */
<Subscriptions extends Readonly<Record<string, Subscription<any, any, any, any>>>>(subscriptions: Subscriptions): (config: GatedLiftConfig<ParentModel, ParentMessage, Subscriptions>) => GatedLiftedSubscriptions<ParentModel, ParentMessage, Subscriptions>
```

### make

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L174)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L365)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/animationFrame.ts#L12)

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

### EntryGates

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L419)

```
/**
 * The per-entry form of a lift's `when`: a partial map from entry name to
 * gate. Entries the map names are gated, and entries it omits are lifted
 * ungated.
 */
type EntryGates = Readonly<Partial<Record<keyof Subscriptions, WhenPredicate<ParentModel>>>>
```

### EntryWithoutKeepAlive

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L24)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/fromEvent.ts#L149)

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
 * `type` is constrained to the event names the target declares, and
 * `mapEvent`'s parameter is the event those two resolve to. Annotating that
 * parameter is checked against the resolved event rather than replacing it.
 * 
 * `mapEvent(event)` transforms each dispatched event into a Stream value. The
 * mapper runs synchronously in the same call stack as the browser's event
 * dispatch, so calling `event.preventDefault()` inside it takes effect,
 * unless the listener is passive. Some browsers default wheel and touch
 * listeners on global targets to passive, where `preventDefault()` is
 * ignored. Pass `options: { passive: false }` explicitly when cancelling
 * those events, or reach for `fromEventFilterMapPreventDefault`, which does
 * so for you.
 * 
 * The output type is inferred from the mapper; `Subscription.make` checks
 * that the final Stream emits the application's Message type.
 */
type FromEventConfig = Readonly<{
  mapEvent: (event: EventOf<Target, Type>) => Output
  options: AddEventListenerOptions
  target: Target | () => Target
  type: Type
}>
```

### FromEventFilterMapConfig

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/fromEvent.ts#L185)

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
 * `type` is constrained to the event names the target declares, and
 * `filterMapEvent`'s parameter is the event those two resolve to. Annotating that
 * parameter is checked against the resolved event rather than replacing it.
 * 
 * `filterMapEvent(event)` returns `Option.some(value)` to emit a value for the
 * event, or `Option.none()` to ignore it. The mapper runs synchronously in the
 * same call stack as the browser's event dispatch, so calling
 * `event.preventDefault()` inside it takes effect, unless the listener is
 * passive. Some browsers default wheel and touch listeners on global targets
 * to passive, where `preventDefault()` is ignored. Pass
 * `options: { passive: false }` explicitly when cancelling those events, or
 * reach for `fromEventFilterMapPreventDefault`, which does so for you.
 * 
 * The output type is inferred from the mapper; `Subscription.make` checks
 * that the final Stream emits the application's Message type.
 */
type FromEventFilterMapConfig = Readonly<{
  filterMapEvent: (event: EventOf<Target, Type>) => Option.Option<Output>
  options: AddEventListenerOptions
  target: Target | () => Target
  type: Type
}>
```

### FromEventFilterMapPreventDefaultConfig

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/fromEvent.ts#L260)

```
/**
 * Configuration for the `fromEventFilterMapPreventDefault` Stream helper.
 * 
 * `target` is read inside the acquire Effect, never before it, so the
 * resolved `EventTarget` is captured at the moment the Subscription's scope
 * opens. Pass a thunk when the target may not exist until the scope opens, or
 * pass the `EventTarget` directly for always-present globals like `window` or
 * `document`.
 * 
 * `type` is constrained to the event names the target declares, and
 * `filterMapEvent`'s parameter is the event those two resolve to. Annotating that
 * parameter is checked against the resolved event rather than replacing it.
 * 
 * `filterMapEvent(event)` returns `Option.some(value)` to mark the dispatch
 * handled, or `Option.none()` to leave the default behavior intact. For a
 * handled dispatch the helper calls `event.preventDefault()` and queues the
 * value before the listener returns; the mapper itself never calls
 * `preventDefault()`.
 * 
 * `options.passive` defaults to `false` so `preventDefault()` keeps working
 * for the events browsers would otherwise register as passive. The config
 * rejects `passive: true`; the runtime guard also throws for unchecked
 * JavaScript inputs.
 * 
 * The output type is inferred from the mapper; `Subscription.make` checks
 * that the final Stream emits the application's Message type.
 */
type FromEventFilterMapPreventDefaultConfig = Readonly<{
  filterMapEvent: (event: EventOf<Target, Type>) => Option.Option<Output>
  options: PreventDefaultEventListenerOptions
  target: Target | () => Target
  type: Type
}>
```

### GatedDependencies

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L408)

```
/**
 * The dependencies of a Subscription lifted through a parent's `when` gate:
 * the child entry's own dependencies under `maybeDependencies`, and `None`
 * for as long as the parent holds the gate closed.
 * 
 * A closed gate is a real teardown rather than a paused Stream. The entry's
 * Stream is torn down, and the child's `modelToDependencies` does not run
 * again until the parent reopens the gate, so child state that changes behind
 * a closed gate causes no restarts.
 */
type GatedDependencies = Readonly<{
  maybeDependencies: Option.Option<Dependencies>
}>
```

### KeyBinding

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/keyBindings.ts#L36)

```
/**
 * One entry in a keyBindings binding table.
 * 
 * A string describes one key press, such as `'/'`, `'Escape'`, or `'Mod+K'`.
 * An array describes a sequence of at least two presses, such as
 * `['G', 'H']` or `['G', 'Shift+G']`.
 */
type KeyBinding = BindingBase<Output> & Readonly<{
  keys: string
  whenRepeated: "Ignore" | "Allow"
}> | Readonly<{
  keys: Readonly<[string, string, ...Array<string>]>
  whenRepeated: never
}>
```

### KeyBindingsConfig

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/keyBindings.ts#L51)

```
/** Configuration for the keyBindings Stream helper. */
type KeyBindingsConfig = Readonly<{
  bindings: ReadonlyArray<KeyBinding<Output>>
  modKey: ModKey
  sequenceTimeout: Duration.Input
  target: EventTarget | () => EventTarget
}>
```

### KeySequence

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/keyBindings.ts#L19)

```
/** A single key press or a sequence of two or more key presses. */
type KeySequence = string | Readonly<[string, string, ...Array<string>]>
```

### Subscription

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L72)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L80)

```
/** A record of named Subscriptions keyed by dependency field name. */
type Subscriptions = Readonly<Record<string, Subscription<Model, Message, any, Services>>>
```

### WhileTyping

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/keyBindings.ts#L16)

```
/** Whether a key binding may fire when its event comes from an editable element. */
type WhileTyping = "Suppress" | "Allow"
```

## Interfaces

### TypedEventTarget

interface

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/fromEvent.ts#L25)

```
/**
 * An `EventTarget` that declares the events it dispatches, so the `fromEvent`
 * helpers can resolve an event name to its event type the way they do for
 * `window`, `document`, and the DOM interfaces lib.dom declares event maps
 * for.
 * 
 * Annotate a target with this and the mapper's parameter follows from the
 * event name, including a `CustomEvent`'s `detail`. A declared event overrides
 * the corresponding native event and otherwise augments the target's native
 * events, so an element that dispatches custom events can be annotated without
 * losing events such as `click`. Any `EventTarget` is assignable to it, so the
 * annotation is the only change needed.
 */
interface TypedEventTarget {
  [EventMapMarker]: EventMap
  addEventListener: unknown
  dispatchEvent: unknown
  removeEventListener: unknown
}
```

## Constants

### aggregate

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/subscription/subscription.ts#L336)

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
