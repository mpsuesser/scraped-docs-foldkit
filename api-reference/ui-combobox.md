---
url: https://foldkit.dev/api-reference/ui-combobox
title: "Ui/Combobox"
description: "API documentation for the Ui/Combobox module."
access_date: 2026-08-12T23:23:45.679Z
current_date: 2026-08-12T23:23:45.679Z
---

# Ui/Combobox

## Functions

### create

function

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/single.ts#L163)

```
/**
 * Pairs the single-select combobox's `view` and `update` (and programmatic
 *  helpers) behind a single Item-typed entry point. See `Listbox.create`
 *  for the rationale; the combobox factory follows the same shape with
 *  `selectItem` taking both `item` and `displayText`. `selectItem` emits
 *  `Selected({ value })` with the input resting on `displayText`; what the
 *  selection becomes is the parent's fold to decide.
 */
<Item extends string = string>(): Bundle<Item>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/single.ts#L38)

```
/** Creates an initial single-select combobox model from a config. Defaults to closed with no active item and an empty input. */
(config: InitConfig): {
  activationTrigger: "Pointer" | "Keyboard"
  animation: Animation.Model
  id: string
  immediate: boolean
  inputValue: string
  isAnimated: boolean
  isModal: boolean
  isOpen: boolean
  maybeActiveItemIndex: Option<number>
  maybeLastPointerPosition: Option<{
    screenX: number
    screenY: number
  }>
  nullable: boolean
  selectInputOnFocus: boolean
}
```

### inputId

function

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L289)

```
/**
 * Returns the bare DOM id of the combobox input, derived from the
 *  combobox's base id. Use this to associate an external label with the
 *  input via a native `<label for={Combobox.inputId(id)}>` or an
 *  `aria-labelledby` reference. Mirrors `inputSelector`, which returns the
 *  CSS selector form (`#${id}-input`) rather than the bare id.
 */
(id: string): string
```

## Types

### ActivationTrigger

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L43)

```
/** Schema for the activation trigger: whether the user interacted via mouse or keyboard. */
type ActivationTrigger = Literals<readonly ["Pointer", "Keyboard"]>
```

### AnchorConfig

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/anchor.ts#L31)

```
/** Static configuration for anchor-based positioning of a floating element relative to a button. */
type AnchorConfig = Struct<{
  gap: optional<Number>
  isPlacementLocked: optional<Boolean>
  offset: optional<Number>
  padding: optional<Number>
  placement: optional<Literals<readonly ["top", "right", "bottom", "left", "top-start", "top-end", "right-start", "right-end", "bottom-start", "bottom-end", "left-start", "left-end"]>>
  portal: optional<Boolean>
}>
```

### BaseViewInputsCommon

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L870)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  The Combobox emits a `Selected({ value })` OutMessage on commit.
 *  Consumers pattern-match this in their `GotComboboxMessage` handler:
 *  single-select stores the value, multi-select toggles the value's
 *  membership. `restingInputValue` is the text the input returns to on
 *  close (the selection's display text for single-select, empty for
 *  multi-select). Everything here except the selection itself; each
 *  variant composes its own selection field on top.
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
  formName: string
  groupAttributes: ReadonlyArray<ChildAttribute>
  groupClassName: string
  groupToHeading: (groupKey: string) => GroupHeading | undefined
  inputAttributes: ReadonlyArray<ChildAttribute>
  inputClassName: string
  inputPlaceholder: string
  inputWrapperAttributes: ReadonlyArray<ChildAttribute>
  inputWrapperClassName: string
  isDisabled: boolean
  isInvalid: boolean
  isItemDisabled: (item: Item, index: number) => boolean
  isReadOnly: boolean
  itemGroupKey: (item: Item, index: number) => string
  items: ReadonlyArray<Item>
  itemsAttributes: ReadonlyArray<ChildAttribute>
  itemsClassName: string
  itemsScrollAttributes: ReadonlyArray<ChildAttribute>
  itemsScrollClassName: string
  itemToConfig: (item: Item, context: Readonly<{
    isActive: boolean
    isDisabled: boolean
    isReadOnly: boolean
    isSelected: boolean
  }>) => ItemConfig
  itemToDisplayText: (item: Item, index: number) => string
  itemToValue: (item: Item, index: number) => Item
  openOnFocus: boolean
  restingInputValue: string
  separatorAttributes: ReadonlyArray<ChildAttribute>
  separatorClassName: string
}>
```

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/single.ts#L121)

