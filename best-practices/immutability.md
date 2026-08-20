---
url: https://foldkit.dev/best-practices/immutability
title: "Immutability"
description: "Update Models immutably with evo, preserving references for unchanged branches and keeping state transitions predictable."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Immutability

## Updating the Model with evo

`update` returns a new Model instead of mutating the current one. Foldkit provides `evo` for these immutable field updates. It wraps Effect's `Struct.evolve` with stricter key checking, so removing or renaming a Model field produces errors at every stale update site.

```
import { Schema as S } from 'effect'
import { evo } from 'foldkit/struct'

const Model = S.Struct({
  count: S.Number,
  status: S.Literals(['Idle', 'Counting']),
})
type Model = typeof Model.Type

const model: Model = { count: 0, status: 'Idle' }

const nextModel = evo(model, {
  count: count => count + 1,
  status: () => 'Counting',
})
```

Each property in the transform object receives that field's current value and returns its next value. Omitted properties remain unchanged.

Use the current value when the change depends on it, such as incrementing a count. Use `() => value` when a Message payload or Command result replaces the field. Keep the transformer deterministic. Time, randomness, browser state, and other outside values belong behind a [Command](https://foldkit.dev/core/commands).
