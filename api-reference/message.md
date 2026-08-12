---
url: https://foldkit.dev/api-reference/message
title: "Message"
description: "API documentation for the Message module."
access_date: 2026-08-12T19:19:57.990Z
current_date: 2026-08-12T19:19:57.990Z
---

# Message

## Functions

### m

function

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/foldkit/src/schema/index.ts#L50)

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
