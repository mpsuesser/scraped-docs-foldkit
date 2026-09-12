---
url: https://foldkit.dev/tooling/oxlint-plugin
title: "Oxlint Plugin"
description: "Install and configure @foldkit/oxlint-plugin, then see what each Foldkit-specific rule accepts and rejects."
access_date: 2026-09-12T22:55:23.086Z
current_date: 2026-09-12T22:55:23.086Z
---

# Oxlint Plugin

## Foldkit Rules

Foldkit projects use `oxlint` for general linting and `@foldkit/oxlint-plugin` for architecture and API conventions specific to Foldkit.

## Scaffolded Projects

[Create Foldkit app](https://foldkit.dev/get-started) includes `.oxlintrc.json`, a `lint` script, `oxlint`, and `@foldkit/oxlint-plugin`. Generated projects extend the recommended Foldkit preset:

```
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "plugins": ["typescript"],
  "jsPlugins": [
    {
      "name": "foldkit",
      "specifier": "@foldkit/oxlint-plugin"
    }
  ],
  "categories": {
    "correctness": "off"
  },
  "rules": {
    "no-unused-vars": [
      "error",
      {
        "argsIgnorePattern": "^_",
        "varsIgnorePattern": "^_",
        "caughtErrorsIgnorePattern": "^_",
        "destructuredArrayIgnorePattern": "^_"
      }
    ],
    "typescript/no-explicit-any": "error",
    "typescript/consistent-type-assertions": [
      "error",
      {
        "assertionStyle": "never"
      }
    ],
    "foldkit/no-noop-message": "error",
    "foldkit/got-submodel-message-name": "error",
    "foldkit/got-prefix-requires-submodel-payload": "error",
    "foldkit/no-empty-commands-array": "error",
    "foldkit/no-empty-to-parent-out-message": "error",
    "foldkit/no-empty-object-tagged-call": "error",
    "foldkit/prefer-callable-message-constructor": "error",
    "foldkit/command-binding-matches-name": "error",
    "foldkit/no-module-level-mutable-state": "error"
  },
  "ignorePatterns": [
    "dist/",
    "node_modules/",
    "repos/",
    "**/*.d.ts",
    "vite.config.ts",
    "vitest.config.ts",
    "**/*.config.js",
    "**/*.config.mjs"
  ]
}
```

Override an individual rule in the project's `rules` block when an application needs a narrower policy. The complete rule set is grouped by the part of the architecture it protects below.

## Effect Imports

### foldkit/prefer-effect-module-names

The recommended and all presets require PascalCase modules imported from `effect` to keep their exported names. Write `import { Match, Schema, String } from 'effect'`, not abbreviated or trailing-underscore aliases such as `Match as M`, `Schema as S`, or `String as String_`.

When an Effect module shares a name with a JavaScript or TypeScript global, keep the Effect import unchanged and qualify the global through `globalThis`, such as `globalThis.String`, `globalThis.Array`, or `globalThis.Record`. If an existing local or public binding must retain the module name, give the Effect import an explicit prefix such as `Order as EffectOrder`. Aliases for lowercase functions and type-only imports remain valid.

The rule safely fixes a binding and its references when the exported name is available. It reports without fixing when a rename could change binding resolution, compete with another alias for the same exported name, change an object shorthand key or named re-export, or discard a comment in the import specifier.

Disable the rule when a project deliberately keeps Effect module aliases:

```json
{
  "rules": {
    "foldkit/prefer-effect-module-names": "off"
  }
}
```

## Server Portability

### foldkit/no-nonportable-server-globals

The recommended and all presets enable this rule in `entry.server.ts`, `entry.server.tsx`, TypeScript files under a `server` directory, and `prerender.ts` or `prerender.tsx`. Files ending in `.test.ts`, `.test.tsx`, `.spec.ts`, or `.spec.tsx` are excluded.

The rule catches direct runtime reads of common browser-only globals: `document`, `window`, `navigator`, `localStorage`, `sessionStorage`, `history`, `location`, `alert`, `confirm`, `prompt`, `requestAnimationFrame`, `cancelAnimationFrame`, `requestIdleCallback`, `cancelIdleCallback`, `getComputedStyle`, `matchMedia`, `customElements`, `screen`, `IntersectionObserver`, `ResizeObserver`, and `MutationObserver`. It also catches static property reads and destructuring from the global `globalThis` object.

Local bindings, parameters, and type-only `typeof` queries remain valid. `Request`, `Response`, `Headers`, `fetch`, and `URL` remain available for host code. A host-specific file can use an Oxlint disable comment or a narrower config override when it deliberately depends on one deployment target.

This rule is a portability guardrail, not a security boundary or an exhaustive catalog of browser APIs. It does not follow aliases, resolve dynamic property names, inspect dependencies, or match filenames outside the patterns above.

## Message Naming and Construction

### foldkit/no-noop-message

Rejects catch-all Messages that make update branches and traces less meaningful. Name the event that happened instead.

```
import { defineMessageUnion } from 'foldkit/message'

// ❌ Bad
const BadMessage = defineMessageUnion({
  NoOp: {},
})

// ✅ Good
const Message = defineMessageUnion({
  ClickedSave: {},
})
```

### foldkit/no-empty-object-tagged-call

Catches no-field variants called with an unnecessary empty object. The rule recognizes namespaces whose names end in Message, Route, or State, plus unions declared in the same file with Foldkit's union helpers. Call those constructors with no arguments.

```
import { defineTaggedUnion } from 'foldkit/schema'

const Submission = defineTaggedUnion({
  NotSubmitted: {},
  Submitting: {},
})

// ❌ Bad
const badSubmission = Submission.NotSubmitted({})

// ✅ Good
const goodSubmission = Submission.NotSubmitted()
```

### foldkit/prefer-callable-message-constructor

Prevents constructing Messages by typing or casting object literals. Use the callable Schema constructor instead.

```
import { Schema } from 'effect'
import { defineMessageUnion } from 'foldkit/message'

const Message = defineMessageUnion({
  ClickedSave: {},
})
type Message = typeof Message.Type

// ❌ Bad
const badMessage: Message = {
  _tag: 'ClickedSave',
}

// ✅ Good
const goodMessage = Message.ClickedSave()
```

## Command Shape

### foldkit/command-binding-matches-name

Keeps a Command binding name in sync with the name passed to Command.define.

```
import { Effect } from 'effect'
import { Command } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'

const Message = defineMessageUnion({
  CompletedFetchUser: {},
})

// ❌ Bad
const SaveUser = Command.define('FetchUser', {
  messages: [Message.CompletedFetchUser],
  execute: Effect.succeed(Message.CompletedFetchUser()),
})

// ✅ Good
const FetchUser = Command.define('FetchUser', {
  messages: [Message.CompletedFetchUser],
  execute: Effect.succeed(Message.CompletedFetchUser()),
})
```

### foldkit/command-define-pascal-const

Requires the const holding a Command.define result to be a non-empty PascalCase identifier that matches the Command name.

```
import { Command } from 'foldkit'

// ❌ Bad
// A Command binding should be PascalCase, like the Command name it holds.
const fetchWeather = Command.define('FetchWeather', {
  messages: [SucceededFetchWeather],
  execute: fetchWeatherEffect,
})

// ✅ Good
const FetchWeather = Command.define('FetchWeather', {
  messages: [SucceededFetchWeather],
  execute: fetchWeatherEffect,
})
```

### foldkit/no-hand-rolled-command-struct

Rejects Command structs assembled by hand. Command.define attaches the identity, args, and tracing metadata a plain object literal skips.

```
import { Command } from 'foldkit'

// ❌ Bad
// Hand-rolling the Command struct skips the identity, args, and tracing
// metadata that Command.define attaches.
const SaveDraft = {
  name: 'SaveDraft',
  effect: saveDraftEffect,
}

// ✅ Good
const FetchWeather = Command.define('FetchWeather', {
  messages: [SucceededFetchWeather],
  execute: fetchWeatherEffect,
})
```

## Commands and Effects

### foldkit/acquire-release-constructs-in-acquire-body

Requires the acquire Effect passed to `Effect.acquireRelease` to construct its resource lazily. Returning a handle captured from an outer binding, or wrapping an eagerly constructed resource in `Effect.succeed`, leaves a window where interruption can leak the resource before its release action is registered.

The Effect type tracks the resource value, failure, and requirements, but not whether the resource was constructed before the acquire Effect began. That timing distinction cannot be enforced by the `Effect.acquireRelease` API or TypeScript alone, so the lint rule checks the construction shape.

```
import { Effect } from 'effect'

const closeSocket = (socket: WebSocket) => Effect.sync(() => socket.close())

// ❌ Bad
// An interruption between constructing the socket and acquire leaks it.
const socket = new WebSocket('/updates')
const badResource = Effect.acquireRelease(Effect.succeed(socket), closeSocket)

// ✅ Good
// Construct the socket inside acquire, so acquire owns the whole lifetime.
const goodResource = Effect.acquireRelease(
  Effect.sync(() => new WebSocket('/updates')),
  closeSocket,
)
```

### foldkit/prefer-command-mapmessage

Lifts a Command result Message with `Command.mapMessage` or `Command.mapMessages`, not by mapping the Effect inside `Command.mapEffect`. Mapping the Effect dispatches correctly in production but records nothing on the message-mapping chain, so Story and Scene `resolve` see the raw child Message.

`Command.mapEffect` is appropriate when the result Message stays the same and the Effect's execution changes, such as providing a service, adding retry or delay behavior, or changing its error or requirement channel. Its type preserves the result Message; use the Message-specific helpers when the result itself changes.

```
import { Effect } from 'effect'
import { Command } from 'foldkit'

// ❌ Bad
// Effect.map lifts the result Message but records nothing on the mapping chain,
// so Story/Scene resolve sees the child's raw Message.
const badCommand = Command.mapEffect(
  childCommand,
  Effect.map(message => Message.GotChildMessage({ message })),
)

// ✅ Good
// mapEffect may change execution while preserving the result Message.
const infallibleCommand = Command.mapEffect(childCommand, Effect.orDie)

// ✅ Good
// mapMessage records the lift, so resolve can recover it in tests.
const goodCommand = Command.mapMessage(childCommand, message =>
  Message.GotChildMessage({ message }),
)
```

## Model Updates

### foldkit/no-empty-commands-array

Catches a literal empty array assigned to `commands`. An ordinary update, init, boot, or component helper omits `commands` when it statically has no Commands. Computed collections remain valid, as does `commands: optionalCommands ?? []` where the next operation requires an array.

The rule can remove the property when doing so will not disturb comments, spreads, or duplicate `commands` keys. It still reports the unsafe cases without a fix.

This is a syntax-only rule. It flags any literal property named `commands`, even when the object is unrelated to an update result. If `commands: []` is genuine domain data, suppress the rule on that property with `// oxlint-disable-next-line foldkit/no-empty-commands-array`.

```
declare const model: Model
declare const commands: ReadonlyArray<Command<Message>>
declare const optionalCommands: ReadonlyArray<Command<Message>> | undefined
declare const buildCommands: (model: Model) => ReadonlyArray<Command<Message>>

// ❌ Bad
// A producer that statically creates no Commands omits the field.
const noCommands = { model, commands: [] }

// ✅ Good
const omittedCommands = { model }
const existingCommands = { model, commands }
const computedCommands = { model, commands: buildCommands(model) }

// Code that spreads, concatenates, executes, or asserts on Commands needs an array.
const normalizedCommands = { model, commands: optionalCommands ?? [] }
```

### foldkit/no-spread-in-evo

Rejects object spreads inside an evo updater. Evolve nested fields with a nested evo instead.

```
import { evo } from 'foldkit/struct'

// ❌ Bad
// Spreading a nested field inside evo defeats the point of evo.
const badUpdate = (model: Model) =>
  evo(model, {
    user: () => ({ ...model.user, name: 'Ada' }),
  })

// ✅ Good
// Evolve the nested field with a nested evo.
const goodUpdate = (model: Model) =>
  evo(model, {
    user: user => evo(user, { name: () => 'Ada' }),
  })
```

## State Modeling

### foldkit/no-switch-on-message-tag

Rejects a `switch` on a Message or state `_tag`. Use the tagged union’s `match` helper for exhaustive dispatch, or Effect `Match` when the union has no matcher, so adding a variant produces a type error instead of a silent fall-through. Matchers are also the idiomatic Foldkit form: they organize behavior around named variants and keep low-level `_tag` branching out of application logic.

```
// ❌ Bad
// A switch on _tag has no exhaustiveness check, so a new variant silently
// falls through. It also exposes dispatch mechanics instead of organizing the
// logic around named variants.
const badLabel = (message: Message): string => {
  switch (message._tag) {
    case 'Incremented':
      return 'up'
    case 'Decremented':
      return 'down'
  }
}

// ✅ Good
// The idiomatic union matcher makes a forgotten variant a type error.
const goodLabel = (message: Message): string =>
  Message.match<string>(message, {
    Incremented: () => 'up',
    Decremented: () => 'down',
  })
```

### foldkit/prefer-option-over-nullable-in-model

Requires a direct field in the `Model` Schema to represent absence with `Schema.Option`, not a nullable, undefined, or optional Schema field. The rule stays scoped to `const Model = Schema.Struct({...})`, leaving wire and API Schemas free to preserve nullable input formats.

```
import { Schema } from 'effect'

// ❌ Bad
// Nullable and optional Schemas model absence as values the update layer must
// guard.
const Model = Schema.Struct({
  currentUser: Schema.NullOr(User),
})

// ✅ Good
// Option makes presence explicit and threads through update without null checks.
const ModelWithOption = Schema.Struct({
  currentUser: Schema.Option(User),
})
```

## Routing

### foldkit/no-hardcoded-route-strings

Rejects hardcoded path and URL strings passed to link and navigation helpers. Build them from the Route module so they stay in sync with the routes.

```
import type { HtmlBuilder } from 'foldkit/html'

import { tasksRouter } from './route'

// ❌ Bad
// A hardcoded path rots when the route changes and bypasses the Route module.
const badLink = (h: HtmlBuilder<Message>) => h.a([h.Href('/tasks')], ['Tasks'])

// ✅ Good
// Build the href from the Router so it stays in sync with the route.
const goodLink = (h: HtmlBuilder<Message>) =>
  h.a([h.Href(tasksRouter())], ['Tasks'])
```

### foldkit/no-route-query-constructor-default

Rejects `Schema.withConstructorDefault` inside `Route.query`. Constructor defaults run only when a Schema constructs a value with `make`; route query parameters are decoded and encoded, so the annotation does not supply a default for a missing parameter. Use `Schema.withDecodingDefaultKey` when an absent key should decode to a value, or `Schema.OptionFromOptional` when absence belongs in the Route.

```
import { Effect, Schema, pipe } from 'effect'
import { Route } from 'foldkit'
import { literal } from 'foldkit/route'

// ❌ Bad
// Constructor defaults run only during make, not Route.query decoding.
const badSearchRouter = pipe(
  literal('search'),
  Route.query(
    Schema.Struct({
      page: Schema.FiniteFromString.pipe(
        Schema.withConstructorDefault(Effect.succeed(1)),
      ),
    }),
  ),
  Route.mapTo(SearchRoute),
)

// ✅ Good
// Use a decoding default when an absent query key should produce a value.
const goodSearchRouter = pipe(
  literal('search'),
  Route.query(
    Schema.Struct({
      page: Schema.FiniteFromString.pipe(
        Schema.withDecodingDefaultKey(Effect.succeed('1')),
      ),
    }),
  ),
  Route.mapTo(SearchRoute),
)
```

## View Keying and Accessibility

### foldkit/no-array-index-view-keys

Rejects the array index as a view key. Key by a stable Model identifier, or reordering the list patches the wrong rows.

```
import type { HtmlBuilder } from 'foldkit/html'

// ❌ Bad
// The array index is not a stable identity: reordering patches the wrong rows.
const badList = (tasks: ReadonlyArray<Task>, h: HtmlBuilder<Message>) =>
  h.ul(
    [],
    tasks.map((task, index) => h.keyed('li')(index, [], [task.title])),
  )

// ✅ Good
// Key by a stable Model identifier.
const goodList = (tasks: ReadonlyArray<Task>, h: HtmlBuilder<Message>) =>
  h.ul(
    [],
    tasks.map(task => h.keyed('li')(task.id, [], [task.title])),
  )
```

### foldkit/keyed-required-for-mapped-rows

Requires an identity-bearing mapped row element to be wrapped in keyed, so the runtime patches the right rows when the list reorders or shrinks.

```
import type { HtmlBuilder } from 'foldkit/html'

// ❌ Bad
// The row carries the task's identity (its id), so leaving it unkeyed lets the
// runtime patch the wrong row when the list reorders or shrinks.
const badList = (tasks: ReadonlyArray<Task>, h: HtmlBuilder<Message>) =>
  h.ul(
    [],
    tasks.map(task =>
      h.li([h.OnClick(ClickedTask({ id: task.id }))], [task.title]),
    ),
  )

// ✅ Good
const goodList = (tasks: ReadonlyArray<Task>, h: HtmlBuilder<Message>) =>
  h.ul(
    [],
    tasks.map(task =>
      h.keyed('li')(
        task.id,
        [h.OnClick(ClickedTask({ id: task.id }))],
        [task.title],
      ),
    ),
  )
```

### foldkit/require-rel-for-external-link

Requires target="_blank" links to carry a rel with noopener or noreferrer.

```
import type { HtmlBuilder } from 'foldkit/html'

// ❌ Bad
// target="_blank" without rel leaves the new tab able to reach window.opener.
const badLink = (h: HtmlBuilder<Message>) =>
  h.a([h.Href('https://example.com'), h.Target('_blank')], ['Docs'])

// ✅ Good
const goodLink = (h: HtmlBuilder<Message>) =>
  h.a(
    [
      h.Href('https://example.com'),
      h.Target('_blank'),
      h.Rel('noopener noreferrer'),
    ],
    ['Docs'],
  )
```

### foldkit/no-raw-dom-event-attributes

Rejects raw DOM event attributes. Use the typed event helpers so handlers dispatch Messages through the runtime.

```
import type { HtmlBuilder } from 'foldkit/html'

// ❌ Bad
// A raw DOM event attribute escapes the typed handlers and the Message flow.
const badButton = (h: HtmlBuilder<Message>) =>
  h.button([h.Attribute('onclick', 'location.reload()')], ['Reload'])

// ✅ Good
// Dispatch a Message through the typed event helper.
const goodButton = (h: HtmlBuilder<Message>) =>
  h.button([h.OnClick(ClickedReload())], ['Reload'])
```

### foldkit/no-empty-children-array

Catches an inline empty array in the children slot, on element builders and on keyed. The argument is optional, so an element with no children omits it. The shorter form needs the Foldkit release that made children optional, so bump `foldkit` alongside the plugin.

```
import type { HtmlBuilder } from 'foldkit/html'

// ❌ Bad
// The trailing [] is what the builder already defaults to, so it carries nothing.
const badDivider = (h: HtmlBuilder<Message>) =>
  h.div([h.Class('h-px bg-gray-200')], [])
const badRows = (tags: ReadonlyArray<Tag>, h: HtmlBuilder<Message>) =>
  h.ul(
    [],
    tags.map(tag => h.keyed('li')(tag.id, [h.Class(tag.className)], [])),
  )

// ✅ Good
// Omit the argument. Attributes stay required, so h.div([]) is an element with neither.
const goodDivider = (h: HtmlBuilder<Message>) =>
  h.div([h.Class('h-px bg-gray-200')])
const goodRows = (tags: ReadonlyArray<Tag>, h: HtmlBuilder<Message>) =>
  h.ul(
    [],
    tags.map(tag => h.keyed('li')(tag.id, [h.Class(tag.className)])),
  )
```

## Purity Boundaries

### foldkit/no-prevent-default-in-stream-operator

Flags `preventDefault()` inside callbacks passed to `Stream.map`, `Stream.mapEffect`, `Stream.filterMap`, `Stream.filterMapEffect`, `Stream.filter`, `Stream.filterEffect`, or `Stream.tap`. A DOM event placed into a callback-backed Stream is queued before downstream operators run, so cancellation there happens after the native listener returns and may be too late for the browser.

Use `Subscription.fromEventFilterMapPreventDefault` instead. Its mapper returns `Option.some(message)` for a handled event or `Option.none()` for an event the browser should handle normally. Foldkit calls `preventDefault()` for handled events before the native listener returns.

The rule recognizes inline callbacks and functions declared in the same module. It is intentionally conservative about the Stream's source. Suppress it when the value is not a DOM event or the Stream is deliberately executed synchronously inside a native listener.

```
import { Effect, Option, Stream } from 'effect'
import { Subscription } from 'foldkit'

// ❌ Bad: fromEventListener queues the event and returns before mapEffect runs.
const keyboardBad = Stream.fromEventListener<KeyboardEvent>(
  document,
  'keydown',
).pipe(
  Stream.mapEffect(event =>
    Effect.sync(() => {
      event.preventDefault() // The browser may have started its default action.
      return Message.PressedKey({ key: event.key })
    }),
  ),
)

// ✅ Good: Some marks Tab handled, so Foldkit cancels it inside the listener.
const keyboardGood = Subscription.fromEventFilterMapPreventDefault<
  KeyboardEvent,
  Message
>({
  target: document,
  type: 'keydown',
  toMessage: event =>
    event.key === 'Tab'
      ? Option.some(Message.PressedKey({ key: event.key }))
      : Option.none(),
})
```

### foldkit/no-impure-call-at-decision-time

Flags these direct calls unless they appear inside a recognized callback that Effect or a Foldkit lifecycle primitive defers until execution:

- `Date.now()`
- `Date()` (which ignores its arguments)
- zero-argument `new Date()`
- `Math.random()`
- `performance.now()`
- `crypto.randomUUID()`
- `crypto.getRandomValues()`

The rule reports the call wherever it is written. Assigning its result to a local variable before passing that variable to a Command does not defer it. Neither does writing the call directly in the Command args. JavaScript obtains the value before constructing the Command in both cases.

Obtain time or randomness inside the Command's `execute` callback instead. Use `Clock` or `Random` for time and ordinary randomness. For UUIDs and cryptographic randomness, use the `Crypto.Crypto` service with the platform's Crypto layer. Return the value in the result Message.

The rule recognizes the deferred callback positions in Effect and Stream. It also recognizes these Foldkit lifecycle callbacks when they are declared inline:

- `execute` in `Command.define`, `Mount.define`, and `Mount.defineStream`
- `dependenciesToStream` in `Subscription.make`
- `acquire` and `release` in `ManagedResource.make`

Not every function passed to Effect is deferred. The rule still checks functions stored as Effect values, `Effect.fromOption`'s `onNone`, callbacks passed to `Effect.run*`, transform callbacks after the body of `Effect.fn` or `Effect.fnUntraced`, and callbacks passed to Effect APIs whose names end in `Eager`. It also checks the surrounding lifecycle builders and their synchronous Model projections. For example, `Subscription.make`'s builder and `modelToDependencies` are not execution callbacks.

The recommended and all presets disable this rule in runtime entry files (`entry.ts`, `entry.tsx`, `entry.client.ts`, `entry.client.tsx`, `entry.server.ts`, and `entry.server.tsx`), where Flags and host integrations obtain outside values. The `.tsx` forms support JSX hosts, such as a React application that embeds Foldkit; Foldkit views still use the Html builder.

The presets also disable the rule in TypeScript files under a `server` directory and in `prerender.ts` or `prerender.tsx`. Those files belong to the host rather than the Foldkit application state machine, so their request handlers and build scripts do not return values through Messages. Test files remain excluded with the rest of the Foldkit rules.

This direct-call catalog does not prove that a file is pure. It recognizes static global member paths and ignores locally shadowed globals. It does not follow a method alias such as `const now = Date.now` to a later `now()` call, nor does it inspect a helper's call graph.

```
import { Crypto, Effect, Schema } from 'effect'
import { Command } from 'foldkit'

import { BrowserCrypto } from '@effect/platform-browser'

const SaveDraftWithId = Command.define('SaveDraftWithId', {
  args: { body: Schema.String, draftId: Schema.String },
  messages: [Message.CompletedSaveDraftWithId],
  execute: ({ draftId }) =>
    Effect.succeed(Message.CompletedSaveDraftWithId({ draftId })),
})

// ❌ Bad: assigning the UUID first does not defer the call.
const saveBad = (body: string) => {
  const draftId = crypto.randomUUID()

  return SaveDraftWithId({ body, draftId })
}

// ✅ Good: the runtime obtains the UUID when it executes the Command.
const SaveDraft = Command.define('SaveDraft', {
  args: { body: Schema.String },
  messages: [Message.CompletedSaveDraft],
  execute: ({ body: _body }) =>
    Effect.gen(function* () {
      const crypto = yield* Crypto.Crypto
      const draftId = yield* Effect.orDie(crypto.randomUUIDv4)
      return Message.CompletedSaveDraft({ draftId })
    }).pipe(Effect.provide(BrowserCrypto.layer)),
})

const saveGood = (body: string) => SaveDraft({ body })
```

### foldkit/no-module-level-mutable-state

Rejects module-level let and var bindings, which hold state outside the Model. Move the data into the Model, or scope a live handle to a lifecycle primitive like Mount or ManagedResource.

```
import { Schema } from 'effect'

// ❌ Bad
let requestCount = 0

// ✅ Good
export const Model = Schema.Struct({
  requestCount: Schema.Number,
})
export type Model = typeof Model.Type
```

### foldkit/no-disabling-dev-guardrails

Flags turning off the freezeModel or slow dev guardrails. Fix the mutation or slow phase they caught instead of silencing the feedback.

```
import { Runtime } from 'foldkit'

// ❌ Bad
// Turning off freezeModel silences the dev warning instead of fixing the
// mutation it caught.
const badApp = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  freezeModel: false,
})

// ✅ Good
// Leave the guardrail on and fix the in-place mutation it flags.
const goodApp = Runtime.makeApplication({ Model, init, update, view })
```

## Submodel Wiring

### foldkit/no-empty-to-parent-out-message

Flags an inline `toParentOutMessage` mapper that directly returns `undefined`. That mapper forwards nothing to the parent, so omit the property.

Partial forwarding is valid. Match every child OutMessage variant. Return a parent OutMessage for each variant you want to forward, and return `undefined` for each variant that stops at this Submodel.

The rule fixes straightforward object literals. If removal could disturb a comment, spread, dynamic computed property, or duplicate `toParentOutMessage` key, it reports the problem without changing the code. It does not inspect async functions, generators, getters, setters, or mappers referenced by name.

```
import { Option } from 'effect'
import { Update } from 'foldkit'
import { evo } from 'foldkit/struct'

import * as Settings from './settings'

// ❌ Bad
const badFoldSettings = Update.foldChild({
  update: Settings.setTheme,
  read: (model: Model) => Option.some(model.settings),
  write: (model, nextSettings) => evo(model, { settings: () => nextSettings }),
  toParentMessage: message => Message.GotSettingsMessage({ message }),
  foldOutMessage: foldSettingsOutMessage,
  // This mapper directly returns undefined, so it forwards no OutMessage.
  toParentOutMessage: () => undefined,
})

// ✅ Good
// This fold emits no parent OutMessage. Other branches in the same update may
// still emit an OutMessage.
const foldSettings = Update.foldChild({
  update: Settings.setTheme,
  read: (model: Model) => Option.some(model.settings),
  write: (model, nextSettings) => evo(model, { settings: () => nextSettings }),
  toParentMessage: message => Message.GotSettingsMessage({ message }),
  foldOutMessage: foldSettingsOutMessage,
})
```

### foldkit/got-submodel-message-name

Requires wrapper Messages around Submodel Messages to use the Got*Message convention.

```
import { defineMessageUnion } from 'foldkit/message'

import * as Child from './child'

// ❌ Bad
const BadMessage = defineMessageUnion({
  ChildChanged: { message: Child.Message },
})

// ✅ Good
const Message = defineMessageUnion({
  GotChildMessage: { message: Child.Message },
})
```

### foldkit/got-prefix-requires-submodel-payload

Reserves the Got* prefix for Submodel wrappers. Any Got-prefixed Message must include a child Message payload named message.

```
import { Schema } from 'effect'
import { defineMessageUnion } from 'foldkit/message'

import * as Child from './child'

{
  // ❌ Bad: Got is reserved for Submodel wrappers.
  const Message = defineMessageUnion({
    GotWeather: { temperature: Schema.Number },
  })
}

{
  // ✅ Good: use a name that does not start with Got for Command results.
  const Message = defineMessageUnion({
    ReceivedWeather: { temperature: Schema.Number },
  })
}

{
  // ❌ Bad: Got-prefixed wrappers must carry child Messages.
  const Message = defineMessageUnion({
    GotChildMessage: { id: Schema.String },
  })
}

{
  // ✅ Good: Got wraps a child Message.
  const Message = defineMessageUnion({
    GotChildMessage: {
      id: Schema.String,
      message: Child.Message,
    },
  })
}
```

### foldkit/wrap-child-output-in-got-message

Requires child Command and Subscription output to be wrapped through a Got*Message constructor, preserving the one-wrap-per-level Submodel convention.

```
import { Command } from 'foldkit'

// ❌ Bad
// The mapper wraps child output in a plain parent Message, not a Got*Message,
// so this Submodel level never records the wrap.
const badCommands = Command.mapMessages(childCommands, message =>
  ForwardedChildMessage({ message }),
)

// ✅ Good
const goodCommands = Command.mapMessages(childCommands, message =>
  GotChildMessage({ message }),
)
```

### foldkit/got-wrapper-carries-only-routing

Keeps a Got wrapper payload to the child Message plus routing keys: message, id, or keys ending in Id.

```
import { Schema } from 'effect'
import { defineMessageUnion } from 'foldkit/message'

// ❌ Bad
// A Got wrapper carries the child Message plus routing context only. Extra
// payload like timestamp belongs on the child Message or a parent Message.
const BadMessage = defineMessageUnion({
  GotSettingsMessage: {
    message: Settings.Message,
    timestamp: Schema.Number,
  },
})

// ✅ Good
// message plus routing keys (id, or keys ending in Id) only.
const Message = defineMessageUnion({
  GotCounterMessage: {
    id: Schema.String,
    message: Counter.Message,
  },
})
```

### foldkit/no-child-message-construction-in-root

Rejects constructing a child Message variant from a parent. Expose a child-owned update capability that applies the internal fact, then integrate it with `Update.foldChild` or `Update.foldChildStep`. A child-owned view, Command, or Subscription may still construct that child's Messages; the boundary is ownership, not file spelling. See [Informing Submodels](https://foldkit.dev/patterns/informing-submodels) for the complete pattern.

```
// ❌ Bad
// The root reaches into the child Message namespace to build a child Message.
const badRouting = () =>
  GotChildMessage({ message: Child.Message.ClickedSave() })

// ✅ Good
// The child exports an update capability. The parent folds the complete child
// result without importing or constructing its internal Message.
const foldChildSave = Update.foldChildStep({
  update: Child.save,
  read: model => Option.some(model.child),
  write: (model, nextChild) => evo(model, { child: () => nextChild }),
  toParentMessage: message => GotChildMessage({ message }),
})

const goodRouting = model => foldChildSave(model)
```

### foldkit/selection-submodel-factory-at-module-scope

Requires selection component factories, such as Combobox, Listbox, Menu, and Tabs, to be created at module scope so their identity stays stable across renders.

```
import { Listbox } from '@foldkit/ui'

const sortListbox = Listbox.create()

// ❌ Bad
// Re-creating the factory on each update gives it a fresh identity, so its
// internal selection state never persists.
const badUpdate = (model: Model, message: Message) => {
  const listbox = Listbox.create()
  return listbox.update(model.sort, message)
}

// ✅ Good
// Reuse the module-scope factory.
const goodUpdate = (model: Model, message: Message) =>
  sortListbox.update(model.sort, message)
```

## Lifecycle Handles

### foldkit/mount-factory-must-use-element

Requires a Mount's `execute` to read or write its element. If it never touches the element, the cause was misidentified and Mount is the wrong primitive.

```
import { Effect } from 'effect'
import { Mount } from 'foldkit'

// ❌ Bad
// execute never reads its element, so Mount is the wrong primitive here.
const MountAnalytics = Mount.define('MountAnalytics', {
  messages: [CompletedMountAnalytics],
  execute: () => Effect.sync(() => startAnalytics()),
})

// ✅ Good
// execute reads its element to wire the observer.
const MountResize = Mount.define('MountResize', {
  messages: [CompletedMountResize],
  execute: ({ element }) => Effect.sync(() => resizeObserver.observe(element)),
})
```

### foldkit/no-duplicate-onmount-per-element

Rejects two OnMount handlers on one element, where the second silently overwrites the first.

```
import type { HtmlBuilder } from 'foldkit/html'

// ❌ Bad
// Two OnMount handlers on one element: the second overwrites the first.
const badPanel = (h: HtmlBuilder<Message>) =>
  h.div([h.OnMount(AnchorPopover()), h.OnMount(SyncScroll())])

// ✅ Good
// One OnMount per element; combine the work into a single Mount if needed.
const goodPanel = (h: HtmlBuilder<Message>) =>
  h.div([h.OnMount(AnchorPopover())])
```

## DOM and UI Helpers

### foldkit/lazy-view-stable-references

Requires lazy view slots to be declared at module scope so their references stay stable and the memoization actually hits its cache.

```
import { type HtmlBuilder, createLazy } from 'foldkit/html'

// ❌ Bad
// Creating the lazy slot inside the view gives it a new identity every render,
// so the memoized view never hits its cache.
const badView = (model: Model, h: HtmlBuilder<Message>) => {
  const lazyHeader = createLazy()
  return lazyHeader(renderHeader, [model.title, h])
}

// ✅ Good
// Declare the lazy slot once at module scope.
const lazyHeader = createLazy()
const goodView = (model: Model, h: HtmlBuilder<Message>) =>
  lazyHeader(renderHeader, [model.title, h])
```
