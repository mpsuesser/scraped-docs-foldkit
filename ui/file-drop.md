---
url: https://foldkit.dev/ui/file-drop
title: "File Drop"
description: "Headless file drop zone that accepts drag-and-drop plus click-to-browse via a hidden native file input."
access_date: 2026-08-03T18:13:14.939Z
current_date: 2026-08-03T18:13:14.939Z
---

# FileDrop

## Overview

A file drop zone that accepts files via both drag-and-drop and a hidden `<input type="file">`. FileDrop is headless. The component owns drag state and file-arrival events; your `toView` callback owns the visual.

FileDrop uses the Submodel pattern: initialize with `FileDrop.init()`, delegate in your parent update via `FileDrop.update()`, and render with `FileDrop.view()`. The update function returns `[Model, Commands, Option<OutMessage>]`. `ReceivedFiles` fires when files arrive with a guaranteed non-empty list; `RejectedNonFiles` fires when a drop or change event produced no files (e.g. a drag of non-file data). Pattern-match on both.

See it in an app

Check out how FileDrop is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/fileDrop.ts).

## Examples

A multi-file drop zone. Drag files on or click to browse. The component exposes `data-drag-over` on the root while a drag hovers, so you can style the highlighted state with `data-[drag-over]:*` utilities.

Drop files or click to browse

Any file type. This demo just lists them.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Effect, Match as M, Option } from 'effect'
import { Command, File } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { FileDrop } from '@foldkit/ui'

// Add the FileDrop Submodel to your Model, plus a list of accepted files:
const Model = S.Struct({
  uploader: FileDrop.Model,
  uploadedFiles: S.Array(File.File),
  // ...your other fields
})

// Initialize both fields:
const init = () => [
  {
    uploader: FileDrop.init({ id: 'uploader' }),
    uploadedFiles: [],
    // ...your other fields
  },
  [],
]

// Embed FileDrop's Message in your parent Message:
const GotFileDropMessage = m('GotFileDropMessage', {
  message: FileDrop.Message,
})

// Inside your update function's M.tagsExhaustive({...}), delegate to
// FileDrop.update and pattern-match on the OutMessage it emits when files
// arrive (via drop or input change):
GotFileDropMessage: ({ message }) => {
  const [nextUploader, commands, maybeOutMessage] = FileDrop.update(
    model.uploader,
    message,
  )

  const nextFiles = Option.match(maybeOutMessage, {
    onNone: () => model.uploadedFiles,
    onSome: M.type<FileDrop.OutMessage>().pipe(
      M.tagsExhaustive({
        ReceivedFiles: ({ files }) => [...model.uploadedFiles, ...files],
        // Fires when something is dropped but no files came through (e.g.
        // a drag of text or a URL). Ignore, or show a hint to the user.
        RejectedNonFiles: () => model.uploadedFiles,
      }),
    ),
  })

  return [
    evo(model, {
      uploader: () => nextUploader,
      uploadedFiles: () => nextFiles,
    }),
    Command.mapMessages(commands, message => GotFileDropMessage({ message })),
  ]
}

// Render the drop zone. The `toView` callback receives attribute groups.
// Spread `root` onto a <label> so clicking opens the picker, and spread
// `input` onto a hidden <input type="file"> nested inside. Style the
// drag-over state via `data-drag-over`.
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'uploader',
    model: model.uploader,
    view: FileDrop.view,
    viewInputs: {
      multiple: true,
      accept: ['application/pdf', '.doc', '.docx'],
      toView: attributes =>
        h.label(
          [
            ...attributes.root,
            h.Class(
              'flex cursor-pointer flex-col items-center gap-2 rounded-xl border-2 border-dashed border-gray-300 p-8 text-center hover:border-accent-400 data-[drag-over]:border-accent-500 data-[drag-over]:bg-accent-50',
            ),
          ],
          [
            h.p([], ['Drop files or click to browse']),
            h.span([h.Class('text-sm text-gray-500')], ['PDF, DOC, or DOCX']),
            h.input(attributes.input),
          ],
        ),
    },
    toParentMessage: message => GotFileDropMessage({ message }),
  })
```

## Styling

FileDrop is headless. Your `toView` callback composes a `<label>` with the `root` attributes and an `<input>` with the `input` attributes. Wrap the input inside the label so native click-to-browse works. Use `data-[drag-over]:*` and `data-[disabled]:*` utilities to style state variants.

Attribute

Condition

`data-drag-over`

Present on the root while a drag is hovering over the zone.

`data-disabled`

Present on the root when isDisabled is true.

## Accessibility

The hidden `<input type="file">` stays in the DOM but visually hidden via the `sr-only` class so keyboard users can tab to it and trigger the native file picker. Wrapping the input in a `<label>` (via `attributes.root`) means clicking anywhere on the drop zone opens the picker.

## API Reference

### InitConfig

Configuration object passed to `FileDrop.init()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the file-drop instance. Assigned to the hidden

`<input type="file">`

for label association.

### ViewConfig

Configuration object passed to `FileDrop.view()`.

Name

Type

Default

Description

`model`

`FileDrop.Model`

—

The file-drop state from your parent Model.

`toParentMessage`

`(childMessage: FileDrop.Message) => ParentMessage`

—

Wraps FileDrop Messages in your parent Message type for Submodel delegation.

`toView`

`(attributes: FileDropAttributes) => Html`

—

Callback that receives attribute groups for the root drop-zone element and the hidden file input.

`accept`

`ReadonlyArray<string>`

—

List of accepted MIME types or file extensions (e.g. ["application/pdf", ".doc"]). Joined with commas and forwarded to the hidden input's accept attribute. Omit or pass an empty array to accept any file type.

`multiple`

`boolean`

`false`

When true, the hidden input accepts multiple files per selection. Drag-and-drop always accepts multiple files.

`isDisabled`

`boolean`

`false`

Strips drag handlers from the root and disables the input. Styling can react via data-disabled on the root.

### FileDropAttributes

Attribute groups provided to the `toView` callback.

Name

Type

Default

Description

`root`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto the outer drop-zone element (typically a

`<label>`

). Includes drag handlers (dragenter/dragleave/dragover/drop) and data attributes (data-drag-over, data-disabled).

`input`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto a hidden

`<input type="file">`

nested inside the root. Includes the id, type, multiple, accept, sr-only class, and the file-change handler.

### OutMessage

The third element of the update tuple (`[Model, Commands, Option<OutMessage>]`). Pattern-match in your parent update handler to process arriving files.

Name

Type

Default

Description

`ReceivedFiles`

`{ files: NonEmptyReadonlyArray<File> }`

—

Emitted when the user drops files on the zone or selects them via the hidden input. The files list is guaranteed non-empty. Pattern-match on the OutMessage in your parent update to process the files (validate, upload, store in Model).

`RejectedNonFiles`

`{}`

—

Emitted when a drop or input-change event fires without any files, typically a drag of non-file data (text, URLs, images from another page). Consumers can ignore this or surface a hint to the user.
