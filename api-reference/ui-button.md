---
url: https://foldkit.dev/api-reference/ui-button
title: "Ui/Button"
description: "API documentation for the Ui/Button module."
access_date: 2026-11-19T17:46:07.734Z
current_date: 2026-11-19T17:46:07.734Z
---

# Ui/Button

## Functions

### view

function

[source](https://github.com/foldkit/foldkit/blob/144eeb4797f1b1d7a07fde8b112bae5d290fe90d/packages/ui/src/button/index.ts#L29)

```
/**
 * Renders an accessible button by building attribute groups and delegating
 * layout to the consumer's `toView` callback.
 * 
 * Takes the consumer's builder, which pins `Message` to the universe of the
 * frame the button is rendered in. `onClick` must come from that same
 * universe, so a Message the consumer's dispatcher cannot route is a compile
 * error here.
 */
<Message>(
  config: ViewConfig<Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### ButtonAttributes

type

[source](https://github.com/foldkit/foldkit/blob/144eeb4797f1b1d7a07fde8b112bae5d290fe90d/packages/ui/src/button/index.ts#L7)

```
/** Attribute groups the button component provides to the consumer's `toView` callback. */
type ButtonAttributes = Readonly<{
  button: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/144eeb4797f1b1d7a07fde8b112bae5d290fe90d/packages/ui/src/button/index.ts#L12)

```
/** Configuration for rendering a button with `view`. */
type ViewConfig = Readonly<{
  isAutofocus: boolean
  isDisabled: boolean
  onClick: Message
  toView: (attributes: ButtonAttributes<Message>) => Html
  type: "button" | "submit" | "reset"
}>
```
