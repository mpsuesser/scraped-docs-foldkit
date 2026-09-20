---
url: https://foldkit.dev/ui/dialog
title: "Dialog"
description: "A modal dialog backed by the native dialog element with focus trapping and scroll locking."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

## Overview

A modal dialog backed by the native `<dialog>` element, opened with `show()` and a high z-index. The framework manages focus trapping, Escape handling, scroll locking, and backdrop rendering. For non-modal floating content, use Popover instead.

See it in an app

Check out how Dialog is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/dialog.ts).

## Examples

### Basic

Open the Dialog from a trigger by dispatching your own Message. Fold `Dialog.open` and `Dialog.close` into the parent with `Update.foldChildStep`; both are no-argument child entry points. Spread the `closeButton` bundle onto a Cancel button to dismiss it. Spread `...title` onto a heading element so the Dialog is labeled for screen readers.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Option, Schema } from 'effect'
import { Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'
import { modifyFields } from 'foldkit/struct'

import { Dialog } from '@foldkit/ui'

// Add a field to your Model for the Dialog Submodel:
const Model = Schema.Struct({
  dialog: Dialog.Model,
  // ...your other fields
})
type Model = typeof Model.Type

// In your init function, initialize the Dialog Submodel with a unique id:
const init = () => ({
  model: {
    dialog: Dialog.init({ id: 'confirm' }),
    // ...your other fields
  },
})

// A fact for the trigger, plus the Dialog Message embedded in your parent
// Message for the submodel delegation:
const Message = defineMessageUnion({
  ClickedOpenDialog: {},
  GotDialogMessage: { message: Dialog.Message },
})
type Message = typeof Message.Type

// One boundary handles Dialog Messages, Commands, and OutMessages. Replace
// either no-op arm with the parent transition that should follow that event.
const foldDialogOutMessage = Dialog.OutMessage.match<
  Update.Step<Model, Message>
>({
  Opened: () => model => ({ model }),
  Closed: () => model => ({ model }),
})

const readDialog = (model: Model) => Option.some(model.dialog)
const writeDialog = (model: Model, dialog: Dialog.Model): Model =>
  modifyFields(model, { dialog: () => dialog })
const toGotDialogMessage = (message: Dialog.Message): Message =>
  Message.GotDialogMessage({ message })

const foldDialog = Update.foldChild({
  update: Dialog.update,
  read: readDialog,
  write: writeDialog,
  toParentMessage: toGotDialogMessage,
  foldOutMessage: foldDialogOutMessage,
})

const foldDialogOpen = Update.foldChildStep({
  update: Dialog.open,
  read: readDialog,
  write: writeDialog,
  toParentMessage: toGotDialogMessage,
  foldOutMessage: foldDialogOutMessage,
})

// In the corresponding Message.match handler:
ClickedOpenDialog: () => foldDialogOpen(model)
GotDialogMessage: ({ message }) => foldDialog(model, message)

// In your view, open from a trigger with the fact, and dismiss from a Cancel
// button by spreading the \`closeButton\` bundle, no parent message needed:
const view = (h: HtmlBuilder<Message>) =>
  h.div(
    [],
    [
      h.button([h.OnClick(Message.ClickedOpenDialog())], ['Open Dialog']),
      h.submodel({
        slotId: model.dialog.id,
        model: model.dialog,
        view: Dialog.view,
        viewInputs: {
          hasDescription: true,
          toView: ({
            dialog,
            backdrop,
            panel,
            title,
            description,
            closeButton,
            isVisible,
          }) =>
            h.dialog(
              [...dialog],
              isVisible
                ? [
                    h.div([...backdrop, h.Class('fixed inset-0 bg-black/50')]),
                    h.div(
                      [
                        ...panel,
                        h.Class('rounded-lg p-6 max-w-md mx-auto shadow-xl'),
                      ],
                      [
                        h.h2([...title], ['Confirm Action']),
                        h.p(
                          [...description],
                          ['Are you sure you want to proceed?'],
                        ),
                        h.button(
                          [
                            ...closeButton,
                            h.Class('px-4 py-2 rounded-lg border'),
                          ],
                          ['Cancel'],
                        ),
                      ],
                    ),
                  ]
                : [],
            ),
        },
        toParentMessage: message => Message.GotDialogMessage({ message }),
      }),
    ],
  )
```

### Animated

Pass `isAnimated: true` at init to coordinate animations. The component manages an Animation submodel internally. Apply transition classes using `data-closed` (e.g. `data-[closed]:opacity-0 data-[closed]:scale-95`).

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Option, Schema } from 'effect'
import { Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'
import { modifyFields } from 'foldkit/struct'

import { Dialog } from '@foldkit/ui'

// Add a field to your Model for the Dialog Submodel:
const Model = Schema.Struct({
  dialog: Dialog.Model,
  // ...your other fields
})
type Model = typeof Model.Type

// In your init function, set isAnimated: true to coordinate CSS transitions:
const init = () => ({
  model: {
    dialog: Dialog.init({ id: 'confirm', isAnimated: true }),
    // ...your other fields
  },
})

// Embed the Dialog Message in your parent Message and delegate to
// Dialog.update (open from a trigger with a fact and Dialog.open, as in
// the basic Dialog example):
const Message = defineMessageUnion({
  GotDialogMessage: { message: Dialog.Message },
})
type Message = typeof Message.Type

const foldDialogOutMessage = Dialog.OutMessage.match<
  Update.Step<Model, Message>
>({
  Opened: () => model => ({ model }),
  Closed: () => model => ({ model }),
})

const foldDialog = Update.foldChild({
  update: Dialog.update,
  read: (model: Model) => Option.some(model.dialog),
  write: (model, nextDialog) =>
    modifyFields(model, { dialog: () => nextDialog }),
  toParentMessage: message => Message.GotDialogMessage({ message }),
  foldOutMessage: foldDialogOutMessage,
})

GotDialogMessage: ({ message }) => foldDialog(model, message)

// Inside your view function, use data-[closed] for enter/leave transitions and
// spread the \`closeButton\` bundle onto your dismiss buttons:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: model.dialog.id,
    model: model.dialog,
    view: Dialog.view,
    viewInputs: {
      hasDescription: true,
      toView: ({
        dialog,
        backdrop,
        panel,
        title,
        description,
        closeButton,
        isVisible,
      }) =>
        h.dialog(
          [
            ...dialog,
            h.Class('bg-transparent p-0 open:flex items-center justify-center'),
          ],
          isVisible
            ? [
                h.div([
                  ...backdrop,
                  h.Class(
                    'fixed inset-0 bg-black/50 transition duration-150 ease-out data-[closed]:opacity-0',
                  ),
                ]),
                h.div(
                  [
                    ...panel,
                    h.Class(
                      'rounded-lg p-6 max-w-md mx-auto shadow-xl transition duration-150 ease-out data-[closed]:opacity-0 data-[closed]:scale-95',
                    ),
                  ],
                  [
                    h.h2([...title], ['Confirm Action']),
                    h.p(
                      [...description],
                      ['Are you sure you want to proceed?'],
                    ),
                    h.div(
                      [h.Class('flex gap-2 justify-end mt-4')],
                      [
                        h.button(
                          [
                            ...closeButton,
                            h.Class('px-4 py-2 rounded-lg border'),
                          ],
                          ['Cancel'],
                        ),
                        h.button(
                          [
                            ...closeButton,
                            h.Class(
                              'px-4 py-2 rounded-lg bg-blue-600 text-white',
                            ),
                          ],
                          ['Confirm'],
                        ),
                      ],
                    ),
                  ],
                ),
              ]
            : [],
        ),
    },
    toParentMessage: message => Message.GotDialogMessage({ message }),
  })
```

### Field

A field inside a dialog can open its own overlay, like a Combobox or DatePicker. By default that overlay portals its panel to the document body, where the dialog renders on top of it. Pass `anchor: { portal: false }` so the panel stays inside the dialog and remains visible.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Option, Schema } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'

