---
url: https://foldkit.dev/api-reference/ui-listbox
title: "Ui/Listbox"
description: "API documentation for the Ui/Listbox module."
access_date: 2026-09-18T16:33:34.359Z
current_date: 2026-09-18T16:33:34.359Z
---

# Ui/Listbox

## Functions

### buttonId

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L195)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/single.ts#L135)

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
 *  // In the parent update, pass ColorListbox.update to Update.foldChild and
 *  // handle Listbox.OutMessage<Color> in foldOutMessage.
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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/single.ts#L32)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L51)

```
/** Schema for the activation trigger: whether the user interacted via mouse or keyboard. */
type ActivationTrigger = Literals<readonly ["Pointer", "Keyboard"]>
```

### BaseViewInputsCommon

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L683)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  The Listbox emits a `Selected({ value })` OutMessage on commit.
 *  Fold it in the Listbox's `Update.foldChild` config: single-select stores
 *  the value, while multi-select toggles its membership.
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
  itemToSearchText: (item: Item, index: number) => string
  name: string
  separatorAttributes: ReadonlyArray<ChildAttribute>
  separatorClassName: string
}>
```

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/single.ts#L105)

```
/**
 * The `view`, `update`, and programmatic helpers that `Listbox.create`
 *  returns, bound to one `Item` and `Value` pair. Name it to annotate a
 *  value that holds a created bundle, such as a field on a config object
 *  or a function parameter that takes the bundle rather than calling
 *  `create` itself.
 */
type Bundle = Readonly<{
  close: (model: Model) => BundleUpdateReturn<Value>
  open: (model: Model) => BundleUpdateReturn<Value>
  selectItem: (model: Model, item: Value) => BundleUpdateReturn<Value>
  update: (model: Model, message: Message) => BundleUpdateReturn<Value>
  view: SubmodelView<Model, Message, ViewInputs<Item, Value>>
}>
```

### GroupHeading

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L673)

```
/** Configuration for a group heading rendered above a group of items. */
type GroupHeading = Readonly<{
  className: string
  content: Html
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/single.ts#L29)

```
/** Configuration for creating a single-select listbox model with `init`. `isAnimated` enables CSS transition coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). */
type InitConfig = BaseInitConfig
```

### ItemConfig

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L667)

```
/** Configuration for an individual listbox item's appearance. */
type ItemConfig = Readonly<{
  className: string
  content: Html
}>
```

### ItemToValueInput

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L741)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L171)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L165)

```
type Selected = Readonly<{
  _tag: "Selected"
  value: Value
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/single.ts#L64)

```
/** Per-render view inputs passed to the view via `h.submodel`'s `viewInputs` field. */
type ViewInputs = BaseViewInputsCommon<Item> & Readonly<{
  maybeSelectedValue: Option.Option<Value>
}> & ItemToValueInput<Item, Value>
```

## Constants

### AnchorListbox

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L630)

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
  _tag: "CompletedAnchorListbox"
}>
```

### ClickItem

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L287)

```
/** Programmatically clicks the active listbox item's DOM element. */
const ClickItem: CommandDefinitionWithArgs<"ClickItem", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedClickItem"
}, never, never>>
```

### DelayClearSearch

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L297)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L306)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L257)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L267)

```
/** Moves focus to the listbox items container after opening. */
const FocusItems: CommandDefinitionWithArgs<"FocusItems", {
  id: String
}, Effect<{
  _tag: "CompletedFocusItems"
}, never, never>>
```

### InertOthers

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L241)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L231)

```
/** Prevents page scrolling while the listbox is open in modal mode. */
const LockScroll: CommandDefinitionNoArgs<"LockScroll", Effect<{
  _tag: "CompletedLockScroll"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L104)

```
/** Union of all messages the listbox component can produce. */
const Message: MessageUnion<{
  ActivatedItem: {
    activationTrigger: Literals<readonly ["Pointer", "Keyboard"]>
    index: Number
  }
  BlurredItems: {}
  Closed: {}
  CompletedAnchorListbox: {}
  CompletedClickItem: {}
  CompletedDelayClearSearch: {
    version: Number
  }
  CompletedFocusButton: {}
  CompletedFocusItems: {}
  CompletedInertOthers: {}
  CompletedLockScroll: {}
  CompletedPortalListboxBackdrop: {}
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
  }
  RequestedItemClick: {
    index: Number
  }
  Searched: {
    key: String
    maybeTargetIndex: Option<Number>
  }
  SelectedItem: {
    item: String
  }
  SuppressedItemCommit: {}
  SuppressedSpaceScroll: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/single.ts#L20)

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

### Orientation

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L55)

```
/** Schema for the listbox orientation: whether items flow vertically or horizontally. */
const Orientation: Literals<readonly ["Vertical", "Horizontal"]>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L171)

```
/** Union of out-messages the listbox component can produce. The parent folds `Selected` into the selection it owns: single-select stores the value, multi-select toggles the value's membership. */
const OutMessage: MessageUnion<{
  Selected: {
    value: String
  }
}>
```

### PortalListboxBackdrop

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L652)

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

### RestoreInert

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L250)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L277)

```
/** Scrolls the active listbox item into view after keyboard navigation. */
const ScrollIntoView: CommandDefinitionWithArgs<"ScrollIntoView", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedScrollIntoView"
}, never, never>>
```

### UnlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/listbox/shared.ts#L236)

```
/** Re-enables page scrolling after the listbox closes. */
const UnlockScroll: CommandDefinitionNoArgs<"UnlockScroll", Effect<{
  _tag: "CompletedUnlockScroll"
}, never, never>>
```
