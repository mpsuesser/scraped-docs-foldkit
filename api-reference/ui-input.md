---
url: https://foldkit.dev/api-reference/ui-input
title: "Ui/Input"
description: "API documentation for the Ui/Input module."
access_date: 2026-08-05T16:54:29.606Z
current_date: 2026-08-05T16:54:29.606Z
---

# Ui/Input

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/c9b3dd3e2e02eeb8a02bd48956c4d2611fd82016/packages/ui/src/input/index.ts#L28)

```
/** Generates the description element ID from the input's base ID. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/c9b3dd3e2e02eeb8a02bd48956c4d2611fd82016/packages/ui/src/input/index.ts#L31)

```
/** Renders an accessible input by building ARIA attribute groups and delegating layout to the consumer's `toView` callback. */
<Message>(
  config: ViewConfig<Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### InputAttributes

type

[source](https://github.com/foldkit/foldkit/blob/c9b3dd3e2e02eeb8a02bd48956c4d2611fd82016/packages/ui/src/input/index.ts#L7)

```
/** Attribute groups the input component provides to the consumer's `toView` callback. */
type InputAttributes = Readonly<{
  description: ReadonlyArray<Attribute<Message>>
  input: ReadonlyArray<Attribute<Message>>
  label: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/c9b3dd3e2e02eeb8a02bd48956c4d2611fd82016/packages/ui/src/input/index.ts#L14)

```
/** Configuration for rendering an input with `view`. */
type ViewConfig = Readonly<{
  id: string
  isAutofocus: boolean
  isDisabled: boolean
  isInvalid: boolean
  name: string
  onInput: (value: string) => Message
  placeholder: string
  toView: (attributes: InputAttributes<Message>) => Html
  type: string
  value: string
}>
```