import { Combobox, Dialog } from '@foldkit/ui'

// One Model field for the dialog, one for the overlay it contains, plus
// the parent-owned selection (\`City\` and \`CityCombobox\` are the
// \`Schema.Literals\` Schema and typed factory from the Combobox example):
const Model = Schema.Struct({
  dialog: Dialog.Model,
  combobox: Combobox.Model,
  maybeCity: Schema.Option(City),
  // ...your other fields
})

const init = () => ({
  model: {
    dialog: Dialog.init({ id: 'edit-filters' }),
    combobox: Combobox.init({ id: 'city' }),
    maybeCity: Option.none(),
    // ...your other fields
  },
})

// Embed each submodel's Message in your parent Message and delegate both to
// their own update (see the Dialog and Combobox examples for the delegation).
const Message = defineMessageUnion({
  GotDialogMessage: { message: Dialog.Message },
  GotComboboxMessage: { message: Combobox.Message },
})

// Render the overlay inside the dialog panel. The key is \`portal: false\` on
// the overlay's anchor. By default the panel portals to the document body,
// where the dialog's high stacking order hides it. With portal: false the
// panel stays inside the dialog and renders above the panel content.
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: model.dialog.id,
    model: model.dialog,
    view: Dialog.view,
    viewInputs: {
      toView: ({ dialog, backdrop, panel, title, isVisible }) =>
        h.dialog(
          [...dialog],
          isVisible
            ? [
                h.div([...backdrop, h.Class('fixed inset-0 bg-black/50')]),
                h.div(
                  [
                    ...panel,
                    h.Class('rounded-lg p-6 max-w-md mx-auto shadow-xl'),
                  ],
                  [
                    h.h2([...title], ['Edit filters']),
                    h.submodel({
                      slotId: model.combobox.id,
                      model: model.combobox,
                      view: CityCombobox.view,
                      viewInputs: {
                        // ...items, itemToConfig, itemToValue, etc.
                        maybeSelectedValue: model.maybeCity,
                        restingInputValue: Option.getOrElse(
                          model.maybeCity,
                          () => '',
                        ),
                        anchor: { placement: 'bottom-start', portal: false },
                      },
                      toParentMessage: message =>
                        Message.GotComboboxMessage({ message }),
                    }),
                  ],
                ),
              ]
            : [],
        ),
    },
    toParentMessage: message => Message.GotDialogMessage({ message }),
  })
