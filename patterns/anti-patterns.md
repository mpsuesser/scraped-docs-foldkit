---
url: https://foldkit.dev/patterns/anti-patterns
title: "Anti-patterns"
description: "Architectural warning signs in Foldkit apps, with idiomatic replacements for ambiguous state, leaky Submodel boundaries, misplaced side effects, stale async results, Command ordering, and live handles."
access_date: 2026-09-12T22:55:23.086Z
current_date: 2026-09-12T22:55:23.086Z
---

# Anti-patterns

## How to Use This Guide

A Foldkit application can behave correctly and still be difficult to explain, test, or change. The examples on this page reflect common anti-patterns in Foldkit applications.

When reviewing an application, ask:

- Can the Model describe a state that should never occur?
- Does more than one place claim to hold the same state?
- Does the Model Schema enforce the same states its TypeScript type promises?
- Does the Runtime control every effect and live resource?
- Does every Message name the event that DevTools should show?
- Does a Command result Message carry enough context for update to decide whether it still applies to the current Model?
- Does dependent or replacement work begin only after the result it depends on?
- Does each Submodel own its state transitions?

The sections below show how each problem appears in code and the Foldkit design that avoids it.

## Make Impossible States Unrepresentable

Imagine a screen that can be idle, loading, showing data, or showing an error. Representing those mutually exclusive states in separate fields allows the Model to describe impossible combinations.

```
// ❌ Bad: isLoading, maybeData, and maybeError can contradict one another.

import { Option, Schema } from 'effect'

const Model = Schema.Struct({
  isLoading: Schema.Boolean,
  maybeData: Schema.Option(Schema.Array(Schema.String)),
  maybeError: Schema.Option(Schema.String),
})
type Model = typeof Model.Type

const impossibleModel: Model = {
  isLoading: true,
  maybeData: Option.some(['Current data']),
  maybeError: Option.some('The request failed'),
}
```

Every handler must now remember which fields to set and which fields to clear. If one handler forgets, view and update receive a state the application never intended to support.

Use one discriminated union when only one state can be active. Each variant carries exactly the data available in that state.

```
// ✅ Good: one variant describes the screen at a time.

import { Schema } from 'effect'
import { defineTaggedUnion } from 'foldkit/schema'

const LoadState = defineTaggedUnion({
  Idle: {},
  Loading: {},
  Loaded: { data: Schema.Array(Schema.String) },
  Failed: { error: Schema.String },
})

const Model = Schema.Struct({
  loadState: LoadState,
})
type Model = typeof Model.Type
```

