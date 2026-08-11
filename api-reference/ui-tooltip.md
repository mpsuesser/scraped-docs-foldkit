---
url: https://foldkit.dev/api-reference/ui-tooltip
title: "Ui/Tooltip"
description: "API documentation for the Ui/Tooltip module."
access_date: 2026-08-11T13:49:05.518Z
current_date: 2026-08-11T13:49:05.518Z
---

# Ui/Tooltip

## Functions

### init

function

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L72)

```
/** Creates an initial tooltip model from a config. Defaults to hidden. */
(config: InitConfig): Tooltip.Model
```

### triggerId

function

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L57)

```
/**
 * Returns the bare DOM id of the tooltip trigger button, derived from the
 *  tooltip's base id. Use this to associate an external label with the
 *  trigger via a native `<label for={Tooltip.triggerId(id)}>` or an
 *  `aria-labelledby` reference.
 */
(id: string): string
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L258)

```
/**
 * Processes a tooltip message and returns the next model, commands, and
 *  an optional OutMessage. `Shown`/`Hidden` fire only on `isOpen`
 *  transitions, so consumers don't get spurious events for messages that
 *  only update hover/focus/delay state without changing visibility.
 */
(
  model: Tooltip.Model,
  message: {
    _tag: "EnteredTrigger"
  } | {
    _tag: "LeftTrigger"
  } | {
    _tag: "FocusedTrigger"
  } | {
    _tag: "BlurredTrigger"
  } | {
    _tag: "PressedEscape"
  } | {
    _tag: "PressedPointerOnTrigger"
    pointerType: string
  } | {
    _tag: "CompletedWaitBeforeShowing"
    version: number
  } | {
    _tag: "CompletedAnchorTooltip"
  }
): UpdateReturn
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L66)

```
/** Configuration for creating a tooltip model with `init`. */
type InitConfig = Readonly<{
  id: string
  showDelay: Duration.Input
}>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L288)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 *  - `trigger`: attribute bundle for the trigger element. Carries the
 *    hover/focus/keyboard handlers + ARIA `aria-describedby` linking to
 *    the panel.
 *  - `panel`: attribute bundle for the panel element. Carries the
 *    `role="tooltip"`, the anchor Mount that positions the panel via
 *    Floating UI, and a `data-open` attribute when visible.
 *  - `isVisible`: derived state. The consumer decides whether to render
 *    the panel conditionally on this.
 */
type RenderInfo = Readonly<{
  isVisible: boolean
  panel: ReadonlyArray<ChildAttribute>
  trigger: ReadonlyArray<ChildAttribute>
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L295)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  anchor: AnchorConfig
  ariaLabel: string
  ariaLabelledBy: string
  isDisabled: boolean
  toView: (render: RenderInfo) => Html
}>
```

## Constants

### AnchorTooltip

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L106)

```
/** The anchor-positioning Mount this Tooltip renders on its panel. */
const AnchorTooltip: MountDefinitionWithArgs<"AnchorTooltip", {
  anchor: Struct<{
    gap: optional<Number>
    isPlacementLocked: optional<Boolean>
    offset: optional<Number>
    padding: optional<Number>
    placement: optional<Literals<readonly ["top", "right", "bottom", "left", "top-start", "top-end", "right-start", "right-end", "bottom-start", "bottom-end", "left-start", "left-end"]>>
    portal: optional<Boolean>
  }>
  buttonId: String
}, {
  _tag: "CompletedAnchorTooltip"
}>
```

### BlurredTrigger

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L13)

```
/** Sent when focus leaves the trigger. */
const BlurredTrigger: CallableTaggedStruct<"BlurredTrigger", {}>
```

### CompletedAnchorTooltip

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L27)

```
/** Sent when the tooltip panel mounts and Floating UI has positioned it. */
const CompletedAnchorTooltip: CallableTaggedStruct<"CompletedAnchorTooltip", {}>
```

### CompletedWaitBeforeShowing

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L23)

```
/** Sent when the show-delay timer fires. */
const CompletedWaitBeforeShowing: CallableTaggedStruct<"CompletedWaitBeforeShowing", {
  version: Number
}>
```

### EnteredTrigger

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L7)

```
/** Sent when the pointer enters the tooltip trigger. */
const EnteredTrigger: CallableTaggedStruct<"EnteredTrigger", {}>
```

### FocusedTrigger

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L11)

```
/** Sent when focus enters the trigger. */
const FocusedTrigger: CallableTaggedStruct<"FocusedTrigger", {}>
```

### Hidden

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L69)

```
/** Emitted once the tooltip transitions to hidden (`isOpen` becomes false). */
const Hidden: CallableTaggedStruct<"Hidden", {}>
```

