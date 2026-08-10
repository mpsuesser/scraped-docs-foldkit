---
url: https://foldkit.dev/api-reference/ui-drag-and-drop
title: "Ui/DragAndDrop"
description: "API documentation for the Ui/DragAndDrop module."
access_date: 2026-08-10T18:30:27.283Z
current_date: 2026-08-10T18:30:27.283Z
---

# Ui/DragAndDrop

## Functions

### draggable

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L843)

```
/** Returns attributes the parent attaches to a draggable element. Handles pointer-down, keyboard activation, and ARIA. */
<ParentMessage>(
  config: DraggableConfig<ParentMessage>,
  h: HtmlBuilder<ParentMessage>
): readonly Array<Attribute<ParentMessage>>
```

### droppable

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L910)

```
/**
 * Returns attributes the parent attaches to a droppable container element.
 *  Handler-free, so the bundle is built with `inertHtml` and spreads into
 *  any Message universe's attribute array.
 */
(
  containerId: string,
  label?: string
): readonly Array<Attribute<never>>
```

### ghostStyle

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L931)

```
/** Returns positioning styles for the ghost element, or None when not dragging with a pointer. */
(model: DragAndDrop.Model): Option<Record<string, string>>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L189)

```
/** Creates an initial drag-and-drop model. Starts in the Idle state with Vertical orientation and 5px activation threshold by default. */
(config: InitConfig): DragAndDrop.Model
```

### isDragging

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L950)

```
/** Returns true when the component is actively dragging (pointer or keyboard). */
(__namedParameters: DragAndDrop.Model): boolean
```

### maybeDraggedItemId

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L954)

```
/** Returns the ID of the item currently being dragged or pending, if any. */
(model: DragAndDrop.Model): Option<string>
```

### maybeDropTarget

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L963)

```
/** Returns the current drop target, if any. Populated during pointer drag (from collision detection) and keyboard drag (from resolved position). */
(model: DragAndDrop.Model): Option<{
  containerId: string
  index: number
}>
```

### sortable

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L923)

```
/**
 * Returns attributes the parent attaches to a sortable item element.
 *  Typically combined with `draggable`. Handler-free, so the bundle is built
 *  with `inertHtml` and spreads into any Message universe's attribute
 *  array.
 */
(itemId: string): readonly Array<Attribute<never>>
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L350)

## Types

### DraggableConfig

type

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L834)

```
/** Configuration for creating draggable attributes with `draggable`. */
type DraggableConfig = Readonly<{
  containerId: string
  index: number
  itemId: string
  model: Model
  toParentMessage: (message: DraggableMessage) => ParentMessage
}>
```

### DraggableMessage

type

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L829)

```
/** Messages the draggable view helper can dispatch. */
type DraggableMessage = typeof PressedDraggable.Type | typeof ActivatedKeyboardDrag.Type
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L182)

```
type InitConfig = Readonly<{
  activationThreshold: number
  id: string
  orientation: "Horizontal" | "Vertical"
}>
```

## Constants

### ActivatedKeyboardDrag

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L103)

```
/** The user activated keyboard drag with Space or Enter on a focused draggable. */
const ActivatedKeyboardDrag: CallableTaggedStruct<"ActivatedKeyboardDrag", {
  containerId: String
  index: Number
  itemId: String
}>
```

### AdvancedAutoScrollFrame

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L127)

```
/** An animation frame fired during auto-scroll. */
const AdvancedAutoScrollFrame: CallableTaggedStruct<"AdvancedAutoScrollFrame", {}>
```

### Cancelled

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L171)

```
/** Emitted when a drag is cancelled via Escape or pointer release without a drop target. */
const Cancelled: CallableTaggedStruct<"Cancelled", {}>
```

### CancelledDrag

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L101)

```
/** Escape was pressed during a drag. */
const CancelledDrag: CallableTaggedStruct<"CancelledDrag", {}>
```

### CompletedFocusItem

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L129)

```
/** The FocusItem Command completed. */
const CompletedFocusItem: CallableTaggedStruct<"CompletedFocusItem", {}>
```

### CompletedResolveKeyboardMove

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L109)

```
/** The ResolveKeyboardMove Command resolved the next keyboard drag position. */
const CompletedResolveKeyboardMove: CallableTaggedStruct<"CompletedResolveKeyboardMove", {
  targetContainerId: String
  targetIndex: Number
}>
```

### ConfirmedKeyboardDrop

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L114)

```
/** The user confirmed a keyboard drop with Space or Enter. */
const ConfirmedKeyboardDrop: CallableTaggedStruct<"ConfirmedKeyboardDrop", {}>
```

### FocusItem

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L202)

```
/** Focuses a draggable item by ID after a keyboard move, drop, or cancel. */
const FocusItem: CommandDefinitionWithArgs<"FocusItem", {
  itemId: String
}, Effect<{
  _tag: "CompletedFocusItem"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L132)

