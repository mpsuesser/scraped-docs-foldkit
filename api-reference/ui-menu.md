---
url: https://foldkit.dev/api-reference/ui-menu
title: "Ui/Menu"
description: "API documentation for the Ui/Menu module."
access_date: 2026-08-04T03:55:42.119Z
current_date: 2026-08-04T03:55:42.119Z
---

# Ui/Menu

## Functions

### buttonId

function

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L303)

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

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L1360)

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
 *  // In update:
 *  const [next, commands, maybeOutMessage] = ActionMenu.update(model.menu, message)
 *  // maybeOutMessage: Option<Menu.OutMessage<Action>>
 *  ```
 */
<Item extends string = string>(): Bundle<Item>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L271)

```
/** Creates an initial menu model from a config. Defaults to closed with no active item. */
(config: InitConfig): Menu.Model
```

## Types

### ActivationTrigger

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L47)

```
/** Schema for the activation trigger: whether the user interacted via mouse or keyboard. */
type ActivationTrigger = Literals<readonly ["Pointer", "Keyboard"]>
```

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L1309)

```
/**
 * The `view`, `update`, and programmatic helpers that `Menu.create`
 *  returns, bound to one `Item` type. Name it to annotate a value that
 *  holds a created bundle, such as a field on a config object or a
 *  function parameter that takes the bundle rather than calling `create`
 *  itself.
 */
type Bundle = Readonly<{
  close: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  open: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  selectItem: (model: Model, item: Item, index: number) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  update: (model: Model, message: Message) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  view: SubmodelView<Model, Message, ViewInputs<Item>>
}>
```

### GroupHeading

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L805)

```
/** Configuration for a group heading rendered above a group of items. */
type GroupHeading = Readonly<{
  className: string
  content: Html
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L264)

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

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L799)

```
/** Configuration for an individual menu item's appearance. */
type ItemConfig = Readonly<{
  className: string
  content: Html
}>
```

### OutMessage

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L234)

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

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L222)

```
/** Sent to the parent when a menu item is selected. Carries both the selected value (from the `viewInputs.items` array supplied at view time) and its index. The menu has already closed when this fires; the parent does not need to dispatch `Menu.close`. */
type Selected = Readonly<{
  _tag: "Selected"
  index: number
  value: Value
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L815)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  The Menu emits a `Selected({ value, index })` OutMessage on commit.
 *  The menu has already closed by the time this fires; consumers
 *  pattern-match it in their `GotMenuMessage` handler to react.
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

### ActivatedItem

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L87)

```
/** Sent when an item is highlighted via arrow keys or mouse hover. Includes activation trigger. */
const ActivatedItem: CallableTaggedStruct<"ActivatedItem", {
  activationTrigger: Literals<readonly ["Pointer", "Keyboard"]>
  index: Number
}>
```

### AnchorMenu

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L743)

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
 *  `Scene.Mount.resolve(AnchorMenu, CompletedAnchorMenu())`.
 */
const AnchorMenu: MountDefinitionWithArgs<"AnchorMenu", {
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
  _tag: "CompletedAnchorMenu"
}>
```

### BlurredItems

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L85)

```
/** Sent when the menu items container loses focus. */
const BlurredItems: CallableTaggedStruct<"BlurredItems", {}>
```

### ClickItem

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L374)

```
/** Programmatically clicks the active menu item's DOM element. */
const ClickItem: CommandDefinitionWithArgs<"ClickItem", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedClickItem"
}, never, never>>
```

### Closed

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L83)

```
/** Sent when the menu closes via Escape key or backdrop click. */
const Closed: CallableTaggedStruct<"Closed", {}>
```

### CompletedAnchorMenu

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L138)

```
/** Sent when the menu items panel mounts and Floating UI has positioned it. Update no-ops; the side effect is the act of positioning, surfaced for DevTools observability. */
const CompletedAnchorMenu: CallableTaggedStruct<"CompletedAnchorMenu", {}>
```

### CompletedClickItem

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L132)

```
/** Sent when the programmatic click command completes. */
const CompletedClickItem: CallableTaggedStruct<"CompletedClickItem", {}>
```

