---
url: https://foldkit.dev/ui/animation
title: "Animation"
description: "Coordinates CSS enter/leave animations via a state machine and data attributes. Works with both CSS transitions and keyframe animations."
access_date: 2026-08-17T01:15:19.180Z
current_date: 2026-08-17T01:15:19.180Z
---

## Animation

## Overview

Animation is a CSS animation lifecycle coordinator that manages enter/leave phases via a state machine and data attributes. If you're coming from imperative animation libraries (for example GSAP, Framer Motion, or `element.animate()`), it will feel inverted: those libraries let you say "do this now" and give you a callback when it's done, while Foldkit is declarative. You dispatch Messages describing what happened, Animation turns the lifecycle into a sequence of more Messages, and your update function reacts at each step. The payoff is that every animation state transition is in your Model, observable in DevTools, testable without a DOM, and can't run outside your update loop.

Concretely, Animation uses the [OutMessage](https://foldkit.dev/core/submodel#surfacing-facts) pattern: the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config handles `StartedLeaveAnimating` (to provide settlement detection) and `TransitionedOut` (to unmount content). It's used internally by Dialog, Menu, Popover, Listbox, and Combobox when `isAnimated` is true, and works with both CSS transitions and CSS keyframe animations.

## Why Does This Exist?

CSS animations only play when an element enters the DOM with one state and changes to another. If an element mounts with its final styles, the browser has no "before" state and nothing animates. Reliably coordinating enter and leave phases takes three pieces of machinery that are easy to get wrong.

First, enter animations need a closed state that sticks for one frame before being removed, so the browser commits it to the DOM and then sees a change. Animation handles this with a double- `requestAnimationFrame` sequence: one frame to apply `data-closed`, another to remove it and trigger the CSS animation.

Second, `transitionend` and `animationend` don't automatically flow into your update function. You could subscribe to them yourself, but that means wiring a subscription per element, filtering by selector, and managing its lifecycle alongside the state machine. Without that coordinator, there's no reliable way to know when a leave animation has finished, and therefore no way to reliably unmount content after it does. Animation emits `TransitionedOut` as the bridge: your update provides `defaultLeaveCommand`, it waits for the element’s animations to settle, and Animation tells you when the leave is complete.

Third, animating `height: auto` isn't possible with pure CSS: `auto` is not an animatable value, so height transitions normally require JavaScript DOM measurement. `animateSize: true` sidesteps this by wrapping content in a CSS grid that animates `grid-template-rows` from `0fr` to `1fr`. The structure works but requires specific DOM nesting that Animation provides for you.

Every component in the library that needs enter/leave animations (for example Dialog, Menu, Popover, Listbox, or Combobox) uses Animation internally rather than reinventing this coordination. If you need the same for your own content, Animation gives you the same machinery.

See it in an app

Check out how Animation is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/animation.ts).

## Examples

Send `Animation.Showed()` to start the enter animation and `Animation.Hid()` to start the leave animation. Style with Tailwind data-attribute selectors like `data-[closed]:opacity-0`.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Effect, Match as M, Option } from 'effect'
import { Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Animation } from '@foldkit/ui'

// Add a field to your Model for the Animation Submodel. Animation tracks
// its own visibility and lifecycle state. No need for a separate flag:
const Model = S.Struct({
  animation: Animation.Model,
  // ...your other fields
})

// In your init function, initialize the Animation Submodel with a unique id:
const init = () => [
  {
    animation: Animation.init({ id: 'content' }),
    // ...your other fields
  },
  [],
]

// Embed the Animation Message in your parent Message:
const GotAnimationMessage = m('GotAnimationMessage', {
  message: Animation.Message,
})

// At module scope, fold the OutMessage into your own Model. It signals
// lifecycle events Animation can't handle on its own. Most importantly, it
// tells you when a leave animation has started so you can provide the Command
// that listens for animation settlement. That Command's result is an Animation
// Message, so take the fold context's \`liftCommand\` and wrap it with the same
// lift the fold gives the Submodel's own Commands. Each arm returns an
// Update.Step over the parent Model, which already has the next Animation
// Model written back:
const foldAnimationOutMessage: (
  outMessage: Animation.OutMessage,
  context: Update.FoldContext<Animation.Message, Message>,
) => Update.Step<Model, Message> = (outMessage, { liftCommand }) =>
  M.value(outMessage).pipe(
    M.withReturnType<Update.Step<Model, Message>>(),
    M.tagsExhaustive({
      // Animation handles enter completion internally but hands leave
      // settlement detection to you here, because the strategy varies
      // by consumer. For example, Foldkit's Dialog just waits for CSS,
      // while its Popover races CSS against the anchor button scrolling
      // off-screen. defaultLeaveCommand is the default strategy: it
      // waits for every CSS transition and keyframe animation on the
      // element to settle, then dispatches EndedAnimation back into
      // Animation.update. Use it unless you need a custom strategy.
      StartedLeaveAnimating: () => model => [
        model,
        [liftCommand(Animation.defaultLeaveCommand(model.animation))],
      ],
      // TransitionedOut is Animation's signal that the leave has fully
      // settled (your leave Command's EndedAnimation message has been
      // processed). Return Commands for any post-animation work, for
      // example: close a native dialog, remove an entry from a list,
      // release a resource. No Commands here because animateSize keeps the
      // element mounted (collapsed to zero height) so there's nothing to
      // tear down.
      TransitionedOut: () => model => [model, []],
    }),
  )

