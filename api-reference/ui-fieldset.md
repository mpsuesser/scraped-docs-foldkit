---
url: https://foldkit.dev/api-reference/ui-fieldset
title: "Ui/Fieldset"
description: "API documentation for the Ui/Fieldset module."
access_date: 2026-09-12T22:55:23.086Z
current_date: 2026-09-12T22:55:23.086Z
---

# Ui/Fieldset

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/ui/src/fieldset/index.ts#L24)

```
/** Returns the description element id, derived from the fieldset's base id. */
(id: string): string
```

### legendId

function

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/ui/src/fieldset/index.ts#L21)

```
/** Returns the legend element id, derived from the fieldset's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/ui/src/fieldset/index.ts#L27)

```
/** Renders an accessible fieldset by building ARIA attribute groups and delegating layout to the consumer's `toView` callback. */
<Message>(
  config: ViewConfig<Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### FieldsetAttributes

type

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/ui/src/fieldset/index.ts#L6)

```
/** Attribute groups the fieldset component provides to the consumer's `toView` callback. */
type FieldsetAttributes = Readonly<{
  description: ReadonlyArray<Attribute<Message>>
  fieldset: ReadonlyArray<Attribute<Message>>
  legend: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/ui/src/fieldset/index.ts#L13)

```
/** Configuration for rendering a fieldset with `view`. */
type ViewConfig = Readonly<{
  hasDescription: boolean
  id: string
  isDisabled: boolean
  toView: (attributes: FieldsetAttributes<Message>) => Html
}>
```