### LeftTrigger

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L9)

```
/** Sent when the pointer leaves the tooltip trigger. */
const LeftTrigger: CallableTaggedStruct<"LeftTrigger", {}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L30)

```
/** Union of all messages the tooltip component can produce. */
const Message: S.Union<[typeof EnteredTrigger, typeof LeftTrigger, typeof FocusedTrigger, typeof BlurredTrigger, typeof PressedEscape, typeof PressedPointerOnTrigger, typeof CompletedWaitBeforeShowing, typeof CompletedAnchorTooltip]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L38)

```
/** Schema for the tooltip component's state. `isOpen` is visibility; `isHovered` tracks pointer on trigger; `isFocused` tracks tooltip-affirming focus on the trigger (focus arriving without a preceding mouse press, like keyboard, touch, or pen; mouse-click-induced focus is excluded since it doesn't affirm the user wants the tooltip visible); `isDismissed` suppresses re-opening after the user dismissed the tooltip (via Escape) until they disengage (leave or blur). `showDelay` is the hover-to-show duration. `maybeLastPointerType` records the most recent pointer type that pressed the trigger, so a mouse-click-induced focus can be distinguished from other focus. */
const Model: Struct<{
  id: String
  isDismissed: Boolean
  isFocused: Boolean
  isHovered: Boolean
  isOpen: Boolean
  maybeLastPointerType: Option<String>
  pendingShowVersion: Number
  showDelay: DurationFromMillis
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L72)

```
/** Union of out-messages the tooltip component can produce. */
const OutMessage: Union<readonly [CallableTaggedStruct<"Shown", {}>, CallableTaggedStruct<"Hidden", {}>]>
```

### PressedEscape

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L15)

```
/** Sent when Escape is pressed while the tooltip is visible. */
const PressedEscape: CallableTaggedStruct<"PressedEscape", {}>
```

### PressedPointerOnTrigger

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L19)

```
/**
 * Sent when a pointer presses the trigger. Recorded so the focus that
 *  follows a mouse press can be told apart from focus that affirms the
 *  tooltip (keyboard, touch, or pen).
 */
const PressedPointerOnTrigger: CallableTaggedStruct<"PressedPointerOnTrigger", {
  pointerType: String
}>
```

### Shown

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/message.ts#L66)

```
/**
 * Emitted once the tooltip transitions to visible (`isOpen` becomes true).
 *  Consumers typically use this for analytics, instrumentation, or to
 *  coordinate with other transient UI.
 */
const Shown: CallableTaggedStruct<"Shown", {}>
```

### WaitBeforeShowing

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L96)

```
/**
 * Waits for the tooltip's show delay before emitting
 *  `CompletedWaitBeforeShowing`.
 */
const WaitBeforeShowing: CommandDefinitionWithArgs<"WaitBeforeShowing", {
  delay: DurationFromMillis
  version: Number
}, Effect<{
  _tag: "CompletedWaitBeforeShowing"
  version: number
}, never, never>>
```

### reflectShowDelay

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L270)

```
/**
 * Reflects an externally-sourced hover show-delay onto the model without
 *  emitting an OutMessage. Use to mirror an external config value (a user
 *  preference, a restored setting) onto the tooltip.
 */
const reflectShowDelay: Reflect<Model, Duration.Input>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/b2cf4e3359f2895c69f49aa286ce5584268d2398/packages/ui/src/tooltip/tooltip.ts#L306)

```
/**
 * Renders a headless tooltip with an anchored non-interactive panel.
 *  Shows on hover (after delay) or focus (from keyboard, touch, or pen;
 *  mouse-click focus is excluded). Hides on leave, blur, or Escape.
 */
const view: SubmodelView<Tooltip.Model, {
  _tag: "EnteredTrigger"
} | {
  _tag: "LeftTrigger"
} | {
  _tag: "FocusedTrigger"
} | {
  _tag: "BlurredTrigger"
} | {
  _tag: "PressedEscape"
} | {
  _tag: "PressedPointerOnTrigger"
  pointerType: string
} | {
  _tag: "CompletedWaitBeforeShowing"
  version: number
} | {
  _tag: "CompletedAnchorTooltip"
}, Readonly<{
  anchor: {
    gap: number
    isPlacementLocked: boolean
    offset: number
    padding: number
    placement: "top" | "right" | "bottom" | "left" | "top-start" | "top-end" | "right-start" | "right-end" | "bottom-start" | "bottom-end" | "left-start" | "left-end"
    portal: boolean
  }
  ariaLabel: string
  ariaLabelledBy: string
  isDisabled: boolean
  toView: (render: RenderInfo) => Html
}>>
```
