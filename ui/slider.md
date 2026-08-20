---
url: https://foldkit.dev/ui/slider
title: "Slider"
description: "Accessible slider for numeric range input. Pointer drag, keyboard navigation, ARIA slider semantics."
access_date: 2026-08-20T02:21:49.544Z
current_date: 2026-08-20T02:21:49.544Z
---

## Overview

A numeric range input for values that sit on a continuous or stepped scale. Common uses include rating scales, volume controls, filter thresholds, and brightness settings. Follows the WAI-ARIA slider pattern with `role="slider"`, full keyboard navigation, and pointer drag.

See it in an app

Check out how Slider is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/slider.ts).

## Examples

Slider is headless. Your `toView` callback controls all markup and styling. The component hands back attribute groups for the root, track, filled track, thumb, label, and an optional hidden input for form submission.

3 of 10

50%

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit each into your own Model, init, Message,
// update, view, and subscription definitions.
import { Match as M, Option, Schema as S } from 'effect'
import { Subscription, Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Slider } from '@foldkit/ui'

// Add two fields to your Model: the value you own, and the Slider Submodel's
// interaction state:
const Model = S.Struct({
  ratingValue: S.Number,
  ratingDemo: Slider.Model,
  // ...your other fields
})

// In your init function, seed the value (snapped to the range) and initialize
// the Slider Submodel with min / max / step and a unique id:
const init = () => [
  {
    ratingValue: Slider.snapAndClamp(3, 0, 10, 1),
    ratingDemo: Slider.init({
      id: 'rating',
      min: 0,
      max: 10,
      step: 1,
    }),
    // ...your other fields
  },
  [],
]

// Embed the Slider Message in your parent Message:
const GotSliderMessage = m('GotSliderMessage', {
  message: Slider.Message,
})

// At module scope, fold the OutMessage into your own Model. \`ChangedValue\`
// carries the new number. Lift it to domain state, validate, or persist on
// each commit. The arm returns an Update.Step over the parent Model, which
// already has the next Slider Model written back:
const foldSliderOutMessage = M.type<Slider.OutMessage>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    // The child has emitted \`ChangedValue\`. Store the new value in the field
    // you own. This arm is also where the parent can validate, persist, or
    // trigger a downstream Command.
    ChangedValue:
      ({ value }) =>
      model => [evo(model, { ratingValue: () => value }), []],
  }),
)

// Update.foldChild wires the child into the parent: it runs Slider.update,
// writes the next Slider Model back, maps the Submodel's Commands into your
// Message type, and hands any OutMessage to foldOutMessage.
const foldSlider = Update.foldChild({
  update: Slider.update,
  read: (model: Model) => Option.some(model.ratingDemo),
  write: (model, nextRatingDemo) =>
    evo(model, { ratingDemo: () => nextRatingDemo }),
  toParentMessage: message => GotSliderMessage({ message }),
  foldOutMessage: foldSliderOutMessage,
})

// Inside your update function's M.tagsExhaustive({...}), call the fold:
GotSliderMessage: ({ message }) => foldSlider(model, message)

// NOTE: wire BOTH dragPointer and dragEscape. Without dragEscape, pressing
// Escape during a drag won't cancel back to the origin value, but every
// other drag mechanic still works. Silent partial breakage.
const sliderSubscriptions = Subscription.lift({
  sliderPointer: Slider.subscriptions.dragPointer,
  sliderEscape: Slider.subscriptions.dragEscape,
})<Model, Message>({
  toChildModel: model => model.ratingDemo,
  toParentMessage: message => GotSliderMessage({ message }),
})

const subscriptions = Subscription.aggregate<Model, Message>()(
  sliderSubscriptions,
  // ...your other subscription records
)

