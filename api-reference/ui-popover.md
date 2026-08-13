---
url: https://foldkit.dev/api-reference/ui-popover
title: "Ui/Popover"
description: "API documentation for the Ui/Popover module."
access_date: 2026-08-13T05:04:07.016Z
current_date: 2026-08-13T05:04:07.016Z
---

# Ui/Popover

## Functions

### buttonId

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L189)

```
/**
 * Returns the bare DOM id of the popover trigger button, derived from the
 *  popover's base id. Use this to associate an external label with the
 *  trigger via a native `<label for={Popover.buttonId(id)}>` or an
 *  `aria-labelledby` reference.
 */
(id: string): string
```

### close

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L477)

```
/**
 * Programmatically closes the popover. When it was open, updates the Model
 *  and returns focus and modal Commands plus a `Closed` OutMessage. When it
 *  was already closed, it is a no-op: no Commands and no OutMessage.
 */
(model: Popover.Model): UpdateReturn
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L164)

```
/** Creates an initial popover model from a config. Defaults to closed. */
(config: InitConfig): Popover.Model
```

### open

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L471)

```
/**
 * Programmatically opens the popover, updating the model and returning
 *  focus and modal commands plus an `Opened` OutMessage.
 */
(model: Popover.Model): UpdateReturn
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L288)

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L156)

```
/** Configuration for creating a popover model with `init`. `isAnimated` enables animation coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). `contentFocus` hands focus ownership to the consumer. The panel is not focusable and does not close on blur, so the consumer must focus a descendant on open and close the popover on its own blur rules (default `false`). */
type InitConfig = Readonly<{
  contentFocus: boolean
  id: string
  isAnimated: boolean
  isModal: boolean
}>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L494)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 *  - `button`: attribute bundle for the trigger button.
 *  - `panel`: attribute bundle for the floating panel. Includes the
 *    anchor Mount that positions the panel via Floating UI, ARIA
 *    linkage to the button, and panel keydown/blur handlers.
 *  - `backdrop`: attribute bundle for the modal backdrop. Includes the
 *    portal Mount that moves the backdrop to document.body. The
 *    backdrop's OnClick closes the popover.
 *  - `isVisible`: derived from `isOpen` and the Animation
 *    `transitionState`. The consumer renders the panel + backdrop only
 *    while this is true.
 */
type RenderInfo = Readonly<{
  backdrop: ReadonlyArray<ChildAttribute>
  button: ReadonlyArray<ChildAttribute>
  isVisible: boolean
  panel: ReadonlyArray<ChildAttribute>
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L502)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  anchor: AnchorConfig
  ariaLabel: string
  ariaLabelledBy: string
  focusSelector: string
  isDisabled: boolean
  toView: (render: RenderInfo) => Html
}>
```

## Constants

### AnchorPopover

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L425)

```
/**
 * The anchor-positioning Mount this Popover renders on its panel. Exposed so
 *  Scene tests can call `Scene.Mount.resolve(AnchorPopover, CompletedAnchorPopover())`
 *  to acknowledge the mount produced by the rendered panel.
 */
const AnchorPopover: MountDefinitionWithArgs<"AnchorPopover", {
  anchor: Struct<{
    gap: optional<Number>
    isPlacementLocked: optional<Boolean>
    offset: optional<Number>
    padding: optional<Number>
    placement: optional<Literals<readonly ["top", "right", "bottom", "left", "top-start", "top-end", "right-start", "right-end", "bottom-start", "bottom-end", "left-start", "left-end"]>>
    portal: optional<Boolean>
  }>
  buttonId: String
  focusSelector: optional<String>
}, {
  _tag: "CompletedAnchorPopover"
}>
```

### BlurredPanel

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L58)

```
/** Sent when the popover panel loses focus. Does NOT return focus to the button. */
const BlurredPanel: CallableTaggedStruct<"BlurredPanel", {}>
```

### Closed

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L142)

```
/** Sent to the parent after the popover transitions to its closed state. */
const Closed: CallableTaggedStruct<"Closed", {}>
```

### CompletedAnchorPopover

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L81)

```
/** Sent when the popover panel mounts and Floating UI has positioned it. Update no-ops; the side effect is the act of positioning, surfaced for DevTools observability. */
const CompletedAnchorPopover: CallableTaggedStruct<"CompletedAnchorPopover", {}>
```

