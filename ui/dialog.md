---
url: https://foldkit.dev/ui/dialog
title: "Dialog"
description: "A modal dialog backed by the native dialog element with focus trapping and scroll locking."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

# Dialog

## Overview

A modal dialog backed by the native `<dialog>` element, opened with `show()` and a high z-index. The framework manages focus trapping, Escape handling, scroll locking, and backdrop rendering. For non-modal floating content, use Popover instead.

See it in an app

Check out how Dialog is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/dialog.ts).

## Examples

### Basic

Open the dialog from a trigger by dispatching your own Message and calling `Dialog.open(model)` in your update. Spread the `closeButton` bundle onto a Cancel button to dismiss it, or call `Dialog.close(model)` directly. Both return `[Model, Commands, Option<OutMessage>]`. Spread `...title` onto a heading element so the dialog is labeled for screen readers.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Dialog } from '@foldkit/ui'

// Add a field to your Model for the Dialog Submodel:
const Model = S.Struct({
  dialog: Dialog.Model,
  // ...your other fields
})

// In your init function, initialize the Dialog Submodel with a unique id:
const init = () => [
  {
    dialog: Dialog.init({ id: 'confirm' }),
    // ...your other fields
  },
  [],
]

// A fact for the trigger, plus the Dialog Message embedded in your parent
// Message for the submodel delegation:
const ClickedOpenDialog = m('ClickedOpenDialog')
const GotDialogMessage = m('GotDialogMessage', {
  message: Dialog.Message,
})

// Open the dialog from your update with Dialog.open. Escape, the backdrop,
// and the closeButton bundle all flow back through GotDialogMessage, where you
// delegate to Dialog.update. (Both return an Option<OutMessage> as the
// third element; match Opened/Closed there to react to the transitions.)
ClickedOpenDialog: () => {
  const [nextDialog, dialogCommands] = Dialog.open(model.dialog)
  return [
    evo(model, { dialog: () => nextDialog }),
    Command.mapMessages(dialogCommands, message =>
      GotDialogMessage({ message }),
    ),
  ]
}

GotDialogMessage: ({ message }) => {
  const [nextDialog, dialogCommands] = Dialog.update(model.dialog, message)
  return [
    evo(model, { dialog: () => nextDialog }),
    Command.mapMessages(dialogCommands, message =>
      GotDialogMessage({ message }),
    ),
  ]
}