```
/**
 * The `view`, `update`, and programmatic helpers that `Combobox.create`
 *  returns, bound to one `Item` type. Name it to annotate a value that
 *  holds a created bundle, such as a field on a config object or a
 *  function parameter that takes the bundle rather than calling `create`
 *  itself.
 */
type Bundle = Readonly<{
  close: (model: Model, restingInputValue: string) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  open: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  selectItem: (model: Model, item: Item, displayText: string) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  update: (model: Model, message: Message) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  view: SubmodelView<Model, Message, ViewInputs<Item>>
}>
```

### GroupHeading

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L856)

```
/** Configuration for a group heading rendered above a group of items. */
type GroupHeading = Readonly<{
  className: string
  content: Html
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/single.ts#L35)

```
/** Configuration for creating a single-select combobox model with `init`. `isAnimated` enables CSS transition coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). */
type InitConfig = BaseInitConfig
```

### ItemConfig

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L850)

```
/** Configuration for an individual combobox item's appearance. */
type ItemConfig = Readonly<{
  className: string
  content: Html
}>
```

### OutMessage

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L272)

```
/**
 * Generic over `Value extends string` so consumers who create the combobox
 *  via `Combobox.create<MyUnion>()` receive `value: MyUnion` in the
 *  `Selected` OutMessage from the factory's `update`, instead of
 *  `value: string`. Defaults to `string`.
 */
type OutMessage = Selected<Value> | ClearedSelection
```

### Selected

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L257)

```
/** Sent when the user activates an item. Carries the neutral fact that the item was activated; the parent owns the selection and decides what it means (single-select stores the value, nullable single-select toggles it, multi-select toggles the value's membership). Generic over `Value extends string`: the runtime schema stores `value: string`, but the type-level OutMessage exposes `value: Value` so consumers who supply `items: ReadonlyArray<MyUnion>` receive `value: MyUnion` from the factory's `update` without casting. */
type Selected = Readonly<{
  _tag: "Selected"
  value: Value
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/single.ts#L105)

```
/** Per-render view inputs passed to the view via `h.submodel`'s `viewInputs` field. */
type ViewInputs = BaseViewInputsCommon<Item> & Readonly<{
  maybeSelectedValue: Option.Option<Item>
}>
```

## Constants

### ActivatedItem

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L113)

```
/** Sent when an item is highlighted via arrow keys or mouse hover. Includes activation trigger and optional immediate selection info. */
const ActivatedItem: CallableTaggedStruct<"ActivatedItem", {
  activationTrigger: Literals<readonly ["Pointer", "Keyboard"]>
  index: Number
  maybeImmediateSelection: Option<Struct<{
    item: String
  }>>
}>
```

### AnchorCombobox

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L744)

```
/**
 * The anchor-positioning Mount this Combobox renders on its items panel.
 *  The panel is always anchored to the input wrapper via Floating UI and
 *  portaled to the document body (opt out of portaling with
 *  `anchor.portal: false`), so it escapes ancestor stacking contexts and
 *  overflow clipping. The Mount also installs the `pointerdown`-cancelling
 *  capture listener that prevents input blur on item presses. Exposed so
 *  Scene tests can call
 *  `Scene.Mount.resolve(AnchorCombobox, CompletedAnchorCombobox())`.
 */
const AnchorCombobox: MountDefinitionWithArgs<"AnchorCombobox", {
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
  _tag: "CompletedAnchorCombobox"
}>
```

### AttachComboboxPreventBlur

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L782)

```
/**
 * The Mount this Combobox renders to install a `pointerdown`-cancelling
 *  capture listener that prevents blur on item presses. Exposed so Scene
 *  tests can call
 *  `Scene.Mount.resolve(AttachComboboxPreventBlur, CompletedAttachComboboxPreventBlur())`.
 */
const AttachComboboxPreventBlur: MountDefinitionNoArgs<"AttachComboboxPreventBlur", {
  _tag: "CompletedAttachComboboxPreventBlur"
}>
```

