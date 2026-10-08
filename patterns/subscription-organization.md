---
url: https://foldkit.dev/patterns/subscription-organization
title: "Subscription Organization"
description: "Organize Subscription records by ownership and lift child Subscriptions through nested Model and Message types."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

# Subscription Organization

## Lifting Child Subscriptions

A Submodel owns the Subscriptions that produce its Messages. Its parent lifts those Subscriptions into the parent Model and Message types, then aggregates them with other Subscription records at that level.

This mirrors the other halves of the boundary. `Update.foldChild` lifts child update, `h.submodel` lifts child view, and `Subscription.lift` lifts child Streams.

## The Composition Levels

Each level declares local entries with `Subscription.make` and lifts child records with `Subscription.lift`. By the time a Stream reaches the root, it emits root Messages that the Runtime can dispatch through update. This diagram follows one leaf record through those lifts:

```
+-------------------------------+
| ThemeMenu                     |
| Stream<ThemeMenu.Message>     |
+-------------------------------+
  |
  | Subscription.lift
  | wraps with GotThemeMenuMessage
  v
+-------------------------------+
| Settings                      |
| Stream<Settings.Message>      |
+-------------------------------+
  |
  | Subscription.lift
  | wraps with GotSettingsMessage
  v
+-------------------------------+
| Root                          |
| Stream<Message>               |
+-------------------------------+
  |
  v
Runtime
```

## The Composition Verbs

Three functions build the hierarchy.

Verb

What it does

When to reach for it

`Subscription.make`

Declares local entries from dependency Schemas,

`modelToDependencies`

, and

`dependenciesToStream`

.

The current level owns a Subscription.

`Subscription.lift`

Reads a child Model and wraps each emitted child Message. An optional

`when`

adds a parent-owned gate.

A child exports a Subscriptions record.

`Subscription.aggregate`

Combines records, infers their shared types, and rejects duplicate keys at startup.

A level has more than one local or lifted record.

## Organization Principles

### Submodel Cohesion

A Subscription that emits child Messages belongs inside that child's folder. The child exports it without knowing which parent will lift it.

### One Wrap Per Level

Each `subscription.ts` produces only the Message type for its level. Every `Subscription.lift` adds one wrapper, just as one `h.submodel` boundary does for view handlers.

### Uniform Interface

Export one `subscriptions` record from the child. The parent decides whether to lift every entry, gate the whole record, or gate named entries. The child does not split its exports around parent-owned conditions.

## Putting It Together

The next three snippets trace one record from a leaf, through a composing Submodel, to the root.

### The Leaf Submodel

A leaf declares its entries with `Subscription.make`.

**Leaf Submodel Subscription file**

```typescript
// page/settings/themeMenu/subscription.ts
import { Effect, Schema, Stream } from 'effect'
import { Subscription } from 'foldkit'

import { type Message, PressedEscape } from './message'
import type { Model } from './model'

export const subscriptions = Subscription.make<Model, Message>()(entry => ({
  escapeKey: entry(
    { isOpen: Schema.Boolean },
    {
      modelToDependencies: model => ({ isOpen: model.isOpen }),
      dependenciesToStream: ({ isOpen }) =>
        Stream.when(
          Stream.fromEventListener<KeyboardEvent>(document, 'keydown').pipe(
            Stream.filter(event => event.key === 'Escape'),
            Stream.map(PressedEscape),
          ),
          Effect.sync(() => isOpen),
        ),
    },
  ),
}))
```

### The Composing Submodel

A composing Submodel lifts child records, declares any local entries, and aggregates the results. Each lift supplies a `read` that returns an `Option` of the child Model. An always-present child is wrapped in `Option.some`.

**Composing Submodel Subscription file**

```typescript
// page/settings/subscription.ts
import { Effect, Option, Schema, Stream } from 'effect'
import { Dom, Subscription } from 'foldkit'

import {
  GotThemeMenuMessage,
  type Message,
  StartedNavigationAway,
} from './message'
import type { Model } from './model'
import * as ThemeMenu from './themeMenu'

const themeMenuSubscriptions = Subscription.lift(ThemeMenu.subscriptions)<
  Model,
  Message
>({
  read: model => Option.some(model.themeMenu),
  toParentMessage: message => GotThemeMenuMessage({ message }),
})

const localSubscriptions = Subscription.make<Model, Message>()(entry => ({
  unsavedChangesWarning: entry(
    { hasUnsavedChanges: Schema.Boolean },
    {
      modelToDependencies: model => ({
        hasUnsavedChanges: model.hasUnsavedChanges,
      }),
      dependenciesToStream: ({ hasUnsavedChanges }) =>
        Stream.when(
          Dom.streamFromEventFilterMapPreventDefault({
            target: window,
            type: 'beforeunload',
            filterMapEvent: event => {
              event.returnValue = true
              return Option.some(StartedNavigationAway())
            },
          }),
          Effect.sync(() => hasUnsavedChanges),
        ),
    },
  ),
}))

export const subscriptions = Subscription.aggregate(
  themeMenuSubscriptions,
  localSubscriptions,
)
```

