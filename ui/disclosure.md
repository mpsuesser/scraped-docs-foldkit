---
url: https://foldkit.dev/ui/disclosure
title: "Disclosure"
description: "An accessible show/hide foundation for toggleable content sections."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

# Disclosure

## Overview

A toggle for showing and hiding content inline. Disclosure is a stateless controlled render helper: call it directly with a ViewConfig in your own view; no Model, update, or `h.submodel` wrapping. Your Model owns the open value, you pass it in as `isOpen`, and `onToggle` dispatches a Message when the user toggles it. In your update handler, just store the value. Use it for FAQs, accordions, and collapsible sections. For overlaying content in a floating panel, use Dialog or Popover instead.

See it in an app

Check out how Disclosure is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/disclosure.ts).

## Examples

Provide a `toView` callback that receives the `button` and `panel` attribute bundles. Spread them onto your own elements; Disclosure manages the ARIA linking and toggle behavior.

Frequently asked

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Schema as S } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Disclosure } from '@foldkit/ui'

// Store the open state as a plain boolean field in your Model:
const Model = S.Struct({
  isFaqOpen: S.Boolean,
  // ...your other fields
})

// In your init function, start it closed:
const init = () => [
  {
    isFaqOpen: false,
    // ...your other fields
  },
  [],
]

// A verb-first, past-tense Message carries the new open state:
const ToggledFaq = m('ToggledFaq', { isOpen: S.Boolean })

const Message = S.Union([ToggledFaq])

// Inside your update function's M.tagsExhaustive({...}), store the value.
// This is the moment to persist the open state, lazy-load panel content, or
// log analytics.
ToggledFaq: ({ isOpen }) => [evo(model, { isFaqOpen: () => isOpen }), []]

// Inside your view function, render the disclosure with Disclosure.view.
// Render the panel unconditionally and pass it through animatePanel: the
// panel stays mounted while collapsed, so the height transition animates the
// open and close. The toggle text below names the button. When the toggle is
// icon-only, give it a name with `ariaLabel`, or point `ariaLabelledBy` at a
// visible label element (target the toggle id with
// `Disclosure.buttonId('faq-1')` for a native `<label for>`). Either
// attribute is only emitted when provided, so the toggle never carries a
// dangling `aria-labelledby`.
const view = (model, h: HtmlBuilder<Message>) =>
  Disclosure.view(
    {
      id: 'faq-1',
      isOpen: model.isFaqOpen,
      onToggle: isOpen => ToggledFaq({ isOpen }),
      // ariaLabel: 'What is Foldkit?',
      toView: ({ button, panel, animatePanel }) =>
        h.div(
          [h.Class('border rounded-lg overflow-hidden')],
          [
            h.button(
              [
                ...button,
                h.Class('flex items-center justify-between w-full p-4'),
              ],
              [h.span([], ['What is Foldkit?'])],
            ),
            animatePanel(
              h.div(
                [...panel, h.Class('p-4 border-t')],
                [h.p([], ['A functional UI framework built on Effect-TS.'])],
              ),
            ),
          ],
        ),
    },
    h,
  )
```

The example renders the panel unconditionally and passes it through `animatePanel`, which wraps the content in a CSS-grid container that transitions its height, keeping the panel mounted while collapsed so there is something to animate. To skip the animation, gate the panel on `isOpen` with a keyed conditional insert instead.

## Styling

Use the `data-open` attribute to style the button and panel differently when open.

Attribute

Condition

`data-open`

Present on both button and panel when the disclosure is open.

`data-disabled`

Present on the button when isDisabled is true.

## Keyboard Interaction

Key

Description

`Enter`

Toggles the disclosure.

`Space`

Toggles the disclosure.

## Accessibility

The toggle button receives `aria-expanded` and `aria-controls` linking to the panel. Toggling is user-driven, so focus stays on the button the user activated; there is no focus Command to handle in update.

Give the toggle an accessible name when its content is not self-describing. For a visible label, wire a native `<label for>` that targets the toggle id with `Disclosure.buttonId(id)` rather than hardcoding the `-button` convention. The `for` association makes the toggle properly labeled: assistive technology announces it by the visible label text, and clicking the label opens the disclosure. That is why it is the recommended pattern.

Two ViewConfig fields cover the cases a `<label for>` does not. Pass `ariaLabel` for an icon-only toggle with no visible label, or `ariaLabelledBy` when the element that names the toggle is not a `<label>` you can point `for` at.

## API Reference

### ViewConfig

Configuration object passed to `Disclosure.view()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the disclosure instance. Used to derive the button and panel ids for ARIA linking.

`isOpen`

`boolean`

—

The current open state, read from your Model.

`aria-expanded`

, the

`data-open`

marker, and

`animatePanel`

derive from it.

`onToggle`

`(isOpen: boolean) => Message`

—

Maps the new open state to a Message when the user toggles the disclosure. Your update handler just stores the value.

`toView`

`(attributes: DisclosureAttributes) => Html`

—

Callback that receives the

`button`

and

`panel`

attribute bundles and returns the composed layout. The consumer reads

`isOpen`

from their own Model when they need to render conditionally on it.

`isDisabled`

`boolean`

`false`

When true, the button is not clickable, gets

`aria-disabled`

and a

`data-disabled`

attribute.

`ariaLabel`

`string`

—

Accessible name for the toggle button. Use for an icon-only trigger with no visible label. Applied as aria-label, and takes precedence over ariaLabelledBy.

`ariaLabelledBy`

`string`

—

Id of an external element that labels the toggle button, applied as aria-labelledby. Pair with a visible label element.

### DisclosureAttributes

Attribute bundles delivered to the `toView` callback each render.

Name

Type

Default

Description

`button`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto the toggle button element. Includes

`aria-expanded`

,

`aria-controls`

,

`tabindex`

, and the click + Enter/Space keyboard handlers.

`panel`

`ReadonlyArray<Attribute<Message>>`

—

Spread onto the panel element. Includes the panel id (

`${id}-panel`

) and a

`data-open`

attribute when open.

`animatePanel`

`(content: Html) => Html`

—

Wraps panel content in a CSS-grid container that animates height as the disclosure opens and closes. Render the panel unconditionally (rather than gating on isOpen) and pass it here; the panel stays mounted while collapsed so the height transition has something to animate. The collapsed content is marked aria-hidden.
