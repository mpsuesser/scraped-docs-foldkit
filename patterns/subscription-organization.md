---
url: https://foldkit.dev/patterns/subscription-organization
title: "Subscription Organization"
description: "Organize Subscription records by ownership and lift child Subscriptions through nested Model and Message types."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Subscription Organization

## Lifting Child Subscriptions

A Submodel owns the Subscriptions that produce its Messages. Its parent lifts those Subscriptions into the parent Model and Message types, then aggregates them with other Subscription records at that level.

This mirrors the other halves of the boundary. `Update.foldChild` lifts child update, `h.submodel` lifts child view, and `Subscription.lift` lifts child Streams.

## The Composition Levels

Each level declares local entries with `Subscription.make` and lifts child records with `Subscription.lift`. By the time a Stream reaches the root, it emits root Messages that the Runtime can dispatch through update. This diagram follows one leaf record through those lifts:

```
page/settings/themeMenu/
  subscription.ts
  Subscription.make
  Stream<ThemeMenu.Message>
               │
       Subscription.lift
 wraps with GotThemeMenuMessage
               ▼
page/settings/
  subscription.ts
  Stream<Settings.Message>
               │
       Subscription.lift
   wraps with GotSettingsMessage
               ▼
subscription.ts (root)
  Stream<Message>
               │
               ▼
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

Combines records and throws at startup when two entries use the same key.

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

```
// page/settings/themeMenu/subscription.ts
import { Effect, Schema as S, Stream } from 'effect'
import { Subscription } from 'foldkit'

import { type Message, PressedEscape } from './message'
import type { Model } from './model'

export const subscriptions = Subscription.make<Model, Message>()(entry => ({
  escapeKey: entry(
    { isOpen: S.Boolean },
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

A composing Submodel lifts child records, declares any local entries, and aggregates the results.

```
// page/settings/subscription.ts
import { Effect, Schema as S, Stream } from 'effect'
import { Subscription } from 'foldkit'

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
  toChildModel: model => model.themeMenu,
  toParentMessage: message => GotThemeMenuMessage({ message }),
})

const localSubscriptions = Subscription.make<Model, Message>()(entry => ({
  unsavedChangesWarning: entry(
    { hasUnsavedChanges: S.Boolean },
    {
      modelToDependencies: model => ({
        hasUnsavedChanges: model.hasUnsavedChanges,
      }),
      dependenciesToStream: ({ hasUnsavedChanges }) =>
        Stream.when(
          Stream.fromEventListener<BeforeUnloadEvent>(
            window,
            'beforeunload',
          ).pipe(Stream.map(StartedNavigationAway)),
          Effect.sync(() => hasUnsavedChanges),
        ),
    },
  ),
}))

export const subscriptions = Subscription.aggregate<Model, Message>()(
  themeMenuSubscriptions,
  localSubscriptions,
)
```

### The Root

The root uses the same shape. Its lifts target the root Model and Message.

```
// subscription.ts
import { Effect, Schema as S, Stream } from 'effect'
import { Subscription } from 'foldkit'

import { ChangedSystemTheme, GotSettingsMessage, type Message } from './message'
import type { Model } from './model'
import * as Settings from './settings'

const settingsSubscriptions = Subscription.lift(Settings.subscriptions)<
  Model,
  Message
>({
  toChildModel: model => model.settings,
  toParentMessage: message => GotSettingsMessage({ message }),
})

const localSubscriptions = Subscription.make<Model, Message>()(entry => ({
  systemTheme: entry(
    { isSystemPreference: S.Boolean },
    {
      modelToDependencies: model => ({
        isSystemPreference: model.themePreference === 'System',
      }),
      dependenciesToStream: ({ isSystemPreference }) =>
        Stream.when(
          Stream.fromEventListener<MediaQueryListEvent>(
            window.matchMedia('(prefers-color-scheme: dark)'),
            'change',
          ).pipe(Stream.map(ChangedSystemTheme)),
          Effect.sync(() => isSystemPreference),
        ),
    },
  ),
}))

export const subscriptions = Subscription.aggregate<Model, Message>()(
  settingsSubscriptions,
  localSubscriptions,
)
```

## Gating a Lifted Record

A child can express conditions from its own Model in its dependencies and Stream construction. It cannot see parent-owned state such as the active Route.

Put a parent-owned condition in `when` on the lift. The predicate receives the parent Model. The gated entries run only while it returns `true`.

```
// subscription.ts
import { Subscription } from 'foldkit'

import { GotSettingsMessage, type Message } from './message'
import type { Model } from './model'
import * as Settings from './settings'

const settingsSubscriptions = Subscription.lift(Settings.subscriptions)<
  Model,
  Message
>({
  toChildModel: model => model.settings,
  toParentMessage: message => GotSettingsMessage({ message }),
  when: ({ route }) => route._tag === 'Settings',
})

export const subscriptions = Subscription.aggregate<Model, Message>()(
  settingsSubscriptions,
)
```

Closing a gate tears down the Stream. Foldkit also stops calling the child's `modelToDependencies` until the gate reopens, so hidden child changes do not restart it.

`when` accepts either one predicate for the whole record or a map of predicates by entry name. An omitted entry remains ungated. For example: a Room page can keep its WebSocket alive across navigation while gating its keyboard listener to the active Room Route.

```
// subscription.ts
import { Subscription } from 'foldkit'

import { GotRoomMessage, type Message } from './message'
import type { Model } from './model'
import * as Room from './room'

// The Room page holds two Subscriptions: a WebSocket stream that should
// outlive navigation, and a keyboard listener that should not. Naming one
// entry gates it and leaves the other alone.
const roomSubscriptions = Subscription.lift(Room.subscriptions)({
  toChildModel: (model: Model) => model.room,
  toParentMessage: (message: Room.Message): Message =>
    GotRoomMessage({ message }),
  when: { roomKeyboard: ({ route }) => route._tag === 'Room' },
})

export const subscriptions = Subscription.aggregate<Model, Message>()(
  roomSubscriptions,
)
```

The parent owns `when`. The child keeps its child-owned conditions in its own Subscription definition.

Attach each gate at the level that owns its condition. When a record passes through several levels, all gates compose. The entry runs only while every gate above it is open.
