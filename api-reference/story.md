---
url: https://foldkit.dev/api-reference/story
title: "Story"
description: "API documentation for the Story module."
access_date: 2026-08-16T21:27:58.480Z
current_date: 2026-08-16T21:27:58.480Z
---

# Story

## Functions

### expectNoOutMessage

function

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L300)

```
/** Asserts that the OutMessage is None. */
(): (simulation: StorySimulation<Model, Message, Option.Option<OutMessage>>) => StorySimulation<Model, Message, Option.Option<OutMessage>>
```

### expectOutMessage

function

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L278)

```
/** Asserts that the OutMessage is Some with the expected value. */
<Expected>(expected: Expected): (simulation: StorySimulation<Model, Message, Option.Option<OutMessage>>) => StorySimulation<Model, Message, Option.Option<OutMessage>>
```

### given

function

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L85)

```
/** Sets the initial Model for a test story. */
<Model>(model: Model): GivenStep<Model>
```

### message

function

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L106)

```
/**
 * Sends a Message through update. Commands stay pending until resolve or
 *  resolveAll.
 */
<MessageInput>(message_: MessageInput): (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
```

### model

function

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L214)

```
/** Runs an assertion function against the current Model. */
<Model>(f: (model: Model) => void): ModelStep<Model>
```

## Types

### CommandMatcher

type

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/internal.ts#L46)

```
/**
 * Pattern for matching a Command in test assertions. A Definition matches
 *  by name only ("a Command with this identity was dispatched"); an Instance
 *  matches by name AND structural-equal args ("a Command with this identity
 *  AND these args was dispatched"). Choose the form per assertion based on
 *  whether the test cares about the args value.
 * 
 *  Two modes only: name-only or name + full args. Partial-args matching is
 *  intentionally unsupported. If a subset of args carries the meaning the
 *  test is verifying, the right assertion is usually against the Model that
 *  the Command's result fed through update, not a partial Command shape.
 */
type CommandMatcher = CommandDefinition<string, unknown> | AnyCommand
```

### GivenStep

type

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L38)

```
/** A callable step that sets the initial Model. Carries phantom type for compile-time validation. */
type GivenStep = Readonly<{
  _phantomModel: Model
}> & (simulation: StorySimulation<M, Message, OutMessage>) => StorySimulation<M, Message, OutMessage>
```

### ModelStep

type

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L44)

```
/** A model-assertion step produced by model. */
type ModelStep = Readonly<{
  _tag: "ModelStep"
  assert: (model: Model) => void
}>
```

### StorySimulation

type

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L29)

```
/** An immutable test simulation of a Foldkit program. */
type StorySimulation = Readonly<{
  commands: ReadonlyArray<AnyCommand>
  model: Model
  outMessage: OutMessage
}>
```

### StoryStep

type

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L51)

```
/**
 * A single step in a story: a GivenStep, a ModelStep,
 *  or a simulation transform.
 */
type StoryStep = GivenStep<NoInfer<Model>> | ModelStep<NoInfer<Model>> | (sim: StorySimulation<any, any, any>) => StorySimulation<any, any, any>
```

## Constants

### Command

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L256)

```
/**
 * Steps that operate on the pending Commands of a story simulation.
 *  Destructure as `const { Command } = Story` for concise call sites.
 */
const Command: {
  expectExact: (matchers: readonly Array<CommandMatcher>) => (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
  expectHas: (matchers: readonly Array<CommandMatcher>) => (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
  expectNone: () => (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
  resolve: (definition: ResolvableCommandDefinition<Name, ResultMessage>, resultMessage: ResultMessage) => (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
  resolveAll: (resolvers: {
    [K in string | number | symbol]: Resolver<Matchers[K]>
  }) => (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
  resolveAllExact: (resolvers: {
    [K in string | number | symbol]: Resolver<Matchers[K]>
  }) => (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
}
```

### story

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/foldkit/src/test/story.ts#L323)

```
/** Executes a test story. Throws if any Commands remain unresolved. */
const story: (updateFn: (model: Model, message: Message) => readonly [
  Model,
  readonly Array<Readonly<{
    args: Record<string, unknown>
    interruptsKey: string
    key: string
    messageMappers: ReadonlyArray<(message: unknown) => unknown>
    name: string
  }>>,
  OutMessage
], steps: readonly Array<StoryStep<NoInfer<Model>>>) => void
```