### CompletedDelayClearSearch

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L108)

```
/** Sent after the search debounce period to clear the accumulated query. */
const CompletedDelayClearSearch: CallableTaggedStruct<"CompletedDelayClearSearch", {
  version: Number
}>
```

### CompletedFocusButton

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L120)

```
/** Sent when the focus-button command completes after closing or selecting. */
const CompletedFocusButton: CallableTaggedStruct<"CompletedFocusButton", {}>
```

### CompletedFocusItems

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L118)

```
/** Sent when the focus-items command completes after opening the menu. */
const CompletedFocusItems: CallableTaggedStruct<"CompletedFocusItems", {}>
```

### CompletedInertOthers

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L126)

```
/** Sent when the inert-others command completes. */
const CompletedInertOthers: CallableTaggedStruct<"CompletedInertOthers", {}>
```

### CompletedLockScroll

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L122)

```
/** Sent when the scroll lock command completes. */
const CompletedLockScroll: CallableTaggedStruct<"CompletedLockScroll", {}>
```

### CompletedPortalMenuBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L140)

```
/** Sent when the menu backdrop mounts and is portaled to the document body. Update no-ops; surfaces the portal side effect for DevTools. */
const CompletedPortalMenuBackdrop: CallableTaggedStruct<"CompletedPortalMenuBackdrop", {}>
```

### CompletedRestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L128)

```
/** Sent when the restore-inert command completes. */
const CompletedRestoreInert: CallableTaggedStruct<"CompletedRestoreInert", {}>
```

### CompletedScrollIntoView

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L130)

```
/** Sent when the scroll-into-view command completes after keyboard activation. */
const CompletedScrollIntoView: CallableTaggedStruct<"CompletedScrollIntoView", {}>
```

### CompletedUnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L124)

```
/** Sent when the scroll unlock command completes. */
const CompletedUnlockScroll: CallableTaggedStruct<"CompletedUnlockScroll", {}>
```

### DeactivatedItem

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L92)

```
/** Sent when the mouse leaves an enabled item. */
const DeactivatedItem: CallableTaggedStruct<"DeactivatedItem", {}>
```

### DelayClearSearch

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L384)

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

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L393)

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

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L354)

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

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L344)

```
/** Moves focus to the menu items container after opening. */
const FocusItems: CommandDefinitionWithArgs<"FocusItems", {
  id: String
}, Effect<{
  _tag: "CompletedFocusItems"
}, never, never>>
```

### GotAnimationMessage

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L142)

```
/** Wraps an Animation submodel message for delegation. */
const GotAnimationMessage: CallableTaggedStruct<"GotAnimationMessage", {
  message: Union<[CallableTaggedStruct<"Showed", {}>, CallableTaggedStruct<"Hid", {}>, CallableTaggedStruct<"CompletedWaitForPaint", {}>, CallableTaggedStruct<"EndedAnimation", {}>]>
}>
```

### IgnoredMouseClick

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L134)

```
/** Sent when a mouse click on the button is ignored because pointer-down already handled the toggle. */
const IgnoredMouseClick: CallableTaggedStruct<"IgnoredMouseClick", {}>
```

### InertOthers

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L328)

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

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L318)

```
/** Prevents page scrolling while the menu is open. */
const LockScroll: CommandDefinitionNoArgs<"LockScroll", Effect<{
  _tag: "CompletedLockScroll"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L161)

```
/** Union of all messages the menu component can produce. */
const Message: S.Union<[typeof Opened, typeof Closed, typeof BlurredItems, typeof ActivatedItem, typeof DeactivatedItem, typeof SelectedItem, typeof MovedPointerOverItem, typeof RequestedItemClick, typeof Searched, typeof CompletedDelayClearSearch, typeof CompletedFocusItems, typeof CompletedFocusButton, typeof CompletedLockScroll, typeof CompletedUnlockScroll, typeof CompletedInertOthers, typeof CompletedRestoreInert, typeof CompletedScrollIntoView, typeof CompletedClickItem, typeof IgnoredMouseClick, typeof SuppressedSpaceScroll, typeof CompletedAnchorMenu, typeof CompletedPortalMenuBackdrop, typeof GotAnimationMessage, typeof PressedPointerOnButton, typeof ReleasedPointerOnItems]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L57)

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

