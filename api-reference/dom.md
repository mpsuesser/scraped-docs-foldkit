---
url: https://foldkit.dev/api-reference/dom
title: "Dom"
description: "API documentation for the Dom module."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

# Dom

## Functions

### advanceFocus

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L798)

```
/**
 * Focuses the next or previous focusable element in the document relative to the element matching the given selector.
 * Fails with `ElementNotFound` if the selector does not match an `HTMLElement`.
 */
(
  selector: string,
  direction: FocusDirection
): Effect<void, ElementNotFound>
```

### clickElement

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L654)

```
/**
 * Programmatically clicks an element matching the given selector.
 * Fails with `ElementNotFound` if the selector does not match an `HTMLElement`.
 */
(selector: string): Effect<void, ElementNotFound>
```

### closeDialog

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L534)

```
/**
 * Closes a dialog element using `.close()`.
 * Cleans up the keyboard handlers installed by `showDialog`, restores modal
 * background isolation, and then returns focus to the element that was focused
 * before the dialog opened (the trigger, or the dialog beneath it when closing
 * a stacked dialog).
 * Resolves to `true` when it released the keyboard handlers, the return
 * focus, and the stack entry.
 * Resolves to `false` when the dialog held none, for example when the close
 * runs before `showDialog` has installed them. A caller that unlocks page
 * scroll after the close should unlock only when the result is `true`.
 * Fails with `ElementNotFound` if the selector does not match an `HTMLDialogElement`.
 */
(selector: string): Effect<boolean, ElementNotFound>
```

### detectElementMovement

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/elementMovement.ts#L23)

```
/**
 * Detects if the element matching the given selector moves in the viewport.
 * Snapshots the element's position via `getBoundingClientRect` and watches for
 * changes using a `ResizeObserver` plus window `scroll` and `resize` listeners.
 * Resolves when movement is detected. Falls back to completing immediately if
 * the element is missing.
 * 
 * Cleanup runs automatically when the fiber is interrupted (e.g. by
 * `Effect.raceFirst`), removing the observer and event listeners via
 * `AbortSignal`.
 */
(selector: string): Effect<void>
```

### focus

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L329)

```
/**
 * Focuses an element matching the given selector after the next render has
 * committed.
 * 
 * Use `Dom.focus` inside a Command for focus that's caused by a Message
 * dispatching: a dialog opening, an input becoming the active step in a
 * form, returning focus to a trigger button after a popover closes,
 * keyboard navigation across a stable layout. The Command fires from
 * `update`'s return; the focus runs after the next render commits, so the
 * element is in place by the time `.focus()` runs.
 * 
 * Do not use `OnMount` for focus. The cause of focus-on-open is the
 * Message, not the element appearing. Mount is for per-instance lifecycle
 * effects bound to a VNode existing where the live element handle is
 * needed (positioning, portaling, observer attachment, library setup).
 * 
 * Waiting for the commit puts the element in the DOM. It does not make the
 * element focusable. `.focus()` is a no-op on an element that is not
 * rendered, so a target behind `visibility: hidden` or `display: none`
 * leaves focus where it was, with nothing to distinguish that from focus
 * having landed. This is worth knowing when something asynchronous reveals
 * the target after the render commits: a panel held at `visibility: hidden`
 * until a positioning library resolves its first layout is still hidden
 * when the Command runs, however long the Command waits. Focus a target
 * like that from whatever performs the reveal, which is the one place that
 * knows the element has become focusable. The Message still causes the
 * reveal, so the focus belongs to that Message rather than to a lifecycle
 * effect standing in for it.
 * 
 * Section headings, articles, and other non-natively-focusable elements
 * are common URL fragment targets, but `.focus()` is a no-op on them
 * without a `tabindex`. Pass `makeFocusable: true` to inject
 * `tabindex="-1"` on the target if it has none, making programmatic
 * focus actually land. Pass `preventScroll: true` to suppress the
 * browser's default scroll-on-focus, useful when the focus call follows
 * a deliberate scroll that should not be undone. The two options compose
 * with `scrollIntoViewAfterPaint` for URL-fragment-navigation
 * accessibility: scroll the section into view, then focus the same
 * selector so keyboard users start Tab navigation from the target.
 * 
 * Fails with `ElementNotFound` if the selector does not match an `HTMLElement`.
 */
(
  selector: string,
  options?: Readonly<{
    makeFocusable: boolean
    preventScroll: boolean
  }>
): Effect<void, ElementNotFound>
```

