---
url: https://foldkit.dev/api-reference/message
title: "Message"
description: "API documentation for the Message module."
access_date: 2026-08-05T20:26:10.582Z
current_date: 2026-08-05T20:26:10.582Z
---

# Message

## Functions

### m

function

[source](https://github.com/foldkit/foldkit/blob/88b047b3f387f48deb1848a7ee12f80ebf73f2d1/packages/foldkit/src/schema/index.ts#L50)

```
/**
 * Wraps `Schema.TaggedStruct` to create a message variant you can call directly as a constructor.
 * Use `m` for message types — enabling `ClickedReset()` instead of `ClickedReset.make()`.
 */
<Tag extends string>(tag: Tag): CallableTaggedStruct<Tag, {}>

<Tag extends string, Fields extends Fields>(
  tag: Tag,
  fields: Fields
): CallableTaggedStruct<Tag, Fields>
```
