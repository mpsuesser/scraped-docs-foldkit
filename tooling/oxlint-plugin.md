---
url: https://foldkit.dev/tooling/oxlint-plugin
title: "Oxlint Plugin"
description: "Install and configure @foldkit/oxlint-plugin, then see what each Foldkit-specific rule accepts and rejects."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Oxlint Plugin

## Foldkit Rules

Foldkit projects use `oxlint` for general linting and `@foldkit/oxlint-plugin` for architecture and API conventions specific to Foldkit.

## Scaffolded Projects

[Create Foldkit app](https://foldkit.dev/get-started/getting-started) includes `.oxlintrc.json`, a `lint` script, `oxlint`, and `@foldkit/oxlint-plugin`. Generated projects enable a starter set of Foldkit rules:

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
    "foldkit/message-binding-matches-tag": "error",
    "foldkit/got-prefix-requires-submodel-payload": "error",
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

The rest of the plugin's rules are opt-in. Enable one by adding `"foldkit/<rule-name>": "error"` to the `rules` block. The complete rule set is grouped by the part of the architecture it protects below.

## Message Naming and Construction

### foldkit/no-noop-message

Rejects catch-all Messages that make update branches and traces less meaningful. Name the event that happened instead.

```
import { m } from 'foldkit/message'

// ❌ Bad
const NoOp = m('NoOp')

// ✅ Good
const ClickedSave = m('ClickedSave')
```

### foldkit/message-binding-matches-tag

Keeps a Message binding and its m() tag identical, so renames do not leave misleading traces behind.

```
import { m } from 'foldkit/message'

// ❌ Bad
const ClickedSave = m('ClickedSubmit')

// ✅ Good
const ClickedSubmit = m('ClickedSubmit')
```

### foldkit/no-empty-object-tagged-call

Catches empty-object calls to no-field Message constructors. A no-field Message should be called with no arguments.

```
import { m } from 'foldkit/message'

const ClickedSave = m('ClickedSave')

// ❌ Bad
const badMessage = ClickedSave({})

// ✅ Good
const goodMessage = ClickedSave()
```

### foldkit/prefer-callable-message-constructor

Prevents constructing Messages by typing or casting object literals. Use the callable Schema constructor instead.

```
import { Schema as S } from 'effect'
import { m } from 'foldkit/message'

const ClickedSave = m('ClickedSave')
const Message = S.Union([ClickedSave])
type Message = typeof Message.Type

// ❌ Bad
const badMessage: Message = {
  _tag: 'ClickedSave',
}

// ✅ Good
const goodMessage = ClickedSave()
```

## Command Shape

### foldkit/command-binding-matches-name

Keeps a Command binding name in sync with the name passed to Command.define.

```
import { Effect } from 'effect'
import { Command } from 'foldkit'
import { m } from 'foldkit/message'

const CompletedFetchUser = m('CompletedFetchUser')

// ❌ Bad
const SaveUser = Command.define('FetchUser', {
  messages: [CompletedFetchUser],
  execute: Effect.succeed(CompletedFetchUser()),
})

// ✅ Good
const FetchUser = Command.define('FetchUser', {
  messages: [CompletedFetchUser],
  execute: Effect.succeed(CompletedFetchUser()),
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

## Model Updates

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

## Routing

### foldkit/no-hardcoded-route-strings

Rejects hardcoded path and URL strings passed to link and navigation helpers. Build them from the Route module so they stay in sync with the routes.

```
import type { HtmlBuilder } from 'foldkit/html'

import { tasksRouter } from '../route'

// ❌ Bad
// A hardcoded path rots when the route changes and bypasses the Route module.
const badLink = (h: HtmlBuilder<Message>) => h.a([h.Href('/tasks')], ['Tasks'])

// ✅ Good
// Build the href from the Router so it stays in sync with the route.
const goodLink = (h: HtmlBuilder<Message>) =>
  h.a([h.Href(tasksRouter())], ['Tasks'])
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

### foldkit/no-module-level-mutable-state

Rejects module-level let and var bindings, which hold state outside the Model. Move the data into the Model, or scope a live handle to a lifecycle primitive like Mount or ManagedResource.

```
import { Schema as S } from 'effect'

// ❌ Bad
let requestCount = 0

// ✅ Good
export const Model = S.Struct({
  requestCount: S.Number,
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

### foldkit/got-submodel-message-name

Requires wrapper Messages around Submodel Messages to use the Got*Message convention.

```
import { m } from 'foldkit/message'

import * as Child from './child'

// ❌ Bad
const ChildChanged = m('ChildChanged', {
  message: Child.Message,
})

// ✅ Good
const GotChildMessage = m('GotChildMessage', {
  message: Child.Message,
})
```

### foldkit/got-prefix-requires-submodel-payload

Reserves the Got* prefix for Submodel wrappers. Any Got-prefixed Message must include a child Message payload named message.

```
import { Schema as S } from 'effect'
import { m } from 'foldkit/message'

import * as Child from './child'

{
  // ❌ Bad: Got is reserved for Submodel wrappers.
  const GotWeather = m('GotWeather', {
    temperature: S.Number,
  })
}

{
  // ✅ Good: use a name that does not start with Got for Command results.
  const ReceivedWeather = m('ReceivedWeather', {
    temperature: S.Number,
  })
}

{
  // ❌ Bad: Got-prefixed wrappers must carry child Messages.
  const GotChildMessage = m('GotChildMessage', {
    id: S.String,
  })
}

{
  // ✅ Good: Got wraps a child Message.
  const GotChildMessage = m('GotChildMessage', {
    id: S.String,
    message: Child.Message,
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
import { Schema as S } from 'effect'
import { m } from 'foldkit/message'

// ❌ Bad
// A Got wrapper carries the child Message plus routing context only. Extra
// payload like timestamp belongs on the child Message or a parent Message.
const GotSettingsMessage = m('GotSettingsMessage', {
  message: Settings.Message,
  timestamp: S.Number,
})

// ✅ Good
// message plus routing keys (id, or keys ending in Id) only.
const GotCounterMessage = m('GotCounterMessage', {
  id: S.String,
  message: Counter.Message,
})
```

### foldkit/no-child-message-construction-in-root

Rejects constructing a child Message variant from outside the child. Call a child-exported helper and route its output through the wrapper.

```
// ❌ Bad
// The root reaches into the child Message namespace to build a child Message.
const badRouting = () =>
  GotChildMessage({ message: Child.Message.ClickedSave() })

// ✅ Good
// The child exports a helper; the root routes its output through the wrapper.
const goodRouting = () => GotChildMessage({ message: Child.clickedSave() })
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

Requires a Mount factory to read or write its element. If it never touches the element, the cause was misidentified and Mount is the wrong primitive.

```
import { Effect } from 'effect'
import { Mount } from 'foldkit'

// ❌ Bad
// The factory never reads its element, so Mount is the wrong primitive here.
const MountAnalytics = Mount.define(
  'MountAnalytics',
  {},
  CompletedMountAnalytics,
)(() => () => Effect.sync(() => startAnalytics()))

// ✅ Good
// The factory reads its element to wire the observer.
const MountResize = Mount.define(
  'MountResize',
  {},
  CompletedMountResize,
)(() => element => Effect.sync(() => resizeObserver.observe(element)))
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
