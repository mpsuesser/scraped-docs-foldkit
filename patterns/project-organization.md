---
url: https://foldkit.dev/patterns/project-organization
title: "Project Organization"
description: "Start with one main module, then separate Messages, Commands, Submodels, and Subscriptions when ownership or file size makes the split useful."
access_date: 2026-10-05T07:06:39.496Z
current_date: 2026-10-05T07:06:39.496Z
---

# Project Organization

Start a Foldkit application in one module. Split it when a feature becomes easier to understand as its own state machine.

## Starting Simple

The smallest application keeps Model, Messages, init, update, and view in `main.ts`. A separate `entry.ts` creates and runs the runtime. Tests can then import `main.ts` without booting the application as a side effect.

Add `story.test.ts` and `scene.test.ts` beside it. The [Counter example](https://foldkit.dev/example-apps/counter) shows this layout.

## File Layout

When one file becomes hard to navigate, separate the root pieces and give each [Submodel](https://foldkit.dev/core/submodel) its own feature folder.

Recommended application structure

Each feature folder owns its Model, Messages, update, view, Commands, Subscriptions, and tests. Do not create empty files only to match the diagram. Add a file when the feature has that concern.

Keep Commands beside the update that returns them. A feature that fetches its own data owns that Command instead of importing it from a root Command collection. Extract `message.ts` when a Command needs to import its result Message constructors without creating a cycle.

A feature that declares Subscriptions owns `subscription.ts`. The parent lifts that record into its own Model and Message types. See [Subscription Organization](https://foldkit.dev/patterns/subscription-organization).

Split a large feature again only when its own files become difficult to navigate. The [Typing Terminal room source](https://github.com/foldkit/foldkit/tree/main/packages/typing-game/client/src/page/room) has `view/` and `update/` subfolders inside one Room feature.

## Where Tests Live

Colocate tests with the boundary they exercise. A feature's `story.test.ts` drives its update. Its `scene.test.ts` drives its view, using `withViewInputs` when the view requires them.

Test rendering, interactions, Commands, and OutMessages inside the feature that owns them. Test at the parent when the contract involves parent-computed ViewInputs, wrapper routing, a lifted Command, or the parent's response to an OutMessage.

Keep root Scene tests for flows that cross features or pages. Split several root flows by subject, such as `checkout.scene.test.ts` and `cart.scene.test.ts`.

When one folder holds more than one test of a kind, prefix with the subject, like `login.story.test.ts`. Pure modules in `domain/` need neither primitive; they take ordinary Vitest tests beside them.

See the [Testing](https://foldkit.dev/testing) page for the full Story and Scene reference.

## Domain Modules

Put shared business concepts in `domain/`. Each module owns its Schema and pure operations.

Domain module

Import the module as a namespace and call operations such as `Cart.addItem` and `Cart.removeItem`.

## Index Re-exports

Use `index.ts` only as a barrel. Re-export the feature's modules from it.

Index re-exports

Consumers can then import the feature as a namespace.

Namespace usage

```
import { Cart, Item } from './domain'
import { Home, Products } from './page'

// Access page modules
Home.Model
Home.view
Home.update

// Access domain modules
Cart.addItem(item)(cart)
Cart.totalItems(cart)
```

`Home.` exposes the feature's public surface without revealing its internal file layout.

### Keep Barrel Dependencies Pointing Inward

A parent barrel may re-export each immediate child namespace, and application code may import those namespaces from the barrel. The exported child modules must not import the application root in return. If `message.ts` imports `Home` from `page/index.ts` while a module re-exported by `page/index.ts` imports that root Message, the barrel closes a circular dependency.

Keep shared leaf modules independent of the root. Pass parent-owned rendering capabilities through function arguments or Submodel `viewInputs`, and make stateful shared behavior its own Submodel. The parent then provides the capability at the view boundary while each Page remains importable through the one-level barrel.
