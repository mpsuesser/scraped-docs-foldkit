---
url: https://foldkit.dev/core/render
title: "Render"
description: "Primitives for synchronizing with the browser render cycle so DOM reads and CSS transitions land on the right frame."
access_date: 2026-08-03T19:45:20.723Z
current_date: 2026-08-03T19:45:20.723Z
---

# Render

## Overview

The `Render` module exposes two primitives for synchronizing with the browser's render cycle: `Render.afterCommit` resumes once the runtime has applied the latest VDOM patch to the DOM. `Render.afterPaint` resumes after the prior state has been displayed to the user. Both are Effects you yield inside your own [Commands](https://foldkit.dev/core/commands) or [Subscriptions](https://foldkit.dev/core/subscriptions).

The runtime batches renders to `requestAnimationFrame`. A Command runs on the microtask queue right after the dispatching Message, which means a synchronous DOM read or write inside that Command sees the tree from before the latest model was patched in. `Render.afterCommit` is how you wait for the matching patch to apply.

## When to reach for it

Reach for `Render.afterCommit` when you need to read or measure an element that was just brought into existence (or moved, or had attributes changed) by the same Message. Custom focus, custom scroll restoration, `IntersectionObserver` setup inside a Subscription, `getBoundingClientRect` for layout work. The [Dom helpers](https://foldkit.dev/api-reference/dom) already gate themselves with this internally, so reach for `Render.afterCommit` directly when building your own.

Reach for `Render.afterPaint` when you need the browser to actually display the prior state before you change to the next one, typically for CSS transition orchestration. A single `requestAnimationFrame` commits the DOM but the pixels have not been painted yet. A second one resumes after that paint is visible, so the from-state is on screen and the to-state can transition smoothly to it.

```
import { Effect } from 'effect'
import { Command, Render } from 'foldkit'

const MeasurePanel = Command.define('MeasurePanel', {
  messages: [MeasuredPanel],
  execute: Effect.gen(function* () {
    yield* Render.afterCommit
    const element = document.getElementById('panel')
    const width =
      element instanceof HTMLElement ? element.getBoundingClientRect().width : 0
    return MeasuredPanel({ width })
  }),
})

const StartTransition = Command.define('StartTransition', {
  messages: [StartedTransition],
  execute: Render.afterPaint.pipe(Effect.as(StartedTransition())),
})
```

## Full API surface

The [Render API reference](https://foldkit.dev/api-reference/render) lists every primitive with its signature and an inline example.
