---
url: https://foldkit.dev/api-reference/ui-animation
title: "Ui/Animation"
description: "API documentation for the Ui/Animation module."
access_date: 2026-08-31T07:29:25.100Z
current_date: 2026-08-31T07:29:25.100Z
---

# Ui/Animation

## Functions

### defaultLeaveCommand

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/update.ts#L117)

```
/** Creates the standard leave-phase command that waits for CSS animations on the element to settle. Use this when handling the `StartedLeaveAnimating` OutMessage for components that don't need custom leave behavior. */
(model: Animation.Model): Command.Command<Message>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/schema.ts#L58)

```
/** Creates an initial animation model from a config. Defaults to hidden. */
(config: InitConfig): Animation.Model
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/update.ts#L46)

```
/**
 * Processes an Animation Message and returns the next Model, optional
 *  Commands, and an optional OutMessage. `Showed` and `Hid` start a transition
 *  but cannot finish one, so direct calls with either Message return a plain
 *  update result.
 */
(
  model: Animation.Model,
  message: {
    _tag: "Showed"
  } | {
    _tag: "Hid"
  }
): Update.Return<Model, Message>

(
  model: Animation.Model,
  message: {
    _tag: "Showed"
  } | {
    _tag: "Hid"
  } | {
    _tag: "CompletedWaitForPaint"
  } | {
    _tag: "EndedAnimation"
  }
): UpdateReturn
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/schema.ts#L52)

```
/** Configuration for creating an animation model with `init`. */
type InitConfig = Readonly<{
  id: string
  isShowing: boolean
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/index.ts#L21)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  animateSize: boolean
  attributes: ReadonlyArray<ChildAttribute>
  className: string
  content: Html
  element: TagName
}>
```

## Constants

### Message

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/schema.ts#L30)

```
/** Union of all messages the animation component can produce. */
const Message: MessageUnion<{
  CompletedWaitForPaint: {}
  EndedAnimation: {}
  Hid: {}
  Showed: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/schema.ts#L19)

```
/** Schema for the animation component's state, tracking its unique ID, visibility intent, and lifecycle phase. */
const Model: Struct<{
  id: String
  isShowing: Boolean
  transitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/schema.ts#L43)

```
const OutMessage: MessageUnion<{
  StartedLeaveAnimating: {}
  TransitionedOut: {}
}>
```

### TransitionState

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/schema.ts#L7)

```
/** Schema for the animation lifecycle state, tracking enter/leave phases. */
const TransitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
```

### WaitForAnimationSettled

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/update.ts#L30)

```
/** Waits for all CSS animations on the element to settle. Covers both CSS transitions and CSS keyframe animations. */
const WaitForAnimationSettled: CommandDefinitionWithArgs<"WaitForAnimationSettled", {
  id: String
}, Effect<{
  _tag: "EndedAnimation"
}, never, never>>
```

### WaitForPaint

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/update.ts#L25)

```
/** Waits for paint via double-rAF before the enter/leave lifecycle advances. */
const WaitForPaint: CommandDefinitionNoArgs<"WaitForPaint", Effect<{
  _tag: "CompletedWaitForPaint"
}, never, never>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/animation/index.ts#L41)

```
/**
 * Renders a headless animation wrapper that coordinates CSS transitions and
 *  CSS keyframe animations via data attributes.
 * 
 *  Data attributes reflect the current lifecycle phase:
 *  - `data-closed`: element is in its hidden/initial state
 *  - `data-enter`: enter animation is active
 *  - `data-leave`: leave animation is active
 *  - `data-transition`: any animation is active
 */
const view: SubmodelView<Animation.Model, {
  _tag: "Showed"
} | {
  _tag: "Hid"
} | {
  _tag: "CompletedWaitForPaint"
} | {
  _tag: "EndedAnimation"
}, Readonly<{
  animateSize: boolean
  attributes: readonly Array<Readonly<{
    __childAttribute: true
    attribute: unknown
    boundaryMappers: readonly Array<(message: unknown) => unknown>
    dispatch: DispatchSync
    resolveUnmount: (message: unknown) => () => void
  }>>
  className: string
  content: Html
  element: TagName
}>>
```