```

### Stacked

Use a separate Dialog Model for each level and open the second from a button in the first. The framework stacks them by z-index, isolates the page around the topmost Dialog, and traps focus there. Escape closes the top Dialog first; its parent becomes interactive again and receives focus.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Option, Schema } from 'effect'
import { Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'
import { modifyFields } from 'foldkit/struct'

import { Dialog } from '@foldkit/ui'

// One Model field per dialog level:
const Model = Schema.Struct({
  settingsDialog: Dialog.Model,
  confirmDialog: Dialog.Model,
  // ...your other fields
})
type Model = typeof Model.Type

const init = () => ({
  model: {
    settingsDialog: Dialog.init({ id: 'settings' }),
    confirmDialog: Dialog.init({ id: 'confirm-delete' }),
    // ...your other fields
  },
})

// Embed each Dialog Message in your parent Message and delegate each to its
// own Dialog.update (see the basic Dialog example for the delegation).
const Message = defineMessageUnion({
  GotSettingsDialogMessage: { message: Dialog.Message },
  GotConfirmDialogMessage: { message: Dialog.Message },
  ClickedDeleteProject: {},
  ConfirmedDeleteProject: {},
})
type Message = typeof Message.Type

const foldConfirmDialogOutMessage = Dialog.OutMessage.match<
  Update.Step<Model, Message>
>({
  Opened: () => model => ({ model }),
  Closed: () => model => ({ model }),
})

const readConfirmDialog = (model: Model) => Option.some(model.confirmDialog)
const writeConfirmDialog = (model: Model, confirmDialog: Dialog.Model): Model =>
  modifyFields(model, { confirmDialog: () => confirmDialog })
const toGotConfirmDialogMessage = (message: Dialog.Message): Message =>
  Message.GotConfirmDialogMessage({ message })

const foldConfirmDialogOpen = Update.foldChildStep({
  update: Dialog.open,
  read: readConfirmDialog,
  write: writeConfirmDialog,
  toParentMessage: toGotConfirmDialogMessage,
  foldOutMessage: foldConfirmDialogOutMessage,
})

const foldConfirmDialogClose = Update.foldChildStep({
  update: Dialog.close,
  read: readConfirmDialog,
  write: writeConfirmDialog,
  toParentMessage: toGotConfirmDialogMessage,
  foldOutMessage: foldConfirmDialogOutMessage,
})

// Opening the confirmation is a parent fact, not a hand-wrapped child message.
// The button dispatches ClickedDeleteProject; the update opens the confirmation
// through Dialog.open, keeping Got* for genuine child results.

// In the corresponding Message.match handler:
ClickedDeleteProject: () => foldConfirmDialogOpen(model)

// Confirming runs the deletion, then closes the confirmation through
// Dialog.close, the same API the opening fact used.
// Add the deletion as another Update.combine step when implementing it.
ConfirmedDeleteProject: () => foldConfirmDialogClose(model)

// Each dialog is its own submodel; the framework stacks them by z-index, traps
// focus in the topmost, and Escape closes the topmost before the one beneath
// it. Cancel dismisses the confirmation through the \`closeButton\` bundle.
// Delete dispatches a fact that runs the work and closes through Dialog.close.
const view = (h: HtmlBuilder<Message>) => {
  const confirmDialog = h.submodel({
    slotId: model.confirmDialog.id,
    model: model.confirmDialog,
    view: Dialog.view,
    viewInputs: {
      toView: ({ dialog, backdrop, panel, title, closeButton, isVisible }) =>
        h.dialog(
          [...dialog],
          isVisible
            ? [
                h.div([...backdrop, h.Class('fixed inset-0 bg-black/50')]),
                h.div(
                  [
                    ...panel,
                    h.Class('rounded-lg p-6 max-w-sm mx-auto shadow-xl'),
                  ],
                  [
                    h.h2([...title], ['Delete project?']),
                    h.button([...closeButton], ['Cancel']),
                    h.button(
                      [h.OnClick(Message.ConfirmedDeleteProject())],
                      ['Delete'],
                    ),
                  ],
                ),
              ]
            : [],
        ),
    },
    toParentMessage: message => Message.GotConfirmDialogMessage({ message }),
  })

  const settingsDialog = h.submodel({
    slotId: model.settingsDialog.id,
    model: model.settingsDialog,
    view: Dialog.view,
    viewInputs: {
      toView: ({ dialog, backdrop, panel, title, isVisible }) =>
        h.dialog(
          [...dialog],
          isVisible
            ? [
                h.div([...backdrop, h.Class('fixed inset-0 bg-black/50')]),
                h.div(
                  [
                    ...panel,
                    h.Class('rounded-lg p-6 max-w-lg mx-auto shadow-xl'),
                  ],
                  [
                    h.h2([...title], ['Project settings']),
                    h.button(
                      [h.OnClick(Message.ClickedDeleteProject())],
                      ['Delete project'],
                    ),
                  ],
                ),
              ]
            : [],
        ),
    },
    toParentMessage: message => Message.GotSettingsDialogMessage({ message }),
  })

  return h.div([], [settingsDialog, confirmDialog])
}
```

