---
url: https://foldkit.dev/api-reference/experimental-machine
title: "Experimental/Machine"
description: "API documentation for the Experimental/Machine module."
access_date: 2026-09-07T07:29:31.695Z
current_date: 2026-09-07T07:29:31.695Z
---

# Experimental/Machine

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L1183)

```
/**
 * Compiles a declarative Machine definition into a Machine.
 * 
 * Two stages: the first takes the state and Message union Schemas and fixes
 * the type parameters, the second takes the initial state, optional shared
 * transition defaults, and the state-local transition table. The split is
 * what lets TypeScript narrow `state` and `message` inside every Edge from its
 * transition-map position: a single-call form checks the definition while the
 * type parameters are still being inferred, and the narrowing collapses.
 * 
 * Build shared defaults with forStates. Each fragment is expanded
 * into the ordinary state-local table before dispatch and static analysis. A
 * state-local transition replaces its shared default for that state and
 * Message; overlapping shared fragments throw when the Machine is defined.
 * 
 * The Machine is not a runtime: `transition` returns an `Update.Return`, so the
 * Machine state lives in the Model and the Foldkit runtime never learns the
 * Machine exists. Use fold to read and write a Machine state field in
 * an enclosing Model. Messages that match no Edge leave the state unchanged;
 * use `step` when the `Ignored` outcome should be observable.
 * 
 * Declare a `context` Schema when transitions need a read-only view of data
 * outside the Machine state. The context is passed to guards and Edge handlers
 * on each call; it is not decoded, stored, or included in static analysis. Data
 * that the state owns for its lifetime belongs in the state as a snapshot.
 * Values that should be visible as facts in Story tests and DevTools should
 * still enter through Messages.
 * 
 * Because every Edge names a literal target tag, the Edge set is plain data:
 * `reachableFrom`, `unreachableStates`, `deadTransitions`, and `toMermaid`
 * all read it directly.
 * 
 * The Machine's requirements `R` are the services its edge Commands need, and
 * flow into `transition` and `step`. `R` defaults to `never`. When every edge
 * Command shares one service, `R` is inferred from the table. When edges need
 * distinct services `R` cannot be inferred to their union, so supply it on the
 * second call: `define(schemas)<UploadsClient | SaveClient>({ ... })`.
 */
<State extends Readonly<{
  _tag: string
}>, Message extends Readonly<{
  _tag: string
}>, ContextSchema extends Top>(schemas: MachineSchemas<State, Message, ContextSchema>): (definition: MachineDefinition<State, Message, R, ContextSchema["Type"]>) => Machine<State, Message, R, ContextSchema["Type"]>

<State extends Readonly<{
  _tag: string
}>, Message extends Readonly<{
  _tag: string
}>>(schemas: MachineSchemas<State, Message>): (definition: MachineDefinition<State, Message, R>) => Machine<State, Message, R>
```

### ignore

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L557)

```
(): Ignore
```

### otherwise

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L535)

```
<State extends Readonly<{
  _tag: string
}>, Message extends Readonly<{
  _tag: string
}>, SourceState extends Readonly<{
  _tag: string
}>, TriggerMessage extends Readonly<{
  _tag: string
}>, R = never, Context = typeof NoMachineContextTypeId>(edge: Edge<State, Message, SourceState, TriggerMessage, void, R, Context>): Otherwise<State, Message, SourceState, TriggerMessage, R, Context>
```

### to

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L428)

```
<State extends Readonly<{
  _tag: string
}>, Message extends Readonly<{
  _tag: string
}>, SourceState extends Readonly<{
  _tag: string
}>, TriggerMessage extends Readonly<{
  _tag: string
}>, TargetTag extends string, R = never, Context = typeof NoMachineContextTypeId>(
  target: TargetTag,
  handler: (input: NoInfer<EdgeInput<SourceState, TriggerMessage, void, Context>>) => Update.Return<NoInfer<Variant<State, TargetTag>>, NoInfer<Message>, R>
): Edge<State, Message, SourceState, TriggerMessage, void, R, Context>
```

### when

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L468)

