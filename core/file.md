---
url: https://foldkit.dev/core/file
title: "File"
description: "Read and select browser files through an opaque File type, with event attributes for native inputs and drop zones."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

## Overview

The `File` module brings browser file APIs into the Foldkit architecture. It opens the native picker through Commands, reads contents through Effects, and exposes synchronous metadata helpers. Inline inputs and drop zones use the typed file handlers in `foldkit/html` or the FileDrop Submodel.

`File.File` is both a direct alias for the browser's native `File` type and a Schema that accepts native File instances. A Model can hold one with `S.Option(File.File)`. The Schema treats the value as an opaque browser object; it validates the instance but does not turn file contents into serializable Model data.

## Metadata and Reading

`File.name`, `File.size`, and `File.mimeType` read browser-provided metadata synchronously. A MIME type may be an empty string when the browser cannot determine it.

`File.readAsText`, `File.readAsDataUrl`, and `File.readAsArrayBuffer` wrap `FileReader` as interruptible Effects. They fail with `FileReadError` when the browser reports an error or returns an unexpected result type. Catch that error inside the Command and convert it into a declared failure Message.

```
import { Effect } from 'effect'
import { Command, File } from 'foldkit'

const describeFile = (file: File.File): string =>
  \`${File.name(file)} (${File.mimeType(file)}, ${File.size(file)} bytes)\`

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

## Selecting Files

`File.select` and `File.selectMultiple` open a temporary native file input. Both accept a list of MIME types or extensions, which Foldkit forwards to the input's `accept` attribute.

The `accept` list filters the picker UI; it does not validate the selected file. Validate type, size, and contents in your own update and Commands when those constraints matter.

Cancellation is a normal result, not an Effect failure. `File.select` returns `Option.some(file)` after a selection and `Option.none()` after cancellation. `File.selectMultiple` returns all selected files, or an empty array after cancellation. Map either result into domain-specific Messages in the Command.

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

Use [FileDrop](https://foldkit.dev/ui/file-drop) for a headless drop zone with a hidden `<input type="file">`. Its Submodel resets the input so the same file can be selected twice, prevents the browser's default drop behavior, and tracks real drag entry and exit.

FileDrop emits a `ReceivedFiles` OutMessage with a guaranteed non-empty file list. It emits `RejectedNonFiles` when a drop or change contains no files. Fold both variants in the parent update. When FileDrop does not fit the interaction, build directly with the `OnFileChange` and `OnDropFiles` attributes from `foldkit/html`.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Effect, Match as M, Option } from 'effect'
import { File, Update } from 'foldkit'
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

// At module scope, fold the OutMessage FileDrop emits when files arrive (via
// drop or input change) into your own Model. Each arm returns an Update.Step
// over the parent Model, which already has the next FileDrop Model written
// back:
const foldFileDropOutMessage = M.type<FileDrop.OutMessage>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    ReceivedFiles:
      ({ files }) =>
      model => [
        evo(model, {
          uploadedFiles: () => [...model.uploadedFiles, ...files],
        }),
        [],
      ],
    // Fires when something is dropped but no files came through (e.g.
    // a drag of text or a URL). Ignore, or show a hint to the user.
    RejectedNonFiles: () => model => [model, []],
  }),
)

// Update.foldChild wires the child into the parent: it runs FileDrop.update,
// writes the next FileDrop Model back, maps the Submodel's Commands into your
// Message type, and hands any OutMessage to foldOutMessage.
const foldFileDrop = Update.foldChild({
  update: FileDrop.update,
  read: (model: Model) => Option.some(model.uploader),
  write: (model, nextUploader) => evo(model, { uploader: () => nextUploader }),
  toParentMessage: message => GotFileDropMessage({ message }),
  foldOutMessage: foldFileDropOutMessage,
})

// Inside your update function's M.tagsExhaustive({...}), call the fold:
GotFileDropMessage: ({ message }) => foldFileDrop(model, message)

// Render the drop zone. The \`toView\` callback receives attribute groups.
// Spread \`root\` onto a <label> so clicking opens the picker, and spread
// \`input\` onto a hidden <input type="file"> nested inside. Style the
// drag-over state via \`data-drag-over\`.
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

Scene provides `changeFiles` for file inputs and `dropFiles` for drop zones. Each takes a target locator and `ReadonlyArray<File>`, then checks that the matching file-event handler is present before dispatching the synthetic event.

For a button-triggered `File.select` Command, click the button and resolve the Command with the result the picker would have produced. `Command.resolveAll` covers flows where selection immediately starts another Command, such as reading a preview.

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

## Full API Surface

The [File API reference](https://foldkit.dev/api-reference/file) lists the Schema, metadata helpers, readers, selectors, and `FileReadError` signatures.
