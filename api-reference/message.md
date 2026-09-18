---
url: https://foldkit.dev/api-reference/message
title: "Message"
description: "API documentation for the Message module."
access_date: 2026-09-18T18:12:22.916Z
current_date: 2026-09-18T18:12:22.916Z
---

# Message

## Functions

### defineMessageUnion

function

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/foldkit/src/schema/index.ts#L514)

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
 * `matchOrElse`, `cases`, `ast`, `members`, or `subset`. TypeScript rejects
 * these names, and untyped calls throw an error.
 */
<CasesByTag extends Record<string, Fields>>(casesByTag: CasesByTag & ValidateVariantNames<CasesByTag>): MessageUnion<CasesByTag>
```

## Types

### MessageUnion

type

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/foldkit/src/schema/index.ts#L411)

```
/**
 * The Schema returned by `defineMessageUnion`. Each variant is a callable
 * property on the union, and `match` handles the union exhaustively. Pass a
 * structurally refined union as `match`'s optional second type argument to
 * preserve narrower payload fields in each handler.
 */
type MessageUnion = UnionSchema<CasesByTag> & {
  readonly [Tag in keyof CasesByTag & string]: CallableTaggedStruct<Tag, CasesByTag[Tag]>
}
```
