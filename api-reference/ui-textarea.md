---
url: https://foldkit.dev/api-reference/ui-textarea
title: "Ui/Textarea"
description: "API documentation for the Ui/Textarea module."
access_date: 2026-09-07T07:29:31.695Z
current_date: 2026-09-07T07:29:31.695Z
---

# Ui/Textarea

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/textarea/index.ts#L34)

```
/** Returns the description element id, derived from the textarea's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/textarea/index.ts#L37)

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

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/textarea/index.ts#L12)

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

[source](https://github.com/foldkit/foldkit/blob/fa58326c9959d93d0edae71522711cb8c11546c1/packages/ui/src/textarea/index.ts#L19)

```
/** Configuration for rendering a textarea with `view`. */
type ViewConfig = Readonly<{
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
