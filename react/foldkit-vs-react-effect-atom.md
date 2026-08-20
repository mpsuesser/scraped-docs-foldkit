---
url: https://foldkit.dev/react/foldkit-vs-react-effect-atom
title: "Foldkit vs React + Effect Atom"
description: "Two Effect-native architectures: Effect Atom distributes state across reactive cells inside React, while Foldkit builds the application around one Model and update function."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Foldkit vs React + Effect Atom

## Overview

This page is for people who have already chosen Effect. In Effect 4, [Effect Atom](https://github.com/Effect-TS/effect/tree/main/packages/atom) provides reactive state primitives through `effect/unstable/reactivity`, with view bindings such as `@effect/atom-react`, `@effect/atom-solid`, and `@effect/atom-vue`. The host framework still owns rendering and components.

Foldkit owns the application runtime and view layer. It renders through a virtual DOM built on [Snabbdom](https://github.com/snabbdom/snabbdom), and it includes routing, UI components, DevTools, and Story and Scene testing. Its architecture has one Model, a Message union, and an update function. Side effects return to the runtime as Commands and other lifecycle primitives.

The choice is therefore larger than where to store state. React with Effect Atom is a React application with a reactive Effect state layer. A Foldkit application uses a different runtime, view model, and testing model. They are different paradigms that happen to share Effect.

Related page

This page assumes you are already using Effect. If you are coming from plain React, [Foldkit vs React](https://foldkit.dev/react/foldkit-vs-react-side-by-side) covers the broader architectural differences.

## What They Share

Both approaches can use Effect values, typed errors, Layers, structured concurrency, and Schema validation at application boundaries. In Effect Atom, an Effect can run inside an async atom or a function atom. In Foldkit, a Command wraps an Effect and returns its result as a Message.

Above that Effect foundation, their responsibilities differ. Effect Atom provides state and reactivity to a host view framework. Foldkit owns the runtime, rendering, routing, lifecycle primitives, and testing tools.

## Many Atoms vs One Model

An atom is a reactive container for a value. You can create state with `Atom.make`, derive an atom from other atoms, read it with `useAtomValue`, and write it with `useAtomSet`. The registry tracks dependencies and notifies the components that read a changed atom.

State is distributed by design, and that is the point. A feature can own its atoms, and a component can update any writable atom it imports. The cost of that locality is that no single value represents the state of the application and no single type lists every way it can change.

Foldkit centralizes state in the [Model](https://foldkit.dev/core/model). The [Message](https://foldkit.dev/core/messages) union lists the facts the application handles, and [update](https://foldkit.dev/core/update) defines how those facts change the Model. Those are framework constraints, not conventions a team maintains by discipline. [Submodels](https://foldkit.dev/core/submodel) split a large application into smaller state machines while preserving the same parent-to-child Message flow.

## How State Changes

The state models become concrete when a user adds or edits a todo.

### Effect Atom: setters at the call site

A React component obtains a setter with `useAtomSet`. The setter can receive an updater closure:

```
import { Atom } from 'effect/unstable/reactivity'

import { useAtomSet, useAtomValue } from '@effect/atom-react'

type Filter = 'All' | 'Active' | 'Done'

// State is a set of independent reactive cells.
const filterAtom = Atom.make<Filter>('All').pipe(Atom.keepAlive)
const todosAtom = Atom.make<ReadonlyArray<Todo>>([]).pipe(Atom.keepAlive)

// Any component can write any atom, with an inline updater closure.
const AddTodoButton = () => {
  const setTodos = useAtomSet(todosAtom)
  return (
    <button onClick={() => setTodos(todos => [...todos, emptyTodo()])}>
      Add
    </button>
  )
}

const ClearDoneButton = () => {
  const setTodos = useAtomSet(todosAtom)
  return (
    <button onClick={() => setTodos(todos => todos.filter(todo => !todo.done))}>
      Clear done
    </button>
  )
}

const FilterTabs = () => {
  const filter = useAtomValue(filterAtom)
  const setFilter = useAtomSet(filterAtom)
  // ... each transition is an anonymous closure, scattered across components
}
```

In this example, the ways `todosAtom` changes live at its setter call sites. An atom application can instead expose named write functions or writable derived atoms. That centralization is an application convention. Foldkit requires every Model transition to pass through update.

### Foldkit: one Message union, one update

The Foldkit version represents the same actions as Messages:

```
import { Array, Match as M, Schema as S } from 'effect'
import { Command } from 'foldkit'
import { m } from 'foldkit/message'

// MODEL

const Filter = S.Literals(['All', 'Active', 'Done'])

export const Model = S.Struct({
  todos: S.Array(Todo),
  filter: Filter,
})
type Model = typeof Model.Type

// MESSAGE

const AddedTodo = m('AddedTodo')
const ClearedDoneTodos = m('ClearedDoneTodos')
const SelectedFilter = m('SelectedFilter', { filter: Filter })

const Message = S.Union([AddedTodo, ClearedDoneTodos, SelectedFilter])
type Message = typeof Message.Type

// UPDATE

export const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    withUpdateReturn,
    M.tagsExhaustive({
      AddedTodo: () => [evo(model, { todos: Array.append(emptyTodo()) }), []],
      ClearedDoneTodos: () => [
        evo(model, { todos: Array.filter(todo => !todo.done) }),
        [],
      ],
      SelectedFilter: ({ filter }) => [
        evo(model, { filter: () => filter }),
        [],
      ],
    }),
  )
```

`AddedTodo`, `ClearedDoneTodos`, and `SelectedFilter` appear in DevTools and in Story or Scene tests. “How can the todo list change?” is answered by one Message union and one update function. `M.tagsExhaustive` reports every place that must handle a newly added Message variant.

## Async State

Both provide a value type for in-progress Effect results, but the value lives in a different place.

### Effect Atom: AsyncResult

`Atom.make` accepts an Effect directly. When the Effect needs Layer-provided services, `Atom.runtime` creates an atom runtime and `runtime.atom` runs the Effect with that context. The resulting atom contains an `AsyncResult`: `Initial`, `Success`, or `Failure`, with a `waiting` flag for refreshes. `AsyncResult.builder` renders those cases.

```
import { Cause, Effect } from 'effect'
import { AsyncResult, Atom } from 'effect/unstable/reactivity'

import { useAtomValue } from '@effect/atom-react'

const runtime = Atom.runtime(Api.Default)

// An async atom evaluates an Effect and exposes an AsyncResult.
const userAtom = runtime.atom(
  Effect.gen(function* () {
    const api = yield* Api
    return yield* api.getUser()
  }),
)

const UserCard = () => {
  const user = useAtomValue(userAtom)

  return AsyncResult.builder(user)
    .onInitial(() => <Spinner />)
    .onFailure(cause => <ErrorBanner message={Cause.pretty(cause)} />)
    .onSuccess(user => <Profile user={user} />)
    .render()
}
```

The Effect runs when the registry first evaluates the atom, and the registry stores the result and tracks dependencies. `Atom.family` creates keyed atoms. `Atom.swr` adds stale-time and revalidation behavior, while `AtomHttpApi` and `AtomRpc` integrate Effect clients. React bindings also provide Suspense hooks.

### Foldkit: remote state in the Model

Foldkit stores remote state in the Model. [AsyncData](https://foldkit.dev/core/async-data) represents six states: `Idle`, `Loading`, `Refreshing`, `Failure`, `Stale`, and `Success`. A Command performs the request, and its result returns through update as a Message.

```
import { Effect, Match as M, Schema as S } from 'effect'
import { AsyncData, Command } from 'foldkit'
import { m } from 'foldkit/message'

import { Api } from './api'

// MODEL

// Remote state is a value in the Model. AsyncData is the shipped six-state
// union, so there is no hand-rolled loading/failure/stale union to maintain.
const UserAsyncData = AsyncData.Schema(User, ApiError)

export const Model = S.Struct({
  user: UserAsyncData.schema,
})
type Model = typeof Model.Type

// MESSAGE

const ClickedLoadUser = m('ClickedLoadUser')
const SucceededLoadUser = m('SucceededLoadUser', { user: User })
const FailedLoadUser = m('FailedLoadUser', { error: ApiError })

const Message = S.Union([ClickedLoadUser, SucceededLoadUser, FailedLoadUser])
type Message = typeof Message.Type

// COMMAND

// Api is an Effect service; Api.Default is its layer.
const FetchUser = Command.define('FetchUser', {
  messages: [SucceededLoadUser, FailedLoadUser],
  execute: Effect.gen(function* () {
    const api = yield* Api
    const user = yield* api.getUser()
    return SucceededLoadUser({ user })
  }).pipe(
    Effect.catch(error => Effect.succeed(FailedLoadUser({ error }))),
    Effect.provide(Api.Default),
  ),
})

// UPDATE

export const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    withUpdateReturn,
    M.tagsExhaustive({
      ClickedLoadUser: () => [
        evo(model, { user: () => UserAsyncData.Loading() }),
        [FetchUser()],
      ],
      SucceededLoadUser: ({ user }) => [
        evo(model, { user: () => UserAsyncData.Success({ data: user }) }),
        [],
      ],
      FailedLoadUser: ({ error }) => [
        evo(model, { user: () => UserAsyncData.Failure({ error }) }),
        [],
      ],
    }),
  )
```

`AsyncData` includes stale-while-revalidate and keep-stale-on-failure states. It does not provide a fetching registry or choose refresh policy. The application models a cache in the Model and decides when to run each Command. Remote data then shares the same Message timeline and tests as the rest of the Model.

Effect Atom gives you more data-fetching machinery out of the box. Foldkit gives remote state no separate architectural lane. That distinction matters when remote data starts interacting with the rest of the application.

## Side Effects and Lifecycle

Both can express side effects as Effect values. They differ in how an effect is connected to application state and lifetime.

### Effect Atom: effects live inside atoms

`runtime.fn` creates a callable atom for a mutation. Its `reactivityKeys` can refresh atoms that subscribe to matching keys. An atom can also acquire a listener and register its cleanup with `addFinalizer`; `useAtomMount` keeps that atom mounted for the component’s lifetime.

```
import { Effect } from 'effect'
import { Atom } from 'effect/unstable/reactivity'

import { useAtomMount, useAtomSet } from '@effect/atom-react'

// A mutation is a function atom. Reactivity keys invalidate dependents.
const createTodoAtom = runtime.fn(
  Effect.fnUntraced(function* (text: string) {
    const api = yield* Api
    yield* api.createTodo(text)
  }),
  { reactivityKeys: ['todos'] },
)

// A global listener is an atom that wires addEventListener in its body,
// then tears it down with a finalizer.
const mouseUpAtom = Atom.make(get => {
  const onUp = () => get.setSelf(false)
  window.addEventListener('mouseup', onUp)
  get.addFinalizer(() => window.removeEventListener('mouseup', onUp))
  return false
})

const Canvas = () => {
  useAtomMount(mouseUpAtom) // keep the listener alive while this component is mounted
  const createTodo = useAtomSet(createTodoAtom)
  // ...
}
```

Effects remain colocated with the atoms that perform them. Dependencies between a mutation and refreshed atoms can be declared through reactivity keys, which are runtime values rather than a TypeScript union checked for exhaustiveness.

### Foldkit: Commands and Subscriptions

Foldkit selects a lifecycle primitive based on what causes the work. A [Command](https://foldkit.dev/core/commands) runs after a Message. A [Subscription](https://foldkit.dev/core/subscriptions) runs while a Model condition holds. A [Mount](https://foldkit.dev/core/mount) follows an element’s lifetime, and a [ManagedResource](https://foldkit.dev/core/managed-resources) follows Model state while exposing a stateful handle to Commands.

```
import { Effect, Schema as S, Stream } from 'effect'
import { Command, Subscription } from 'foldkit'

import { Api } from './api'

// A side effect is a Command returned from update. It has a name, shows up
// in DevTools next to the Message that produced it, and is assertable in
// tests. Api is an Effect service; Api.Default is its layer.
const CreateTodo = Command.define('CreateTodo', {
  args: { text: S.String },
  messages: [SucceededCreateTodo, FailedCreateTodo],
  execute: ({ text }) =>
    Effect.gen(function* () {
      const api = yield* Api
      yield* api.createTodo(text)
      return SucceededCreateTodo()
    }).pipe(
      Effect.provide(Api.Default),
      Effect.catch(() => Effect.succeed(FailedCreateTodo())),
    ),
})

// Here the global listener becomes a Subscription: an external event source
// bound to a slice of the Model. The runtime subscribes and unsubscribes as
// model.isDrawing changes. No addEventListener, no cleanup, no stale closure.
export const subscriptions = Subscription.make<Model, Message>()(entry => ({
  mouseRelease: entry(
    { isDrawing: S.Boolean },
    {
      modelToDependencies: model => ({ isDrawing: model.isDrawing }),
      dependenciesToStream: ({ isDrawing }) =>
        Stream.when(
          Subscription.fromEvent<MouseEvent, Message>({
            target: document,
            type: 'mouseup',
            toMessage: () => ReleasedMouse(),
          }),
          Effect.sync(() => isDrawing),
        ),
    },
  ),
}))
```

The difference is locational. Effect Atom colocates effects with atoms, and any atom may run one. Foldkit assigns effects to a small set of lifecycle primitives based on what causes them. The `mouseRelease` Subscription starts and stops from Model state, then emits Messages so the resulting change still passes through update.

## The View Layer Is Still React

Effect Atom changes the state layer, not React’s rendering rules. Components still use hooks, closures, and React’s memoization tools.

### Effect Atom plus React

This version stores the todos in one array atom. It uses `memo` and `useCallback` to avoid rendering an unchanged row when the array changes:

```
import { memo, useCallback } from 'react'

import { useAtomSet } from '@effect/atom-react'

// The view layer is still React: memo to skip re-renders, useCallback to keep
// the handler reference stable, a dependency array you have to get right.
const TodoItem = memo(({ todo }: { todo: Todo }) => {
  const setTodos = useAtomSet(todosAtom)

  const toggle = useCallback(
    () =>
      setTodos(todos =>
        todos.map(candidate =>
          candidate.id === todo.id
            ? { ...candidate, done: !candidate.done }
            : candidate,
        ),
      ),
    [setTodos, todo.id],
  )

  return (
    <li>
      <input type="checkbox" checked={todo.done} onChange={toggle} />
      {todo.text}
    </li>
  )
})
```

Those optimizations are not required for correctness. A per-item atom could also give each row a narrower subscription. Either design still follows React’s hook and closure rules, and React Compiler can automate some memoization when the component satisfies its constraints.

### Foldkit

The Foldkit item is a function that returns virtual DOM data. Its event handler dispatches a Message value:

```
import type { Html, HtmlBuilder } from 'foldkit/html'

// The view is a plain function returning data. No memo, no useCallback, no
// dependency array. The event is a Message value, not a closure, so there is
// nothing to stabilize at the boundary.
const todoItem = (todo: Todo, h: HtmlBuilder<Message>): Html =>
  h.li(
    [],
    [
      h.input([
        h.Type('checkbox'),
        h.Checked(todo.done),
        h.OnClick(ClickedTodo({ id: todo.id })),
      ]),
      todo.text,
    ],
  )
```

There is no component Hook state, dependency array, or callback identity to stabilize. When a view subtree is expensive, [view memoization](https://foldkit.dev/core/view-memoization) skips it based on Model-derived inputs.

## One Timeline vs Many Cells

Every Foldkit state change is a Message processed by update. Foldkit DevTools records those Messages and the resulting Models, so it can replay the application timeline. This site runs on Foldkit; the DEV button in the bottom-right corner opens its DevTools.

Effect Atom’s registry holds the current value of many cells and updates their dependents. It does not define one app-wide union of named events, so there is no equivalent built-in Message timeline to replay. An application can add its own event model or logging when that history is useful.

## Testing

Foldkit’s update function is pure: given a Model and a Message, it returns the next Model and Commands. That shape supports two [testing](https://foldkit.dev/testing) tools.

[Story](https://foldkit.dev/testing/story) sends Messages through update, inspects the Model, and resolves Commands by supplying their result Messages. The test can assert that a Command was returned without executing its Effect.

```
import { AsyncData } from 'foldkit'
import { Command, given, message, model, story } from 'foldkit/story'
import { expect, test } from 'vitest'

test('loading a user: the Command fires, resolves, the Model lands on Success', () => {
  story(
    update,
    given({ user: AsyncData.Idle() }),
    message(ClickedLoadUser()),
    Command.expectExact(FetchUser),
    Command.resolve(FetchUser, SucceededLoadUser({ user: ada })),
    model(model => {
      expect(model.user).toStrictEqual(AsyncData.Success({ data: ada }))
    }),
  )
})
```

[Scene](https://foldkit.dev/testing/scene) renders the view, finds elements by accessible role or text, dispatches events through update, and resolves Commands inline.

```
import {
  Command,
  click,
  expect,
  given,
  inside,
  role,
  scene,
  text,
} from 'foldkit/scene'
import { test } from 'vitest'

test('click load, resolve the fetch, see the profile', () => {
  scene(
    { update, view },
    given(model),
    click(role('button', { name: 'Load user' })),
    expect(text('Loading…')).toExist(),
    Command.expectExact(FetchUser),
    Command.resolve(FetchUser, SucceededLoadUser({ user: ada })),
    inside(role('article'), expect(text('Ada Lovelace')).toExist()),
  )
})
```

An Effect Atom application uses the testing tools of its host framework. In React, a user-facing test commonly renders a component with React Testing Library and jsdom, then waits for the atom’s Effect and React render to finish:

```
import { expect, test, vi } from 'vitest'

import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

test('loading a user renders the profile', async () => {
  // userAtom runs an Effect that fetches the user, so the test stubs the
  // network boundary and renders the component tree in jsdom.
  vi.spyOn(globalThis, 'fetch').mockResolvedValue(
    Response.json({ name: 'Ada Lovelace' }),
  )

  render(<UserCard />)

  await userEvent.click(screen.getByRole('button', { name: /load user/i }))
  expect(screen.getByText('Loading…')).toBeInTheDocument()

  // The atom resolves asynchronously, so findByText has to poll until the
  // AsyncResult transitions to Success and React re-renders.
  expect(await screen.findByText('Ada Lovelace')).toBeInTheDocument()
})
```

Atom Effects can also be tested below the component boundary. The architectural difference is that Foldkit exposes Commands as returned values. The Story can name the requested effect without running it, while the React component test observes the Effect through the atom and rendered interface.

## Scaling Complexity

An Effect Atom application grows a graph of atoms. Adding an independent atom is local, derived atoms declare their dependencies, and fine-grained subscriptions limit which components update. Understanding a cross-feature change may require following atom reads, writes, and reactivity keys across files.

A Foldkit application grows its Model, Message union, and update logic. Exhaustive matching keeps the transition catalog complete, but a large update function eventually needs to be divided into [Submodels](https://foldkit.dev/core/submodel). Submodel composition adds explicit parent-child wiring.

The trade-off is locality versus a central index. Effect Atom favors independently composable reactive cells. Foldkit favors an explicit state machine whose transitions share one runtime path.

## AI-Assisted Development

Foldkit’s closed Message unions and exhaustive updates give a coding agent a bounded list of state transitions and compile errors when a new variant is unhandled. The type system turns a new Message into a concrete to-do list. Commands, Subscriptions, and views also have distinct roles, which narrows where related code should live.

Effect Atom favors feature-local atoms, so an agent instead follows imports, derived dependencies, setters, and reactivity keys. That can make a local feature compact while requiring more repository search for changes that cross several atoms. Foldkit’s [AI tools](https://foldkit.dev/ai/overview) document its framework-specific patterns for agents that do not already know them.

## Practical Trade-offs

Architecture aside, several practical factors affect the choice.

React + Effect Atom

Foldkit

Ecosystem

React components, tooling, and libraries

[Foldkit UI](https://foldkit.dev/ui/overview)

, plus

[Mount](https://foldkit.dev/core/mount)

and

[CustomElement](https://foldkit.dev/core/custom-element)

for third-party integration

Incremental adoption

Add atoms to an existing React application

Owns the application runtime, though

[Embedding](https://foldkit.dev/core/embedding)

can mount it inside another page

Data fetching

`AsyncResult`

, SWR, Suspense, families, and Effect HTTP/RPC integration

[AsyncData](https://foldkit.dev/core/async-data)

for state, with fetching, caching, and refresh policy modeled through the Model and Commands

Fine-grained reactivity

Components subscribe to the atoms they read

Top-down view evaluation with virtual DOM diffing and

[view memoization](https://foldkit.dev/core/view-memoization)

View model

React components and JSX

Typed view functions and an HTML builder DSL

## Conclusion

They share Effect. Above that foundation, they choose different application architectures.

Choose React with Effect Atom when you want Effect-native reactive state inside React, incremental adoption, fine-grained subscriptions, and its async atom tools. Choose Foldkit when you want the Elm Architecture to govern the whole application, including rendering, lifecycle, DevTools, and tests. Effect Atom composes a graph of reactive cells inside a host framework. Foldkit gives the application one state machine and makes the rest of its tools follow from that constraint.
