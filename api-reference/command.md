---
url: https://foldkit.dev/api-reference/command
title: "Command"
description: "API documentation for the Command module."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

# Command

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L220)

## Types

### Command

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L12)

```
/**
 * A named Effect that produces a message, optionally carrying the args used
 *  to construct it. `key` is present on Commands built by
 *  `define` with `interrupt`: it addresses the invocation in the runtime's
 *  interrupt registry so a later Interrupt Command can stop it.
 */
type Command = [T] extends [Schema.Top]
  ? Readonly<{
    args: Record<string, unknown>
    effect: Effect.Effect<Schema.Schema.Type<T>, E, R>
    key: string
    name: string
  }>
  : Readonly<{
    args: Record<string, unknown>
    effect: Effect.Effect<T, E, R>
    key: string
    name: string
  }>
```

### CommandDefinition

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L76)

```
/** A Command definition created with `Command.define`. Union over the no-args and with-args shapes; consumers that only need name/identity can accept this. */
type CommandDefinition = CommandDefinitionNoArgs<Name, Effect.Effect<ResultMessage, any, any>> | CommandDefinitionWithArgs<Name, any, Effect.Effect<ResultMessage, any, any>>
```

### InterruptOption

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L92)

```
/**
 * Makes a Command interruptible. `true` keys every invocation by the Command
 *  name, which is what you want when at most one invocation is meaningfully in
 *  flight. `{ keyFields, toKey }` derives the key part from declared args, so
 *  invocations that run concurrently can be interrupted independently.
 *  `keyFields` declares the exact args the Interrupt constructor requires.
 *  Derive the key part from the Model identity that owns the in-flight work (a
 *  list item id, an entity id), never from a generated value: update is pure,
 *  and any invocation you can meaningfully target is already distinguished by
 *  data in the Model.
 */
type InterruptOption = true | Readonly<{
  keyFields: Array.NonEmptyReadonlyArray<KeyField>
  toKey: (keyArgs: Pick<Args, KeyField>) => string
}>
```

## Interfaces

### CommandDefinitionNoArgs

interface

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L51)

```
/** A Command definition for a Command with no declared args. Call as `Definition()` to produce a Command instance. */
interface CommandDefinitionNoArgs {
  [CommandDefinitionTypeId]: typeof CommandDefinitionTypeId
  name: Name
}
```

### CommandDefinitionWithArgs

interface

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L61)

```
/** A Command definition for a Command with declared args. Call as `Definition(args)` to produce a Command instance. */
interface CommandDefinitionWithArgs {
  [CommandDefinitionTypeId]: typeof CommandDefinitionTypeId
  name: Name
}
```

## Constants

### CommandDefinitionTypeId

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/brand.ts#L3)

```
/** Type-level brand for CommandDefinition values. */
const CommandDefinitionTypeId: unique symbol
```

### mapEffect

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L402)

```
/**
 * Transforms the Effect inside a Command while preserving its result Message,
 *  name, args, and message-mapping chain. Reach for this to provide a service,
 *  add a delay or retry, or change the error or requirement channel. Its type
 *  prevents the transform from changing the result Message. Never use it to
 *  lift that Message, even via
 *  `Effect.map(message => Message.GotChildMessage({ message }))`. That dispatches
 *  correctly in production but is invisible to `Story`/`Scene` `resolve`, which
 *  replays only the recorded chain and never runs the Effect, so the test would
 *  see the child's raw Message instead of the wrapped one. Lift result Messages
 *  with mapMessage / mapMessages, which record the lift. The
 *  `foldkit/prefer-command-mapmessage` lint rule flags this misuse.
 */
const mapEffect: (f: (effect: Effect<A, E1, R1>) => Effect<NoInfer<A>, E2, R2>) => (command: Command<A, E1, R1>) => Command<A, E2, R2>
```

### mapMessage

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L447)

```
/**
 * Lifts a single Command's result Message through `f`, transforming
 *  `FromMessage` to `ToMessage`. The singular complement to
 *  mapMessages: reach for this when a child returns one Command
 *  (e.g. an animation leave Command), reach for `mapMessages` when it
 *  returns a list.
 * 
 *  Fuses `f` into the Effect so production dispatch is unchanged, and also
 *  records `f` on the Command's internal message-mapping chain. The chain is
 *  recoverable metadata the runtime never reads: `Story.Command.resolve` and
 *  `Scene.Command.resolve` replay the matched Command's chain over a substitute
 *  result, so a root test resolves with the child's raw result Message and
 *  never restates the wrapping by hand.
 * 
 *  Preserves the Command's `name` and `args` so traces still attribute
 *  it to the originating Submodel. When you need to transform the
 *  Effect itself (not just the result Message), reach for
 *  mapEffect instead.
 * 
 *  Typed against Command in argument and result positions, so a
 *  generic combinator over a type-parameter Message unifies with
 *  `Command.Command<Message>` directly.
 */
const mapMessage: (command: Command<FromMessage, E, R>, f: (message: FromMessage) => ToMessage) => Command<ToMessage, E, R>
```

### mapMessages

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/command/index.ts#L512)

```
/**
 * Lifts every Command in a list through `f`, transforming the result
 *  Message type from `FromMessage` to `ToMessage`. When `commands` is
 *  `undefined`, it returns an empty array. `Update.foldChild`,
 *  `Update.foldChildStep`, `Update.foldChildInit`, and `Update.foldChildInits`
 *  handle this mapping for application Submodels. Use `mapMessages` for a
 *  standalone batch of Commands or when route-gated initialization includes
 *  only the active child's Commands:
 * 
 *  ```ts
 *  const commands = Command.mapMessages(
 *    [Feed.RefreshSummary(), Feed.RefreshTimeline()],
 *    toGotFeedMessage,
 *  )
 *  ```
 * 
 *  Fuses `f` into each Command's Effect and also records it on the Command's
 *  internal message-mapping chain, so production dispatch is unchanged while
 *  `Story.Command.resolve` / `Scene.Command.resolve` can recover the mapping
 *  from the matched Command.
 *  Preserves each Command's `name` and `args` so traces still attribute the
 *  Command to the originating Submodel. When you need to transform the Effect
 *  itself (not just the result Message), reach for mapEffect instead.
 * 
 *  Typed against Command in argument and result positions, so a
 *  generic combinator over a type-parameter Message unifies with
 *  `Command.Command<Message>` directly.
 */
const mapMessages: (commands: readonly Array<Command<FromMessage, E, R>> | undefined, f: (message: FromMessage) => ToMessage) => readonly Array<Command<ToMessage, E, R>>
```
