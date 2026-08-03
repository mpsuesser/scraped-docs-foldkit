---
url: https://foldkit.dev/ui/button
title: "Button"
description: "A thin wrapper around the native button with accessibility attributes and styling hooks."
access_date: 2026-08-03T19:01:53.147Z
current_date: 2026-08-03T19:01:53.147Z
---

# Button

## Overview

A thin wrapper around the native button element that provides consistent accessibility attributes and data-attribute hooks for styling. Button is a stateless render helper: call it directly with a ViewConfig in your own view. No Model, Messages, update, or `h.submodel` wrapping.

See it in an app

Check out how Button is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/button.ts).

## Examples

### Basic

Pass an `onClick` Message and a `toView` callback that spreads the provided attributes onto a `<button>` element.

Clicked 0 times

```
// Pseudocode — Button is view-only. Replace ClickedSave() with your own
// Message constructor.
import type { HtmlBuilder } from 'foldkit/html'

import { Button } from '@foldkit/ui'

const view = (h: HtmlBuilder<Message>) =>
  Button.view(
    {
      onClick: ClickedSave(), // your Message
      toView: attributes =>
        h.button(
          [
            ...attributes.button,
            h.Class('px-4 py-2 rounded-lg bg-blue-600 text-white'),
          ],
          ['Save'],
        ),
    },
    h,
  )
```

### Disabled

Set `isDisabled: true` to disable the button. Foldkit uses `aria-disabled` instead of the native `disabled` attribute so the button remains focusable for screen readers.

```
// Pseudocode — Button is view-only. A disabled button still needs an
// onClick Message for the view config; it just won't fire.
import type { HtmlBuilder } from 'foldkit/html'

import { Button } from '@foldkit/ui'

const view = (h: HtmlBuilder<Message>) =>
  Button.view(
    {
      isDisabled: true,
      toView: attributes =>
        h.button(
          [
            ...attributes.button,
            h.Class('px-4 py-2 rounded-lg bg-gray-300 text-gray-500'),
          ],
          ['Disabled'],
        ),
    },
    h,
  )
```

## Styling

Button is headless. It provides no default styles. Your `toView` callback receives attribute groups to spread onto the element, and you control all markup and styling.

Use the following data attributes to style different states:

Attribute

Condition

`data-disabled`

Present when

`isDisabled`

is true.

## Keyboard Interaction

Button uses the native `<button>` element, so keyboard interaction is handled by the browser.

Key

Description

`Enter`

Activates the button.

`Space`

Activates the button.

## Accessibility

Button sets `aria-disabled="true"` when disabled instead of the native `disabled` attribute. This ensures the button remains in the tab order and is announced by screen readers, while preventing click handlers from firing.

`tabindex="0"` is always set to ensure focusability. The `type` attribute defaults to `"button"` to prevent accidental form submissions.

## API Reference

### ViewConfig

Configuration object passed to `Button.view()`.

Name

Type

Default

Description

`toView`

`(attributes: ButtonAttributes) => Html`

—

Callback that receives attribute groups and returns the button markup.

`onClick`

`Message`

—

Message to dispatch when the button is clicked.

`isDisabled`

`boolean`

`false`

Whether the button is disabled. Uses

`aria-disabled`

instead of the

`disabled`

attribute to preserve focusability.

`type`

`'button' | 'submit' | 'reset'`

`'button'`

The HTML button type attribute.

`isAutofocus`

`boolean`

`false`

Whether the button receives focus when the page loads.

### ButtonAttributes

Attribute groups provided to the `toView` callback.

Name

Type

Default

Description

`button`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto the

`<button>`

element. Includes type, tabindex, ARIA attributes, and event handlers.