### inertOthers

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/inert.ts#L203)

```
/**
 * Marks all DOM elements outside the given selectors as `inert` and
 * `aria-hidden="true"`. Walks each allowed element up to `document.body`,
 * marking siblings that don't contain an allowed element. Uses reference
 * counting so nested calls are safe. A restore before the pending render
 * commits invalidates the request before it can change the DOM.
 */
(
  id: string,
  allowedSelectors: readonly Array<string>
): Effect<void>
```

### releaseDialogResources

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L636)

```
/**
 * Releases the framework hygiene a dialog holds while open: the focus-trap
 * keyboard handler, modal background isolation, the recorded return focus,
 * the dialog stack entry, the z-index counter, and one page scroll lock.
 * Use this when the element is removed without a close Message, such as
 * navigation away from a route-keyed subtree. The runtime also calls it
 * directly on disposal. The normal close path already releases these.
 * That path is `closeDialog` first, then the Dialog component's scroll unlock
 * when `closeDialog` reports a release. This function is the cleanup for the
 * case where no close Message ever reaches `update`.
 * 
 * Addressed by the dialog's id, not a selector, because the element is
 * typically already gone from the DOM by the time this runs (that is the whole
 * point of the backstop). The hygiene installed by `showDialog` is tracked by
 * id so it can be reclaimed without a live element handle. The id must be
 * non-empty and unique within the document, since it keys this cleanup
 * accounting; a duplicate or empty id would release the wrong dialog's hygiene.
 * 
 * Idempotent and exactly-once. It releases only when the dialog currently
 * holds hygiene, then clears the per-dialog marker, so calling it after a
 * normal close, or twice, is a no-op that never under-counts the shared
 * scroll lock. Carries no application close semantics: the Dialog component
 * owns the user-facing close (animation, `Closed` OutMessage, consumer
 * Commands); this only reclaims framework resources.
 * 
 * Resolves to `true` when it released resources, `false` when there was
 * nothing to release. Never fails: an id with no held hygiene is a no-op,
 * since the goal is reclaiming resources that may already be gone.
 */
(id: string): Effect<boolean>
```

### restoreInert

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/inert.ts#L235)

```
/**
 * Restores all elements previously marked inert by `inertOthers` for the
 * given ID. Safe to call without a preceding `inertOthers`. Acts as a no-op
 * in that case.
 */
(id: string): Effect<void>
```

### scrollIntoView

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L678)

```
/**
 * Scrolls an element into view by selector. Resolves the selector after
 * `Render.afterCommit`. Defaults to `{ block: 'nearest' }`; pass a different
 * `block` for use cases like URL-fragment landing where `'start'` is right.
 * For a target the same Message just brought into the DOM,
 * `scrollIntoViewAfterPaint` is the right choice.
 * 
 * Fails with `ElementNotFound` if the selector does not match an `HTMLElement`.
 */
(
  selector: string,
  options?: Readonly<{
    block: ScrollLogicalPosition
  }>
): Effect<void, ElementNotFound>
```

### scrollIntoViewAfterPaint

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L709)

```
/**
 * Like `scrollIntoView`, but waits for `Render.afterPaint` instead of
 * `Render.afterCommit` before resolving the selector.
 * 
 * Reach for this when the target was just brought into the DOM by the same
 * Message that dispatches the scroll, such as a routing flow landing at a
 * URL fragment. The two-frame wait gives the runtime time to commit the new
 * Model and the browser time to lay it out before the scroll runs. For a
 * target that's already on screen, `scrollIntoView` is the lighter choice.
 * 
 * Defaults to `{ block: 'nearest' }`; pass `{ block: 'start' }` for URL
 * fragment landings where the target should sit at the top of the viewport.
 * 
 * Fails with `ElementNotFound` if the selector does not match an `HTMLElement`.
 */
(
  selector: string,
  options?: Readonly<{
    block: ScrollLogicalPosition
  }>
): Effect<void, ElementNotFound>
```

### scrollIntoViewIfNotVisible

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L755)

