---
url: https://foldkit.dev/ui/popover
title: "Popover"
description: "Floating content panels anchored to trigger elements."
access_date: 2026-08-03T17:27:13.509Z
current_date: 2026-08-03T17:27:13.509Z
---

# Popover

## Overview

An anchored floating panel with natural Tab navigation. Unlike Dialog (which is modal and traps focus) or Menu (which uses aria-activedescendant for item navigation), Popover holds arbitrary content and uses the disclosure ARIA pattern. Focus flows naturally through the panel content.

For programmatic control in update functions, use `Popover.open(model)` and `Popover.close(model)` which return `[Model, Commands, Option<OutMessage>]` directly.

See it in an app

Check out how Popover is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/popover.ts).

## Examples

### Basic

Pass `anchor` to position the panel relative to the button. The panel can hold any content: links, forms, or informational text.

Product menu

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Match as M, Option } from 'effect'
import { Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Popover } from '@foldkit/ui'

// Add a field to your Model for the Popover Submodel:
const Model = S.Struct({
  popover: Popover.Model,
  // ...your other fields
})

// In your init function, initialize the Popover Submodel with a unique id:
const init = () => [
  {
    popover: Popover.init({ id: 'info' }),
    // ...your other fields
  },
  [],
]

// Embed the Popover Message in your parent Message:
const GotPopoverMessage = m('GotPopoverMessage', {
  message: Popover.Message,
})

// Inside your update function's M.tagsExhaustive({...}), delegate to
// Popover.update. The OutMessages `Opened` and `Closed` mark the
// visibility transitions. Fire analytics, coordinate with other UI,
// or clear ephemeral state on close.
GotPopoverMessage: ({ message }) => {
  const [nextPopover, commands, maybeOutMessage] = Popover.update(
    model.popover,
    message,
  )
  const mappedCommands = Command.mapMessages(commands, message =>
    GotPopoverMessage({ message }),
  )

  return Option.match(maybeOutMessage, {
    onNone: () => [evo(model, { popover: () => nextPopover }), mappedCommands],
    onSome: M.type<Popover.OutMessage>().pipe(
      M.tagsExhaustive({
        Opened: () => [
          // The child has emitted `Opened`. The body commits the
          // child's next state as usual. In this arm the parent can
          // also update its own state or dispatch its own Commands,
          // for example lazy-load panel content, log analytics, or
          // trigger a downstream Command.
          evo(model, { popover: () => nextPopover }),
          mappedCommands,
        ],
        Closed: () => [
          // The child has emitted `Closed`. The body commits the
          // child's next state as usual. In this arm the parent can
          // also update its own state or dispatch its own Commands,
          // for example persist a draft, clear ephemeral state, or
          // trigger a downstream Command.
          evo(model, { popover: () => nextPopover }),
          mappedCommands,
        ],
      }),
    ),
  })
}

