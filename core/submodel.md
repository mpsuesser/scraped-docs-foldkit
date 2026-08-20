---
url: https://foldkit.dev/core/submodel
title: "Submodel"
description: "Split a large application into child state machines while preserving parent-to-child Message flow. Covers Update.foldChild, h.submodel, OutMessages, reflection, testing, and DevTools."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

## When to Create a Submodel

Use a Submodel when part of the application owns a state machine, not merely a section of markup. A Submodel has its own Model, Message, update, view, and Commands. Its parent stores the child Model, routes child Messages, and decides what to do with facts that cross the boundary.

Two needs commonly create that boundary:

- **Encapsulation:** A reusable component owns interaction state, keyboard behavior, focus, or accessibility wiring that consumers should not manage. Stateful [Foldkit UI Submodels](https://foldkit.dev/ui/overview) such as `Dialog`, `Menu`, and `Listbox` use this shape.
- **Decomposition:** A feature area such as Settings or Profile owns enough state and Messages that separating its update loop makes the application easier to navigate.

Both use the same contract. Internal state stays behind the boundary, and values crossing it have named roles. A `Listbox` can report that an item was selected without exposing its highlight index or focus bookkeeping. A feature Submodel can [read shared parent state](#reading-parent-state) and [surface domain facts](#surfacing-facts) without taking ownership of the whole application.

The word "boundary"

Each `h.submodel` call creates a runtime boundary identified by `slotId`. When the child dispatches a Message, `toParentMessage` wraps it in the parent's Message type. Nested Submodels repeat that process at every level until the Message reaches the root update.

The restaurant analogy

A Submodel is one station in the restaurant. The station owns its work and internal order state. The head waiter routes work to it and responds to the facts it reports without directing each step inside the station.

When a view function is enough

If a section only renders parent state and owns no Messages or update logic, make it a view function. A reusable card that receives a title and content does not need a Submodel.

## The Child Submodel

A child Submodel does not know which parent embeds it. This Settings Submodel owns its state and handles its Messages without importing the root Model or Message.

```
// page/settings.ts
import { Match as M, Schema as S } from 'effect'
import { Command } from 'foldkit'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

// MODEL

export const Theme = S.Literals(['Light', 'Dark', 'System'])
export type Theme = typeof Theme.Type

export const FontSize = S.Literals(['Small', 'Medium', 'Large'])
export type FontSize = typeof FontSize.Type

export const Model = S.Struct({
  theme: Theme,
  fontSize: FontSize,
  notificationsEnabled: S.Boolean,
})

export type Model = typeof Model.Type

// MESSAGE

export const ChangedTheme = m('ChangedTheme', { theme: Theme })
export const ChangedFontSize = m('ChangedFontSize', { fontSize: FontSize })
export const ToggledNotifications = m('ToggledNotifications')

export const Message = S.Union([
  ChangedTheme,
  ChangedFontSize,
  ToggledNotifications,
])
export type Message = typeof Message.Type

// UPDATE

export const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    M.withReturnType<
      readonly [Model, ReadonlyArray<Command.Command<Message>>]
    >(),
    M.tagsExhaustive({
      ChangedTheme: ({ theme }) => [evo(model, { theme: () => theme }), []],
      ChangedFontSize: ({ fontSize }) => [
        evo(model, { fontSize: () => fontSize }),
        [],
      ],
      ToggledNotifications: () => [
        evo(model, { notificationsEnabled: enabled => !enabled }),
        [],
      ],
    }),
  )
```

## Embedding the Submodel

The parent has three jobs: embed the child’s Model, wrap its Messages, and delegate to its update.

### Embedding the Model

The child’s Model becomes a field in the parent’s Model:

```
import { Schema as S } from 'effect'

import * as Settings from './page/settings'

export const Model = S.Struct({
  username: S.String,
  settings: Settings.Model,
})

export type Model = typeof Model.Type
```

### Never Bypass the Child’s Update

The parent stores the child Model, but the child still owns it. Do not use [evo](https://foldkit.dev/best-practices/immutability#immutable-updates) to change fields inside that slice from the parent.

```
// ❌ Don't reach into the child's Model from the parent's update.
// This bypasses Settings.update, so DevTools never sees the change,
// and any invariant Settings.update was enforcing is silently violated.
ClickedResetSettings: () => [
  evo(model, {
    settings: settings => evo(settings, { theme: () => 'Light' }),
  }),
  [],
]
```

For a parent-initiated change, export a helper from the child and fold that helper with `Update.foldChild`. The parent can call `Settings.setTheme` without importing the internal `ChangedTheme` constructor.

```
// CHILD

export const setTheme = (model: Model, theme: Theme) =>
  update(model, ChangedTheme({ theme }))

// PARENT UPDATE

const foldSettingsTheme = Update.foldChild({
  update: Settings.setTheme,
  read: (model: Model) => Option.some(model.settings),
  write: (model, nextSettings) => evo(model, { settings: () => nextSettings }),
  toParentMessage: message => GotSettingsMessage({ message }),
})

ClickedResetSettings: () => foldSettingsTheme(model, 'Light')
```

Stateful Foldkit UI components expose the same kind of entry point. For example: `Popover.close` and a Listbox instance's `selectItem` helper run the component's update without exposing its internal Message constructors.

Messages produced by the child take the regular wrapper path described below. Helpers cover the opposite direction, when the parent needs to initiate a child transition.

Bypassing update creates three problems:

- The transition bypasses invariants enforced by the child's update.
- Commands and OutMessages that the child's update would return are skipped.
- A later change to the child Model can leave the parent writing obsolete fields.

### Wrapping Messages

Every Message eventually reaches the root update. Each parent therefore declares a wrapper Message for the child Message type. Name it with the `Got*Message` convention, such as `GotSettingsMessage`.

```
import { Schema as S } from 'effect'
import { m } from 'foldkit/message'

import * as Settings from './page/settings'

export const GotSettingsMessage = m('GotSettingsMessage', {
  message: Settings.Message,
})

export const Message = S.Union([GotSettingsMessage])
export type Message = typeof Message.Type
```

DevTools expects this naming convention

The Foldkit DevTools use the `Got*Message` pattern to power the Submodel filter, which lets you scope DevTools Messages to a chosen Submodel. If your wrapper Messages don’t follow this naming convention, they won’t appear in the list of filterable Submodel Messages.

A wrapper carries routing information only. It holds the child `message` and, for repeated instances, an identifier such as `entryId`. Domain data belongs inside the child Message that uses it.

### Folding Update with Update.foldChild

`Update.foldChild` is the update half of embedding a Submodel. Its configuration tells Foldkit how to:

- run the child update;
- read and write the child Model;
- wrap result Messages from child Commands.

The resulting fold reads the child, runs its update, writes it back, and lifts its Commands through `toParentMessage`.

```
import { Match as M, Option } from 'effect'
import { Command, Update } from 'foldkit'
import { evo } from 'foldkit/struct'

const foldSettings = Update.foldChild({
  update: Settings.update,
  read: (model: Model) => Option.some(model.settings),
  write: (model, nextSettings) => evo(model, { settings: () => nextSettings }),
  toParentMessage: message => GotSettingsMessage({ message }),
})

export const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    M.tagsExhaustive({
      GotSettingsMessage: ({ message }) => foldSettings(model, message),
    }),
  )
```

`read` returns an `Option` because a routed page or keyed child may no longer exist when its Message arrives. `None` makes the fold a no-op. An always-present child returns `Option.some(model.settings)`.

The fold is dual. `foldSettings(model, message)` runs it immediately. `foldSettings(message)` returns an `Update.Step` for `Update.combine`. Close over per-dispatch context in the `update` field, and apply route gates before calling the fold.

Use `Update.foldChildStep` for an entry point that takes only the child Model, such as `Dialog.close`. It accepts the same boundary fields and returns an `Update.Step` directly.

### Wiring the View with h.submodel

Define the child view with `Submodel.defineView<Model, Message>`. It receives the child Model and a builder for child Messages.

`defineView` brands the function with its child Model and Message types. The parent can then embed it without repeating those types, and handlers inside the child accept only child Messages.

```
// page/settings.ts
import { Submodel } from 'foldkit'

import {
  ChangedFontSize,
  ChangedTheme,
  type Message,
  ToggledNotifications,
} from './message'
import type { Model } from './model'

// The Submodel exports a view defined with Submodel.defineView<Model, Message>.
// The view takes the child's Model and the child's typed builder \`h\`, which
// the runtime supplies, and produces Html. The <Model, Message> type arguments
// brand the view with its Message type so the parent can lift each emitted
// Message into its wrapper Message when it embeds the Submodel, and they type
// \`h\`: its handlers accept exactly the Messages this Submodel dispatches.
export const view = Submodel.defineView<Model, Message>((model, h) =>
  h.div(
    [h.Class('flex flex-col gap-4')],
    [
      h.h2([h.Class('text-xl font-bold')], ['Settings']),
      h.div(
        [h.Class('flex gap-2')],
        [
          h.button([h.OnClick(ChangedTheme({ theme: 'Light' }))], ['Light']),
          h.button([h.OnClick(ChangedTheme({ theme: 'Dark' }))], ['Dark']),
          h.button([h.OnClick(ChangedTheme({ theme: 'System' }))], ['System']),
        ],
      ),
      h.div(
        [h.Class('flex gap-2')],
        [
          h.button(
            [h.OnClick(ChangedFontSize({ fontSize: 'Small' }))],
            ['Small'],
          ),
          h.button(
            [h.OnClick(ChangedFontSize({ fontSize: 'Medium' }))],
            ['Medium'],
          ),
          h.button(
            [h.OnClick(ChangedFontSize({ fontSize: 'Large' }))],
            ['Large'],
          ),
        ],
      ),
      h.button(
        [h.OnClick(ToggledNotifications())],
        [
          model.notificationsEnabled
            ? 'Disable notifications'
            : 'Enable notifications',
        ],
      ),
    ],
  ),
)
```

The parent passes four required fields to `h.submodel`:

- `slotId` identifies this position under the current boundary.
- `model` supplies the child Model.
- `view` supplies the branded child view.
- `toParentMessage` wraps a child Message for the parent.

```
// main.ts (parent)
import type { Document, HtmlBuilder } from 'foldkit/html'

import { GotSettingsMessage, type Message } from './message'
import type { Model } from './model'
import * as Settings from './page/settings'

// The parent embeds the child via h.submodel. The slotId is unique within
// the parent's view, view is the child's exported view function, model is the
// embedded slice, and toParentMessage lifts every Message the child emits
// into the parent's GotSettingsMessage envelope. The child stays decoupled
// from this parent; the same Settings.view embeds under any parent that
// supplies a compatible wrapping.
export const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: 'My App',
  body: h.div(
    [h.Class('min-h-screen bg-gray-50')],
    [
      h.submodel({
        slotId: 'settings',
        model: model.settings,
        view: Settings.view,
        toParentMessage: message => GotSettingsMessage({ message }),
      }),
    ],
  ),
})
```

Any parent with the required Model and wrapper can embed the same `Settings.view`.

### Per-render View Inputs

Use `ViewInputs` for parent-owned data the child needs only while rendering. A Listbox may need items and an item renderer, while its Model owns highlight and selection state.

Pass `ViewInputs` as the third type parameter to `defineView`. The view then receives `(model, viewInputs, h)`.

```
// page/commandMenu.ts
import { Array } from 'effect'
import { Submodel } from 'foldkit'
import type { Html } from 'foldkit/html'

import { ClosedMenu, type Message, OpenedMenu, SelectedItem } from './message'
import type { Model } from './model'

// The third type parameter to defineView is \`ViewInputs\`: per-render
// data the parent passes alongside the model. Here, the parent supplies
// the trigger content and the items; the child supplies the open/closed
// state and the selection behavior. With ViewInputs present, the builder
// \`h\` moves to third position.
export type ViewInputs = Readonly<{
  buttonLabel: Html
  items: ReadonlyArray<string>
}>

export const view = Submodel.defineView<Model, Message, ViewInputs>(
  (model, viewInputs, h) => {
    const toggleMessage = model.isOpen ? ClosedMenu() : OpenedMenu()

    return h.div(
      [],
      [
        h.button([h.OnClick(toggleMessage)], [viewInputs.buttonLabel]),
        ...(model.isOpen
          ? [
              h.div(
                [h.Role('menu')],
                Array.map(viewInputs.items, (label, index) =>
                  h.keyed('div')(
                    label,
                    [
                      h.Role('menuitem'),
                      h.OnClick(SelectedItem({ index, label })),
                    ],
                    [label],
                  ),
                ),
              ),
            ]
          : []),
      ],
    )
  },
)
```

The parent supplies `viewInputs` at the embed site.

```
// main.ts (parent)
import type { Document, HtmlBuilder } from 'foldkit/html'

import { GotCommandMenuMessage, type Message } from './message'
import type { Model } from './model'
import * as CommandMenu from './page/commandMenu'

const MENU_ITEMS: ReadonlyArray<string> = ['Open', 'Rename', 'Archive']

// The parent passes \`viewInputs\` alongside model/view/toParentMessage.
// \`buttonLabel\` and \`items\` are configuration the parent owns; the child
// slots them into its open/closed widget. The child has no idea what the
// items mean. Only that they exist.
export const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: 'My App',
  body: h.div(
    [],
    [
      h.submodel({
        slotId: 'command-menu',
        model: model.commandMenu,
        view: CommandMenu.view,
        viewInputs: {
          buttonLabel: h.span([], ['Actions']),
          items: MENU_ITEMS,
        },
        toParentMessage: message => GotCommandMenuMessage({ message }),
      }),
    ],
  ),
})
```

Keep state in the child Model and per-render configuration in `viewInputs`. The child changes its Model through update. The parent rebuilds `viewInputs` on each render.

A top-level slot callback such as `toView` lets the parent choose markup while the child supplies state and attributes. Foldkit runs top-level `viewInputs` functions in the parent's boundary. A handler the parent builds inside the callback therefore dispatches a parent Message. [childAttributes](#child-attributes) handles the opposite case, when child-owned attributes cross into that markup.

Keep slot callbacks at the top level

Functions nested inside an object or array in `viewInputs` throw when Foldkit builds the view. The error names the path, such as `viewInputs.config.onSubmit`. Move the function to the top level so Foldkit can run it in the parent's boundary.

## Boundary Id and Model Identity

`slotId` identifies a rendered position, not a Model value. Every `h.submodel` call under one parent boundary needs a distinct `slotId`, even when two positions render the same child Model.

For fixed positions, name the position: `'desktop-sidebar'` and `'mobile-sidebar'`. For a list, use the stable item identifier because each item occupies its own position.

Foldkit throws while building the view when sibling boundaries reuse a `slotId`.

## Multiple Instances

A parent can hold a fixed or dynamic number of child instances.

For a fixed set, give each child its own Model field and `slotId`. For a dynamic set, store the children in an array. Use the same stable identifier for the row key, `slotId`, and wrapper Message.

```
import { Array, Option } from 'effect'
import { Update } from 'foldkit'
import type { Html, HtmlBuilder } from 'foldkit/html'
import { evo } from 'foldkit/struct'

import { Applicant } from './applicant'
import { GotApplicantMessage, type Message } from './message'
import type { Model } from './model'

export const view = (model: Model, h: HtmlBuilder<Message>): Html =>
  h.ul(
    [h.Class('flex flex-col gap-4')],
    Array.map(model.applicants, applicant =>
      h.keyed('li')(
        applicant.id,
        [],
        [
          h.submodel({
            slotId: applicant.id,
            model: applicant.entry,
            view: Applicant.view,
            toParentMessage: message =>
              GotApplicantMessage({ entryId: applicant.id, message }),
          }),
        ],
      ),
    ),
  )

const foldApplicant = (entryId: string) =>
  Update.foldChild({
    update: Applicant.update,
    read: (model: Model) =>
      Option.map(
        Array.findFirst(
          model.applicants,
          applicant => applicant.id === entryId,
        ),
        applicant => applicant.entry,
      ),
    write: (model, nextEntry) =>
      evo(model, {
        applicants: Array.map(applicant =>
          applicant.id === entryId
            ? evo(applicant, { entry: () => nextEntry })
            : applicant,
        ),
      }),
    toParentMessage: message => GotApplicantMessage({ entryId, message }),
  })

GotApplicantMessage: ({ entryId, message }) =>
  foldApplicant(entryId)(model, message)
```

`foldApplicant(entryId)` reads and writes only the matching child. When the child no longer exists, `read` returns `None` and a late Message becomes a no-op. The [job-application example](https://foldkit.dev/example-apps/job-application) uses this shape for repeated education and work-history entries.

Start with an array. If profiling shows that finding and replacing a child is expensive, use a `HashMap` keyed by the same identifier. `Update.foldChild` still works because `HashMap.get` already returns an `Option`.

## Memoization Across Submodel Boundaries

By default, a parent render runs each child view again. If profiling finds repeated work in a long list or expensive child view, place the embed site behind `createKeyedLazy` from `foldkit/html`.

Foldkit keeps the boundary registration alive across cache hits and removes it when the VNode leaves the tree. Key the lazy view with the same stable identifier used by `slotId`.

The [View Memoization](https://foldkit.dev/core/view-memoization) page covers cache identity, limits, and measurement.

## Reading Parent State

Do not copy shared parent state into a child Model merely so the child can read it. Choose the boundary based on when the child needs the value:

- When a child view needs to render state that lives in the parent Model, thread it through `viewInputs` on `h.submodel`.
- When a child update needs context from the parent, add a third `context` argument to the child’s update.

The parent remains the single source of truth in both cases.

### Passing Parent State to a Child Submodel’s view

Pass parent state through `viewInputs` when the child needs it for rendering. The parent supplies the current value on every render.

```
import { Submodel } from 'foldkit'

import type { User } from '../user'
import type { Message } from './message'
import type { Model } from './model'

// The child declares the parent state it needs via the third type
// parameter on \`Submodel.defineView\`. The view receives it as
// \`viewInputs\` alongside \`model\`, before the builder \`h\`.
type ViewInputs = Readonly<{
  currentUser: User
}>

export const view = Submodel.defineView<Model, Message, ViewInputs>(
  (model, { currentUser }, h) =>
    h.div(
      [],
      [
        h.h2([], [\`Settings for ${currentUser.name}\`]),
        // ...rest of the Settings UI driven by \`model\`
      ],
    ),
)

// Inside the parent's view, slice currentUser out of the parent Model
// and pass it through viewInputs. Rebuilt every render, so the child always
// sees the current value:
h.submodel({
  slotId: 'settings',
  model: model.settings,
  view: Settings.view,
  viewInputs: {
    currentUser: model.currentUser,
  },
  toParentMessage: message => GotSettingsMessage({ message }),
})
```

### Providing Parent State to a Child Submodel’s update

Add a third `context` argument when child update needs the current parent value while processing a Message. Close over that value when constructing the fold.

```
import { Match as M, Option } from 'effect'
import { type Command, Update } from 'foldkit'
import { evo } from 'foldkit/struct'

import { GotSettingsMessage } from '../../message'
import type { Model as AppModel } from '../../model'
import type { User } from '../user'
import { PersistSettings, type Message as SettingsMessage } from './message'
import type { Model as SettingsModel } from './model'

type Context = Readonly<{
  currentUser: User
}>

type UpdateReturn = readonly [
  SettingsModel,
  ReadonlyArray<Command.Command<SettingsMessage>>,
]

export const update = (
  model: SettingsModel,
  message: SettingsMessage,
  context: Context,
): UpdateReturn =>
  M.value(message).pipe(
    M.withReturnType<UpdateReturn>(),
    M.tagsExhaustive({
      ChangedTheme: ({ theme }) => [
        evo(model, { theme: () => theme }),
        [PersistSettings({ userId: context.currentUser.id, theme })],
      ],
      // ...other arms
    }),
  )

// PARENT UPDATE

const foldSettings = (currentUser: User) =>
  Update.foldChild({
    update: (settings: SettingsModel, message: SettingsMessage) =>
      update(settings, message, { currentUser }),
    read: (model: AppModel) => Option.some(model.settings),
    write: (model, nextSettings) =>
      evo(model, { settings: () => nextSettings }),
    toParentMessage: message => GotSettingsMessage({ message }),
  })

GotSettingsMessage: ({ message }) =>
  foldSettings(model.currentUser)(model, message)
```

The update stays pure because the context is an explicit input. Constructing `foldSettings(model.currentUser)` for each dispatch gives the child the current user without storing a second copy.

Context does not notify the child when a value changes. If the child must react to that change, expose an `inform*` helper and fold it from the parent handler that observed the change. See [Informing Submodels](https://foldkit.dev/patterns/informing-submodels).

## Surfacing Facts to the Parent

Wrapper Messages route child work back into the child update. An OutMessage reports a fact the parent may need to act on, such as a committed date, selected tab, or completed login.

The child update returns `Option<OutMessage>` as a third tuple element. The child describes what happened, and the parent decides the consequence. A Login Submodel can emit `SucceededLogin` without knowing how the root stores a session or changes the URL.

### Defining OutMessages

Define OutMessages beside the child Message. Name them as past-tense facts: `SucceededLogin`, not `TransitionToLoggedIn`; `RequestedLogout`, not `DoLogout`.

```
import { Schema as S } from 'effect'
import { m } from 'foldkit/message'

// MESSAGE

export const SubmittedLoginForm = m('SubmittedLoginForm')
export const SucceededAuthenticate = m('SucceededAuthenticate', {
  sessionId: S.String,
})

export const Message = S.Union([SubmittedLoginForm, SucceededAuthenticate])
export type Message = typeof Message.Type

// OUT MESSAGE

export const SucceededLogin = m('SucceededLogin', {
  sessionId: S.String,
})

export const OutMessage = S.Union([SucceededLogin])
export type OutMessage = typeof OutMessage.Type
```

### Emitting from the Child

The child update returns its Model, Commands, and an `Option<OutMessage>`. Most branches return `Option.none()`. A branch returns `Option.some(...)` only when it has a fact to surface.

```
import { Match as M, Option } from 'effect'
import { Command } from 'foldkit'

export const update = (
  model: Model,
  message: Message,
): readonly [
  Model,
  ReadonlyArray<Command.Command<Message>>,
  Option.Option<OutMessage>,
] =>
  M.value(message).pipe(
    M.tagsExhaustive({
      SubmittedLoginForm: () => [
        model,
        [Authenticate(model.email, model.password)],
        Option.none(),
      ],
      SucceededAuthenticate: ({ sessionId }) => [
        model,
        [],
        Option.some(SucceededLogin({ sessionId })),
      ],
    }),
  )
```

`SubmittedLoginForm` starts authentication but has no result to report. `SucceededAuthenticate` emits `SucceededLogin({ sessionId })` after the Command completes.

### Handling in the Parent

Handle the OutMessage through `foldOutMessage` on [Update.foldChild](#fold-child). Bind the fold as a standalone `fold<Child>OutMessage` value and match on every OutMessage tag. The returned `Update.Step` receives the parent Model after the updated child has been written back.

```
import { Match as M, Option } from 'effect'
import { Command, Update } from 'foldkit'
import { evo } from 'foldkit/struct'

const foldLoginOutMessage = M.type<Login.OutMessage>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    SucceededLogin:
      ({ sessionId }) =>
      () => [LoggedIn({ sessionId }), [SaveSession(sessionId)]],
  }),
)

const foldLogin = Update.foldChild({
  update: Login.update,
  read: (model: Model) => Option.some(model.login),
  write: (model, nextLogin) => evo(model, { login: () => nextLogin }),
  toParentMessage: message => GotLoginMessage({ message }),
  foldOutMessage: foldLoginOutMessage,
})

export const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    M.tagsExhaustive({
      GotLoginMessage: ({ message }) => foldLogin(model, message),
    }),
  )
```

The fold appends the Step's Commands after the child's lifted Commands. If the Step returns a child Command, use `liftCommand` or `liftCommands` from `Update.FoldContext`. The lifter wraps the Command's result Message with the same `toParentMessage` used by the child fold.

In this example, only the parent knows the redirect Route for `Login.SendMagicLink`. The child emits `RequestedMagicLink`, and the parent fills in the Route while keeping the Command result inside the Login boundary.

```
import { Match as M } from 'effect'
import { Update } from 'foldkit'

const foldLoginOutMessage = (
  outMessage: Login.OutMessage,
  { liftCommand }: Update.FoldContext<Login.Message, Message>,
) =>
  M.value(outMessage).pipe(
    M.withReturnType<Update.Step<Model, Message>>(),
    M.tagsExhaustive({
      RequestedMagicLink:
        ({ email }) =>
        model => [
          model,
          [
            liftCommand(
              Login.SendMagicLink({ email, redirectRoute: model.route }),
            ),
          ],
        ],
    }),
  )
```

[Update.foldChildStep](#fold-child) supplies the same fold context for no-argument child entry points.

A parent that is itself a Submodel adds `toParentOutMessage`. Match the child OutMessage and return `Option.some(parentOutMessage)` when the fact should continue upward, or `Option.none()` when it stops at that level. The [Auth example](https://foldkit.dev/example-apps/auth) carries a successful login through two Submodel levels to the root.

## Reflecting External State

OutMessages move facts from child to parent. A `reflect*` helper handles the inbound direction, when an external source such as the URL, restored storage, or a sibling field requires the child to conform.

A `reflect*` helper returns the child Model directly. It does not return Commands or an OutMessage. The external value is already the source of truth, so emitting it back could create a write loop.

Define reflect helpers with `Function.dual` so they work point-free in [evo](https://foldkit.dev/best-practices/immutability#immutable-updates). Here the URL owns the price range, and the parent reflects that range onto a Slider.

```
ChangedUrl: ({ route }) => [
  evo(model, {
    // The URL owns the price bounds, so reflect them onto the Slider. reflectRange
    // returns Model (point-free in evo) and emits nothing, so it can't echo the
    // route back and loop.
    priceSlider: Slider.reflectRange({
      min: route.minPrice,
      max: route.maxPrice,
    }),
  }),
  [],
]
```

Only the owner calls a child's `reflect*` helper. User interactions still go through the child update and may emit OutMessages.

External means outside this child boundary, not outside the application. For example: when a start date changes, the parent can call `reflectMinDate` on the end-date picker. The end-date child did not cause the change, but it must obey the new constraint.

Foldkit UI uses domain verbs such as `selectItem` and `selectDate` for user choices that can emit. Silent inbound setters use the `reflect*` prefix, including Calendar and DatePicker constraints and Slider's `reflectRange`.

## Which Boundary a Handler Dispatches Through

An element dispatches through the boundary where it is built. The builder carries that boundary, which is why every view receives `h` as an argument.

Inside a Submodel there are two frames, with opposite defaults:

| Where you build the element | Dispatches through | To change it |
| --- | --- | --- |
| The child’s own view body | the **child’s** boundary | have the parent supply the element through a `viewInputs` slot callback |
| A slot callback the parent passed in `viewInputs` | the **parent’s** boundary | [`childAttributes`](#child-attributes) binds it back to the child |

The child view normally builds child handlers. A slot callback normally builds parent handlers.

A shared helper may need the parent builder. For example: a copy button inside a documentation Submodel dispatches an app-level Message. Let the parent build that renderer and pass it through a top-level `viewInputs` callback.

```
// view/docs.ts (inside the parent's view, with its builder \`h\` in scope)
h.submodel({
  slotId: 'coming-from-react',
  model: model.comingFromReact,
  view: Page.ComingFromReact.view,
  viewInputs: {
    // Both close over the parent's builder, so the app-level Messages they
    // dispatch reach update unwrapped however deep the child renders them.
    renderCopyButton: defaultRenderCopyButton(model.copiedSnippets, h),
    renderHeadingLink: defaultRenderHeadingLink(h),
  },
  toParentMessage: message => GotComingFromReactMessage({ message }),
})
```

The callback runs in the parent's boundary, so its Message reaches the parent update without a child wrapper.

Thread `h` through view functions. Never store it in module state. A stored builder can outlive its render boundary and fail when a handler dispatches.

## childAttributes

Some Submodels let the parent render the DOM while the child supplies behavior. Tooltip, Dialog, Popover, and selection Submodels publish attribute bundles through a slot callback. The parent chooses the elements and styling.

Those child-built handlers must still dispatch through the child boundary after the parent spreads them onto an element. `childAttributes` preserves that routing.

### The Problem

Imagine a CommandMenu builds `h.OnClick(OpenedMenu())` but asks the parent to render the button. The click must produce `GotCommandMenuMessage({ message: OpenedMenu() })` for the parent fold.

Without the child dispatcher, the parent-built button would send raw `OpenedMenu()` to the parent update. The child would never receive its own Message.

### How It Works

`childAttributes` brands each attribute with the current child dispatcher. When the parent builds an element, its constructor uses the branded dispatcher for those attributes instead of the parent dispatcher.

The child publishes branded attribute groups with the state the slot needs.

```
// page/commandMenu.ts
import { Array, Option } from 'effect'
import { Submodel } from 'foldkit'
import { type ChildAttribute, type Html, childAttributes } from 'foldkit/html'

import {
  ClosedMenu,
  HoveredItem,
  type Message,
  OpenedMenu,
  SelectedItem,
} from './message'
import { type Model, buttonId, itemId, menuId } from './model'

type SlotItem = Readonly<{
  id: string
  label: string
  isActive: boolean
  attributes: ReadonlyArray<ChildAttribute>
}>

type Slot = Readonly<{
  isOpen: boolean
  buttonAttributes: ReadonlyArray<ChildAttribute>
  menuAttributes: ReadonlyArray<ChildAttribute>
  items: ReadonlyArray<SlotItem>
}>

type ViewInputs = Readonly<{
  items: ReadonlyArray<string>
  toView: (slot: Slot) => Html
}>

export const view = Submodel.defineView<Model, Message, ViewInputs>(
  (model, viewInputs, h) => {
    const toggleMessage = model.isOpen ? ClosedMenu() : OpenedMenu()

    const toSlotItem = (label: string, index: number): SlotItem => {
      const id = itemId(model.id, label)
      const isActive = Option.contains(model.maybeActiveItemIndex, index)

      return {
        id,
        label,
        isActive,
        attributes: childAttributes([
          h.Id(id),
          h.Role('menuitem'),
          ...(isActive ? [h.DataAttribute('active', '')] : []),
          h.OnMouseEnter(HoveredItem({ index })),
          h.OnClick(SelectedItem({ index, label })),
        ]),
      }
    }

    return viewInputs.toView({
      isOpen: model.isOpen,
      buttonAttributes: childAttributes([
        h.Id(buttonId(model.id)),
        h.AriaHasPopup('menu'),
        h.AriaExpanded(model.isOpen),
        h.AriaControls(menuId(model.id)),
        h.OnClick(toggleMessage),
      ]),
      menuAttributes: childAttributes([h.Id(menuId(model.id)), h.Role('menu')]),
      items: Array.map(viewInputs.items, toSlotItem),
    })
  },
)
```

The parent consumes that slot data without reading the child Model.

```
// main.ts (parent)
import { Array } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'

import { GotCommandMenuMessage, type Message } from './message'
import type { Model } from './model'
import * as CommandMenu from './page/commandMenu'

const MENU_ITEMS: ReadonlyArray<string> = ['Open', 'Rename', 'Archive']

export const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'command-menu',
    model: model.commandMenu,
    view: CommandMenu.view,
    viewInputs: {
      items: MENU_ITEMS,
      toView: slot =>
        h.div(
          [],
          [
            h.button(
              [...slot.buttonAttributes, h.Class('px-3 py-2 rounded')],
              ['Actions'],
            ),
            ...(slot.isOpen
              ? [
                  h.div(
                    [...slot.menuAttributes, h.Class('mt-2 p-1 bg-gray-50')],
                    Array.map(slot.items, item =>
                      h.keyed('div')(
                        item.id,
                        [
                          ...item.attributes,
                          ...(item.isActive ? [h.Class('bg-blue-50')] : []),
                        ],
                        [item.label],
                      ),
                    ),
                  ),
                ]
              : []),
          ],
        ),
    },
    toParentMessage: message => GotCommandMenuMessage({ message }),
  })
```

The child `OnClick` uses the carried dispatcher. Parent attributes such as `h.Class` behave normally.

### When to Reach For It

Consumers do not call `childAttributes`. Spread the bundle the Submodel provides.

Authors must wrap every child-owned attribute group before publishing it to a parent slot. Otherwise, handlers route through the parent boundary and skip the child update.

Render helpers don’t need this

Stateless helpers such as `Button` and controlled helpers such as `Checkbox` are not Submodels. Their Messages already belong to the consumer boundary, so they do not use `childAttributes`.

## Testing Submodels

Test child update with Story and child view with Scene. Both accept the same shapes at the child level that they accept at the root.

Use `expectOutMessage` for a child's OutMessage. For an update with context, close over the test context: `(model, message) => Settings.update(model, message, { currentUser })`.

Test the parent when the behavior crosses the boundary, such as routing a wrapper by id or folding an OutMessage. See [Testing](https://foldkit.dev/testing) for choosing the level.

## Debugging Submodels in DevTools

The `Got*Message` convention powers the Submodel filter in [Foldkit DevTools](https://foldkit.dev/core/devtools). Selecting a child scopes the timeline and Model diffs to that boundary.

Another wrapper name still routes correctly, but DevTools cannot discover it for the filter.

An OutMessage is processed within the parent fold. Any later parent Message appears as its own timeline entry.

## Common Pitfalls

Issues new Submodel users hit, and where to read about the fix:

- **Duplicate `slotId` during view construction:** Two sibling `h.submodel` calls identify the same position. See [Boundary Id and Model Identity](#boundary-id-and-model-identity).
- **Child Message reaches the root but nothing changes:** The parent Message union or update is missing the wrapper variant. See [Wrapping Messages](#wrapping-messages) and [Folding Update](#fold-child).
- **Wrapper missing from the DevTools filter:** The wrapper does not follow the `Got*Message` convention. See [Debugging Submodels](#debugging-in-devtools).
- **Child view sees stale parent state:** Parent state was copied into the child Model. Pass it through `viewInputs` instead. See [Reading Parent State](#reading-parent-state).
- **Child-owned handler uses the parent boundary:** The child published attributes without `childAttributes`. See [childAttributes](#child-attributes).
- **Error names a nested `viewInputs` path:** Move the nested callback to the top level of `viewInputs`.
- **A long child list rerenders slowly:** Profile it before adding `createKeyedLazy`. See [Memoization](#memoization).

## API Reference

### h.submodel

`h.submodel(config: SubmodelConfig<View>): Html`

Embeds a child Submodel under the current boundary. Creates a runtime boundary holding the embed site’s `slotId` and `toParentMessage`, dispatches Messages from inside the child through the wrap chain to the parent, and deregisters the boundary when the DOM node is destroyed. See [Wiring the View with h.submodel](#wiring-the-view) for usage; see [Boundary Id and Model Identity](#boundary-id-and-model-identity) for `slotId` semantics.

### SubmodelConfig

The configuration record passed to `h.submodel`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `slotId` | `string` | — | DOM-slot identity for this embed site under the current boundary. Must be distinct from every other h.submodel slotId under the same parent boundary. For lists, use a per-item id (row.id); for fixed slots, name by position. |
| `model` | `View extends SubmodelView<infer Model, ...> ? Model : never` | — | The child Submodel’s slice of the parent Model. Type is inferred from the branded view. |
| `view` | `SubmodelView<Model, Message, ViewInputs?>` | — | The child’s exported view, branded via Submodel.defineView so the embed site can infer the child’s Message type. |
| `viewInputs` | `ViewInputs \| undefined` | — | Optional per-render data threaded into the view’s second argument. Top-level functions are auto-wrapped to execute in the parent’s boundary; nested functions throw at view-build time. |
| `toParentMessage` | `(message: ChildMessage) => ParentMessage` | — | Lifts each child Message into the parent’s wrapper Message type, typically a closure over the Got\*Message constructor. |

### Submodel.defineView

`Submodel.defineView<Model, Message, ViewInputs = void>(fn): SubmodelView<Model, Message, ViewInputs>`

Brands a view function with its Message type so `h.submodel` can type-check the embed site without a per-call type argument, and types the builder the runtime passes the view as its last parameter. The `<Model, Message>` parameters are required at the definition site; `ViewInputs` is optional and, when supplied, makes the view take a second `viewInputs` argument before the builder. Also exported as `defineView` from `foldkit/html`.

### Submodel.View

`Submodel.View<Model, Message, ViewInputs = void> = (model, viewInputs, h) => Html`

The branded view type produced by `Submodel.defineView`. Without `ViewInputs` the shape is `(model, h) => Html`. Carries the child’s Message type at the type level. Consumers don’t usually annotate values with this type directly; the brand and `Parameters<View>` carry the inference at the embed site. Also exported as `SubmodelView` from `foldkit/html`.

### childAttributes

`childAttributes<Attribute>(attributes: ReadonlyArray<Attribute>): ReadonlyArray<ChildAttribute>`

Snapshots the Submodel’s dispatcher at publish time and brands each attribute so handlers route through the Submodel’s boundary when later spread into the consumer’s elements. Called inside a Submodel that publishes attribute bundles to a consumer’s `toView` slot. See [childAttributes](#child-attributes) for the full mechanism.

### ChildAttribute

`ChildAttribute` is the branded attribute type returned by `childAttributes`. Element constructors (`h.button`, `h.input`, etc.) accept `ChildAttribute` alongside ordinary `Attribute<Message>` values, using the carried dispatcher when present.

With Model, Messages, update, view, Commands, and Submodels in place, you have the full vocabulary for describing a Foldkit app. The next page covers the [Runtime](https://foldkit.dev/core/runtime): the engine that executes Commands, runs Subscriptions, manages Mount and ManagedResource lifecycles, and routes Messages back into update.
