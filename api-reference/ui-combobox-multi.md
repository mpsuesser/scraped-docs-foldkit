---
url: https://foldkit.dev/api-reference/ui-combobox-multi
title: "Ui/Combobox/Multi"
description: "API documentation for the Ui/Combobox/Multi module."
access_date: 2026-09-07T07:29:31.695Z
current_date: 2026-09-07T07:29:31.695Z
---

# Ui/Combobox/Multi

## Functions

### create

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/combobox/multi.ts#L109)

```
/**
 * Pairs the multi-select combobox's `view` and `update` (and programmatic
 *  helpers) behind a single Item-typed entry point. `selectItem` emits
 *  `Selected({ value })`; the parent toggles the value's membership.
 */
<Item extends string = string>(): Bundle<Item>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/combobox/multi.ts#L33)

```
/** Creates an initial multi-select combobox model from a config. Defaults to closed with no active item and an empty input. */
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

## Types

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/combobox/multi.ts#L98)

```
/**
 * The `view`, `update`, and programmatic helpers that
 *  `Combobox.Multi.create` returns, bound to one `Item` type. Name it to
 *  annotate a value that holds a created bundle, such as a field on a
 *  config object or a function parameter that takes the bundle rather than
 *  calling `create` itself.
 */
type Bundle = Readonly<{
  close: (model: Model) => BundleUpdateReturn<Item>
  open: (model: Model) => BundleUpdateReturn<Item>
  selectItem: (model: Model, item: Item) => BundleUpdateReturn<Item>
  update: (model: Model, message: Message) => BundleUpdateReturn<Item>
  view: SubmodelView<Model, Message, ViewInputs<Item>>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/combobox/multi.ts#L30)

```
/** Configuration for creating a multi-select combobox model with `init`. `isAnimated` enables CSS transition coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). */
type InitConfig = BaseInitConfig
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/combobox/multi.ts#L83)

```
/** Per-render view inputs passed to the view via `h.submodel`'s `viewInputs` field. */
type ViewInputs = BaseViewInputs<Item>
```

## Constants

### Model

const

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/combobox/multi.ts#L21)

```
/** Schema for the multi-select combobox's private interaction state (open/closed status, active item, activation trigger, typed input value). The selection is owned by the parent and passed in via `ViewInputs.selectedValues`. */
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
