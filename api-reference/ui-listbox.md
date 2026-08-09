---
url: https://foldkit.dev/api-reference/ui-listbox
title: "Ui/Listbox"
description: "API documentation for the Ui/Listbox module."
access_date: 2026-08-09T15:49:55.717Z
current_date: 2026-08-09T15:49:55.717Z
---

# Ui/Listbox

## Functions

### buttonId

function

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L279)

```
/**
 * Returns the bare DOM id of the listbox trigger button, derived from the
 *  listbox's base id. Use this to associate an external label with the
 *  trigger via a native `<label for={Listbox.buttonId(id)}>` or an
 *  `aria-labelledby` reference. Mirrors `buttonSelector`, which returns the
 *  CSS selector form (`#${id}-button`) rather than the bare id.
 */
(id: string): string
```

### create

function

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/single.ts#L157)

```
/**
 * Pairs the single-select listbox's `view` and `update` (and programmatic
 *  helpers) behind a single Item-typed entry point. Declaring the listbox
 *  once at module scope ensures the view's `Item` type and the update's
 *  OutMessage `item` type can't drift:
 * 
 *  ```ts
 *  const ColorListbox = Listbox.create<Color>()
 * 
 *  // In view:
 *  h.submodel({ view: ColorListbox.view, ... })
 * 
 *  // In update:
 *  const [next, commands, maybeOutMessage] = ColorListbox.update(model, message)
 *  // maybeOutMessage: Option<Listbox.OutMessage<Color>>
 *  ```
 * 
 *  Two type params support object-typed items with an `itemToValue`
 *  extractor: pass `<Person, string>` when items are objects whose
 *  extracted value is a plain string. `Value` defaults to `Item` when
 *  `Item extends string`, else defaults to `string`.
 */
<Item = string, Value extends string = Item extends string
  ? Item
  : string>(): Bundle<Item, Value>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/single.ts#L36)

```
/** Creates an initial single-select listbox model from a config. Defaults to closed with no active item. */
(config: InitConfig): {
  activationTrigger: "Pointer" | "Keyboard"
  animation: Animation.Model
  id: string
  isAnimated: boolean
  isModal: boolean
  isOpen: boolean
  maybeActiveItemIndex: Option<number>
  maybeLastButtonPointerType: Option<string>
  maybeLastPointerPosition: Option<{
    screenX: number
    screenY: number
  }>
  orientation: "Horizontal" | "Vertical"
  searchQuery: string
  searchVersion: number
}
```

## Types

### ActivationTrigger

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L49)

```
/** Schema for the activation trigger: whether the user interacted via mouse or keyboard. */
type ActivationTrigger = Literals<readonly ["Pointer", "Keyboard"]>
```

### BaseViewInputsCommon

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L806)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  The Listbox emits a `Selected({ value })` OutMessage on commit.
 *  Consumers pattern-match this in their `GotListboxMessage` handler:
 *  single-select stores the value, multi-select toggles its membership.
 */
type BaseViewInputsCommon = Readonly<{
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
  form: string
  groupAttributes: ReadonlyArray<ChildAttribute>
  groupClassName: string
  groupToHeading: (groupKey: string) => GroupHeading | undefined
  isButtonDisabled: boolean
  isDisabled: boolean
  isInvalid: boolean
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
    isSelected: boolean
  }>) => ItemConfig
  itemToSearchText: (item: Item, index: number) => string
  name: string
  separatorAttributes: ReadonlyArray<ChildAttribute>
  separatorClassName: string
}>
```

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/single.ts#L100)

```
/**
 * The `view`, `update`, and programmatic helpers that `Listbox.create`
 *  returns, bound to one `Item` and `Value` pair. Name it to annotate a
 *  value that holds a created bundle, such as a field on a config object
 *  or a function parameter that takes the bundle rather than calling
 *  `create` itself.
 */
type Bundle = Readonly<{
  close: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Value>>]
  open: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Value>>]
  selectItem: (model: Model, item: Value) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Value>>]
  update: (model: Model, message: Message) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Value>>]
  view: SubmodelView<Model, Message, ViewInputs<Item, Value>>
}>
```

### GroupHeading

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L796)

```
/** Configuration for a group heading rendered above a group of items. */
type GroupHeading = Readonly<{
  className: string
  content: Html
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/single.ts#L33)

```
/** Configuration for creating a single-select listbox model with `init`. `isAnimated` enables CSS transition coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). */
type InitConfig = BaseInitConfig
```

### ItemConfig

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L790)

```
/** Configuration for an individual listbox item's appearance. */
type ItemConfig = Readonly<{
  className: string
  content: Html
}>
```

