---
url: https://foldkit.dev/api-reference/struct
title: "Struct"
description: "API documentation for the Struct module."
access_date: 2026-08-07T02:46:48.844Z
current_date: 2026-08-07T02:46:48.844Z
---

# Struct

## Functions

### makeConstrainedEvo

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/struct/index.ts#L36)

```
/** Creates a variant of `evo` whose transforms are checked against a supertype. Useful in generic contexts where `evo`'s `StrictKeys` can't resolve `keyof` on an open type parameter. The returned function evolves a subtype model, preserving all fields not in the transform, and returns the subtype. */
<Constraint extends Record<string, unknown>>(): (model: Model, transforms: EvolveTransform<Constraint>) => Model
```

## Constants

### evo

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/struct/index.ts#L25)

```
/** Immutably updates fields of a struct by applying transform functions. Wraps Effect's `Struct.evolve` with stricter key checking. */
const evo: (t: StrictKeys<O, T>) => (obj: O) => Evolved<O, T>
```
