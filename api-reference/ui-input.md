---
url: https://foldkit.dev/api-reference/ui-input
title: "Ui/Input"
description: "API documentation for the Ui/Input module."
access_date: 2026-08-08T21:58:00.646Z
current_date: 2026-08-08T21:58:00.646Z
---

# Ui/Input

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/ui/src/input/index.ts#L29)

```
/** Returns the description element id, derived from the input's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/ui/src/input/index.ts#L32)

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

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/ui/src/input/index.ts#L7)

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

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/ui/src/input/index.ts#L14)

```
/** Configuration for rendering an input with `view`. */
type ViewConfig = Readonly<{
  id: string
  isAutofocus: boolean
  isDisabled: boolean
  isInvalid: boolean
  isReadOnly: boolean
  name: string
  onInput: (value: string) => Message
  placeholder: string
  toView: (attributes: InputAttributes<Message>) => Html
  type: string
  value: string
}>
```
