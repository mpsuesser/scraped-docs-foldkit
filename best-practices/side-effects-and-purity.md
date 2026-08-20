---
url: https://foldkit.dev/best-practices/side-effects-and-purity
title: "Side Effects & Purity"
description: "Keep update and view deterministic by confining outside work to Commands, Subscriptions, Mounts, ManagedResources, and other Runtime-managed boundaries."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Side Effects and Purity

## Purity in Foldkit

Foldkit keeps view and update pure. They describe the next UI and any work to perform, but they do not perform that work themselves.

`view` returns a `Document` or `Html` value. Its event handlers construct Messages. `update` returns the next Model, Commands, and, for a Submodel, an optional OutMessage. Given the same inputs, both functions make the same decisions without reading or changing the outside world.

Effectful work lives at boundaries managed by the Runtime. Depending on the boundary, that work is described by an Effect, Stream, or Layer:

- [Commands](https://foldkit.dev/core/commands) describe one-shot work caused by a Message, such as an HTTP request, navigation, storage operation, or focus change.
- [Mount](https://foldkit.dev/core/mount) describes work tied to one live `Element`. Use it for element measurement, observers, portaling, and imperative third-party libraries.
- [Flags](https://foldkit.dev/core/init-and-flags#flags) obtain the outside data needed before init can construct the first Model.
- [Subscriptions](https://foldkit.dev/core/subscriptions) describe ongoing work whose lifetime follows dependencies derived from the Model.
- [Resources](https://foldkit.dev/core/resources) provide app-lifetime services shared by Commands, Subscriptions, Mounts, and Flags.
- [ManagedResource](https://foldkit.dev/core/managed-resources) acquires a typed stateful handle while a Model condition holds. Commands and Subscriptions can use that handle while it is live.

These descriptions do nothing until the Runtime starts them. One narrow exception stays inside its boundary: a mapper passed to `Subscription.fromEvent` may perform synchronous browser work such as `event.preventDefault()` before returning a Message.

A [CustomElement](https://foldkit.dev/core/custom-element) binding remains declarative. Properties flow from the Model into the native element, and its events return as Messages. The browser owns the custom element's internal implementation.

## Why Purity Matters

- **Replay stays safe.** DevTools can replay Messages through update without firing network requests, analytics, storage writes, or DOM work again.
- **State remains explainable.** Each Model follows from the previous Model and one Message. The history does not depend on a hidden callback changing data elsewhere.
- **Tests stay deterministic.** Story tests resolve Command results explicitly, while Scene tests acknowledge effect boundaries surfaced by the rendered view.

## Common Mistakes

For example:

- Production logging inside update runs again during DevTools replay. Put logging and error reporting in a Command. Temporary `console.log` calls are still useful while debugging, but remove them when the investigation ends.
- `Date.now()` and `Math.random()` inside update make the result depend on when it runs. Ask for time or randomness through a Command and return the value in its result Message.
- `fetch` inside view starts work whenever the view renders. Start the request with a Command returned by update.
- Reading `document` or `window` inside view or update hides browser state outside the Model. Use a Command for one-shot reads, a Subscription for ongoing external state, or Mount when the work requires a particular live element.

## View and update

### View is Pure

View reads the Model and any declared ViewInputs, then returns `Document` or `Html`. It does not fetch, schedule timers, subscribe, or read live DOM state. Event attributes construct Messages for the Runtime to dispatch.

```
import type { Document, HtmlBuilder } from 'foldkit/html'

import type { Message } from './message'
import type { Model } from './model'

// ❌ Don't do this in view
const view = (model: Model, h: HtmlBuilder<Message>): Document => {
  fetch('/api/user').then(res => res.json())
  setTimeout(() => console.log('tick'), 1000)
  window.addEventListener('resize', () => {})

  return { title: model.title, body: h.div([], [model.title]) }
}
```

```
import type { Document, HtmlBuilder } from 'foldkit/html'

import { ClickedIncrement, type Message } from './message'
import type { Model } from './model'

// ✅ Keep view pure
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

Update reads the current Model and one Message. It returns a new Model plus descriptions of any work that should follow. It does not mutate the Model, touch the DOM, or execute a Command.

```
import { Match as M } from 'effect'
import { type Command } from 'foldkit'
import { evo } from 'foldkit/struct'

import type { Message } from './message'
import type { Model } from './model'

type UpdateReturn = readonly [Model, ReadonlyArray<Command.Command<Message>>]

// ❌ Don't do this in update
const update = (model: Model, message: Message) =>
  M.value(message).pipe(
    M.withReturnType<UpdateReturn>(),
    M.tagsExhaustive({
      OpenedDialog: () => {
        document.querySelector<HTMLInputElement>('#search-input')?.focus()
        return [evo(model, { dialogState: () => 'Open' }), []]
      },
    }),
  )
```

```
import { Effect, Match as M } from 'effect'
import { Command } from 'foldkit'
import * as Dom from 'foldkit/dom'
import { evo } from 'foldkit/struct'

import { CompletedFocusSearchInput, type Message } from './message'
import type { Model } from './model'

type UpdateReturn = readonly [Model, ReadonlyArray<Command.Command<Message>>]

const FocusSearchInput = Command.define('FocusSearchInput', {
  messages: [CompletedFocusSearchInput],
  execute: Dom.focus('#search-input').pipe(
    Effect.ignore,
    Effect.as(CompletedFocusSearchInput()),
  ),
})

// ✅ Return the next Model and a Command
const update = (model: Model, message: Message) =>
  M.value(message).pipe(
    M.withReturnType<UpdateReturn>(),
    M.tagsExhaustive({
      OpenedDialog: () => [
        evo(model, { dialogState: () => 'Open' }),
        [FocusSearchInput()],
      ],
      CompletedFocusSearchInput: () => [model, []],
    }),
  )
```

The [Testing](https://foldkit.dev/testing) guide shows how Story drives update and resolves Commands without a DOM, while Scene exercises the effect boundaries exposed by a rendered view.

## Requesting Outside Values

Randomness, clocks, storage, and browser APIs produce values that are not already in the Model or Message. Request those values through a Command.

This version generates a different position each time update receives the same inputs:

```
import { Match as M } from 'effect'
import { type Command } from 'foldkit'
import { evo } from 'foldkit/struct'

import { GRID_SIZE } from './constants'
import type { Message } from './message'
import type { Model } from './model'

type UpdateReturn = readonly [Model, ReadonlyArray<Command.Command<Message>>]

// ❌ Don't call random directly in update
const update = (model: Model, message: Message) =>
  M.value(message).pipe(
    M.withReturnType<UpdateReturn>(),
    M.tagsExhaustive({
      RequestedApple: () => {
        const x = Math.floor(Math.random() * GRID_SIZE)
        const y = Math.floor(Math.random() * GRID_SIZE)
        return [evo(model, { apple: () => ({ x, y }) }), []]
      },
    }),
  )
```

The pure version returns `GenerateApplePosition`. Its Effect generates the coordinates and sends them back in `CompletedGenerateApplePosition`:

```
import { Effect, Match as M, Random } from 'effect'
import { Command } from 'foldkit'
import { evo } from 'foldkit/struct'

import { GRID_SIZE } from './constants'
import { CompletedGenerateApplePosition, type Message } from './message'
import type { Model } from './model'

type UpdateReturn = readonly [Model, ReadonlyArray<Command.Command<Message>>]

// ✅ Run random work in a Command
const GenerateApplePosition = Command.define('GenerateApplePosition', {
  messages: [CompletedGenerateApplePosition],
  execute: Effect.gen(function* () {
    const x = yield* Random.nextIntBetween(0, GRID_SIZE, { halfOpen: true })
    const y = yield* Random.nextIntBetween(0, GRID_SIZE, { halfOpen: true })
    return CompletedGenerateApplePosition({ position: { x, y } })
  }),
})

const update = (model: Model, message: Message) =>
  M.value(message).pipe(
    M.withReturnType<UpdateReturn>(),
    M.tagsExhaustive({
      RequestedApple: () => [model, [GenerateApplePosition()]],
      CompletedGenerateApplePosition: ({ position }) => [
        evo(model, { apple: () => position }),
        [],
      ],
    }),
  )
```

`RequestedApple` now returns the same Model and Command every time. Only the result handler writes the generated position into the Model.

See the [Snake example](https://github.com/foldkit/foldkit/blob/main/examples/snake/src/main.ts#L220-L245) for a complete implementation of this pattern.