// Inside your view function, render the slider. You control every element's
// markup and classes through the \`toView\` callback. The \`attributes\` groups
// provide ARIA, pointer, and keyboard wiring:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'rating',
    model: model.ratingDemo,
    view: Slider.view,
    viewInputs: {
      value: model.ratingValue,
      formatValue: value => \`${String(value)} of 10\`,
      toView: attributes =>
        h.div(
          [h.Class('flex flex-col gap-2 w-full max-w-sm')],
          [
            h.div(
              [h.Class('flex items-center justify-between text-sm')],
              [
                h.label(
                  [...attributes.label, h.Class('font-medium')],
                  ['Rating'],
                ),
                h.span(
                  [h.Class('tabular-nums text-gray-600')],
                  [\`${String(model.ratingValue)} / 10\`],
                ),
              ],
            ),
            h.div(
              [
                ...attributes.root,
                h.Class('relative h-6 w-full flex items-center'),
              ],
              [
                h.div(
                  [
                    ...attributes.track,
                    h.Class('h-1.5 w-full rounded-full bg-gray-200'),
                  ],
                  [
                    h.div([
                      ...attributes.filledTrack,
                      h.Class('h-full rounded-full bg-blue-600'),
                    ]),
                  ],
                ),
                h.div([
                  ...attributes.thumb,
                  h.Class(
                    'h-5 w-5 rounded-full bg-white border-2 border-blue-600 shadow cursor-grab focus-visible:ring-2 focus-visible:ring-blue-600 data-[dragging]:cursor-grabbing',
                  ),
                ]),
              ],
            ),
          ],
        ),
    },
    toParentMessage: message => GotSliderMessage({ message }),
  })
```

## Subscriptions

Pointer drag needs document-level `pointermove` / `pointerup` tracking (the cursor can leave the slider element). Slider exposes this as a Subscription you wire into your app’s `subscriptions` alongside an Escape-key Subscription that cancels an in-progress drag. The example snippet above shows the full wiring.

## Styling

Slider exposes `data-dragging` while the user is actively dragging, `data-disabled` when disabled, `data-readonly` when read-only, and `data-orientation` on the root. The `filledTrack` attribute group carries an inline width so the filled portion always matches the current value.

| Attribute | Condition |
| --- | --- |
| `data-dragging` | Present on the root, track, filled track, and thumb while the user is actively dragging. |
| `data-disabled` | Present on all groups when isDisabled is true. |
| `data-readonly` | Present on all groups when isReadOnly is true. |
| `data-orientation` | Present on the root. Always "horizontal" in v1; vertical is planned. |

## Keyboard Interaction

| Key | Description |
| --- | --- |
| `ArrowRight / ArrowUp` | Increases the value by one step. |
| `ArrowLeft / ArrowDown` | Decreases the value by one step. |
| `PageUp` | Increases the value by ten steps. |
| `PageDown` | Decreases the value by ten steps. |
| `Home` | Jumps to the minimum value. |
| `End` | Jumps to the maximum value. |
| `Escape` | During a pointer drag, cancels the drag and restores the pre-drag value. |

Every key in this table is inert when `isDisabled` or `isReadOnly` is true, because both remove the thumb's keydown handler. Escape is the exception. It cancels a drag through a Subscription rather than the handler, so a drag that began before the slider became read-only can still be cancelled, and can still run to pointerup on its own. Flipping `isReadOnly` mid-drag does not interrupt the drag in flight. `isDisabled` behaves the same way.

## Accessibility

The thumb receives `role="slider"`, `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, and `aria-orientation`. When `formatValue` is provided, the formatted string is announced via `aria-valuetext`. By default the thumb is labeled via `aria-labelledby` pointing at the id carried on the `label` attribute group; you can override this with an explicit `ariaLabel` or `ariaLabelledBy`.

`isReadOnly` and `isDisabled` both stop the Slider from reacting to pointer drags and keys. They differ in the semantics exposed to assistive technology, so they are not interchangeable.

`aria-disabled="true"`, which `isDisabled` emits, communicates that the Slider is unavailable. `aria-readonly="true"`, which `isReadOnly` emits, communicates that its value cannot be changed but remains relevant to the user. It sits on the thumb, the element carrying `role="slider"`. Both states keep `tabindex="0"`, following Foldkit's convention that unavailable controls remain discoverable by keyboard and assistive technology.

Assistive technology support for `aria-readonly` on sliders varies. Pair it with a visible read-only treatment or explanatory text when users must distinguish it from disabled, and test the browser and assistive technology combinations your app supports.

Use `isReadOnly` when the value is still information the user needs, such as a level set by another control, and `isDisabled` when the Slider is unavailable.

The two flags are independent. Setting both emits both sets of attributes, and either one on its own removes the pointer and keyboard handlers.

## API Reference

### InitConfig

Configuration object passed to `Slider.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the slider instance. |
| `min` | `number` | — | Minimum value. |
| `max` | `number` | — | Maximum value. |
| `step` | `number` | — | Increment between allowed values. Fractional steps are rounded to the step’s decimal precision to avoid floating-point drift. |

### ViewConfig

Configuration object passed to `Slider.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `model` | `Slider.Model` | — | The slider state from your parent Model. |
| `toParentMessage` | `(childMessage: Slider.Message) => ParentMessage` | — | Wraps Slider Messages in your parent Message type for Submodel delegation. |
| `value` | `number` | — | The current value, read from your parent Model. The thumb position, aria-valuenow, and filled track derive from it. Fold the ChangedValue OutMessage into this field in your update. |
| `toView` | `(attributes: SliderAttributes) => Html` | — | Callback that receives attribute groups for the root, track, filled track, thumb, label, and hidden input elements. |
| `ariaLabel` | `string` | — | Accessible name for screen readers when there is no visible label. |
| `ariaLabelledBy` | `string` | — | ID of an external element whose text serves as the slider’s accessible name. |
| `formatValue` | `(value: number) => string` | — | Produces the aria-valuetext announced to screen readers. Use it when the numeric value needs a natural-language form (e.g. "3 of 10" or "50 percent"). |
| `isDisabled` | `boolean` | `false` | Whether the slider is disabled. Removes pointer and keyboard interactivity while preserving focusability. |
| `isReadOnly` | `boolean` | `false` | Whether the slider is readable but not adjustable. Carries `aria-readonly` rather than `aria-disabled`. Independent of `isDisabled`. Removes pointer and keyboard interactivity while preserving focusability. |
| `name` | `string` | — | Form field name. When provided, a hidden input carrying the current numeric value is included for native form submission. |
| `getTrackRoot` | `(() => Document \| ShadowRoot) \| undefined` | — | Optional accessor returning the DOM root that contains the slider track. Defaults to `document`. Override when rendering inside a Shadow DOM so the drag subscription can find the track element to measure cursor position. |

### SliderAttributes

Attribute groups provided to the `toView` callback.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `root` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the outer wrapper. Carries data-slider-id, data-orientation, and state data attributes. |
| `track` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the track element (the bar). Carries data-slider-track-id (used by the drag subscription to measure cursor position), positioning styles, and the pointerdown handler for click-to-jump. |
| `filledTrack` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto an element nested inside the track. Its inline width reflects the current value as a percentage of the range. |
| `thumb` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the draggable handle. Carries role="slider", tabindex, aria-value\*, the pointerdown handler, the keyboard handler, and positioning. |
| `label` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto the visible label element. Carries the id the thumb’s aria-labelledby points to by default. |
| `hiddenInput` | `ReadonlyArray<Attribute<Message>>` | — | Spread onto a hidden `<input>` for form submission. Only populated when the name prop is set. |

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Parents fold the OutMessage in the `foldOutMessage` of their [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `ChangedValue` | `{ value: number }` | — | Emitted whenever the slider value changes via drag, click-to-jump, or keyboard navigation. Fold it in the `foldOutMessage` of your Slider fold to react, for example: persist the value, validate, or trigger a downstream Command. |
