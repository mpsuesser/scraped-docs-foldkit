---
url: https://foldkit.dev/api-reference/ui-animation
title: "Ui/Animation"
description: "API documentation for the Ui/Animation module."
access_date: 2026-08-16T21:27:58.480Z
current_date: 2026-08-16T21:27:58.480Z
---

# Ui/Animation

## Functions

### defaultLeaveCommand

function

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/update.ts#L124)

```
/** Creates the standard leave-phase command that waits for CSS animations on the element to settle. Use this when handling the `StartedLeaveAnimating` OutMessage for components that don't need custom leave behavior. */
(model: Animation.Model): Command.Command<Message>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L71)

```
/** Creates an initial animation model from a config. Defaults to hidden. */
(config: InitConfig): Animation.Model
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/update.ts#L48)

```
/** Processes an animation message and returns the next model, commands, and optional OutMessage. */
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

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L65)

```
/** Configuration for creating an animation model with `init`. */
type InitConfig = Readonly<{
  id: string
  isShowing: boolean
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/index.ts#L45)

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

### CompletedWaitForPaint

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L34)

```
/** Sent internally when a double-rAF completes, advancing the lifecycle to its animating phase. */
const CompletedWaitForPaint: CallableTaggedStruct<"CompletedWaitForPaint", {}>
```

### EndedAnimation

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L36)

```
/** Sent internally when all CSS animations on the element have settled. Covers both CSS transitions and CSS keyframe animations. */
const EndedAnimation: CallableTaggedStruct<"EndedAnimation", {}>
```

### Hid

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L32)

```
/** Sent when the animation should leave (become hidden). Starts the leave sequence. */
const Hid: CallableTaggedStruct<"Hid", {}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L39)

```
/** Union of all messages the animation component can produce. */
const Message: S.Union<[typeof Showed, typeof Hid, typeof CompletedWaitForPaint, typeof EndedAnimation]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L19)

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

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L59)

```
const OutMessage: Union<readonly [CallableTaggedStruct<"StartedLeaveAnimating", {}>, CallableTaggedStruct<"TransitionedOut", {}>]>
```

### Showed

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L30)

```
/** Sent when the animation should enter (become visible). Starts the enter sequence. */
const Showed: CallableTaggedStruct<"Showed", {}>
```

### StartedLeaveAnimating

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L55)

```
/** Sent to the parent when the leave sequence advances to LeaveAnimating. The parent is responsible for providing the command that detects when the leave animation completes (e.g. WaitForAnimationSettled or a racing command). Use `defaultLeaveCommand` for the standard behavior. */
const StartedLeaveAnimating: CallableTaggedStruct<"StartedLeaveAnimating", {}>
```

### TransitionState

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L7)

```
/** Schema for the animation lifecycle state, tracking enter/leave phases. */
const TransitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
```

### TransitionedOut

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/schema.ts#L57)

```
/** Sent to the parent when the leave animation completes. The parent can use this to unmount content or update its own state. */
const TransitionedOut: CallableTaggedStruct<"TransitionedOut", {}>
```

### WaitForAnimationSettled

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/update.ts#L35)

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

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/update.ts#L30)

```
/** Waits for paint via double-rAF before the enter/leave lifecycle advances. */
const WaitForPaint: CommandDefinitionNoArgs<"WaitForPaint", Effect<{
  _tag: "CompletedWaitForPaint"
}, never, never>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/6fd31988a1c18fb9d006d82fe3b82ae3fcaefe61/packages/ui/src/animation/index.ts#L65)

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
