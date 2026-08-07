---
url: https://foldkit.dev/api-reference/message
title: "Message"
description: "API documentation for the Message module."
access_date: 2026-08-07T20:55:54.427Z
current_date: 2026-08-07T20:55:54.427Z
---

# Message

## Functions

### m

function

[source](https://github.com/foldkit/foldkit/blob/e528df4cc3ccd2d5719a4a86039ed9f920c6e724/packages/foldkit/src/schema/index.ts#L50)

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