See [State with Variants](https://foldkit.dev/core/model#state-with-variants) for the general Foldkit pattern. [AsyncData](https://foldkit.dev/core/async-data) provides the usual remote-data states.

For smaller custom flows, enforce legal transitions in an exhaustive Message match. Consider the experimental [Machine](https://foldkit.dev/core/machine) module when a substantial transition table needs to be collected and traced. Use a Boolean for an independent yes-or-no fact and `Option` when one value may be absent independently of the surrounding state.

## Store Each Piece of State Once

Suppose the Model stores `items`, the current `query`, and a filtered copy named `visibleItems`. Every handler that changes either `items` or `query` must also update `visibleItems`.

```
// ❌ Bad: visibleItems can stop matching items and query.

import { Schema } from 'effect'
import type { Update } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'
import { evo } from 'foldkit/struct'

const Item = Schema.Struct({ id: Schema.String, name: Schema.String })

const Model = Schema.Struct({
  items: Schema.Array(Item),
  query: Schema.String,
  visibleItems: Schema.Array(Item),
})
type Model = typeof Model.Type

const Message = defineMessageUnion({
  UpdatedQuery: { query: Schema.String },
})
type Message = typeof Message.Type

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    UpdatedQuery: ({ query }) => ({
      model: evo(model, { query: () => query }),
    }),
  })
```

The source fields and `visibleItems` now give two answers to the same question: which items should be shown. Store only `items` and `query`, then derive the filtered collection when the view needs it.

```
// ✅ Good: the Model stores the source values and derives the result.

import { Schema } from 'effect'

const Item = Schema.Struct({ id: Schema.String, name: Schema.String })

const Model = Schema.Struct({
  items: Schema.Array(Item),
  query: Schema.String,
})
type Model = typeof Model.Type

const visibleItems = (model: Model) =>
  model.items.filter(item =>
    item.name.toLowerCase().includes(model.query.toLowerCase()),
  )
```

The same rule applies outside derived collections. Browser storage may persist a snapshot of the Model, and a DOM attribute may display its current state. Neither should become a second value that update treats as authoritative. A module variable that mirrors a Model field creates the same conflict while also hiding the value from the Runtime and DevTools.

Copying parent-owned data into a child Model just so the child can render it also creates two sources of truth. Pass that data through the child's `viewInputs` instead. When child update needs current parent context while handling its own Message, pass the value through update's third `context` argument. Use a public child helper when a parent-owned change must trigger a child transition.

Cache a derived Model value only after profiling shows that deriving it is expensive. If the cost comes from building or diffing the view, prefer [view memoization](https://foldkit.dev/core/view-memoization). A value users can change independently is domain state, not a cache. The [performance guide](https://foldkit.dev/faq/performance#the-optimization-toolkit) compares Model caching with view memoization.

## Define the Model with Schema

Suppose a handwritten TypeScript type says that `session` contains a user, while the Model Schema accepts any value in that field. TypeScript lets update and view read `userId`, but the decoder can produce a value with no `userId` at all.

```
// ❌ Bad: TypeScript and the decoder disagree about session.

import { Schema } from 'effect'

type Model = {
  session: {
    userId: string
    displayName: string
  }
}

const ModelSchema = Schema.Struct({
  session: Schema.Unknown,
})
```

Define the Model with Schema and derive the TypeScript type from it. The decoder and the type checker then agree on the Model's shape.

```
// ✅ Good: the TypeScript type comes from the Schema.

import { Schema } from 'effect'

const Session = Schema.Struct({
  userId: Schema.String,
  displayName: Schema.String,
})

const Model = Schema.Struct({
  session: Session,
})
type Model = typeof Model.Type
```

Deriving the Model type from its Schema matters even when the application does not decode an API response. Foldkit uses the Model Schema when it restores state after a development reload. Use `Schema.Unknown` only when no narrower Schema describes the value at that boundary, and represent every valid state variant in the Schema itself. See [Model](https://foldkit.dev/core/model) for the complete pattern.

## Keep Effects Out of init, update, and view

Calling `fetch` inside view starts another request whenever Foldkit renders that view. Reading `Date.now()` or `Math.random()` inside init or update lets the same inputs produce a different Model. Writing to the DOM or browser storage inside update repeats that write when DevTools replays the Message.

```
// ❌ Bad: update performs a DOM effect during the state transition.

import type { Update } from 'foldkit'
import { evo } from 'foldkit/struct'

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    ClickedOpenDialog: () => {
      document.querySelector<HTMLInputElement>('#search-input')?.focus()

      return { model: evo(model, { dialogState: () => 'Open' }) }
    },
  })
```

Keep init, update, and view deterministic. The `ClickedOpenDialog` handler should set `dialogState` to `Open` and return a `FocusSearchInput` Command.

```
// ✅ Good: update returns a Command and the Runtime performs the effect.

import { Effect } from 'effect'
import { Command, Dom, type Update } from 'foldkit'
import { evo } from 'foldkit/struct'

const FocusSearchInput = Command.define('FocusSearchInput', {
  messages: [Message.CompletedFocusSearchInput],
  execute: Dom.focus('#search-input').pipe(
    Effect.ignore,
    Effect.as(Message.CompletedFocusSearchInput()),
  ),
})

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    ClickedOpenDialog: () => ({
      model: evo(model, { dialogState: () => 'Open' }),
      commands: [FocusSearchInput()],
    }),
    CompletedFocusSearchInput: () => ({ model }),
  })
```

`Effect`, `Stream`, and `Layer` values describe work; constructing one should not perform that work. Supply the lazy description to the boundary that controls its lifetime, and let the Runtime execute it.

Use [Flags](https://foldkit.dev/core/init-and-flags#flags) when the initial Model needs outside data. For work after initialization, choose the Runtime boundary based on what starts the work and how long it must live.

## Keep Live Handles Behind Runtime Boundaries

Imagine a chat screen that needs a `WebSocket` only while the visitor is in a room. Opening the socket inside update and storing it in a module variable puts it outside the Runtime's lifecycle. The Runtime therefore cannot close it when the visitor leaves the room, and replaying the Message in DevTools can open another connection.

Storing the socket in the Model does not give it a Runtime-controlled lifetime. The live connection cannot be recreated from the Model Schema after a development reload, and update would still be responsible for opening and closing it. Keep the room and connection status in the Model, and let a ManagedResource own the `WebSocket` itself.

```
// ❌ Bad: module state owns the WebSocket outside the Runtime.

let socket: WebSocket | undefined

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    JoinedRoom: ({ roomId }) => {
      socket = new WebSocket(`/rooms/${roomId}`)
      return { model }
    },
    LeftRoom: () => {
      socket?.close()
      socket = undefined
      return { model }
    },
  })
```

```
// ✅ Good: Model state controls the WebSocket's lifetime.

import { Effect, Schema } from 'effect'
import { Command, ManagedResource } from 'foldkit'

const ChatSocket = ManagedResource.tag<WebSocket>()('ChatSocket')
const RoomRequirements = Schema.Struct({ roomId: Schema.String })

const managedResources = ManagedResource.make<Model, Message>()(entry => ({
  chatSocket: entry(Schema.Option(RoomRequirements), {
    resource: ChatSocket,
    modelToMaybeRequirements: model => model.maybeRoomRequirements,
    acquire: ({ roomId }) =>
      Effect.try(() => new WebSocket(`/rooms/${roomId}`)),
    release: socket => Effect.sync(() => socket.close()),
    onAcquired: () => Message.AcquiredChatSocket(),
    onReleased: () => Message.ReleasedChatSocket(),
    onAcquireError: error =>
      Message.FailedAcquireChatSocket({ error: globalThis.String(error) }),
  }),
}))

const SendChatMessage = Command.define('SendChatMessage', {
  args: { text: Schema.String },
  messages: [Message.CompletedSendChatMessage, Message.FailedSendChatMessage],
  execute: ({ text }) =>
    ChatSocket.get.pipe(
      Effect.flatMap(socket => Effect.try(() => socket.send(text))),
      Effect.match({
        onFailure: error =>
          Message.FailedSendChatMessage({
            error: globalThis.String(error),
          }),
        onSuccess: () => Message.CompletedSendChatMessage(),
      }),
    ),
})
```

ManagedResource fits when Commands or Subscriptions need the typed handle. In this example, `acquire` returns the Effect that creates the socket, and `release` returns the Effect that closes it when the room requirements disappear. The Runtime performs both Effects. When the Runtime performs `SendChatMessage`, that Command's Effect retrieves the live `ChatSocket`. When the socket only produces an event stream, a Subscription can own the connection directly.

Choose the boundary whose lifetime matches the work:

Cause or lifetime

Boundary

One-time work started while handling a Message

[Command](https://foldkit.dev/core/commands)

Work that requires one rendered element

[Mount](https://foldkit.dev/core/mount)

Ongoing work controlled by Model-derived dependencies

[Subscription](https://foldkit.dev/core/subscriptions)

A typed handle needed while a Model condition holds

[ManagedResource](https://foldkit.dev/core/managed-resources)

A service shared for the entire application Runtime

[Resources](https://foldkit.dev/core/resources)

A native web component whose properties and events remain declarative

[CustomElement](https://foldkit.dev/core/custom-element)

If a Mount's `execute` function never uses its element, the work should not be tied to that element's lifetime. Choose the boundary based on what actually starts and stops the work. The [Mount comparison](https://foldkit.dev/core/mount#when-to-reach-for-mount) and [Managed Resources](https://foldkit.dev/core/managed-resources) guides cover these choices in depth.

## Name Messages After Facts

Suppose a visitor clicks a refresh button. A Message named `FetchWeather` sounds like an instruction to perform a request, but it does not say what happened in the interface.

```
// ❌ Bad: FetchWeather tells update what to do, not what happened.

const Message = defineMessageUnion({
  FetchWeather: {},
})

const handlers = {
  FetchWeather: () => ({ model, commands: [FetchWeather()] }),
}
```

Name the Message `ClickedRefresh`. When update handles it, update may return a Command named `FetchWeather`. Message names describe events that happened; Command names describe work to perform.

```
// ✅ Good: the Message records the click. The Command names the work.

const Message = defineMessageUnion({
  ClickedRefresh: {},
})

const handlers = {
  ClickedRefresh: () => ({ model, commands: [FetchWeather()] }),
}
```

See [Event Names](https://foldkit.dev/best-practices/messages#events) and [Command Result Names](https://foldkit.dev/best-practices/messages#command-results) for the complete conventions.

## Never Name a Message `NoOp`

Suppose an event handler must return a Message, but update does not need to change the Model for that event. Naming the Message `NoOp` means DevTools and tests record only `NoOp`, not the mouse click that occurred.

```
// ❌ Bad: NoOp does not identify the event.

const Message = defineMessageUnion({
  NoOp: {},
})

const handleMouseClick = () => Message.NoOp()
```

Name the Message `IgnoredMouseClick` even though its update handler returns the unchanged Model.

```
// ✅ Good: IgnoredMouseClick records the event even when the Model stays unchanged.

const Message = defineMessageUnion({
  IgnoredMouseClick: {},
})

const handleMouseClick = () => Message.IgnoredMouseClick()

const handlers = {
  IgnoredMouseClick: () => ({ model }),
}
```

See [Descriptive Results](https://foldkit.dev/best-practices/messages#no-noop) for more examples of Messages whose handlers leave the Model unchanged.

## Include Enough Context in Command Result Messages

Imagine that a visitor searches for `ca` and then immediately searches for `cat`. Both requests remain in flight, and the `ca` response may arrive last. If the result Message carries only the suggestions, the older response overwrites the current results.

```
// ❌ Bad: the result does not identify which search produced it.

import { Schema } from 'effect'
import type { Update } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'
import { evo } from 'foldkit/struct'

const Message = defineMessageUnion({
  SucceededSearch: { suggestions: Schema.Array(Schema.String) },
})
type Message = typeof Message.Type

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    SucceededSearch: ({ suggestions }) => ({
      model: evo(model, { suggestions: () => suggestions }),
    }),
  })
```

Assign each search an incrementing generation number and include it in `SucceededSearch`. The handler accepts the response only when that number matches the current generation in the Model.

```
// ✅ Good: the result carries the generation that started the search.

import { Schema } from 'effect'
import type { Update } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'
import { evo } from 'foldkit/struct'

const Message = defineMessageUnion({
  SucceededSearch: {
    generation: Schema.Number,
    suggestions: Schema.Array(Schema.String),
  },
})
type Message = typeof Message.Type

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    SucceededSearch: ({ generation, suggestions }) => {
      if (generation !== model.searchGeneration) {
        return { model }
      }

      return { model: evo(model, { suggestions: () => suggestions }) }
    },
  })
```

Choose context that tells update whether the result still applies. A generation number distinguishes repeated searches for the same query, an entity id can distinguish entity requests, and a revision can distinguish editor saves. Omit additional context when an older result cannot overwrite or invalidate newer state.

## Sequence Dependent Commands Through Messages

Suppose saving a draft must finish before the application leaves the editor. When update returns `SaveDraft` and `NavigateToDocuments` together, the Runtime starts both Commands independently. Their positions in the array do not determine which finishes first.

```
// ❌ Bad: both Commands start independently.

const handlers = {
  ClickedSave: () => ({
    model,
    commands: [SaveDraft(), NavigateToDocuments()],
  }),
}
```

Have update return only `SaveDraft`. When `SucceededSaveDraft` reaches update, return `NavigateToDocuments`.

```
// ✅ Good: update returns NavigateToDocuments after SaveDraft succeeds.

const handlers = {
  ClickedSave: () => ({
    model,
    commands: [SaveDraft()],
  }),

  SucceededSaveDraft: () => ({
    model,
    commands: [NavigateToDocuments()],
  }),
}
```

Return Commands together only when they can run independently. Otherwise, when the first result Message reaches update, have update return the next Command. If no distinct result Message or update decision belongs between two operations, sequence them inside one Command.

## Wait for Interruption Before Starting Replacement Work

Suppose `FetchSuggestions` is interruptible and a new search should replace the running search. When update returns `FetchSuggestions.Interrupt` and a new `FetchSuggestions` Command together, the Runtime starts both without an ordering guarantee. The replacement may register under the same interruption key before the Interrupt runs, so the Interrupt can cancel both requests and leave no current search in flight.

```
// ❌ Bad: interruption and replacement start independently.

import { Number } from 'effect'

const handlers = {
  UpdatedQuery: ({ query }) => {
    const nextSearchGeneration = Number.increment(model.searchGeneration)

    return {
      model: evo(model, {
        query: () => query,
        searchGeneration: () => nextSearchGeneration,
      }),
      commands: [
        FetchSuggestions.Interrupt(() =>
          Message.CompletedCancelFetchSuggestions(),
        ),
        FetchSuggestions({ query, generation: nextSearchGeneration }),
      ],
    }
  },
}
```

On the first query change, move the search from `Running` to `Cancelling`, increment its generation, and return only the Interrupt. Further query changes while `Cancelling` update the stored query without dispatching another Interrupt. When `CompletedCancelFetchSuggestions` reaches update, return one `FetchSuggestions` Command with the latest query and generation from the Model.

```
// ✅ Good: update returns FetchSuggestions after interruption.

import { Number } from 'effect'
import type { Update } from 'foldkit'

type UpdateReturn = Update.Return<Model, Message>

const handlers = {
  UpdatedQuery: ({ query }) =>
    SearchState.match<UpdateReturn>(model.searchState, {
      Running: () => ({
        model: evo(model, {
          query: () => query,
          searchGeneration: Number.increment,
          searchState: () => SearchState.Cancelling(),
        }),
        commands: [
          FetchSuggestions.Interrupt(() =>
            Message.CompletedCancelFetchSuggestions(),
          ),
        ],
      }),
      Cancelling: () => ({
        model: evo(model, { query: () => query }),
      }),
    }),

  CompletedCancelFetchSuggestions: () => ({
    model: evo(model, {
      searchState: () => SearchState.Running(),
    }),
    commands: [
      FetchSuggestions({
        query: model.query,
        generation: model.searchGeneration,
      }),
    ],
  }),
}
```

The old `FetchSuggestions` Command may finish just before the Interrupt runs, leaving `SucceededFetchSuggestions` already queued. Incrementing the generation before interruption makes that completion stale. The result handler must still compare the result's generation with the current Model before accepting it. See [Include Enough Context in Command Result Messages](#reject-stale-async-results) and [Sequencing Replacement Work](https://foldkit.dev/core/commands#sequencing-replacement-work) for both parts of the protocol.

## Preserve Submodel Boundaries

A parent stores a Submodel's Model, but the child owns every transition of that state. If the parent changes a child field directly, the child's update function does not run.

```
// ❌ Bad: the parent changes the Settings Model directly.

const handlers = {
  ClickedResetSettings: () => ({
    model: evo(model, {
      settings: settings => evo(settings, { theme: () => 'Light' }),
    }),
  }),
}
```

The direct change may skip validation, Commands, or OutMessages that the child needs. Importing an internal child Message does not fix the boundary: it turns that private event into an API the parent depends on.

```
// ❌ Bad: the parent imports and constructs an internal Settings Message.

import { Message as SettingsMessage } from './settings/message'

const foldSettings = Update.foldChild({
  update: Settings.update,
  read: (model: Model) => Option.some(model.settings),
  write: (model, nextSettings) => evo(model, { settings: () => nextSettings }),
  toParentMessage: message => Message.GotSettingsMessage({ message }),
})

const handlers = {
  ClickedResetSettings: () =>
    foldSettings(model, SettingsMessage.ChangedTheme({ theme: 'Light' })),
}
```

Manually copying the child Model from a helper result causes a separate problem. The parent can keep the next child Model while silently discarding Commands returned with it.

```
// ❌ Bad: manual copying drops the Commands returned with the child Model.

const handlers = {
  ClickedResetSettings: () => {
    const settingsReset = Settings.setTheme(model.settings, 'Light')

    return {
      model: evo(model, {
        settings: () => settingsReset.model,
      }),
    }
  },
}
```

Export a helper from the child for transitions the parent is allowed to request. The helper constructs the internal Message and immediately passes it through the child's update function. The parent calls the helper without importing the child's Message constructor.

```
// ✅ Good: the child exposes a helper that still runs its update.

// SETTINGS SUBMODEL

import { Message as SettingsMessage } from './message'

export const setTheme = (model: Model, theme: Theme) =>
  update(model, SettingsMessage.ChangedTheme({ theme }))

// PARENT UPDATE

const foldSettingsTheme = Update.foldChild({
  update: Settings.setTheme,
  read: (model: Model) => Option.some(model.settings),
  write: (model, nextSettings) => evo(model, { settings: () => nextSettings }),
  toParentMessage: message => Message.GotSettingsMessage({ message }),
})

const handlers = {
  ClickedResetSettings: () => foldSettingsTheme(model, 'Light'),
}
```

`Update.foldChild` and `Update.foldChildStep` write the returned child Model into the parent and lift the child's Commands. When the child can emit OutMessages, pass `foldOutMessage` so the parent handles every variant instead of accidentally omitting one. The [Submodel guide](https://foldkit.dev/core/submodel) explains child Messages and OutMessages. [Informing Submodels](https://foldkit.dev/patterns/informing-submodels) covers parent-owned facts that a child needs to hear about.

## Know What Linting Can Catch

The [Foldkit linter](https://foldkit.dev/tooling/oxlint-plugin) can recognize code shapes such as a module-level `let`, `Date.now()` inside update, a parent constructing a child Message, or a Mount whose `execute` function never uses its element. It cannot decide whether two domain states may coexist, whether an async result can become stale, or which part of an application should own a value. Those questions still require design review.
