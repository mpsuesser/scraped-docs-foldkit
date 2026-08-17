---
url: https://foldkit.dev/api-reference/ui-fieldset
title: "Ui/Fieldset"
description: "API documentation for the Ui/Fieldset module."
access_date: 2026-08-17T05:12:55.977Z
current_date: 2026-08-17T05:12:55.977Z
---

# Ui/Fieldset

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/ui/src/fieldset/index.ts#L23)

```
/** Returns the description element id, derived from the fieldset's base id. */
(id: string): string
```

### legendId

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/ui/src/fieldset/index.ts#L20)

```
/** Returns the legend element id, derived from the fieldset's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/ui/src/fieldset/index.ts#L26)

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

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/ui/src/fieldset/index.ts#L6)

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

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/ui/src/fieldset/index.ts#L13)

```
/** Configuration for rendering a fieldset with `view`. */
type ViewConfig = Readonly<{
  id: string
  isDisabled: boolean
  toView: (attributes: FieldsetAttributes<Message>) => Html
}>
```
