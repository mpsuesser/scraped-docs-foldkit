---
url: https://foldkit.dev/api-reference/ui-textarea
title: "Ui/Textarea"
description: "API documentation for the Ui/Textarea module."
access_date: 2026-09-12T18:49:33.387Z
current_date: 2026-09-12T18:49:33.387Z
---

# Ui/Textarea

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/textarea/index.ts#L35)

```
/** Returns the description element id, derived from the textarea's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/textarea/index.ts#L38)

```
/** Renders an accessible textarea by building ARIA attribute groups and delegating layout to the consumer's `toView` callback. */
<Message>(
  config: ViewConfig<Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### TextareaAttributes

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/textarea/index.ts#L12)

```
/** Attribute groups the textarea component provides to the consumer's `toView` callback. */
type TextareaAttributes = Readonly<{
  description: ReadonlyArray<Attribute<Message>>
  label: ReadonlyArray<Attribute<Message>>
  textarea: ReadonlyArray<TextareaAttribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/ui/src/textarea/index.ts#L19)

```
/** Configuration for rendering a textarea with `view`. */
type ViewConfig = Readonly<{
  hasDescription: boolean
  id: string
  isAutofocus: boolean
  isDisabled: boolean
  isInvalid: boolean
  isReadOnly: boolean
  name: string
  onInput: (value: string) => Message
  placeholder: string
  rows: number
  toView: (attributes: TextareaAttributes<Message>) => Html
  value: string
}>
```
