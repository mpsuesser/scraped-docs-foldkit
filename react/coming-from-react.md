---
url: https://foldkit.dev/react/coming-from-react
title: "Coming from React"
description: "See how Foldkit replaces component-owned state and Effects with one Model, Messages, update, Commands, Subscriptions, and Submodels."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Coming from React

If you know React, you already have the instincts for building declarative interfaces. Foldkit puts those instincts inside a different structure. React organizes behavior around components and Hooks. Foldkit organizes it around one Model, Messages, update, and a view.

Foldkit does not compete with React on the brevity of a small component, and it is not trying to. Its first counter is longer because it names the state machine before the application needs much of one. That gap is deliberate. The examples below keep adding behavior to the same counter so you can see what the structure buys as effects and time enter the picture.

## A Simple Counter

Here is a counter in React:

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

The Foldkit version separates state, events, transitions, and rendering:

```
import { Match as M, Schema as S } from 'effect'
import { Command } from 'foldkit'
import type { Document, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

// MODEL - Your entire application state

const Model = S.Struct({
  count: S.Number,
})
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
      ClickedIncrement: () => [evo(model, { count: count => count + 1 }), []],
    }),
  )

// VIEW - A pure function from Model to a Document

const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: `Count: ${model.count}`,
  body: h.div(
    [],
    [
      h.p([], [`Count: ${model.count}`]),
      h.button([h.OnClick(ClickedIncrement())], ['Increment']),
    ],
  ),
})
```

For one number and one button, React is more compact. Foldkit’s structure starts paying for itself when the same state participates in timers, network requests, keyboard input, or several views. The rest of this page adds one of those concerns at a time.

## Adding Auto-Count

The next requirement is a play/pause button that increments the counter every second.

React uses an Effect to synchronize an interval with `isAutoCounting`:

```
import { useEffect, useState } from 'react'

const TICK_INTERVAL_MS = 1000

function Counter() {
  const [count, setCount] = useState(0)
  const [isAutoCounting, setIsPlaying] = useState(false)

  const handleClickIncrement = () => {
    setCount(count => count + 1)
  }

  const handleClickAutoCount = () => {
    setIsPlaying(isAutoCounting => !isAutoCounting)
  }

  useEffect(() => {
    if (!isAutoCounting) {
      return
    }

    const intervalId = setInterval(() => {
      setCount(count => count + 1)
    }, TICK_INTERVAL_MS)

    return () => clearInterval(intervalId)
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

The Effect starts the interval when auto-counting is active and returns the cleanup that stops it. React runs the cleanup before the Effect starts again and when the component unmounts. The functional state updater keeps the interval from depending on a captured `count`.

Foldkit adds a Subscription and a `Ticked` Message:

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

The Subscription emits `Ticked` while `isAutoCounting` is true. Foldkit scopes the Stream to that Model condition, so the runtime starts and stops it as the condition changes. The interval does not live in the view, and its ticks enter the application through the same update function as button clicks.

## Adding a Step Size

Now the user can choose how much each manual click and timer tick adds.

A naive React interval that reads `step` from its original closure keeps using that old value. Adding `step` to the Effect dependencies gives the interval the latest value, but also restarts the interval whenever the input changes. If the interval should keep its rhythm, React 19.2’s `useEffectEvent` lets the tick read the latest committed `step` without making `step` a synchronization dependency:

```
import { useEffect, useEffectEvent, useState } from 'react'

const TICK_INTERVAL_MS = 1000

function Counter() {
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
    if (!isAutoCounting) {
      return
    }

    const intervalId = setInterval(() => onTick(), TICK_INTERVAL_MS)

    return () => clearInterval(intervalId)
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

The distinction is meaningful in React. `isAutoCounting` controls whether the external interval exists, so it is an Effect dependency. `step` is data read when the interval fires, so the Effect Event reads its current value without restarting the interval. The Hooks linter enforces where an Effect Event may be called and keeps it out of the dependency array.

The Foldkit version adds `step` to the Model and handles `ChangedStep`:

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

Each `Ticked` Message is handled with the current Model, so `model.step` is current when update calculates the next count. The Subscription still depends only on whether auto-counting is active. There is no closure decision to make and no second mechanism for reading the latest value.

The architectural difference

React synchronizes an external resource from component state, so the Effect must distinguish values that control the resource from values read when it emits. Foldkit’s Subscription controls the resource from a Model condition and emits Messages. Update reads the current Model when each Message arrives.

`useEffectEvent` is a good answer to the React problem. Foldkit does not create that problem. The timer emits a fact, and update decides what that fact means using the current Model.

The Foldkit example can be tested below the view by passing Models and Messages directly to update. A view-level Scene test can exercise the same flow through the buttons and input. Neither test needs to wait for a real interval because `Ticked` is already a value the test can dispatch.

## Translating React Concepts

The mappings below are starting points, not one-to-one replacements:

React ecosystem

Foldkit

`useState`

/ component state

Fields in the Model

`useReducer`

The update function and Message union

Event-driven side effect

A Command returned from update

External event source tied to state

A Subscription gated by Model dependencies

DOM work tied to an element

`Mount.define`

or

`Mount.defineStream`

Stateful resource shared with Commands

ManagedResource

Context used for application state

The Model

Context used for services

Effect services and Layers

`useMemo`

/

`useCallback`

Often no equivalent;

`createLazy`

and

`createKeyedLazy`

skip expensive view work when needed

Custom Hook

A domain module, pure helper, lifecycle primitive, or combination of them

JSX

Typed HTML builder functions

Component props

Function parameters

Event handler

A Message value or a function that constructs one

React Router / TanStack Router

Built-in typed routing

Next.js SSG / SSR

[Server rendering](https://foldkit.dev/core/server-rendering)

, at build time or per request

React Hook Form / Formik

Model, Messages, and

[field validation](https://foldkit.dev/core/field-validation)

Headless UI / Radix UI

[Foldkit UI](https://foldkit.dev/ui/overview)

Error Boundary for an unexpected rendering crash

[Crash view](https://foldkit.dev/core/crash-view)

; expected Effect failures return as Messages and become explicit Model state

If you know Redux

The Model-View-Update pattern will feel familiar. The Model resembles the store, Messages resemble actions, and update resembles a reducer. Foldkit’s update also returns Commands, and its Message union is exhaustively matched.

## FAQ