```
/**
 * Like `scrollIntoViewAfterPaint`, but skips the scroll if the element is
 * already fully visible within its scroll container.
 * 
 * Defaults to `{ block: 'center' }`; pass a different `block` when the
 * target should land at a specific position when it does need to scroll.
 * 
 * `when` selects the timing gate. `'Paint'` (the default) waits for
 * `Render.afterPaint`, so the scroll lands after the target is on screen.
 * `'Commit'` waits for `Render.afterCommit` instead, so the scroll lands in
 * the same frame the DOM patch applies, before the browser paints. Use
 * `'Commit'` when the target is brought into view and scrolled by the same
 * Message (such as a menu opening), so it appears already scrolled rather
 * than visibly jumping.
 * 
 * Fails with `ElementNotFound` if the selector does not match an `HTMLElement`.
 */
(
  selector: string,
  options?: Readonly<{
    block: ScrollLogicalPosition
    when: "Paint" | "Commit"
  }>
): Effect<void, ElementNotFound>
```

### showDialog

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L373)

```
/**
 * Opens a dialog element using `show()` with high z-index, focus trapping,
 * and Escape key handling. An unhandled Escape on the topmost dialog dispatches
 * a `CustomEvent` named `cancel`, distinguishing it from native `cancel` events
 * while preserving the dialog event contract. Uses `show()` instead of
 * `showModal()` so that DevTools (and any other high-z-index overlay) remains
 * interactive. Pass `isModal: true` to make the background inert and hide it
 * from assistive technology. `allowedOutsideSelectors` keeps separate developer
 * overlays available while modal isolation is active. Stacked modal dialogs
 * isolate against the topmost one, and closing it restores the dialog beneath.
 * The Dialog component provides its own backdrop, scroll locking,
 * and transitions. Fails with `ElementNotFound` if the selector does not match
 * an `HTMLDialogElement`.
 * 
 * Pass `focusSelector` to focus an element inside the dialog when it opens.
 * When it does not match a focusable element, or when none is provided, focus
 * falls back to the first focusable descendant and then to the dialog itself.
 * 
 * Records the element that had focus when the dialog opened so `closeDialog`
 * can return focus there, the way `showModal()` would natively. Resolves to
 * `true` when it installs the dialog resources, or `false` when that id already
 * holds them. The latter makes concurrent lifecycle recovery and application
 * Commands safe without duplicating focus traps or stack entries.
 */
(
  selector: string,
  options?: Readonly<{
    allowedOutsideSelectors: readonly Array<string>
    focusSelector: string
    isModal: boolean
  }>
): Effect<boolean, ElementNotFound>
```

### streamFromEvent

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromEvent.ts#L394)

```
/**
 * Build a Stream that emits a value for every dispatch of a DOM event,
 * registering the listener when the Stream's scope opens and removing it when
 * the scope closes.
 * 
 * The target, the event name, and the event the mapper receives are one fact:
 * `type` is constrained to the names the target declares, and the mapper's
 * parameter is what those two resolve to, so annotating it narrows nothing and
 * cannot contradict the name. A target that is neither annotated nor one
 * lib.dom declares a map for accepts any name and reports `Event`; annotate it
 * with TypedEventTarget to resolve its own events.
 * 
 * The listener lifecycle uses `Effect.acquireRelease`. The `addEventListener`
 * call happens inside the acquire Effect, and the matching
 * `removeEventListener` is registered only after acquire completes, so the
 * listener never leaks on interruption.
 * 
 * This is a Stream, not a Subscription entry. Wrap it with
 * `Subscription.persistentEntry` for a listener with no local Model dependencies,
 * or plug it into a `Subscription.make` entry's
 * `dependenciesToStream` (typically behind `Stream.when`) to gate it on a
 * Model condition. The mapper's output type is inferred (even a raw Event is
 * accepted here); `Subscription.make` checks the final Stream against the
 * application's Message type.
 * 
 * For a listener that reacts to only some events, reach for
 * `streamFromEventFilterMap`, whose mapper returns `Option<Output>`. For a
 * listener that also cancels the default action of the events it handles,
 * reach for `streamFromEventFilterMapPreventDefault`.
 */
<Target extends EventTarget, Type extends string, Output>(config: StreamFromEventConfig<Target, Type, Output>): Stream<Output>
```

### streamFromEventFilterMap

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromEvent.ts#L335)

