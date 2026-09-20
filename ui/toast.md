---
url: https://foldkit.dev/ui/toast
title: "Toast"
description: "Stack of transient notifications anchored to a corner of the viewport with per-entry enter/leave animations and auto-dismiss."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

## Overview

A stack of transient notifications positioned along an edge of the viewport. Each entry has its own enter and leave animation and hover state. Non-sticky entries have independent auto-dismiss timers that pause on hover. Swipe-to-dismiss is opt-in for the container. Keep the container at the app root and add entries with `Toast.show`.

Toast is parameterized on a payload Schema that you provide. The component owns entry IDs, variants, animations, optional dismiss timers, hover state, and swipe state. Your payload holds the content rendered by `entryToView`. `Toast.make(PayloadSchema)` returns a module whose Model, helpers, and view are bound to that payload type.

See it in an app

Check out how Toast is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/toast.ts).

## Examples

Click a variant to add a toast. Hover a non-sticky toast to pause its auto-dismiss; move away and the timer restarts. Drag a toast right to dismiss it. If you release after more than 40px, it continues off-screen; otherwise it animates back. Press Escape while dragging to cancel.

No toasts dismissed yet

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Option, Schema } from 'effect'
import { Subscription, Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { defineMessageUnion } from 'foldkit/message'
import { modifyFields } from 'foldkit/struct'

import { Toast as UiToast } from '@foldkit/ui'

// Define the payload shape for your toast. Toast owns the entry lifecycle,
// accessibility role, optional dismiss timer, hover state, and swipe state.
// The payload is yours, whatever you can encode in a Schema:
const ToastPayload = Schema.Struct({
  bodyText: Schema.String,
  maybeLink: Schema.Option(
    Schema.Struct({ href: Schema.String, text: Schema.String }),
  ),
})

// Bind a Toast module to your payload schema. The factory returns Model,
// Message, OutMessage, update, view, show/dismiss/dismissAll, and the
// DismissedToast OutMessage variant:
export const Toast = UiToast.make(ToastPayload)

// Add Toast.Model to your app Model. Track anything you want to lift from
// a toast's lifecycle alongside it. Here, the last dismissed bodyText so
// the UI can show "just dismissed: ..." after a toast goes away:
const Model = Schema.Struct({
  toast: Toast.Model,
  maybeLastDismissedBody: Schema.Option(Schema.String),
  // ...your other fields
})
type Model = typeof Model.Type

// In your init function, initialize it. Swipe is opt-in: omit
// swipeToDismiss to leave it disabled, or pass {} for the default rightward
// 40px gesture. Set threshold to customize the dismissal distance:
const init = () => ({
  model: {
    toast: Toast.init({
      id: 'app-toast',
      swipeToDismiss: {},
    }),
    maybeLastDismissedBody: Option.none(),
    // ...your other fields
  },
})

// Embed the Toast Message in your parent Message, plus any domain Messages
// that should push a toast:
const Message = defineMessageUnion({
  GotToastMessage: { message: Toast.Message },
  ClickedSave: {},
})
type Message = typeof Message.Type

// At module scope, fold the OutMessage into your own Model, lifting the
// DismissedToast event into domain state. The arm returns an Update.Step over
// the parent Model, which already has the next Toast Model written back:
const foldToastOutMessage = Toast.OutMessage.match<Update.Step<Model, Message>>(
  {
    DismissedToast:
      ({ payload }) =>
      model => ({
        model: modifyFields(model, {
          maybeLastDismissedBody: () => Option.some(payload.bodyText),
        }),
      }),
  },
)

// Update.foldChild wires the child into the parent: it delegates Toast's own
// Messages to Toast.update, writes the next Toast Model back, maps the
// Submodel's Commands into your Message type, and hands any OutMessage to
// foldOutMessage.
const foldToast = Update.foldChild({
  update: Toast.update,
  read: (model: Model) => Option.some(model.toast),
  write: (model, nextToast) => modifyFields(model, { toast: () => nextToast }),
  toParentMessage: message => Message.GotToastMessage({ message }),
  foldOutMessage: foldToastOutMessage,
})

const foldToastShow = Update.foldChild({
  update: Toast.show,
  read: (model: Model) => Option.some(model.toast),
  write: (model, nextToast) => modifyFields(model, { toast: () => nextToast }),
  toParentMessage: message => Message.GotToastMessage({ message }),
  foldOutMessage: foldToastOutMessage,
})

// In the corresponding Message.match handler, call the fold:
GotToastMessage: ({ message }) => foldToast(model, message)

ClickedSave: () =>
  foldToastShow(model, {
    variant: 'Success',
    payload: {
      bodyText: 'Changes saved',
      // Generate the href via your app's router (Foldkit's biparser-based
      // routing builds URLs from typed values, e.g. \`changesRouter()\`),
      // not a string literal, so renames flow through.
      maybeLink: Option.some({ href: changesRouter(), text: 'View' }),
    },
  })

// Wire pointer subscriptions once at the app root so swipe tracking
// continues when the pointer leaves the entry. Without this lift the
// view still sets data-swipe on pointerdown but never receives move/up.
export const subscriptions = Subscription.lift(Toast.subscriptions)<
  Model,
  Message
>({
  toChildModel: model => model.toast,
  toParentMessage: message => Message.GotToastMessage({ message }),
})

// In your view, embed Toast via h.submodel once at the app root. The
// entryToView callback lays out each entry from its payload. The
// component handles the <div> wrapper, hover-to-pause, swipe-to-dismiss
// (pointerdown + data-swipe="move"/"settling"/"end" +
// translate/--toast-swipe-move-x), and enter/leave animations.
const view = (h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'app-toast',
    model: model.toast,
    view: Toast.view,
    viewInputs: {
      position: 'BottomRight',
      entryClassName: 'toast-entry w-80',
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
                  [
                    h.span(
                      [h.DataAttribute('toast-swipe-ignore', '')],
                      [entry.payload.bodyText],
                    ),
                  ],
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
    toParentMessage: message => Message.GotToastMessage({ message }),
  })
```

## Styling

Toast supplies layout and gesture styles but leaves the appearance to you. Its container has fixed positioning and a flex layout that stacks entries according to `position`. Use `containerClassName`, `entryClassName`, and `entryToView` for your styling; `data-variant` is available for per-variant CSS.

Each entry’s enter and leave phases flow through [Animation](https://foldkit.dev/ui/animation). Style the entry wrapper with CSS transitions or keyframe animations; each phase waits for those animations to finish.

## Gestures

Swipe is pointer-driven and opt-in. Pass `swipeToDismiss` to `Toast.init`. For example: `{}` enables a rightward swipe with a 40px threshold, `{ threshold: 120 }` requires a longer swipe, and `{ direction: 'Left' }` enables a leftward swipe. Without it, `Toast.view` attaches no `pointerdown` handler or `touch-action` restriction. Direction is independent of `position`, so configure it explicitly for a left-anchored stack.

With swipe enabled, `Toast.view` handles `pointerdown` on each entry. Wire `Toast.subscriptions` at the app root with `Subscription.lift(Toast.subscriptions)` to track `pointermove`, `pointerup`, and `pointercancel` even when the pointer leaves the entry. Without those subscriptions, a drag cannot finish. Escape cancels active drags. While dragging, Toast prevents text selection and shows the grabbing cursor. The example above includes the lift; see also [Toast subscriptions in the demo app](https://github.com/foldkit/foldkit/blob/main/packages/website/src/page/ui/subscriptions.ts).

Presses on buttons, links, form controls, and editable elements do not start a swipe, so a close button keeps its normal pointer behavior. To make text selectable with a mouse or pen, put `data-toast-swipe-ignore` on a span around the text, as the demo does. A touch can still start a swipe over that text. Other areas of the entry remain draggable.

While dragging, the entry follows the pointer only in the configured direction; movement in the opposite direction is clamped to zero. `Toast.view` sets `data-swipe="move"` and the entry's inline `translate` offset. The `translate` property composes with any `transform` animation you apply. Dismissal depends on release distance, not velocity. Releasing more than the threshold sets `data-swipe="end"`, holds the release offset until leave animation starts, then targets `100vw` or `-100vw` to carry the entry off-screen before `DismissedToast` fires.

Releasing at or below the threshold, or cancelling with Escape, returns the entry to zero offset and resumes auto-dismiss when applicable. Toast holds `data-swipe="settling"` for 150ms (`SWIPE_SETTLE_DURATION`) after either case. Add a `translate` transition for the snap-back and exit, as the demos do:

```
.toast-entry[data-swipe='settling'] {
  transition: translate 150ms ease-out;
}

.toast-entry[data-swipe='end'] {
  transition: translate 240ms ease-in;
}
```

While an entry is translated, the view exposes its drag or release offset as `--toast-swipe-move-x`. You can use it to style an action background behind the moving entry.

| Attribute | Condition |
| --- | --- |
| `data-variant` | Present on each entry, with the variant value (Info, Success, Warning, Error). Use for per-variant CSS. |
| `data-enter` | Present on an entry while its enter animation runs. |
| `data-leave` | Present on an entry while its leave animation runs. |
| `data-closed` | Present on an entry at the closed extreme of its enter or leave animation. Pair with data-enter or data-leave to drive the starting and ending CSS states. |
| `data-transition` | Present on an entry while either animation runs. |
| `data-swipe` | `move` while an entry is being dragged, `settling` while a short or cancelled swipe returns to rest, and `end` while a successful swipe exits. The view positions the entry with the inline `translate` property (which composes with your `transform` animations) and exposes the pointer offset as `--toast-swipe-move-x` for custom styling. |

## Accessibility

The container has `role="region"` and `aria-live="polite"`. It stays in the DOM when empty so screen readers can observe notifications added later. Each entry has `aria-atomic="true"` and receives `role="status"` for Info and Success or `role="alert"` for Warning and Error. Auto-dismiss pauses on pointer hover and while dragging. Swipe and close-button dismissal both use the leave animation and emit `DismissedToast` when the entry is removed.

## API Reference

### InitConfig

Configuration object passed to `Toast.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the toast container. |
| `defaultDuration` | `Duration.Input` | `Duration.seconds(4)` | Auto-dismiss duration for a `Toast.show` call without its own `duration` or `sticky: true`. Accepts any Effect Duration input; a bare number means milliseconds. |
| `swipeToDismiss` | `{ threshold?: number; direction?: 'Left' \| 'Right' }` | — | Opts the container into swipe-to-dismiss. Omit it to leave swipe disabled. Defaults to a rightward 40px threshold. Set `threshold` for the dismissal distance and `direction: 'Left'` for a leftward swipe. Applies per container. |

### ShowInput

Input shape for `Toast.show(model, input)`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `payload` | `A (your payload type)` | — | Content for this entry, in the shape you supplied to `Toast.make`. Toast passes it to your `entryToView` callback without reading it. |
| `variant` | `'Info' \| 'Success' \| 'Warning' \| 'Error'` | `'Info'` | Semantic category. Maps to data-variant for styling and to role=status (Info, Success) or role=alert (Warning, Error) for accessibility. The only content-adjacent field the component owns. Everything else is in payload. |
| `duration` | `Duration.Input` | — | Overrides the container's defaultDuration for this entry. No auto-dismiss timer is scheduled when `sticky: true`. |
| `sticky` | `boolean` | `false` | When true, the entry does not auto-dismiss. It can still be closed with the close button, an enabled swipe, or `Toast.dismiss`. |

### ViewInputs

Pass these fields in the `viewInputs` of `h.submodel({ view: Toast.view, ... })`. Pass the Toast Model and `toParentMessage` to `h.submodel` itself, as shown in the example above.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `position` | `'TopLeft' \| 'TopCenter' \| 'TopRight' \| 'BottomLeft' \| 'BottomCenter' \| 'BottomRight'` | — | Where the toast viewport is anchored on the screen. |
| `entryToView` | `(entry: typeof Toast.Entry.Type, handlers: EntryHandlers) => Html` | — | Renders content inside the entry `<div>`, which Toast owns along with its role, lifecycle handlers, and data attributes. Spread `handlers.dismiss` onto a close button to let users dismiss the entry. |
| `ariaLabel` | `string` | `'Notifications'` | aria-label on the container region. |
| `containerClassName` | `string` | — | CSS class for the container `<div>`. |
| `entryClassName` | `string` | — | CSS class applied to every entry `<div>`. |

### Programmatic Helpers

Toast helpers are child entry points. Fold `show` and `dismiss` with `Update.foldChild` because they take additional input. Fold `dismissAll` with `Update.foldChildStep`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `show` | `(model: Model, input: ShowInput) => Update.ReturnWithOutMessage<Model, Message, OutMessage>` | — | Adds a new toast entry. Fold it from any parent handler that needs to surface a notification. Returns the next Model plus Commands for the enter animation and, unless sticky, auto-dismiss. |
| `dismiss` | `(model: Model, entryId: string) => Update.ReturnWithOutMessage<Model, Message, OutMessage>` | — | Begins dismissing a specific entry. Calling it for an entry that is already leaving or has been removed is a no-op. |
| `dismissAll` | `(model: Model) => Update.ReturnWithOutMessage<Model, Message, OutMessage>` | — | Begins dismissing every currently-visible entry. |

### Subscriptions

Toast exposes `Toast.subscriptions` with `swipePointer` and `swipeEscape`. Lift them once at the app root so pointer tracking continues even when the pointer leaves the entry. The [example above](#examples) shows the lift alongside the Toast Model and Message, and the [demo app](https://github.com/foldkit/foldkit/blob/main/packages/website/src/page/ui/subscriptions.ts) shows it in context.

Without the lift the view still renders `data-swipe="move"` for the initial `pointerdown`, but `pointermove` and `pointerup` never reach the update and the gesture cannot complete.

If you render the Toast Model without `Toast.view`, you own the entry markup, accessibility roles, `pointerdown` handler, and swipe styling. Read the pointer offset with `Toast.swipeOffset(entry.swipeState)` and apply the `translate` property yourself. Mirror the `data-swipe` phases: `move` while the entry is `Dragging`, `settling` while it is `Settling`, and `end` while it is `Dismissing`. In the `end` phase, retain the offset at `LeaveStart`, then move toward `100vw` or `-100vw` at `LeaveAnimating` according to the swipe direction. By contrast, `entryToView` only renders content inside the `<div>` wrapper owned by `Toast.view`.

### OutMessage

Messages emitted to the parent through the optional `outMessage` field. Fold the OutMessage in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `DismissedToast` | `{ payload: Payload }` | — | Emitted when an entry finishes its leave animation and is removed from the Model. It carries the payload from your `Payload` Schema. Handle it in `foldOutMessage` to update domain state or return an analytics Command. It reports removal, not the initial dismiss request. |
