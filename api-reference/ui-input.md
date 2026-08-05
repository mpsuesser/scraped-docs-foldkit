---
url: https://foldkit.dev/api-reference/ui-input
title: "Ui/Input"
description: "API documentation for the Ui/Input module."
access_date: 2026-08-05T01:22:40.949Z
current_date: 2026-08-05T01:22:40.949Z
---

# Ui/Input

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/ui/src/input/index.ts#L28)

```
/** Generates the description element ID from the input's base ID. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/ui/src/input/index.ts#L31)

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

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/ui/src/input/index.ts#L7)

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

[source](https://github.com/foldkit/foldkit/blob/aa1c805330dd128d1e1b7fff3308820befb4325c/packages/ui/src/input/index.ts#L14)

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