```
<State extends Readonly<{
  _tag: string
}>, Message extends Readonly<{
  _tag: string
}>, SourceState extends Readonly<{
  _tag: string
}>, TriggerMessage extends Readonly<{
  _tag: string
}>, GuardResult extends boolean | Option<unknown>, TargetTag extends string, R = never, Context = typeof NoMachineContextTypeId>(
  guard: (state: NoInfer<SourceState>, message: NoInfer<TriggerMessage>, context: ContextArguments<NoInfer<Context>>) => GuardResult,
  target: TargetTag,
  handler: (input: NoInfer<EdgeInput<SourceState, TriggerMessage, GuardValueOf<GuardResult>, Context>>) => Update.Return<NoInfer<Variant<State, TargetTag>>, NoInfer<Message>, R>
): When<State, Message, SourceState, TriggerMessage, GuardValueOf<GuardResult>, R, Context>
```

## Types

### DeadTransition

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L647)

```
/** An Edge that cannot fire in a walk of the declared Edge set, with the reason. */
type DeadTransition = Readonly<{
  edge: EdgeSummary<State, Message>
  reason: DeadTransitionReason
}>
```

### DeadTransitionReason

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L641)

```
/**
 * Why an Edge cannot fire in a walk of the declared Edge set:
 * `UnreachableSource` means no path from the walk roots reaches the Edge's
 * source state, and `ShadowedByOtherwise` means an earlier `otherwise` in the
 * Edge's guard list always fires first. `ShadowedByIgnore` means an earlier
 * ignore stops evaluation first.
 */
type DeadTransitionReason = "UnreachableSource" | "ShadowedByOtherwise" | "ShadowedByIgnore"
```

### Edge

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L103)

```
/**
 * A single transition edge. The target state tag is a literal value, so the
 * edge set of a Machine is enumerable data. `handler` returns the target
 * variant and any transition-time Commands as an `Update.Return` record.
 * 
 * The correlation between `target` and `handler`'s Model variant is enforced
 * by to's signature, not by this type. Keeping this type free of the
 * target tag is what lets TypeScript infer the source state and trigger
 * Message from the transition-map position.
 * 
 * Construct with to.
 */
type Edge = Readonly<{
  _tag: "Edge"
  ~foldkit/EdgeGuardValue: GuardValue
  handler: (input: EdgeInput<SourceState, TriggerMessage, unknown, Context>) => Update.Return<State, Message, R>
  target: TagOf<State>
}>
```

### EdgeGuard

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L618)

```
/** Which guard construct an Edge sits under, with its position in the guard list. */
type EdgeGuard = Readonly<{
  _tag: "Unguarded"
}> | Readonly<{
  _tag: "When"
  position: number
}> | Readonly<{
  _tag: "Otherwise"
  position: number
}>
```

### EdgeInput

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L72)

```
/**
 * The single argument an Edge handler receives: the source state, the
 * triggering Message, the guard value produced by the Edge's when
 * guard (`void` on unguarded and boolean-guarded Edges), and the read-only
 * context declared for the Machine when one exists. Destructure the fields you
 * need.
 */
type EdgeInput = HasMachineContext<Context> extends true
  ? Readonly<{
    context: Context
    guardValue: GuardValue
    message: TriggerMessage
    state: SourceState
  }>
  : Readonly<{
    guardValue: GuardValue
    message: TriggerMessage
    state: SourceState
  }>
```

### EdgeSummary

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L624)

```
/** One Edge of the table as plain data: source, trigger, target, and guard placement. */
type EdgeSummary = Readonly<{
  from: TagOf<State>
  guard: EdgeGuard
  messageTag: TagOf<Message>
  target: TagOf<State>
}>
```

### FoldConfig

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L738)

```
/**
 * The capabilities needed to fold a Machine state field into its enclosing
 * Model. `read` returns an `Option` because the field may be absent in the
 * current Model variant; `write` replaces it after a transition. A contextual
 * Machine also requires `context`, which reads the current context from the
 * enclosing Model for each transition.
 */
type FoldConfig = Readonly<{
  machine: Machine<State, Message, R, Context>
  read: (model: ParentModel) => Option.Option<State>
  write: (model: ParentModel, nextState: State) => ParentModel
}> & FoldContextField<ParentModel, Context>
```

### GuardedEdge

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L165)

