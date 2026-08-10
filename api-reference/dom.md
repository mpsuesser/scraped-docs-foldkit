---
url: https://foldkit.dev/api-reference/dom
title: "Dom"
description: "API documentation for the Dom module."
access_date: 2026-08-10T18:30:27.283Z
current_date: 2026-08-10T18:30:27.283Z
---

# Dom

## Functions

### advanceFocus

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L473)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L329)

```
/**
 * Programmatically clicks an element matching the given selector.
 * Fails with `ElementNotFound` if the selector does not match an `HTMLElement`.
 */
(selector: string): Effect<void, ElementNotFound>
```

### closeDialog

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L246)

```
/**
 * Closes a dialog element using `.close()`.
 * Cleans up the keyboard handlers installed by `showDialog` and restores focus to
 * the element that was focused before the dialog opened (the trigger, or the
 * dialog beneath it when closing a stacked dialog).
 * Fails with `ElementNotFound` if the selector does not match an `HTMLDialogElement`.
 */
(selector: string): Effect<void, ElementNotFound>
```

### detectElementMovement

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/elementMovement.ts#L23)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L104)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/inert.ts#L103)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L311)

```
/**
 * Releases the framework hygiene a dialog holds while open: the focus-trap
 * keyboard handler, the recorded return focus, the dialog stack entry, the
 * z-index counter, and one page scroll lock. Use this as a backstop for the
 * case where a dialog's element is removed from the DOM without a purposeful
 * close, the classic example being navigation away from a route-keyed subtree
 * that contains the dialog. The normal close path (`closeDialog` plus the
 * Dialog component's `unlockScroll` Command) already releases these, so this
 * is the missing teardown when no close Message ever flows through `update`.
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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/inert.ts#L143)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L353)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L384)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L430)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L135)

```
/**
 * Opens a dialog element using `show()` with high z-index, focus trapping,
 * and Escape key handling. Uses `show()` instead of `showModal()` so that
 * DevTools (and any other high-z-index overlay) remains interactive. The
 * Dialog component provides its own backdrop, scroll locking, and transitions.
 * Fails with `ElementNotFound` if the selector does not match an `HTMLDialogElement`.
 * 
 * Pass `focusSelector` to focus an element inside the dialog when it opens.
 * 
 * Records the element that had focus when the dialog opened so `closeDialog`
 * can return focus there, the way `showModal()` would natively.
 */
(
  selector: string,
  options?: Readonly<{
    focusSelector: string
  }>
): Effect<void, ElementNotFound>
```

### waitForAnimationSettled

function

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/waitForAnimation.ts#L17)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/dom.ts#L462)

```
/** Direction for focus advancement: forward or backward in tab order. */
type FocusDirection = "Next" | "Previous"
```

## Constants

### lockScroll

const

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/scrollLock.ts#L59)

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

[source](https://github.com/foldkit/foldkit/blob/14bb759c9ccc122a11ec694312d9ab163654b474/packages/foldkit/src/dom/scrollLock.ts#L95)

```
/**
 * Releases one scroll lock. When the last lock is released, restores the
 * original `overflow` and `padding-right` on the document element.
 * On iOS Safari, removes the `touchmove` listener.
 */
const unlockScroll: Effect.Effect<void>
```
