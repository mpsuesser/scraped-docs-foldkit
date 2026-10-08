---
url: https://foldkit.dev/core/dom
title: "Dom"
description: "Use Effects for one-time DOM work and composable Streams for events, media queries, and key bindings."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

# Dom

## Overview

An application may need to act on the document once after update or observe browser input while part of the UI is active. The `Dom` module provides Effects for one-time operations and Streams for ongoing input.

Use a Dom Effect when a Message should cause a one-time DOM operation. For example: opening a dialog can return a Command that focuses its first input. The operation stays outside view, and its result still comes back through update as a Message.

Use a Dom Stream when browser events, media-query changes, or key bindings should produce Messages over time. A [Subscription](https://foldkit.dev/core/subscriptions) runs ongoing work according to Model-derived dependencies. A [Mount](https://foldkit.dev/core/mount) runs it while a rendered element exists.

## Using Dom Effects

Each Effect helper exposes its failure type in the Effect channel. `Dom.focus` returns `Effect.Effect<void, ElementNotFound>`, while helpers without an expected application failure, such as `Dom.lockScroll`, return `Effect.Effect<void>`.

Wrap the helper in a Command and map its success or failure into one of that Command's declared Messages.

**Focusing an input from a Command**

```typescript
import { Effect } from 'effect'
import { Command, Dom } from 'foldkit'

const FocusEmailInput = Command.define('FocusEmailInput', {
  messages: [Focused],
  execute: Dom.focus('#email-input').pipe(Effect.ignore, Effect.as(Focused())),
})
```

Most helpers that resolve a live element wait until Foldkit has committed the latest render before querying the DOM. This lets update return a Command for an element that the same Message just brought into the view. You do not need to add `Render.afterCommit` before `Dom.focus`, `Dom.showDialog`, `Dom.clickElement`, `Dom.scrollIntoView`, or `Dom.advanceFocus`.

`Dom.showDialog` resolves to `true` when it installs the focus trap, return focus, stack entry, and optional modal isolation. It resolves to `false` when that Dialog id already holds those resources, so concurrent lifecycle recovery and application Commands do not acquire them twice.

Scrolling has two later-timing variants. `Dom.scrollIntoViewAfterPaint` waits until the target has been painted, which suits a route that just inserted a fragment target. `Dom.scrollIntoViewIfNotVisible` also waits through paint by default, but accepts `{ when: 'Commit' }` when the first visible frame should already be scrolled.

Cleanup and global-state helpers run immediately because they do not need a newly rendered target. These include `Dom.closeDialog`, `Dom.releaseDialogResources`, `Dom.lockScroll`, `Dom.unlockScroll`, and `Dom.restoreInert`. `Dom.waitForAnimationSettled` has its own timing contract: it checks the target's active Web Animations on the next animation frame and waits for them to settle.

Use Render for custom timing

Dom helpers include the timing their operation needs. Reach for [Render](https://foldkit.dev/core/render) directly when writing a custom Command or DOM-observing Subscription that must wait for a Foldkit commit or a browser paint.

### Selector Failures

Helpers that require one matching element fail with `ElementNotFound` when the selector resolves to the wrong element type or no element at all:

- `Dom.focus`
- `Dom.showDialog`
- `Dom.closeDialog`
- `Dom.clickElement`
- `Dom.scrollIntoView`
- `Dom.scrollIntoViewAfterPaint`
- `Dom.scrollIntoViewIfNotVisible`
- `Dom.advanceFocus`

Catch a meaningful failure with `Effect.catch` and turn it into a Message. Use `Effect.ignore` only when a missing target is expected and does not matter, such as a stale focus Command after navigation.

## Using Dom Streams

Each Dom Stream helper returns a composable Stream, not a complete Subscription entry. Compose the Stream with Effect Stream operators, then give it to a Subscription or Mount. For a listener attached to one rendered element, `Mount.defineStream` starts it when the element appears and stops it when the element leaves the view:

**Element pointer Stream owned by a Mount**

```typescript
import { Schema } from 'effect'
import { Dom, Mount } from 'foldkit'
import type { Html, HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'

const Message = defineMessageUnion({
  MovedPointer: { clientX: Schema.Number, clientY: Schema.Number },
})
type Message = typeof Message.Type

const TrackPointer = Mount.defineStream('TrackPointer', {
  messages: [Message.MovedPointer],
  execute: ({ element }) =>
    Dom.streamFromEvent({
      target: element,
      type: 'pointermove',
      mapEvent: event =>
        Message.MovedPointer({
          clientX: event.clientX,
          clientY: event.clientY,
        }),
    }),
})

const panelView = (h: HtmlBuilder<Message>): Html =>
  h.div([h.Class('h-48'), h.OnMount(TrackPointer())])
```

The [Subscriptions guide](https://foldkit.dev/core/subscriptions) covers Model-driven lifetimes and `Subscription.persistentEntry`.

### Event Streams

`Dom.streamFromEvent` turns an `EventTarget` into a Stream. The target may be `window`, `document`, or a rendered element. The helper adds the listener when the Stream starts and removes it when the Stream stops.

Its `mapEvent` callback can produce any output type, including the raw event. A Subscription or Mount that runs the Stream checks that its output is a declared Message type.

**Window keydown Stream in a Subscription**

```typescript
import { Effect, Schema, Stream } from 'effect'
import { Dom, Subscription } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'

// MESSAGE

const Message = defineMessageUnion({
  PressedKey: { key: Schema.String },
})
type Message = typeof Message.Type

// MODEL

const Model = Schema.Struct({
  isListening: Schema.Boolean,
})
type Model = typeof Model.Type

// SUBSCRIPTION

const subscriptions = Subscription.make<Model, Message>()(entry => ({
  shortcut: entry(
    { isListening: Schema.Boolean },
    {
      modelToDependencies: model => ({ isListening: model.isListening }),
      dependenciesToStream: ({ isListening }) =>
        Stream.when(
          Dom.streamFromEvent({
            target: window,
            type: 'keydown',
            mapEvent: event => Message.PressedKey({ key: event.key }),
          }),
          Effect.sync(() => isListening),
        ),
    },
  ),
}))
```

The `mapEvent` mapper runs synchronously in the same call stack as the browser event, so it may call `event.preventDefault()` unless the listener is passive. Some browsers default wheel and touch listeners on global targets to passive, where cancellation is ignored. Pass `options: { passive: false }` when cancelling those events. Pass `target` as a thunk if it may not exist until the scope opens; pass always-present globals such as `window` and `document` directly.

#### Typed Event Targets

The `type` field accepts only event names declared by the target, and `mapEvent` receives the corresponding event type. For example, `window` with `'keydown'` gives the mapper a `KeyboardEvent`. A bare `EventTarget` accepts any name and reports `Event`. Annotate a target with `Dom.TypedEventTarget` to declare its custom events, including `CustomEvent` detail:

**Typed custom EventTarget**

```typescript
import { Dom } from 'foldkit'

const slowWarningTarget: Dom.TypedEventTarget<{
  'foldkit:slow-warning': CustomEvent<{ durationMs: number }>
}> = new EventTarget()

const slowWarnings = Dom.streamFromEvent({
  target: slowWarningTarget,
  type: 'foldkit:slow-warning',
  mapEvent: event => event.detail.durationMs,
})
```

Annotating a native target adds its declared events without losing the native ones. If a declared event uses the same name as a native event, the declared type takes precedence.

#### Filtered Events and Synchronous Cancellation

When only some events should produce a value, use `Dom.streamFromEventFilterMap`. Its `filterMapEvent` returns `Option.some(value)` to emit it or `Option.none()` to ignore the event. A mapper that never emits produces a `Stream<never>`, which still composes wherever a Message-producing Stream is expected.

**Filtered Escape key presses**

```typescript
import { Option } from 'effect'
import { Dom } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'

const Message = defineMessageUnion({
  PressedEscape: {},
})

const escapePresses = Dom.streamFromEventFilterMap({
  target: document,
  type: 'keydown',
  filterMapEvent: event =>
    event.key === 'Escape'
      ? Option.some(Message.PressedEscape())
      : Option.none(),
})
```

Cancellation must happen inside the listener callback. A downstream `Stream` operator runs after the browser has committed the default action. When a handled event should also cancel its default action, use `Dom.streamFromEventFilterMapPreventDefault`. Its `filterMapEvent` returns `Option.some(value)` to handle the event or `Option.none()` to leave its default behavior intact. The helper evaluates the mapper, calls `preventDefault()`, and queues the value before the native listener returns. Both filtered helpers infer their Stream output from `filterMapEvent`. The cancelling helper registers the listener with `passive: false` by default and does not accept `passive: true`, which would make cancellation ineffective.

**Cancel handled search shortcuts**

```typescript
import { Option } from 'effect'
import { Dom } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'

const Message = defineMessageUnion({
  PressedSearchShortcut: {},
})

const searchShortcuts = Dom.streamFromEventFilterMapPreventDefault({
  target: document,
  type: 'keydown',
  filterMapEvent: event =>
    (event.metaKey || event.ctrlKey) && event.key.toLowerCase() === 'k'
      ? Option.some(Message.PressedSearchShortcut())
      : Option.none(),
})
```

### Media Queries

`Dom.streamFromMediaQuery` creates a Stream from a CSS media query. When the Stream starts, it emits the query's current `matches` value through `mapMatches`. It emits again whenever the value changes. Handle those values as Messages in update to store the result in the Model. Most apps therefore do not need a separate `window.matchMedia` read at boot. An app that must use the value before its Subscriptions start, such as one that applies a theme before hydration, should still read it at boot.

Reading the current value also prevents stale state when a gated entry restarts. Suppose a color-scheme Subscription runs only while the theme preference is `System`. The user selects `Dark`, changes the operating system to a light theme, and then selects `System` again. A new `change` listener waits for the next change, so the Model still records a dark system theme. `Dom.streamFromMediaQuery` reads the current light value as soon as the Stream restarts.

**Reduced motion media query**

```typescript
import { Schema } from 'effect'
import { Dom, Subscription } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'

// MESSAGE

const Message = defineMessageUnion({
  ChangedReducedMotion: { isReducedMotion: Schema.Boolean },
})
type Message = typeof Message.Type

// MODEL

const Model = Schema.Struct({
  isReducedMotion: Schema.Boolean,
})
type Model = typeof Model.Type

// SUBSCRIPTION

const subscriptions = Subscription.make<Model, Message>()(_entry => ({
  reducedMotion: Subscription.persistentEntry(
    Dom.streamFromMediaQuery({
      query: '(prefers-reduced-motion: reduce)',
      mapMatches: isMatching =>
        Message.ChangedReducedMotion({ isReducedMotion: isMatching }),
    }),
  ),
}))
```

Creating the Stream does not access `window`; `window.matchMedia` is called only when the Stream starts. The same helper works for `prefers-reduced-motion`, `prefers-color-scheme`, and viewport breakpoints such as `(max-width: 1023px)`.

### Key Bindings

`Dom.streamFromKeyBindings` builds a `keydown` Stream from a declarative key-binding table. It listens on `document` by default; pass `target` to listen on a different `EventTarget`, including an element owned by a Mount. Use `keys` with a string for one press, such as `'Escape'` or `'Mod+K'`, and an array for an ordered sequence, such as `['G', 'H']`. Every step in a sequence uses the same grammar, including modifiers.

Modifier matching is exact: `'Mod+K'` does not also match Shift-Mod-K. `Mod` resolves to Meta on Apple platforms and Control elsewhere; `modKey` provides a deterministic override when needed. Matching uses the layout-aware `KeyboardEvent.key`, so include `Shift` and the resulting character for shifted punctuation. `Space` and `Plus` name keys that would otherwise be awkward in the `+`-separated syntax.

By default, a binding calls `preventDefault()` and does not fire from an `input`, `textarea`, `select`, or contenteditable composed path. `whileTyping: 'Allow'` opts in bindings such as Escape that must work inside an editor. Events during IME composition and held-key repeats are ignored; a one-press binding can opt into repeats with `whenRepeated: 'Allow'`. An event that an element-level handler already canceled is also ignored, so local interactions take precedence over global bindings.

#### Sequences

Sequences may have any length and expire after one second unless `sequenceTimeout` overrides the duration. The helper rejects duplicate bindings, a one-press binding that is also a sequence prefix, and shared sequence prefixes with inconsistent `preventDefault` policies. A mismatched key clears the current sequence and is reconsidered as a fresh press.

#### Model-Dependent Key Bindings

The Stream's output type comes from each binding's `mapEvent`. Put a fixed table in `Subscription.persistentEntry`, or build the table inside an entry when availability follows the Model. Derive `isEnabled` from that entry's dependencies, as the example does for Escape. If the meaning of a key depends on the Model, dispatch a factual Message such as `PressedEscape` and make the decision in update; `mapEvent` should not read application state.

**Model-dependent key bindings**

```typescript
import { Schema } from 'effect'
import { Dom, Subscription } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'
import { defineTaggedUnion } from 'foldkit/schema'

const SearchState = defineTaggedUnion({
  Closed: {},
  Open: {},
})

const Model = Schema.Struct({
  searchState: SearchState,
})
type Model = typeof Model.Type

const Message = defineMessageUnion({
  PressedSearchShortcut: {},
  PressedEscape: {},
  PressedHomeShortcut: {},
})
type Message = typeof Message.Type

const subscriptions = Subscription.make<Model, Message>()(entry => ({
  keyBindings: entry(
    { searchState: SearchState },
    {
      modelToDependencies: model => ({ searchState: model.searchState }),
      dependenciesToStream: ({ searchState }) =>
        Dom.streamFromKeyBindings<Message>({
          bindings: [
            {
              keys: 'Mod+K',
              whileTyping: 'Allow',
              mapEvent: () => Message.PressedSearchShortcut(),
            },
            {
              keys: 'Escape',
              isEnabled: searchState._tag === 'Open',
              whileTyping: 'Allow',
              mapEvent: () => Message.PressedEscape(),
            },
            {
              keys: ['G', 'H'],
              mapEvent: () => Message.PressedHomeShortcut(),
            },
          ],
        }),
    },
  ),
}))
```

## Full API Surface

The [Dom API reference](https://foldkit.dev/api-reference/dom) lists every helper with its signature and an inline example.