### MovedPointerOverItem

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L112)

```
/** Sent when the pointer moves over a menu item, carrying screen coordinates for tracked-pointer comparison. */
const MovedPointerOverItem: CallableTaggedStruct<"MovedPointerOverItem", {
  index: Number
  screenX: Number
  screenY: Number
}>
```

### Opened

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L79)

```
/** Sent when the menu opens via button click or keyboard. Contains an optional initial active item index: None for pointer, Some for keyboard. */
const Opened: CallableTaggedStruct<"Opened", {
  maybeActiveItemIndex: Option<Number>
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L234)

```
/** Union of out-messages the menu component can produce. Surfaced as the third element of `update`'s return tuple and pattern-matched by the parent. */
const OutMessage: Union<readonly [
  CallableTaggedStruct<"Selected", {
    index: Number
    value: String
  }>
]>
```

### PortalMenuBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L766)

```
/**
 * The backdrop-portaling Mount this Menu renders. Exposed so Scene tests can
 *  call `Scene.Mount.resolve(PortalMenuBackdrop, CompletedPortalMenuBackdrop())` to
 *  acknowledge the mount produced by the rendered backdrop.
 */
const PortalMenuBackdrop: MountDefinitionNoArgs<"PortalMenuBackdrop", {
  _tag: "CompletedPortalMenuBackdrop"
}>
```

### PressedPointerOnButton

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L146)

```
/** Sent when the user presses a pointer device on the menu button. Records pointer type and toggles for mouse. */
const PressedPointerOnButton: CallableTaggedStruct<"PressedPointerOnButton", {
  button: Number
  pointerType: String
  screenX: Number
  screenY: Number
  timeStamp: Number
}>
```

### ReleasedPointerOnItems

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L154)

```
/** Sent when the user releases a pointer on the items container, enabling drag-to-select for mouse. */
const ReleasedPointerOnItems: CallableTaggedStruct<"ReleasedPointerOnItems", {
  screenX: Number
  screenY: Number
  timeStamp: Number
}>
```

### RequestedItemClick

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L99)

```
/** Sent when Enter or Space is pressed on the active item, triggering a programmatic click on the DOM element. */
const RequestedItemClick: CallableTaggedStruct<"RequestedItemClick", {
  index: Number
}>
```

### RestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L337)

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

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L364)

```
/** Scrolls the active menu item into view after keyboard navigation. */
const ScrollIntoView: CommandDefinitionWithArgs<"ScrollIntoView", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedScrollIntoView"
}, never, never>>
```

### Searched

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L103)

```
/** Sent when a printable character is typed for typeahead search. */
const Searched: CallableTaggedStruct<"Searched", {
  key: String
  maybeTargetIndex: Option<Number>
}>
```

### Selected

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L222)

```
/** Sent to the parent when a menu item is selected. Carries both the selected value (from the `viewInputs.items` array supplied at view time) and its index. The menu has already closed when this fires; the parent does not need to dispatch `Menu.close`. */
const Selected: CallableTaggedStruct<"Selected", {
  index: Number
  value: String
}>
```

### SelectedItem

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L94)

```
/** Sent when an item is selected via Enter, Space, or click. */
const SelectedItem: CallableTaggedStruct<"SelectedItem", {
  index: Number
  item: String
}>
```

### SuppressedSpaceScroll

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L136)

```
/** Sent when a Space key-up is captured to prevent page scrolling. */
const SuppressedSpaceScroll: CallableTaggedStruct<"SuppressedSpaceScroll", {}>
```

### UnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/ui/src/menu/index.ts#L323)

```
/** Re-enables page scrolling after the menu closes. */
const UnlockScroll: CommandDefinitionNoArgs<"UnlockScroll", Effect<{
  _tag: "CompletedUnlockScroll"
}, never, never>>
```