```
/**
 * One entry in an ordered guard list: a When, the Otherwise
 * transition fallback, or the Ignore no-transition fallback.
 */
type GuardedEdge = When<State, Message, SourceState, TriggerMessage, unknown, R, Context> | Otherwise<State, Message, SourceState, TriggerMessage, R, Context> | Ignore
```

### Ignore

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L161)

```
/** An explicit no-transition fallback at the end of a guard list. Construct with ignore. */
type Ignore = Readonly<{
  _tag: "Ignore"
}>
```

### Ignored

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L597)

```
/**
 * A step that matched no Edge: the state is unchanged and the Message is
 * observable as ignored. `reason` distinguishes the four causes, described
 * on IgnoredReason.
 */
type Ignored = Readonly<{
  _tag: "Ignored"
  messageTag: TagOf<Message>
  reason: IgnoredReason
  state: State
  stateTag: TagOf<State>
}>
```

### IgnoredReason

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L586)

```
/**
 * Why a step matched no Edge. `OutOfAlphabet` means the Message tag appears
 * in no state's `on` record anywhere in the table, so the Message is outside
 * the Machine's alphabet. `NotApplicable` means the Message tag is in the
 * alphabet, but no Edge for it exists from the current state, whether the
 * state is absent from the table or its `on` record lacks the tag.
 * `GuardsFellThrough` means an Edge entry exists for this state and Message,
 * but every guard declined and no otherwise or ignore fallback
 * was present.
 * `ExplicitlyIgnored` means evaluation reached an ignore fallback.
 */
type IgnoredReason = "OutOfAlphabet" | "NotApplicable" | "GuardsFellThrough" | "ExplicitlyIgnored"
```

### Machine

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L661)

```
/**
 * A compiled state Machine: a pure transition function plus static analysis over the Edge set.
 * 
 *  Ships from `foldkit/experimental/machine`; expect breaking changes while the API settles.
 */
type Machine = Readonly<{
  deadTransitions: (extraRoots?: ReadonlyArray<TagOf<State>>) => ReadonlyArray<DeadTransition<State, Message>>
  edges: ReadonlyArray<EdgeSummary<State, Message>>
  initial: State
  reachableFrom: (tag: TagOf<State>) => ReadonlySet<TagOf<State>>
  stateTags: ReadonlyArray<TagOf<State>>
  step: (state: State, message: Message, context: MachineContextArguments<Context>) => TransitionResult<State, Message, R>
  toMermaid: () => string
  transition: (state: State, message: Message, context: MachineContextArguments<Context>) => Update.Return<State, Message, R>
  unreachableStates: (extraRoots?: ReadonlyArray<TagOf<State>>) => ReadonlyArray<TagOf<State>>
}>
```

### MachineDefinition

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L865)

```
/**
 * The Machine definition: the initial state, shared transition defaults, and
 * the state-local transition table. A state-local transition replaces a
 * shared transition for the same state and Message.
 * 
 *  Ships from `foldkit/experimental/machine`; expect breaking changes while the API settles.
 */
type MachineDefinition = Readonly<{
  initial: State
  shared: ReadonlyArray<ForStatesFragment<State, Message, R, Context>>
  states: TransitionTable<State, Message, R, Context>
}>
```

### MachineSchemas

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L850)

```
/**
 * The Schemas a Machine is defined over, including an optional read-only
 * context Schema. A declared context is required by the Machine's guards,
 * Edge handlers, `transition`, and `step`.
 */
type MachineSchemas = MachineSchemaFields<State, Message> & [ContextSchema] extends [Schema.Top]
  ? Readonly<{
    context: ContextSchema
  }>
  : Readonly<{
    context: never
  }>
```

### Otherwise

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L148)

```
/** The unconditional fallback Edge at the end of a guard list. Construct with otherwise. */
type Otherwise = Readonly<{
  _tag: "Otherwise"
  edge: Edge<State, Message, SourceState, TriggerMessage, void, R, Context>
}>
```

### StateTransitions

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L364)

```
/**
 * The transition table entry for one source state. Use this alias when
 * extracting an entry from a Machine's `states` record to preserve the source
 * state and triggering Message narrowing inside its Edges.
 * 
 *  Ships from `foldkit/experimental/machine`; expect breaking changes while the API settles.
 */
type StateTransitions = NonNullable<TransitionTable<State, Message, R, Context>[SourceTag]>
```

