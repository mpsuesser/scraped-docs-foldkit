---
url: https://foldkit.dev/api-reference/struct
title: "Struct"
description: "API documentation for the Struct module."
access_date: 2026-10-11T22:31:42.984Z
current_date: 2026-10-11T22:31:42.984Z
---

# Struct

## Functions

### makeModifyFieldsFor

function

[source](https://github.com/foldkit/foldkit/blob/060271955cc2fdd34ddb5e17a64b6662268f79c5/packages/foldkit/src/struct/index.ts#L36)

```
/** Creates a field modifier for a base shape. Use in generic helpers whose Model extends that shape. Transformers are checked against the base shape, and the returned function preserves the Model's subtype and all fields not in the transform. */
<Base extends Record<string, unknown>>(): (model: Model, transforms: EvolveTransform<Base>) => Model
```

## Constants

### modifyFields

const

[source](https://github.com/foldkit/foldkit/blob/060271955cc2fdd34ddb5e17a64b6662268f79c5/packages/foldkit/src/struct/index.ts#L25)

```
/** Immutably modifies fields of a struct by applying transform functions. Each transformer must return its field's existing type. Wraps Effect's `Struct.evolve` with stricter key checking. */
const modifyFields: (t: StrictKeys<O, T>) => (obj: O) => Evolved<O, T>
```
