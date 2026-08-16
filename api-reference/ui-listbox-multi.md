---
url: https://foldkit.dev/api-reference/ui-listbox-multi
title: "Ui/Listbox/Multi"
description: "API documentation for the Ui/Listbox/Multi module."
access_date: 2026-08-16T19:09:52.991Z
current_date: 2026-08-16T19:09:52.991Z
---

# Ui/Listbox/Multi

## Functions

### create

function

[source](https://github.com/foldkit/foldkit/blob/7014ef6d904c897802df3797298255bb3ce65498/packages/ui/src/listbox/multi.ts#L120)

```
/**
 * Pairs the multi-select listbox's `view` and `update` (and programmatic
 *  helpers) behind a single Item-typed entry point. Same shape as
 *  `Listbox.create`. Two type params support object-typed items via
 *  `itemToValue`: `Value` defaults to `Item` when `Item extends string`,
 *  else `string`.
 */
<Item = string, Value extends string = Item extends string
  ? Item
  : string>(): Bundle<Item, Value>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/7014ef6d904c897802df3797298255bb3ce65498/packages/ui/src/listbox/multi.ts#L35)

```
/** Creates an initial multi-select listbox model from a config. Defaults to closed with no active item. */
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

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/7014ef6d904c897802df3797298255bb3ce65498/packages/ui/src/listbox/multi.ts#L78)

```
/**
 * The `view`, `update`, and programmatic helpers that
 *  `Listbox.Multi.create` returns, bound to one `Item` and `Value` pair.
 *  Name it to annotate a value that holds a created bundle, such as a
 *  field on a config object or a function parameter that takes the bundle
 *  rather than calling `create` itself.
 */
type Bundle = Readonly<{
  close: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Value>>]
  open: (model: Model) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Value>>]
  selectItem: (model: Model, item: Value) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Value>>]
  update: (model: Model, message: Message) => readonly [Model, ReadonlyArray<Command.Command<Message>>, Option.Option<OutMessage<Value>>]
  view: SubmodelView<Model, Message, ViewInputs<Item, Value>>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/7014ef6d904c897802df3797298255bb3ce65498/packages/ui/src/listbox/multi.ts#L32)

```
/** Configuration for creating a multi-select listbox model with `init`. `isAnimated` enables CSS transition coordination (default `false`). `isModal` locks page scroll and inerts other elements when open (default `false`). */
type InitConfig = BaseInitConfig
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/7014ef6d904c897802df3797298255bb3ce65498/packages/ui/src/listbox/multi.ts#L66)

```
/** Per-render view inputs passed to the view via `h.submodel`'s `viewInputs` field. */
type ViewInputs = BaseViewInputs<Item, Value>
```

## Constants

### Model

const

[source](https://github.com/foldkit/foldkit/blob/7014ef6d904c897802df3797298255bb3ce65498/packages/ui/src/listbox/multi.ts#L23)

```
/** Schema for the multi-select listbox's private interaction state (open/closed status, active item, activation trigger, typeahead search). The selection is owned by the parent and passed in via `ViewInputs.selectedValues`. */
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
