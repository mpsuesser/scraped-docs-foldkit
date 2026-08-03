---
url: https://foldkit.dev/api-reference/ui-checkbox
title: "Ui/Checkbox"
description: "API documentation for the Ui/Checkbox module."
access_date: 2026-08-03T18:23:29.850Z
current_date: 2026-08-03T18:23:29.850Z
---

# Ui/Checkbox

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/checkbox/index.ts#L52)

```
/** Generates the description element ID from the checkbox's base ID. */
(id: string): string
```

### labelId

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/checkbox/index.ts#L49)

```
/** Generates the label element ID from the checkbox's base ID. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/checkbox/index.ts#L76)

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

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/checkbox/index.ts#L20)

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

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/checkbox/index.ts#L37)

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
 */
type ViewConfig = Readonly<{
  id: string
  isChecked: boolean
  isDisabled: boolean
  isIndeterminate: boolean
  name: string
  onToggle: (isChecked: boolean) => Message
  toView: (attributes: CheckboxAttributes<Message>) => Html
  value: string
}>
```
