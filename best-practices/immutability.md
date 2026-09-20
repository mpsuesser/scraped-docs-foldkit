---
url: https://foldkit.dev/best-practices/immutability
title: "Immutability"
description: "Update Models immutably with modifyFields, preserving references for unchanged branches and keeping state transitions predictable."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

# Immutability

## Updating the Model with modifyFields

`update` returns a new Model instead of mutating the current one. Foldkit provides `modifyFields` for these immutable field updates. It wraps Effect's `Struct.evolve` with stricter key checking, so removing or renaming a Model field produces errors at every stale update site.

```
import { Schema } from 'effect'
import { modifyFields } from 'foldkit/struct'

const Model = Schema.Struct({
  count: Schema.Number,
  status: Schema.Literals(['Idle', 'Counting']),
})
type Model = typeof Model.Type

const model: Model = { count: 0, status: 'Idle' }

const nextModel = modifyFields(model, {
  count: count => count + 1,
  status: () => 'Counting',
})
```

Each property in the transform object receives that field's current value and returns its next value. Omitted properties remain unchanged.

Use the current value when the change depends on it, such as incrementing a count. Use `() => value` when a Message payload or Command result replaces the field. Keep the transformer deterministic. Time, randomness, browser state, and other outside values belong behind a [Command](https://foldkit.dev/core/commands).