```
/** Union of all messages the drag-and-drop component can produce. */
const Message: S.Union<[typeof PressedDraggable, typeof MovedPointer, typeof ReleasedPointer, typeof CancelledDrag, typeof ActivatedKeyboardDrag, typeof CompletedResolveKeyboardMove, typeof ConfirmedKeyboardDrop, typeof PressedArrowKey, typeof AdvancedAutoScrollFrame, typeof CompletedFocusItem]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L71)

```
/** Schema for the drag-and-drop component's state, tracking its unique ID, orientation, and current drag phase. */
const Model: Struct<{
  activationThreshold: Number
  dragState: Union<readonly [
    CallableTaggedStruct<"Idle", {}>,
    CallableTaggedStruct<"Pending", {
      containerId: String
      index: Number
      itemId: String
      origin: Struct<{
        screenX: Number
        screenY: Number
      }>
    }>,
    CallableTaggedStruct<"Dragging", {
      current: Struct<{
        clientX: Number
        clientY: Number
      }>
      itemId: String
      maybeDropTarget: Option<Struct<{
        containerId: String
        index: Number
      }>>
      origin: Struct<{
        screenX: Number
        screenY: Number
      }>
      sourceContainerId: String
      sourceIndex: Number
    }>,
    CallableTaggedStruct<"KeyboardDragging", {
      itemId: String
      sourceContainerId: String
      sourceIndex: Number
      targetContainerId: String
      targetIndex: Number
    }>
  ]>
  id: String
  orientation: Literals<readonly ["Horizontal", "Vertical"]>
}>
```

### MovedPointer

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L91)

```
/** The pointer moved during a drag, with collision detection results. */
const MovedPointer: CallableTaggedStruct<"MovedPointer", {
  clientX: Number
  clientY: Number
  maybeDropTarget: Option<Struct<{
    containerId: String
    index: Number
  }>>
  screenX: Number
  screenY: Number
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L174)

```
/** Union of all out-messages the drag-and-drop component can emit to its parent. */
const OutMessage: Union<readonly [
  CallableTaggedStruct<"Reordered", {
    fromContainerId: String
    fromIndex: Number
    itemId: String
    toContainerId: String
    toIndex: Number
  }>,
  CallableTaggedStruct<"Cancelled", {}>
]>
```

### PressedArrowKey

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L116)

```
/** The user pressed an arrow key during keyboard drag. */
const PressedArrowKey: CallableTaggedStruct<"PressedArrowKey", {
  direction: Literals<readonly ["Up", "Down", "Left", "Right", "NextContainer", "PreviousContainer"]>
}>
```

### PressedDraggable

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L83)

```
/** The user pressed a pointer on a draggable item. */
const PressedDraggable: CallableTaggedStruct<"PressedDraggable", {
  containerId: String
  index: Number
  itemId: String
  screenX: Number
  screenY: Number
}>
```

### ReleasedPointer

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L99)

```
/** The pointer was released. */
const ReleasedPointer: CallableTaggedStruct<"ReleasedPointer", {}>
```

### Reordered

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L163)

```
/** Emitted when a drag completes with a valid drop target. The parent uses this to commit the reorder. */
const Reordered: CallableTaggedStruct<"Reordered", {
  fromContainerId: String
  fromIndex: Number
  itemId: String
  toContainerId: String
  toIndex: Number
}>
```

### ResolveKeyboardMove

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L322)

```
/** Resolves the next keyboard drag position by querying the DOM for adjacent sortable items and containers. */
const ResolveKeyboardMove: CommandDefinitionWithArgs<"ResolveKeyboardMove", {
  currentContainerId: String
  currentIndex: Number
  direction: Literals<readonly ["Up", "Down", "Left", "Right", "NextContainer", "PreviousContainer"]>
  itemId: String
}, Effect<{
  _tag: "CompletedResolveKeyboardMove"
  targetContainerId: string
  targetIndex: number
}, never, never>>
```

### subscriptions

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/ui/src/dragAndDrop/index.ts#L629)

