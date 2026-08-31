---
url: https://foldkit.dev/api-reference/ui-combobox
title: "Ui/Combobox"
description: "API documentation for the Ui/Combobox module."
access_date: 2026-08-31T07:29:25.100Z
current_date: 2026-08-31T07:29:25.100Z
---

# Ui/Combobox

## Functions

### create

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/single.ts#L144)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/single.ts#L33)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L198)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L44)

```
/** Schema for the activation trigger: whether the user interacted via mouse or keyboard. */
type ActivationTrigger = Literals<readonly ["Pointer", "Keyboard"]>
```

### BaseViewInputsCommon

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L766)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  The Combobox emits a `Selected({ value })` OutMessage on commit.
 *  Fold it in the Combobox's `Update.foldChild` config: single-select
 *  stores the value, while multi-select toggles the value's membership.
 *  `restingInputValue` is the text the input returns to on
 *  close (the selection's display text for single-select, empty for
 *  multi-select). The selection is not part of this shared shape. Each
 *  variant adds its own selection field.
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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/single.ts#L126)

```
/**
 * The `view`, `update`, and programmatic helpers that `Combobox.create`
 *  returns, bound to one `Item` type. Name it to annotate a value that
 *  holds a created bundle, such as a field on a config object or a
 *  function parameter that takes the bundle rather than calling `create`
 *  itself.
 */
type Bundle = Readonly<{
  close: (model: Model, restingInputValue: string) => BundleUpdateReturn<Item>
  open: (model: Model) => BundleUpdateReturn<Item>
  selectItem: (model: Model, item: Item, displayText: string) => BundleUpdateReturn<Item>
  update: (model: Model, message: Message) => BundleUpdateReturn<Item>
  view: SubmodelView<Model, Message, ViewInputs<Item>>
}>
```

### GroupHeading

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L752)

```
/** Configuration for a group heading rendered above a group of items. */
type GroupHeading = Readonly<{
  className: string
  content: Html
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/single.ts#L30)

```
/** Configuration for creating a single-select combobox model with `init`. `isAnimated` enables CSS transition coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). */
type InitConfig = BaseInitConfig
```

### ItemConfig

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L746)

```
/** Configuration for an individual combobox item's appearance. */
type ItemConfig = Readonly<{
  className: string
  content: Html
}>
```

### OutMessage

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L179)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L171)

```
type Selected = Readonly<{
  _tag: "Selected"
  value: Value
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/single.ts#L104)

```
/** Per-render view inputs passed to the view via `h.submodel`'s `viewInputs` field. */
type ViewInputs = BaseViewInputsCommon<Item> & Readonly<{
  maybeSelectedValue: Option.Option<Item>
}>
```

## Constants

### AnchorCombobox

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L640)

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
  _tag: "CompletedAnchorCombobox"
}>
```

### AttachComboboxPreventBlur

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L675)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L704)

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

### ClickItem

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L283)

```
/** Programmatically clicks the active combobox item's DOM element. */
const ClickItem: CommandDefinitionWithArgs<"ClickItem", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedClickItem"
}, never, never>>
```

### DetectMovementOrAnimationEnd

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L293)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L263)

```
/** Moves focus to the combobox input after selection or close. */
const FocusInput: CommandDefinitionWithArgs<"FocusInput", {
  id: String
}, Effect<{
  _tag: "CompletedFocusInput"
}, never, never>>
```

### InertOthers

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L247)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L237)

```
/** Prevents page scrolling while the combobox popup is open in modal mode. */
const LockScroll: CommandDefinitionNoArgs<"LockScroll", Effect<{
  _tag: "CompletedLockScroll"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L100)

```
/** Union of all messages the combobox component can produce. */
const Message: MessageUnion<{
  ActivatedItem: {
    activationTrigger: Literals<readonly ["Pointer", "Keyboard"]>
    index: Number
    maybeImmediateSelection: Option<Struct<{
      item: String
    }>>
  }
  BlurredInput: {
    isClearable: Boolean
    restingInputValue: String
  }
  Closed: {
    isClearable: Boolean
    restingInputValue: String
  }
  CompletedAnchorCombobox: {}
  CompletedAttachComboboxPreventBlur: {}
  CompletedAttachComboboxSelectOnFocus: {}
  CompletedClickItem: {}
  CompletedFocusInput: {}
  CompletedInertOthers: {}
  CompletedLockScroll: {}
  CompletedPortalComboboxBackdrop: {}
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
  MovedPointerOverItem: {
    index: Number
    screenX: Number
    screenY: Number
  }
  Opened: {
    maybeActiveItemIndex: Option<Number>
  }
  PressedToggleButton: {
    isClearable: Boolean
    restingInputValue: String
  }
  RequestedItemClick: {
    index: Number
  }
  SelectedItem: {
    displayText: String
    item: String
    wasSelected: Boolean
  }
  SuppressedItemCommit: {}
  UpdatedInputValue: {
    value: String
  }
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/single.ts#L21)

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

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L179)

```
/** Union of out-messages the combobox component can produce. The parent folds `Selected` into the selection it owns and clears that selection on `ClearedSelection`. */
const OutMessage: MessageUnion<{
  ClearedSelection: {}
  Selected: {
    value: String
  }
}>
```

### PortalComboboxBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L731)

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

### RestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L256)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L273)

```
/** Scrolls the active combobox item into view after keyboard navigation. */
const ScrollIntoView: CommandDefinitionWithArgs<"ScrollIntoView", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedScrollIntoView"
}, never, never>>
```

### UnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/ui/src/combobox/shared.ts#L242)

```
/** Re-enables page scrolling after the combobox popup closes. */
const UnlockScroll: CommandDefinitionNoArgs<"UnlockScroll", Effect<{
  _tag: "CompletedUnlockScroll"
}, never, never>>
```
