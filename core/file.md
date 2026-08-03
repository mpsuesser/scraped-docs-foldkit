---
url: https://foldkit.dev/core/file
title: "File"
description: "Read and select files from the browser using an opaque File type and event attributes for inputs and drop zones."
access_date: 2026-08-03T18:55:49.002Z
current_date: 2026-08-03T18:55:49.002Z
---

# File

## Overview

The `File` module wraps the browser file APIs as Effects you can run from a Command. It mirrors the design of Elm’s `elm/file` package: file values are opaque, file selection happens imperatively through a Command (not a form event), and file contents are read asynchronously via `FileReader`.

A `File` is a direct alias for the browser’s native `File` type. You can hold one in your Model with `S.Option(File.File)`. Foldkit never serializes files, so the schema acts as an opaque guard rather than a parser.

## Metadata and reading

`File.name`, `File.size`, and `File.mimeType` return metadata synchronously. `File.readAsText`, `File.readAsDataUrl`, and `File.readAsArrayBuffer` wrap the browser’s `FileReader` as Effects that can fail with a `FileReadError`. Use `readAsDataUrl` when you want a preview thumbnail without uploading the file first.

```
import { Effect } from 'effect'
import { Command, File } from 'foldkit'

const describeFile = (file: File.File): string =>
  `${File.name(file)} (${File.mimeType(file)}, ${File.size(file)} bytes)`

const ReadAvatarPreview = Command.define('ReadAvatarPreview', {
  args: { file: File.File },
  messages: [SucceededReadAvatarPreview, FailedReadAvatarPreview],
  execute: ({ file }) =>
    File.readAsDataUrl(file).pipe(
      Effect.map(dataUrl => SucceededReadAvatarPreview({ dataUrl })),
      Effect.catch(error =>
        Effect.succeed(FailedReadAvatarPreview({ reason: error.reason })),
      ),
    ),
})
```

## Selecting files

`File.select` and `File.selectMultiple` open the native file picker and resolve with what the user chose. Both take a list of accepted MIME types or extensions. `File.select` resolves with `Option.some(file)` on a pick or `Option.none()` on cancel; `File.selectMultiple` resolves with the array of chosen files, empty if the user cancels. Mirrors Elm’s `File.Select.file` and `File.Select.files`.

Wrap the Effect in a Command at the call site with `Effect.map` to produce your own Message. The `File` module never defines Messages, so you keep full control of your domain vocabulary.

```
import { Effect, Option } from 'effect'
import { Command, File } from 'foldkit'

const SelectResume = Command.define('SelectResume', {
  messages: [CompletedSelectResume, CancelledSelectResume],
  execute: File.select(['application/pdf']).pipe(
    Effect.map(
      Option.match({
        onNone: () => CancelledSelectResume(),
        onSome: file => CompletedSelectResume({ file }),
      }),
    ),
  ),
})

const SelectAttachments = Command.define('SelectAttachments', {
  messages: [CompletedSelectAttachments],
  execute: File.selectMultiple(['image/*', 'application/pdf']).pipe(
    Effect.map(files => CompletedSelectAttachments({ files })),
  ),
})
```

## Components

For drop zones and inline file pickers, reach for `FileDrop`. It is a Submodel that wires a drop zone and a hidden `<input type="file">` together and emits a `ReceivedFiles` OutMessage when files arrive (whether dropped or picked through the input). It handles the easy-to-miss details for you: it resets the input so the same file can be picked again, calls `preventDefault` on drop, and tracks drag state that flips only on true entry and exit. When you need a shape it does not cover, build directly with the `OnFileChange` and `OnDropFiles` attributes in `foldkit/html`.

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

## Testing

Scene tests exercise file flows through two helpers. `dropFiles` dispatches a synthetic drop event on a drop zone (e.g. the root of a `FileDrop`), and `changeFiles` dispatches a synthetic change event on a file input. Both accept a target locator and a `ReadonlyArray<File>`, and throw a clear error if the target element does not have the matching file-event handler registered.

For button-triggered pickers that use the `File.select` Command, scene tests use `click` on the button and then `Command.resolve` to synthesize the result, bypassing the native file picker entirely. Use `Command.resolveAll` when an update returns multiple Commands at once, or when resolving one Command cascades into others, like reading a preview immediately after a successful selection.

```
import {
  Command,
  changeFiles,
  click,
  dropFiles,
  expect,
  given,
  label,
  role,
  scene,
  text,
} from 'foldkit/scene'
import { describe, test } from 'vitest'

describe('resume upload flow', () => {
  const resume = new File(['%PDF-'], 'resume.pdf', {
    type: 'application/pdf',
  })

  test('inline file input: changeFiles simulates selection', () => {
    scene(
      { update, view },
      given(initialModel),
      changeFiles(label('resume'), [resume]),
      expect(text('resume.pdf')).toExist(),
    )
  })

  test('button-triggered picker: resolve the SelectResume Command', () => {
    const previewDataUrl = 'data:application/pdf;base64,JVBERi0='

    scene(
      { update, view },
      given(initialModel),
      click(role('button', { name: 'Choose resume' })),
      Command.resolveAll(
        [SelectResume, CompletedSelectResume({ file: resume })],
        [ReadResumePreview, SucceededReadPreview({ dataUrl: previewDataUrl })],
      ),
      expect(role('img', { name: 'Resume preview' })).toExist(),
    )
  })

  test('drop zone: dropFiles simulates a drag-and-drop', () => {
    const coverLetter = new File(['cover'], 'cover.txt', {
      type: 'text/plain',
    })
    const portfolio = new File(['<svg/>'], 'portfolio.svg', {
      type: 'image/svg+xml',
    })

    scene(
      { update, view },
      given(initialModel),
      dropFiles(label('attachments'), [coverLetter, portfolio]),
      expect(text('2 attachments selected')).toExist(),
    )
  })
})
```
