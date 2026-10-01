---
url: https://foldkit.dev/api-reference/ui-progress
title: "Ui/Progress"
description: "API documentation for the Ui/Progress module."
access_date: 2026-10-01T05:29:00.002Z
current_date: 2026-10-01T05:29:00.002Z
---

# Ui/Progress

## Functions

### labelId

function

[source](https://github.com/foldkit/foldkit/blob/24f1c43eaaf74a83b6e338d72ba98422b7ac8ec4/packages/ui/src/progress/index.ts#L35)

```
/** Returns the label element id, derived from the progress base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/24f1c43eaaf74a83b6e338d72ba98422b7ac8ec4/packages/ui/src/progress/index.ts#L38)

```
/** Renders an accessible progress indicator as a stateless controlled view. */
<Message>(
  config: ViewConfig<Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### ProgressAttributes

type

[source](https://github.com/foldkit/foldkit/blob/24f1c43eaaf74a83b6e338d72ba98422b7ac8ec4/packages/ui/src/progress/index.ts#L15)

```
/** Attribute groups provided to a Progress view. */
type ProgressAttributes = Readonly<{
  indicator: ReadonlyArray<Attribute<Message>>
  label: ReadonlyArray<Attribute<Message>>
  progress: ReadonlyArray<Attribute<Message>>
  track: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/24f1c43eaaf74a83b6e338d72ba98422b7ac8ec4/packages/ui/src/progress/index.ts#L23)

```
/** Configuration for rendering Progress with view. */
type ViewConfig = Readonly<{
  ariaLabel: string
  ariaLabelledBy: string
  id: string
  max: number
  min: number
  toView: (attributes: ProgressAttributes<Message>) => Html
  value: number
  valueText: string | (value: number, max: number) => string
}>
```