// In your view, open from a trigger with the fact, and dismiss from a Cancel
// button by spreading the `closeButton` bundle, no parent message needed:
const view = (h: HtmlBuilder<Message>) =>
  h.div(
    [],
    [
      h.button([h.OnClick(ClickedOpenDialog())], ['Open Dialog']),
      h.submodel({
        slotId: model.dialog.id,
        model: model.dialog,
        view: Dialog.view,
        viewInputs: {
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
        toParentMessage: message => GotDialogMessage({ message }),
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
import { Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Dialog } from '@foldkit/ui'

// Add a field to your Model for the Dialog Submodel:
const Model = S.Struct({
  dialog: Dialog.Model,
  // ...your other fields
})

// In your init function, set isAnimated: true to coordinate CSS transitions:
const init = () => [
  {
    dialog: Dialog.init({ id: 'confirm', isAnimated: true }),
    // ...your other fields
  },
  [],
]

// Embed the Dialog Message in your parent Message and delegate to
// Dialog.update (open from a trigger with a fact and Dialog.open, as in
// the basic Dialog example):
const GotDialogMessage = m('GotDialogMessage', {
  message: Dialog.Message,
})

GotDialogMessage: ({ message }) => {
  const [nextDialog, dialogCommands] = Dialog.update(model.dialog, message)
  return [
    evo(model, { dialog: () => nextDialog }),
    Command.mapMessages(dialogCommands, message =>
      GotDialogMessage({ message }),
    ),
  ]
}

// Inside your view function, use data-[closed] for enter/leave transitions and
// spread the `closeButton` bundle onto your dismiss buttons:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: model.dialog.id,
    model: model.dialog,
    view: Dialog.view,
    viewInputs: {
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
    toParentMessage: message => GotDialogMessage({ message }),
  })
```

### Field

A field inside a dialog can open its own overlay, like a Combobox or DatePicker. By default that overlay portals its panel to the document body, where the dialog renders on top of it. Pass `anchor: { portal: false }` so the panel stays inside the dialog and remains visible.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Option } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'

import { Combobox, Dialog } from '@foldkit/ui'

// One Model field for the dialog, one for the overlay it contains, plus
// the parent-owned selection (`City` and `CityCombobox` are the
// `S.Literals` Schema and typed factory from the Combobox example):
const Model = S.Struct({
  dialog: Dialog.Model,
  combobox: Combobox.Model,
  maybeCity: S.Option(City),
  // ...your other fields
})

const init = () => [
  {
    dialog: Dialog.init({ id: 'edit-filters' }),
    combobox: Combobox.init({ id: 'city' }),
    maybeCity: Option.none(),
    // ...your other fields
  },
  [],
]

// Embed each submodel's Message in your parent Message and delegate both to
// their own update (see the Dialog and Combobox examples for the delegation).
const GotDialogMessage = m('GotDialogMessage', { message: Dialog.Message })
const GotComboboxMessage = m('GotComboboxMessage', {
  message: Combobox.Message,
})

// Render the overlay inside the dialog panel. The key is `portal: false` on
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
                        GotComboboxMessage({ message }),
                    }),
                  ],
                ),
              ]
            : [],
        ),
    },
    toParentMessage: message => GotDialogMessage({ message }),
  })
```

### Stacked

Use a separate Dialog Model for each level and open the second from a button in the first. The framework stacks them by z-index, traps focus in the topmost, and closes them one at a time: Escape closes the top dialog before the one beneath it.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Dialog } from '@foldkit/ui'

// One Model field per dialog level:
const Model = S.Struct({
  settingsDialog: Dialog.Model,
  confirmDialog: Dialog.Model,
  // ...your other fields
})

const init = () => [
  {
    settingsDialog: Dialog.init({ id: 'settings' }),
    confirmDialog: Dialog.init({ id: 'confirm-delete' }),
    // ...your other fields
  },
  [],
]

// Embed each Dialog Message in your parent Message and delegate each to its
// own Dialog.update (see the basic Dialog example for the delegation).
const GotSettingsDialogMessage = m('GotSettingsDialogMessage', {
  message: Dialog.Message,
})
const GotConfirmDialogMessage = m('GotConfirmDialogMessage', {
  message: Dialog.Message,
})

// Opening the confirmation is a parent fact, not a hand-wrapped child message.
// The button dispatches ClickedDeleteProject; the update opens the confirmation
// through Dialog.open, keeping Got* for genuine child results.
const ClickedDeleteProject = m('ClickedDeleteProject')
const ConfirmedDeleteProject = m('ConfirmedDeleteProject')

// ...in your update's M.tagsExhaustive({...}):
ClickedDeleteProject: () => {
  const [nextConfirmDialog, confirmDialogCommands] = Dialog.open(
    model.confirmDialog,
  )
  return [
    evo(model, { confirmDialog: () => nextConfirmDialog }),
    Command.mapMessages(confirmDialogCommands, message =>
      GotConfirmDialogMessage({ message }),
    ),
  ]
}

// Confirming runs the deletion, then closes the confirmation through
// Dialog.close, the same API the opening fact used.
ConfirmedDeleteProject: () => {
  // ...run the deletion here, then:
  const [nextConfirmDialog, confirmDialogCommands] = Dialog.close(
    model.confirmDialog,
  )
  return [
    evo(model, { confirmDialog: () => nextConfirmDialog }),
    Command.mapMessages(confirmDialogCommands, message =>
      GotConfirmDialogMessage({ message }),
    ),
  ]
}

// Each dialog is its own submodel; the framework stacks them by z-index, traps
// focus in the topmost, and Escape closes the topmost before the one beneath
// it. Cancel dismisses the confirmation by spreading the `closeButton` bundle; Delete
// dispatches a fact that runs the work and closes through Dialog.close.
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
                    h.button([h.OnClick(ConfirmedDeleteProject())], ['Delete']),
                  ],
                ),
              ]
            : [],
        ),
    },
    toParentMessage: message => GotConfirmDialogMessage({ message }),
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
                      [h.OnClick(ClickedDeleteProject())],
                      ['Delete project'],
                    ),
                  ],
                ),
              ]
            : [],
        ),
    },
    toParentMessage: message => GotSettingsDialogMessage({ message }),
  })

  return h.div([], [settingsDialog, confirmDialog])
}
```

## Styling

Dialog is headless. The `toView` callback receives attribute bundles for the dialog, backdrop, panel, and closeButton, and the consumer composes the markup. Dialog renders no backdrop of its own, so build your own from the `backdrop` bundle for full control over its appearance.

When `isAnimated` is true, enter/leave animations flow through the [Animation](https://foldkit.dev/ui/animation) module. Style with CSS transitions or CSS keyframe animations. Animation advances once every animation on the element has settled.

Attribute

Condition

`data-open`

Present on the dialog when visible.

`data-closed`

Present during close animation.

`data-transition`

Present during any animation phase.

`data-enter`

Present during the enter animation.

`data-leave`

Present during the leave animation.

## Keyboard Interaction

Key

Description

`Escape`

Closes the dialog.

`Tab`

Cycles focus within the dialog.

## Accessibility

The dialog sets `aria-labelledby` and `aria-describedby` on the native element and hands you the matching ids through the render info. Spread `...title` onto your heading (`h.h2([...title], [...])`) and `...description` onto your description element (`h.p([...description], [...])`). You never construct the id yourself. Focus trapping is handled by the framework.

The ids are framework-managed (the `-dialog-title`, `-dialog-description`, and `-panel` suffixes on the configured id). Going through the render info keeps them unique for you. The `Dialog.titleId(model)` and `Dialog.descriptionId(model)` helpers return the same ids as plain strings for the cases where you need the id as a value outside `toView`, such as a Command that calls `getElementById` or a cross-element reference. As a backstop, the runtime warns on any duplicate id in the rendered tree in development.

## API Reference

### InitConfig

Configuration object passed to `Dialog.init()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the dialog instance.