### ItemToValueInput

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L850)

```
/**
 * The `itemToValue` extractor piece of a Listbox's view inputs. The
 *  extractor is optional when `Item` is itself a string (the default
 *  returns the item unchanged) and required when items are objects, so the
 *  OutMessage payload type can't drift from what the consumer actually
 *  emits.
 */
type ItemToValueInput = [Item] extends [string]
  ? Readonly<{
    itemToValue: (item: Item) => Value
  }>
  : Readonly<{
    itemToValue: (item: Item) => Value
  }>
```

### OutMessage

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L257)

```
/**
 * Generic over `Value extends string` so consumers who create the listbox
 *  via `Listbox.create<MyUnion>()` receive `value: MyUnion` in the
 *  `Selected` OutMessage from the factory's `update`, instead of
 *  `value: string`. Defaults to `string`.
 */
type OutMessage = Selected<Value>
```

### Selected

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L247)

```
/** Sent when the user activates an item (single-select commit or multi-select toggle). Carries the neutral fact that the item was activated; the parent owns the selection and decides what it means (single-select sets it, multi-select toggles membership). Generic over `Value extends string`: the runtime schema stores `value: string`, but the type-level OutMessage exposes `value: Value` so consumers who supply `items: ReadonlyArray<MyUnion>` receive `value: MyUnion` from `update<MyUnion>` without casting. The cast is fenced inside this module's `update` return, sound because the value was extracted from the items array the consumer supplied. */
type Selected = Readonly<{
  _tag: "Selected"
  value: Value
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/single.ts#L65)

```
/** Per-render view inputs passed to the view via `h.submodel`'s `viewInputs` field. */
type ViewInputs = BaseViewInputsCommon<Item> & Readonly<{
  maybeSelectedValue: Option.Option<Value>
}> & ItemToValueInput<Item, Value>
```

## Constants

### ActivatedItem

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L110)

```
/** Sent when an item is highlighted via arrow keys or mouse hover. Includes activation trigger. */
const ActivatedItem: CallableTaggedStruct<"ActivatedItem", {
  activationTrigger: Literals<readonly ["Pointer", "Keyboard"]>
  index: Number
}>
```

### AnchorListbox

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L751)

```
/**
 * The anchor-positioning Mount this Listbox renders on its items panel.
 *  The panel is always anchored to the button via Floating UI and portaled
 *  to the document body (opt out of portaling with `anchor.portal: false`),
 *  so it escapes ancestor stacking contexts and overflow clipping.
 * 
 *  It also carries the open-focus for the anchored panel. An anchored panel
 *  renders `visibility: hidden` until Floating UI resolves its first position,
 *  and `.focus()` does not land on a hidden element, so `FocusItems` alone
 *  cannot focus it. `focusAfterPosition` focuses the panel as part of that
 *  first reveal. `FocusItems` still focuses the panel when no anchor is
 *  configured, where the panel is visible as soon as the render commits.
 * 
 *  Exposed so Scene tests can call
 *  `Scene.Mount.resolve(AnchorListbox, CompletedAnchorListbox())`.
 */
