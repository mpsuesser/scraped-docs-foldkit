---
url: https://foldkit.dev/api-reference/ui-combobox-multi
title: "Ui/Combobox/Multi"
description: "API documentation for the Ui/Combobox/Multi module."
access_date: 2026-08-07T17:24:33.589Z
current_date: 2026-08-07T17:24:33.589Z
---

# Ui/Combobox/Multi

## Functions

### create

function

[source](https://github.com/foldkit/foldkit/blob/1995dedf6b1fb51cd51dca135cbc2e9540b690fb/packages/ui/src/combobox/multi.ts#L137)

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

[source](https://github.com/foldkit/foldkit/blob/1995dedf6b1fb51cd51dca135cbc2e9540b690fb/packages/ui/src/combobox/multi.ts#L38)

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

[source](https://github.com/foldkit/foldkit/blob/1995dedf6b1fb51cd51dca135cbc2e9540b690fb/packages/ui/src/combobox/multi.ts#L100)

```
/**
 * The `view`, `update`, and programmatic helpers that
 *  `Combobox.Multi.create` returns, bound to one `Item` type. Name it to
 *  annotate a value that holds a created bundle, such as a field on a
 *  config object or a function parameter that takes the bundle rather than
 *  calling `create` itself.
 */
type Bundle = Readonly<{
  close: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  open: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  selectItem: (model: Model, item: Item) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  update: (model: Model, message: Message) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Item>>]
  view: SubmodelView<Model, Message, ViewInputs<Item>>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/1995dedf6b1fb51cd51dca135cbc2e9540b690fb/packages/ui/src/combobox/multi.ts#L35)

```
/** Configuration for creating a multi-select combobox model with `init`. `isAnimated` enables CSS transition coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). */
type InitConfig = BaseInitConfig
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/1995dedf6b1fb51cd51dca135cbc2e9540b690fb/packages/ui/src/combobox/multi.ts#L91)

```
/** Per-render view inputs passed to the view via `h.submodel`'s `viewInputs` field. */
type ViewInputs = BaseViewInputs<Item>
```

## Constants

### Model

const

[source](https://github.com/foldkit/foldkit/blob/1995dedf6b1fb51cd51dca135cbc2e9540b690fb/packages/ui/src/combobox/multi.ts#L26)

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
