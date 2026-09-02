---
url: https://foldkit.dev/ui/animation
title: "Animation"
description: "Coordinates CSS enter/leave animations via a state machine and data attributes. Works with both CSS transitions and keyframe animations."
access_date: 2026-09-02T07:05:07.578Z
current_date: 2026-09-02T07:05:07.578Z
---

## Overview

Animation coordinates CSS enter and leave phases with a state machine and data attributes. A parent invokes the child-owned `show`, `hide`, or `toggle` update capability through `Update.foldChildStep`; Animation applies its internal `Showed` or `Hid` fact, records each lifecycle phase in its Model, and exposes the resulting Commands through the fold. The transitions stay visible in DevTools and can be tested through update without waiting for a browser animation.

Animation uses the [OutMessage](https://foldkit.dev/core/submodel#surfacing-facts) pattern. The `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config handles `StartedLeaveAnimating` by providing a Command that detects settlement, then handles `TransitionedOut` when post-animation cleanup can begin. Dialog, Menu, Popover, Listbox, and Combobox use the same Submodel internally when `isAnimated` is true.

## Lifecycle Coordination

CSS can animate between two rendered styles, but an application still has to stage those styles across paints and keep a leaving element mounted until its animations finish. Animation coordinates three parts of that lifecycle.

First, an entering element renders with `data-closed`. Animation waits for paint before removing that attribute, so the browser sees the change and starts the CSS animation.

Second, browser animation settlement does not enter update on its own. `defaultLeaveCommand` waits for every transition and keyframe animation reported by the element's Web Animations API to settle, then returns `EndedAnimation`. Animation responds with `TransitionedOut`, which tells the parent that it can unmount content or perform other cleanup.

Third, `animateSize: true` supplies the nested CSS grid structure needed to animate content between collapsed and expanded rows without measuring its height in JavaScript. It keeps the hidden content mounted at zero height instead of removing it.

Use Animation directly when your own content needs the same lifecycle. Components such as Dialog, Menu, Popover, Listbox, and Combobox already provide it through their `isAnimated` option.

See it in an app

Check out how Animation is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/animation.ts).

## Examples

Fold `Animation.show` with `Update.foldChildStep` to start the enter animation and fold `Animation.hide` to start the leave animation. Style with Tailwind data-attribute selectors like `data-[closed]:opacity-0`.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Option, Schema } from 'effect'
import { Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Animation } from '@foldkit/ui'

// Add a field to your Model for the Animation Submodel. Animation tracks
// its own visibility and lifecycle state. No need for a separate flag:
const Model = Schema.Struct({
  animation: Animation.Model,
  // ...your other fields
})

// In your init function, initialize the Animation Submodel with a unique id:
const init = () => ({
  model: {
    animation: Animation.init({ id: 'content' }),
    // ...your other fields
  },
})

// Embed the Animation Message in your parent Message:
const Message = defineMessageUnion({
  GotAnimationMessage: { message: Animation.Message },
  ClickedToggleAnimation: {},
})
type Message = typeof Message.Type

// At module scope, fold the OutMessage into your own Model. It signals
// lifecycle events Animation can't handle on its own. Most importantly, it
// tells you when a leave animation has started so you can provide the Command
// that listens for animation settlement. That Command's result is an Animation
// Message, so use \`liftCommand\` from the fold context to lift it into the
// parent Message type. Each arm returns an Update.Step over the parent Model,
// which already has the next Animation Model written back:
const foldAnimationOutMessage = (
  outMessage: Animation.OutMessage,
  { liftCommand }: Update.FoldContext<Animation.Message, Message>,
) =>
  Animation.OutMessage.match<Update.Step<Model, Message>>(outMessage, {
    // Animation handles enter completion internally but hands leave
    // settlement detection to you here, because the strategy varies
    // by consumer. For example, Foldkit's Dialog just waits for CSS,
    // while its Popover races CSS against the anchor button scrolling
    // off-screen. defaultLeaveCommand is the default strategy: it
    // waits for every CSS transition and keyframe animation on the
    // element to settle, then dispatches EndedAnimation back into
    // Animation.update. Use it unless you need a custom strategy.
    StartedLeaveAnimating: () => model => ({
      model,
      commands: [liftCommand(Animation.defaultLeaveCommand(model.animation))],
    }),
    // TransitionedOut is Animation's signal that the leave has fully
    // settled (your leave Command's EndedAnimation message has been
    // processed). Return Commands for any post-animation work, for
    // example: close a native dialog, remove an entry from a list,
    // release a resource. No Commands here because animateSize keeps the
    // element mounted (collapsed to zero height) so there's nothing to
    // tear down.
    TransitionedOut: () => model => ({ model }),
  })

// Update.foldChild wires the child into the parent: it runs Animation.update,
// writes the next Animation Model back, maps the Submodel's Commands into your
// Message type, and hands any OutMessage to foldOutMessage.
const foldAnimation = Update.foldChild({
  update: Animation.update,
  read: (model: Model) => Option.some(model.animation),
  write: (model, nextAnimation) =>
    evo(model, { animation: () => nextAnimation }),
  toParentMessage: message => Message.GotAnimationMessage({ message }),
  foldOutMessage: foldAnimationOutMessage,
})

// Programmatic child capabilities take the child Model and return its next
// Model and Commands. foldChildStep integrates that result without exposing
// the internal Showed or Hid Message constructors to the parent.
const foldAnimationShow = Update.foldChildStep({
  update: Animation.show,
  read: (model: Model) => Option.some(model.animation),
  write: (model, nextAnimation) =>
    evo(model, { animation: () => nextAnimation }),
  toParentMessage: message => Message.GotAnimationMessage({ message }),
})

