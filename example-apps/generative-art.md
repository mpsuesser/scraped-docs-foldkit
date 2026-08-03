---
url: https://foldkit.dev/example-apps/generative-art
title: "Generative Art"
description: "Move the mouse to stir a Perlin-noise flow field, click to bloom prismatic particle bursts. Demonstrates Canvas.view with hundreds of evolving Path strokes per frame, Effect Random for spawning, and tunable simulation knobs wired through Messages."
access_date: 2026-08-03T18:23:29.850Z
current_date: 2026-08-03T18:23:29.850Z
---

[All Examples](https://foldkit.dev/example-apps)

# Generative Art

Move the mouse to stir a Perlin-noise flow field, click to bloom prismatic particle bursts. Demonstrates Canvas.view with hundreds of evolving Path strokes per frame, Effect Random for spawning, and tunable simulation knobs wired through Messages.

Canvas

Animation

Subscriptions

Generative

[Launch Playground](https://foldkit.dev/playground/generative-art)

[View source on GitHub](https://github.com/foldkit/foldkit/tree/main/examples/generative-art/src)

/

```
import { Array, Option } from 'effect'
import { Runtime } from 'foldkit'

import { Slider } from '@foldkit/ui'

import { GenerateAmbientParticle } from './command'
import {
  FLOW_STRENGTH_MAX,
  FLOW_STRENGTH_MIN,
  FLOW_STRENGTH_STEP,
  INITIAL_FLOW_STRENGTH,
  INITIAL_NOISE_SCALE,
  INITIAL_PARTICLE_COUNT,
  NOISE_SCALE_MAX_DIVISOR,
  NOISE_SCALE_MIN_DIVISOR,
  NOISE_SCALE_STEP,
} from './constant'
import { Message } from './message'
import { Model } from './model'
import { subscriptions } from './subscription'
import { update } from './update'
import { view } from './view'

export const init: Runtime.ApplicationInit<Model, Message> = () => [
  {
    particles: [],
    nextId: 0,
    elapsedSeconds: 0,
    maybeMousePosition: Option.none(),
    isRunning: true,
    flowStrength: Slider.snapAndClamp(
      INITIAL_FLOW_STRENGTH,
      FLOW_STRENGTH_MIN,
      FLOW_STRENGTH_MAX,
      FLOW_STRENGTH_STEP,
    ),
    flowStrengthSlider: Slider.init({
      id: 'flow-strength-slider',
      min: FLOW_STRENGTH_MIN,
      max: FLOW_STRENGTH_MAX,
      step: FLOW_STRENGTH_STEP,
    }),
    noiseScale: Slider.snapAndClamp(
      INITIAL_NOISE_SCALE,
      NOISE_SCALE_MIN_DIVISOR,
      NOISE_SCALE_MAX_DIVISOR,
      NOISE_SCALE_STEP,
    ),
    noiseScaleSlider: Slider.init({
      id: 'noise-scale-slider',
      min: NOISE_SCALE_MIN_DIVISOR,
      max: NOISE_SCALE_MAX_DIVISOR,
      step: NOISE_SCALE_STEP,
    }),
  },
  Array.makeBy(INITIAL_PARTICLE_COUNT, () => GenerateAmbientParticle()),
]

export { Message, Model, subscriptions, update, view }
```
