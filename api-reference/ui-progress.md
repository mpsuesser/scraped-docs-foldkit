---
url: https://foldkit.dev/api-reference/ui-progress
title: "Ui/Progress"
description: "API documentation for the Ui/Progress module."
access_date: 2026-10-01T05:11:45.759Z
current_date: 2026-10-01T05:11:45.759Z
---

# Ui/Progress

## Functions

### labelId

function

[source](https://github.com/foldkit/foldkit/blob/c9c641bf81797e88b2b20f2758b05455a0c5ab6b/packages/ui/src/progress/index.ts#L35)

```
/** Returns the label element id, derived from the progress base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/c9c641bf81797e88b2b20f2758b05455a0c5ab6b/packages/ui/src/progress/index.ts#L38)

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

[source](https://github.com/foldkit/foldkit/blob/c9c641bf81797e88b2b20f2758b05455a0c5ab6b/packages/ui/src/progress/index.ts#L15)

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

[source](https://github.com/foldkit/foldkit/blob/c9c641bf81797e88b2b20f2758b05455a0c5ab6b/packages/ui/src/progress/index.ts#L23)

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
