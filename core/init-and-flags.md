---
url: https://foldkit.dev/core/init-and-flags
title: "Init & Flags"
description: "Set up the initial Model, pass external data via flags, and run startup Commands."
access_date: 2026-08-19T19:38:38.072Z
current_date: 2026-08-19T19:38:38.072Z
---

## Init & Flags

## The First Model

`init` constructs the first Model and returns any Commands that should run when the application starts. Its result has the same shape as update's result: `[Model, ReadonlyArray<Command<Message>>]`.

The counter starts at zero and has no startup work:

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

A non-routing application or element calls `init` with no arguments. A routing application passes the current URL, so its first Model can reflect the route. When the application declares Flags, they become the first argument in either form.

## Startup Data from Flags

Flags carry data from outside the application into `init`. Typical sources include persisted state, runtime configuration, and request-specific data supplied during server rendering.

Define the boundary with a Flags Schema. For a fresh client boot, also define an Effect that obtains a value matching that Schema:

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

`init` receives the decoded Flags value and folds it into the first Model:

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

### Fresh Client Boot

Pass the Schema to `Runtime.makeApplication` as `Flags`, then pass the Effect to `Runtime.run`. The runtime resolves the Effect before calling `init`. If the configuration omits the Schema, `init` takes no Flags argument and the compiler rejects mismatched wiring.

```
import { Runtime } from 'foldkit'

import { Flags, Model, flags, init, update, view } from './main'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  Flags,
  container: document.getElementById('root'),
})

Runtime.run(application, { flags })
```

The example provides `KeyValueStore` inside the Flags Effect because that service is used only during startup. If the same singleton is also needed by Commands or Subscriptions, leave the requirement in the Effect type and provide it through the application's `resources` Layer. The runtime builds that Layer once and shares it. See [Resources](https://foldkit.dev/core/resources) for the full setup.

### Server Rendering and Hydration

Server rendering provides Flags from the request or build instead of running a client Flags Effect. `renderToString` uses that value to call `init`, encodes it through the Schema, and embeds the result in the HTML. `Runtime.hydrate` decodes the same value and calls the same `init`, so the client reconstructs the Model that produced the server HTML.

A hydrating entry does not provide a client Flags Effect. Missing or invalid handoff data fails startup instead of silently booting a different Model. The [Server Rendering](https://foldkit.dev/core/server-rendering#flags-and-browser-facts) guide explains which data is safe and reproducible across that boundary.

Once one Model, Message union, and update function become too large to reason about as a unit, decompose the state machine into [Submodels](https://foldkit.dev/core/submodel). Each child owns its own Model, Messages, update, and Commands behind an explicit parent boundary.