```
/**
 * Build a Stream that emits a value for the dispatches of a DOM event the
 * mapper chooses to keep, registering the listener when the Stream's scope
 * opens and removing it when the scope closes.
 * 
 * This is the filtered variant of `streamFromEvent`. Its `filterMapEvent` returns
 * `Option.some(value)` to emit and `Option.none()` to ignore the event, so a
 * single listener can react to some dispatches while passing on the rest. A
 * mapper that never emits produces a `Stream<never>`.
 * 
 * Reach for this over a downstream `Stream.filterMap` whenever the decision to
 * keep an event is paired with `event.preventDefault()`. The mapper runs
 * synchronously inside the browser's event dispatch, so `preventDefault()`
 * takes effect, while a downstream filter would run on a later turn after the
 * default action has already happened. The exception is a passive listener,
 * which ignores `preventDefault()`. Some browsers default wheel and touch
 * listeners on global targets to passive. Pass
 * `options: { passive: false }` explicitly when cancelling those events, or
 * reach for `streamFromEventFilterMapPreventDefault`, which does so for you.
 * 
 * The target, the event name, and the event the mapper receives are one fact:
 * `type` is constrained to the names the target declares, and the mapper's
 * parameter is what those two resolve to, so annotating it narrows nothing and
 * cannot contradict the name. A target that is neither annotated nor one
 * lib.dom declares a map for accepts any name and reports `Event`; annotate it
 * with TypedEventTarget to resolve its own events.
 * 
 * The listener lifecycle uses `Effect.acquireRelease`. The `addEventListener`
 * call happens inside the acquire Effect, and the matching
 * `removeEventListener` is registered only after acquire completes, so the
 * listener never leaks on interruption.
 * 
 * This is a Stream, not a Subscription entry. Wrap it with
 * `Subscription.persistentEntry` for a listener with no local Model dependencies,
 * or plug it into a `Subscription.make` entry's
 * `dependenciesToStream` (typically behind `Stream.when`) to gate it on a
 * Model condition. The mapper's output type is inferred (even a raw Event is
 * accepted here); `Subscription.make` checks the final Stream against the
 * application's Message type.
 */
<Target extends EventTarget, Type extends string, Output>(config: StreamFromEventFilterMapConfig<Target, Type, Output>): Stream<Output>
```

### streamFromEventFilterMapPreventDefault

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromEvent.ts#L465)

```
/**
 * Build a Stream that emits a value for the dispatches of a DOM event the
 * mapper marks handled, calling `event.preventDefault()` on each of them,
 * registering the listener when the Stream's scope opens and removing it when
 * the scope closes.
 * 
 * This is the cancelling variant of `streamFromEventFilterMap`, mirroring
 * `h.OnKeyDownPreventDefault` from `foldkit/html`. Its `filterMapEvent` returns
 * `Option.some(value)` to mark a dispatch handled. The helper evaluates the
 * mapper, calls `event.preventDefault()`, and queues the value before the
 * native listener returns. `Option.none()` leaves the default behavior intact.
 * The mapper never calls `preventDefault()` itself.
 * 
 * Because cancelling is the point, the listener registers with
 * `passive: false` when the config does not say otherwise. This keeps wheel
 * and touch events cancelable when a browser would otherwise make listeners
 * on a global target passive. The config rejects `passive: true`; the runtime
 * guard also throws for unchecked JavaScript inputs.
 * 
 * The target, event name, and mapper parameter are one fact: `type` is
 * constrained to the names the target declares, and the mapper receives the
 * event those two resolve to. A target with no declared event map accepts any
 * name and reports `Event`; annotate it with TypedEventTarget to
 * resolve its own events.
 * 
 * The listener lifecycle uses `Effect.acquireRelease`. The `addEventListener`
 * call happens inside the acquire Effect, and the matching
 * `removeEventListener` is registered only after acquire completes, so the
 * listener never leaks on interruption.
 * 
 * This is a Stream, not a Subscription entry. Wrap it with
 * `Subscription.persistentEntry` for a listener with no local Model dependencies,
 * or plug it into a `Subscription.make` entry's
 * `dependenciesToStream` (typically behind `Stream.when`) to gate it on a
 * Model condition. The mapper's output type is inferred (even a raw Event is
 * accepted here); `Subscription.make` checks the final Stream against the
 * application's Message type.
 */
<Target extends EventTarget, Type extends string, Output>(config: StreamFromEventFilterMapPreventDefaultConfig<Target, Type, Output>): Stream<Output>
```

### streamFromKeyBindings

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromKeyBindings.ts#L955)