// Inside your view function, embed the popover via h.submodel. Give the
// trigger an accessible name: target the trigger id with
// `Popover.buttonId('info')` from a native `<label for>`, and pass
// `ariaLabelledBy` so the trigger is named by the label. The attribute is
// only emitted when provided, so the trigger never carries a dangling
// `aria-labelledby`.
const view = (h: HtmlBuilder<Message>) => {
  const labelId = 'info-label'

  return h.submodel({
    slotId: 'info',
    model: model.popover,
    view: Popover.view,
    viewInputs: {
      ariaLabelledBy: labelId,
      anchor: { placement: 'bottom-start', gap: 4, padding: 8 },
      toView: ({ button, panel, backdrop, isVisible }) =>
        h.div(
          [h.Class('relative inline-block')],
          [
            h.label(
              [h.Id(labelId), h.For(Popover.buttonId('info'))],
              ['Solutions'],
            ),
            h.button(
              [
                ...button,
                h.Class('rounded-lg border px-3 py-2 cursor-pointer'),
              ],
              [h.span([], ['Solutions'])],
            ),
            ...(isVisible
              ? [
                  h.div([...backdrop, h.Class('fixed inset-0')]),
                  h.div(
                    [...panel, h.Class('rounded-lg border shadow-lg p-4 w-80')],
                    [
                      h.h3([h.Class('font-medium')], ['Analytics']),
                      h.p(
                        [h.Class('text-sm text-gray-500')],
                        [
                          'Get a better understanding of where your traffic is coming from.',
                        ],
                      ),
                    ],
                  ),
                ]
              : []),
          ],
        ),
    },
    toParentMessage: message => GotPopoverMessage({ message }),
  })
}
```

### Animated

Pass `isAnimated: true` at init for animation coordination.

Product menu

### Nested

Use a separate Popover Model for each level. For a parent panel that opens onto another Popover trigger, pass `contentFocus: true` at init and `focusSelector` in the view so focus lands on the nested trigger.

Account

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Popover } from '@foldkit/ui'

// Add one Popover Submodel field for each level:
const Model = S.Struct({
  accountPopover: Popover.Model,
  accountDetailsPopover: Popover.Model,
  // ...your other fields
})

// The parent uses contentFocus so focus can move into its nested trigger
// instead of staying on the panel:
const init = () => [
  {
    accountPopover: Popover.init({
      id: 'account-popover',
      contentFocus: true,
    }),
    accountDetailsPopover: Popover.init({ id: 'account-details-popover' }),
    // ...your other fields
  },
  [],
]

// Embed each Popover Message in your parent Message:
const GotAccountPopoverMessage = m('GotAccountPopoverMessage', {
  message: Popover.Message,
})

const GotAccountDetailsPopoverMessage = m('GotAccountDetailsPopoverMessage', {
  message: Popover.Message,
})

// Inside your update function's M.tagsExhaustive({...}), delegate each
// Popover to its own Model field:
GotAccountPopoverMessage: ({ message }) => {
  const [nextAccountPopover, commands] = Popover.update(
    model.accountPopover,
    message,
  )

  return [
    evo(model, { accountPopover: () => nextAccountPopover }),
    Command.mapMessages(commands, message =>
      GotAccountPopoverMessage({ message }),
    ),
  ]
}

GotAccountDetailsPopoverMessage: ({ message }) => {
  const [nextAccountDetailsPopover, commands] = Popover.update(
    model.accountDetailsPopover,
    message,
  )

  return [
    evo(model, { accountDetailsPopover: () => nextAccountDetailsPopover }),
    Command.mapMessages(commands, message =>
      GotAccountDetailsPopoverMessage({ message }),
    ),
  ]
}

// Inside your view function, render the child Popover inside the parent
// panel. `focusSelector` points at the child trigger, which Popover derives
// from the child id as `${id}-button`.
const view = (h: HtmlBuilder<Message>) => {
  const detailsPopover = h.submodel({
    slotId: 'account-details-popover',
    model: model.accountDetailsPopover,
    view: Popover.view,
    viewInputs: {
      anchor: { placement: 'right-start', gap: 8, padding: 8 },
      toView: ({ button, panel, backdrop, isVisible }) =>
        h.div(
          [h.Class('relative inline-block')],
          [
            h.button(
              [
                ...button,
                h.Class('rounded-lg border px-3 py-2 cursor-pointer'),
              ],
              [h.span([], ['Advanced settings'])],
            ),
            ...(isVisible
              ? [
                  h.div([...backdrop, h.Class('fixed inset-0')]),
                  h.div(
                    [...panel, h.Class('rounded-lg border shadow-lg p-4 w-64')],
                    [
                      h.p([h.Class('font-medium')], ['Permissions']),
                      h.p(
                        [h.Class('text-sm text-gray-500')],
                        [
                          'Review who can change billing, members, and integrations.',
                        ],
                      ),
                    ],
                  ),
                ]
              : []),
          ],
        ),
    },
    toParentMessage: message => GotAccountDetailsPopoverMessage({ message }),
  })

  return h.submodel({
    slotId: 'account-popover',
    model: model.accountPopover,
    view: Popover.view,
    viewInputs: {
      anchor: { placement: 'bottom-start', gap: 4, padding: 8 },
      focusSelector: '#account-details-popover-button',
      toView: ({ button, panel, backdrop, isVisible }) =>
        h.div(
          [h.Class('relative inline-block')],
          [
            h.button(
              [
                ...button,
                h.Class('rounded-lg border px-3 py-2 cursor-pointer'),
              ],
              [h.span([], ['Account'])],
            ),
            ...(isVisible
              ? [
                  h.div([...backdrop, h.Class('fixed inset-0')]),
                  h.div(
                    [...panel, h.Class('rounded-lg border shadow-lg p-4 w-72')],
                    [
                      h.p([], ['Manage account settings from this panel.']),
                      detailsPopover,
                    ],
                  ),
                ]
              : []),
          ],
        ),
    },
    toParentMessage: message => GotAccountPopoverMessage({ message }),
  })
}
```

