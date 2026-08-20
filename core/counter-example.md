---
url: https://foldkit.dev/core/counter-example
title: "Counter Example"
description: "Build and trace a minimal Counter through its Model, Message Schema, update, view, init, and Runtime wiring."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# A Simple Counter Example

## See the Whole Loop

This counter puts the core loop from [Architecture](https://foldkit.dev/core/architecture) into one small application. Its Model holds the count. Its Messages record button clicks. Its update function decides the next count, and its view renders the result.

The example uses two files. `src/main.ts` holds the pure application definitions: Model, Messages, update, init, and view. Larger applications can split those definitions into focused modules. `src/entry.ts` remains the runtime boundary, so tests can import the application without starting it as a side effect.

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

The entry imports those definitions and passes them to `Runtime.makeApplication`. `Runtime.run` then starts the application in the selected container.

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

Read the example once for its shape. The next four pages examine the [Model](https://foldkit.dev/core/model), [Messages](https://foldkit.dev/core/messages), [update](https://foldkit.dev/core/update), and [view](https://foldkit.dev/core/view) in order. Later pages extend the same counter with a delayed reset, automatic counting, and saved state to introduce side effects and ongoing work.

Start with the Model, the single data structure that describes the application right now.