### TagOf

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L23)

```
/** The union of `_tag` literals in a Tagged union. */
type TagOf = Union["_tag"]
```

### Tagged

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L20)

```
/** Any value discriminated by a `_tag` field. Both states and Messages satisfy this shape. */
type Tagged = Readonly<{
  _tag: string
}>
```

### TransitionResult

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L609)

```
/**
 * The observable outcome of one step: `Transitioned` or `Ignored`.
 * 
 *  Ships from `foldkit/experimental/machine`; expect breaking changes while the API settles.
 */
type TransitionResult = Transitioned<State, Message, R> | Ignored<State, Message>
```

### TransitionTable

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L193)

```
/**
 * The transition table: for each source state tag, the Messages it responds
 * to and the Edge (or ordered guard list) each Message fires. States absent
 * from the table, and Messages absent from a state's `on` record, are
 * ignored: Machine.step reports them as `Ignored` rather than
 * transitioning.
 * 
 *  Ships from `foldkit/experimental/machine`; expect breaking changes while the API settles.
 */
type TransitionTable = Readonly<{
  [SourceTag in TagOf<State>]: Readonly<{
    on: Readonly<{
      [MessageTag in TagOf<Message>]: Edge<State, Message, Variant<State, SourceTag>, Variant<Message, MessageTag>, void, R, Context> | ReadonlyArray<GuardedEdge<State, Message, Variant<State, SourceTag>, Variant<Message, MessageTag>, R, Context>>
    }>
  }>
}>
```

### Transitioned

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L562)

```
/** A step that matched an Edge: the next state plus any transition-time Commands. */
type Transitioned = Readonly<{
  _tag: "Transitioned"
  commands: ReadonlyArray<Command<Message, never, R>>
  from: TagOf<State>
  messageTag: TagOf<Message>
  state: State
  target: TagOf<State>
}>
```

### Variant

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L26)

```
/** The single variant of a Tagged union carrying the given tag. */
type Variant = Extract<Union, Readonly<{
  _tag: Tag
}>>
```

### When

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L121)

```
/** A guarded Edge that fires only when its guard passes. Construct with when. */
type When = Readonly<{
  _tag: "When"
  edge: Edge<State, Message, SourceState, TriggerMessage, GuardValue, R, Context>
  guard: (state: SourceState, message: TriggerMessage, context: ContextArguments<Context>) => Option.Option<unknown>
}>
```

## Constants

### fold

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L798)

```
/**
 * Folds a Machine state field into an enclosing Model. Any transition-time
 * Commands pass through unchanged.
 * 
 * When `read` returns `None`, the fold returns the original Model without
 * running a transition. For a contextual Machine, `context` is read only when
 * the Machine state is present and is supplied to that transition.
 * 
 * ```ts
 * const foldUpload = Machine.fold({
 *   machine: uploadMachine,
 *   read: (model: Model) => Option.some(model.upload),
 *   write: (model, nextUpload) =>
 *     evo(model, { upload: () => nextUpload }),
 *   context: model => model.uploadQueues,
 * })
 * 
 * // Data-first in update
 * foldUpload(model, message)
 * 
 * // Data-last in a composed update
 * Update.combine(model, [foldUpload(message), recordUploadAttempt])
 * ```
 * 
 *  Ships from `foldkit/experimental/machine`; expect breaking changes while the API settles.
 */
const fold: (config: FoldConfig<ParentModel, State, Message, R, Context>) => Fold<ParentModel, Message, Message, R>
```

### forStates

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/experimental/machine/machine.ts#L345)

```
/**
 * Selects source state tags for a shared transition map. The `on` handler's
 * `state` is narrowed to the union of the selected variants, and its `message`
 * is narrowed by the map key.
 * 
 * Add the resulting fragment to a Machine definition's `shared` array. Shared
 * transitions are defaults: a state-local transition for the same Message
 * replaces the shared transition. Defining the same state/Message pair in two
 * shared fragments throws when the Machine is defined.
 */
const forStates: (sourceTags: SourceTags) => ForStatesBuilder<SourceTags>
```
