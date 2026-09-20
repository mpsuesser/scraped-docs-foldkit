---
url: https://foldkit.dev/example-apps/livestore
title: "LiveStore"
description: "A LiveStore-backed task list persisted in OPFS that stays reactive across browser tabs. Commands commit events, materializers project them into SQLite, and one Subscription feeds the live query into the Foldkit Model."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

[All Examples](https://foldkit.dev/example-apps)

# LiveStore

A LiveStore-backed task list persisted in OPFS that stays reactive across browser tabs. Commands commit events, materializers project them into SQLite, and one Subscription feeds the live query into the Foldkit Model.

Storage

Subscriptions

Commands

Third-Party Library

[Launch Playground](https://foldkit.dev/playground/livestore)

[View source on GitHub](https://github.com/foldkit/foldkit/tree/main/examples/livestore/src)

/

```
import { Effect, Option, Schema } from 'effect'
import { Runtime } from 'foldkit'

import { Items } from './domain'
import { Message } from './message'
import { Model } from './model'
import {
  ItemsStore,
  type ItemsStoreRequirements,
  orderedItemsQuery,
} from './store'

export const Flags = Schema.Struct({
  items: Items.Items,
})
export type Flags = typeof Flags.Type

export const flags: Effect.Effect<Flags, never, ItemsStoreRequirements> =
  Effect.gen(function* () {
    const items = yield* ItemsStore.query(orderedItemsQuery)

    return Flags.make({ items })
  })

export const init: Runtime.ApplicationInit<Model, Message, Flags> = flags => ({
  model: {
    items: flags.items,
    maybeAddItemError: Option.none(),
    newItemText: '',
    filter: 'All',
  },
})

export { Message, Model }
export { subscriptions } from './subscription'
export { update } from './update'
export { view } from './view'
```