// Update.foldChild wires the child into the parent: it runs Animation.update,
// writes the next Animation Model back, maps the Submodel's Commands into your
// Message type, and hands any OutMessage to foldOutMessage.
const foldAnimation = Update.foldChild({
  update: Animation.update,
  read: (model: Model) => Option.some(model.animation),
  write: (model, nextAnimation) =>
    evo(model, { animation: () => nextAnimation }),
  toParentMessage: message => GotAnimationMessage({ message }),
  foldOutMessage: foldAnimationOutMessage,
})

// Inside your update function's M.tagsExhaustive({...}), call the fold:
GotAnimationMessage: ({ message }) => foldAnimation(model, message)

// Inside your view function, toggle visibility by dispatching Animation.Showed()
// or Hid() wrapped in your parent Message. model.animation.isShowing is your
// source of truth for whether content is currently visible. The Animation
// view wraps your content. Data attributes drive the CSS transitions or
// keyframe animations defined in className:
const view = (h: HtmlBuilder<Message>) =>
  h.div(
    [],
    [
      h.button(
        [
          h.OnClick(
            GotAnimationMessage({
              message: model.animation.isShowing
                ? Animation.Hid()
                : Animation.Showed(),
            }),
          ),
        ],
        [model.animation.isShowing ? 'Hide' : 'Show'],
      ),
      h.submodel({
        slotId: 'content',
        model: model.animation,
        view: Animation.view,
        viewInputs: {
          animateSize: true,
          className:
            'transition duration-200 ease-out data-[closed]:opacity-0 data-[closed]:scale-95',
          content: h.p([], ['This content animates in and out.']),
        },
        toParentMessage: message => GotAnimationMessage({ message }),
      }),
    ],
  )
```

## Lifecycle

Animation drives the enter phase to completion on its own. The leave phase hands control back to the parent halfway through so the parent can decide how settlement is detected. For example, Foldkit's [Dialog](https://foldkit.dev/ui/dialog) just waits for CSS, while its [Popover](https://foldkit.dev/ui/popover) races CSS against the anchor button scrolling off-screen. The asymmetry exists because leave detection varies by consumer, while enter detection does not.

```
ENTER  (Animation drives to completion on its own)

         Showed()
            |
            ↓
   +-----------------+
   |   EnterStart    |
   +--------+--------+
            | rAF × 2
            ↓
   +-----------------+
   | EnterAnimating  |
   +--------+--------+
            | EndedAnimation (internal)
            ↓
   +-----------------+
   |      Idle       |
   +-----------------+

LEAVE  (Animation hands settlement detection to the parent)

         Hid()
            |
            ↓
   +-----------------+
   |   LeaveStart    |
   +--------+--------+
            | rAF × 2
            ↓
   +-----------------+  ← emits StartedLeaveAnimating
   | LeaveAnimating  |    parent supplies leave Command
   +--------+--------+
            | leave Command dispatches EndedAnimation
            ↓
   +-----------------+  ← emits TransitionedOut
   |      Idle       |    parent handles post-animation cleanup
   +-----------------+
```

The double-rAF timing (one frame to set the start state, another to trigger the animation) ensures browsers flush layout between phases so the CSS animation actually plays.

## Styling

Animation is headless. It only manages data attributes. You can style the lifecycle with either CSS transitions or CSS keyframe animations; the state machine advances once every animation on the element has settled.

For CSS transitions, use data-attribute selectors like `data-[closed]:opacity-0 data-[closed]:scale-95` together with a `transition` property on the element. For CSS keyframe animations, apply an `animation` shorthand scoped to `data-[enter]` or `data-[leave]`. The state machine waits for every animation on the element to settle, whether they fire `transitionend`, `animationend`, or both.

Leave animations must be finite. `animation-iteration-count: infinite` never fires `animationend`, which leaves the state machine in `LeaveAnimating` forever and the element in the DOM. Reserve infinite animations for decorative or ambient effects that don’t gate a leave phase.

The `animateSize` option uses CSS grid (`grid-template-rows: 0fr` → `1fr`) for smooth height animation without JavaScript measurement.

| Attribute | Condition |
| --- | --- |
| `data-closed` | Present at the start of enter and during leave. Target this for your hidden state styles. |
| `data-enter` | Present during the enter animation. |
| `data-leave` | Present during the leave animation. |
| `data-transition` | Present during any animation phase. |

## API Reference

### InitConfig

Configuration object passed to `Animation.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the animation instance. |
| `isShowing` | `boolean` | `false` | Initial visibility state. |

### ViewConfig

Configuration object passed to `Animation.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `model` | `Animation.Model` | — | The animation state from your parent Model. |
| `content` | `Html` | — | The content to animate in and out. |
| `animateSize` | `boolean` | `false` | Animates height collapse/expand using CSS grid. When true, the element stays in the DOM with grid-template-rows transitioning between 0fr and 1fr. |
| `className` | `string` | — | CSS class for the animation wrapper. |
| `attributes` | `ReadonlyArray<Attribute<Message>>` | — | Additional attributes for the wrapper. |
| `element` | `TagName` | `'div'` | The HTML element for the wrapper. |

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Fold the OutMessage in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `StartedLeaveAnimating` | `OutMessage` | — | Emitted when the leave animation begins. Return Animation.defaultLeaveCommand(model) from the fold, lifted with the fold context's liftCommand, to detect animation settlement. |
| `TransitionedOut` | `OutMessage` | — | Emitted when the leave animation finishes. Use this to unmount content or update your Model. |
