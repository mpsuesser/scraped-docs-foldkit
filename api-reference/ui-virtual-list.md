---
url: https://foldkit.dev/api-reference/ui-virtual-list
title: "Ui/VirtualList"
description: "API documentation for the Ui/VirtualList module."
access_date: 2026-08-18T16:38:52.332Z
current_date: 2026-08-18T16:38:52.332Z
---

# Ui/VirtualList

## Functions

### init

function

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L105)

```
/**
 * Creates an initial virtual list model from a config. The container starts
 *  in `Unmeasured` state. The first `ResizeObserver` entry transitions it to
 *  `Measured`.
 */
(config: InitConfig): VirtualList.Model
```

### scrollToIndex

function

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L223)

```
/**
 * Programmatically scrolls the container so the row at `index` is visible.
 *  Returns the next model and a Command that mutates `element.scrollTop`. The
 *  natural scroll event then flows back through `ScrolledContainer` and the
 *  component re-renders the new visible slice.
 * 
 *  Uses version-based cancellation: each call increments
 *  `pendingScrollVersion` so a stale `CompletedApplyScroll` (e.g. from a
 *  previous in-flight scroll) is ignored when its version no longer matches.
 * 
 *  Should be called after the container has rendered. If the container is not
 *  yet in the DOM the Command silently no-ops (the model still transitions
 *  through `ScrollingToIndex` → `Idle` via the version-matched completion).
 * 
 *  Assumes uniform row heights: target scroll position is computed as
 *  `index * model.rowHeightPx`. For variable-height rows, use
 *  `scrollToIndexVariable`.
 */
(
  model: VirtualList.Model,
  index: number
): readonly [
  VirtualList.Model,
  readonly Array<Readonly<{
    args: Record<string, unknown>
    effect: Effect<{
      _tag: "ScrolledContainer"
      scrollTop: number
    } | {
      _tag: "MeasuredContainer"
      containerHeight: number
    } | {
      _tag: "CompletedApplyScroll"
      version: number
    }, never, never>
    key: string
    name: string
  }>>
]
```

### scrollToIndexVariable

function

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L242)

```
/**
 * Variable-height counterpart of `scrollToIndex`. Walks the heights of items
 *  before `index` to compute the target `scrollTop`. Use this when rendering
 *  the list with `itemToRowHeightPx`; use `scrollToIndex` for uniform heights.
 * 
 *  Out-of-range indices clamp to the corresponding edge: negative or zero
 *  scrolls to the top, indices past the end scroll past the last row.
 * 
 *  Note: when restoring `initialScrollTop` on the first measurement of a
 *  variable-height list, the runtime falls back to uniform-height math (using
 *  `model.rowHeightPx`) because items aren't reachable from the `update`
 *  function. Consumers who need an accurate initial scroll on a
 *  variable-height list should call `scrollToIndexVariable` after the first
 *  `MeasuredContainer` arrives.
 */
<Item>(
  model: VirtualList.Model,
  items: readonly Array<Item>,
  itemToRowHeightPx: (item: Item, index: number) => number,
  index: number
): readonly [
  VirtualList.Model,
  readonly Array<Readonly<{
    args: Record<string, unknown>
    effect: Effect<{
      _tag: "ScrolledContainer"
      scrollTop: number
    } | {
      _tag: "MeasuredContainer"
      containerHeight: number
    } | {
      _tag: "CompletedApplyScroll"
      version: number
    }, never, never>
    key: string
    name: string
  }>>
]
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L130)

### view

function

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L566)

```
<Item>(): ViewForItem<Item>
```

### visibleWindow

function

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L296)

```
/**
 * Computes the visible slice of a data array given the current scroll
 *  position, container height, row height, and an overscan buffer.
 * 
 *  Assumes uniform row heights via `model.rowHeightPx`. For variable-height
 *  rows, use `visibleWindowVariable`.
 * 
 *  Returns `Option.none()` when the container has not yet been measured;
 *  callers should render a placeholder (or `Html.empty`) and wait for the
 *  first `MeasuredContainer` message.
 */
(
  model: VirtualList.Model,
  itemCount: number,
  overscan: number
): Option<Readonly<{
  bottomSpacerHeight: number
  endIndex: number
  startIndex: number
  topSpacerHeight: number
}>>
```

### visibleWindowVariable

function

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L340)

```
/**
 * Variable-height counterpart of `visibleWindow`. Walks the heights of every
 *  item to build a prefix-sum array, then locates the visible slice with two
 *  linear searches.
 * 
 *  Cost is O(N) per call, walking the whole `items` array once to build the
 *  prefix sums. For lists in the 10k-item range, this comfortably fits inside
 *  a 60Hz scroll budget. Larger lists or hotter scroll paths can layer a
 *  prefix-sum cache invalidated when items change; that lives behind the same
 *  return shape so consumers don't have to know.
 * 
 *  Returns `Option.none()` when the container has not yet been measured.
 */
