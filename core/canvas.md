---
url: https://foldkit.dev/core/canvas
title: "Canvas"
description: "Declarative 2D rendering with a Schema-defined Shape AST and pointer events translated to canvas-local coordinates."
access_date: 2026-08-03T19:45:20.723Z
current_date: 2026-08-03T19:45:20.723Z
---

## Canvas

## Overview

The `Canvas` module is a declarative 2D rendering surface. `Canvas.view` produces a `<canvas>` VNode whose pixels are a pure function of a `shapes` prop: same shapes, same pixels. Because the view function is pure and the runtime re-paints on every patch, DevTools time-travel reproduces past frames exactly.

Reach for it for pixel art, board games, card games, 2D puzzlers, generative art, charts, and dataviz. The [canvas-art example](https://foldkit.dev/example-apps/canvas-art) demonstrates the API end-to-end with a click-to-spawn bouncing-balls scene.

## Shapes

A scene is a `ReadonlyArray<Shape>`. Each `Shape` is a tagged variant: `Rect`, `Circle`, `Path`, `Text`, or `Group`. `Group` wraps children in a 2D transform (`translate`, `rotate`, `scale`, `opacity`) and composes recursively. `Path` is built from a sequence of `PathInstruction` values: `MoveTo`, `LineTo`, `QuadTo`, `BezierTo`, `Close`.

## Animation and input

For continuous animation, pair `Canvas.view` with [Subscription.animationFrame](https://foldkit.dev/core/subscriptions#animation-frames): a helper that emits a Message every `requestAnimationFrame` tick with the inter-frame delta in milliseconds. update advances the Model in response, and `Canvas.view` re-renders whenever the Model changes (Foldkit batches renders to one per frame).

Pointer handlers are config args on `Canvas.view`: `onPointerDown`, `onPointerMove`, `onPointerUp`. Each receives a `Point` already translated into the canvas’s internal coordinate space (the `width` and `height` you passed), regardless of how the canvas is sized in CSS.

```
import { Canvas, Subscription } from 'foldkit'
import type { Html, HtmlBuilder } from 'foldkit/html'

const subscriptions = Subscription.make<Model, Message>()(_entry => ({
  frame: Subscription.animationFrame({
    isActive: model => model.isPlaying,
    toMessage: deltaTime => TickedFrame({ deltaTime }),
  }),
}))

const view = (model: Model, h: HtmlBuilder<Message>): Html =>
  Canvas.view(
    {
      width: 600,
      height: 400,
      shapes: [
        Canvas.Rect({ x: 0, y: 0, width: 600, height: 400, fill: '#0a0a0f' }),
        Canvas.Group({
          translate: { x: 300, y: 200 },
          rotate: model.angle,
          shapes: [
            Canvas.Circle({ x: 0, y: 0, radius: 50, fill: '#ff2d55' }),
            Canvas.Path({
              instructions: [
                Canvas.MoveTo({ x: -30, y: -30 }),
                Canvas.LineTo({ x: 30, y: -30 }),
                Canvas.LineTo({ x: 0, y: 30 }),
                Canvas.Close(),
              ],
              fill: '#ffcc00',
            }),
          ],
        }),
      ],
      onPointerDown: ({ x, y }) => ClickedCanvas({ x, y }),
    },
    h,
  )
```

## Full API surface

The [Canvas API reference](https://foldkit.dev/api-reference/canvas) lists every shape constructor, the `PathInstruction` variants, and `Canvas.view` with full signatures.
