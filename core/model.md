---
url: https://foldkit.dev/core/model
title: "Model"
description: "Define application state as one Schema-backed Model. Foldkit uses its runtime Schema to preserve state across hot updates and validate unknown data."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Model

## One State Tree

The Model is the complete application state in one immutable data structure. Everything the application can be at a moment lives here, rather than being divided between component-local and global stores.

In the [restaurant analogy](https://foldkit.dev/core/architecture#the-restaurant-analogy), this is the waiter's notebook. The analogy is a memory aid; the literal contract is one state tree that every transition receives and returns.

The counter defines its Model with [Effect Schema](https://effect.website/docs/schema/introduction/):

```
import { Schema as S } from 'effect'

// MODEL

const Model = S.Struct({
  count: S.Number,
})
type Model = typeof Model.Type
```

`S.Struct` creates the runtime Schema. `typeof Model.Type` derives the TypeScript type from that same definition, so the runtime and compiler agree on the Model’s shape.

That runtime value matters because TypeScript types disappear after compilation. Foldkit uses the Model Schema to encode and decode state preserved across hot updates. The same Schema can validate unknown data at application boundaries.

The counter starts with one field. When automatic counting becomes part of the application state, the Model grows to record it:

```
import { Schema as S } from 'effect'

// When the counter gains auto-counting,
// the Model grows to hold new state:

const Model = S.Struct({
  count: S.Number,
  isAutoCounting: S.Boolean,
})
type Model = typeof Model.Type
```

Model the application, not the screen

Store facts the application needs to remember. Values used only to render one frame can usually be derived in view instead of becoming another Model field.

The Model describes the current state. Every change begins with a [Message](https://foldkit.dev/core/messages), a fact about something that happened.
