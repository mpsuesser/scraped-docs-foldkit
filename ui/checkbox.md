---
url: https://foldkit.dev/ui/checkbox
title: "Checkbox"
description: "Accessible checkbox with indeterminate state support."
access_date: 2026-08-03T19:01:53.147Z
current_date: 2026-08-03T19:01:53.147Z
---

# Checkbox

## Overview

A toggle with checked, unchecked, and indeterminate states. Checkbox is a stateless controlled render helper: call it directly with a ViewConfig in your own view; no Model, update, or `h.submodel` wrapping. Your Model owns the checked value, you pass it in as `isChecked`, and `onToggle` dispatches a Message when the user toggles it. In your update handler, just store the value. For an on/off toggle that represents an immediate action (like a light switch), use Switch instead.

See it in an app

Check out how Checkbox is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/checkbox.ts).

## Examples

### Basic

The checkbox element is typically a `<button>`. Spread `attributes.checkbox` onto it for role, ARIA state, and keyboard/click handlers. The label click handler also toggles the checkbox.

Accept terms and conditions

You agree to our Terms of Service and Privacy Policy.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Schema as S } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Checkbox } from '@foldkit/ui'

// Store the checked state as a plain boolean field in your Model:
const Model = S.Struct({
  acceptedTerms: S.Boolean,
  // ...your other fields
})

// In your init function, start it unchecked:
const init = () => [
  {
    acceptedTerms: false,
    // ...your other fields
  },
  [],
]

// A verb-first, past-tense Message carries the new checked state:
const ToggledTerms = m('ToggledTerms', { isChecked: S.Boolean })

const Message = S.Union([ToggledTerms])

// Inside your update function's M.tagsExhaustive({...}), store the value.
// This is the moment to fire analytics, validate a form, or push the value
// to a backend.
ToggledTerms: ({ isChecked }) => [
  evo(model, { acceptedTerms: () => isChecked }),
  [],
]

// Inside your view function, render the checkbox with Checkbox.view. It reads
// the checked state from your Model and calls onToggle with the new state.
const view = (model, h: HtmlBuilder<Message>) =>
  Checkbox.view(
    {
      id: 'accept-terms',
      isChecked: model.acceptedTerms,
      onToggle: isChecked => ToggledTerms({ isChecked }),
      toView: attributes =>
        h.div(
          [h.Class('flex flex-col gap-1')],
          [
            h.div(
              [h.Class('flex items-center gap-2')],
              [
                h.button(
                  [...attributes.checkbox, h.Class('h-5 w-5 rounded border')],
                  model.acceptedTerms ? ['✓'] : [],
                ),
                h.label(
                  [...attributes.label, h.Class('text-sm')],
                  ['Accept terms and conditions'],
                ),
              ],
            ),
            h.p(
              [...attributes.description, h.Class('text-sm text-gray-500')],
              ['You agree to our Terms of Service.'],
            ),
          ],
        ),
    },
    h,
  )