```
/**
 * Build a Stream that maps declarative key bindings to values. The output is
 * inferred from each binding's `mapEvent` callback; `Subscription.make`
 * checks that the final Stream emits the application's Message type.
 * 
 * A string describes one key press. Modifiers are joined with `+`:
 * `'Mod+K'`, `'Control+Shift+P'`, or `'Alt+ArrowDown'`. The supported modifiers
 * are `Mod`, `Control`, `Meta`, `Alt`, and `Shift`. `Mod` resolves to Meta on
 * Apple platforms and Control elsewhere; `modKey` can override that choice.
 * Matching uses `KeyboardEvent.key`, case-insensitively, after the active
 * keyboard layout has been applied. Use `Space` and `Plus` for those keys.
 * 
 * An array describes an ordered sequence of two or more presses. Every press
 * uses the same grammar, so `['G', 'Shift+G']` is valid. Sequences reset after
 * one second by default; `sequenceTimeout` accepts any Effect Duration input.
 * Modifier-only events and repeated keydowns do not advance a sequence.
 * 
 * Bindings are suppressed by default when the event's composed path contains
 * an `input`, `textarea`, `select`, or contenteditable element. Set
 * `whileTyping` to `'Allow'` for a binding that must work there. Events emitted
 * during IME composition are always ignored. Repeated keydowns are ignored for
 * one-press bindings unless `whenRepeated` is `'Allow'`. An event another
 * handler already canceled is ignored and clears any sequence in progress.
 * 
 * Matched key presses call `preventDefault()` before dispatching. For a
 * sequence, that policy applies to every matched press. Set `preventDefault`
 * to `false` to opt out. Sequences sharing a prefix must use the same policy.
 * Duplicate bindings and a complete binding that is also a sequence prefix
 * are rejected when the Stream is created.
 * 
 * This helper returns a Stream, not a complete Subscription entry. Use
 * `Subscription.persistentEntry` for a fixed table. When availability depends on
 * the Model that owns the entry, build it inside `dependenciesToStream` and
 * derive each binding's `isEnabled` from the dependency record. A dependency
 * change opens a new Stream scope and resets any sequence in progress. If a
 * parent owns a condition for a lifted child, declare the table at that parent
 * or put bindings with different parent-owned lifetimes in separate child
 * entries so `Subscription.lift` can gate them individually. If the meaning
 * of a key depends on the Model, dispatch a factual key Message and decide
 * what it means in update instead of reading the Model from `mapEvent`.
 */
<Output>(config: StreamFromKeyBindingsConfig<Output>): Stream<Output>
```

### streamFromMediaQuery

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromMediaQuery.ts#L58)

```
/**
 * Creates a Stream from a CSS media query. When the Stream starts, it emits the
 * current `matches` value through `mapMatches`. It emits again whenever that
 * value changes. Stopping the Stream removes the listener.
 * 
 * The Stream reads the current value again each time it restarts. Suppose a
 * color-scheme Subscription runs only while the theme preference is `System`.
 * The user selects `Dark`, changes the operating system to a light theme, and
 * then selects `System` again. A new `change` listener waits for the next
 * change, so the Model still records a dark system theme. This helper emits the
 * current light value as soon as the Stream restarts.
 * 
 * Creating the Stream does not access `window`; `window.matchMedia` is called
 * only when the Stream starts. The Stream can therefore be created during
 * server rendering as long as it runs only in the browser.
 * 
 * This helper returns a Stream, not a Subscription entry. Pass it to
 * `Subscription.persistentEntry` for a query the application always follows. To
 * follow the query only in a particular Model state, use it with `Stream.when`
 * inside a `Subscription.make` entry.
 */
<Output>(config: StreamFromMediaQueryConfig<Output>): Stream<Output>
```

### waitForAnimationSettled

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/waitForAnimation.ts#L17)

```
/**
 * Waits for all CSS animations on the element matching the selector to settle.
 * Covers both CSS transitions and CSS keyframe animations via the Web Animations
 * API. Falls back to completing immediately if the element is missing or has no
 * active animations.
 * 
 * Leave animations must be finite. `animation-iteration-count: infinite` will
 * keep the underlying `.finished` promise pending and hang the caller.
 */
(selector: string): Effect<void>
```

## Types

### FocusDirection

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/dom.ts#L787)

