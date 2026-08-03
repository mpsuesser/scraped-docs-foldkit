---
url: https://foldkit.dev/example-apps/charting
title: "Charting"
description: "Live dashboard for public Foldkit telemetry from GitHub and npm. Demonstrates HTTP Commands, async state, an ECharts Mount adapter, and a Subscription that turns chart clicks back into Messages."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

[All Examples](https://foldkit.dev/example-apps)

# Charting

Live dashboard for public Foldkit telemetry from GitHub and npm. Demonstrates HTTP Commands, async state, an ECharts Mount adapter, and a Subscription that turns chart clicks back into Messages.

Charts

HTTP

Mount

Subscriptions

Third-Party Library

[Launch Playground](https://foldkit.dev/playground/charting)

[View source on GitHub](https://github.com/foldkit/foldkit/tree/main/examples/charting/src)

/

```
import type { EChartsType } from 'echarts/core'
import { Option } from 'effect'

const chartsByHostId = new Map<string, EChartsType>()

export const setChart = (hostId: string, chart: EChartsType): void => {
  chartsByHostId.set(hostId, chart)
}

export const getChart = (hostId: string): Option.Option<EChartsType> =>
  Option.fromNullishOr(chartsByHostId.get(hostId))

export const removeChart = (hostId: string): void => {
  const maybeChart = getChart(hostId)

  if (Option.isSome(maybeChart)) {
    maybeChart.value.dispose()
    chartsByHostId.delete(hostId)
  }
}
```