const AnchorListbox: MountDefinitionWithArgs<"AnchorListbox", {
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
  _tag: "CompletedAnchorListbox"
}>
```

### BlurredItems

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L108)

```
/** Sent when the listbox items container loses focus. */
const BlurredItems: CallableTaggedStruct<"BlurredItems", {}>
```

### ClickItem

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L379)

```
/** Programmatically clicks the active listbox item's DOM element. */
const ClickItem: CommandDefinitionWithArgs<"ClickItem", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedClickItem"
}, never, never>>
```

### Closed

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L106)

```
/** Sent when the listbox closes via Escape key or backdrop click. */
const Closed: CallableTaggedStruct<"Closed", {}>
```

### CompletedAnchorListbox

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L158)

```
/** Sent when the listbox items panel mounts and Floating UI has positioned it. Update no-ops; surfaces the positioning side effect for DevTools. */
const CompletedAnchorListbox: CallableTaggedStruct<"CompletedAnchorListbox", {}>
```

### CompletedClickItem

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L152)

```
/** Sent when the programmatic item click command completes. */
const CompletedClickItem: CallableTaggedStruct<"CompletedClickItem", {}>
```

### CompletedDelayClearSearch

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L128)

```
/** Sent after the search debounce period to clear the accumulated query. */
const CompletedDelayClearSearch: CallableTaggedStruct<"CompletedDelayClearSearch", {
  version: Number
}>
```

### CompletedFocusButton

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L146)

```
/** Sent when the focus-button command completes after closing. */
const CompletedFocusButton: CallableTaggedStruct<"CompletedFocusButton", {}>
```

### CompletedFocusItems

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L148)

```
/** Sent when the focus-items command completes after opening. */
const CompletedFocusItems: CallableTaggedStruct<"CompletedFocusItems", {}>
```

### CompletedInertOthers

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L142)

```
/** Sent when the inert-others command completes. */
const CompletedInertOthers: CallableTaggedStruct<"CompletedInertOthers", {}>
```

### CompletedLockScroll

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L138)

```
/** Sent when the scroll lock command completes. */
const CompletedLockScroll: CallableTaggedStruct<"CompletedLockScroll", {}>
```

### CompletedPortalListboxBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L160)

```
/** Sent when the listbox backdrop mounts and is portaled to the document body. Update no-ops; surfaces the portal side effect for DevTools. */
const CompletedPortalListboxBackdrop: CallableTaggedStruct<"CompletedPortalListboxBackdrop", {}>
```

### CompletedRestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L144)

```
/** Sent when the restore-inert command completes. */
const CompletedRestoreInert: CallableTaggedStruct<"CompletedRestoreInert", {}>
```

### CompletedScrollIntoView

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L150)

```
/** Sent when the scroll-into-view command completes after keyboard activation. */
const CompletedScrollIntoView: CallableTaggedStruct<"CompletedScrollIntoView", {}>
```

### CompletedUnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L140)

```
/** Sent when the scroll unlock command completes. */
const CompletedUnlockScroll: CallableTaggedStruct<"CompletedUnlockScroll", {}>
```

### DeactivatedItem

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L115)

```
/** Sent when the mouse leaves an enabled item. */
const DeactivatedItem: CallableTaggedStruct<"DeactivatedItem", {}>
```

### DelayClearSearch

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L389)

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

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L398)

```
/** Detects whether the listbox button moved or the leave animation ended. Whichever comes first; both outcomes signal the Animation submodel that leave is complete. */
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

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L349)

```
/** Moves focus back to the listbox button after closing. */
const FocusButton: CommandDefinitionWithArgs<"FocusButton", {
  id: String
}, Effect<{
  _tag: "CompletedFocusButton"
}, never, never>>
```

### FocusItems

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L359)

```
/** Moves focus to the listbox items container after opening. */
const FocusItems: CommandDefinitionWithArgs<"FocusItems", {
  id: String
}, Effect<{
  _tag: "CompletedFocusItems"
}, never, never>>
```

### GotAnimationMessage

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L164)

```
/** Wraps an Animation submodel message for delegation. */
const GotAnimationMessage: CallableTaggedStruct<"GotAnimationMessage", {
  message: Union<[CallableTaggedStruct<"Showed", {}>, CallableTaggedStruct<"Hid", {}>, CallableTaggedStruct<"CompletedWaitForPaint", {}>, CallableTaggedStruct<"EndedAnimation", {}>]>
}>
```

### IgnoredMouseClick

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L154)

```
/** Sent when a mouse click on the button is ignored because pointer-down already handled the toggle. */
const IgnoredMouseClick: CallableTaggedStruct<"IgnoredMouseClick", {}>
```

### InertOthers

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L333)

```
/** Marks all elements outside the listbox as inert for modal behavior. */
const InertOthers: CommandDefinitionWithArgs<"InertOthers", {
  id: String
}, Effect<{
  _tag: "CompletedInertOthers"
}, never, never>>
```

### LockScroll

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L323)

```
/** Prevents page scrolling while the listbox is open in modal mode. */
const LockScroll: CommandDefinitionNoArgs<"LockScroll", Effect<{
  _tag: "CompletedLockScroll"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L174)

```
/** Union of all messages the listbox component can produce. */
const Message: S.Union<[typeof Opened, typeof Closed, typeof BlurredItems, typeof ActivatedItem, typeof DeactivatedItem, typeof SelectedItem, typeof MovedPointerOverItem, typeof RequestedItemClick, typeof Searched, typeof CompletedDelayClearSearch, typeof CompletedLockScroll, typeof CompletedUnlockScroll, typeof CompletedInertOthers, typeof CompletedRestoreInert, typeof CompletedFocusButton, typeof CompletedFocusItems, typeof CompletedScrollIntoView, typeof CompletedClickItem, typeof IgnoredMouseClick, typeof SuppressedSpaceScroll, typeof CompletedAnchorListbox, typeof CompletedPortalListboxBackdrop, typeof GotAnimationMessage, typeof PressedPointerOnButton]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/single.ts#L24)