<Item>(
  model: VirtualList.Model,
  items: readonly Array<Item>,
  itemToRowHeightPx: (item: Item, index: number) => number,
  overscan: number
): Option<Readonly<{
  bottomSpacerHeight: number
  endIndex: number
  startIndex: number
  topSpacerHeight: number
}>>
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L96)

```
/** Configuration for creating a virtual list model with `init`. */
type InitConfig = Readonly<{
  id: string
  initialScrollTop: number
  rowHeightPx: number
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L545)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  VirtualList does not surface event handlers in the view. All input
 *  (scroll events and resize observations) flows through the
 *  `containerEvents` Subscription. The consumer wraps that
 *  Subscription's stream into their parent Message in their own
 *  `subscriptions` definition.
 */
type ViewInputs = Readonly<{
  containerAttributes: ReadonlyArray<ChildAttribute>
  containerClassName: string
  items: ReadonlyArray<Item>
  itemToKey: (item: Item, index: number) => string
  itemToRowHeightPx: (item: Item, index: number) => number
  itemToView: (item: Item, index: number) => Html
  overscan: number
  rowElement: TagName
}>
```

### VisibleWindow

type

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L262)

```
/**
 * Slice of the data array that the view should render, plus the spacer
 *  heights that keep the scrollbar physically correct. The first row in the
 *  slice corresponds to data index `startIndex`.
 */
type VisibleWindow = Readonly<{
  bottomSpacerHeight: number
  endIndex: number
  startIndex: number
  topSpacerHeight: number
}>
```

## Constants

### CompletedApplyScroll

const

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L75)

```
/**
 * Sent when a `scrollToIndex` Command completes. Carries the version it was
 *  issued with so the update can ignore stale completions.
 */
const CompletedApplyScroll: CallableTaggedStruct<"CompletedApplyScroll", {
  version: Number
}>
```

### MeasuredContainer

const

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L70)

```
/**
 * Sent when the container resizes. Carries the new container height read
 *  from the `ResizeObserver` entry.
 */
const MeasuredContainer: CallableTaggedStruct<"MeasuredContainer", {
  containerHeight: Number
}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L80)

```
/** Union of all messages the virtual list component can produce. */
const Message: S.Union<[typeof ScrolledContainer, typeof MeasuredContainer, typeof CompletedApplyScroll]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L50)

```
/**
 * Schema for the virtual list's state. Tracks scroll position, container
 *  measurement, and any in-flight programmatic scroll.
 */
const Model: Struct<{
  id: String
  measurement: Union<readonly [
    CallableTaggedStruct<"Unmeasured", {}>,
    CallableTaggedStruct<"Measured", {
      containerHeight: Number
    }>
  ]>
  pendingScroll: Union<readonly [
    CallableTaggedStruct<"Idle", {}>,
    CallableTaggedStruct<"ScrollingToIndex", {
      index: Number
      version: Number
    }>
  ]>
  pendingScrollVersion: Number
  rowHeightPx: Number
  scrollTop: Number
}>
```

### ScrolledContainer

const

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L65)

```
/**
 * Sent when the user scrolls the container. Carries the new scroll position
 *  read from the scroll event.
 */
const ScrolledContainer: CallableTaggedStruct<"ScrolledContainer", {
  scrollTop: Number
}>
```

### subscriptions

const

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/ui/src/virtualList/index.ts#L422)

```
/**
 * Subscriptions that track the container's scroll position and size.
 * 
 *  - **scroll**: listens for `scroll` events on the container element and
 *    emits `ScrolledContainer` with the new `scrollTop`.
 *  - **resize**: observes the container with `ResizeObserver` and emits
 *    `MeasuredContainer` with the new height.
 * 
 *  A `MutationObserver` watches the document for the container element
 *  appearing and disappearing, so the listeners attach the moment the
 *  element is inserted into the DOM and clean up when it is removed. This
 *  makes the subscription robust across SPA route changes: navigating to a
 *  page that mounts the list, away, and back all reattach correctly without
 *  the consumer having to teach the framework about navigation.
 */
const subscriptions: {
  containerEvents: EntryWithoutKeepAlive<VirtualList.Model, {
    _tag: "ScrolledContainer"
    scrollTop: number
  } | {
    _tag: "MeasuredContainer"
    containerHeight: number
  } | {
    _tag: "CompletedApplyScroll"
    version: number
  }, {
    id: string
  }, never> & SubscriptionBrand
}
```
