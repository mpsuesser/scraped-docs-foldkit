---
url: https://foldkit.dev/api-reference/managed-resource
title: "ManagedResource"
description: "API documentation for the ManagedResource module."
access_date: 2026-08-12T20:07:47.178Z
current_date: 2026-08-12T20:07:47.178Z
---

# ManagedResource

## Functions

### aggregate

function

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L403)

```
/**
 * Combines multiple Managed Resources records into one. Throws on duplicate
 * keys so a misconfigured aggregate fails loudly at startup rather than
 * silently overriding.
 */
<Model, Message>(): (records: Records) => MergeRecords<Records>
```

### lift

function

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L340)

```
/**
 * Lifts a record of child Managed Resources into a parent's Model and Message
 * context, applying a Model accessor and a Message wrapper uniformly to every
 * entry. Per-entry requirements schemas and resource services are preserved.
 * 
 * Unlike `Subscription.lift`, `toChildModel` returns an `Option`: a managed
 * resource already speaks in `Option` (`modelToMaybeRequirements` returns
 * `Option.none()` to release), and a child Submodel that owns a managed
 * resource is itself something that mounts and unmounts. A missing child is
 * just another `None` and flows through the same acquire/release channel, so
 * each lifted entry's requirements must be `S.Option`-wrapped.
 */
<Resources extends Record<string, Entry<any, any, Option<any>, any, any, (value: any) => any>>>(resources: Resources): (config: {
  toChildModel: (parentModel: ParentModel) => Option<ChildModelOf<Resources>>
  toParentMessage: (message: ChildMessageOf<Resources>) => ParentMessage
}) => {
  readonly [Key in string | number | symbol]: Resources[Key] extends Entry<any, any, Requirements, Value, Service, OnAcquired>
    ? Entry<ParentModel, ParentMessage, Requirements, Value, Service, (args: Parameters<OnAcquired>) => ParentMessage>
    : never
}
```

### make

function

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L292)

```
/**
 * Declares a Managed Resources record. The Model and Message generics are
 * provided up front; the entries record follows, built from calls to the
 * `entry` builder passed into the inner function.
 * 
 * Use this when a resource is expensive or stateful and should only exist while
 * the model is in a particular state: a camera stream during a video call, a
 * WebSocket connection while on a chat page, or a Web Worker pool during a
 * computation. For resources that live for the entire application lifetime, use
 * the static `resources` config instead.
 * 
 * Reach for `ManagedResource.aggregate` to combine multiple records, and
 * `ManagedResource.lift` to translate a child Submodel's record into a parent
 * context.
 * 
 * **Lifecycle** — The runtime watches each entry's `modelToMaybeRequirements`
 * after every model update, structurally comparing the result against the
 * previous value:
 * 
 * - `Option.none()` → `Option.some(params)`: calls `acquire(params)`, then
 *   dispatches `onAcquired(value)`.
 * - `Option.some(paramsA)` → `Option.some(paramsB)` (structurally different):
 *   releases the old resource, then acquires a new one with `paramsB`.
 * - `Option.some(params)` → `Option.none()`: calls `release(value)`, then
 *   dispatches `onReleased()`. No re-acquisition occurs.
 * 
 * If `acquire` fails, `onAcquireError` is dispatched and the resource daemon
 * continues watching for the next requirements change: a failed acquisition
 * does not crash the application.
 * 
 * **Config fields:**
 * 
 * - `resource` — The identity tag created with `ManagedResource.tag`. Appears
 *   in the Effect R channel so commands that call `.get` are type-checked.
 * - `modelToMaybeRequirements` — Extracts requirements from the model.
 *   `Option.none()` means "release", `Option.some(params)` means
 *   "acquire/re-acquire if params changed". For resources with no
 *   parameters, use `S.Option(S.Null)` and return `Option.some(null)`.
 * - `acquire` — Creates the resource from the unwrapped params. The returned
 *   Effect should fail when acquisition fails: errors in the error channel
 *   flow to `onAcquireError` as a message instead of crashing the runtime.
 *   `acquire` runs with the resource-lifetime `Scope.Scope` in its context, so
 *   it can build an Effect `Layer` with `Layer.build` or register finalizers
 *   with `Effect.addFinalizer` whose teardown is tied to the resource. Those
 *   finalizers run when the resource is released or re-acquired, after the
 *   explicit `release` callback (scope finalizers run in last-in-first-out
 *   order, and the runtime registers `release` after `acquire` completes). A
 *   `Layer`-built resource therefore needs only `release: () => Effect.void`:
 *   the `Layer` finalizers run automatically when the scope closes.
 * - `release` — Tears down the resource. Errors thrown here are silently
 *   swallowed: release must not block cleanup. Resources that register their
 *   teardown as scope finalizers in `acquire` leave this as `() => Effect.void`.
 * - `onAcquired` — Message dispatched when `acquire` succeeds.
 * - `onAcquireError` — Message dispatched when `acquire` fails.
 * - `onReleased` — Message dispatched after `release` completes.
 */
<Model, Message>(): (build: (entry: EntryBuilder<Model, Message>) => Entries) => Entries
```