## Styling

Dialog is headless. The `toView` callback receives attribute bundles for the dialog, backdrop, panel, and closeButton, and the consumer composes the markup. Dialog renders no backdrop of its own, so build your own from the `backdrop` bundle for full control over its appearance.

When `isAnimated` is true, enter/leave animations flow through the [Animation](https://foldkit.dev/ui/animation) module. Style with CSS transitions or CSS keyframe animations. Animation advances once every animation on the element has settled.

| Attribute | Condition |
| --- | --- |
| `data-open` | Present on the dialog when visible. |
| `data-closed` | Present during close animation. |
| `data-transition` | Present during any animation phase. |
| `data-enter` | Present during the enter animation. |
| `data-leave` | Present during the leave animation. |

## Starting with an Open Dialog

Use `Dialog.boot()` when a Dialog should be open when the application starts. Pass its result to `Update.foldChildInit` so the parent incorporates the Dialog Model, maps its Commands to the parent Message type, and handles its `Opened` OutMessage.

```
return Update.foldChildInit(Dialog.boot({ id: 'confirm' }), {
  toParentModel: dialog => ({ dialog }),
  toParentMessage: toGotDialogMessage,
  foldOutMessage: foldDialogOutMessage,
})
```

See [Folding Update with Update.foldChild](https://foldkit.dev/core/submodel#fold-child) for the general child-initialization pattern.

## Keyboard Interaction

| Key | Description |
| --- | --- |
| `Escape` | Closes the dialog. |
| `Tab` | Cycles focus within the dialog. |

## Accessibility

When the Dialog opens, Foldkit marks surrounding content inert and `aria-hidden="true"`, and sets `aria-modal="true"` on the Dialog. Keyboard focus stays inside until it closes, then the original inert and ARIA state is restored before focus returns to the element that opened it. The DevTools overlay remains interactive during development.

The dialog always sets `aria-labelledby` on the native element. Set `hasDescription: true` when you render a description to add `aria-describedby`; leaving it false prevents a dangling reference when no description is present. Spread `...title` onto your heading (`h.h2([...title], [...])`) and `...description` onto your description element (`h.p([...description], [...])`). You never construct either id yourself.

The ids are framework-managed (the `-dialog-title`, `-dialog-description`, and `-panel` suffixes on the configured id). Going through the render info keeps them unique for you. The `Dialog.titleId(model)` and `Dialog.descriptionId(model)` helpers return the same ids as plain strings for the cases where you need the id as a value outside `toView`, such as a Command that calls `getElementById` or a cross-element reference. As a backstop, the runtime warns on any duplicate id in the rendered tree in development.

## API Reference

### InitConfig

Configuration object passed to `Dialog.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the dialog instance. |
| `isAnimated` | `boolean` | `false` | Enables animation coordination for open/close animations. |
| `focusSelector` | `string` | — | CSS selector for the element that receives focus when the dialog opens. A selector-based override of the `initialFocus` marker, for an element whose id you do not own or a descendant selector. Takes precedence over `initialFocus`; with neither set, focus falls to the first focusable element. |

### boot

`Dialog.boot(config)` accepts the same configuration as `Dialog.init(config)` and returns `Update.ReturnWithOutMessage<Dialog.Model, Dialog.Message, Dialog.OutMessage>`. Its Model is open, its Commands contain `ShowDialog`, and its OutMessage is `Opened`.

### ViewConfig

Configuration object passed to `Dialog.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `model` | `Dialog.Model` | — | The dialog state from your parent Model. |
| `toParentMessage` | `(childMessage: Dialog.Message) => ParentMessage` | — | Wraps Dialog Messages in your parent Message type for Submodel delegation. |
| `toView` | `(render: RenderInfo) => Html` | — | Callback that receives the dialog, backdrop, panel, and closeButton attribute bundles plus a derived `isVisible` flag, and returns the composed layout. The consumer MUST render an `h.dialog(...)` element so the framework can open and close it. |
| `hasDescription` | `boolean` | `false` | Whether the dialog renders a description. Adds `aria-describedby` to the dialog when true. |

### RenderInfo

Payload delivered to the `toView` callback each render.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `dialog` | `ReadonlyArray<ChildAttribute>` | — | Spread onto an `h.dialog(...)` element. Carries the id, ARIA labelling and modal state, `open` prop, positioning style, lifecycle resource acquisition and cleanup, a handler that suppresses native `cancel` events so canceling a file picker leaves the dialog open, and a mapping from `Dom.showDialog` 's distinct Escape signal to `RequestedClose`. |
| `backdrop` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the backdrop element. Includes the Animation data attributes and the outside-click handler that dispatches `RequestedClose` (suppressed while a leave animation is in progress). |
| `panel` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the panel element. Includes the panel id (`${id}-panel`) and the Animation data attributes. |
| `title` | `ReadonlyArray<ChildAttribute>` | — | Spread onto your accessible-name heading (`h.h2([...title], [...])`). Carries the framework-managed id the dialog’s `aria-labelledby` points at, so labelling wires up without hand-rolling the id. |
| `description` | `ReadonlyArray<ChildAttribute>` | — | Spread onto your description element (`h.p([...description], [...])`). Carries the framework-managed id referenced when `hasDescription` is true. |
| `initialFocus` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the element that should receive focus when the dialog opens (`h.input([...initialFocus])`). A configured `focusSelector` takes precedence; to focus an element whose id you do not own, use `focusSelector`. |
| `closeButton` | `ReadonlyArray<ChildAttribute>` | — | Spread onto an in-panel close control such as a Cancel button. Carries the click handler that closes the dialog, so a plain dismiss needs no parent message, and `type="button"` so a close control inside a form does not submit it. |
| `isVisible` | `boolean` | — | Derived from `isOpen` and the Animation `transitionState`. Render the backdrop and panel only while this is true. |

### OutMessage

Messages emitted to the parent through the optional `outMessage` field. Match on the OutMessage in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `Opened` | `{}` | — | Emitted once the dialog has transitioned to open. Fires after `update` has processed `RequestedOpen` and `isOpen` reflects the new state. |
| `Closed` | `{}` | — | Emitted once the dialog has transitioned to closed. Programmatic `Dialog.close` on an already-closed model is a no-op that does not re-emit, as is calling close while a leave animation is already in progress. |