```
/** Schema for the single-select listbox's private interaction state (open/closed status, active item, activation trigger, typeahead search). The selection is owned by the parent and passed in via `ViewInputs.maybeSelectedValue`. */
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
  orientation: Literals<readonly ["Vertical", "Horizontal"]>
  searchQuery: String
  searchVersion: Number
}>
```

### MovedPointerOverItem

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L132)

```
/** Sent when the pointer moves over a listbox item, carrying screen coordinates for tracked-pointer comparison. */
const MovedPointerOverItem: CallableTaggedStruct<"MovedPointerOverItem", {
  index: Number
  screenX: Number
  screenY: Number
}>
```

### Opened

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L102)

```
/** Sent when the listbox opens via button click or keyboard. Contains an optional initial active item index: None for pointer, Some for keyboard. */
const Opened: CallableTaggedStruct<"Opened", {
  maybeActiveItemIndex: Option<Number>
}>
```

### Orientation

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L53)

```
/** Schema for the listbox orientation: whether items flow vertically or horizontally. */
const Orientation: Literals<readonly ["Vertical", "Horizontal"]>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L257)

```
/** Union of out-messages the listbox component can produce. The parent folds `Selected` into the selection it owns: single-select stores the value, multi-select toggles the value's membership. */
const OutMessage: Union<readonly [
  CallableTaggedStruct<"Selected", {
    value: String
  }>
]>
```

### PortalListboxBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L774)

```
/**
 * The backdrop-portaling Mount this Listbox renders. Exposed so Scene tests can
 *  call `Scene.Mount.resolve(PortalListboxBackdrop, CompletedPortalListboxBackdrop())` to
 *  acknowledge the mount produced by the rendered backdrop.
 */
const PortalListboxBackdrop: MountDefinitionNoArgs<"PortalListboxBackdrop", {
  _tag: "CompletedPortalListboxBackdrop"
}>
```

### PressedPointerOnButton

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L168)

```
/** Sent when the user presses a pointer device on the listbox button. Records pointer type for click handling. */
const PressedPointerOnButton: CallableTaggedStruct<"PressedPointerOnButton", {
  button: Number
  pointerType: String
}>
```

### RequestedItemClick

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L119)

```
/** Sent when Enter or Space is pressed on the active item, triggering a programmatic click on the DOM element. */
const RequestedItemClick: CallableTaggedStruct<"RequestedItemClick", {
  index: Number
}>
```

### RestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L342)

```
/** Removes the inert attribute from elements outside the listbox. */
const RestoreInert: CommandDefinitionWithArgs<"RestoreInert", {
  id: String
}, Effect<{
  _tag: "CompletedRestoreInert"
}, never, never>>
```

### ScrollIntoView

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L369)

```
/** Scrolls the active listbox item into view after keyboard navigation. */
const ScrollIntoView: CommandDefinitionWithArgs<"ScrollIntoView", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedScrollIntoView"
}, never, never>>
```

### Searched

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L123)

```
/** Sent when a printable character is typed for typeahead search. */
const Searched: CallableTaggedStruct<"Searched", {
  key: String
  maybeTargetIndex: Option<Number>
}>
```

### Selected

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L247)

```
/** Sent when the user activates an item (single-select commit or multi-select toggle). Carries the neutral fact that the item was activated; the parent owns the selection and decides what it means (single-select sets it, multi-select toggles membership). Generic over `Value extends string`: the runtime schema stores `value: string`, but the type-level OutMessage exposes `value: Value` so consumers who supply `items: ReadonlyArray<MyUnion>` receive `value: MyUnion` from `update<MyUnion>` without casting. The cast is fenced inside this module's `update` return, sound because the value was extracted from the items array the consumer supplied. */
const Selected: CallableTaggedStruct<"Selected", {
  value: String
}>
```

### SelectedItem

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L117)

```
/** Sent when an item is selected via Enter, Space, or click. Contains the item's string value. */
const SelectedItem: CallableTaggedStruct<"SelectedItem", {
  item: String
}>
```

### SuppressedSpaceScroll

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L156)

```
/** Sent when a Space key-up is captured to prevent page scrolling. */
const SuppressedSpaceScroll: CallableTaggedStruct<"SuppressedSpaceScroll", {}>
```

### UnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/listbox/shared.ts#L328)

```
/** Re-enables page scrolling after the listbox closes. */
const UnlockScroll: CommandDefinitionNoArgs<"UnlockScroll", Effect<{
  _tag: "CompletedUnlockScroll"
}, never, never>>
```
