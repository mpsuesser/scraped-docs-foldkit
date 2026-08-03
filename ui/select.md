---
url: https://foldkit.dev/ui/select
title: "Select"
description: "A thin wrapper around the native select with ARIA linking and styling hooks."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

# Select

## Overview

A wrapper around the native `<select>` element with ARIA label/description linking and data-attribute hooks. Select is a stateless render helper: call it directly with a ViewConfig in your own view; no Model, update, or `h.submodel` wrapping. For a custom dropdown with keyboard navigation and custom rendering, use Listbox or Combobox instead.

See it in an app

Check out how Select is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/select.ts).

## Examples

### Basic

Pass an `onChange` handler that receives the selected option’s value as a string. You provide the `<option>` elements inside the `<select>` in your `toView` callback.

Country

United States

Canada

United Kingdom

Australia

Where you currently reside.

```
// Pseudocode — Select is view-only. The selected value lives in your own
// Model as a string. Replace model.country and UpdatedCountry with your
// own field and Message.
import type { HtmlBuilder } from 'foldkit/html'

import { Select } from '@foldkit/ui'

const view = (h: HtmlBuilder<Message>) =>
  Select.view(
    {
      id: 'country',
      value: model.country, // your Model field
      onChange: value => UpdatedCountry({ value }), // your Message
      toView: attributes =>
        h.div(
          [h.Class('flex flex-col gap-1.5')],
          [
            h.label(
              [...attributes.label, h.Class('text-sm font-medium')],
              ['Country'],
            ),
            h.select(
              [
                ...attributes.select,
                h.Class('w-full rounded-lg border px-3 py-2'),
              ],
              [
                h.option([h.Value('us')], ['United States']),
                h.option([h.Value('ca')], ['Canada']),
                h.option([h.Value('gb')], ['United Kingdom']),
              ],
            ),
            h.span(
              [...attributes.description, h.Class('text-sm text-gray-500')],
              ['Where you currently reside.'],
            ),
          ],
        ),
    },
    h,
  )
```

### Disabled

Set `isDisabled: true` to disable the select.

Country

United States

Canada

United Kingdom

Australia

This select is disabled.

```
// Pseudocode — Select is view-only. Disabled selects display a fixed value
// and ignore onChange events.
import type { HtmlBuilder } from 'foldkit/html'

import { Select } from '@foldkit/ui'

const view = (h: HtmlBuilder<Message>) =>
  Select.view(
    {
      id: 'country-disabled',
      isDisabled: true,
      value: 'us',
      toView: attributes =>
        h.div(
          [h.Class('flex flex-col gap-1.5')],
          [
            h.label(
              [...attributes.label, h.Class('text-sm font-medium')],
              ['Country'],
            ),
            h.select(
              [
                ...attributes.select,
                h.Class(
                  'w-full rounded-lg border px-3 py-2 data-[disabled]:opacity-50',
                ),
              ],
              [
                h.option([h.Value('us')], ['United States']),
                h.option([h.Value('ca')], ['Canada']),
              ],
            ),
            h.span(
              [...attributes.description, h.Class('text-sm text-gray-500')],
              ['This select is disabled.'],
            ),
          ],
        ),
    },
    h,
  )
```

## Styling

Select is headless. Your `toView` callback controls all markup and styling. The native `<select>` dropdown appearance varies by browser and OS. Use `appearance-none` in CSS and add a custom chevron icon for a consistent look.

Attribute

Condition

`data-disabled`

Present when isDisabled is true.

`data-invalid`

Present when isInvalid is true.

## Keyboard Interaction

Select uses the native `<select>` element, so keyboard interaction is handled by the browser.

Key

Description

`Space`

Opens the native dropdown.

`Enter`

Opens the native dropdown.

`Arrow Up/Down`

Navigates between options.

## Accessibility

Select provides the same ARIA wiring as Input. The `label` group links via `for`, and the `description` group is referenced by `aria-describedby`. You can access the description ID directly with `Select.descriptionId(id)`.

## API Reference

### ViewConfig

Configuration object passed to `Select.view()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the select element. Used to link the label and description via ARIA attributes.

`toView`

`(attributes: SelectAttributes) => Html`

—

Callback that receives attribute groups for the select, label, and description elements.

`onChange`

`(value: string) => Message`

—

Function that maps the selected value to a Message when the selection changes.

`value`

`string`

—

The currently selected value.

`isDisabled`

`boolean`

`false`

Whether the select is disabled. Sets both the native disabled attribute and aria-disabled.

`isInvalid`

`boolean`

`false`

Whether the select is in an invalid state. Sets aria-invalid and adds a data-invalid attribute for styling.

`isAutofocus`

`boolean`

`false`

Whether the select receives focus when the page loads.

`name`

`string`

—

The form field name for native form submission.

### SelectAttributes

Attribute groups provided to the `toView` callback.

Name

Type

Default

Description

`select`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto the

`<select>`

element. Includes id, value, ARIA attributes, and event handlers.

`label`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto the

`<label>`

element. Includes a for attribute linking to the select id.

`description`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto a description element. Includes an id that the select references via aria-describedby.
