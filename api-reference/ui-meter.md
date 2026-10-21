---
url: https://foldkit.dev/api-reference/ui-meter
title: "Ui/Meter"
description: "API documentation for the Ui/Meter module."
access_date: 2026-10-21T00:39:13.665Z
current_date: 2026-10-21T00:39:13.665Z
---

# Ui/Meter

## Functions

### labelId

function

[source](https://github.com/foldkit/foldkit/blob/7ee214268c602967937b80f6f20d1e831f2e8ab6/packages/ui/src/meter/index.ts#L37)

```
/** Returns the label element id, derived from the meter's base id. */
(id: string): string
```

### view

function

[source](https://github.com/foldkit/foldkit/blob/7ee214268c602967937b80f6f20d1e831f2e8ab6/packages/ui/src/meter/index.ts#L40)

```
/** Renders an accessible meter as a stateless controlled view. */
<Message>(
  config: ViewConfig<Message>,
  h: HtmlBuilder<Message>
): Html
```

## Types

### MeterAttributes

type

[source](https://github.com/foldkit/foldkit/blob/7ee214268c602967937b80f6f20d1e831f2e8ab6/packages/ui/src/meter/index.ts#L15)

```
/** Attribute groups provided to a Meter view. */
type MeterAttributes = Readonly<{
  fill: ReadonlyArray<Attribute<Message>>
  label: ReadonlyArray<Attribute<Message>>
  meter: ReadonlyArray<Attribute<Message>>
}>
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/7ee214268c602967937b80f6f20d1e831f2e8ab6/packages/ui/src/meter/index.ts#L22)

```
/** Configuration for rendering a Meter with view. */
type ViewConfig = Readonly<{
  ariaLabel: string
  ariaLabelledBy: string
  high: number
  id: string
  low: number
  max: number
  min: number
  optimum: number
  toView: (attributes: MeterAttributes<Message>) => Html
  value: number
  valueText: string | (value: number, max: number) => string
}>
```
