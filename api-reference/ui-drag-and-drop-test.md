---
url: https://foldkit.dev/api-reference/ui-drag-and-drop-test
title: "Ui/DragAndDrop/test"
description: "API documentation for the Ui/DragAndDrop/test module."
access_date: 2026-10-21T00:39:13.665Z
current_date: 2026-10-21T00:39:13.665Z
---

# Ui/DragAndDrop/test

## Functions

### resolveKeyboardMove

function

[source](https://github.com/foldkit/foldkit/blob/7ee214268c602967937b80f6f20d1e831f2e8ab6/packages/ui/src/dragAndDrop/test.ts#L64)

```
/**
 * Returns Story steps that resolve a pending `ResolveKeyboardMove` to the
 *  supplied container and index, then resolve `FocusItem` for `itemId`.
 * 
 *  The steps fail if either Command is missing or multiple keyboard moves are
 *  pending. Unrelated Commands stay pending.
 */
(__namedParameters: ResolveKeyboardMoveInput): Readonly<{
  _tag: "StoryStepsStep"
  steps: readonly Array<StoryStep<any, any, any>>
}>
```

### settleFocus

function

[source](https://github.com/foldkit/foldkit/blob/7ee214268c602967937b80f6f20d1e831f2e8ab6/packages/ui/src/dragAndDrop/test.ts#L28)

```
/**
 * Returns a Story step that resolves `FocusItem({ itemId })` after a keyboard
 *  drop, cancellation, or move. The step fails if no matching Command is pending
 *  and leaves unrelated Commands pending.
 */
(__namedParameters: SettleFocusInput): (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
```

## Types

### ResolveKeyboardMoveInput

type

[source](https://github.com/foldkit/foldkit/blob/7ee214268c602967937b80f6f20d1e831f2e8ab6/packages/ui/src/dragAndDrop/test.ts#L32)

```
/** The dragged item and target position supplied to resolveKeyboardMove. */
type ResolveKeyboardMoveInput = Readonly<{
  itemId: string
  targetContainerId: string
  targetIndex: number
}>
```

### SettleFocusInput

type

[source](https://github.com/foldkit/foldkit/blob/7ee214268c602967937b80f6f20d1e831f2e8ab6/packages/ui/src/dragAndDrop/test.ts#L6)

```
/** Identifies the item whose focus Command settleFocus resolves. */
type SettleFocusInput = Readonly<{
  itemId: string
}>
```
