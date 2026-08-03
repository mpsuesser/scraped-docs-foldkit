---
url: https://foldkit.dev/react/coming-from-react
title: "Coming from React"
description: "Moving from React to a principled architecture? Foldkit replaces hooks, useEffect, and component state with The Elm Architecture: one Model, one update function, explicit effects. Built on Effect-TS."
access_date: 2026-08-03T18:13:14.939Z
current_date: 2026-08-03T18:13:14.939Z
---

# Coming from React

If you know React, you already have the instincts for building UIs. Foldkit channels those instincts through a different structure: one where every state change, every side effect, and every event is explicit and visible. The best way to feel the difference is to build the same thing in both.

Foldkit doesn’t compete with React on brevity, and it isn’t trying to. The first counter you see below is longer than its React counterpart, and the shape will feel unfamiliar: a separate Model, Message union, update function, and view, where React fits the same idea into a single component with a hook. That gap is the point. Foldkit names every piece React leaves implicit (state, events, side effects, subscriptions) so they stay legible as the app grows.

The trade is upfront verbosity for structural guarantees that compound. If you read the small example and think “that’s a lot of code for a counter,” you’re right. Keep reading: the next two sections add features that turn React into stale-closure debugging and leave Foldkit unchanged in shape.

## A Simple Counter

A counter in React:

```
import { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)

  const handleClickIncrement = () => {
    setCount(count => count + 1)
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClickIncrement}>Increment</button>
    </div>
  )
}
```

The same counter in Foldkit:

```
import { Match as M, Schema as S } from 'effect'
import { Command } from 'foldkit'
import type { Document, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'

// MODEL - Your entire application state

const Model = S.Number
type Model = typeof Model.Type

// MESSAGE - Events that can happen in your app

const ClickedIncrement = m('ClickedIncrement')

const Message = S.Union([ClickedIncrement])
type Message = typeof Message.Type

// UPDATE - How Messages change the Model

type UpdateReturn = readonly [Model, ReadonlyArray<Command.Command<Message>>]
const withUpdateReturn = M.withReturnType<UpdateReturn>()

const update = (model: Model, message: Message): UpdateReturn =>
  M.value(message).pipe(
    withUpdateReturn,
    M.tagsExhaustive({
      ClickedIncrement: () => [model + 1, []],
    }),
  )

// VIEW - A pure function from Model to a Document

const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: `Count: ${model}`,
  body: h.div(
    [],
    [
      h.p([], [`Count: ${model}`]),
      h.button([h.OnClick(ClickedIncrement())], ['Increment']),
    ],
  ),
})
```

More lines, same result. At this scale, Foldkit’s structure (Model, Message, update, view) looks like overhead. The benefits come with scale. Every piece earns its place as more complex behavior is introduced.

## Adding Auto-Count

New requirement: a play/pause button that auto-increments the counter every second.

React adds a ref to hold the interval ID and a `useEffect` to start and stop the interval:

```
import { useEffect, useRef, useState } from 'react'

const TICK_INTERVAL_MS = 1000

function Counter() {
  const intervalRef = useRef<number>()

  const [count, setCount] = useState(0)
  const [isAutoCounting, setIsPlaying] = useState(false)

  const handleClickIncrement = () => {
    setCount(count => count + 1)
  }

  const handleClickAutoCount = () => {
    setIsPlaying(isAutoCounting => !isAutoCounting)
  }

  useEffect(() => {
    if (isAutoCounting) {
      intervalRef.current = setInterval(() => {
        setCount(count => count + 1)
      }, TICK_INTERVAL_MS)
    }

    return () => clearInterval(intervalRef.current)
  }, [isAutoCounting])

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClickIncrement}>Increment</button>
      <button onClick={handleClickAutoCount}>
        {isAutoCounting ? 'Stop' : 'Auto-Count'}
      </button>
    </div>
  )
}
```

The interval state lives outside React’s state system (in a ref) because the effect needs to clear the previous interval before starting a new one. The cleanup function is critical: miss it and you leak intervals.

Foldkit adds a Subscription and a Message:

