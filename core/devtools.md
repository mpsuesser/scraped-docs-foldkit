---
url: https://foldkit.dev/core/devtools
title: "DevTools"
description: "Inspect Message history, Model changes, Commands, and Mounts in the development overlay. Configure time travel, filtering, history limits, and AI dispatch."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

# DevTools

## Using the Overlay

By default, Foldkit DevTools records every Message flowing through your app. Open the overlay to inspect what happened, what changed, and which work the update returned. The overlay renders inside a shadow DOM, so it does not interfere with your styles or layout.

You can see it in action right now. Look for the tab on the bottom right edge of this page.

The panel lists every recorded Message, with the newest at the bottom. Select a row and use the four inspector tabs:

- `Model` shows the full state tree and highlights changed paths.
- `Message` shows the Message payload.
- `Commands` lists the Commands returned by update.
- `Mounts` shows which Mounts started or ended during that render.

The `Live` badge tells you whether the inspector shows the latest state or a past entry. In time-travel mode, selecting an earlier row installs a paused historical view. It does not pause the live application behind that view. Select `Resume` to patch the latest live view back into the DOM. `Clear` drops the recorded history without restarting the app.

AI agent integration

Foldkit also exposes DevTools to AI agents over the Model Context Protocol. See the [DevTools MCP](https://foldkit.dev/ai/mcp) page for setup.

DevTools records application data

DevTools records Models, Message payloads, Command arguments, and Mount arguments. A configured [DevTools MCP](https://foldkit.dev/ai/mcp) server lets an agent read that data. Keep secrets out of browser state, use Effect `Redacted` for sensitive values the browser must temporarily hold, and disable the MCP relay or DevTools when the Runtime may contain data the agent should not receive.

A browser may still need to hold a short-lived access token. Wrap it before it enters the Model, keep the `Redacted` value intact in any Message or Command arguments that carry it, and recover the raw value only where the Command constructs the authenticated request. The numbered comments trace the Model-to-request path through the example.

**Redacting a short-lived access token**

```typescript
import { Effect, Redacted, Schema } from 'effect'
import { Command } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'

const accessTokenLabel = 'access token'

// 1. Prevent JSON encoding, and give the placeholder a useful label.
const AccessToken = Schema.Redacted(Schema.String, {
  label: accessTokenLabel,
  disallowJsonEncode: true,
})
type AccessToken = typeof AccessToken.Type

const makeAccessToken = (accessToken: string): AccessToken =>
  Redacted.make(accessToken, { label: accessTokenLabel })

// MODEL

// 2. Store the wrapper in the Model, never the raw string.
const Model = Schema.Struct({ accessToken: AccessToken })
type Model = typeof Model.Type

const init = (accessToken: string) => ({
  model: { accessToken: makeAccessToken(accessToken) },
})

// MESSAGE

const Message = defineMessageUnion({
  CompletedFetchProfile: {},
  FailedFetchProfile: {},
})

// COMMAND

const FetchProfile = Command.define('FetchProfile', {
  // 3. Require a Redacted value because DevTools records Command arguments.
  args: { accessToken: AccessToken },
  messages: [Message.CompletedFetchProfile, Message.FailedFetchProfile],
  execute: ({ accessToken }) =>
    Effect.tryPromise(() =>
      fetch('/api/profile', {
        headers: {
          // 4. Recover the raw token only at the request boundary that needs it.
          Authorization: `Bearer ${Redacted.value(accessToken)}`,
        },
      }),
    ).pipe(
      Effect.as(Message.CompletedFetchProfile()),
      Effect.catch(() => Effect.succeed(Message.FailedFetchProfile())),
    ),
})

const fetchProfile = (model: Model) =>
  FetchProfile({ accessToken: model.accessToken })
```

DevTools displays the value as `<redacted:access token>`. This prevents accidental inspection and serialization; it does not encrypt the token or hide it from someone who controls the browser.

## Development and Production

DevTools are enabled by default in development. Recording and the MCP bridge live in the core runtime. The browser overlay ships separately in `@foldkit/devtools`. When that package is installed as a development dependency, `@foldkit/vite-plugin` mounts the overlay automatically during development. Production builds omit it without an application-level environment check.

Add a `devTools` object to `makeApplication` only when you need to configure DevTools or allow MCP dispatch. To include the overlay in production, move `@foldkit/devtools` to regular `dependencies` and set `show: 'Always'`. You do not need to import the overlay.

**Configuring DevTools**

```typescript
import { Runtime } from 'foldkit'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  container: document.getElementById('root'),
  devTools: {
    position: 'BottomLeft',
  },
})

Runtime.run(application)
```

## Configuration

The `devTools` field accepts an object with the following optional properties, or `false` to disable DevTools entirely.

### show

`'Development'` (the default) enables DevTools only in development. `'Always'` enables them in all environments, including production.

### position

Controls where the badge and panel appear on screen. One of `'BottomRight'` (default), `'BottomLeft'`, `'TopRight'`, or `'TopLeft'`.

### mode

`'TimeTravel'` (the default) renders the state at an earlier Message and pauses that historical view. The live Model, history, Commands, Subscriptions, ManagedResources, and live-acquired Mounts continue normally. Their Messages and state changes keep appearing in the panel even though the historical DOM stays in place. Foldkit event handlers and Mounts created by the historical render cannot dispatch, and the overlay blocks pointer interaction. A surviving live Mount observes its `viewStateChanges` Stream and must make its imperative integration read-only while paused so keyboard or programmatic DOM interaction cannot produce Messages. Select `Resume` to patch the latest live view back into the DOM; a replay-created Mount whose element is reused is released before the live action starts.

`'Inspect'` lets you browse recorded states without pausing the app. Use it when visitors can open DevTools in production or staging.

Pass `{ development, production }` to choose a mode for each environment. When `show: 'Always'` keeps DevTools available in production, use `'TimeTravel'` for local debugging and `'Inspect'` in production. Selecting a row will not pause a visitor's app.

**TimeTravel locally, Inspect in production**

```typescript
import { Runtime } from 'foldkit'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  container: document.getElementById('root'),
  devTools: {
    show: 'Always',
    mode: { development: 'TimeTravel', production: 'Inspect' },
    banner: 'Welcome to our app! Browse the state tree to see how it works.',
  },
})

Runtime.run(application)
```

### banner

An optional string displayed as a banner at the top of the panel. Useful for welcoming visitors or leaving a note for your team.

### Message

The application’s `Message` Schema. Required only for AI agent integration: when set and the running app is connected to the [DevTools MCP](https://foldkit.dev/ai/mcp) server, agents can dispatch Messages into the live runtime. The Schema decodes inbound dispatch payloads at the bridge boundary and rejects mismatches with a clean error. Omit this field to disable agent dispatch entirely.

### excludeFromHistory

A list of Message `_tag` values that DevTools should not record. The Messages still run through update and change the application as usual. They do not appear in the history panel or incur the per-Message diff cost.

Use this option when animation frames, pointer moves, scroll events, or another high-frequency source would flood the history.

Do not use `excludeFromHistory` to hide sensitive data. The current Model and the Models at later recorded entries still include changes made by an excluded Message.

When the list contains at least one tag, DevTools stores a full Model snapshot for every recorded entry. That preserves changes made by excluded Messages when you travel to a recorded state. Excluded Messages also update the `Live` Model view, but they do not append a history entry or compute a diff.

**Excluding high-frequency Messages from history**

```typescript
import { Runtime } from 'foldkit'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  subscriptions,
  container: document.getElementById('root'),
  devTools: {
    excludeFromHistory: ['TickedFrame', 'MovedPointer'],
  },
})

Runtime.run(application)
```

### maxEntries

The maximum number of recorded Messages retained before DevTools evicts the oldest entry. The default is `100`, and values are clamped between `20` and `500`.

Smaller values reduce work under high Message rates. Larger values provide more history. Memory use grows with `maxEntries` and Model size, especially when `excludeFromHistory` makes every recorded entry store a full Model snapshot.

**Raising the DevTools history cap**

```typescript
import { Runtime } from 'foldkit'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  container: document.getElementById('root'),
  devTools: {
    maxEntries: 250,
  },
})

Runtime.run(application)
```

### keyframeInterval

The number of recorded Messages between full Model snapshots. The default is `31`, and the minimum is `1`.

To reconstruct an entry, DevTools starts at the nearest earlier snapshot and replays update. A smaller interval stores more snapshots but shortens that replay. Set the interval to `1` when update is expensive and time-travel feels slow. Every entry then has its own snapshot, so no replay is needed.

DevTools automatically uses `1` when `excludeFromHistory` is active because excluded Messages are not available for replay.

**Snapshotting every entry for constant-time jumps**

```typescript
import { Runtime } from 'foldkit'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  container: document.getElementById('root'),
  devTools: {
    keyframeInterval: 1,
  },
})

Runtime.run(application)
```
