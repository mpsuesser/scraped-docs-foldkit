---
url: https://foldkit.dev/api-reference/ui-hover-intent
title: "Ui/HoverIntent"
description: "API documentation for the Ui/HoverIntent module."
access_date: 2026-09-07T07:29:31.695Z
current_date: 2026-09-07T07:29:31.695Z
---

# Ui/HoverIntent

## Functions

### close

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L218)

```
/**
 * Programmatically closes HoverIntent immediately, invalidating pending
 *  transitions and suppressing reopening until the trigger disengages.
 */
(model: HoverIntent.Model): UpdateReturn
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L39)

```
/** Creates a HoverIntent Model. Pointer entry opens after 200 milliseconds and pointer departure closes after a 300-millisecond grace period by default. Focus entry opens immediately, and focus departure closes without that pointer grace period. */
(config: InitConfig): HoverIntent.Model
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L228)

```
/** Processes a HoverIntent Message and returns the next Model, optional Commands, and an optional OutMessage. */
(
  model: HoverIntent.Model,
  message: {
    _tag: "BlurredPanel"
  } | {
    _tag: "EnteredTrigger"
  } | {
    _tag: "LeftTrigger"
  } | {
    _tag: "EnteredPanel"
  } | {
    _tag: "LeftPanel"
  } | {
    _tag: "FocusedTrigger"
  } | {
    _tag: "BlurredTrigger"
  } | {
    _tag: "FocusedPanel"
  } | {
    _tag: "PressedEscape"
    source: "Trigger" | "Panel"
  } | {
    _tag: "CompletedWaitBeforeOpening"
    version: number
  } | {
    _tag: "CompletedWaitBeforeClosing"
    version: number
  }
): UpdateReturn
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L33)

```
/** Configuration for creating a HoverIntent Model. */
type InitConfig = Readonly<{
  closeDelay: Duration.Input
  openDelay: Duration.Input
}>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L278)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 * - `trigger`: event attributes for the element that starts intent.
 * - `panel`: event attributes for the element that remains open while hovered or focused.
 * - `isVisible`: whether the consumer should render its panel.
 */
type RenderInfo = Readonly<{
  isVisible: boolean
  panel: ReadonlyArray<ChildAttribute>
  trigger: ReadonlyArray<ChildAttribute>
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L285)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  focusTriggerSelector: string
  toView: (render: RenderInfo) => Html
}>
```

## Constants

### Message

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/message.ts#L7)

```
/** Union of all Messages the HoverIntent component can produce. */
const Message: MessageUnion<{
  BlurredPanel: {}
  BlurredTrigger: {}
  CompletedWaitBeforeClosing: {
    version: Number
  }
  CompletedWaitBeforeOpening: {
    version: Number
  }
  EnteredPanel: {}
  EnteredTrigger: {}
  FocusedPanel: {}
  FocusedTrigger: {}
  LeftPanel: {}
  LeftTrigger: {}
  PressedEscape: {
    source: Literals<readonly ["Trigger", "Panel"]>
  }
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L16)

```
/** Schema for HoverIntent state. It tracks pointer and focus engagement over a trigger and its panel, visibility, delay timers, and Escape dismissal. */
const Model: Struct<{
  closeDelay: DurationFromMillis
  isDismissed: Boolean
  isOpen: Boolean
  isPanelHovered: Boolean
  isTriggerHovered: Boolean
  maybeFocusLocation: Option<Literals<readonly ["Trigger", "Panel"]>>
  openDelay: DurationFromMillis
  pendingCloseVersion: Number
  pendingOpenVersion: Number
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/message.ts#L39)

```
/** Union of visibility-transition OutMessages emitted by HoverIntent. */
const OutMessage: MessageUnion<{
  Closed: {}
  Opened: {}
}>
```

### WaitBeforeClosing

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L70)

```
/** Waits before closing, then emits the version that scheduled the wait. */
const WaitBeforeClosing: CommandDefinitionWithArgs<"WaitBeforeClosing", {
  delay: DurationFromMillis
  version: Number
}, Effect<{
  _tag: "CompletedWaitBeforeClosing"
  version: number
}, never, never>>
```

### WaitBeforeOpening

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L60)

```
/** Waits before opening, then emits the version that scheduled the wait. */
const WaitBeforeOpening: CommandDefinitionWithArgs<"WaitBeforeOpening", {
  delay: DurationFromMillis
  version: Number
}, Effect<{
  _tag: "CompletedWaitBeforeOpening"
  version: number
}, never, never>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/hoverIntent/hoverIntent.ts#L292)

```
/** Renders headless HoverIntent event bundles. It owns no markup, ARIA semantics, positioning, or styling. */
const view: SubmodelView<HoverIntent.Model, {
  _tag: "BlurredPanel"
} | {
  _tag: "EnteredTrigger"
} | {
  _tag: "LeftTrigger"
} | {
  _tag: "EnteredPanel"
} | {
  _tag: "LeftPanel"
} | {
  _tag: "FocusedTrigger"
} | {
  _tag: "BlurredTrigger"
} | {
  _tag: "FocusedPanel"
} | {
  _tag: "PressedEscape"
  source: "Trigger" | "Panel"
} | {
  _tag: "CompletedWaitBeforeOpening"
  version: number
} | {
  _tag: "CompletedWaitBeforeClosing"
  version: number
}, Readonly<{
  focusTriggerSelector: string
  toView: (render: RenderInfo) => Html
}>>
```
