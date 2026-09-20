---
url: https://foldkit.dev/api-reference/struct
title: "Struct"
description: "API documentation for the Struct module."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

# Struct

## Functions

### makeModifyFieldsFor

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/struct/index.ts#L36)

```
/** Creates a field modifier for a base shape. Use in generic helpers whose Model extends that shape. Transformers are checked against the base shape, and the returned function preserves the Model's subtype and all fields not in the transform. */
<Base extends Record<string, unknown>>(): (model: Model, transforms: EvolveTransform<Base>) => Model
```

## Constants

### modifyFields

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/foldkit/src/struct/index.ts#L25)

```
/** Immutably modifies fields of a struct by applying transform functions. Each transformer must return its field's existing type. Wraps Effect's `Struct.evolve` with stricter key checking. */
const modifyFields: (t: StrictKeys<O, T>) => (obj: O) => Evolved<O, T>
```
