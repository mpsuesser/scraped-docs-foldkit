---
url: https://foldkit.dev/ui/tooltip
title: "Tooltip"
description: "Non-interactive floating label that appears on hover or focus and hides on leave, blur, or Escape."
access_date: 2026-08-03T19:40:01.169Z
current_date: 2026-08-03T19:40:01.169Z
---

# Tooltip

## Overview

A non-interactive floating label anchored to a trigger. Tooltips appear on hover after a short delay, or immediately on keyboard focus. They hide on pointer-leave, blur, or Escape. Use tooltips for short hints about a control. Because they rely on hover and keyboard focus, don’t use them for content that must be reachable on touch; for that, or for rich or interactive content, use `Popover` instead.

The positioning engine is shared with `Popover` and `Menu`. Pass `anchor` to control placement and spacing.

See it in an app

Check out how Tooltip is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/tooltip.ts).

## Examples

Hover or tab into the trigger to reveal the tooltip. Hover waits for `showDelay` (default 500ms); keyboard focus shows it immediately.

Tooltip trigger

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Match as M, Option } from 'effect'
import { Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Tooltip } from '@foldkit/ui'

// Add a field to your Model for the Tooltip Submodel:
const Model = S.Struct({
  tooltip: Tooltip.Model,
  // ...your other fields
})

// In your init function, initialize the Tooltip Submodel with a unique id:
const init = () => [
  {
    tooltip: Tooltip.init({ id: 'save-button' }),
    // ...your other fields
  },
  [],
]

// Embed the Tooltip Message in your parent Message:
const GotTooltipMessage = m('GotTooltipMessage', {
  message: Tooltip.Message,
})

// Inside your update function's M.tagsExhaustive({...}), delegate to
// Tooltip.update. The OutMessages `Shown` and `Hidden` mark the
// visibility transitions. Fire analytics or coordinate with the rest
// of your UI from the parent.
GotTooltipMessage: ({ message }) => {
  const [nextTooltip, commands, maybeOutMessage] = Tooltip.update(
    model.tooltip,
    message,
  )
  const mappedCommands = Command.mapMessages(commands, message =>
    GotTooltipMessage({ message }),
  )

  return Option.match(maybeOutMessage, {
    onNone: () => [evo(model, { tooltip: () => nextTooltip }), mappedCommands],
    onSome: M.type<Tooltip.OutMessage>().pipe(
      M.tagsExhaustive({
        Shown: () => [
          // The child has emitted `Shown`. The body commits the
          // child's next state as usual. In this arm the parent can
          // also update its own state or dispatch its own Commands,
          // for example log analytics, prefetch content, or trigger
          // a downstream Command.
          evo(model, { tooltip: () => nextTooltip }),
          mappedCommands,
        ],
        Hidden: () => [
          // The child has emitted `Hidden`. The body commits the
          // child's next state as usual. In this arm the parent can
          // also update its own state or dispatch its own Commands,
          // for example clear ephemeral state, fire analytics, or
          // trigger a downstream Command.
          evo(model, { tooltip: () => nextTooltip }),
          mappedCommands,
        ],
      }),
    ),
  })
}

// Inside your view function, embed the tooltip via h.submodel. The tooltip
// describes the trigger but does not name it, so give an icon-only trigger
// an accessible name with `ariaLabel`. (Point `ariaLabelledBy` at a visible
// label element instead when one exists.) The attribute is only emitted when
// provided, so the trigger never carries a dangling `aria-labelledby`.
const view = (h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'save-button',
    model: model.tooltip,
    view: Tooltip.view,
    viewInputs: {
      ariaLabel: 'Save',
      anchor: { placement: 'top', gap: 6, padding: 8 },
      toView: ({ trigger, panel, isVisible }) =>
        h.div(
          [h.Class('relative inline-block')],
          [
            h.button(
              [
                ...trigger,
                h.Class('rounded-lg border px-3 py-2 cursor-pointer'),
              ],
              // Icon-only content; `ariaLabel` above supplies the name.
              [h.span([h.AriaHidden(true)], ['💾'])],
            ),
            ...(isVisible
              ? [
                  h.div(
                    [
                      ...panel,
                      h.Class(
                        'rounded-md bg-gray-900 px-3 py-1.5 text-sm text-white shadow-lg',
                      ),
                    ],
                    [h.span([], ['Save your changes (⌘S)'])],
                  ),
                ]
              : []),
          ],
        ),
    },
    toParentMessage: message => GotTooltipMessage({ message }),
  })
