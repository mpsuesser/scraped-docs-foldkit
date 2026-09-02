---
url: https://foldkit.dev/api-reference/experimental-machine
title: "Experimental/Machine"
description: "API documentation for the Experimental/Machine module."
access_date: 2026-09-02T07:05:07.578Z
current_date: 2026-09-02T07:05:07.578Z
---

# Experimental/Machine

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L556)

```
<State extends Readonly<{
  _tag: string
}>, Message extends Readonly<{
  _tag: string
}>>(schemas: MachineSchemas<State, Message>): (definition: MachineDefinition<State, Message, R>) => Machine<State, Message, R>
```

### otherwise

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L285)

```
<State extends Readonly<{
  _tag: string
}>, Message extends Readonly<{
  _tag: string
}>, SourceState extends Readonly<{
  _tag: string
}>, TriggerMessage extends Readonly<{
  _tag: string
}>, R = never>(edge: Edge<State, Message, SourceState, TriggerMessage, void, R>): Otherwise<State, Message, SourceState, TriggerMessage, R>
```

### to

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L202)

```
<State extends Readonly<{
  _tag: string
}>, Message extends Readonly<{
  _tag: string
}>, SourceState extends Readonly<{
  _tag: string
}>, TriggerMessage extends Readonly<{
  _tag: string
}>, TargetTag extends string, R = never>(
  target: TargetTag,
  build: (input: NoInfer<Readonly<{
    guardValue: void
    message: TriggerMessage
    state: SourceState
  }>>) => NoInfer<Extract<State, Readonly<{
    _tag: TargetTag
  }>>>,
  commands?: (input: NoInfer<Readonly<{
    guardValue: void
    message: TriggerMessage
    state: SourceState
  }>>) => readonly Array<Command<NoInfer<Message>, never, R>>
): Edge<State, Message, SourceState, TriggerMessage, void, R>
```

### when

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L233)

## Types

### DeadTransition

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L360)

```
/** An Edge that cannot fire in a walk of the declared Edge set, with the reason. */
type DeadTransition = Readonly<{
  edge: EdgeSummary<State, Message>
  reason: DeadTransitionReason
}>
```

### DeadTransitionReason

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L357)

```
/**
 * Why an Edge cannot fire in a walk of the declared Edge set:
 * `UnreachableSource` means no path from the walk roots reaches the Edge's
 * source state, and `ShadowedByOtherwise` means an earlier `otherwise` in the
 * Edge's guard list always fires first.
 */
type DeadTransitionReason = "UnreachableSource" | "ShadowedByOtherwise"
```

### Edge

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L51)

```
/**
 * A single transition edge. The target state tag is a literal value, so the
 * edge set of a Machine is enumerable data. `build` constructs the target
 * variant from its EdgeInput. `maybeCommands` holds transition-time
 * effects, dispatched as ordinary Commands.
 * 
 * The correlation between `target` and `build`'s return variant is enforced
 * by to's signature, not by this type. Keeping this type free of the
 * target tag is what lets TypeScript infer the source state and trigger
 * Message from the transition table position.
 * 
 * Construct with to.
 */
type Edge = Readonly<{
  _tag: "Edge"
  ~foldkit/EdgeGuardValue: GuardValue
  build: (input: EdgeInput<SourceState, TriggerMessage, unknown>) => State
  maybeCommands: Option.Option<(input: EdgeInput<SourceState, TriggerMessage, unknown>) => ReadonlyArray<Command<Message, never, R>>>
  target: TagOf<State>
}>
```

### EdgeGuard

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L335)

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

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L28)

```
/**
 * The single argument an Edge's `build` and `commands` callbacks receive:
 * the source state, the triggering Message, and the guard value produced by
 * the Edge's when guard (`void` on unguarded and boolean-guarded
 * Edges). Destructure the fields you need.
 */
type EdgeInput = Readonly<{
  guardValue: GuardValue
  message: TriggerMessage
  state: SourceState
}>
```

### EdgeSummary

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L341)

```
/** One Edge of the table as plain data: source, trigger, target, and guard placement. */
type EdgeSummary = Readonly<{
  from: TagOf<State>
  guard: EdgeGuard
  messageTag: TagOf<Message>
  target: TagOf<State>
}>
```

### GuardedEdge

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L97)

```
/** One entry in an ordered guard list: a When or the Otherwise fallback. */
type GuardedEdge = When<State, Message, SourceState, TriggerMessage, unknown, R> | Otherwise<State, Message, SourceState, TriggerMessage, R>
```

### Ignored

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L315)

```
/** A step that matched no Edge: the state is unchanged and the Message is observable as ignored. */
type Ignored = Readonly<{
  _tag: "Ignored"
  messageTag: TagOf<Message>
  state: State
  stateTag: TagOf<State>
}>
```

### Machine

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L374)

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
  step: (state: State, message: Message) => TransitionResult<State, Message, R>
  toMermaid: () => string
  transition: (state: State, message: Message) => Update.Return<State, Message, R>
  unreachableStates: (extraRoots?: ReadonlyArray<TagOf<State>>) => ReadonlyArray<TagOf<State>>
}>
```

### MachineDefinition

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L440)

```
/**
 * The Machine definition: the initial state and the transition table.
 * 
 *  Ships from `foldkit/experimental/machine`; expect breaking changes while the API settles.
 */
type MachineDefinition = Readonly<{
  initial: State
  states: TransitionTable<State, Message, R>
}>
```

### MachineSchemas

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L427)

```
/**
 * The Schemas a Machine is defined over: the state union and the Message
 * union. Passed to `define`'s first stage so the type parameters are
 * fully resolved before the transition table is checked.
 */
type MachineSchemas = Readonly<{
  message: Schema.Top & Readonly<{
    Type: Message
  }>
  state: Schema.Top & Readonly<{
    members: ReadonlyArray<Schema.Top>
    Type: State
  }>
}>
```

### Otherwise

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L85)

```
/** The unconditional fallback Edge at the end of a guard list. Construct with otherwise. */
type Otherwise = Readonly<{
  _tag: "Otherwise"
  edge: Edge<State, Message, SourceState, TriggerMessage, void, R>
}>
```

### TagOf

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L12)

```
/** The union of `_tag` literals in a Tagged union. */
type TagOf = Union["_tag"]
```

### Tagged

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L9)

```
/** Any value discriminated by a `_tag` field. Both states and Messages satisfy this shape. */
type Tagged = Readonly<{
  _tag: string
}>
```

### TransitionResult

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L326)

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

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L123)

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
      [MessageTag in TagOf<Message>]: Edge<State, Message, Variant<State, SourceTag>, Variant<Message, MessageTag>, void, R> | ReadonlyArray<GuardedEdge<State, Message, Variant<State, SourceTag>, Variant<Message, MessageTag>, R>>
    }>
  }>
}>
```

### Transitioned

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L301)

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

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L15)

```
/** The single variant of a Tagged union carrying the given tag. */
type Variant = Extract<Union, Readonly<{
  _tag: Tag
}>>
```

### When

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/experimental/machine/machine.ts#L71)

```
/** A guarded Edge that fires only when its guard passes. Construct with when. */
type When = Readonly<{
  _tag: "When"
  edge: Edge<State, Message, SourceState, TriggerMessage, GuardValue, R>
  guard: (state: SourceState, message: TriggerMessage) => Option.Option<unknown>
}>
```
