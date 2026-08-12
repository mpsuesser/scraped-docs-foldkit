---
url: https://foldkit.dev/api-reference/ui-textarea
title: "Ui/Textarea"
description: "API documentation for the Ui/Textarea module."
access_date: 2026-08-12T19:19:57.990Z
current_date: 2026-08-12T19:19:57.990Z
---

# Ui/Textarea

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/ui/src/textarea/index.ts#L29)

```
/** Returns the description element id, derived from the textarea's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/ui/src/textarea/index.ts#L32)

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

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/ui/src/textarea/index.ts#L7)

```
/** Attribute groups the textarea component provides to the consumer's `toView` callback. */
type TextareaAttributes = Readonly<{
  description: ReadonlyArray<Attribute<Message>>
  label: ReadonlyArray<Attribute<Message>>
  textarea: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/d5993a873d2a0aadaae590b207b001a71fc3fd1c/packages/ui/src/textarea/index.ts#L14)

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
