---
url: https://foldkit.dev/example-apps/kanban
title: "Kanban"
description: "Drag-and-drop kanban board with cross-column reordering, keyboard navigation, fractional indexing, and screen reader announcements."
access_date: 2026-08-03T19:40:01.169Z
current_date: 2026-08-03T19:40:01.169Z
---

[All Examples](https://foldkit.dev/example-apps)

# Kanban

Drag-and-drop kanban board with cross-column reordering, keyboard navigation, fractional indexing, and screen reader announcements.

Drag & Drop

Submodels

OutMessage

Storage

[Launch Playground](https://foldkit.dev/playground/kanban)

[View source on GitHub](https://github.com/foldkit/foldkit/tree/main/examples/kanban/src)

/

```
import { Effect, Option, Schema as S } from 'effect'
import { KeyValueStore } from 'effect/unstable/persistence'
import { Runtime } from 'foldkit'

import { BrowserKeyValueStore } from '@effect/platform-browser'
import { DragAndDrop } from '@foldkit/ui'

import { DEFAULT_COLUMNS, STORAGE_KEY } from './constant'
import { Message } from './message'
import { Model, SavedBoard } from './model'
import { subscriptions } from './subscription'
import { update } from './update'
import { view } from './view'

// FLAGS

export const Flags = S.Struct({
  maybeSavedBoard: S.Option(SavedBoard),
})
export type Flags = typeof Flags.Type

export const flags: Effect.Effect<Flags> = Effect.gen(function* () {
  const store = yield* KeyValueStore.KeyValueStore
  const json = yield* Effect.fromOption(
    Option.fromNullishOr(yield* store.get(STORAGE_KEY)),
  )
  const decoded = yield* S.decodeEffect(S.fromJsonString(SavedBoard))(json)
  return Flags.make({ maybeSavedBoard: Option.some(decoded) })
}).pipe(
  Effect.catch(() =>
    Effect.succeed(Flags.make({ maybeSavedBoard: Option.none() })),
  ),
  Effect.provide(BrowserKeyValueStore.layerLocalStorage),
)

// INIT

export const init: Runtime.ApplicationInit<Model, Message, Flags> = flags => {
  const columns = Option.match(flags.maybeSavedBoard, {
    onNone: () => DEFAULT_COLUMNS,
    onSome: ({ columns }) => columns,
  })

  return [
    {
      columns,
      dragAndDrop: DragAndDrop.init({ id: 'kanban' }),
      maybeNewCardColumnId: Option.none(),
      newCardTitle: '',
      announcement: '',
    },
    [],
  ]
}

export { Message, Model, subscriptions, update, view }
```
