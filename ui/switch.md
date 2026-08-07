---
url: https://foldkit.dev/ui/switch
title: "Switch"
description: "Accessible toggle switch for boolean settings."
access_date: 2026-08-07T15:31:08.863Z
current_date: 2026-08-07T15:31:08.863Z
---

## Switch

## Overview

An on/off toggle. Semantically different from Checkbox: Switch represents an immediate action (like a light switch), while Checkbox represents a form value that gets submitted. Switch is a stateless controlled render helper with the same wiring as Checkbox: your Model owns the on/off value, you pass it in as `isChecked`, and `onToggle` dispatches a Message when the user toggles it. In your update handler, just store the value.

See it in an app

Check out how Switch is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/switch.ts).

## Examples

The switch renders as a `<button>` with `role="switch"`. The typical visual is a track with a sliding knob, styled with the `data-checked` attribute for the on state.

Get notified when something important happens.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Schema as S } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Switch } from '@foldkit/ui'

// Store the on/off state as a plain boolean field in your Model:
const Model = S.Struct({
  notificationsEnabled: S.Boolean,
  // ...your other fields
})

// In your init function, start it off:
const init = () => [
  {
    notificationsEnabled: false,
    // ...your other fields
  },
  [],
]

// A verb-first, past-tense Message carries the new checked state:
const ToggledNotifications = m('ToggledNotifications', {
  isChecked: S.Boolean,
})

const Message = S.Union([ToggledNotifications])

// Inside your update function's M.tagsExhaustive({...}), store the value.
// This is the moment to persist the preference, sync to a backend, or fire
// analytics.
ToggledNotifications: ({ isChecked }) => [
  evo(model, { notificationsEnabled: () => isChecked }),
  [],
]

// Inside your view function, render the switch with Switch.view. It reads the
// checked state from your Model and calls onToggle with the new state. The
// track color keys off the data-checked attribute; the knob position derives
// from the same Model field.
const view = (model, h: HtmlBuilder<Message>) =>
  Switch.view(
    {
      id: 'notifications',
      isChecked: model.notificationsEnabled,
      onToggle: isChecked => ToggledNotifications({ isChecked }),
      toView: attributes =>
        h.div(
          [h.Class('flex items-center gap-3')],
          [
            h.button(
              [
                ...attributes.button,
                h.Class(
                  'relative inline-flex h-6 w-11 items-center rounded-full transition-colors data-[checked]:bg-blue-600 bg-gray-200',
                ),
              ],
              [
                h.span([
                  h.Class(
                    \`inline-block h-4 w-4 rounded-full bg-white shadow transition-transform ${model.notificationsEnabled ? 'translate-x-6' : 'translate-x-1'}\`,
                  ),
                ]),
              ],
            ),
            h.div(
              [],
              [
                h.label(
                  [...attributes.label, h.Class('text-sm font-medium')],
                  ['Enable notifications'],
                ),
                h.p(
                  [...attributes.description, h.Class('text-sm text-gray-500')],
                  ['Get notified when something important happens.'],
                ),
              ],
            ),
          ],
        ),
    },
    h,
  )
```

## Styling

Switch is headless. Your `toView` callback controls all markup and styling. Use `data-[checked]` to change the track color and translate the knob, and the data attributes below to style the disabled and read-only states.

| Attribute | Condition |
| --- | --- |
| `data-checked` | Present when the switch is on. |
| `data-disabled` | Present when isDisabled is true. |
| `data-readonly` | Present when isReadOnly is true. |

## Keyboard Interaction

| Key | Description |
| --- | --- |
| `Space` | Toggles the Switch when `isDisabled` and `isReadOnly` are both `false`. |

## Accessibility

The switch button receives `role="switch"` and `aria-checked`. The label is linked via `aria-labelledby` and the description via `aria-describedby`. Clicking the label toggles the switch.

The `label` attribute group includes an id (accessible via `Switch.labelId(id)`) and the `description` group includes an id (accessible via `Switch.descriptionId(id)`), so a consumer can reference either element without re-declaring the naming convention.

`isReadOnly` and `isDisabled` both stop the Switch from reacting to clicks and Space. They differ in the semantics exposed to assistive technology, so they are not interchangeable.

`aria-disabled="true"`, which `isDisabled` emits, communicates that the Switch is unavailable. `aria-readonly="true"`, which `isReadOnly` emits, communicates that its value cannot be changed but remains relevant to the user. Both states keep `tabindex="0"`, following Foldkit's convention that unavailable controls remain discoverable by keyboard and assistive technology.

Assistive technology support for `aria-readonly` on switches varies. Pair it with a visible read-only treatment or explanatory text when users must distinguish it from disabled, and test the browser and assistive technology combinations your app supports.

Use `isReadOnly` when the on/off state is still information the user needs, such as a setting controlled elsewhere, and `isDisabled` when the Switch is unavailable.

The two flags are independent. Setting both emits both sets of attributes, and either one on its own removes the click and Space handlers.

## API Reference

### ViewConfig

Configuration object passed to `Switch.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the switch instance. Used to link the label and description via ARIA. |
| `isChecked` | `boolean` | — | The current on/off state, read from your Model. `aria-checked` and the `data-checked` marker derive from it. |
| `onToggle` | `(isChecked: boolean) => Message` | — | Maps the new on/off state to a Message when the user toggles the switch. Your update handler just stores the value. |
| `toView` | `(attributes: SwitchAttributes) => Html` | — | Callback that receives attribute groups for the button, label, description, and hidden input elements. |
| `isDisabled` | `boolean` | `false` | Whether the switch is disabled. |
| `isReadOnly` | `boolean` | `false` | Whether the switch is readable but not toggleable. Carries `aria-readonly` rather than `aria-disabled`. Independent of `isDisabled`. |
| `name` | `string` | — | Form field name. When provided, a hidden input is included for native form submission. |
| `value` | `string` | `'on'` | Value sent in the form when checked. |

### SwitchAttributes

Attribute groups provided to the `toView` callback.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `button` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the switch button element. Includes role, aria-checked, tabindex, click/keyboard handlers, and `type="button"` so a switch inside a form does not submit it. |
| `label` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the label element. Includes an id for aria-labelledby and a click handler that toggles the switch. |
| `description` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto a description element. Includes an id referenced by aria-describedby on the switch. |
| `hiddenInput` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto a hidden `<input>` for form submission. Only needed when the name prop is set. |
