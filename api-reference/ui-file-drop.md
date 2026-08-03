---
url: https://foldkit.dev/api-reference/ui-file-drop
title: "Ui/FileDrop"
description: "API documentation for the Ui/FileDrop module."
access_date: 2026-08-03T18:13:14.939Z
current_date: 2026-08-03T18:13:14.939Z
---

# Ui/FileDrop

## Functions

### init

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L81)

```
/** Creates an initial file-drop model. Drag state starts cleared. */
(config: InitConfig): FileDrop.Model
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L96)

```
/**
 * Processes a file-drop message and returns the next model, commands,
 * and optional OutMessage.
 */
(
  model: FileDrop.Model,
  message: {
    _tag: "EnteredDragZone"
  } | {
    _tag: "LeftDragZone"
  } | {
    _tag: "DroppedFiles"
    files: readonly [File, File]
  } | {
    _tag: "DroppedNonFiles"
  }
): UpdateReturn
```

## Types

### FileDropAttributes

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L127)

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

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L76)

```
/** Configuration for creating a file-drop model with `init`. */
type InitConfig = Readonly<{
  id: string
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L139)

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

### DroppedFiles

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L35)

```
/**
 * Sent when the user drops files on the zone or selects them via the
 * hidden `<input type="file">`. Carries a non-empty list of `File`
 * objects, resets `isDragOver`, and emits `ReceivedFiles` as an
 * OutMessage.
 */
const DroppedFiles: CallableTaggedStruct<"DroppedFiles", {
  files: NonEmptyArray<Schema<File>>
}>
```

### DroppedNonFiles

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L43)

```
/**
 * Sent when a drop or input-change event fires without any files,
 * typically a drag of non-file data (text, URLs, images from another
 * page). Resets `isDragOver` and emits `RejectedNonFiles` as an
 * OutMessage so the consumer can surface a message (e.g. "Only files are
 * accepted").
 */
const DroppedNonFiles: CallableTaggedStruct<"DroppedNonFiles", {}>
```

### EnteredDragZone

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L27)

```
/**
 * Sent when a drag enters the drop zone. Flips `isDragOver` to true so
 * the consumer's styling can highlight the zone.
 */
const EnteredDragZone: CallableTaggedStruct<"EnteredDragZone", {}>
```

### LeftDragZone

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L30)

```
/**
 * Sent when a drag leaves the drop zone without dropping. Flips
 * `isDragOver` back to false.
 */
const LeftDragZone: CallableTaggedStruct<"LeftDragZone", {}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L46)

```
/** Union of all messages the file-drop component can produce. */
const Message: Union<readonly [
  CallableTaggedStruct<"EnteredDragZone", {}>,
  CallableTaggedStruct<"LeftDragZone", {}>,
  CallableTaggedStruct<"DroppedFiles", {
    files: NonEmptyArray<Schema<File>>
  }>,
  CallableTaggedStruct<"DroppedNonFiles", {}>
]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L17)

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

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L70)

```
/**
 * The file-drop component's OutMessages: `ReceivedFiles` on the happy
 * path and `RejectedNonFiles` when a drop event fires without files.
 */
const OutMessage: Union<readonly [
  CallableTaggedStruct<"ReceivedFiles", {
    files: NonEmptyArray<Schema<File>>
  }>,
  CallableTaggedStruct<"RejectedNonFiles", {}>
]>
```

### ReceivedFiles

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L59)

```
/**
 * Emitted when files arrive via drop or input-change. The consumer's
 * parent update handles this to process the files (validate, upload,
 * store in Model, etc.). The files list is non-empty.
 */
const ReceivedFiles: CallableTaggedStruct<"ReceivedFiles", {
  files: NonEmptyArray<Schema<File>>
}>
```

### RejectedNonFiles

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L66)

```
/**
 * Emitted when a drop or input-change event produces no files. The
 * consumer's parent update handles this to surface a message (e.g. "Only
 * files are accepted").
 */
const RejectedNonFiles: CallableTaggedStruct<"RejectedNonFiles", {}>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/fileDrop/index.ts#L154)

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