```
/** Direction for focus advancement: forward or backward in tab order. */
type FocusDirection = "Next" | "Previous"
```

### KeyBinding

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromKeyBindings.ts#L36)

```
/**
 * One entry in a streamFromKeyBindings binding table.
 * 
 * A string describes one key press, such as `'/'`, `'Escape'`, or `'Mod+K'`.
 * An array describes a sequence of at least two presses, such as
 * `['G', 'H']` or `['G', 'Shift+G']`.
 */
type KeyBinding = BindingBase<Output> & Readonly<{
  keys: string
  whenRepeated: "Ignore" | "Allow"
}> | Readonly<{
  keys: Readonly<[string, string, ...Array<string>]>
  whenRepeated: never
}>
```

### KeySequence

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromKeyBindings.ts#L19)

```
/** A single key press or a sequence of two or more key presses. */
type KeySequence = string | Readonly<[string, string, ...Array<string>]>
```

### StreamFromEventConfig

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromEvent.ts#L149)

```
/**
 * Configuration for the `streamFromEvent` Stream helper.
 * 
 * `target` is read inside the acquire Effect, never before it, so the
 * resolved `EventTarget` is captured at the moment the Stream's scope
 * opens. Pass a thunk when the target may not exist until the scope opens, or
 * pass the `EventTarget` directly for always-present globals like `window` or
 * `document`.
 * 
 * `type` is constrained to the event names the target declares, and
 * `mapEvent`'s parameter is the event those two resolve to. Annotating that
 * parameter is checked against the resolved event rather than replacing it.
 * 
 * `mapEvent(event)` transforms each dispatched event into a Stream value. The
 * mapper runs synchronously in the same call stack as the browser's event
 * dispatch, so calling `event.preventDefault()` inside it takes effect,
 * unless the listener is passive. Some browsers default wheel and touch
 * listeners on global targets to passive, where `preventDefault()` is
 * ignored. Pass `options: { passive: false }` explicitly when cancelling
 * those events, or reach for `streamFromEventFilterMapPreventDefault`, which does
 * so for you.
 * 
 * The output type is inferred from the mapper; `Subscription.make` checks
 * that the final Stream emits the application's Message type.
 */
type StreamFromEventConfig = Readonly<{
  mapEvent: (event: EventOf<Target, Type>) => Output
  options: AddEventListenerOptions
  target: Target | () => Target
  type: Type
}>
```

### StreamFromEventFilterMapConfig

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromEvent.ts#L185)

```
/**
 * Configuration for the `streamFromEventFilterMap` Stream helper.
 * 
 * `target` is read inside the acquire Effect, never before it, so the
 * resolved `EventTarget` is captured at the moment the Stream's scope
 * opens. Pass a thunk when the target may not exist until the scope opens, or
 * pass the `EventTarget` directly for always-present globals like `window` or
 * `document`.
 * 
 * `type` is constrained to the event names the target declares, and
 * `filterMapEvent`'s parameter is the event those two resolve to. Annotating that
 * parameter is checked against the resolved event rather than replacing it.
 * 
 * `filterMapEvent(event)` returns `Option.some(value)` to emit a value for the
 * event, or `Option.none()` to ignore it. The mapper runs synchronously in the
 * same call stack as the browser's event dispatch, so calling
 * `event.preventDefault()` inside it takes effect, unless the listener is
 * passive. Some browsers default wheel and touch listeners on global targets
 * to passive, where `preventDefault()` is ignored. Pass
 * `options: { passive: false }` explicitly when cancelling those events, or
 * reach for `streamFromEventFilterMapPreventDefault`, which does so for you.
 * 
 * The output type is inferred from the mapper; `Subscription.make` checks
 * that the final Stream emits the application's Message type.
 */
type StreamFromEventFilterMapConfig = Readonly<{
  filterMapEvent: (event: EventOf<Target, Type>) => Option.Option<Output>
  options: AddEventListenerOptions
  target: Target | () => Target
  type: Type
}>
```

### StreamFromEventFilterMapPreventDefaultConfig

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromEvent.ts#L260)

