---
url: https://foldkit.dev/ui/toast
title: "Toast"
description: "Stack of transient notifications anchored to a corner of the viewport with per-entry enter/leave animations and auto-dismiss."
access_date: 2026-08-11T02:16:28.996Z
current_date: 2026-08-11T02:16:28.996Z
---

## Toast

## Overview

A stack of transient notifications anchored to a corner of the viewport. Each entry has its own enter and leave animation, its own auto-dismiss timer, and its own hover-to-pause behavior. One container lives at the app root; entries are added dynamically via `Toast.show`.

Toast is parameterized on a user-provided payload schema. The component owns only lifecycle and a11y fields: id, variant (drives ARIA role), transition, dismiss timer, hover state. Everything else lives in your payload and is rendered by your `entryToView` callback. `Toast.make(PayloadSchema)` returns a module with `Model`, `show`, `view`, and the rest bound to your payload type.

See it in an app

Check out how Toast is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/toast.ts).

## Examples

Click a variant to push a toast onto the stack. Hover a toast to pause its auto-dismiss; move away and the timer restarts.

No toasts dismissed yet

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Match as M, Option, Schema as S } from 'effect'
import { Command, Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Toast as UiToast } from '@foldkit/ui'

// Define the payload shape for your toast. The Toast component owns only
// lifecycle + a11y fields (id, variant, transition, dismiss timer, hover
// state). The payload is yours, whatever you can encode in a Schema:
const ToastPayload = S.Struct({
  bodyText: S.String,
  maybeLink: S.Option(S.Struct({ href: S.String, text: S.String })),
})

// Bind a Toast module to your payload schema. The factory returns Model,
// Message, OutMessage, update, view, show/dismiss/dismissAll, and the
// DismissedToast OutMessage variant:
export const Toast = UiToast.make(ToastPayload)

// Add Toast.Model to your app Model. Track anything you want to lift from
// a toast's lifecycle alongside it. Here, the last dismissed bodyText so
// the UI can show "just dismissed: ..." after a toast goes away:
const Model = S.Struct({
  toast: Toast.Model,
  maybeLastDismissedBody: S.Option(S.String),
  // ...your other fields
})

// In your init function, initialize it:
const init = () => [
  {
    toast: Toast.init({ id: 'app-toast' }),
    maybeLastDismissedBody: Option.none(),
    // ...your other fields
  },
  [],
]

// Embed the Toast Message in your parent Message, plus any domain Messages
// that should push a toast:
const GotToastMessage = m('GotToastMessage', { message: Toast.Message })
const ClickedSave = m('ClickedSave')

// At module scope, fold the OutMessage into your own Model, lifting the
// DismissedToast event into domain state. The arm returns an Update.Step over
// the parent Model, which already has the next Toast Model written back:
const foldToastOutMessage = M.type<typeof Toast.OutMessage.Type>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    DismissedToast:
      ({ payload }) =>
      model => [
        evo(model, {
          maybeLastDismissedBody: () => Option.some(payload.bodyText),
        }),
        [],
      ],
  }),
)

// Update.foldChild wires the child into the parent: it delegates Toast's own
// Messages to Toast.update, writes the next Toast Model back, maps the
// Submodel's Commands into your Message type, and hands any OutMessage to
// foldOutMessage.
const foldToast = Update.foldChild({
  update: Toast.update,
  read: (model: Model) => Option.some(model.toast),
  write: (model, nextToast) => evo(model, { toast: () => nextToast }),
  toParentMessage: message => GotToastMessage({ message }),
  foldOutMessage: foldToastOutMessage,
})

// Inside your update's M.tagsExhaustive({...}), call the fold:
GotToastMessage: ({ message }) => foldToast(model, message)

ClickedSave: () => {
  const [nextToast, commands] = Toast.show(model.toast, {
    variant: 'Success',
    payload: {
      bodyText: 'Changes saved',
      // Generate the href via your app's router (Foldkit's biparser-based
      // routing builds URLs from typed values, e.g. \`changesRouter()\`),
      // not a string literal, so renames flow through.
      maybeLink: Option.some({ href: changesRouter(), text: 'View' }),
    },
  })

  return [
    evo(model, { toast: () => nextToast }),
    Command.mapMessages(commands, message => GotToastMessage({ message })),
  ]
}

// In your view, embed Toast via h.submodel once at the app root. The
// entryToView callback lays out each entry from its payload. The
// component handles the <li> wrapper, hover-to-pause, and enter/leave
// animations.
const view = (h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'app-toast',
    model: model.toast,
    view: Toast.view,
    viewInputs: {
      position: 'BottomRight',
      entryClassName: 'w-80',
      entryToView: (entry, handlers) =>
        h.div(
          [
            h.Class(
              'flex items-start gap-3 rounded-lg border bg-white p-3 shadow',
            ),
          ],
          [
            h.div(
              [h.Class('flex-1')],
              [
                h.p(
                  [h.Class('font-semibold text-sm')],
                  [entry.payload.bodyText],
                ),
                ...Option.match(entry.payload.maybeLink, {
                  onNone: () => [],
                  onSome: ({ href, text }) => [
                    h.a([h.Class('text-sm underline'), h.Href(href)], [text]),
                  ],
                }),
              ],
            ),
            h.button([...handlers.dismiss], ['Close']),
          ],
        ),
    },
    toParentMessage: message => GotToastMessage({ message }),
  })
