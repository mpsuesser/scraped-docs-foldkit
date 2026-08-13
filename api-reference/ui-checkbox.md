---
url: https://foldkit.dev/api-reference/ui-checkbox
title: "Ui/Checkbox"
description: "API documentation for the Ui/Checkbox module."
access_date: 2026-08-13T04:22:29.183Z
current_date: 2026-08-13T04:22:29.183Z
---

# Ui/Checkbox

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/df7c0dd15d94b9fe623d2ea8e4574dbd6619a269/packages/ui/src/checkbox/index.ts#L67)

```
/** Returns the description element id, derived from the checkbox's base id. */
(id: string): string
```

### labelId

function

[source](https://github.com/foldkit/foldkit/blob/df7c0dd15d94b9fe623d2ea8e4574dbd6619a269/packages/ui/src/checkbox/index.ts#L64)

```
/** Returns the label element id, derived from the checkbox's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/df7c0dd15d94b9fe623d2ea8e4574dbd6619a269/packages/ui/src/checkbox/index.ts#L91)

```
/**
 * Renders an accessible checkbox as a stateless controlled component. The
 *  parent owns the checked state (`isChecked`) and receives the new state via
 *  `onToggle` when the user toggles it.
 * 
 *  ```ts
 *  // In view:
 *  Checkbox.view(
 *    {
 *      id: 'accept-terms',
 *      isChecked: model.acceptedTerms,
 *      onToggle: isChecked => ToggledTerms({ isChecked }),
 *      toView: attributes => ...,
 *    },
 *    h,
 *  )
 * 
 *  // In update:
 *  ToggledTerms: ({ isChecked }) => [
 *    evo(model, { acceptedTerms: () => isChecked }),
 *    [],
 *  ],
 *  ```
 */
<Message>(
  config: ViewConfig<Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### CheckboxAttributes

type

[source](https://github.com/foldkit/foldkit/blob/df7c0dd15d94b9fe623d2ea8e4574dbd6619a269/packages/ui/src/checkbox/index.ts#L26)

```
/**
 * Attribute groups the checkbox provides to the consumer's `toView`
 *  callback. Each group is a `ReadonlyArray<Attribute<Message>>` the
 *  consumer spreads directly into its own element attribute arrays:
 * 
 *  ```ts
 *  toView: attributes =>
 *    h.div(
 *      [...attributes.checkbox, h.Class('my-class')],
 *      [...],
 *    )
 *  ```
 * 
 *  The `checkbox` and `label` bundles carry the click and Space handlers that
 *  dispatch the configured `onToggle` Message.
 * 
 *  The `checkbox` bundle sets `type="button"` so that rendering the control as a
 *  `button` element inside a `form` element toggles without also submitting the
 *  form. Setting it is harmless on the other elements a control might use, such
 *  as a `div` or a `span`, because the builder assigns a DOM property rather
 *  than an HTML attribute. Spread a later `h.Type` to override it.
 */
type CheckboxAttributes = Readonly<{
  checkbox: ReadonlyArray<Attribute<Message>>
  description: ReadonlyArray<Attribute<Message>>
  hiddenInput: ReadonlyArray<Attribute<Message>>
  label: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/df7c0dd15d94b9fe623d2ea8e4574dbd6619a269/packages/ui/src/checkbox/index.ts#L51)

```
/**
 * Per-render view configuration for the stateless controlled view.
 *  Generic over `Message` (the message `onToggle` dispatches).
 * 
 *  - `isChecked`: the current checked state, read straight from the parent
 *    Model. `aria-checked` and the `data-checked` marker derive from it.
 *  - `onToggle`: dispatched with the new checked state when the user clicks
 *    the checkbox or its label, or presses Space. Handle it in the parent's
 *    `update` by storing the value.
 *  - `toView`: receives the CheckboxAttributes and lays out the
 *    checkbox.
 *  - `isDisabled`: marks the checkbox unavailable with `aria-disabled="true"`
 *    and `data-disabled`, keeping it focusable. Use it when the control does
 *    not apply; use `isReadOnly` when its state is still information the user
 *    needs.
 *  - `isReadOnly`: prevents toggling while exposing read-only semantics with
 *    `aria-readonly="true"` and `data-readonly`. The checkbox remains
 *    focusable. Independent of `isDisabled`: setting both emits both
 *    attribute sets, and either one removes the interaction handlers.
 */
type ViewConfig = Readonly<{
  id: string
  isChecked: boolean
  isDisabled: boolean
  isIndeterminate: boolean
  isReadOnly: boolean
  name: string
  onToggle: (isChecked: boolean) => Message
  toView: (attributes: CheckboxAttributes<Message>) => Html
  value: string
}>
```
