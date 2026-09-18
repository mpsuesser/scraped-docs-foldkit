---
url: https://foldkit.dev/api-reference/struct
title: "Struct"
description: "API documentation for the Struct module."
access_date: 2026-09-18T04:36:53.681Z
current_date: 2026-09-18T04:36:53.681Z
---

# Struct

## Functions

### makeConstrainedEvo

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/foldkit/src/struct/index.ts#L36)

```
/** Creates a variant of `evo` whose transforms are checked against a supertype. Useful in generic contexts where `evo`'s `StrictKeys` can't resolve `keyof` on an open type parameter. The returned function evolves a subtype model, preserving all fields not in the transform, and returns the subtype. */
<Constraint extends Record<string, unknown>>(): (model: Model, transforms: EvolveTransform<Constraint>) => Model
```

## Constants

### evo

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/foldkit/src/struct/index.ts#L25)

```
/** Immutably updates fields of a struct by applying transform functions. Wraps Effect's `Struct.evolve` with stricter key checking. */
const evo: (t: StrictKeys<O, T>) => (obj: O) => Evolved<O, T>
```
