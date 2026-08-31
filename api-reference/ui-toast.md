---
url: https://foldkit.dev/api-reference/ui-toast
title: "Ui/Toast"
description: "API documentation for the Ui/Toast module."
access_date: 2026-08-31T07:29:25.100Z
current_date: 2026-08-31T07:29:25.100Z
---

# Ui/Toast

## Functions

### make

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/toast/index.ts#L136)

## Types

### EntryHandlers

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/toast/index.ts#L100)

```
/**
 * Handlers passed to `entryToView`. Spread `dismiss` onto a close
 *  button's attribute array (typically inside `h.button([...dismiss])`)
 *  to let users dismiss the entry manually. The attribute carries the
 *  Toast's dismiss handler bound to this entry's id; it routes through
 *  the Toast boundary's wrap chain at click time.
 */
type EntryHandlers = Readonly<{
  dismiss: ReadonlyArray<ChildAttribute>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/toast/schema.ts#L120)

```
/**
 * Configuration for creating a toast container model. `defaultDuration` is
 *  applied to any `show()` call that doesn't provide its own `duration` or
 *  pass `sticky: true`. Accepts any Effect Duration input; a bare number is
 *  interpreted as milliseconds.
 */
type InitConfig = Readonly<{
  defaultDuration: Duration.Input
  id: string
}>
```

### ShowInput

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/toast/update.ts#L42)

```
/**
 * Input for `show()`. `payload` is the consumer-defined content shape for an
 *  entry. Omit `duration` to use the container's `defaultDuration`; pass
 *  `sticky: true` to skip auto-dismiss entirely.
 */
type ShowInput = Readonly<{
  duration: Duration.Input
  payload: A
  sticky: boolean
  variant: Variant
}>
```

## Constants

### Message

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/toast/schema.ts#L69)

```
/** Payload-independent Message variants shared by every bound Toast module. */
const Message: MessageUnion<{
  CompletedWaitBeforeDismissal: {
    entryId: String
    version: Number
  }
  Dismissed: {
    entryId: String
  }
  DismissedAll: {}
  GotAnimationMessage: {
    entryId: String
    message: MessageUnion<{
      CompletedWaitForPaint: {}
      EndedAnimation: {}
      Hid: {}
      Showed: {}
    }>
  }
  HoveredEntry: {
    entryId: String
  }
  LeftEntry: {
    entryId: String
  }
}>
```

### Position

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/toast/schema.ts#L22)

```
/** Where the toast viewport is anchored on the screen and how entries stack. */
const Position: Literals<readonly ["TopLeft", "TopCenter", "TopRight", "BottomLeft", "BottomCenter", "BottomRight"]>
```

### Variant

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/toast/schema.ts#L16)

```
/**
 * Semantic category of a toast. Drives the default ARIA role: `status` for
 *  `Info` / `Success`, `alert` for `Warning` / `Error`. Also surfaced as
 *  `data-variant` on each entry for per-variant CSS. This is the only
 *  content-adjacent field the component owns. The rest of the entry's
 *  content lives in the user-provided payload.
 */
const Variant: Literals<readonly ["Info", "Success", "Warning", "Error"]>
```

### WaitBeforeDismissal

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/toast/update.ts#L53)

```
/**
 * Schedules an auto-dismiss timer for an entry. The result Message carries a
 *  version so stale timers (from hover or manual dismiss) are discarded in
 *  the update function. Static. The Command definition doesn't depend on
 *  payload.
 */
const WaitBeforeDismissal: CommandDefinitionWithArgs<"WaitBeforeDismissal", {
  duration: DurationFromMillis
  entryId: String
  version: Number
}, Effect<{
  _tag: "CompletedWaitBeforeDismissal"
  entryId: string
  version: number
}, never, never>>
```
