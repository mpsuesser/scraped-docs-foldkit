---
url: https://foldkit.dev/api-reference/ui-file-drop
title: "Ui/FileDrop"
description: "API documentation for the Ui/FileDrop module."
access_date: 2026-09-02T07:05:07.578Z
current_date: 2026-09-02T07:05:07.578Z
---

# Ui/FileDrop

## Functions

### init

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L52)

```
/** Creates an initial file-drop model. Drag state starts cleared. */
(config: InitConfig): FileDrop.Model
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L61)

## Types

### FileDropAttributes

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L84)

```
/**
 * Attribute groups the file-drop component provides to the consumer's
 *  `toView` callback.
 */
type FileDropAttributes = Readonly<{
  input: ReadonlyArray<ChildAttribute>
  root: ReadonlyArray<ChildAttribute>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L47)

```
/** Configuration for creating a file-drop model with `init`. */
type InitConfig = Readonly<{
  id: string
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L96)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  accept: ReadonlyArray<string>
  isDisabled: boolean
  multiple: boolean
  toView: (attributes: FileDropAttributes) => Html
}>
```

## Constants

### Message

const

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L26)

```
/** Union of all messages the file-drop component can produce. */
const Message: MessageUnion<{
  DroppedFiles: {
    files: NonEmptyArray<Schema<File>>
  }
  DroppedNonFiles: {}
  EnteredDragZone: {}
  LeftDragZone: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L17)

```
/**
 * Schema for the file-drop component's state.
 * 
 * `isDragOver` controls the `data-drag-over` attribute on the root while a
 * drag is hovering. The html layer's `OnDragEnter`/`OnDragLeave` handlers
 * track the per-element active state internally so transitions between
 * children of the zone do not flicker the boolean off-and-on.
 */
const Model: Struct<{
  id: String
  isDragOver: Boolean
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L38)

```
/**
 * The file-drop component's OutMessages: `ReceivedFiles` on the happy
 * path and `RejectedNonFiles` when a drop event fires without files.
 */
const OutMessage: MessageUnion<{
  ReceivedFiles: {
    files: NonEmptyArray<Schema<File>>
  }
  RejectedNonFiles: {}
}>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/2f739758a2786fb72c0983e125a6f3d2f8eae56a/packages/ui/src/fileDrop/index.ts#L112)

```
/**
 * Renders an accessible file-drop zone by publishing attribute groups
 *  for a `<label>`-wrapped hidden file input.
 */
const view: SubmodelView<FileDrop.Model, {
  _tag: "EnteredDragZone"
} | {
  _tag: "LeftDragZone"
} | {
  _tag: "DroppedFiles"
  files: readonly [File, File]
} | {
  _tag: "DroppedNonFiles"
}, Readonly<{
  accept: readonly Array<string>
  isDisabled: boolean
  multiple: boolean
  toView: (attributes: FileDropAttributes) => Html
}>>
```
