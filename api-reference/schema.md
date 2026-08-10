---
url: https://foldkit.dev/api-reference/schema
title: "Schema"
description: "API documentation for the Schema module."
access_date: 2026-08-10T02:30:27.972Z
current_date: 2026-08-10T02:30:27.972Z
---

# Schema

## Functions

### ts

function

[source](https://github.com/foldkit/foldkit/blob/3db96663e6d3790ee2866baaf1a00c57fd1da381/packages/foldkit/src/schema/index.ts#L95)

```
/**
 * Wraps `Schema.TaggedStruct` to create a callable tagged struct you can call directly as a constructor.
 * Use `ts` for non-message, non-route tagged structs — enabling `Loading()`
 * instead of `Loading.make()`.
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

[source](https://github.com/foldkit/foldkit/blob/3db96663e6d3790ee2866baaf1a00c57fd1da381/packages/foldkit/src/schema/index.ts#L4)

```
/** A `TaggedStruct` schema that can be called directly as a constructor: `Foo({ count: 1 })` instead of `Foo.make({ count: 1 })`. */
type CallableTaggedStruct = S.TaggedStruct<Tag, Fields> & keyof Fields extends never
  ? (value?: Parameters<S.TaggedStruct<Tag, Fields>["make"]>[0] | void) => Types.Simplify<S.Struct.Type<{
    _tag: S.tag<Tag>
  } & Fields>>
  : (value: Parameters<S.TaggedStruct<Tag, Fields>["make"]>[0]) => Types.Simplify<S.Struct.Type<{
    _tag: S.tag<Tag>
  } & Fields>>
```
