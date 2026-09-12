---
url: https://foldkit.dev/api-reference/ui-drag-and-drop
title: "Ui/DragAndDrop"
description: "API documentation for the Ui/DragAndDrop module."
access_date: 2026-09-12T18:49:33.387Z
current_date: 2026-09-12T18:49:33.387Z
---

# Ui/DragAndDrop

## Functions

### draggable

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L780)

```
/** Returns attributes the parent attaches to a draggable element. Handles pointer-down, keyboard activation, and ARIA. */
<ParentMessage>(
  config: DraggableConfig<ParentMessage>,
  h: HtmlBuilder<ParentMessage>
): readonly Array<Attribute<ParentMessage>>
```

### droppable

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L847)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L868)

```
/** Returns positioning styles for the ghost element, or None when not dragging with a pointer. */
(model: DragAndDrop.Model): Option<Record<string, string>>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L151)

```
/** Creates an initial drag-and-drop model. Starts idle with Vertical orientation and a 5px activation threshold by default. */
(config: InitConfig): DragAndDrop.Model
```

### isDragging

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L887)

```
/** Returns true when the component is actively dragging (pointer or keyboard). */
(__namedParameters: DragAndDrop.Model): boolean
```

### maybeDraggedItemId

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L891)

```
/** Returns the ID of the item currently being dragged or pending, if any. */
(model: DragAndDrop.Model): Option<string>
```

### maybeDropTarget

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L900)

```
/** Returns the current drop target, if any. Populated during pointer drag (from collision detection) and keyboard drag (from resolved position). */
(model: DragAndDrop.Model): Option<{
  containerId: string
  index: number
}>
```

### sortable

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L860)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L309)

## Types

### DraggableConfig

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L771)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L766)

```
/** Messages the draggable view helper can dispatch. */
type DraggableMessage = typeof Message.PressedDraggable.Type | typeof Message.ActivatedKeyboardDrag.Type
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L144)

```
type InitConfig = Readonly<{
  activationThreshold: number
  id: string
  orientation: "Horizontal" | "Vertical"
}>
```

## Constants

### FocusItem

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L164)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L81)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L69)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L127)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L284)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/dragAndDrop/index.ts#L570)

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
