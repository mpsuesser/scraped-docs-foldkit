---
url: https://foldkit.dev/example-apps/charting
title: "Charting"
description: "A live dashboard for public Foldkit telemetry from GitHub and npm. Demonstrates HTTP Commands, asynchronous state, an ECharts Mount adapter, and a Subscription that turns chart clicks into Messages."
access_date: 2026-08-31T07:29:25.100Z
current_date: 2026-08-31T07:29:25.100Z
---

[All Examples](https://foldkit.dev/example-apps)

# Charting

A live dashboard for public Foldkit telemetry from GitHub and npm. Demonstrates HTTP Commands, asynchronous state, an ECharts Mount adapter, and a Subscription that turns chart clicks into Messages.

Charts

HTTP

Mount

Subscriptions

Third-Party Library

[Launch Playground](https://foldkit.dev/playground/charting)

[View source on GitHub](https://github.com/foldkit/foldkit/tree/main/examples/charting/src)

/

```
import { defineConfig } from 'vite'

import { foldkit } from '@foldkit/vite-plugin'
import tailwindcss from '@tailwindcss/vite'

import { foldkitAliases } from '../vite.aliases'

export default defineConfig({
  plugins: [tailwindcss(), foldkit({ devToolsMcpPort: 9988 })],
  resolve: {
    alias: foldkitAliases(__dirname),
  },
  server: {
    fs: {
      allow: ['../../'],
    },
  },
})
```
