---
url: https://foldkit.dev/api-reference/message
title: "Message"
description: "API documentation for the Message module."
access_date: 2026-08-10T22:19:09.629Z
current_date: 2026-08-10T22:19:09.629Z
---

# Message

## Functions

### m

function

[source](https://github.com/foldkit/foldkit/blob/8f7312dbf0e42c629f8198c425ed1607b8a1d867/packages/foldkit/src/schema/index.ts#L50)

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