```

## Styling

Toast is headless. The container gets `position: fixed` and flex-column layout from the component (so entries stack correctly for each `position`); every other visual decision lives in your `entryToView` callback and your `entryClassName`. Use `data-variant` on the entry to drive per-variant styling.

Each entry’s enter/leave animations flow through the [Animation](https://foldkit.dev/ui/animation) module. Style with CSS transitions or CSS keyframe animations. Animation advances once every animation on the element has settled.

| Attribute | Condition |
| --- | --- |
| `data-variant` | Present on each entry, with the variant value (Info, Success, Warning, Error). Use for per-variant CSS. |
| `data-enter` | Present on an entry while its enter animation runs. |
| `data-leave` | Present on an entry while its leave animation runs. |
| `data-closed` | Present on an entry at the closed extreme of its enter or leave animation. Pair with data-enter or data-leave to drive the starting and ending CSS states. |
| `data-transition` | Present on an entry while either animation runs. |

## Accessibility

The container is a `role="region"` with `aria-live="polite"`, always rendered (even when empty) so screen readers observe the live region from page load. Individual entries receive `role="status"` for Info and Success variants, `role="alert"` for Warning and Error. Auto-dismiss pauses on pointer hover.

## API Reference

### InitConfig

Configuration object passed to `Toast.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the toast container. |
| `defaultDuration` | `Duration.Input` | `Duration.seconds(4)` | Auto-dismiss duration applied to any show() call that does not provide its own duration or pass sticky: true. Accepts any Effect Duration input; a bare number is interpreted as milliseconds. |

### ShowInput

Input shape for `Toast.show(model, input)`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `payload` | `A (your payload type)` | — | Content for this entry, in whatever shape you supplied to Toast.make(). The component never reads it; it flows through to your entryToView callback. |
| `variant` | `'Info' \| 'Success' \| 'Warning' \| 'Error'` | `'Info'` | Semantic category. Maps to data-variant for styling and to role=status (Info, Success) or role=alert (Warning, Error) for accessibility. The only content-adjacent field the component owns. Everything else is in payload. |
| `duration` | `Duration.Input` | — | Overrides the container's defaultDuration for this entry. Ignored when sticky: true. |
| `sticky` | `boolean` | `false` | When true, the entry never auto-dismisses. The user must close it manually. |

### ViewConfig

Configuration object passed to `Toast.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `model` | `Toast.Model` | — | The toast container state from your parent Model. |
| `position` | `'TopLeft' \| 'TopCenter' \| 'TopRight' \| 'BottomLeft' \| 'BottomCenter' \| 'BottomRight'` | — | Where the toast viewport is anchored on the screen. |
| `toParentMessage` | `(childMessage: Dismissed \| HoveredEntry \| LeftEntry) => ParentMessage` | — | Wraps the subset of Toast Messages that fire from DOM events in your parent Message type. |
| `entryToView` | `(entry: typeof Toast.Entry.Type, handlers: { dismiss: ReadonlyArray<ChildAttribute> }) => Html` | — | Renders each entry from its lifecycle fields (for example id, variant, and animation) and its payload (your shape). The component wraps the return in an `<li>` with role, lifecycle handlers, and transition data attributes. Spread handlers.dismiss onto a close button (h.button(\[...handlers.dismiss\], \[...\])) so users can dismiss the entry manually. |
| `ariaLabel` | `string` | `'Notifications'` | aria-label on the container region. |
| `containerClassName` | `string` | — | CSS class for the container `<ol>`. |
| `entryClassName` | `string` | — | CSS class applied to every `<li>` entry. |

### Programmatic Helpers

Helper functions for driving toasts from parent update handlers, returning `[Model, Commands]`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `show` | `(model: Model, input: ShowInput) => [Model, Commands]` | — | Adds a new toast entry. Call this from any parent update handler that needs to surface a notification. Returns the next model plus commands for the enter animation and the auto-dismiss timer. |
| `dismiss` | `(model: Model, entryId: string) => [Model, Commands]` | — | Begins dismissing a specific entry. Safe to call for an entry that is already leaving or has been removed. |
| `dismissAll` | `(model: Model) => [Model, Commands]` | — | Begins dismissing every currently-visible entry. |

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Fold the OutMessage in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `DismissedToast` | `{ payload: Payload }` | — | Emitted once an entry has finished its leave animation and is being removed from the model. Carries the toast’s payload typed as your `Payload` schema. Fold it in the `foldOutMessage` of your Toast fold to lift the dismissal into domain state (e.g., resolving a pending action or firing analytics). Only fires after `TransitionedOut`, so it represents the actual removal, not the initial dismiss request. |