### The Root

The root uses the same shape. Its lifts target the root Model and Message.

**Root Subscription file**

```typescript
// subscription.ts
import { Effect, Option, Schema, Stream } from 'effect'
import { Dom, Subscription } from 'foldkit'

import { ChangedSystemTheme, GotSettingsMessage, type Message } from './message'
import type { Model } from './model'
import * as Settings from './settings'

const settingsSubscriptions = Subscription.lift(Settings.subscriptions)<
  Model,
  Message
>({
  read: model => Option.some(model.settings),
  toParentMessage: message => GotSettingsMessage({ message }),
})

const localSubscriptions = Subscription.make<Model, Message>()(entry => ({
  systemTheme: entry(
    { isSystemPreference: Schema.Boolean },
    {
      modelToDependencies: model => ({
        isSystemPreference: model.themePreference === 'System',
      }),
      dependenciesToStream: ({ isSystemPreference }) =>
        Stream.when(
          Dom.streamFromMediaQuery({
            query: '(prefers-color-scheme: dark)',
            mapMatches: isDark =>
              ChangedSystemTheme({ theme: isDark ? 'Dark' : 'Light' }),
          }),
          Effect.sync(() => isSystemPreference),
        ),
    },
  ),
}))

export const subscriptions = Subscription.aggregate(
  settingsSubscriptions,
  localSubscriptions,
)
```

## Optional Children

Return `Some(child)` from `read` when the child is present, or `None` when it is absent. Foldkit stops the child's Subscriptions and skips its dependency functions while `read` returns `None`.

**Reading an optional child Model**

```typescript
import { Option } from 'effect'
import { Subscription } from 'foldkit'

import { Message } from './message'
import type { Model } from './model'
import * as SignedIn from './signedIn'

const readSignedIn = (model: Model) =>
  Option.liftPredicate(model, model => model._tag === 'SignedIn')

const signedInSubscriptions = Subscription.lift(SignedIn.subscriptions)<
  Model,
  Message
>({
  read: readSignedIn,
  toParentMessage: message => Message.GotSignedInMessage({ message }),
})

export const subscriptions = Subscription.aggregate(signedInSubscriptions)
```

## Gating a Lifted Record

A child can express conditions from its own Model in its dependencies and Stream construction. It cannot see parent-owned state such as the active Route.

Put a parent-owned condition in `when` on the lift. The predicate receives the parent Model. The gated entries run only while it returns `true` and `read` returns a child. A closed gate skips `read` and the child’s dependency functions.

**Route-gated lift**

```typescript
// subscription.ts
import { Option } from 'effect'
import { Subscription } from 'foldkit'

import { GotSettingsMessage, type Message } from './message'
import type { Model } from './model'
import * as Settings from './settings'

const settingsSubscriptions = Subscription.lift(Settings.subscriptions)<
  Model,
  Message
>({
  read: model => Option.some(model.settings),
  toParentMessage: message => GotSettingsMessage({ message }),
  when: ({ route }) => route._tag === 'Settings',
})

export const subscriptions = Subscription.aggregate(settingsSubscriptions)
```

Closing a gate tears down the Stream. Foldkit also stops calling the child's `modelToDependencies` until the gate reopens, so hidden child changes do not restart it.

`when` accepts either one predicate for the whole record or a map of predicates by entry name. An omitted entry has no additional activity condition, but still stops when `read` returns `None`. For example: a Room page can keep its WebSocket alive across navigation while gating its keyboard listener to the active Room Route.

**Per-entry gated lift**

```typescript
// subscription.ts
import { Option } from 'effect'
import { Subscription } from 'foldkit'

import { GotRoomMessage, type Message } from './message'
import type { Model } from './model'
import * as Room from './room'

// The Room page holds two Subscriptions: a WebSocket stream that should
// outlive navigation, and a keyboard listener that should not. Naming one
// entry gates it and leaves the other alone.
const roomSubscriptions = Subscription.lift(Room.subscriptions)({
  read: (model: Model) => Option.some(model.room),
  toParentMessage: (message: Room.Message): Message =>
    GotRoomMessage({ message }),
  when: { roomKeyboard: ({ route }) => route._tag === 'Room' },
})

export const subscriptions = Subscription.aggregate(roomSubscriptions)
```

The parent owns `when`. The child keeps its child-owned conditions in its own Subscription definition.

Attach each gate at the level that owns its condition. When a record passes through several levels, all gates compose. The entry runs only while every gate above it is open.
