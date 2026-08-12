---
url: https://foldkit.dev/api-reference/ui-disclosure
title: "Ui/Disclosure"
description: "API documentation for the Ui/Disclosure module."
access_date: 2026-08-12T19:19:57.990Z
current_date: 2026-08-12T19:19:57.990Z
---

# Ui/Disclosure

## Functions

### buttonId

function

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/ui/src/disclosure/index.ts#L10)

```
/**
 * Returns the bare DOM id of the disclosure toggle button, derived from the
 *  disclosure's base id. Use this to associate an external label with the
 *  button via a native `<label for={Disclosure.buttonId(id)}>` or an
 *  `aria-labelledby` reference.
 */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/ui/src/disclosure/index.ts#L83)

```
/**
 * Renders a headless disclosure as a stateless controlled component. The
 *  parent owns the open state (`isOpen`) and receives the new state via
 *  `onToggle` when the user toggles it. The consumer composes the layout
 *  through `toView`, spreading the `button` and `panel` bundles onto their own
 *  elements.
 * 
 *  ```ts
 *  // In view:
 *  Disclosure.view(
 *    {
 *      id: 'details',
 *      isOpen: model.isDetailsOpen,
 *      onToggle: isOpen => ToggledDetails({ isOpen }),
 *      toView: ({ button, panel, animatePanel }) => ...,
 *    },
 *    h,
 *  )
 * 
 *  // In update:
 *  ToggledDetails: ({ isOpen }) => [
 *    evo(model, { isDetailsOpen: () => isOpen }),
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

### DisclosureAttributes

type

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/ui/src/disclosure/index.ts#L26)

```
/**
 * Attribute groups the disclosure provides to the consumer's `toView`
 *  callback. The consumer composes the button + panel layout themselves using
 *  these bundles. The `button` bundle carries the click and Enter/Space
 *  handlers that dispatch the configured `onToggle` Message.
 * 
 *  The `button` bundle sets `type="button"` so that rendering the trigger as a
 *  `button` element inside a `form` element toggles without also submitting the
 *  form. Setting it is harmless on the other elements a trigger might use, such
 *  as a `div` or a `span`, because the builder assigns a DOM property rather
 *  than an HTML attribute. Spread a later `h.Type` to override it.
 */
type DisclosureAttributes = Readonly<{
  animatePanel: (content: Html) => Html
  button: ReadonlyArray<Attribute<Message>>
  panel: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/ui/src/disclosure/index.ts#L49)

```
/**
 * Per-render view configuration for the stateless controlled view.
 *  Generic over `Message` (the message `onToggle` dispatches).
 * 
 *  - `isOpen`: the current open state, read straight from the parent Model.
 *    `aria-expanded`, the `data-open` marker, and `animatePanel` derive from
 *    it.
 *  - `onToggle`: dispatched with the new open state when the user clicks the
 *    button or presses Enter/Space. Handle it in the parent's `update` by
 *    storing the value.
 *  - `toView`: receives the DisclosureAttributes and lays out the
 *    disclosure.
 */
type ViewConfig = Readonly<{
  ariaLabel: string
  ariaLabelledBy: string
  id: string
  isDisabled: boolean
  isOpen: boolean
  onToggle: (isOpen: boolean) => Message
  toView: (attributes: DisclosureAttributes<Message>) => Html
}>
```
