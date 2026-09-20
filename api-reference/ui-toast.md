---
url: https://foldkit.dev/api-reference/ui-toast
title: "Ui/Toast"
description: "API documentation for the Ui/Toast module."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

# Ui/Toast

## Functions

### make

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/index.ts#L155)

### swipeOffset

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/update.ts#L92)

```
/**
 * Horizontal offset in pixels for an entry's swipe state. `Dragging`
 *  reports the distance travelled from the press point; `Dismissing` reports
 *  the offset the release left behind; `Idle` and `Settling` report zero.
 */
(swipeState: {
  _tag: "Idle"
} | {
  _tag: "Dragging"
  currentX: number
  pointerId: number
  startX: number
} | {
  _tag: "Settling"
  offsetX: number
} | {
  _tag: "Dismissing"
  direction: "Left" | "Right"
  offsetX: number
}): number
```

## Types

### EntryHandlers

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/index.ts#L118)

```
/**
 * Handlers passed to `entryToView`. Spread `dismiss` onto a close button's
 *  attributes to dispatch `Dismissed` for this entry.
 */
type EntryHandlers = Readonly<{
  dismiss: ReadonlyArray<ChildAttribute>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L209)

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
  swipeToDismiss: SwipeToDismissConfig
}>
```

### ShowInput

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/update.ts#L49)

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

### SwipeToDismissConfig

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L200)

```
/**
 * Opt-in configuration for swipe-to-dismiss. Without `swipeToDismiss`,
 *  the view attaches no pointer handler and swipe Messages do nothing.
 *  Pass `{}` for the default rightward swipe, `{ threshold }` to change
 *  the distance, or `{ direction: 'Left' }` to swipe left.
 */
type SwipeToDismissConfig = Readonly<{
  direction: SwipeDirection
  threshold: number
}>
```

## Constants

### DEFAULT_SWIPE_DIRECTION

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L69)

```
/** Default direction in which a pointer can dismiss a Toast. */
const DEFAULT_SWIPE_DIRECTION: SwipeDirection
```

### DEFAULT_SWIPE_THRESHOLD

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L66)

```
/** Default distance in pixels a pointer must travel to dismiss a Toast. */
const DEFAULT_SWIPE_THRESHOLD: 40
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L116)

```
/** Payload-independent Message variants shared by every bound Toast module. */
const Message: MessageUnion<{
  CancelledSwipe: {
    pointerId: Number
  }
  CompletedWaitBeforeDismissal: {
    entryId: String
    version: Number
  }
  CompletedWaitForSwipeSettled: {
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
  MovedSwipePointer: {
    clientX: Number
    pointerId: Number
  }
  PressedEntryPointer: {
    clientX: Number
    entryId: String
    pointerId: Number
  }
  PressedEscape: {}
  ReleasedSwipePointer: {
    clientX: Number
    pointerId: Number
  }
}>
```

### Position

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L20)

```
/** Where the toast viewport is anchored on the screen and how entries stack. */
const Position: Literals<readonly ["TopLeft", "TopCenter", "TopRight", "BottomLeft", "BottomCenter", "BottomRight"]>
```

### SWIPE_SETTLE_DURATION

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L73)

```
/**
 * Time an entry remains in `Settling` after a short or cancelled swipe,
 *  allowing consumer CSS to animate it back to rest.
 */
const SWIPE_SETTLE_DURATION: Duration
```

### SwipeDirection

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L33)

```
/** Direction in which a pointer can drag an entry to dismiss it. */
const SwipeDirection: Literals<readonly ["Left", "Right"]>
```

### SwipeState

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L48)

```
/**
 * Per-entry swipe gesture state. `Dragging` retains the initiating
 *  `pointerId`, so move, release, and cancel Messages update only the entry
 *  that started the gesture and ignore unrelated touches. `Settling` returns
 *  a cancelled or short swipe to rest. `Dismissing` retains the release offset
 *  and direction while the leave animation carries the entry off-screen. The
 *  settle generation lives in the entry's `swipeVersion` so a stale settle
 *  timer cannot clear a later gesture.
 */
const SwipeState: TaggedUnion<{
  Dismissing: {
    direction: Literals<readonly ["Left", "Right"]>
    offsetX: Number
  }
  Dragging: {
    currentX: Number
    pointerId: Number
    startX: Number
  }
  Idle: {}
  Settling: {
    offsetX: Number
  }
}>
```

### Variant

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/schema.ts#L14)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/update.ts#L58)

```
/**
 * Waits for an entry's auto-dismiss duration, then emits a versioned
 *  `CompletedWaitBeforeDismissal` Message so update can ignore stale timers.
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

### WaitForSwipeSettled

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/toast/update.ts#L76)

```
/**
 * Waits for a short or cancelled swipe to animate back, then emits
 *  `CompletedWaitForSwipeSettled` so update can clear `Settling`.
 */
const WaitForSwipeSettled: CommandDefinitionWithArgs<"WaitForSwipeSettled", {
  entryId: String
  version: Number
}, Effect<{
  _tag: "CompletedWaitForSwipeSettled"
  entryId: string
  version: number
}, never, never>>
```