### tag

function

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L48)

```
/** Creates a managed resource identity with a `.get` accessor for use in commands. */
<Value>(): (key: Key) => ManagedResource<Value, ManagedResourceService<Key>>
```

## Types

### Entry

type

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L133)

```
/**
 * A single Managed Resource entry produced by `ManagedResource.make`,
 * `ManagedResource.lift`, or `ManagedResource.aggregate`. The brand field is
 * `never`, so application code cannot manually construct one: it must go
 * through a constructor.
 * 
 * The `Value` parameter is the type the resource holds while active: what
 * `acquire` produces, what `release` and `onAcquired` receive, and what the
 * tag's `.get` yields to commands. It is inferred from the `resource` tag.
 * 
 * The `Service` parameter carries the resource tag's identity so `make`,
 * `lift`, and `aggregate` can union the services a record requires. Read the
 * union off a finished record with `ManagedResource.ServicesOf`.
 * 
 * The `OnAcquired` parameter preserves the arity of the `onAcquired` handler:
 * an entry whose handler ignores the acquired value keeps that fact in its
 * type, so callers that replay the handler (the Scene test steps) only demand
 * the value when the handler consumes it.
 */
type Entry = {
  acquire: (params: AcquireParams<Requirements>) => Effect.Effect<Value, unknown, Scope.Scope>
  modelToMaybeRequirements: (model: Model) => Requirements
  onAcquired: OnAcquired
  onAcquireError: (error: unknown) => Message
  onReleased: () => Message
  release: (value: Value) => Effect.Effect<void>
  resource: ManagedResource<Value, Service>
  schema: Schema.Schema<Requirements>
} & EntryBrand<Model, Message>
```

### ManagedResourceConfig

type

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L83)

```
/** Internal configuration for a single Managed Resource, used by the runtime. */
type ManagedResourceConfig = {
  acquire: (params: any) => Effect.Effect<any, unknown, Scope.Scope>
  modelToMaybeRequirements: (model: Model) => any
  onAcquired: (value: any) => Message
  onAcquireError: (error: unknown) => Message
  onReleased: () => Message
  release: (value: any) => Effect.Effect<void>
  resource: ManagedResource<any>
  schema: Schema.Schema<any>
}
```

### ManagedResources

type

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L95)

```
/** A record of named Managed Resource configurations, keyed by resource name. */
type ManagedResources = Record<string, ManagedResourceConfig<Model, Message>> & {
  __managedResourceServices: Services
}
```

### ServiceOf

type

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L80)

```
/** Type-level utility to extract the service identity type from a ManagedResource. */
type ServiceOf = T extends ManagedResource<any, infer S>
  ? S
  : never
```

### ServicesOf

type

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L154)

```
/** Type-level utility to extract the service union from a Managed Resources record. */
type ServicesOf = {
  [Key in keyof Resources]: Resources[Key] extends {
    resource: ManagedResource<any, infer Service>
  }
    ? Service
    : never
}[keyof Resources]
```

### Value

type

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L77)

```
/** Type-level utility to extract the value type from a ManagedResource. */
type Value = T extends ManagedResource<infer V, any>
  ? V
  : never
```

## Interfaces

### ManagedResource

interface

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L39)

```
/**
 * A model-driven resource with acquire/release lifecycle. Access the resource
 * value in commands via `.get`, which fails with `ResourceNotAvailable` when
 * the resource is not currently active. The service identity appears in the
 * Effect R channel, providing compile-time enforcement that the resource is
 * registered.
 */
interface ManagedResource {
  [ManagedResourceTypeId]: typeof ManagedResourceTypeId
  get: Effect<Value, ResourceNotAvailable, Service>
  key: string
}
```

### ManagedResourceService

interface

[source](https://github.com/foldkit/foldkit/blob/98d070e4410098f14d851fd4d0ddb7448fc63b95/packages/foldkit/src/managedResource/managedResource.ts#L28)

```
/** Branded identity type for a managed resource, used in the Effect R channel. */
interface ManagedResourceService {
  [ManagedResourceBrand]: Key
}
```