### AttachComboboxSelectOnFocus

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L809)

```
/**
 * The Mount this Combobox renders to install the input's select-on-focus
 *  behavior. Exposed so Scene tests can call
 *  `Scene.Mount.resolve(AttachComboboxSelectOnFocus, CompletedAttachComboboxSelectOnFocus())`.
 */
const AttachComboboxSelectOnFocus: MountDefinitionNoArgs<"AttachComboboxSelectOnFocus", {
  _tag: "CompletedAttachComboboxSelectOnFocus"
}>
```

### BlurredInput

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L108)

```
/** Sent when the combobox input loses focus. `restingInputValue` is what the input returns to on close (the parent-owned selection's display text, or empty), computed by the view from `ViewInputs.restingInputValue`. `isClearable` carries whether this close may emit `ClearedSelection`, which a read-only combobox denies. */
const BlurredInput: CallableTaggedStruct<"BlurredInput", {
  isClearable: Boolean
  restingInputValue: String
}>
```

### ClearedSelection

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L267)

```
/** Sent when a nullable combobox closes with an empty input, meaning the user cleared it. The parent clears the selection it owns. */
const ClearedSelection: CallableTaggedStruct<"ClearedSelection", {}>
```

### ClickItem

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L373)

```
/** Programmatically clicks the active combobox item's DOM element. */
const ClickItem: CommandDefinitionWithArgs<"ClickItem", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedClickItem"
}, never, never>>
```

### Closed

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L103)

```
/** Sent when the combobox closes via Escape key or backdrop click. `restingInputValue` is what the input returns to on close (the parent-owned selection's display text, or empty), computed by the view from `ViewInputs.restingInputValue`. `isClearable` carries whether this close may emit `ClearedSelection`, which a read-only combobox denies; the view holds `isReadOnly` and the update does not. */
const Closed: CallableTaggedStruct<"Closed", {
  isClearable: Boolean
  restingInputValue: String
}>
```

### CompletedAnchorCombobox

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L153)

```
/** Sent when the items panel mounts and Floating UI has positioned it. Update no-ops; surfaces the positioning side effect for DevTools. */
const CompletedAnchorCombobox: CallableTaggedStruct<"CompletedAnchorCombobox", {}>
```

### CompletedAttachComboboxPreventBlur

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L155)

```
/** Sent when the items panel mounts and the capture-phase pointerdown listener is attached (with or without anchor). Update no-ops; surfaces the listener-attach side effect for DevTools. */
const CompletedAttachComboboxPreventBlur: CallableTaggedStruct<"CompletedAttachComboboxPreventBlur", {}>
```

### CompletedAttachComboboxSelectOnFocus

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L159)

```
/** Sent when the input mounts and the focus listener that auto-selects on focus is attached. Update no-ops; surfaces the listener-attach side effect for DevTools. */
const CompletedAttachComboboxSelectOnFocus: CallableTaggedStruct<"CompletedAttachComboboxSelectOnFocus", {}>
```

### CompletedClickItem

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L151)

```
/** Sent when the programmatic item click command completes. */
const CompletedClickItem: CallableTaggedStruct<"CompletedClickItem", {}>
```

### CompletedFocusInput

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L147)

```
/** Sent when the focus-input command completes. */
const CompletedFocusInput: CallableTaggedStruct<"CompletedFocusInput", {}>
```

### CompletedInertOthers

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L143)

```
/** Sent when the inert-others command completes. */
const CompletedInertOthers: CallableTaggedStruct<"CompletedInertOthers", {}>
```

### CompletedLockScroll

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L139)

```
/** Sent when the scroll lock command completes. */
const CompletedLockScroll: CallableTaggedStruct<"CompletedLockScroll", {}>
```

### CompletedPortalComboboxBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L163)

```
/** Sent when the combobox backdrop mounts and is portaled to the document body. Update no-ops; surfaces the portal side effect for DevTools. */
const CompletedPortalComboboxBackdrop: CallableTaggedStruct<"CompletedPortalComboboxBackdrop", {}>
```

### CompletedRestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L145)