```

### Indeterminate

Pass `isIndeterminate: true` to show a mixed state. This is typically computed from child checkbox states: when some but not all children are checked, the parent shows the indeterminate mark. Toggling the parent sets all children to the same state.

All notifications

Email notifications

Push notifications

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Schema as S } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Checkbox } from '@foldkit/ui'

// Store each child's checked state as a plain boolean field in your Model:
const Model = S.Struct({
  optionA: S.Boolean,
  optionB: S.Boolean,
  // ...your other fields
})

// In your init function, start each unchecked:
const init = () => [
  {
    optionA: false,
    optionB: false,
    // ...your other fields
  },
  [],
]

// One Message per child, plus one for the "Select All" parent. Each carries
// the new checked state:
const ToggledSelectAll = m('ToggledSelectAll', { isChecked: S.Boolean })
const ToggledOptionA = m('ToggledOptionA', { isChecked: S.Boolean })
const ToggledOptionB = m('ToggledOptionB', { isChecked: S.Boolean })

const Message = S.Union([ToggledSelectAll, ToggledOptionA, ToggledOptionB])

// Inside your update function's M.tagsExhaustive({...}), toggling "Select All"
// writes the same value to every child:
ToggledSelectAll: ({ isChecked }) => [
  evo(model, {
    optionA: () => isChecked,
    optionB: () => isChecked,
  }),
  [],
]

// Inside your view function, compute the parent's checked and indeterminate
// state from the children and pass isIndeterminate straight to Checkbox.view:
const view = (model, h: HtmlBuilder<Message>) => {
  const isAllChecked = model.optionA && model.optionB
  const isNoneChecked = !model.optionA && !model.optionB
  const isIndeterminate = !isAllChecked && !isNoneChecked

  const resolveSelectAllMark = () => {
    if (isIndeterminate) {
      return ['—']
    } else if (isAllChecked) {
      return ['✓']
    } else {
      return []
    }
  }

  return Checkbox.view(
    {
      id: 'select-all',
      isChecked: isAllChecked,
      isIndeterminate,
      onToggle: isChecked => ToggledSelectAll({ isChecked }),
      toView: attributes =>
        h.div(
          [h.Class('flex items-center gap-2')],
          [
            h.button(
              [...attributes.checkbox, h.Class('h-5 w-5 rounded border')],
              resolveSelectAllMark(),
            ),
            h.label(
              [...attributes.label, h.Class('text-sm')],
              ['All notifications'],
            ),
          ],
        ),
    },
    h,
  )
}
```

## Styling

Checkbox is headless. Your `toView` callback controls all markup and styling. Use the data attributes below to style checked, indeterminate, and disabled states.

Attribute

Condition

`data-checked`

Present when checked and not indeterminate.

`data-indeterminate`

Present when isIndeterminate is true.

`data-disabled`

Present when isDisabled is true.

## Keyboard Interaction

Key

Description

`Space`

Toggles the checkbox.

## Accessibility

The checkbox element receives `role="checkbox"` and `aria-checked` which is set to `"true"`, `"false"`, or `"mixed"` depending on the checked and indeterminate state. The label is linked via `aria-labelledby` and the description via `aria-describedby`.

The `label` attribute group includes an id (accessible via `Checkbox.labelId(id)`) and the `description` group includes an id (accessible via `Checkbox.descriptionId(id)`), so a consumer can reference either element without re-declaring the naming convention.

## API Reference

### ViewConfig

Configuration object passed to `Checkbox.view()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the checkbox instance. Used to link the label and description via ARIA.

`isChecked`

`boolean`

—

The current checked state, read from your Model.

`aria-checked`

and the

`data-checked`

marker derive from it.

`onToggle`

`(isChecked: boolean) => Message`

—

Maps the new checked state to a Message when the user toggles the checkbox. Your update handler just stores the value.

`toView`

`(attributes: CheckboxAttributes) => Html`

—

Callback that receives attribute groups for the checkbox, label, description, and hidden input elements.

`isDisabled`

`boolean`

`false`

Whether the checkbox is disabled.

`isIndeterminate`

`boolean`

`false`

Whether to show the indeterminate (mixed) state. Useful for "select all" checkboxes where some but not all children are checked.

`name`

`string`

—

Form field name. When provided, a hidden input is included for native form submission.

`value`

`string`

`'on'`

Value sent in the form when checked.

### CheckboxAttributes

Attribute groups provided to the `toView` callback.

Name

Type

Default

Description

`checkbox`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto the checkbox element (typically a

`<button>`

). Includes role, aria-checked, tabindex, and click/keyboard handlers.

`label`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto the label element. Includes an id for aria-labelledby and a click handler that toggles the checkbox.

`description`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto a description element. Includes an id referenced by aria-describedby on the checkbox.

`hiddenInput`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto a hidden

`<input>`

for form submission. Only needed when the name prop is set.
