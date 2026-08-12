---
url: https://foldkit.dev/api-reference/ui-select
title: "Ui/Select"
description: "API documentation for the Ui/Select module."
access_date: 2026-08-12T01:45:44.345Z
current_date: 2026-08-12T01:45:44.345Z
---

# Ui/Select

## Functions

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/ui/src/select/index.ts#L26)

```
/** Returns the description element id, derived from the select's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/ui/src/select/index.ts#L29)

```
/** Renders an accessible select by building ARIA attribute groups and delegating layout to the consumer's `toView` callback. */
<Message>(
  config: ViewConfig<Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### SelectAttributes

type

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/ui/src/select/index.ts#L7)

```
/** Attribute groups the select component provides to the consumer's `toView` callback. */
type SelectAttributes = Readonly<{
  description: ReadonlyArray<Attribute<Message>>
  label: ReadonlyArray<Attribute<Message>>
  select: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/ui/src/select/index.ts#L14)

```
/** Configuration for rendering a select with `view`. */
type ViewConfig = Readonly<{
  id: string
  isAutofocus: boolean
  isDisabled: boolean
  isInvalid: boolean
  name: string
  onChange: (value: string) => Message
  toView: (attributes: SelectAttributes<Message>) => Html
  value: string
}>
```