```
/** Sent when the restore-inert command completes. */
const CompletedRestoreInert: CallableTaggedStruct<"CompletedRestoreInert", {}>
```

### CompletedScrollIntoView

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L149)

```
/** Sent when the scroll-into-view command completes after keyboard activation. */
const CompletedScrollIntoView: CallableTaggedStruct<"CompletedScrollIntoView", {}>
```

### CompletedUnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L141)

```
/** Sent when the scroll unlock command completes. */
const CompletedUnlockScroll: CallableTaggedStruct<"CompletedUnlockScroll", {}>
```

### DeactivatedItem

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L119)

```
/** Sent when the mouse leaves an enabled item. */
const DeactivatedItem: CallableTaggedStruct<"DeactivatedItem", {}>
```

### DetectMovementOrAnimationEnd

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L383)

```
/** Detects whether the combobox input wrapper moved or the leave animation ended. Whichever comes first; both outcomes signal the Animation submodel that leave is complete. */
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

### FocusInput

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L353)

```
/** Moves focus to the combobox input after selection or close. */
const FocusInput: CommandDefinitionWithArgs<"FocusInput", {
  id: String
}, Effect<{
  _tag: "CompletedFocusInput"
}, never, never>>
```

### GotAnimationMessage

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L167)

```
/** Wraps an Animation submodel message for delegation. */
const GotAnimationMessage: CallableTaggedStruct<"GotAnimationMessage", {
  message: Union<[CallableTaggedStruct<"Showed", {}>, CallableTaggedStruct<"Hid", {}>, CallableTaggedStruct<"CompletedWaitForPaint", {}>, CallableTaggedStruct<"EndedAnimation", {}>]>
}>
```

### InertOthers

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L337)

```
/** Marks all elements outside the combobox as inert for modal behavior. */
const InertOthers: CommandDefinitionWithArgs<"InertOthers", {
  id: String
}, Effect<{
  _tag: "CompletedInertOthers"
}, never, never>>
```

### LockScroll

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L327)

```
/** Prevents page scrolling while the combobox popup is open in modal mode. */
const LockScroll: CommandDefinitionNoArgs<"LockScroll", Effect<{
  _tag: "CompletedLockScroll"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L181)

```
/** Union of all messages the combobox component can produce. */
const Message: S.Union<[typeof Opened, typeof Closed, typeof BlurredInput, typeof ActivatedItem, typeof DeactivatedItem, typeof SelectedItem, typeof MovedPointerOverItem, typeof RequestedItemClick, typeof SuppressedItemCommit, typeof CompletedLockScroll, typeof CompletedUnlockScroll, typeof CompletedInertOthers, typeof CompletedRestoreInert, typeof CompletedFocusInput, typeof CompletedScrollIntoView, typeof CompletedClickItem, typeof CompletedAnchorCombobox, typeof CompletedAttachComboboxPreventBlur, typeof CompletedAttachComboboxSelectOnFocus, typeof CompletedPortalComboboxBackdrop, typeof GotAnimationMessage, typeof UpdatedInputValue, typeof PressedToggleButton]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/single.ts#L26)

```
/** Schema for the single-select combobox's private interaction state (open/closed status, active item, activation trigger, typed input value). The selection is owned by the parent and passed in via `ViewInputs.maybeSelectedValue`. */
const Model: Struct<{
  activationTrigger: Literals<readonly ["Pointer", "Keyboard"]>
  animation: Struct<{
    id: String
    isShowing: Boolean
    transitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
  }>
  id: String
  immediate: Boolean
  inputValue: String
  isAnimated: Boolean
  isModal: Boolean
  isOpen: Boolean
  maybeActiveItemIndex: Option<Number>
  maybeLastPointerPosition: Option<Struct<{
    screenX: Number
    screenY: Number
  }>>
  nullable: Boolean
  selectInputOnFocus: Boolean
}>
```

### MovedPointerOverItem

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L127)

```
/** Sent when the pointer moves over a combobox item. */
const MovedPointerOverItem: CallableTaggedStruct<"MovedPointerOverItem", {
  index: Number
  screenX: Number
  screenY: Number
}>
```

### Opened

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L99)

```
/** Sent when the combobox popup opens. Contains an optional initial active item index. */
const Opened: CallableTaggedStruct<"Opened", {
  maybeActiveItemIndex: Option<Number>
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L272)