### CompletedFocusButton

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L67)

```
/** Sent when the focus-button command completes after closing. */
const CompletedFocusButton: CallableTaggedStruct<"CompletedFocusButton", {}>
```

### CompletedFocusPanel

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L65)

```
/** Sent when the focus-panel command completes after opening the popover. */
const CompletedFocusPanel: CallableTaggedStruct<"CompletedFocusPanel", {}>
```

### CompletedInertOthers

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L73)

```
/** Sent when the inert-others command completes. */
const CompletedInertOthers: CallableTaggedStruct<"CompletedInertOthers", {}>
```

### CompletedLockScroll

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L69)

```
/** Sent when the scroll lock command completes. */
const CompletedLockScroll: CallableTaggedStruct<"CompletedLockScroll", {}>
```

### CompletedPortalPopoverBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L83)

```
/** Sent when the popover backdrop mounts and is portaled to the document body. Update no-ops; surfaces the portal side effect for DevTools. */
const CompletedPortalPopoverBackdrop: CallableTaggedStruct<"CompletedPortalPopoverBackdrop", {}>
```

### CompletedRestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L75)

```
/** Sent when the restore-inert command completes. */
const CompletedRestoreInert: CallableTaggedStruct<"CompletedRestoreInert", {}>
```

### CompletedUnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L71)

```
/** Sent when the scroll unlock command completes. */
const CompletedUnlockScroll: CallableTaggedStruct<"CompletedUnlockScroll", {}>
```

### DetectMovementOrAnimationEnd

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L245)

```
/** Detects whether the popover button moved or the leave animation ended. Whichever comes first; both outcomes signal the Animation submodel that leave is complete. */
const DetectMovementOrAnimationEnd: CommandDefinitionWithArgs<"DetectMovementOrAnimationEnd", {
  id: String
}, Effect<{
  _tag: "GotAnimationMessage"
  message: {
    _tag: "Showed"
  } | {
    _tag: "Hid"
  } | {
    _tag: "CompletedWaitForPaint"
  } | {
    _tag: "EndedAnimation"
  }
}, never, never>>
```

### FocusButton

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L235)

```
/** Moves focus back to the popover button after closing. */
const FocusButton: CommandDefinitionWithArgs<"FocusButton", {
  id: String
}, Effect<{
  _tag: "CompletedFocusButton"
}, never, never>>
```

### FocusPanel

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L225)

```
/** Moves focus to the popover panel after opening. */
const FocusPanel: CommandDefinitionWithArgs<"FocusPanel", {
  id: String
}, Effect<{
  _tag: "CompletedFocusPanel"
}, never, never>>
```

### GotAnimationMessage

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L87)

```
/** Wraps an Animation submodel message for delegation. */
const GotAnimationMessage: CallableTaggedStruct<"GotAnimationMessage", {
  message: Union<[CallableTaggedStruct<"Showed", {}>, CallableTaggedStruct<"Hid", {}>, CallableTaggedStruct<"CompletedWaitForPaint", {}>, CallableTaggedStruct<"EndedAnimation", {}>]>
}>
```

### IgnoredMouseClick

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L77)

```
/** Sent when a mouse click on the button is ignored because pointer-down already handled the toggle. */
const IgnoredMouseClick: CallableTaggedStruct<"IgnoredMouseClick", {}>
```

### InertOthers

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L209)

```
/** Marks all elements outside the popover as inert for modal behavior. */
const InertOthers: CommandDefinitionWithArgs<"InertOthers", {
  id: String
}, Effect<{
  _tag: "CompletedInertOthers"
}, never, never>>
```

### LockScroll

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L199)

```
/** Prevents page scrolling while the popover is open in modal mode. */
const LockScroll: CommandDefinitionNoArgs<"LockScroll", Effect<{
  _tag: "CompletedLockScroll"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L92)

```
/** Union of all messages the popover component can produce. */
const Message: S.Union<[typeof RequestedOpen, typeof RequestedClose, typeof BlurredPanel, typeof PressedPointerOnButton, typeof CompletedFocusPanel, typeof CompletedFocusButton, typeof CompletedLockScroll, typeof CompletedUnlockScroll, typeof CompletedInertOthers, typeof CompletedRestoreInert, typeof IgnoredMouseClick, typeof SuppressedSpaceScroll, typeof CompletedAnchorPopover, typeof CompletedPortalPopoverBackdrop, typeof GotAnimationMessage]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L39)