```
import { Duration, Effect, Match as M, Schema as S, Stream } from 'effect'
import { Command, Subscription } from 'foldkit'
import type { Document, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

const TICK_INTERVAL_MS = 1000

// MODEL

const Model = S.Struct({
  count: S.Number,
  isAutoCounting: S.Boolean,
})
type Model = typeof Model.Type

// MESSAGE

const ClickedIncrement = m('ClickedIncrement')
const ClickedToggleAutoCount = m('ClickedToggleAutoCount')
const Ticked = m('Ticked')

const Message = S.Union([ClickedIncrement, ClickedToggleAutoCount, Ticked])
type Message = typeof Message.Type

// SUBSCRIPTION

const subscriptions = Subscription.make<Model, Message>()(entry => ({
  tick: entry(
    { isAutoCounting: S.Boolean },
    {
      modelToDependencies: model => ({
        isAutoCounting: model.isAutoCounting,
      }),
      dependenciesToStream: ({ isAutoCounting }) =>
        Stream.when(
          Stream.tick(Duration.millis(TICK_INTERVAL_MS)).pipe(
            Stream.map(Ticked),
          ),
          Effect.sync(() => isAutoCounting),
        ),
    },
  ),
}))

// UPDATE

type UpdateReturn = readonly [Model, ReadonlyArray<Command.Command<Message>>]
const withUpdateReturn = M.withReturnType<UpdateReturn>()

const update = (model: Model, message: Message): UpdateReturn =>
  M.value(message).pipe(
    withUpdateReturn,
    M.tagsExhaustive({
      ClickedIncrement: () => [evo(model, { count: count => count + 1 }), []],
      ClickedToggleAutoCount: () => [
        evo(model, {
          isAutoCounting: isAutoCounting => !isAutoCounting,
        }),
        [],
      ],
      Ticked: () => [evo(model, { count: count => count + 1 }), []],
    }),
  )

// VIEW

const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: `Count: ${model.count}`,
  body: h.div(
    [],
    [
      h.p([], [`Count: ${model.count}`]),
      h.button([h.OnClick(ClickedIncrement())], ['Increment']),
      h.button(
        [h.OnClick(ClickedToggleAutoCount())],
        [model.isAutoCounting ? 'Stop' : 'Auto-Count'],
      ),
    ],
  ),
})
```

The Subscription emits `Ticked` every second while `isAutoCounting` is true. Foldkit manages the stream lifecycle: starts it when the dependency changes to true, tears it down when it changes to false. No refs, no manual cleanup.

## Adding a Step Size

One more feature: an input that controls how much each tick and manual click increments by.

This is where the React version gets subtle. The `setInterval` callback captures `step` at creation time. If you change the step while playing, the interval keeps using the old value: a stale closure. Nothing flags it at build time; the counter just increments by the wrong amount. React’s current fix is `useEffectEvent`, stable since 19.2: declare the tick as an Effect Event and every call reads current state:

```
import { useEffect, useEffectEvent, useRef, useState } from 'react'

const TICK_INTERVAL_MS = 1000

function Counter() {
  const intervalRef = useRef<number>()

  const [count, setCount] = useState(0)
  const [isAutoCounting, setIsPlaying] = useState(false)
  const [step, setStep] = useState(1)

  const handleClickIncrement = () => {
    setCount(count => count + step)
  }

  const handleClickAutoCount = () => {
    setIsPlaying(isAutoCounting => !isAutoCounting)
  }

  const onTick = useEffectEvent(() => {
    setCount(count => count + step)
  })

  useEffect(() => {
    if (isAutoCounting) {
      intervalRef.current = setInterval(() => onTick(), TICK_INTERVAL_MS)
    }

    return () => clearInterval(intervalRef.current)
  }, [isAutoCounting])

  return (
    <div>
      <p>Count: {count}</p>
      <label>
        Step:
        <input
          type="number"
          value={step}
          onChange={e => setStep(Number(e.target.value))}
        />
      </label>
      <button onClick={handleClickIncrement}>Increment</button>
      <button onClick={handleClickAutoCount}>
        {isAutoCounting ? 'Stop' : 'Auto-Count'}
      </button>
    </div>
  )
}
```

This is React’s best answer, and look at what it asks of you. First you meet the bug at runtime, because nothing flags a stale closure. Then you classify the read: `step` must be non-reactive here, so it belongs in an Effect Event. The nearest wrong answer is silent: for the naive version, `react-hooks/exhaustive-deps` suggests adding `step` to the dependency array instead, which restarts the interval on every keystroke and quietly resets its rhythm. And codebases older than React 19.2 solve this with a ref plus a sync effect to carry the current value past the closure, a pattern you will still meet everywhere. Most React developers have been burned by this.

In Foldkit, there is no stale closure:

