---
url: https://foldkit.dev/core/counter-example
title: "Counter Example"
description: "A minimal Foldkit application explained step by step."
access_date: 2026-08-03T18:55:49.002Z
current_date: 2026-08-03T18:55:49.002Z
---

# A Simple Counter Example

## Overview

Here’s a complete counter application. It wires up the core of the loop from the [Architecture](https://foldkit.dev/core/architecture) page (a Model, Messages, update, init, and view).

A Foldkit app lives in two files. `src/main.ts` holds the pure definitions: Model, Messages, update, init, view, etc. `src/entry.ts` imports them and boots the runtime. The split keeps `main.ts` importable from tests without booting a runtime as a side effect.

```
import { Match as M, Schema as S } from 'effect'
import { Command, Runtime } from 'foldkit'
import type { Document, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

// MODEL

export const Model = S.Struct({
  count: S.Number,
})
export type Model = typeof Model.Type

// MESSAGE

const ClickedDecrement = m('ClickedDecrement')
const ClickedIncrement = m('ClickedIncrement')
const ClickedReset = m('ClickedReset')

export const Message = S.Union([
  ClickedDecrement,
  ClickedIncrement,
  ClickedReset,
])
export type Message = typeof Message.Type

// UPDATE

export const update = (
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

// INIT

export const init: Runtime.ApplicationInit<Model, Message> = () => [
  { count: 0 },
  [],
]

// VIEW

export const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: `Counter: ${model.count}`,
  body: h.div(
    [
      h.Class(
        'min-h-screen bg-white flex flex-col items-center justify-center gap-6 p-6',
      ),
    ],
    [
      h.div(
        [h.Class('text-6xl font-bold text-gray-800')],
        [model.count.toString()],
      ),
      h.div(
        [h.Class('flex flex-wrap justify-center gap-4')],
        [
          h.button(
            [h.OnClick(ClickedDecrement()), h.Class(buttonStyle)],
            ['-'],
          ),
          h.button(
            [h.OnClick(ClickedReset()), h.Class(buttonStyle)],
            ['Reset'],
          ),
          h.button(
            [h.OnClick(ClickedIncrement()), h.Class(buttonStyle)],
            ['+'],
          ),
        ],
      ),
    ],
  ),
})

// STYLE

const buttonStyle = 'bg-black text-white hover:bg-gray-700 px-4 py-2 transition'
```

`entry.ts` is the only place runtime side effects happen. `Runtime.makeApplication` bundles the pieces together. `Runtime.run` starts the app.

```
import { Runtime } from 'foldkit'

import { Model, init, update, view } from './main'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  container: document.getElementById('root'),
})

Runtime.run(application)
```

Don’t worry about understanding every line yet. The next four pages break this code apart piece by piece. After that, we’ll add new features to the counter (a delayed reset, auto-counting, loading saved state) and each one will introduce a new concept.

Let’s start with the Model: the single data structure that holds everything your application can be.
