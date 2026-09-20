---
url: https://foldkit.dev/example-apps/ssr
title: "Server-Side Rendering"
description: "A server renders each request into HTML using Flags read from a cookie, and the client hydrates with the exact values the server used. Reload the page and your latest count arrives already in the markup, before any JavaScript runs."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

[All Examples](https://foldkit.dev/example-apps)

# Server-Side Rendering

A server renders each request into HTML using Flags read from a cookie, and the client hydrates with the exact values the server used. Reload the page and your latest count arrives already in the markup, before any JavaScript runs.

Server Rendering

Hydration

Flags

[Launch Playground](https://foldkit.dev/playground/ssr)

[View source on GitHub](https://github.com/foldkit/foldkit/tree/main/examples/ssr/src)

Server-Side Rendering renders each page on a server at request time, so a static preview cannot demonstrate it. Launch the playground to see the server round-trip live, or run the example locally.

```
import { Effect, Schema } from 'effect'
import { Command, Runtime, type Update } from 'foldkit'
import { type Document, type Html, type HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'
import { modifyFields } from 'foldkit/struct'

import { Button } from '@foldkit/ui'

import { COUNT_COOKIE } from './cookie'

// MODEL

export const Model = Schema.Struct({
  count: Schema.Number,
  renderedAt: Schema.String,
  renderedOn: Schema.Literals(['Server', 'Client']),
})
export type Model = typeof Model.Type

// FLAGS

export const Flags = Schema.Struct({
  initialCount: Schema.Number,
  renderedAt: Schema.String,
  renderedOn: Schema.Literals(['Server', 'Client']),
})
export type Flags = typeof Flags.Type

// MESSAGE

export const Message = defineMessageUnion({
  ClickedDecrement: {},
  ClickedIncrement: {},
  CompletedPersistCount: {},
})

export type Message = typeof Message.Type

// UPDATE

export const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    ClickedDecrement: () => {
      const nextCount = model.count - 1
      return {
        model: modifyFields(model, { count: () => nextCount }),
        commands: [PersistCount({ count: nextCount })],
      }
    },
    ClickedIncrement: () => {
      const nextCount = model.count + 1
      return {
        model: modifyFields(model, { count: () => nextCount }),
        commands: [PersistCount({ count: nextCount })],
      }
    },
    CompletedPersistCount: () => ({ model }),
  })

// COMMAND

const COUNT_COOKIE_MAX_AGE_SECONDS = 31536000

export const PersistCount = Command.define('PersistCount', {
  args: { count: Schema.Number },
  messages: [Message.CompletedPersistCount],
  execute: ({ count }) =>
    Effect.try(() => {
      document.cookie = `${COUNT_COOKIE}=${count}; path=/; max-age=${COUNT_COOKIE_MAX_AGE_SECONDS}`
    }).pipe(
      Effect.map(() => Message.CompletedPersistCount()),
      Effect.catch(() => Effect.succeed(Message.CompletedPersistCount())),
    ),
})

// INIT

export const init: Runtime.ApplicationInit<Model, Message, Flags> = flags => ({
  model: {
    count: flags.initialCount,
    renderedAt: flags.renderedAt,
    renderedOn: flags.renderedOn,
  },
})

// VIEW

export const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: `Count ${model.count}`,
  body: h.div(
    [
      h.Class(
        'min-h-screen bg-white flex flex-col items-center justify-center gap-6 p-6',
      ),
    ],
    [
      h.h1(
        [h.Class('text-2xl font-semibold text-gray-800')],
        ['Server-rendered counter'],
      ),
      h.p(
        [h.Id('count'), h.Class('text-6xl font-bold text-gray-800')],
        [model.count.toString()],
      ),
      h.div(
        [h.Class('flex flex-wrap justify-center gap-4')],
        [
          Button.view(
            {
              onClick: Message.ClickedDecrement(),
              toView: attributes =>
                h.button([...attributes.button, h.Class(buttonStyle)], ['-']),
            },
            h,
          ),
          Button.view(
            {
              onClick: Message.ClickedIncrement(),
              toView: attributes =>
                h.button([...attributes.button, h.Class(buttonStyle)], ['+']),
            },
            h,
          ),
        ],
      ),
      h.p(
        [h.Id('provenance'), h.Class('text-sm text-gray-500')],
        [`Rendered on the ${model.renderedOn} at ${model.renderedAt}`],
      ),
      h.p(
        [h.Class('text-sm text-gray-500 max-w-md text-center')],
        [
          'The count persists in a cookie. Reload the page and the server ' +
            'renders your latest count into the HTML before any JavaScript runs.',
        ],
      ),
      parseEquivalenceView(h),
    ],
  ),
})

// NOTE: elements whose served markup and whose freshly built DOM are easy to
// get subtly different. A browser gives a later `selected` option ownership
// while the DOM `value` setter takes the first match, and it drops one newline
// after a <pre> or <textarea> start tag that assigning `innerHTML` would keep.
// The e2e suite loads this page once with scripting off and once hydrated, and
// requires the two readings to agree.
const parseEquivalenceView = (h: HtmlBuilder<Message>): Html =>
  h.div(
    [h.Id('parse-equivalence'), h.Class('hidden')],
    [
      h.select(
        [h.Id('equivalence-select'), h.Value('a')],
        [
          h.option([h.Value('a')], ['A']),
          h.option([h.Value('a'), h.Selected(true)], ['B']),
        ],
      ),
      h.pre([h.Id('equivalence-pre'), h.InnerHTML('\nleading')]),
      h.textarea([h.Id('equivalence-textarea'), h.Value('\nleading')]),
    ],
  )

// STYLE

const buttonStyle = 'bg-black text-white hover:bg-gray-700 px-4 py-2 transition'
```