```
/** Document-level subscriptions for pointer and keyboard events during drag operations. */
const subscriptions: {
  autoScroll: EntryWithKeepAlive<DragAndDrop.Model, {
    _tag: "PressedDraggable"
    containerId: string
    index: number
    itemId: string
    screenX: number
    screenY: number
  } | {
    _tag: "MovedPointer"
    clientX: number
    clientY: number
    maybeDropTarget: Option<{
      containerId: string
      index: number
    }>
    screenX: number
    screenY: number
  } | {
    _tag: "ReleasedPointer"
  } | {
    _tag: "CancelledDrag"
  } | {
    _tag: "ActivatedKeyboardDrag"
    containerId: string
    index: number
    itemId: string
  } | {
    _tag: "CompletedResolveKeyboardMove"
    targetContainerId: string
    targetIndex: number
  } | {
    _tag: "ConfirmedKeyboardDrop"
  } | {
    _tag: "PressedArrowKey"
    direction: "Up" | "Down" | "Left" | "Right" | "NextContainer" | "PreviousContainer"
  } | {
    _tag: "AdvancedAutoScrollFrame"
  } | {
    _tag: "CompletedFocusItem"
  }, {
    clientY: number
    isDragging: boolean
  }, never> & SubscriptionBrand
  documentEscape: EntryWithoutKeepAlive<DragAndDrop.Model, {
    _tag: "PressedDraggable"
    containerId: string
    index: number
    itemId: string
    screenX: number
    screenY: number
  } | {
    _tag: "MovedPointer"
    clientX: number
    clientY: number
    maybeDropTarget: Option<{
      containerId: string
      index: number
    }>
    screenX: number
    screenY: number
  } | {
    _tag: "ReleasedPointer"
  } | {
    _tag: "CancelledDrag"
  } | {
    _tag: "ActivatedKeyboardDrag"
    containerId: string
    index: number
    itemId: string
  } | {
    _tag: "CompletedResolveKeyboardMove"
    targetContainerId: string
    targetIndex: number
  } | {
    _tag: "ConfirmedKeyboardDrop"
  } | {
    _tag: "PressedArrowKey"
    direction: "Up" | "Down" | "Left" | "Right" | "NextContainer" | "PreviousContainer"
  } | {
    _tag: "AdvancedAutoScrollFrame"
  } | {
    _tag: "CompletedFocusItem"
  }, {
    dragActivity: "Idle" | "Active"
  }, never> & SubscriptionBrand
  documentKeyboard: EntryWithoutKeepAlive<DragAndDrop.Model, {
    _tag: "PressedDraggable"
    containerId: string
    index: number
    itemId: string
    screenX: number
    screenY: number
  } | {
    _tag: "MovedPointer"
    clientX: number
    clientY: number
    maybeDropTarget: Option<{
      containerId: string
      index: number
    }>
    screenX: number
    screenY: number
  } | {
    _tag: "ReleasedPointer"
  } | {
    _tag: "CancelledDrag"
  } | {
    _tag: "ActivatedKeyboardDrag"
    containerId: string
    index: number
    itemId: string
  } | {
    _tag: "CompletedResolveKeyboardMove"
    targetContainerId: string
    targetIndex: number
  } | {
    _tag: "ConfirmedKeyboardDrop"
  } | {
    _tag: "PressedArrowKey"
    direction: "Up" | "Down" | "Left" | "Right" | "NextContainer" | "PreviousContainer"
  } | {
    _tag: "AdvancedAutoScrollFrame"
  } | {
    _tag: "CompletedFocusItem"
  }, {
    dragActivity: "Idle" | "Active"
  }, never> & SubscriptionBrand
  documentPointer: EntryWithoutKeepAlive<DragAndDrop.Model, {
    _tag: "PressedDraggable"
    containerId: string
    index: number
    itemId: string
    screenX: number
    screenY: number
  } | {
    _tag: "MovedPointer"
    clientX: number
    clientY: number
    maybeDropTarget: Option<{
      containerId: string
      index: number
    }>
    screenX: number
    screenY: number
  } | {
    _tag: "ReleasedPointer"
  } | {
    _tag: "CancelledDrag"
  } | {
    _tag: "ActivatedKeyboardDrag"
    containerId: string
    index: number
    itemId: string
  } | {
    _tag: "CompletedResolveKeyboardMove"
    targetContainerId: string
    targetIndex: number
  } | {
    _tag: "ConfirmedKeyboardDrop"
  } | {
    _tag: "PressedArrowKey"
    direction: "Up" | "Down" | "Left" | "Right" | "NextContainer" | "PreviousContainer"
  } | {
    _tag: "AdvancedAutoScrollFrame"
  } | {
    _tag: "CompletedFocusItem"
  }, {
    dragActivity: "Idle" | "Active"
    orientation: "Horizontal" | "Vertical"
  }, never> & SubscriptionBrand
}
```
