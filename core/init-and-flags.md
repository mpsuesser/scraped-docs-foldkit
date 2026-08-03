---
url: https://foldkit.dev/core/init-and-flags
title: "Init & Flags"
description: "Set up the initial Model, pass external data via flags, and run startup Commands."
access_date: 2026-08-03T19:45:20.723Z
current_date: 2026-08-03T19:45:20.723Z
---

## Init & Flags

## Init

The counter works, but every time the user refreshes the page, the count resets to zero. What if we want to remember the last count? That’s where `init` comes in, and where flags let you pass data into your app at startup.

In the restaurant analogy, init is the waiter’s notebook at the start of the shift: the state of every table before the first customer walks in.

The `init` function returns the initial Model and any Commands to run on startup. It returns a tuple of `[Model, ReadonlyArray<Command<Message>>]`.

```
import { Schema as S } from 'effect'
import type { Runtime } from 'foldkit'
import { m } from 'foldkit/message'

const Model = S.Struct({
  count: S.Number,
})
type Model = typeof Model.Type

const ClickedIncrement = m('ClickedIncrement')
const ClickedDecrement = m('ClickedDecrement')

const Message = S.Union([ClickedIncrement, ClickedDecrement])
type Message = typeof Message.Type

const init: Runtime.ApplicationInit<Model, Message> = () => [{ count: 0 }, []]
```

For elements (components without routing), init takes no arguments. For applications with routing, init receives the current URL so you can set up initial state based on the route.

## Flags

In the restaurant analogy, flags are what the manager tells the waiter before the shift: “table 5 has a reservation at 7, and we’re out of the salmon.” Information from outside the app that shapes the initial state.

Flags let you pass initialization data into your application, like persisted state from localStorage or configuration values. Define a Flags schema and provide an Effect that loads the flags.

```
import { Effect, Option, Schema as S } from 'effect'
import { KeyValueStore } from 'effect/unstable/persistence'

import { BrowserKeyValueStore } from '@effect/platform-browser'

const Todo = S.Struct({
  id: S.String,
  text: S.String,
  completed: S.Boolean,
})

const Todos = S.Array(Todo)

const Flags = S.Struct({
  todos: S.Option(Todos),
})

type Flags = typeof Flags.Type

const flags: Effect.Effect<Flags> = Effect.gen(function* () {
  const store = yield* KeyValueStore.KeyValueStore
  const todosJson = yield* Effect.fromOption(
    Option.fromNullishOr(yield* store.get('todos')),
  )

  const decodeTodos = S.decodeEffect(S.fromJsonString(Todos))
  const todos = yield* decodeTodos(todosJson)

  return Flags.make({ todos: Option.some(todos) })
}).pipe(
  Effect.catch(() => Effect.succeed(Flags.make({ todos: Option.none() }))),
  Effect.provide(BrowserKeyValueStore.layerLocalStorage),
)
```

When using flags, your init function receives them as the first argument:

```
import { Option, Schema as S } from 'effect'
import type { Runtime } from 'foldkit'
import { m } from 'foldkit/message'

const Model = S.Struct({
  count: S.Number,
  startingCount: S.Option(S.Number),
})
type Model = typeof Model.Type

const Flags = S.Struct({
  savedCount: S.Option(S.Number),
})
type Flags = typeof Flags.Type

const ClickedIncrement = m('ClickedIncrement')
const Message = S.Union([ClickedIncrement])
type Message = typeof Message.Type

const init: Runtime.ApplicationInit<Model, Message, Flags> = flags => [
  {
    count: Option.getOrElse(flags.savedCount, () => 0),
    startingCount: flags.savedCount,
  },
  [],
]
```

Both the schema and the Effect are passed to `makeApplication` as `Flags` and `flags`. Without them the runtime calls `init` with no arguments and the compiler rejects the config.

```
import { Runtime } from 'foldkit'

import { Flags, Model, flags, init, update, view } from './main'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  Flags,
  flags,
  container: document.getElementById('root'),
})

Runtime.run(application)
```

The example above discharges its own `KeyValueStore` requirement with `Effect.provide`, which is the right placement for a service used only at startup. When the flags Effect needs an app-wide singleton that Commands also use, leave the requirement in its type and let the `resources` Layer provide it. The runtime builds that Layer once and shares it, so [Resources](https://foldkit.dev/core/resources) covers the details.

Once your app outgrows a single Model, Message, and update, the next step is to decompose it into [Submodels](https://foldkit.dev/core/submodel): self-contained modules with their own state, Messages, and update, embedded under a parent.
