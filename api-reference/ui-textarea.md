---
url: https://foldkit.dev/api-reference/ui-textarea
title: "Ui/Textarea"
description: "API documentation for the Ui/Textarea module."
access_date: 2026-08-05T16:54:29.606Z
current_date: 2026-08-05T16:54:29.606Z
---

# Ui/Textarea

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/c9b3dd3e2e02eeb8a02bd48956c4d2611fd82016/packages/ui/src/textarea/index.ts#L28)

```
/** Generates the description element ID from the textarea's base ID. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/c9b3dd3e2e02eeb8a02bd48956c4d2611fd82016/packages/ui/src/textarea/index.ts#L31)

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

[source](https://github.com/foldkit/foldkit/blob/c9b3dd3e2e02eeb8a02bd48956c4d2611fd82016/packages/ui/src/textarea/index.ts#L7)

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

[source](https://github.com/foldkit/foldkit/blob/c9b3dd3e2e02eeb8a02bd48956c4d2611fd82016/packages/ui/src/textarea/index.ts#L14)

```
/** Configuration for rendering a textarea with `view`. */
type ViewConfig = Readonly<{
  id: string
  isAutofocus: boolean
  isDisabled: boolean
  isInvalid: boolean
  name: string
  onInput: (value: string) => Message
  placeholder: string
  rows: number
  toView: (attributes: TextareaAttributes<Message>) => Html
  value: string
}>
```