`isOpen`

`boolean`

`false`

Initial open/closed state.

`isAnimated`

`boolean`

`false`

Enables animation coordination for open/close animations.

`focusSelector`

`string`

—

CSS selector for the element that receives focus when the dialog opens. A selector-based override of the

`initialFocus`

marker, for an element whose id you do not own or a descendant selector. Takes precedence over

`initialFocus`

; with neither set, focus falls to the first focusable element.

### ViewConfig

Configuration object passed to `Dialog.view()`.

Name

Type

Default

Description

`model`

`Dialog.Model`

—

The dialog state from your parent Model.

`toParentMessage`

`(childMessage: Dialog.Message) => ParentMessage`

—

Wraps Dialog Messages in your parent Message type for Submodel delegation.

`toView`

`(render: RenderInfo) => Html`

—

Callback that receives the dialog, backdrop, panel, and closeButton attribute bundles plus a derived

`isVisible`

flag, and returns the composed layout. The consumer MUST render an

`h.dialog(...)`

element so the framework can open and close it.

### RenderInfo

Payload delivered to the `toView` callback each render.

Name

Type

Default

Description

`dialog`

`ReadonlyArray<ChildAttribute>`

—

Spread onto an

`h.dialog(...)`

element. Carries the id, ARIA labelling,

`open`

prop, positioning style, and the Escape handler that wires to

`RequestedClose`

.

`backdrop`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the backdrop element. Includes the Animation data attributes and the outside-click handler that dispatches

`RequestedClose`

(suppressed while a leave animation is in progress).

`panel`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the panel element. Includes the panel id (

`${id}-panel`

) and the Animation data attributes.

`title`

`ReadonlyArray<ChildAttribute>`

—

Spread onto your accessible-name heading (

`h.h2([...title], [...])`

). Carries the framework-managed id the dialog’s

`aria-labelledby`

points at, so labelling wires up without hand-rolling the id.

`description`

`ReadonlyArray<ChildAttribute>`

—

Spread onto your description element (

`h.p([...description], [...])`

). Carries the framework-managed id the dialog’s

`aria-describedby`

points at, so the association wires up without hand-rolling the id.

`initialFocus`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the element that should receive focus when the dialog opens (

`h.input([...initialFocus])`

). A configured

`focusSelector`

takes precedence; to focus an element whose id you do not own, use

`focusSelector`

.

`closeButton`

`ReadonlyArray<ChildAttribute>`

—

Spread onto an in-panel close control such as a Cancel button. Carries the click handler that closes the dialog, so a plain dismiss needs no parent message.

`isVisible`

`boolean`

—

Derived from

`isOpen`

and the Animation

`transitionState`

. Render the backdrop and panel only while this is true.

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Pattern-match on the OutMessage in your update handler.

Name

Type

Default

Description

`Opened`

`{}`

—

Emitted once the dialog has transitioned to open. Fires after

`update`

has processed

`RequestedOpen`

and

`isOpen`

reflects the new state.

`Closed`

`{}`

—

Emitted once the dialog has transitioned to closed. Programmatic

`Dialog.close`

on an already-closed model is a no-op that does not re-emit, as is calling close while a leave animation is already in progress.
