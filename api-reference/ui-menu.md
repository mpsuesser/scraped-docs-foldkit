---
url: https://foldkit.dev/api-reference/ui-menu
title: "Ui/Menu"
description: "API documentation for the Ui/Menu module."
access_date: 2026-09-18T04:36:53.681Z
current_date: 2026-09-18T04:36:53.681Z
---

# Ui/Menu

## Functions

### buttonId

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L209)

```
/**
 * Returns the bare DOM id of the menu trigger button, derived from the
 *  menu's base id. Use this to associate an external label with the trigger
 *  via a native `<label for={Menu.buttonId(id)}>` or an `aria-labelledby`
 *  reference.
 */
(id: string): string
```

### create

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L1244)

```
/**
 * Pairs the menu's `view` and `update` (and programmatic helpers)
 *  behind a single Item-typed entry point. Declaring the menu once at
 *  module scope ensures the view's `Item` type and the OutMessage's
 *  `item` type can't drift:
 * 
 *  ```ts
 *  const ActionMenu = Menu.create<Action>()
 * 
 *  // In view:
 *  h.submodel({ view: ActionMenu.view, ... })
 * 
 *  // In the parent update, pass ActionMenu.update to Update.foldChild and
 *  // handle Menu.OutMessage<Action> in foldOutMessage.
 *  ```
 */
<Item extends string = string>(): Bundle<Item>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L177)

```
/** Creates an initial menu model from a config. Defaults to closed with no active item. */
(config: InitConfig): Menu.Model
```

## Types

### ActivationTrigger

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L48)

```
/** Schema for the activation trigger: whether the user interacted via mouse or keyboard. */
type ActivationTrigger = Literals<readonly ["Pointer", "Keyboard"]>
```

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L1217)

```
type Bundle = Readonly<{
  close: (model: Model) => BundleUpdateReturn<Item>
  open: (model: Model) => BundleUpdateReturn<Item>
  selectItem: (model: Model, item: Item, index: number) => BundleUpdateReturn<Item>
  update: (model: Model, message: Message) => BundleUpdateReturn<Item>
  view: SubmodelView<Model, Message, ViewInputs<Item>>
}>
```

### GroupHeading

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L696)

```
/** Configuration for a group heading rendered above a group of items. */
type GroupHeading = Readonly<{
  className: string
  content: Html
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L170)

```
/** Configuration for creating a menu model with `init`. `isAnimated` enables animation coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). */
type InitConfig = Readonly<{
  id: string
  isAnimated: boolean
  isModal: boolean
}>
```

### ItemConfig

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L690)

```
/** Configuration for an individual menu item's appearance. */
type ItemConfig = Readonly<{
  className: string
  content: Html
}>
```

### OutMessage

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L131)

```
/**
 * Generic over `Value extends string` so consumers using the typed
 *  `Menu.create<MyUnion>()` factory receive `value: MyUnion` in the
 *  `Selected` OutMessage. Defaults to `string`.
 */
type OutMessage = Selected<Value>
```

### Selected

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L135)

```
type Selected = Readonly<{
  _tag: "Selected"
  index: number
  value: Value
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L706)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  The Menu emits an `OutMessage.Selected({ value, index })` OutMessage on commit.
 *  The menu has already closed by the time this fires. Handle it in the
 *  `foldOutMessage` of the Menu's `Update.foldChild` config.
 */
type ViewInputs = Readonly<{
  anchor: AnchorConfig
  ariaLabel: string
  ariaLabelledBy: string
  attributes: ReadonlyArray<ChildAttribute>
  backdropAttributes: ReadonlyArray<ChildAttribute>
  backdropClassName: string
  buttonAttributes: ReadonlyArray<ChildAttribute>
  buttonClassName: string
  buttonContent: Html
  className: string
  groupAttributes: ReadonlyArray<ChildAttribute>
  groupClassName: string
  groupToHeading: (groupKey: string) => GroupHeading | undefined
  isButtonDisabled: boolean
  isItemDisabled: (item: Item, index: number) => boolean
  itemGroupKey: (item: Item, index: number) => string
  items: ReadonlyArray<Item>
  itemsAttributes: ReadonlyArray<ChildAttribute>
  itemsClassName: string
  itemsScrollAttributes: ReadonlyArray<ChildAttribute>
  itemsScrollClassName: string
  itemToConfig: (item: Item, context: Readonly<{
    isActive: boolean
    isDisabled: boolean
  }>) => ItemConfig
  itemToSearchText: (item: Item, index: number) => string
  separatorAttributes: ReadonlyArray<ChildAttribute>
  separatorClassName: string
}>
```

## Constants

### AnchorMenu

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L633)

```
/**
 * The anchor-positioning Mount this Menu renders on its panel. The panel is
 *  always anchored to the button via Floating UI and portaled to the document
 *  body (opt out of portaling with `anchor.portal: false`), so it escapes
 *  ancestor stacking contexts and overflow clipping.
 * 
 *  It also carries the open-focus for the anchored panel. An anchored panel
 *  renders `visibility: hidden` until Floating UI resolves its first position,
 *  and `.focus()` does not land on a hidden element, so `FocusItems` alone
 *  cannot focus it. `focusAfterPosition` focuses the panel as part of that
 *  first reveal. `FocusItems` still focuses the panel when no anchor is
 *  configured, where the panel is visible as soon as the render commits.
 * 
 *  Exposed so Scene tests can call
 *  `Scene.Mount.resolve(AnchorMenu, Message.CompletedAnchorMenu())`.
 */
const AnchorMenu: MountDefinitionWithArgs<"AnchorMenu", {
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
  _tag: "CompletedAnchorMenu"
}>
```

