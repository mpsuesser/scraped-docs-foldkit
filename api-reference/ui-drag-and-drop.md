---
url: https://foldkit.dev/api-reference/ui-drag-and-drop
title: "Ui/DragAndDrop"
description: "API documentation for the Ui/DragAndDrop module."
access_date: 2026-09-18T16:33:34.359Z
current_date: 2026-09-18T16:33:34.359Z
---

# Ui/DragAndDrop

## Functions

### draggable

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L783)

```
/** Returns attributes the parent attaches to a draggable element. Handles pointer-down, keyboard activation, and ARIA. */
<ParentMessage>(
  config: DraggableConfig<ParentMessage>,
  h: HtmlBuilder<ParentMessage>
): readonly Array<Attribute<ParentMessage>>
```

### droppable

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L850)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L871)

```
/** Returns positioning styles for the ghost element, or None when not dragging with a pointer. */
(model: DragAndDrop.Model): Option<Record<string, string>>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L154)

```
/** Creates an initial drag-and-drop model. Starts idle with Vertical orientation and a 5px activation threshold by default. */
(config: InitConfig): DragAndDrop.Model
```

### isDragging

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L890)

```
/** Returns true when the component is actively dragging (pointer or keyboard). */
(__namedParameters: DragAndDrop.Model): boolean
```

### maybeDraggedItemId

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L894)

```
/** Returns the ID of the item currently being dragged or pending, if any. */
(model: DragAndDrop.Model): Option<string>
```

### maybeDropTarget

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L903)

```
/** Returns the current drop target, if any. Populated during pointer drag (from collision detection) and keyboard drag (from resolved position). */
(model: DragAndDrop.Model): Option<{
  containerId: string
  index: number
}>
```

### sortable

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L863)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L312)

## Types

### DraggableConfig

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L774)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L769)

```
/** Messages the draggable view helper can dispatch. */
type DraggableMessage = typeof Message.PressedDraggable.Type | typeof Message.ActivatedKeyboardDrag.Type
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L147)

```
type InitConfig = Readonly<{
  activationThreshold: number
  id: string
  orientation: "Horizontal" | "Vertical"
}>
```

## Constants

### DragState

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L44)

```
/** Schema for the current pointer or keyboard drag phase. */
const DragState: TaggedUnion<{
  Dragging: {
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
  }
  Idle: {}
  KeyboardDragging: {
    itemId: String
    sourceContainerId: String
    sourceIndex: Number
    targetContainerId: String
    targetIndex: Number
  }
  Pending: {
    containerId: String
    index: Number
    itemId: String
    origin: Struct<{
      screenX: Number
      screenY: Number
    }>
  }
}>
```

### FocusItem

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L167)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L84)

```
/** Union of all messages the drag-and-drop component can produce. */
const Message: MessageUnion<{
  ActivatedKeyboardDrag: {
    containerId: String
    index: Number
    itemId: String
  }
  AdvancedAutoScrollFrame: {}
  CancelledDrag: {}
  CompletedFocusItem: {}
  CompletedResolveKeyboardMove: {
    targetContainerId: String
    targetIndex: Number
  }
  ConfirmedKeyboardDrop: {}
  MovedPointer: {
    clientX: Number
    clientY: Number
    maybeDropTarget: Option<Struct<{
      containerId: String
      index: Number
    }>>
    screenX: Number
    screenY: Number
  }
  PressedArrowKey: {
    direction: Literals<readonly ["Up", "Down", "Left", "Right", "NextContainer", "PreviousContainer"]>
  }
  PressedDraggable: {
    containerId: String
    index: Number
    itemId: String
    screenX: Number
    screenY: Number
  }
  ReleasedPointer: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L72)

```
/** Schema for the drag-and-drop component's state, tracking its unique ID, orientation, and current drag phase. */
const Model: Struct<{
  activationThreshold: Number
  dragState: TaggedUnion<{
    Dragging: {
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
    }
    Idle: {}
    KeyboardDragging: {
      itemId: String
      sourceContainerId: String
      sourceIndex: Number
      targetContainerId: String
      targetIndex: Number
    }
    Pending: {
      containerId: String
      index: Number
      itemId: String
      origin: Struct<{
        screenX: Number
        screenY: Number
      }>
    }
  }>
  id: String
  orientation: Literals<readonly ["Horizontal", "Vertical"]>
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L130)

```
/** Union of all out-messages the drag-and-drop component can emit to its parent. */
const OutMessage: MessageUnion<{
  Cancelled: {}
  Reordered: {
    fromContainerId: String
    fromIndex: Number
    itemId: String
    toContainerId: String
    toIndex: Number
  }
}>
```

### ResolveKeyboardMove

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L287)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/dragAndDrop/index.ts#L573)

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