```
/**
 * Configuration for the `streamFromEventFilterMapPreventDefault` Stream helper.
 * 
 * `target` is read inside the acquire Effect, never before it, so the
 * resolved `EventTarget` is captured at the moment the Stream's scope
 * opens. Pass a thunk when the target may not exist until the scope opens, or
 * pass the `EventTarget` directly for always-present globals like `window` or
 * `document`.
 * 
 * `type` is constrained to the event names the target declares, and
 * `filterMapEvent`'s parameter is the event those two resolve to. Annotating that
 * parameter is checked against the resolved event rather than replacing it.
 * 
 * `filterMapEvent(event)` returns `Option.some(value)` to mark the dispatch
 * handled, or `Option.none()` to leave the default behavior intact. For a
 * handled dispatch the helper calls `event.preventDefault()` and queues the
 * value before the listener returns; the mapper itself never calls
 * `preventDefault()`.
 * 
 * `options.passive` defaults to `false` so `preventDefault()` keeps working
 * for the events browsers would otherwise register as passive. The config
 * rejects `passive: true`; the runtime guard also throws for unchecked
 * JavaScript inputs.
 * 
 * The output type is inferred from the mapper; `Subscription.make` checks
 * that the final Stream emits the application's Message type.
 */
type StreamFromEventFilterMapPreventDefaultConfig = Readonly<{
  filterMapEvent: (event: EventOf<Target, Type>) => Option.Option<Output>
  options: PreventDefaultEventListenerOptions
  target: Target | () => Target
  type: Type
}>
```

### StreamFromKeyBindingsConfig

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromKeyBindings.ts#L51)

```
/** Configuration for the streamFromKeyBindings Stream helper. */
type StreamFromKeyBindingsConfig = Readonly<{
  bindings: ReadonlyArray<KeyBinding<Output>>
  modKey: ModKey
  sequenceTimeout: Duration.Input
  target: EventTarget | () => EventTarget
}>
```

### StreamFromMediaQueryConfig

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromMediaQuery.ts#L19)

```
/**
 * Options for `streamFromMediaQuery`.
 * 
 * `query` accepts any media query string supported by `window.matchMedia`,
 * such as `'(prefers-reduced-motion: reduce)'`,
 * `'(prefers-color-scheme: dark)'`, or a viewport breakpoint like
 * `'(max-width: 1023px)'`. `window.matchMedia` is called when the Stream
 * starts, not when the Stream is created.
 * 
 * `mapMatches` converts the Boolean `matches` result into each value the Stream
 * emits.
 * 
 * The return type of `mapMatches` determines the Stream's output type.
 * `Subscription.make` checks that output against the application's Message
 * type.
 */
type StreamFromMediaQueryConfig = Readonly<{
  mapMatches: (isMatching: boolean) => Output
  query: string
}>
```

### WhileTyping

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromKeyBindings.ts#L16)

```
/** Whether a key binding may fire when its event comes from an editable element. */
type WhileTyping = "Suppress" | "Allow"
```

## Interfaces

### TypedEventTarget

interface

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/streamFromEvent.ts#L25)

```
/**
 * An `EventTarget` that declares the events it dispatches, so the `streamFromEvent`
 * helpers can resolve an event name to its event type the way they do for
 * `window`, `document`, and the DOM interfaces lib.dom declares event maps
 * for.
 * 
 * Annotate a target with this and the mapper's parameter follows from the
 * event name, including a `CustomEvent`'s `detail`. A declared event overrides
 * the corresponding native event and otherwise augments the target's native
 * events, so an element that dispatches custom events can be annotated without
 * losing events such as `click`. Any `EventTarget` is assignable to it, so the
 * annotation is the only change needed.
 */
interface TypedEventTarget {
  [EventMapMarker]: EventMap
  addEventListener: unknown
  dispatchEvent: unknown
  removeEventListener: unknown
}
```

## Constants

### lockScroll

const

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/scrollLock.ts#L59)

```
/**
 * Locks page scroll by setting `overflow: hidden` on the document element.
 * Compensates for scrollbar width with padding to prevent layout shift.
 * On iOS Safari, intercepts `touchmove` events to prevent page scroll
 * while allowing scrolling within overflow containers.
 * Uses reference counting so nested locks are safe. The page only unlocks
 * when every lock has been released.
 */
const lockScroll: Effect.Effect<void>
```

### unlockScroll

const

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/dom/scrollLock.ts#L95)

```
/**
 * Releases one scroll lock. When the last lock is released, restores the
 * original `overflow` and `padding-right` on the document element.
 * On iOS Safari, removes the `touchmove` listener.
 */
const unlockScroll: Effect.Effect<void>
```
