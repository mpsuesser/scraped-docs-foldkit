---
url: https://foldkit.dev/best-practices/side-effects-and-purity
title: "Side Effects & Purity"
description: "Why Foldkit programs should have zero side effects outside of Commands."
access_date: 2026-08-03T18:13:14.939Z
current_date: 2026-08-03T18:13:14.939Z
---

# Side Effects & Purity

## Overview

A correct Foldkit program is a pure description with zero side effects, period. Yes, zero (0). Your program is the Foldkit application you define: your Model, update, view, the Command values returned by update, and more. Evaluating it does not perform side effects.

Every side effect is described as an Effect: a value that represents a computation without executing it. An Effect does nothing when you construct it. The side effects still happen, but only when the Foldkit runtime runs your program and executes the Effects it produces.

Both `view` and `update` are pure functions. They take inputs and return outputs without touching the outside world.

You encapsulate side effects in exactly six places:

- [Commands](https://foldkit.dev/core/commands): an Effect that performs a side effect and returns a Message. HTTP requests, DOM operations, reading from storage. This is where most of your side effects live.
- [Mount](https://foldkit.dev/core/mount): an Effect run with the live `Element` when a view element enters the DOM, paired with cleanup that fires when it unmounts. The seam where view code reaches a real DOM node, like portaling an overlay to the body or handing the element to a third-party library that owns its own DOM.
- [flags](https://foldkit.dev/core/init-and-flags#flags): an Effect that returns the initial data your program needs to start. Reading from local storage, detecting browser capabilities, or fetching configuration.
- [Subscription](https://foldkit.dev/core/subscriptions) streams: a `Stream<Message>`. Subscriptions model ongoing processes like keyboard events, window resizing, or intersection observers. When a stream callback needs to perform a side effect before producing a Message (like calling `event.preventDefault()`), use `Stream.mapEffect`. The runtime controls when streams subscribe and unsubscribe based on your Model.
- [Resources](https://foldkit.dev/core/resources): an Effect Layer that provides long-lived services to your Commands. One-time setup like assembling an RPC client or opening a database connection.
- [Managed Resources](https://foldkit.dev/core/managed-resources): `acquire` and `release` Effects for stateful resources that activate and deactivate based on your Model. Camera streams, WebSocket connections, media recorders.

That’s it. Every side effect in your program is an Effect value, managed by the runtime. Your logic is pure.

## Why Zero Side Effects?

Foldkit gains powerful guarantees from zero side effects:

- DevTools replay: the DevTools can replay any sequence of Messages against your `update` function because it’s pure. If `update` had side effects, replaying would double-fire them.
- Time-travel debugging: you can jump to any point in your app’s history and see exactly what the Model looked like, because each state is a deterministic function of the previous state plus the Message.
- Predictability: reading `update` tells you everything about how a Message changes the Model. There are no hidden effects, no action-at-a-distance, no callbacks firing behind the scenes.

## Common Mistakes

- `console.log` in `update`: `console.log` during development is fine for quick debugging. But production logging or error monitoring is a side effect that belongs in a Command. It will fire again during DevTools replay, and you want structured control over what gets reported.
- `Date.now()` in `update`: calling `Date.now()` breaks purity because the same Model and Message produce different results depending on when they run. Request the current time via a Command using Effect’s [DateTime](https://effect.website/docs/data-types/datetime/) module and return it as a Message.
- `fetch` in `view`: the view is called on every render. Instead, return a Command from `update` that fetches your data and returns a Message. Handle the Message to update your Model.
- DOM access anywhere: reading `document.getElementById` or `window.innerWidth` breaks purity. Use Subscriptions for reactive values, or Commands for one-off reads.

## Pure Functions Everywhere

### View is Pure

- No hooks, no lifecycle methods
- No fetching data, no timers, no subscriptions
- Given the same Model, always returns the same Html

```
import type { Document, HtmlBuilder } from 'foldkit/html'

import type { Message } from './message'
import { Model } from './model'

// ❌ Don't do this in view
const view = (model: Model, h: HtmlBuilder<Message>): Document => {
  // Fetching data in view
  fetch('/api/user').then(res => res.json())

  // Setting timers
  setTimeout(() => console.log('tick'), 1000)

  // Subscriptions
  window.addEventListener('resize', handleResize)

  return { title: 'Hello', body: h.div([], ['Hello']) }
}
```

```
import type { Document, HtmlBuilder } from 'foldkit/html'

import { ClickedIncrement, Message } from './message'
import { Model } from './model'

// ✅ View is a pure function from Model to a Document describing the page
const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: model.title,
  body: h.div(
    [h.Class('container')],
    [
      h.h1([], [model.title]),
      h.p([], [`Count: ${model.count}`]),
      h.button([h.OnClick(ClickedIncrement())], ['+']),
    ],
  ),
})
```

### Update is Pure

- Returns a new Model and a list of Commands. It doesn’t execute anything. Each Command carries a name for tracing and testing. Foldkit runs the provided Commands.
- No mutations, no side effects
- Given the same Model and Message, always returns the same result

```
import { Match } from 'effect'

import { Message } from './message'
import { Model } from './model'

// ❌ Don't do this in update
const update = (model: Model, message: Message) =>
  Match.value(message).pipe(
    Match.tagsExhaustive({
      ClickedFetchUser: () => {
        // Making HTTP requests directly
        fetch('/api/user').then(res => {
          model.user = res.json() // Mutating state!
        })
        return [model, []]
      },
    }),
  )
```

```
import { Match } from 'effect'
import { evo } from 'foldkit/struct'

import { fetchUser } from './command'
import { Message } from './message'
import { Model } from './model'

// ✅ Update returns new state and commands
const update = (model: Model, message: Message) =>
  Match.value(message).pipe(
    Match.tagsExhaustive({
      ClickedFetchUser: () => [
        evo(model, { isLoading: () => true }),
        [fetchUser(model.userId)], // Command handles the side effect
      ],

      SucceededFetchUser: ({ user }) => [
        evo(model, { isLoading: () => false, user: () => user }),
        [], // Result received, no more commands needed
      ],
    }),
  )
```

This purity has a practical payoff: testing is trivial. Foldkit ships `foldkit/test`: a simulation module that lets you send Messages, declare Command resolvers, and assert on the Model in a single pipe chain. See the [Testing](https://foldkit.dev/testing) guide for the full API.

## Requesting Values

A common mistake is computing random or time-based values directly in `update`. This breaks purity. Calling the function twice with the same inputs would return different results.

### Don’t Compute in Update

```
import { Match } from 'effect'

import { GRID_SIZE } from './constants'
import { Message, RequestedApple } from './message'
import { Model } from './model'

// ❌ Don't do this - calling random directly in update
const update = (model: Model, message: Message) =>
  Match.value(message).pipe(
    Match.tagsExhaustive({
      RequestedApple: () => {
        const x = Math.floor(Math.random() * GRID_SIZE)
        const y = Math.floor(Math.random() * GRID_SIZE)
        return [{ ...model, apple: { x, y } }, []]
      },
    }),
  )

// Same inputs produce different outputs - this breaks purity!
const model = { snake: [{ x: 0, y: 0 }], apple: { x: 5, y: 5 } }
const message = RequestedApple()

console.log(update(model, message)[0].apple) // { x: 12, y: 7 }
console.log(update(model, message)[0].apple) // { x: 3, y: 19 }
console.log(update(model, message)[0].apple) // { x: 8, y: 2 }
```

### Request Via Command

Instead, return a Command that generates the value and sends it back as a Message:

```
import { Effect, Match, Random } from 'effect'
import { Command } from 'foldkit'

import { GRID_SIZE } from './constants'
import {
  CompletedGenerateApplePosition,
  Message,
  RequestedApple,
} from './message'
import { Model } from './model'

const update = (model: Model, message: Message) =>
  Match.value(message).pipe(
    Match.tagsExhaustive({
      RequestedApple: () => [model, [GenerateApplePosition()]],
      CompletedGenerateApplePosition: ({ position }) => [
        { ...model, apple: position },
        [],
      ],
    }),
  )

const GenerateApplePosition = Command.define('GenerateApplePosition', {
  messages: [CompletedGenerateApplePosition],
  execute: Effect.gen(function* () {
    const x = yield* Random.nextIntBetween(0, GRID_SIZE, { halfOpen: true })
    const y = yield* Random.nextIntBetween(0, GRID_SIZE, { halfOpen: true })
    return CompletedGenerateApplePosition({ position: { x, y } })
  }),
})

const model = { snake: [{ x: 0, y: 0 }], apple: { x: 5, y: 5 } }
const message = RequestedApple()

console.log(update(model, message))
console.log(update(model, message))
console.log(update(model, message))
```

This “request/response” pattern keeps `update` pure. The `RequestedApple` handler always returns the same result. It just emits a Command. The actual random generation happens in the Effect, and the result comes back via `CompletedGenerateApplePosition`.

See the [Snake example](https://github.com/foldkit/foldkit/blob/main/examples/snake/src/main.ts#L220-L234) for a complete implementation of this pattern.
