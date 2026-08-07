---
url: https://foldkit.dev/api-reference/ui-radio-group
title: "Ui/RadioGroup"
description: "API documentation for the Ui/RadioGroup module."
access_date: 2026-08-07T15:54:22.903Z
current_date: 2026-08-07T15:54:22.903Z
---

# Ui/RadioGroup

## Functions

### view

function

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/radioGroup/index.ts#L120)

```
/**
 * Renders an accessible radio group as a stateless controlled component. The
 *  parent owns the selection (`selectedValue`) and receives the parent's own
 *  Message via `onSelect` when an option is chosen. Moving focus onto the
 *  newly-active option is the radio group's own concern: it happens inside the
 *  component's click and keydown handlers, so the parent's `update` only stores
 *  the value.
 * 
 *  ```ts
 *  // In view:
 *  RadioGroup.view(
 *    {
 *      id: TOOL_RADIO_GROUP_ID,
 *      selectedValue: Option.some(model.tool),
 *      options: TOOLS,
 *      ariaLabel: 'Tool',
 *      onSelect: tool => SelectedTool({ tool }),
 *      toView: ({ group, options }) => ...,
 *    },
 *    h,
 *  )
 * 
 *  // In update:
 *  SelectedTool: ({ tool }) => [evo(model, { tool: () => tool }), []],
 *  ```
 */
<Value extends string, Message>(
  config: ViewConfig<Value, Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### OptionInfo

type

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/radioGroup/index.ts#L42)

```
/**
 * Per-option render info passed to the consumer's `toView`. The consumer
 *  spreads `option`, `label`, and `description` onto whichever elements carry
 *  that role in their layout. Generic over `Value extends string` so
 *  `option.value` carries the consumer's union type, and over `Message`
 *  so the attribute bundles dispatch the parent's own Message.
 * 
 *  The `option` bundle sets `type="button"` so that rendering the option as a
 *  `button` element inside a `form` element selects without also submitting the
 *  form. Setting it is harmless on the other elements an option might use, such
 *  as a `div` or a `span`, because the builder assigns a DOM property rather
 *  than an HTML attribute. Spread a later `h.Type` to override it.
 */
type OptionInfo = Readonly<{
  description: ReadonlyArray<Attribute<Message>>
  index: number
  isActive: boolean
  isDisabled: boolean
  isSelected: boolean
  label: ReadonlyArray<Attribute<Message>>
  option: ReadonlyArray<Attribute<Message>>
  value: Value
}>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/radioGroup/index.ts#L65)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 *  - `group`: ARIA + role attributes for the wrapping radiogroup element.
 *  - `options`: one entry per option in `options`, in the same order. Includes
 *    the value, derived state, and the attribute bundles for the option
 *    element, its label, and its description.
 *  - `selectedValue`: the currently-selected value, if any. Convenient for the
 *    consumer when rendering selected-state visuals next to the option
 *    attributes.
 *  - `hiddenInput`: when `name` was supplied, attributes for a hidden form
 *    input carrying the selected value. The consumer renders the `<input>`
 *    themselves. Empty array when `name` is undefined.
 */
type RenderInfo = Readonly<{
  group: ReadonlyArray<Attribute<Message>>
  hiddenInput: ReadonlyArray<Attribute<Message>>
  options: ReadonlyArray<OptionInfo<Value, Message>>
  selectedValue: Option.Option<Value>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/radioGroup/index.ts#L83)

```
/**
 * Per-render view configuration for the stateless controlled view.
 *  Generic over `Value extends string` (the option union) and `Message`
 *  (the message `onSelect` dispatches).
 * 
 *  - `selectedValue`: the current selection, read straight from the parent
 *    Model. The roving tabindex and checked state derive from it.
 *  - `onSelect`: dispatched with the chosen value when an option is clicked or
 *    navigated to. Handle it in the parent's `update` by storing the value.
 *    The radio group manages focus itself, so the handler needs to do nothing
 *    else.
 *  - `toView`: receives the RenderInfo and lays out the group.
 */
type ViewConfig = Readonly<{
  ariaLabel: string
  id: string
  isDisabled: boolean
  isOptionDisabled: (value: Value, index: number) => boolean
  name: string
  onSelect: (value: Value) => Message
  options: ReadonlyArray<Value>
  orientation: Orientation
  selectedValue: Option.Option<Value>
  toView: (render: RenderInfo<Value, Message>) => Html
}>
```

## Constants

### Orientation

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/radioGroup/index.ts#L18)

```
/** Controls the radio group layout direction and which arrow keys navigate between options. */
const Orientation: Literals<readonly ["Horizontal", "Vertical"]>
```
