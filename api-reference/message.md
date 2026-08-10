---
url: https://foldkit.dev/api-reference/message
title: "Message"
description: "API documentation for the Message module."
access_date: 2026-08-10T14:39:49.977Z
current_date: 2026-08-10T14:39:49.977Z
---

# Message

## Functions

### m

function

[source](https://github.com/foldkit/foldkit/blob/7632ab7e37ad7c9e336559a8993e66cd0c5b28bf/packages/foldkit/src/schema/index.ts#L50)

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
