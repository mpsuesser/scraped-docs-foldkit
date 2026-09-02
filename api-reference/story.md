---
url: https://foldkit.dev/api-reference/story
title: "Story"
description: "API documentation for the Story module."
access_date: 2026-09-02T07:05:07.578Z
current_date: 2026-09-02T07:05:07.578Z
---

# Story

## Functions

### expectNoOutMessage

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L373)

```
/** Asserts that update emitted no OutMessage. */
(): (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
```

### expectOutMessage

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L365)

```
/**
 * Asserts by structural equality that update emitted the expected OutMessage,
 *  so callers can pass a freshly constructed expected value.
 */
<OutMessage>(expected: OutMessage): OutMessageStep<OutMessage>
```

### given

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L120)

```
/** Sets the initial Model for a test story. */
<Model>(model: Model): GivenStep<Model>
```

### message

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L161)

```
/**
 * Sends a Message through update. Commands stay pending until resolve or
 *  resolveAll.
 */
<Message>(message_: Message): MessageStep<Message>
```

### model

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L248)

```
/** Runs an assertion function against the current Model. */
<Model>(f: (model: Model) => void): ModelStep<Model>
```

### steps

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L255)

## Types

### CommandMatcher

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/internal.ts#L46)

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

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L39)

```
/** A callable step that sets the initial Model. Carries phantom type for compile-time validation. */
type GivenStep = Readonly<{
  _phantomModel: Model
}> & (simulation: StorySimulation<M, Message, OutMessage>) => StorySimulation<M, Message, OutMessage>
```

### MessageStep

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L51)

```
/** A typed Message-dispatch step produced by message. */
type MessageStep = Readonly<{
  _tag: "MessageStep"
  message: Message
}>
```

### ModelStep

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L45)

```
/** A model-assertion step produced by model. */
type ModelStep = Readonly<{
  _tag: "ModelStep"
  assert: (model: Model) => void
}>
```

### OutMessageStep

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L57)

```
/** A typed OutMessage assertion step produced by expectOutMessage. */
type OutMessageStep = Readonly<{
  _tag: "OutMessageStep"
  expected: OutMessage
}>
```

### StorySimulation

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L30)

```
/** An immutable test simulation of a Foldkit program. */
type StorySimulation = Readonly<{
  commands: ReadonlyArray<AnyCommand>
  model: Model
  outMessage: OutMessage | undefined
}>
```

### StoryStep

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L80)

```
/**
 * A single step in a story: a GivenStep, ModelStep,
 *  MessageStep, OutMessageStep, StoryStepsStep, or
 *  simulation transform.
 */
type StoryStep = GivenStep<NoInfer<Model>> | ModelStep<NoInfer<Model>> | MessageStep<NoInfer<Message>> | OutMessageStep<NoInfer<OutMessage>> | StoryStepsStep<NoInfer<Model>, (model: NoInfer<Model>) => void, NoInfer<Message>, NoInfer<OutMessage>> | (simulation: StorySimulation<any, any, any>) => StorySimulation<any, any, any>
```

### StoryStepsStep

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L63)

```
/** A grouped sequence of Story steps produced by steps. */
type StoryStepsStep = Readonly<{
  _tag: "StoryStepsStep"
  steps: ReadonlyArray<StoryStep<any, any, any>>
}>
```

## Constants

### Command

const

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L328)

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

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/foldkit/src/test/story.ts#L422)

```
/** Executes a test story. Throws if any Commands remain unresolved. */
const story: (updateFn: (model: Model, message: Message) => Readonly<{
  commands: ReadonlyArray<AnyCommand>
  model: Model
  outMessage: OutMessage
}>, steps: readonly Array<StoryStep<NoInfer<Model>, NoInfer<Message>, NoInfer<OutMessage>>>) => void
```
