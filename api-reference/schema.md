---
url: https://foldkit.dev/api-reference/schema
title: "Schema"
description: "API documentation for the Schema module."
access_date: 2026-09-07T07:29:31.695Z
current_date: 2026-09-07T07:29:31.695Z
---

# Schema

## Functions

### defineTaggedUnion

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/schema/index.ts#L454)

```
/**
 * Declares every variant of a domain union in one object. Use it for Model
 * states, submission results, filter modes, and other unions that are not
 * Messages or Routes.
 * 
 * The result is both a Schema and a namespace. It provides:
 * 
 * - One callable Schema constructor per variant.
 * - `match` for exhaustive handling.
 * - `guards` and `isAnyOf` for variant checks.
 * - `subset` for a Schema that accepts only the named variants.
 * - `members` for APIs such as `Machine.define` that enumerate the union.
 * 
 * Use `taggedStruct` when the variants cannot be declared together. Recursive
 * unions and standalone tagged structs are the common cases.
 * 
 * A tag cannot use a name already owned by the union, such as `make`, `match`,
 * `cases`, `ast`, `members`, or `subset`. TypeScript rejects these names, and
 * untyped calls throw an error.
 */
<CasesByTag extends Record<string, Fields>>(casesByTag: CasesByTag & ValidateVariantNames<CasesByTag>): TaggedUnion<CasesByTag>
```

### taggedStruct

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/schema/index.ts#L531)

```
/**
 * Declares one tagged struct as a callable Schema. Call `Loading()` instead of
 * `Loading.make()`.
 * 
 * Prefer `defineTaggedUnion` when every variant can be declared together. Use
 * `taggedStruct` for a recursive union, a union assembled across modules, a
 * tagged child struct that is not a union variant, or a variant created inside
 * a generic Schema factory.
 */
<Tag extends string>(tag: Tag): CallableTaggedStruct<Tag, {}>

<Tag extends string, Fields extends Fields>(
  tag: Tag,
  fields: Fields
): CallableTaggedStruct<Tag, Fields>
```

## Types

### CallableTaggedStruct

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/schema/index.ts#L4)

```
/** A `TaggedStruct` schema that can be called directly as a constructor: `Foo({ count: 1 })` instead of `Foo.make({ count: 1 })`. */
type CallableTaggedStruct = Schema.TaggedStruct<Tag, Fields> & keyof Fields extends never
  ? (value?: Parameters<Schema.TaggedStruct<Tag, Fields>["make"]>[0] | void) => Types.Simplify<Schema.Struct.Type<{
    _tag: Schema.tag<Tag>
  } & Fields>>
  : (value: Parameters<Schema.TaggedStruct<Tag, Fields>["make"]>[0]) => Types.Simplify<Schema.Struct.Type<{
    _tag: Schema.tag<Tag>
  } & Fields>>
```

### TaggedUnion

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/foldkit/src/schema/index.ts#L296)

```
/**
 * The Schema returned by `defineTaggedUnion`. It includes callable variant
 * constructors, exhaustive `match`, `guards`, `isAnyOf`, `subset`, and
 * `members`.
 */
type TaggedUnion = RichUnionSchema<CasesByTag> & {
  readonly [Tag in keyof CasesByTag & string]: CallableTaggedStruct<Tag, CasesByTag[Tag]>
}
```
