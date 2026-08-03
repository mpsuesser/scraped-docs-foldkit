---
url: https://foldkit.dev/ui/textarea
title: "Textarea"
description: "A thin wrapper around the native textarea with ARIA linking and styling hooks."
access_date: 2026-08-03T19:45:20.723Z
current_date: 2026-08-03T19:45:20.723Z
---

## Textarea

## Overview

An accessible multi-line text input that links a label and description via ARIA attributes. Textarea is a stateless render helper: call it directly with a ViewConfig in your own view; no Model, update, or `h.submodel` wrapping. It exposes the same three attribute groups as Input (`textarea`, `label`, and `description`) plus a `rows` prop to control the visible height.

See it in an app

Check out how Textarea is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/textarea.ts).

## Examples

### Basic

The `toView` callback receives attribute groups for the label, description, and textarea element. Spread `attributes.textarea` onto a `<textarea>` in your layout to wire up ARIA, focus, and change handling.

A brief introduction about yourself.

```
// Pseudocode — Textarea is view-only. The value lives in your own Model as
// a string. Replace model.bio and UpdatedBio with your own field and Message.
import type { HtmlBuilder } from 'foldkit/html'

import { Textarea } from '@foldkit/ui'

const view = (model: Model, h: HtmlBuilder<Message>) =>
  Textarea.view(
    {
      id: 'bio',
      value: model.bio, // your Model field
      onInput: value => UpdatedBio({ value }), // your Message
      placeholder: 'Tell us about yourself...',
      rows: 4,
      toView: attributes =>
        h.div(
          [h.Class('flex flex-col gap-1.5')],
          [
            h.label(
              [...attributes.label, h.Class('text-sm font-medium')],
              ['Bio'],
            ),
            h.textarea([
              ...attributes.textarea,
              h.Class('w-full rounded-lg border border-gray-300 px-3 py-2'),
            ]),
            h.span(
              [...attributes.description, h.Class('text-sm text-gray-500')],
              ['A brief introduction about yourself.'],
            ),
          ],
        ),
    },
    h,
  )
```

### Disabled

Set `isDisabled: true` to disable the textarea. Like Input, this sets both the native `disabled` attribute and `aria-disabled`.

This textarea is disabled.

```
// Pseudocode — Textarea is view-only. Disabled textareas display a fixed
// value and ignore onInput events.
import type { HtmlBuilder } from 'foldkit/html'

import { Textarea } from '@foldkit/ui'

const view = (h: HtmlBuilder<Message>) =>
  Textarea.view(
    {
      id: 'bio-disabled',
      isDisabled: true,
      value: 'Known for work on the Analytical Engine.',
      rows: 3,
      toView: attributes =>
        h.div(
          [h.Class('flex flex-col gap-1.5')],
          [
            h.label(
              [...attributes.label, h.Class('text-sm font-medium')],
              ['Bio'],
            ),
            h.textarea([
              ...attributes.textarea,
              h.Class(
                'w-full rounded-lg border px-3 py-2 data-[disabled]:opacity-50',
              ),
            ]),
            h.span(
              [...attributes.description, h.Class('text-sm text-gray-500')],
              ['This textarea is disabled.'],
            ),
          ],
        ),
    },
    h,
  )
```

## Styling

Textarea is headless. Your `toView` callback controls all markup and styling. Use the data attributes below to style different states.

| Attribute | Condition |
| --- | --- |
| `data-disabled` | Present when isDisabled is true. |
| `data-invalid` | Present when isInvalid is true. |

## Keyboard Interaction

Textarea uses the native `<textarea>` element, so all keyboard interaction is handled by the browser.

| Key | Description |
| --- | --- |
| `Tab` | Moves focus to or away from the textarea. |

## Accessibility

Textarea provides the same ARIA wiring as Input. The `label` group links via `for`, and the `description` group is referenced by `aria-describedby` on the textarea. You can access the description ID directly with `Textarea.descriptionId(id)`.

When `isInvalid` is true, `aria-invalid="true"` is set on the textarea element.

## API Reference

### ViewConfig

Configuration object passed to `Textarea.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the textarea element. Used to link the label and description via ARIA attributes. |
| `toView` | `(attributes: TextareaAttributes) => Html` | — | Callback that receives attribute groups for the textarea, label, and description elements. |
| `onInput` | `(value: string) => Message` | — | Function that maps the current textarea value to a Message on each input event. |
| `value` | `string` | — | The current value of the textarea. |
| `isDisabled` | `boolean` | `false` | Whether the textarea is disabled. Sets both the native disabled attribute and aria-disabled. |
| `isInvalid` | `boolean` | `false` | Whether the textarea is in an invalid state. Sets aria-invalid and adds a data-invalid attribute for styling. |
| `isAutofocus` | `boolean` | `false` | Whether the textarea receives focus when the page loads. |
| `name` | `string` | — | The form field name for native form submission. |
| `rows` | `number` | — | The visible number of text lines. |
| `placeholder` | `string` | — | Placeholder text shown when the textarea is empty. |

### TextareaAttributes

Attribute groups provided to the `toView` callback.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `textarea` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the `<textarea>` element. Includes id, rows, value, ARIA attributes, and event handlers. |
| `label` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the `<label>` element. Includes a for attribute linking to the textarea id. |
| `description` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto a description element. Includes an id that the textarea references via aria-describedby. |
