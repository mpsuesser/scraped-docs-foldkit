---
url: https://foldkit.dev/ui/popover
title: "Popover"
description: "An anchored floating panel for arbitrary content, with dismissal, focus return, portaling, and optional modal behavior."
access_date: 2026-08-31T07:29:25.100Z
current_date: 2026-08-31T07:29:25.100Z
---

## Overview

An anchored floating panel with natural Tab navigation. Unlike Dialog (which is modal and traps focus) or Menu (which uses aria-activedescendant for item navigation), Popover holds arbitrary content and uses the disclosure ARIA pattern. Focus flows naturally through the panel content.

For programmatic control in a parent update, fold `Popover.open` and `Popover.close` with `Update.foldChildStep`. Both are no-argument child entry points, so each fold produces a step that can run directly or inside `Update.combine`.

See it in an app

Check out how Popover is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/popover.ts).

## Examples

### Basic

Pass `anchor` to position the panel relative to the button. The panel can hold any content: links, forms, or informational text.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Match as M, Option, Schema as S } from 'effect'
import { Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Popover } from '@foldkit/ui'

// Add a field to your Model for the Popover Submodel:
const Model = S.Struct({
  popover: Popover.Model,
  // ...your other fields
})

// In your init function, initialize the Popover Submodel with a unique id:
const init = () => ({
  model: {
    popover: Popover.init({ id: 'info' }),
    // ...your other fields
  },
})

// Embed the Popover Message in your parent Message:
const Message = defineMessageUnion({
  GotPopoverMessage: { message: Popover.Message },
})

// At module scope, fold the OutMessage into your own Model. \`Opened\` and
// \`Closed\` mark the visibility transitions. Fire analytics, coordinate with
// other UI, or clear ephemeral state on close. Each arm returns an
// Update.Step over the parent Model, which already has the next Popover Model
// written back:
const foldPopoverOutMessage = M.type<Popover.OutMessage>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    // The child has emitted \`Opened\`. In this arm the parent can update its
    // own state or dispatch its own Commands, for example lazy-load panel
    // content, log analytics, or trigger a downstream Command.
    Opened: () => model => ({ model }),
    // The child has emitted \`Closed\`. In this arm the parent can update its
    // own state or dispatch its own Commands, for example persist a draft,
    // clear ephemeral state, or trigger a downstream Command.
    Closed: () => model => ({ model }),
  }),
)

// Update.foldChild wires the child into the parent: it runs Popover.update,
// writes the next Popover Model back, maps the Submodel's Commands into your
// Message type, and hands any OutMessage to foldOutMessage.
const foldPopover = Update.foldChild({
  update: Popover.update,
  read: (model: Model) => Option.some(model.popover),
  write: (model, nextPopover) => evo(model, { popover: () => nextPopover }),
  toParentMessage: message => Message.GotPopoverMessage({ message }),
  foldOutMessage: foldPopoverOutMessage,
})

// In the corresponding Message.match handler, call the fold:
GotPopoverMessage: ({ message }) => foldPopover(model, message)

// Inside your view function, embed the popover via h.submodel. Give the
// trigger an accessible name: target the trigger id with
// \`Popover.buttonId('info')\` from a native \`<label for>\`, and pass
// \`ariaLabelledBy\` so the trigger is named by the label. The attribute is
// only emitted when provided, so the trigger never carries a dangling
// \`aria-labelledby\`.
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
    toParentMessage: message => Message.GotPopoverMessage({ message }),
  })
}
```

### Arrow

Popover does not draw an arrow. It positions one. Spread the `arrow` bundle onto your own element inside the panel and write the CSS in [Drawing an Arrow](#drawing-an-arrow) below.

```
// Pseudocode walkthrough of what an arrow adds to a Popover you already have.
// Popover positions the arrow. The CSS below the demo draws it.
import type { HtmlBuilder } from 'foldkit/html'

import { Popover } from '@foldkit/ui'

