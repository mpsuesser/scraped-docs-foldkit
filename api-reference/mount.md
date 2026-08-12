---
url: https://foldkit.dev/api-reference/mount
title: "Mount"
description: "API documentation for the Mount module."
access_date: 2026-08-12T01:45:44.345Z
current_date: 2026-08-12T01:45:44.345Z
---

# Mount

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/mount/index.ts#L210)

```
/**
 * Defines a one-shot Mount. The factory returns `Effect<Message>` that runs
 * once when the element mounts and produces exactly one Message. Cleanup
 * composes via `Effect.acquireRelease` inside the Effect: registered
 * finalizers run when the element unmounts. The Mount's scope stays open
 * across the element's full lifetime, even after the Effect completes.
 * 
 * At least one result Message schema is required. The Effect's success
 * type is `Schema.Schema.Type<Results[number]>`; without a declared
 * result, the factory would have to return `Effect.never`, leaving
 * `update` with no record of the work and removing DevTools, Scene,
 * and time-travel replay's reference point. Fire-and-forget Mounts
 * follow the same convention as fire-and-forget Commands: declare a
 * `Completed*` result Message that `update` no-ops on. The side
 * effect stays observable; `update` simply has nothing meaningful to
 * do with the acknowledgment.
 * 
 * Two forms, distinguished by whether the second argument is a Schema (a
 * result message) or a record of Schemas (the args declaration). Cleanup is
 * asynchronous with respect to snabbdom's `destroy` hook: the runtime forks
 * `Fiber.interrupt` and returns immediately, so finalizers run on a separate
 * fiber after `destroy` has already completed. For idempotent DOM operations
 * (`element.remove()`, observer `disconnect()`, `removeEventListener`) this
 * is fine; if your cleanup has ordering requirements relative to other DOM
 * removals, prefer doing the imperative work synchronously inside `acquire`
 * and using `release` only for self-contained teardown.
 * 
 * **Construct resources INSIDE the acquire body, never before it.**
 * `Effect.acquireRelease` only guarantees atomicity of "acquire body
 * completes → release is registered". If you construct a handle before
 * calling `acquireRelease` and your acquire body just returns that handle
 * (`Effect.sync(() => alreadyExistingValue)`), interruption between the
 * construction and the registration leaks the handle. For third-party
 * library instantiation, express the construction as the success value of
 * the acquire Effect: `Effect.tryPromise(() => import(...)).pipe(Effect.map(...))`
 * for async imports, `Effect.sync(() => new Thing(...))` for sync
 * construction. The discipline: whatever the release function needs as
 * input must be the success value of the acquire Effect.
 * 
 * Use this form whenever a Mount produces a single Message at acquire and
 * holds lifecycle-scoped resources for the element's lifetime. For Mounts
 * that emit a continuum of events (scroll listeners, IntersectionObservers,
 * MutationObservers), reach for `Mount.defineStream`.
 */
<Name extends string, Results extends readonly [Top, Top]>(
  name: Name,
  results: Results
): (factory: (element: Element) => Effect<Type<Results[number]>, never, Scope>) => MountDefinitionNoArgs<Name, Type<Results[number]>>

<Name extends string, Fields extends Fields, Results extends readonly [Top, Top]>(
  name: Name,
  args: Fields,
  results: Results
): (factoryBuilder: (args: View<Fields>) => (element: Element) => Effect<Type<Results[number]>, never, Scope>) => MountDefinitionWithArgs<Name, Fields, Type<Results[number]>>
```

### defineStream

function

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/mount/index.ts#L380)