### ClickItem

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L275)

```
/** Programmatically clicks the active menu item's DOM element. */
const ClickItem: CommandDefinitionWithArgs<"ClickItem", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedClickItem"
}, never, never>>
```

### DelayClearSearch

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L285)

```
/** Waits for the typeahead search debounce period before clearing the query. */
const DelayClearSearch: CommandDefinitionWithArgs<"DelayClearSearch", {
  version: Number
}, Effect<{
  _tag: "CompletedDelayClearSearch"
  version: number
}, never, never>>
```

### DetectMovementOrAnimationEnd

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L294)

```
/** Detects whether the menu button moved or the leave animation ended. Whichever comes first; both outcomes signal the Animation submodel that leave is complete. */
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

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L255)

```
/** Moves focus back to the menu button after closing. */
const FocusButton: CommandDefinitionWithArgs<"FocusButton", {
  id: String
}, Effect<{
  _tag: "CompletedFocusButton"
}, never, never>>
```

### FocusItems

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L245)

```
/** Moves focus to the menu items container after opening. */
const FocusItems: CommandDefinitionWithArgs<"FocusItems", {
  id: String
}, Effect<{
  _tag: "CompletedFocusItems"
}, never, never>>
```

### InertOthers

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L229)

```
/** Marks all elements outside the menu as inert for modal behavior. */
const InertOthers: CommandDefinitionWithArgs<"InertOthers", {
  id: String
}, Effect<{
  _tag: "CompletedInertOthers"
}, never, never>>
```

### LockScroll

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L219)

```
/** Prevents page scrolling while the menu is open. */
const LockScroll: CommandDefinitionNoArgs<"LockScroll", Effect<{
  _tag: "CompletedLockScroll"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L80)

```
/** Union of all messages the menu component can produce. */
const Message: MessageUnion<{
  ActivatedItem: {
    activationTrigger: Literals<readonly ["Pointer", "Keyboard"]>
    index: Number
  }
  BlurredItems: {}
  Closed: {}
  CompletedAnchorMenu: {}
  CompletedClickItem: {}
  CompletedDelayClearSearch: {
    version: Number
  }
  CompletedFocusButton: {}
  CompletedFocusItems: {}
  CompletedInertOthers: {}
  CompletedLockScroll: {}
  CompletedPortalMenuBackdrop: {}
  CompletedRestoreInert: {}
  CompletedScrollIntoView: {}
  CompletedUnlockScroll: {}
  DeactivatedItem: {}
  GotAnimationMessage: {
    message: MessageUnion<{
      CompletedWaitForPaint: {}
      EndedAnimation: {}
      Hid: {}
      Showed: {}
    }>
  }
  IgnoredMouseClick: {}
  MovedPointerOverItem: {
    index: Number
    screenX: Number
    screenY: Number
  }
  Opened: {
    maybeActiveItemIndex: Option<Number>
  }
  PressedPointerOnButton: {
    button: Number
    pointerType: String
    screenX: Number
    screenY: Number
    timeStamp: Number
  }
  ReleasedPointerOnItems: {
    screenX: Number
    screenY: Number
    timeStamp: Number
  }
  RequestedItemClick: {
    index: Number
  }
  Searched: {
    key: String
    maybeTargetIndex: Option<Number>
  }
  SelectedItem: {
    index: Number
    item: String
  }
  SuppressedSpaceScroll: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L58)

```
/** Schema for the menu component's state, tracking open/closed status, active item, activation trigger, and typeahead search. */
const Model: Struct<{
  activationTrigger: Literals<readonly ["Pointer", "Keyboard"]>
  animation: Struct<{
    id: String
    isShowing: Boolean
    transitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
  }>
  id: String
  isAnimated: Boolean
  isModal: Boolean
  isOpen: Boolean
  maybeActiveItemIndex: Option<Number>
  maybeLastButtonPointerType: Option<String>
  maybeLastPointerPosition: Option<Struct<{
    screenX: Number
    screenY: Number
  }>>
  maybePointerOrigin: Option<Struct<{
    screenX: Number
    screenY: Number
    timeStamp: Number
  }>>
  searchQuery: String
  searchVersion: Number
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L131)

```
/**
 * Union of OutMessages the menu component can produce. The parent's
 *  `Update.foldChild` config handles them through `foldOutMessage`.
 */
const OutMessage: MessageUnion<{
  Selected: {
    index: Number
    value: String
  }
}>
```

### PortalMenuBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L655)

```
/**
 * The backdrop-portaling Mount this Menu renders. Exposed so Scene tests can
 *  call `Scene.Mount.resolve(PortalMenuBackdrop, Message.CompletedPortalMenuBackdrop())` to
 *  acknowledge the mount produced by the rendered backdrop.
 */
const PortalMenuBackdrop: MountDefinitionNoArgs<"PortalMenuBackdrop", {
  _tag: "CompletedPortalMenuBackdrop"
}>
```

### RestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L238)

```
/** Removes the inert attribute from elements outside the menu. */
const RestoreInert: CommandDefinitionWithArgs<"RestoreInert", {
  id: String
}, Effect<{
  _tag: "CompletedRestoreInert"
}, never, never>>
```

### ScrollIntoView

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L265)

```
/** Scrolls the active menu item into view after keyboard navigation. */
const ScrollIntoView: CommandDefinitionWithArgs<"ScrollIntoView", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedScrollIntoView"
}, never, never>>
```

### UnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/menu/index.ts#L224)

```
/** Re-enables page scrolling after the menu closes. */
const UnlockScroll: CommandDefinitionNoArgs<"UnlockScroll", Effect<{
  _tag: "CompletedUnlockScroll"
}, never, never>>
```