```
/** Union of out-messages the combobox component can produce. The parent folds `Selected` into the selection it owns and clears that selection on `ClearedSelection`. */
const OutMessage: Union<readonly [
  CallableTaggedStruct<"Selected", {
    value: String
  }>,
  CallableTaggedStruct<"ClearedSelection", {}>
]>
```

### PortalComboboxBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L834)

```
/**
 * The backdrop-portaling Mount this Combobox renders. Exposed so Scene tests can
 *  call `Scene.Mount.resolve(PortalComboboxBackdrop, CompletedPortalComboboxBackdrop())` to
 *  acknowledge the mount produced by the rendered backdrop.
 */
const PortalComboboxBackdrop: MountDefinitionNoArgs<"PortalComboboxBackdrop", {
  _tag: "CompletedPortalComboboxBackdrop"
}>
```

### PressedToggleButton

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L175)

```
/** Sent when the optional toggle button is clicked. `restingInputValue` is what the input returns to when the press closes the combobox (the parent-owned selection's display text, or empty), computed by the view from `ViewInputs.restingInputValue`. `isClearable` carries whether a close from this press may emit `ClearedSelection`, which a read-only combobox denies. */
const PressedToggleButton: CallableTaggedStruct<"PressedToggleButton", {
  isClearable: Boolean
  restingInputValue: String
}>
```

### RequestedItemClick

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L133)

```
/** Sent when Enter or Space is pressed on the active item, triggering a programmatic click. */
const RequestedItemClick: CallableTaggedStruct<"RequestedItemClick", {
  index: Number
}>
```

### RestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L346)

```
/** Removes the inert attribute from elements outside the combobox. */
const RestoreInert: CommandDefinitionWithArgs<"RestoreInert", {
  id: String
}, Effect<{
  _tag: "CompletedRestoreInert"
}, never, never>>
```

### ScrollIntoView

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L363)

```
/** Scrolls the active combobox item into view after keyboard navigation. */
const ScrollIntoView: CommandDefinitionWithArgs<"ScrollIntoView", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedScrollIntoView"
}, never, never>>
```

### Selected

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L257)

```
/** Sent when the user activates an item. Carries the neutral fact that the item was activated; the parent owns the selection and decides what it means (single-select stores the value, nullable single-select toggles it, multi-select toggles the value's membership). Generic over `Value extends string`: the runtime schema stores `value: string`, but the type-level OutMessage exposes `value: Value` so consumers who supply `items: ReadonlyArray<MyUnion>` receive `value: MyUnion` from the factory's `update` without casting. */
const Selected: CallableTaggedStruct<"Selected", {
  value: String
}>
```

### SelectedItem

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L121)

```
/** Sent when an item is selected via Enter or click. `displayText` is the item's resting input text, and `wasSelected` reports whether the item was already in the parent-owned selection when activated, so nullable deselect logic works without the Model knowing the selection. */
const SelectedItem: CallableTaggedStruct<"SelectedItem", {
  displayText: String
  item: String
  wasSelected: Boolean
}>
```

### SuppressedItemCommit

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L137)

```
/** Sent when Enter is pressed on the active item of a read-only combobox. Update no-ops; the Message exists so the keydown handler returns `Option.some` and calls `preventDefault`, which stops a surrounding form from submitting, and so the keypress stays visible for DevTools. */
const SuppressedItemCommit: CallableTaggedStruct<"SuppressedItemCommit", {}>
```

### UnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L332)

```
/** Re-enables page scrolling after the combobox popup closes. */
const UnlockScroll: CommandDefinitionNoArgs<"UnlockScroll", Effect<{
  _tag: "CompletedUnlockScroll"
}, never, never>>
```

### UpdatedInputValue

const

[source](https://github.com/foldkit/foldkit/blob/75fa2fe95d725060dfa826760924c1790ba9a5bc/packages/ui/src/combobox/shared.ts#L171)

```
/** Sent when the user types in the input. */
const UpdatedInputValue: CallableTaggedStruct<"UpdatedInputValue", {
  value: String
}>
```