```
/**
 * Defines a streaming Mount. The factory returns `Stream<Message>` whose
 * lifetime is bound to the element's lifetime: each emitted Message is
 * dispatched, and the Stream's scope is closed (running any registered
 * `Effect.acquireRelease` finalizers) when the element unmounts. Use this
 * form when the Mount emits a continuum of events from observers or
 * listeners attached to the element.
 * 
 * At least one result Message schema is required. The Stream's emission
 * type is `Schema.Schema.Type<Results[number]>`; without a declared
 * result, the factory would have to return `Stream<never>`, leaving
 * `update` with no record of the work and removing DevTools, Scene,
 * and time-travel replay's reference point. Fire-and-forget Mounts
 * follow the same convention as fire-and-forget Commands: declare a
 * `Completed*` result Message that `update` no-ops on. The side
 * effect stays observable; `update` simply has nothing meaningful to
 * do with the acknowledgment. Re-check the cause.
 * 
 * Two forms, distinguished by whether the second argument is a Schema or a
 * record of Schemas (the args declaration). Cleanup timing relative to
 * snabbdom's `destroy` hook is the same as `Mount.define` (asynchronous via
 * `Fiber.interrupt`).
 * 
 * For a Mount that produces exactly one Message at acquire and then holds
 * lifecycle-scoped resources, use `Mount.define` with `Effect<Message>`.
 * That form encodes "exactly one Message" in the type system. Reserve
 * `defineStream` for cases that genuinely emit a stream of events.
 */
<Name extends string, Results extends readonly [Top, Top]>(
  name: Name,
  results: Results
): (factory: (element: Element) => Stream<Type<Results[number]>, never, never>) => MountDefinitionNoArgs<Name, Type<Results[number]>>

<Name extends string, Fields extends Fields, Results extends readonly [Top, Top]>(
  name: Name,
  args: Fields,
  results: Results
): (factoryBuilder: (args: View<Fields>) => (element: Element) => Stream<Type<Results[number]>, never, never>) => MountDefinitionWithArgs<Name, Fields, Type<Results[number]>>
```

## Types

### MountAction

type

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/mount/index.ts#L46)

```
/**
 * A named, type-constrained per-element side effect, optionally carrying the
 *  args used to construct it. The runtime invokes `f` with the live `Element`
 *  when the element mounts, and dispatches each Message emitted by the
 *  returned Stream. The Stream's scope is tied to the element's lifetime: when
 *  the element unmounts, the runtime interrupts the fiber, which closes the
 *  Stream's scope and runs any registered `acquireRelease` finalizers.
 * 
 *  Authors typically don't construct Stream-returning factories directly.
 *  `Mount.define` wraps an `Effect<Message>` for the one-shot case; only
 *  `Mount.defineStream` exposes the raw Stream shape for continuous-event
 *  cases.
 */
type MountAction = Readonly<{
  args: Record<string, unknown>
  f: (element: Element) => Stream.Stream<Message, E>
  name: string
}>
```

### MountDefinition

type

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/mount/index.ts#L80)

```
/**
 * A Mount definition created with `Mount.define` or `Mount.defineStream`.
 *  Union over the no-args and with-args shapes; consumers that only need
 *  name/identity can accept this.
 */
type MountDefinition = MountDefinitionNoArgs<Name, ResultMessage> | MountDefinitionWithArgs<Name, any, ResultMessage>
```

## Interfaces

### MountDefinitionNoArgs

interface

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/mount/index.ts#L53)

```
/** A Mount definition for a Mount with no declared args. Call as `Definition()` to produce a MountAction. */
interface MountDefinitionNoArgs {
  [MountDefinitionTypeId]: typeof MountDefinitionTypeId
  name: Name
}
```

### MountDefinitionWithArgs

interface

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/mount/index.ts#L63)

```
/** A Mount definition for a Mount with declared args. Call as `Definition(args)` to produce a MountAction. */
interface MountDefinitionWithArgs {
  [MountDefinitionTypeId]: typeof MountDefinitionTypeId
  name: Name
}
```

## Constants

### MountDefinitionTypeId

const

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/mount/index.ts#L28)

```
/** Type-level brand for MountDefinition values. */
const MountDefinitionTypeId: unique symbol
```

### mapMessage

const

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/mount/index.ts#L460)

```
/**
 * Lifts a `MountAction` from one Message universe to another by mapping its
 *  dispatched Messages through a transform. Used by Submodel components to
 *  emit lifecycle action results into the parent's Message union via the
 *  consumer-supplied `toParentMessage` lift. Preserves `name` and `args`.
 */
const mapMessage: (f: (message: A) => B) => (action: MountAction<A, E>) => MountAction<B, E>
```
