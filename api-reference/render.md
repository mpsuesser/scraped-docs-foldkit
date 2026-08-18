---
url: https://foldkit.dev/api-reference/render
title: "Render"
description: "API documentation for the Render module."
access_date: 2026-08-18T16:38:52.332Z
current_date: 2026-08-18T16:38:52.332Z
---

# Render

## Constants

### afterCommit

const

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/foldkit/src/render/render.ts#L62)

```
/**
 * Completes after the runtime's next render commits. The runtime batches
 * renders to `requestAnimationFrame`, so a Command, Subscription, or other
 * Effect that runs immediately after a dirtying Message would otherwise
 * query the DOM before the matching VDOM patch has applied. Yield this
 * before any DOM read or write whose target was just brought into existence
 * (or moved, or had its attributes changed) by the same Message.
 * 
 * The `Dom` helpers (`focus`, `clickElement`, `scrollIntoView`, etc.)
 * already gate themselves with this internally; reach for `afterCommit`
 * directly when building custom Commands or DOM-observing Subscriptions
 * that need the same guarantee.
 * 
 * When the runtime has a patch outstanding, this waits for that patch and
 * nothing else, so it holds even when the frame renders inside a View
 * Transition and the browser decides when the patch applies. With nothing
 * outstanding the DOM already matches the Model, and this yields one frame
 * to the browser.
 */
const afterCommit: Effect.Effect<void>
```

### afterPaint

const

[source](https://github.com/foldkit/foldkit/blob/6a4fac4ffd2cd09a3bdfab46eafc67e63984d771/packages/foldkit/src/render/render.ts#L84)

```
/**
 * Completes after the prior state has been painted to the screen. Waits for
 * the runtime's next commit and then one further animation frame, which
 * resumes once that commit's paint is visible. Use this for CSS transition
 * orchestration where the from-state must be displayed before the to-state
 * changes are applied, otherwise the browser collapses both states into a
 * single frame and the transition does not play.
 */
const afterPaint: Effect.Effect<void>
```