## Styling

Popover is headless. The `toView` callback receives attribute bundles for the button, panel, and backdrop, and the consumer composes the markup.

When `isAnimated` is true, enter/leave animations flow through the [Animation](https://foldkit.dev/ui/animation) module. Style with CSS transitions or CSS keyframe animations. Animation advances once every animation on the element has settled.

Attribute

Condition

`data-open`

Present on button and panel when open.

`data-disabled`

Present on the button when disabled.

`data-closed`

Present during close animation.

`data-placement`

Present on the panel, set to the side it currently sits on: top, right, bottom, or left. Fixed to the first resolved side when isPlacementLocked is true.

## Keyboard Interaction

By default, the panel receives `tabindex="0"` so it can receive focus. Tab navigates naturally through the panel content. Escape closes and returns focus to the button.

Key

Description

`Enter / Space`

Toggles the popover.

`Escape`

Closes the popover and returns focus to the button.

`Tab`

Navigates within the panel. By default, closes the popover when focus leaves.

## Accessibility

The button receives `aria-expanded` and `aria-controls` linking to the panel. The panel has no role. Popover uses the disclosure pattern, not the menu pattern.

Give the trigger an accessible name. For a visible label, wire a native `<label for>` that targets the trigger id with `Popover.buttonId(id)` rather than hardcoding the `-button` convention. The `for` association makes the trigger properly labeled: assistive technology announces it by the visible label text, and clicking the label opens the popover. That is why it is the recommended pattern.

Two ViewConfig fields cover the cases a `<label for>` does not. Pass `ariaLabel` for an icon-only trigger with no visible label, or `ariaLabelledBy` when the element that names the trigger is not a `<label>` you can point `for` at.

## API Reference

### InitConfig

Configuration object passed to `Popover.init()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the popover instance.

`isAnimated`

`boolean`

`false`

Enables animation coordination.

`isModal`

`boolean`

`false`

Locks page scroll and marks other elements inert when open.

`contentFocus`

`boolean`

`false`

Hands focus ownership to the consumer. When true, the panel is not focusable and does not close on blur; the consumer must focus a descendant on open and decide on its own blur rules.

### ViewConfig

Configuration object passed to `Popover.view()`.

Name

Type

Default

Description

`model`

`Popover.Model`

—

The popover state from your parent Model.

`toParentMessage`

`(childMessage: Popover.Message) => ParentMessage`

—

Wraps Popover Messages in your parent Message type for Submodel delegation.

`anchor`

`AnchorConfig`

—

Floating positioning config: placement, gap, offset, padding, isPlacementLocked, and portal. Required. Portaled to the document body by default; pass portal: false to keep the panel inside its wrapper.

`toView`

`(render: RenderInfo) => Html`

—

Callback that receives the button, panel, and backdrop attribute bundles plus a derived

`isVisible`

flag, and returns the composed layout.

`isDisabled`

`boolean`

`false`

Disables the trigger button.

`focusSelector`

`string`

—

CSS selector for the element to focus after the panel is positioned. Defaults to the panel itself.

`ariaLabel`

`string`

—

Accessible name for the trigger button. Use for an icon-only trigger with no visible label. Applied as aria-label, and takes precedence over ariaLabelledBy.

`ariaLabelledBy`

`string`

—

Id of an external element that labels the trigger button, applied as aria-labelledby. Pair with a visible label element.

### RenderInfo

Payload delivered to the `toView` callback each render.

Name

Type

Default

Description

`button`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the trigger button. Includes the button id,

`aria-expanded`

,

`aria-controls`

, and pointer/keyboard handlers.

`panel`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the floating panel. Includes the anchor Mount that positions the panel via Floating UI, ARIA linkage to the button, and panel keydown/blur handlers.

`backdrop`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the modal backdrop element. Includes the portal Mount that moves the backdrop to

`document.body`

. The backdrop's click handler dispatches

`RequestedClose`

.

`isVisible`

`boolean`

—

Derived from

`isOpen`

and the Animation

`transitionState`

. Render the panel and backdrop only while this is true.

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Pattern-match on the OutMessage in your update handler.

Name

Type

Default

Description

`Opened`

`{}`

—

Emitted once the popover has transitioned to open. Fires after

`update`

has processed

`RequestedOpen`

and

`isOpen`

reflects the new state.

`Closed`

`{}`

—

Emitted once the popover has transitioned to closed. Programmatic

`Popover.close`

on an already-closed model is a no-op that does not re-emit.
