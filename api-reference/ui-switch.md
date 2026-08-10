---
url: https://foldkit.dev/api-reference/ui-switch
title: "Ui/Switch"
description: "API documentation for the Ui/Switch module."
access_date: 2026-08-10T14:39:49.977Z
current_date: 2026-08-10T14:39:49.977Z
---

# Ui/Switch

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/7632ab7e37ad7c9e336559a8993e66cd0c5b28bf/packages/ui/src/switch/index.ts#L57)

```
/** Returns the description element id, derived from the switch's base id. */
(id: string): string
```

### labelId

function

[source](https://github.com/foldkit/foldkit/blob/7632ab7e37ad7c9e336559a8993e66cd0c5b28bf/packages/ui/src/switch/index.ts#L54)

```
/** Returns the label element id, derived from the switch's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/7632ab7e37ad7c9e336559a8993e66cd0c5b28bf/packages/ui/src/switch/index.ts#L81)

```
/**
 * Renders an accessible switch as a stateless controlled component. The
 *  parent owns the checked state (`isChecked`) and receives the new state via
 *  `onToggle` when the user toggles it.
 * 
 *  ```ts
 *  // In view:
 *  Switch.view(
 *    {
 *      id: 'notifications',
 *      isChecked: model.notificationsEnabled,
 *      onToggle: isChecked => ToggledNotifications({ isChecked }),
 *      toView: attributes => ...,
 *    },
 *    h,
 *  )
 * 
 *  // In update:
 *  ToggledNotifications: ({ isChecked }) => [
 *    evo(model, { notificationsEnabled: () => isChecked }),
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

### SwitchAttributes

type

[source](https://github.com/foldkit/foldkit/blob/7632ab7e37ad7c9e336559a8993e66cd0c5b28bf/packages/ui/src/switch/index.ts#L17)

```
/**
 * Attribute groups the switch provides to the consumer's `toView` callback.
 *  Each group is a `ReadonlyArray<Attribute<Message>>` the consumer
 *  spreads into its own element attribute arrays. The `button` and `label`
 *  bundles carry the click and Space handlers that dispatch the configured
 *  `onToggle` Message.
 * 
 *  The `button` bundle sets `type="button"` so that rendering the switch as a
 *  `button` element inside a `form` element toggles without also submitting the
 *  form. Setting it is harmless on the other elements a switch might use, such
 *  as a `div` or a `span`, because the builder assigns a DOM property rather
 *  than an HTML attribute. Spread a later `h.Type` to override it.
 */
type SwitchAttributes = Readonly<{
  button: ReadonlyArray<Attribute<Message>>
  description: ReadonlyArray<Attribute<Message>>
  hiddenInput: ReadonlyArray<Attribute<Message>>
  label: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/7632ab7e37ad7c9e336559a8993e66cd0c5b28bf/packages/ui/src/switch/index.ts#L42)

```
/**
 * Per-render view configuration for the stateless controlled view.
 *  Generic over `Message` (the message `onToggle` dispatches).
 * 
 *  - `isChecked`: the current checked state, read straight from the parent
 *    Model. `aria-checked` and the `data-checked` marker derive from it.
 *  - `onToggle`: dispatched with the new checked state when the user clicks
 *    the switch or its label, or presses Space. Handle it in the parent's
 *    `update` by storing the value.
 *  - `toView`: receives the SwitchAttributes and lays out the
 *    switch.
 *  - `isDisabled`: marks the switch unavailable with `aria-disabled="true"`
 *    and `data-disabled`, keeping it focusable. Use it when the control does
 *    not apply; use `isReadOnly` when its state is still information the user
 *    needs.
 *  - `isReadOnly`: prevents toggling while exposing read-only semantics with
 *    `aria-readonly="true"` and `data-readonly`. The switch remains
 *    focusable. Independent of `isDisabled`: setting both emits both
 *    attribute sets, and either one removes the interaction handlers.
 */
type ViewConfig = Readonly<{
  id: string
  isChecked: boolean
  isDisabled: boolean
  isReadOnly: boolean
  name: string
  onToggle: (isChecked: boolean) => Message
  toView: (attributes: SwitchAttributes<Message>) => Html
  value: string
}>
```
