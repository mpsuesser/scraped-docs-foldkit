---
url: https://foldkit.dev/api-reference/message
title: "Message"
description: "API documentation for the Message module."
access_date: 2026-08-31T07:29:25.100Z
current_date: 2026-08-31T07:29:25.100Z
---

# Message

## Functions

### defineMessageUnion

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/schema/index.ts#L368)

```
/**
 * Declares every Message variant in one object. Each key is a tag, and its
 * value lists that Message's fields.
 * 
 * The result is both a Schema and a namespace. Construct a Message with a
 * variant such as `Message.ClickedReset()`, and handle every variant with
 * `Message.match`. Each constructor is also a Schema, so it can appear in a
 * `Command.define` `messages` list.
 * 
 * Message unions intentionally expose only constructors and exhaustive
 * `match`. Use Effect `Match` when only some tags need handling or several tags
 * share one handler.
 * 
 * Declare a Submodel's OutMessages in their own `defineMessageUnion`. Messages
 * are facts the Submodel handles; OutMessages are facts it reports to its
 * parent. Keep the two unions separate even when two variants carry the same
 * fields.
 * 
 * A tag cannot use a name already owned by the union, such as `make`, `match`,
 * `cases`, `ast`, `members`, or `subset`. TypeScript rejects these names, and
 * untyped calls throw an error.
 */
<CasesByTag extends Record<string, Fields>>(casesByTag: CasesByTag & ValidateVariantNames<CasesByTag>): MessageUnion<CasesByTag>
```

## Types

### MessageUnion

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/schema/index.ts#L270)

```
/**
 * The Schema returned by `defineMessageUnion`. Each variant is a callable
 * property on the union, and `match` handles the union exhaustively.
 */
type MessageUnion = UnionSchema<CasesByTag> & {
  readonly [Tag in keyof CasesByTag & string]: CallableTaggedStruct<Tag, CasesByTag[Tag]>
}
```
