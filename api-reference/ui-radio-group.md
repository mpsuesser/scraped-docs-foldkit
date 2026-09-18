---
url: https://foldkit.dev/api-reference/ui-radio-group
title: "Ui/RadioGroup"
description: "API documentation for the Ui/RadioGroup module."
access_date: 2026-09-18T18:12:22.916Z
current_date: 2026-09-18T18:12:22.916Z
---

# Ui/RadioGroup

## Functions

### create

function

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L461)

```
/**
 * Pairs the radio group `view` and `update` behind a single Value-typed
 *  entry point. Declare once at module scope so consumers receive
 *  `option.value: Value` in `toView` and the `Selected` OutMessage without an
 *  `as` cast:
 * 
 *  ```ts
 *  const PlanRadioGroup = RadioGroup.create<Plan>()
 * 
 *  // In view (selectedValue is the parent-owned selection):
 *  h.submodel({ view: PlanRadioGroup.view, viewInputs: { selectedValue, ... }, ... })
 * 
 *  // In the parent update, pass PlanRadioGroup.update to Update.foldChild
 *  // and fold the Selected OutMessage into your Model.
 *  ```
 * 
 *  The internal view stays typed `ReadonlyArray<string>`; consumers can
 *  pass a `ReadonlyArray<MyUnion>` (assignable) and the fenced cast inside
 *  `create` types `OptionInfo.value` as `MyUnion`.
 */
<Value extends string = string>(): Bundle<Value>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L89)

```
/**
 * Creates an initial radio group model from a config. Focus follows the
 *  selected option until the user navigates a read-only group, so
 *  `maybeFocusedIndex` starts `None`.
 */
(config: InitConfig): RadioGroup.Model
```

## Types

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L435)

```
/**
 * The `view` and `update` pair that `RadioGroup.create` returns, bound to one
 *  `Value` type. Name it to annotate a value that holds a created bundle,
 *  such as a field on a config object or a function parameter that takes
 *  the bundle rather than calling `create` itself.
 */
type Bundle = Readonly<{
  update: (model: Model, message: Message) => Update.ReturnWithOutMessage<Model, Message, OutMessage<Value>>
  view: SubmodelView<Model, Message, ViewInputs<Value>>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L82)

```
/** Configuration for creating a radio group model with `init`. */
type InitConfig = Readonly<{
  id: string
}>
```

### OptionInfo

type

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L147)

```
/**
 * Per-option render info passed to the consumer's `toView`. The consumer
 *  spreads `option`, `label`, and `description` onto whichever elements carry
 *  that role in their layout. Generic over `Value extends string` so
 *  `option.value` carries the consumer's union type.
 * 
 *  The `option` bundle sets `type="button"` so that rendering the option as a
 *  `button` element inside a `form` element selects without also submitting the
 *  form. Setting it is harmless on the other elements an option might use, such
 *  as a `div` or a `span`, because the builder assigns a DOM property rather
 *  than an HTML attribute. Spread a later `h.Type` to override it.
 */
type OptionInfo = Readonly<{
  description: ReadonlyArray<ChildAttribute>
  index: number
  isActive: boolean
  isDisabled: boolean
  isReadOnly: boolean
  isSelected: boolean
  label: ReadonlyArray<ChildAttribute>
  option: ReadonlyArray<ChildAttribute>
  value: Value
}>
```

### OutMessage

type

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L67)

```
/**
 * Generic over `Value extends string` so consumers using
 *  `RadioGroup.create<MyUnion>()` receive `value: MyUnion` in the
 *  `Selected` OutMessage. Defaults to `string`.
 */
type OutMessage = Selected<Value>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L171)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 *  - `group`: ARIA + role attributes for the wrapping radiogroup element.
 *  - `options`: one entry per option in `viewInputs.options`, in the same
 *    order. Includes the value, derived state, and the attribute bundles for
 *    the option element, its label, and its description.
 *  - `selectedValue`: the currently-selected value, if any. Convenient for the
 *    consumer when rendering selected-state visuals next to the option
 *    attributes.
 *  - `hiddenInput`: when `name` was supplied, attributes for a hidden form
 *    input carrying the selected value. The consumer renders the `<input>`
 *    themselves. Empty array when `name` is undefined.
 */
type RenderInfo = Readonly<{
  group: ReadonlyArray<ChildAttribute>
  hiddenInput: ReadonlyArray<ChildAttribute>
  options: ReadonlyArray<OptionInfo<Value>>
  selectedValue: Option.Option<Value>
}>
```

### Selected

type

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L59)

```
type Selected = Readonly<{
  _tag: "Selected"
  index: number
  value: Value
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L191)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs`
 *  field. Generic over `Value extends string` so consumers using
 *  `RadioGroup.create<MyUnion>()` receive `option.value: MyUnion` in `toView`
 *  and `(value: MyUnion, index) => boolean` in `isOptionDisabled`, without
 *  casting.
 * 
 *  - `selectedValue`: the current selection, read straight from the parent
 *    Model. `aria-checked` and the `data-checked` marker derive from it, as
 *    does the roving tabindex whenever keyboard focus has not diverged.
 *  - `isReadOnly`: keeps the group navigable but not selectable. Arrow, Home,
 *    End, PageUp, and PageDown still move focus, and the group reports that
 *    focus through `FocusedOption`, so `data-active` and `tabindex` follow it.
 *    Space and clicking do nothing.
 */
type ViewInputs = Readonly<{
  ariaLabel: string
  hasOptionDescription: (value: Value, index: number) => boolean
  isDisabled: boolean
  isOptionDisabled: (value: Value, index: number) => boolean
  isReadOnly: boolean
  name: string
  options: ReadonlyArray<Value>
  orientation: Orientation
  selectedValue: Option.Option<Value>
  toView: (render: RenderInfo<Value>) => Html
}>
```

## Constants

### FocusOption

const

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L105)

```
/** Moves focus to the option at the given index. */
const FocusOption: CommandDefinitionWithArgs<"FocusOption", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedFocusOption"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L43)

```
/** Union of all messages the radio group can produce. */
const Message: MessageUnion<{
  CompletedFocusOption: {}
  FocusedOption: {
    index: Number
  }
  SelectedOption: {
    index: Number
    value: String
  }
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L33)

```
/**
 * Schema for the radio group's private interaction state. The selected
 *  option is owned by the parent and passed in via `ViewInputs.selectedValue`,
 *  so it is not stored here. `maybeFocusedIndex` is the roving-tabindex
 *  cursor: `None` means keyboard focus follows the selection, and a read-only
 *  group stores `Some(index)` while focus diverges from it.
 */
const Model: Struct<{
  id: String
  maybeFocusedIndex: Option<Number>
}>
```

### Orientation

const

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L25)

```
/** Controls the radio group layout direction and which arrow keys navigate between options. */
const Orientation: Literals<readonly ["Horizontal", "Vertical"]>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/a5413bbc0fc8f6578285632d7bbd7847efcf1a09/packages/ui/src/radioGroup/index.ts#L67)

```
/**
 * Union of OutMessages the radio group can produce. The parent's
 *  `Update.foldChild` config handles them through `foldOutMessage`.
 */
const OutMessage: MessageUnion<{
  Selected: {
    index: Number
    value: String
  }
}>
```