```

## Styling

Tooltip is headless. The `toView` callback receives attribute bundles for the trigger and panel, and the consumer composes the markup. The panel is rendered with `pointer-events: none` so it never captures hover or clicks, which keeps the open/close logic tied to the trigger.

Attribute

Condition

`data-open`

Present on trigger and panel when the tooltip is visible.

`data-disabled`

Present on the trigger when disabled.

`data-placement`

Present on the panel, set to the side it currently sits on: top, right, bottom, or left. Fixed to the first resolved side when isPlacementLocked is true.

## Keyboard Interaction

Key

Description

`Escape`

Hides the tooltip while visible. It will not reopen until the user disengages by moving the pointer away or blurring the trigger.

## Accessibility

The panel has `role="tooltip"` and the trigger is linked via `aria-describedby`. Focus is never moved into the tooltip, so assistive technology announces the panel contents as a description of the trigger.

The tooltip describes the trigger but does not name it, so give the trigger an accessible name. For a visible label, wire a native `<label for>` that targets the trigger id with `Tooltip.triggerId(id)` rather than hardcoding the `-trigger` convention. The `for` association makes the trigger properly labeled: assistive technology announces it by the visible label text, and clicking the label focuses the trigger. That is why it is the recommended pattern.

Two ViewConfig fields cover the cases a `<label for>` does not. Pass `ariaLabel` for an icon-only trigger with no visible label, or `ariaLabelledBy` when the element that names the trigger is not a `<label>` you can point `for` at.

## API Reference

### InitConfig

Configuration object passed to `Tooltip.init()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the tooltip instance.

`showDelay`

`Duration.Input`

`Duration.millis(500)`

How long the pointer must hover before the tooltip appears. Accepts any Effect Duration input. A bare number is interpreted as milliseconds. Keyboard focus shows the tooltip immediately regardless of this value.

### ViewConfig

Configuration object passed to `Tooltip.view()`.

Name

Type

Default

Description

`model`

`Tooltip.Model`

—

The tooltip state from your parent Model.

`toParentMessage`

`(childMessage: Tooltip.Message) => ParentMessage`

—

Wraps Tooltip Messages in your parent Message type for Submodel delegation.

`anchor`

`AnchorConfig`

—

Floating positioning config: placement, gap, offset, padding, isPlacementLocked, and portal. Required. Portaled to the document body by default; pass portal: false to keep the panel inside its wrapper.

`toView`

`(render: RenderInfo) => Html`

—

Callback that receives the

`trigger`

and

`panel`

attribute bundles plus a derived

`isVisible`

flag, and returns the composed layout.

`isDisabled`

`boolean`

`false`

Disables the trigger. Hover, focus, and keyboard events are ignored and the tooltip will not open.

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

`trigger`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the trigger element. Carries

`type="button"`

, the hover/focus/keyboard handlers, and

`aria-describedby`

linking to the panel.

`panel`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the panel element. Carries

`role="tooltip"`

, the anchor Mount that positions the panel via Floating UI, and a

`data-open`

attribute when visible.

`isVisible`

`boolean`

—

Whether the tooltip is currently visible. The consumer decides whether to render the panel conditionally on this.

### Programmatic Helpers

Helper functions for driving the tooltip from parent update handlers, returning `[Model, Commands]`.

Name

Type

Default

Description

`reflectShowDelay`

`(model: Model, showDelay: Duration.Input) => Model`

—

Reflects an externally-sourced hover show-delay onto the Model (a user preference, a restored setting) without emitting an OutMessage. Accepts any Effect Duration input; a bare number is milliseconds. The new delay applies on the next hover. Dual: pass just the delay for a point-free setter in an evo callback.

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Fire only on visibility transitions, so consumers don’t see spurious events for Messages that only update internal hover/focus/delay state.

Name

Type

Default

Description

`Shown`

`{}`

—

Emitted once the tooltip transitions to visible (isOpen becomes true). Pattern-match the third tuple element of Tooltip.update to react. Useful for analytics, instrumentation, or coordinating with other transient UI.

`Hidden`

`{}`

—

Emitted once the tooltip transitions to hidden (isOpen becomes false).
