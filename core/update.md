---
url: https://foldkit.dev/core/update
title: "Update"
description: "Pure functions that transform the Model and return Commands in response to Messages. Foldkit's update replaces useReducer and useEffect with a single, exhaustive pattern match."
access_date: 2026-08-19T19:38:38.072Z
current_date: 2026-08-19T19:38:38.072Z
---

# Update

## One Function Defines Every Transition

The update function receives the current Model and a Message, then returns the next Model and any Commands for the runtime to execute. It is the only place application state changes.

Update is pure. Given the same Model and Message, it returns the same result. It does not mutate state, call browser APIs, start timers, or make requests. That makes a transition direct to test: pass in the inputs and assert on the returned values.

Use [Effect's `Match`](https://effect.website/docs/code-style/pattern-matching/) and `M.tagsExhaustive` to handle the Message union. If you add a Message and omit its branch, TypeScript reports the missing case. No `default` branch silently absorbs a new variant.

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

Each branch describes one transition. `ClickedDecrement` and `ClickedIncrement` transform the current count. `ClickedReset` replaces it with zero. All three return an empty Commands array because this version of the counter has no side effects.

The branches build their next Model with [evo](https://foldkit.dev/best-practices/immutability#immutable-updates). Each named field receives a function from its current value to its next value. Omitted fields keep their existing values and references, so the same update style continues to work as the Model grows.

Update returns a tuple containing the next Model and an array of Commands. A Command describes one side effect, such as an HTTP request, timer, or browser API call. The [Commands](https://foldkit.dev/core/commands) page adds a delayed reset and puts that second tuple element to work.

First, the [view function](https://foldkit.dev/core/view) completes the basic loop by turning the Model into what the user sees.
