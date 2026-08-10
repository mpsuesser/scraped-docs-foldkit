---
url: https://foldkit.dev/api-reference/ui-toast
title: "Ui/Toast"
description: "API documentation for the Ui/Toast module."
access_date: 2026-08-10T15:05:45.218Z
current_date: 2026-08-10T15:05:45.218Z
---

# Ui/Toast

## Functions

### make

function

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/index.ts#L147)

## Types

### EntryHandlers

type

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/index.ts#L111)

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

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L134)

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

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/update.ts#L48)

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

### CompletedWaitBeforeDismissal

const

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L76)

```
/**
 * Sent when an entry's auto-dismiss timer fires. Carries a version echoed
 *  from the scheduling moment so stale timers (from hover or manual dismiss)
 *  are discarded.
 */
const CompletedWaitBeforeDismissal: CallableTaggedStruct<"CompletedWaitBeforeDismissal", {
  entryId: String
  version: Number
}>
```

### Dismissed

const

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L70)

```
/**
 * Sent when an entry should begin dismissing. Starts the leave animation;
 *  the entry is removed from the stack when `TransitionedOut` fires.
 */
const Dismissed: CallableTaggedStruct<"Dismissed", {
  entryId: String
}>
```

### DismissedAll

const

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L72)

```
/** Sent when every currently-visible entry should begin dismissing. */
const DismissedAll: CallableTaggedStruct<"DismissedAll", {}>
```

### GotAnimationMessage

const

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L87)

```
/** Wraps a single entry's Animation submodel message for delegation. */
const GotAnimationMessage: CallableTaggedStruct<"GotAnimationMessage", {
  entryId: String
  message: Union<[CallableTaggedStruct<"Showed", {}>, CallableTaggedStruct<"Hid", {}>, CallableTaggedStruct<"CompletedWaitForPaint", {}>, CallableTaggedStruct<"EndedAnimation", {}>]>
}>
```

### HoveredEntry

const

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L82)

```
/**
 * Sent when the pointer enters an entry. Pauses the auto-dismiss timer by
 *  advancing the entry's version.
 */
const HoveredEntry: CallableTaggedStruct<"HoveredEntry", {
  entryId: String
}>
```

### LeftEntry

const

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L85)

```
/**
 * Sent when the pointer leaves an entry. Restarts the auto-dismiss timer
 *  with the entry's full duration.
 */
const LeftEntry: CallableTaggedStruct<"LeftEntry", {
  entryId: String
}>
```

### Position

const

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L22)

```
/** Where the toast viewport is anchored on the screen and how entries stack. */
const Position: Literals<readonly ["TopLeft", "TopCenter", "TopRight", "BottomLeft", "BottomCenter", "BottomRight"]>
```

### Variant

const

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/schema.ts#L16)

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

[source](https://github.com/foldkit/foldkit/blob/b49c0b85d71d6dae0f470ca305fa00303ff30b9f/packages/ui/src/toast/update.ts#L59)

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
