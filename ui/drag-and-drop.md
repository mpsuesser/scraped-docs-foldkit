---
url: https://foldkit.dev/ui/drag-and-drop
title: "Drag and Drop"
description: "Accessible drag and drop with keyboard support, auto-scrolling, and screen reader announcements."
access_date: 2026-08-11T02:16:28.996Z
current_date: 2026-08-11T02:16:28.996Z
---

## Drag and Drop

## Overview

Sortable lists and cross-container movement with pointer tracking, keyboard navigation, collision detection, auto-scrolling, and screen reader announcements.

DragAndDrop is different from other Foldkit UI components in two ways. First, it doesn’t have a `view()` function. Instead, you spread `draggable()` and `droppable()` attributes onto your own elements. Second, its update function returns a three-tuple: `[Model, Commands, Option<OutMessage>]`. You handle `Reordered` and `Cancelled` OutMessages to decide how to reorder your data.

Integration requires four pieces: a `DragAndDrop.Model` field in your Model, an [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) fold with a `foldOutMessage`, `DragAndDrop.subscriptions` for document-level pointer and keyboard listeners, and `draggable()` / `droppable()` attributes in your view.

See it in an app

Check out how DragAndDrop is wired up in the [kanban example](https://github.com/foldkit/foldkit/tree/main/examples/kanban/src) or the [UI showcase](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/dragAndDrop.ts).

## Examples

### Demo

The snippet below shows a minimal sortable list with all four integration pieces. For a full example with persistence, cross-container moves, and add-card forms, see the [Kanban example](https://foldkit.dev/example-apps/kanban).

Backlog

Done

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit each into your own Model, init, Message,
// update, subscriptions, and view definitions.
import { Effect, Match as M, Option } from 'effect'
import { Subscription, Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { DragAndDrop } from '@foldkit/ui'

// Add a field to your Model for the DragAndDrop Submodel plus the items being sorted:
const Model = S.Struct({
  items: S.Array(S.Struct({ id: S.String, label: S.String })),
  dragAndDrop: DragAndDrop.Model,
  // ...your other fields
})

// In your init function, initialize the DragAndDrop Submodel with a unique id:
const init = () => [
  {
    items: [
      { id: '1', label: 'First' },
      { id: '2', label: 'Second' },
      { id: '3', label: 'Third' },
    ],
    dragAndDrop: DragAndDrop.init({ id: 'sortable-list' }),
    // ...your other fields
  },
  [],
]

// Embed the DragAndDrop Message in your parent Message:
const GotDragAndDropMessage = m('GotDragAndDropMessage', {
  message: DragAndDrop.Message,
})

// At module scope, fold the OutMessage into your own Model. \`Reordered\`
// carries the move so you can apply it to your own list. Each arm returns an
// Update.Step over the parent Model, which already has the next DragAndDrop
// Model written back:
const foldDragAndDropOutMessage = M.type<DragAndDrop.OutMessage>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    Reordered:
      ({ itemId, fromIndex, toIndex }) =>
      model => [
        evo(model, {
          // reorder is your own function that moves the item
          items: () => reorder(model.items, itemId, fromIndex, toIndex),
        }),
        [],
      ],
    // The child has emitted \`Cancelled\`. In this arm the parent can update
    // its own state or dispatch its own Commands, for example revert an
    // optimistic UI change, log analytics, or trigger a downstream Command.
    Cancelled: () => model => [model, []],
  }),
)

// Update.foldChild wires the child into the parent: it runs
// DragAndDrop.update, writes the next DragAndDrop Model back, maps the
// Submodel's Commands into your Message type, and hands any OutMessage to
// foldOutMessage.
const foldDragAndDrop = Update.foldChild({
  update: DragAndDrop.update,
  read: (model: Model) => Option.some(model.dragAndDrop),
  write: (model, nextDragAndDrop) =>
    evo(model, { dragAndDrop: () => nextDragAndDrop }),
  toParentMessage: message => GotDragAndDropMessage({ message }),
  foldOutMessage: foldDragAndDropOutMessage,
})

// Inside your update function's M.tagsExhaustive({...}), call the fold:
GotDragAndDropMessage: ({ message }) => foldDragAndDrop(model, message)