const foldAnimationHide = Update.foldChildStep({
  update: Animation.hide,
  read: (model: Model) => Option.some(model.animation),
  write: (model, nextAnimation) =>
    evo(model, { animation: () => nextAnimation }),
  toParentMessage: message => Message.GotAnimationMessage({ message }),
})

// In the corresponding Message.match handlers, route child Messages through
// the regular fold and parent-owned actions through the child capabilities:
GotAnimationMessage: ({ message }) => foldAnimation(model, message)
ClickedToggleAnimation: () =>
  model.animation.isShowing
    ? foldAnimationHide(model)
    : foldAnimationShow(model)

// Inside your view function, dispatch a parent-owned fact. The parent update
// invokes the child capability. model.animation.isShowing is your source of
// truth for whether content is currently visible. The Animation view wraps
// your content. Data attributes drive the CSS transitions or keyframe
// animations defined in className:
const view = (h: HtmlBuilder<Message>) =>
  h.div(
    [],
    [
      h.button(
        [h.OnClick(Message.ClickedToggleAnimation())],
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
        toParentMessage: message => Message.GotAnimationMessage({ message }),
      }),
    ],
  )
```

## Lifecycle

Animation drives the enter phase to completion on its own. The leave phase hands control back to the parent halfway through so the parent can decide how settlement is detected. For example, Foldkit's [Dialog](https://foldkit.dev/ui/dialog) just waits for CSS, while its [Popover](https://foldkit.dev/ui/popover) races CSS against the anchor button scrolling off-screen. The asymmetry exists because leave detection varies by consumer, while enter detection does not.

Internally, the `show`, `hide`, and `toggle` capabilities apply the `Showed` or `Hid` Message according to the current Animation Model and requested transition.

```
ENTER                                LEAVE
Animation drives completion          Parent detects settlement

show()                                hide()
   |                                    |
   v                                    v
EnterStart                           LeaveStart
   | rAF x 2                            | rAF x 2
   v                                    v
EnterAnimating                       LeaveAnimating
   | EndedAnimation (internal)          +-> emits StartedLeaveAnimating
   v                                    |   parent supplies leave Command
 Idle                                   |
                                        |   Command dispatches
                                        |   EndedAnimation
                                        v
                                       Idle
                                        +-> emits TransitionedOut
                                            parent handles cleanup
```

The double-rAF timing (one frame to set the start state, another to trigger the animation) ensures browsers flush layout between phases so the CSS animation actually plays.

## Styling

Animation is headless. You style its lifecycle with CSS transitions or CSS keyframe animations, and the state machine advances after every animation returned by `element.getAnimations()` has settled. With `animateSize: true`, Animation also adds a fixed 200ms CSS grid-row transition and the wrappers it requires.

For CSS transitions, use data-attribute selectors like `data-[closed]:opacity-0 data-[closed]:scale-95` together with a `transition` property on the element. For CSS keyframe animations, apply an `animation` shorthand scoped to `data-[enter]` or `data-[leave]`. The state machine waits for every animation returned by the element's `getAnimations()` call to settle.

Leave animations must be finite. `animation-iteration-count: infinite` never fires `animationend`, which leaves the state machine in `LeaveAnimating` forever and the element in the DOM. Reserve infinite animations for decorative or ambient effects that don’t gate a leave phase.

The `animateSize` option uses CSS grid (`grid-template-rows: 0fr` → `1fr`) for smooth height animation without JavaScript measurement.

| Attribute | Condition |
| --- | --- |
| `data-closed` | Present during `EnterStart` and `LeaveAnimating`. Target this for hidden-state styles. |
| `data-enter` | Present during the enter animation. |
| `data-leave` | Present during the leave animation. |
| `data-transition` | Present during any animation phase. |

## API Reference

### toggle

`Animation.toggle(model)` returns the Animation update result that starts the enter or leave lifecycle according to `model.isShowing`. Use it as the `update` entry point of `Update.foldChildStep` when the parent does not need to choose a directional capability.

### InitConfig

Configuration object passed to `Animation.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the animation instance. |
| `isShowing` | `boolean` | `false` | Initial visibility state. |

### show and hide

`Animation.show(model)` and `Animation.hide(model)` return the next Animation Model and any Commands. Fold these child entry points into the parent with `Update.foldChildStep`; do not construct `Animation.Message.Showed()` or `Animation.Message.Hid()` from the parent.

### ViewConfig

Configuration object passed to `Animation.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `model` | `Animation.Model` | — | The animation state from your parent Model. |
| `content` | `Html` | — | The content to animate in and out. |
| `animateSize` | `boolean` | `false` | Animates height collapse/expand using CSS grid. When true, the element stays in the DOM with grid-template-rows transitioning between 0fr and 1fr. |
| `className` | `string` | — | CSS class for the animation wrapper. |
| `attributes` | `ReadonlyArray<Attribute<Message>>` | — | Additional attributes for the wrapper. |
| `element` | `Exclude<TagName, 'textarea'>` | `'div'` | The HTML element for the wrapper. Textarea is excluded because the wrapper renders children. |

### OutMessage

Messages emitted to the parent through the optional `outMessage` field. Fold the OutMessage in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `StartedLeaveAnimating` | `OutMessage` | — | Emitted when the leave animation begins. Return Animation.defaultLeaveCommand(model) from the fold, lifted with the fold context's liftCommand, to detect animation settlement. |
| `TransitionedOut` | `OutMessage` | — | Emitted when the leave animation finishes. Use this to unmount content or update your Model. |
