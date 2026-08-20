---
url: https://foldkit.dev/ui/fieldset
title: "Fieldset"
description: "A stateless wrapper around the native fieldset with linked legend and description attributes."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

## Overview

A semantic form section that groups related controls with a legend and description. Fieldset is a stateless render helper that wraps the native `<fieldset>` element: call it directly with a ViewConfig in your own view; no Model, update, or `h.submodel` wrapping. When disabled, the browser propagates the disabled state to all child form controls automatically.

See it in an app

Check out how Fieldset is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/fieldset.ts).

## Examples

### Basic

The `toView` callback receives three attribute groups: `fieldset` for the wrapper, `legend` for the group title, and `description` for help text. Nest other Foldkit UI components inside the fieldset body.

```
// Pseudocode — Fieldset is view-only. Nest other Foldkit UI components
// inside the fieldset body in your own view function.
import type { HtmlBuilder } from 'foldkit/html'

import { Fieldset } from '@foldkit/ui'

const view = (h: HtmlBuilder<Message>) =>
  Fieldset.view(
    {
      id: 'personal-info',
      toView: attributes =>
        h.fieldset(
          [...attributes.fieldset, h.Class('rounded-lg border p-6')],
          [
            h.legend(
              [...attributes.legend, h.Class('text-base font-semibold')],
              ['Personal Information'],
            ),
            h.span(
              [
                ...attributes.description,
                h.Class('text-sm text-gray-500 mt-1'),
              ],
              ['We just need a few details.'],
            ),
            h.div(
              [h.Class('mt-4 flex flex-col gap-4')],
              [
                // Nest Input, Textarea, Checkbox, etc. here
              ],
            ),
          ],
        ),
    },
    h,
  )
```

### Disabled

Set `isDisabled: true` to disable the entire group. The native `<fieldset disabled>` attribute propagates to all child inputs, textareas, buttons, and selects. You don’t need to disable each control individually.

```
// Pseudocode — Fieldset is view-only. Setting isDisabled on the fieldset
// propagates to all child form elements via the native <fieldset disabled>
// attribute.
import type { HtmlBuilder } from 'foldkit/html'

import { Fieldset } from '@foldkit/ui'

const view = (h: HtmlBuilder<Message>) =>
  Fieldset.view(
    {
      id: 'personal-info-disabled',
      isDisabled: true,
      toView: attributes =>
        h.fieldset(
          [...attributes.fieldset, h.Class('rounded-lg border p-6')],
          [
            h.legend(
              [...attributes.legend, h.Class('text-base font-semibold')],
              ['Personal Information'],
            ),
            h.span(
              [
                ...attributes.description,
                h.Class('text-sm text-gray-500 mt-1'),
              ],
              ['This section is disabled.'],
            ),
            // All nested form controls inherit the disabled state
          ],
        ),
    },
    h,
  )
```

## Styling

Fieldset is headless. Your `toView` callback controls all markup and styling.

| Attribute | Condition |
| --- | --- |
| `data-disabled` | Present when isDisabled is true. |

## Accessibility

The `legend` attribute group includes an id (accessible via `Fieldset.legendId(id)`) and the `description` group includes an id (accessible via `Fieldset.descriptionId(id)`) that the fieldset references through `aria-describedby`.

## API Reference

### ViewConfig

Configuration object passed to `Fieldset.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the fieldset element. Used to generate linked IDs for legend and description. |
| `toView` | `(attributes: FieldsetAttributes) => Html` | — | Callback that receives attribute groups for the fieldset, legend, and description elements. |
| `isDisabled` | `boolean` | `false` | Whether the fieldset is disabled. The native disabled attribute on `<fieldset>` propagates to all child form controls. |

### FieldsetAttributes

Attribute groups provided to the `toView` callback.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `fieldset` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the `<fieldset>` element. Includes id, aria-describedby, and the disabled attribute when applicable. |
| `legend` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the `<legend>` element. Includes an id for programmatic reference. |
| `description` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto a description element. Includes an id that the fieldset references via aria-describedby. |