```
import { Duration, Effect, Match as M, Schema as S, Stream } from 'effect'
import { Command, Subscription } from 'foldkit'
import type { Document, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

const TICK_INTERVAL_MS = 1000

// MODEL

const Model = S.Struct({
  count: S.Number,
  step: S.Number,
  isAutoCounting: S.Boolean,
})
type Model = typeof Model.Type

// MESSAGE

const ClickedIncrement = m('ClickedIncrement')
const ClickedToggleAutoCount = m('ClickedToggleAutoCount')
const ChangedStep = m('ChangedStep', { step: S.Number })
const Ticked = m('Ticked')

const Message = S.Union([
  ClickedIncrement,
  ClickedToggleAutoCount,
  ChangedStep,
  Ticked,
])
type Message = typeof Message.Type

// SUBSCRIPTION

const subscriptions = Subscription.make<Model, Message>()(entry => ({
  tick: entry(
    { isAutoCounting: S.Boolean },
    {
      modelToDependencies: model => ({
        isAutoCounting: model.isAutoCounting,
      }),
      dependenciesToStream: ({ isAutoCounting }) =>
        Stream.when(
          Stream.tick(Duration.millis(TICK_INTERVAL_MS)).pipe(
            Stream.map(Ticked),
          ),
          Effect.sync(() => isAutoCounting),
        ),
    },
  ),
}))

// UPDATE

type UpdateReturn = readonly [Model, ReadonlyArray<Command.Command<Message>>]
const withUpdateReturn = M.withReturnType<UpdateReturn>()

const update = (model: Model, message: Message): UpdateReturn =>
  M.value(message).pipe(
    withUpdateReturn,
    M.tagsExhaustive({
      ClickedIncrement: () => [
        evo(model, { count: count => count + model.step }),
        [],
      ],
      ClickedToggleAutoCount: () => [
        evo(model, {
          isAutoCounting: isAutoCounting => !isAutoCounting,
        }),
        [],
      ],
      ChangedStep: ({ step }) => [evo(model, { step: () => step }), []],
      Ticked: () => [evo(model, { count: count => count + model.step }), []],
    }),
  )

// VIEW

const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: `Count: ${model.count}`,
  body: h.div(
    [],
    [
      h.p([], [`Count: ${model.count}`]),
      h.label(
        [],
        [
          'Step: ',
          h.input([h.OnInput(value => ChangedStep({ step: Number(value) }))]),
        ],
      ),
      h.button([h.OnClick(ClickedIncrement())], ['Increment']),
      h.button(
        [h.OnClick(ClickedToggleAutoCount())],
        [model.isAutoCounting ? 'Stop' : 'Auto-Count'],
      ),
    ],
  ),
})
```

`model.step` is always current. The update function receives the latest Model every time a Message arrives. Both `ClickedIncrement` and `Ticked` use `model.step` and it just works. No refs, no Effect Events, and no deciding which reads are reactive.

Read the update function top to bottom. Every behavior in the app is right there. Each case is independent. They don’t interact through shared mutable state or overlapping effect dependencies. Adding a feature meant adding cases, not restructuring existing ones.

The pattern

In React, each new feature interacts with the effects, refs, and closures already there. In Foldkit, each new feature adds Messages, update cases, and possibly Commands or Subscriptions to structures that already exist, and the cases don’t interact through shared mutable state.

This structure also makes testing trivial. Your update function is pure. Pass a Model and a Message, assert on the returned Model. No rendering, no mocking `useEffect`, no wrapping in providers.

This is a toy example. Consider what happens at real scale: a multiplayer game with WebSocket streams, a mix of client and server state, handling keyboard events, animations, and reconnection logic. In React, every feature adds effects that interact with every other effect. In Foldkit, the architecture is the same as the counter: Messages come in, the update function decides what to do, Commands and Subscriptions handle the rest. The complexity of your domain grows, but the complexity of your architecture doesn’t.

## Translating React Concepts

Here’s how React patterns map to Foldkit:

React Ecosystem

Foldkit

`useState`

Model (single state tree)

`useReducer`

`update`

function

`useEffect`

(one-off)

Commands (returned from

`update`

)

`useRef`

+

`useEffect`

(DOM access)

Mount (

`OnMount`

with paired cleanup)

`useContext`

/ Redux / Zustand

Single Model (no prop drilling)

`useMemo`

/

`useCallback`

`createLazy`

/

`createKeyedLazy`

(memoize on data, not closure identity)

Custom hooks

Domain modules with pure functions

JSX

Plain functions from Model to HTML

Component props

Function parameters

Component state

Part of the single Model

Event handlers

Messages dispatched to

`update`

React Router / TanStack Router

Built-in typed routing

React Hook Form / Formik

Model + Messages +

`foldkit/fieldValidation`

Event streams (useEffect / RxJS)

Subscriptions (automatic lifecycle)

Headless UI / Radix UI

Foldkit UI (headless, typed components)

Error boundaries

Typed errors in Effects +

`crash.view`

If you know Redux...

The Model-View-Update pattern will feel familiar. Think of the Model as your Redux store, Messages as actions, and update as your reducer, but without action creators, selectors, or middleware.

## FAQ
