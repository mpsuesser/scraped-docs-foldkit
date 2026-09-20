---
url: https://foldkit.dev/api-reference/ui-tooltip
title: "Ui/Tooltip"
description: "API documentation for the Ui/Tooltip module."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

# Ui/Tooltip

## Functions

### init

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L61)

```
/** Creates an initial tooltip model from a config. Defaults to hidden. */
(config: InitConfig): Tooltip.Model
```

### triggerId

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L46)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L224)

```
/**
 * Processes a Tooltip Message and returns the next Model, optional Commands,
 *  and an optional OutMessage. `Shown`/`Hidden` fire only on `isOpen`
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
): Update.ReturnWithOutMessage<Model, Message, OutMessage>
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L55)

```
/** Configuration for creating a tooltip model with `init`. */
type InitConfig = Readonly<{
  id: string
  showDelay: Duration.Input
}>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L260)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L267)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L89)

```
/** The anchor-positioning Mount this Tooltip renders on its panel. */
const AnchorTooltip: MountDefinitionWithArgs<"AnchorTooltip", {
  anchor: Struct<{
    gap: optional<Number>
    isPlacementLocked: optional<Boolean>
    offset: optional<Number>
    padding: optional<Union<readonly [
      Number,
      Struct<{
        bottom: optionalKey<Number>
        left: optionalKey<Number>
        right: optionalKey<Number>
        top: optionalKey<Number>
      }>
    ]>>
    placement: optional<Literals<readonly ["top", "right", "bottom", "left", "top-start", "top-end", "right-start", "right-end", "bottom-start", "bottom-end", "left-start", "left-end"]>>
    portal: optional<Boolean>
  }>
  buttonId: String
}, {
  _tag: "CompletedAnchorTooltip"
}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/message.ts#L7)

```
/** Union of all messages the tooltip component can produce. */
const Message: MessageUnion<{
  BlurredTrigger: {}
  CompletedAnchorTooltip: {}
  CompletedWaitBeforeShowing: {
    version: Number
  }
  EnteredTrigger: {}
  FocusedTrigger: {}
  LeftTrigger: {}
  PressedEscape: {}
  PressedPointerOnTrigger: {
    pointerType: String
  }
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L27)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/message.ts#L31)

```
/** Union of out-messages the tooltip component can produce. */
const OutMessage: MessageUnion<{
  Hidden: {}
  Shown: {}
}>
```

### WaitBeforeShowing

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L79)

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

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L240)

```
/**
 * Reflects an externally-sourced hover show-delay onto the Model without
 *  emitting an OutMessage. Use to mirror an external config value (a user
 *  preference, a restored setting) onto the tooltip.
 */
const reflectShowDelay: Reflect<Model, Duration.Input>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tooltip/tooltip.ts#L278)

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
  anchor: Anchor.AnchorConfig
  ariaLabel: string
  ariaLabelledBy: string
  isDisabled: boolean
  toView: (render: RenderInfo) => Html
}>>
```