const view = (h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'info',
    model: model.popover,
    view: Popover.view,
    viewInputs: {
      // Leave room for the arrow tip, which reaches 8px past the panel:
      anchor: { placement: 'bottom-start', gap: 10, padding: 8 },
      // Keep the arrow clear of the panel's rounded corners:
      arrowPadding: 12,
      // Take the arrow bundle from the render payload:
      toView: ({ button, panel, backdrop, arrow, isVisible }) =>
        h.div(
          [h.Class('relative inline-block')],
          [
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
                    [
                      ...panel,
                      // The placement rules target the arrow through the
                      // panel, so the panel needs a class they can name:
                      h.Class(
                        'popover-panel rounded-lg border shadow-lg p-4 w-80',
                      ),
                    ],
                    [
                      // Spread the bundle onto your own element, inside the
                      // panel. The fill masks the panel border, while the
                      // nested SVG clips the outline at the panel edge:
                      h.svg(
                        [
                          ...arrow,
                          h.Class('popover-arrow'),
                          h.ViewBox('0 0 16 16'),
                        ],
                        [
                          h.path([
                            h.Class('popover-arrow-fill'),
                            h.D('M 0.5 8 L 8 0.5 L 15.5 8 V 10 H 0.5 Z'),
                          ]),
                          h.svg(
                            [
                              h.Class('popover-arrow-outline-clip'),
                              h.Width('16'),
                              h.Height('8'),
                              h.ViewBox('0 0 16 8'),
                            ],
                            [
                              h.path([
                                h.Class('popover-arrow-outline'),
                                h.D('M 0.5 8 L 8 0.5 L 15.5 8'),
                              ]),
                            ],
                          ),
                        ],
                      ),
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
    toParentMessage: message => Message.GotPopoverMessage({ message }),
  })
```

### Animated

Pass `isAnimated: true` at init for animation coordination.

### Nested

Use a separate Popover Model for each level. For a parent panel that opens onto another Popover trigger, pass `contentFocus: true` at init and `focusSelector` in the view so focus lands on the nested trigger.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Match as M, Option, Schema as S } from 'effect'
import { Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Popover } from '@foldkit/ui'

// Add one Popover Submodel field for each level:
const Model = S.Struct({
  accountPopover: Popover.Model,
  accountDetailsPopover: Popover.Model,
  // ...your other fields
})
type Model = typeof Model.Type

// The parent uses contentFocus so focus can move into its nested trigger
// instead of staying on the panel:
const init = () => ({
  model: {
    accountPopover: Popover.init({
      id: 'account-popover',
      contentFocus: true,
    }),
    accountDetailsPopover: Popover.init({ id: 'account-details-popover' }),
    // ...your other fields
  },
})

// Embed each Popover Message in your parent Message:
const Message = defineMessageUnion({
  GotAccountPopoverMessage: { message: Popover.Message },
  GotAccountDetailsPopoverMessage: { message: Popover.Message },
})
type Message = typeof Message.Type

const foldPopoverOutMessage = M.type<Popover.OutMessage>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    Opened: () => model => ({ model }),
    Closed: () => model => ({ model }),
  }),
)

const foldAccountPopover = Update.foldChild({
  update: Popover.update,
  read: (model: Model) => Option.some(model.accountPopover),
  write: (model, nextAccountPopover) =>
    evo(model, { accountPopover: () => nextAccountPopover }),
  toParentMessage: message => Message.GotAccountPopoverMessage({ message }),
  foldOutMessage: foldPopoverOutMessage,
})

const foldAccountDetailsPopover = Update.foldChild({
  update: Popover.update,
  read: (model: Model) => Option.some(model.accountDetailsPopover),
  write: (model, nextAccountDetailsPopover) =>
    evo(model, { accountDetailsPopover: () => nextAccountDetailsPopover }),
  toParentMessage: message =>
    Message.GotAccountDetailsPopoverMessage({ message }),
  foldOutMessage: foldPopoverOutMessage,
})

// In the corresponding Message.match handlers, delegate each
// Popover to its own Model field:
GotAccountPopoverMessage: ({ message }) => foldAccountPopover(model, message)

GotAccountDetailsPopoverMessage: ({ message }) =>
  foldAccountDetailsPopover(model, message)

// Inside your view function, render the child Popover inside the parent
// panel. \`focusSelector\` points at the child trigger, which Popover derives
// from the child id as \`${id}-button\`.
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
    toParentMessage: message =>
      Message.GotAccountDetailsPopoverMessage({ message }),
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
    toParentMessage: message => Message.GotAccountPopoverMessage({ message }),
  })
}
```

## Styling

Popover is headless. The `toView` callback receives attribute bundles for the button, panel, backdrop, and arrow, and the consumer composes the markup.

When `isAnimated` is true, enter/leave animations flow through the [Animation](https://foldkit.dev/ui/animation) module. Style with CSS transitions or CSS keyframe animations. Animation advances once every animation on the element has settled.

| Attribute | Condition |
| --- | --- |
| `data-open` | Present on button and panel when open. |
| `data-disabled` | Present on the button when disabled. |
| `data-closed` | Present during close animation. |
| `data-placement` | Present on the panel, set to the side it currently sits on: top, right, bottom, or left. Fixed to the first resolved side when isPlacementLocked is true. |

### Drawing an Arrow

`toView` receives an `arrow` bundle carrying the element's id. Popover does not draw the arrow. Spread the bundle onto your own element, a direct child of the panel, and place it with the custom properties Anchor publishes:

```
.popover-panel {
  --popover-background: white;
  --popover-border: #e5e7eb;
  background: var(--popover-background);
  border-color: var(--popover-border);
}

.popover-arrow {
  position: absolute;
  width: 16px;
  height: 16px;
  left: var(--arrow-x);
  top: var(--arrow-y);
}

.popover-arrow-fill {
  fill: var(--popover-background);
  stroke: none;
}

.popover-arrow-outline-clip {
  overflow: hidden;
}

.popover-arrow-outline {
  fill: none;
  stroke: var(--popover-border);
  stroke-linejoin: round;
  stroke-width: 1px;
}

.popover-panel[data-placement='top'] > .popover-arrow {
  bottom: -8px;
  transform: rotate(180deg);
}

.popover-panel[data-placement='bottom'] > .popover-arrow {
  top: -8px;
}

.popover-panel[data-placement='left'] > .popover-arrow {
  right: -8px;
  transform: rotate(90deg);
}

.popover-panel[data-placement='right'] > .popover-arrow {
  left: -8px;
  transform: rotate(-90deg);
}
```

`--arrow-x` and `--arrow-y` position the arrow along the panel edge. Anchor sets one of them for each placement, while the matching `data-placement` rule pins the arrow to the correct side.

Write a rule for every side the panel can use because a `bottom` placement can flip to `top`, and a `left` placement can flip to `right`. The direct-child selector keeps a nested popover tied to its own panel. Pass `arrowPadding` to keep the arrow clear of rounded corners.

The square SVG keeps its measurements stable when the placement flips. Its fill extends 2px into the panel to mask the border beneath it, while a separate path outlines only the two outward-facing edges. The 16px arrow reaches 8px past the panel, so the demo uses `gap: 10` to keep it clear of the trigger. Anchor centers the arrow on the trigger along the panel's edge.

### Scrollable Panels with an Arrow

An arrow sits outside the panel, so scrolling the panel itself would clip it. Anchor leaves the panel unclipped when an arrow resolves and still writes its `max-height`. If the content can outgrow that height, make the panel a flex column and scroll an inner container:

```
.popover-panel {
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.popover-content {
  min-height: 0;
  overflow-y: auto;
}
```

`min-height: 0` lets the child shrink below its content height so it can scroll. `box-sizing: border-box` keeps the panel's padding and border within the height Anchor measured.

Do not put `max-height: inherit` on the child. That copies the panel's full measured height without accounting for the panel's padding and border. The flex layout avoids that arithmetic.

## Keyboard Interaction

By default, the panel receives `tabindex="0"` so it can receive focus. Tab navigates naturally through the panel content. Escape closes and returns focus to the button.

| Key | Description |
| --- | --- |
| `Enter / Space` | Toggles the popover. |
| `Escape` | Closes the popover and returns focus to the button. |
| `Tab` | Navigates within the panel. By default, closes the popover when focus leaves. |

## Accessibility

The button receives `aria-expanded` and `aria-controls` linking to the panel. The panel has no role. Popover uses the disclosure pattern, not the menu pattern.

Give the trigger an accessible name. For a visible label, wire a native `<label for>` that targets the trigger id with `Popover.buttonId(id)` rather than hardcoding the `-button` convention. The `for` association makes the trigger properly labeled: assistive technology announces it by the visible label text, and clicking the label opens the popover. That is why it is the recommended pattern.

Two ViewConfig fields cover the cases a `<label for>` does not. Pass `ariaLabel` for an icon-only trigger with no visible label, or `ariaLabelledBy` when the element that names the trigger is not a `<label>` you can point `for` at.

## Testing

Spreading the `arrow` bundle supplies the arrow id in application markup. A Scene test that asserts the panel Mount's `arrowId` argument needs to construct that value directly. Use `Popover.arrowId(id)` rather than hardcoding the `-arrow` convention.

## API Reference

### InitConfig

Configuration object passed to `Popover.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the popover instance. |
| `isAnimated` | `boolean` | `false` | Enables animation coordination. |
| `isModal` | `boolean` | `false` | Locks page scroll and marks other elements inert when open. |
| `contentFocus` | `boolean` | `false` | Hands focus ownership to the consumer. When true, the panel is not focusable and does not close on blur; the consumer must focus a descendant on open and decide on its own blur rules. |

### ViewConfig

Configuration object passed to `Popover.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `model` | `Popover.Model` | — | The popover state from your parent Model. |
| `toParentMessage` | `(childMessage: Popover.Message) => ParentMessage` | — | Wraps Popover Messages in your parent Message type for Submodel delegation. |
| `anchor` | `AnchorConfig` | — | Floating positioning config: placement, gap, offset, padding, isPlacementLocked, and portal. Required. Portaled to the document body by default; pass portal: false to keep the panel inside its wrapper. |
| `toView` | `(render: RenderInfo) => Html` | — | Callback that receives the button, panel, backdrop, and arrow attribute bundles plus a derived `isVisible` flag, and returns the composed layout. |
| `isDisabled` | `boolean` | `false` | Disables the trigger button. |
| `focusSelector` | `string` | — | CSS selector for the element to focus after the panel is positioned. Defaults to the panel itself. |
| `arrowPadding` | `number` | `0` | Distance in pixels the arrow keeps from the panel's corners. |
| `ariaLabel` | `string` | — | Accessible name for the trigger button. Use for an icon-only trigger with no visible label. Applied as aria-label, and takes precedence over ariaLabelledBy. |
| `ariaLabelledBy` | `string` | — | Id of an external element that labels the trigger button, applied as aria-labelledby. Pair with a visible label element. |

### RenderInfo

Payload delivered to the `toView` callback each render.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `button` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the trigger button. Includes the button id, `aria-expanded`, `aria-controls`, and pointer/keyboard handlers. |
| `panel` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the floating panel. Includes the anchor Mount that positions the panel via Floating UI, ARIA linkage to the button, and panel keydown/blur handlers. |
| `backdrop` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the modal backdrop element. Includes the portal Mount that moves the backdrop to `document.body`. The backdrop's click handler dispatches `RequestedClose`. |
| `arrow` | `ReadonlyArray<ChildAttribute>` | — | Spread onto your arrow element inside the panel. Carries the id the anchor Mount resolves and `aria-hidden`. Nothing renders until you add the element and the CSS above. |
| `isVisible` | `boolean` | — | Derived from `isOpen` and the Animation `transitionState`. Render the panel and backdrop only while this is true. |

### OutMessage

Messages emitted to the parent through the optional `outMessage` field. Fold the OutMessage in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `Opened` | `{}` | — | Emitted once the popover has transitioned to open. Fires after `update` has processed `RequestedOpen` and `isOpen` reflects the new state. |
| `Closed` | `{}` | — | Emitted once the popover has transitioned to closed. Programmatic `Popover.close` on an already-closed model is a no-op that does not re-emit. |
