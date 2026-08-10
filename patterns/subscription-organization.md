---
url: https://foldkit.dev/patterns/subscription-organization
title: "Subscription Organization"
description: "Canonical layout for subscription wiring across nested submodels."
access_date: 2026-08-10T01:37:55.778Z
current_date: 2026-08-10T01:37:55.778Z
---

# Subscription Organization

## Overview

Once your app uses [Submodel](https://foldkit.dev/core/submodel) and any of those Submodels need [Subscriptions](https://foldkit.dev/core/subscriptions), a question shows up: where do the Subscription definitions live, and who translates the Stream of child Messages into the parent’s Message type?

This page documents the canonical answer. The shape mirrors how `update` and `view` compose across Submodels. `Subscription.lift` translates a child Submodel’s Stream into the parent’s Message type, and `Subscription.aggregate` combines that with any other Subscription records the level holds.

## The Composition Levels

Subscriptions compose in levels. Each level can declare its own Streams via `Subscription.make` and lift child Streams via `Subscription.lift` into the level’s Message type. The Stream emerging at the top is in the root’s Message type, ready for the runtime to dispatch through `update`.

```
         ready for runtime processing
                      ↑
+--------------------------------------------+
| in subscription.ts (root)                  |
| lift to Message                            |
| via GotSettingsMessage                     |
| to declare                                 |
| Stream<Message>                            |
+--------------------------------------------+
                      ↑
+--------------------------------------------+
| in page/settings/subscription.ts           |
| lift to Settings.Message                   |
| via GotThemeMenuMessage                    |
| to declare                                 |
| Stream<Settings.Message>                   |
+--------------------------------------------+
                      ↑
+--------------------------------------------+
| in page/settings/themeMenu/subscription.ts |
| declare                                    |
| Stream<ThemeMenu.Message>                  |
+--------------------------------------------+
```

## The Composition Verbs

Three verbs on the `Subscription` namespace do almost all of the composition work. Knowing which one applies at a given level is what makes a Subscription file easy to read.

Verb

What it does

When to reach for it

`Subscription.make`

Declares a Subscriptions record at the current level. Each entry pairs a dependency field map with

`modelToDependencies`

and

`dependenciesToStream`

callbacks.

The current level has Subscriptions of its own to declare.

`Subscription.lift`

Lifts a child Submodel’s Subscriptions into the current level’s Model and Message via one

`toChildModel`

lens and one

`toParentMessage`

constructor. An optional

`when`

gates the whole record on the parent Model.

Embedding a child whose Subscriptions all share the same wrapper Message.

`Subscription.aggregate`

Combines two or more Subscriptions records into one. Throws at startup on duplicate keys instead of silently overriding.

A level combines multiple sources of Subscriptions (lifted children, inline entries, or both).

## Organization Principles

### Submodel Cohesion

A Submodel’s Subscription wiring belongs next to its Model, Message, init, update, and view. Subscriptions that emit Messages for a Submodel are part of that Submodel’s set of concerns.

### One Wrap Per Level

A Subscription file produces Messages in its own Message type, and only that one. When a parent embeds it, the parent wraps the emitted Messages via `Subscription.lift`.

### Uniform Interface

Every Submodel that exposes Subscriptions exports one named value: a `subscriptions` record built via `Subscription.make`. A parent embeds it by combining it through `Subscription.aggregate` alongside its own Subscriptions. One record stays right even when the parent [gates](#gating) some of its entries and not others, because a gate attaches per entry at the lift. A Submodel never organizes its exports around its parent's conditions.

## Putting It Together

Here is one composition traced through every level: a leaf Submodel, a composing Submodel that embeds it, and a root that combines them.

### The Leaf Submodel

A leaf Submodel has no children with Subscriptions of their own. Its `subscription.ts` declares entries via `Subscription.make`:

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

A Submodel that hosts Subscription-bearing children lifts each child via `Subscription.lift`, declares any local Subscriptions via `Subscription.make`, and combines them through `Subscription.aggregate`:

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

The root `subscription.ts` uses the same shape as a composing Submodel. Its lifts target the root `Model` and `Message`:

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

A Submodel decides through its own `modelToDependencies` whether its Streams should run, but it can only read facts it holds. The route it sits behind is the common thing it cannot see: a page Submodel listening for key presses has no way to know the app is showing a different page.

The parent owns that half of the condition, so the gate lives on the parent’s `lift` call. `when` receives the **parent** Model, never the child’s, and a gated entry runs only while it returns `true`:

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

A closed gate is a teardown, not a pause. The entry’s Stream stops, and the child’s `modelToDependencies` does not run again until the parent reopens the gate, so child state that changes behind a closed gate causes no restarts.

`when` takes either shape. One predicate gates every entry in the record, which is the common case where the whole child answers to one parent condition. A map keyed by entry name gates entries individually, for a child whose Subscriptions answer to different conditions: a page that holds both a WebSocket stream and a keyboard listener wants the socket to survive navigation and the keys to stop at it. Entries the map omits are lifted ungated.

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

Ownership splits cleanly. The parent writes `when` and answers it from parent state, and the child neither declares nor sees the gate: it keeps holding its own condition in `modelToDependencies`.

Lifts chain, so a gate attaches at whichever level knows the condition. A record lifted through two levels can be gated at the outer one on a fact only the root holds, and the level in between passes it along without knowing a gate is there. Gates at different levels compose: the entry runs only while every gate above it is open.
