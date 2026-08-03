---
url: https://foldkit.dev/core/update
title: "Update"
description: "Pure functions that transform the Model and return Commands in response to Messages. Foldkit's update replaces useReducer and useEffect with a single, exhaustive pattern match."
access_date: 2026-08-03T19:45:20.723Z
current_date: 2026-08-03T19:45:20.723Z
---

# Update

## Overview

The update function is the heart of your application logic. It’s a pure function that takes the current Model and a Message, and returns a new Model along with any Commands to execute.

In the [restaurant analogy](https://foldkit.dev/core/architecture#the-restaurant-analogy), the update function is the waiter. Something happens (a customer flags them down, the kitchen rings the bell) and the waiter decides what to do next. Update the notebook, maybe write a slip for the kitchen. The waiter doesn’t cook the food or serve it directly. They take in what happened and decide on next steps.

Pure means predictable: given the same Model and the same Message, update always returns the same result. No hidden state, no ambient mutation, no surprises. This makes every state transition easy to reason about and trivial to test: pass in a Model and a Message, assert on the output.

Foldkit uses [Effect.Match](https://effect.website/docs/code-style/pattern-matching/) for exhaustive pattern matching on Messages. The TypeScript compiler will error if you forget to handle a Message type.

Add a new Message to your app and forget to handle it here? The compiler tells you. No forgotten cases, no `default` branches silently swallowing new Messages (unless you explicitly opt into a catch-all).

```
import { Match as M } from 'effect'
import { Command } from 'foldkit'
import { evo } from 'foldkit/struct'

// UPDATE

const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    M.withReturnType<
      readonly [Model, ReadonlyArray<Command.Command<Message>>]
    >(),
    M.tagsExhaustive({
      ClickedDecrement: () => [evo(model, { count: count => count - 1 }), []],
      ClickedIncrement: () => [evo(model, { count: count => count + 1 }), []],
      ClickedReset: () => [evo(model, { count: () => 0 }), []],
    }),
  )
```

Each branch builds the next Model with [evo](https://foldkit.dev/best-practices/immutability#immutable-updates), which evolves the Model you already have instead of rebuilding it. Every field you name takes a function from its current value to its next one, and fields you leave out keep their existing references. The counter has a single field today, so there’s nothing else to carry, but the shape stays the same as the Model grows.

Notice that update returns a tuple: the new Model and an array of Commands. Commands represent side effects: HTTP requests, timers, browser API calls. Each Command carries a name for tracing and testing. For the counter, the Commands array is always empty. But when we add a delayed reset on the [Commands](https://foldkit.dev/core/commands) page, that will change.

Before we get to side effects, there’s one more piece of the counter to understand: the view function, which turns your Model into what the user sees on screen.
