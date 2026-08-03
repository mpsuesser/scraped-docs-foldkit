---
url: https://foldkit.dev/best-practices/immutability
title: "Immutability"
description: "Immutable model updates with evo for predictable state transitions."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

# Immutability

## Immutable Updates with evo

Foldkit provides `evo` for immutable Model updates. It wraps Effect’s `Struct.evolve` with stricter type checking. If you remove or rename a key from your Model, you’ll get type errors everywhere you try to update it.

```
import { evo } from 'foldkit/struct'

type Model = { count: number; lastUpdated: number }
const model: Model = { count: 0, lastUpdated: 0 }

// evo takes the model and an object of transform functions
const newModel = evo(model, {
  count: count => count + 1,
  lastUpdated: () => Date.now(),
})

// Invalid keys are caught at compile time
const badModel = evo(model, {
  counnt: count => count + 1, // ❌ Error: 'counnt' does not exist in Model
})
```

Each property in the transform object is a function that takes the current value and returns the new value. Properties not included remain unchanged.