```
/** Schema for the popover component's state, tracking open/closed status and animation lifecycle. */
const Model: Struct<{
  animation: Struct<{
    id: String
    isShowing: Boolean
    transitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
  }>
  contentFocus: Boolean
  id: String
  isAnimated: Boolean
  isModal: Boolean
  isOpen: Boolean
  maybeLastButtonPointerType: Option<String>
}>
```

### Opened

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L140)

```
/** Sent to the parent after the popover transitions to its open state. Fires once `update` has processed `RequestedOpen` and `isOpen` reflects the new state. */
const Opened: CallableTaggedStruct<"Opened", {}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L145)

```
/** Union of out-messages the popover component can produce. Parents reacting to open/close transitions (e.g. to reset related state, fire analytics) read this from the third element of `update`'s return tuple. */
const OutMessage: Union<readonly [CallableTaggedStruct<"Opened", {}>, CallableTaggedStruct<"Closed", {}>]>
```

### PortalPopoverBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L456)

```
/**
 * The backdrop-portaling Mount this Popover renders. Exposed so Scene tests can
 *  call `Scene.Mount.resolve(PortalPopoverBackdrop, CompletedPortalPopoverBackdrop())` to
 *  acknowledge the mount produced by the rendered backdrop.
 */
const PortalPopoverBackdrop: MountDefinitionNoArgs<"PortalPopoverBackdrop", {
  _tag: "CompletedPortalPopoverBackdrop"
}>
```

### PressedPointerOnButton

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L60)

```
/** Sent when the user presses a pointer device on the popover button. Records pointer type and toggles for mouse. */
const PressedPointerOnButton: CallableTaggedStruct<"PressedPointerOnButton", {
  button: Number
  pointerType: String
}>
```

### RequestedClose

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L56)

```
/** Sent when the popover should close via Escape key or backdrop click. Returns focus to the button. */
const RequestedClose: CallableTaggedStruct<"RequestedClose", {}>
```

### RequestedOpen

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L54)

```
/** Sent when the popover should open via button click or keyboard activation. */
const RequestedOpen: CallableTaggedStruct<"RequestedOpen", {}>
```

### RestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L218)

```
/** Removes the inert attribute from elements outside the popover. */
const RestoreInert: CommandDefinitionWithArgs<"RestoreInert", {
  id: String
}, Effect<{
  _tag: "CompletedRestoreInert"
}, never, never>>
```

### SuppressedSpaceScroll

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L79)

```
/** Sent when a Space key-up is captured to prevent page scrolling. */
const SuppressedSpaceScroll: CallableTaggedStruct<"SuppressedSpaceScroll", {}>
```

### UnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L204)

```
/** Re-enables page scrolling after the popover closes. */
const UnlockScroll: CommandDefinitionNoArgs<"UnlockScroll", Effect<{
  _tag: "CompletedUnlockScroll"
}, never, never>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/popover/index.ts#L512)

```
/** Renders a headless popover with a trigger button and a floating panel. */
const view: SubmodelView<Popover.Model, {
  _tag: "RequestedOpen"
} | {
  _tag: "RequestedClose"
} | {
  _tag: "BlurredPanel"
} | {
  _tag: "PressedPointerOnButton"
  button: number
  pointerType: string
} | {
  _tag: "IgnoredMouseClick"
} | {
  _tag: "SuppressedSpaceScroll"
} | {
  _tag: "CompletedFocusPanel"
} | {
  _tag: "CompletedFocusButton"
} | {
  _tag: "CompletedLockScroll"
} | {
  _tag: "CompletedUnlockScroll"
} | {
  _tag: "CompletedInertOthers"
} | {
  _tag: "CompletedRestoreInert"
} | {
  _tag: "CompletedAnchorPopover"
} | {
  _tag: "CompletedPortalPopoverBackdrop"
} | {
  _tag: "GotAnimationMessage"
  message: {
    _tag: "Showed"
  } | {
    _tag: "Hid"
  } | {
    _tag: "CompletedWaitForPaint"
  } | {
    _tag: "EndedAnimation"
  }
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
  focusSelector: string
  isDisabled: boolean
  toView: (render: RenderInfo) => Html
}>>
```
