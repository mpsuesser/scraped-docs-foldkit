---
url: https://foldkit.dev/api-reference/ui-popover
title: "Ui/Popover"
description: "API documentation for the Ui/Popover module."
access_date: 2026-09-07T07:29:31.695Z
current_date: 2026-09-07T07:29:31.695Z
---

# Ui/Popover

## Functions

### arrowId

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L143)

```
/**
 * Returns the bare DOM id of the popover arrow, derived from the popover's
 *  base id. The `arrow` bundle already carries this id, so reach for this when
 *  you need the id on its own, such as asserting the panel Mount's `arrowId`
 *  argument in a Scene test.
 */
(id: string): string
```

### buttonId

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L137)

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

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L456)

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

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L112)

```
/** Creates an initial popover model from a config. Defaults to closed. */
(config: InitConfig): Popover.Model
```

### open

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L450)

```
/**
 * Programmatically opens the Popover, updating the Model and returning
 *  focus and modal Commands plus an `Opened` OutMessage.
 */
(model: Popover.Model): UpdateReturn
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L256)

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L104)

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

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L478)

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
 *  - `arrow`: attribute bundle for an arrow element inside the panel.
 *    Carries the id the anchor Mount resolves and hides the element from
 *    assistive technology. Spread it onto your own element and place it
 *    with the `--arrow-x` and `--arrow-y` custom properties Anchor
 *    publishes on the panel. Nothing renders until you do.
 *  - `isVisible`: derived from `isOpen` and the Animation
 *    `transitionState`. The consumer renders the panel + backdrop only
 *    while this is true.
 */
type RenderInfo = Readonly<{
  arrow: ReadonlyArray<ChildAttribute>
  backdrop: ReadonlyArray<ChildAttribute>
  button: ReadonlyArray<ChildAttribute>
  isVisible: boolean
  panel: ReadonlyArray<ChildAttribute>
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L487)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  anchor: AnchorConfig
  ariaLabel: string
  ariaLabelledBy: string
  arrowPadding: number
  focusSelector: string
  isDisabled: boolean
  toView: (render: RenderInfo) => Html
}>
```

## Constants

### AnchorPopover

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L397)

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
  arrowId: optional<String>
  arrowPadding: optional<Number>
  buttonId: String
  focusSelector: optional<String>
}, {
  _tag: "CompletedAnchorPopover"
}>
```

### DetectMovementOrAnimationEnd

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L194)

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

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L184)

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

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L174)

```
/** Moves focus to the popover panel after opening. */
const FocusPanel: CommandDefinitionWithArgs<"FocusPanel", {
  id: String
}, Effect<{
  _tag: "CompletedFocusPanel"
}, never, never>>
```

### InertOthers

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L158)

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

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L148)

```
/** Prevents page scrolling while the popover is open in modal mode. */
const LockScroll: CommandDefinitionNoArgs<"LockScroll", Effect<{
  _tag: "CompletedLockScroll"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L55)

```
/** Union of all messages the popover component can produce. */
const Message: MessageUnion<{
  BlurredPanel: {}
  CompletedAnchorPopover: {}
  CompletedFocusButton: {}
  CompletedFocusPanel: {}
  CompletedInertOthers: {}
  CompletedLockScroll: {}
  CompletedPortalPopoverBackdrop: {}
  CompletedRestoreInert: {}
  CompletedUnlockScroll: {}
  GotAnimationMessage: {
    message: MessageUnion<{
      CompletedWaitForPaint: {}
      EndedAnimation: {}
      Hid: {}
      Showed: {}
    }>
  }
  IgnoredMouseClick: {}
  PressedPointerOnButton: {
    button: Number
    pointerType: String
  }
  RequestedClose: {}
  RequestedOpen: {}
  SuppressedSpaceScroll: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L40)

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

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L90)

```
/**
 * Union of OutMessages the popover component can produce. Handle open and
 *  close transitions in the `foldOutMessage` of the Popover's
 *  `Update.foldChild` config.
 */
const OutMessage: MessageUnion<{
  Closed: {}
  Opened: {}
}>
```

### PortalPopoverBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L436)

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

### RestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L167)

```
/** Removes the inert attribute from elements outside the popover. */
const RestoreInert: CommandDefinitionWithArgs<"RestoreInert", {
  id: String
}, Effect<{
  _tag: "CompletedRestoreInert"
}, never, never>>
```

### UnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L153)

```
/** Re-enables page scrolling after the popover closes. */
const UnlockScroll: CommandDefinitionNoArgs<"UnlockScroll", Effect<{
  _tag: "CompletedUnlockScroll"
}, never, never>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/popover/index.ts#L498)

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
  anchor: Anchor.AnchorConfig
  ariaLabel: string
  ariaLabelledBy: string
  arrowPadding: number
  focusSelector: string
  isDisabled: boolean
  toView: (render: RenderInfo) => Html
}>>
```