// In your subscriptions, lift all four document-level listeners through
// Subscription.lift in one shot:
const dragAndDropSubscriptions = Subscription.lift({
  dragPointer: DragAndDrop.subscriptions.documentPointer,
  dragEscape: DragAndDrop.subscriptions.documentEscape,
  dragKeyboard: DragAndDrop.subscriptions.documentKeyboard,
  autoScroll: DragAndDrop.subscriptions.autoScroll,
})<Model, Message>({
  toChildModel: model => model.dragAndDrop,
  toParentMessage: message => GotDragAndDropMessage({ message }),
})

const subscriptions = Subscription.aggregate<Model, Message>()(
  dragAndDropSubscriptions,
  // ...your other subscription records
)

// Inside your view function, spread draggable() onto items and droppable()
// onto containers:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.ul(
    [
      ...DragAndDrop.droppable('list', 'Sortable items'),
      h.Class('flex flex-col gap-2'),
    ],
    model.items.map((item, index) =>
      h.li(
        [
          ...DragAndDrop.draggable(
            {
              model: model.dragAndDrop,
              toParentMessage: message => GotDragAndDropMessage({ message }),
              itemId: item.id,
              containerId: 'list',
              index,
            },
            h,
          ),
          h.Class('p-3 rounded-lg border cursor-grab'),
        ],
        [h.span([], [item.label])],
      ),
    ),
  )
```

## Styling

DragAndDrop is fully headless. You render all items, containers, and ghost elements. Use `isDragging()` and `maybeDraggedItemId()` to conditionally style items during drag (e.g. reduced opacity on the source, a drop placeholder at the target).

| Attribute | Condition |
| --- | --- |
| `data-draggable-id` | Set on draggable items with the item ID. |
| `data-sortable-id` | Set on sortable items with the item ID. |
| `data-droppable-id` | Set on drop containers with the container ID. |

## Keyboard Interaction

DragAndDrop supports full keyboard navigation. Space/Enter activates drag mode, arrow keys move the item, Tab/Shift+Tab moves between containers, and Escape cancels.

| Key | Description |
| --- | --- |
| `Space / Enter` | Starts a keyboard drag on the focused item. |
| `Arrow Up / Down` | Moves the item within its container (vertical orientation). |
| `Arrow Left / Right` | Moves the item within its container (horizontal orientation). |
| `Tab / Shift+Tab` | Moves the item to the next / previous container. |
| `Space / Enter` | Drops the dragged item at its current position. |
| `Escape` | Cancels the drag and returns the item to its original position. |

## Accessibility

Draggable items receive `role="option"` with `aria-roledescription="draggable"`. Drop containers receive `role="listbox"`. Screen reader announcements are emitted for drag start, movement, and drop via a live region.

## API Reference

### InitConfig

Configuration object passed to `DragAndDrop.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the drag-and-drop instance. |
| `orientation` | `'Vertical' \| 'Horizontal'` | `'Vertical'` | Item flow direction. Controls arrow key mapping. |
| `activationThreshold` | `number` | `5` | Minimum pointer movement in pixels before a drag activates. Prevents accidental drags from clicks. |

### View Helpers

Functions for attaching drag-and-drop behavior to your elements and reading drag state.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `draggable(config)` | `ReadonlyArray<Attribute>` | — | Spread onto draggable items. Attaches pointer-down, keyboard activation, and ARIA attributes. Config requires model, toParentMessage, itemId, containerId, and index. |
| `droppable(containerId, label?)` | `ReadonlyArray<Attribute>` | — | Spread onto drop containers. Attaches the container ID for collision detection and optional ARIA label. |
| `sortable(itemId)` | `ReadonlyArray<Attribute>` | — | Spread onto items that are both draggable and sortable targets. |
| `ghostStyle(model)` | `Option<CSSProperties>` | — | Returns positioning styles for a ghost element that follows the pointer during drag. Use with Option.match to conditionally render. |
| `isDragging(model)` | `boolean` | — | Whether a drag is currently in progress. |
| `maybeDraggedItemId(model)` | `Option<string>` | — | The ID of the item being dragged, if any. |
| `maybeDropTarget(model)` | `Option<DropTarget>` | — | The current drop target (containerId + index), if any. |

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Fold the OutMessage in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `Reordered` | `{ itemId, fromContainerId, fromIndex, toContainerId, toIndex }` | — | Emitted when a drag completes with a valid drop target. The parent uses this to commit the reorder against its own data (move the item in the source array, splice it into the destination). Fold it in the `foldOutMessage` of your DragAndDrop fold. |
| `Cancelled` | `{}` | — | Emitted when a drag is cancelled via Escape or a pointer release without a valid drop target. No reorder should be applied. |
