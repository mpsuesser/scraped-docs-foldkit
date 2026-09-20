---
url: https://foldkit.dev/api-reference/ui-dialog
title: "Ui/Dialog"
description: "API documentation for the Ui/Dialog module."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

# Ui/Dialog

## Functions

### boot

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L459)

```
/**
 * Creates a Dialog and opens it through the normal update path. Use the
 *  returned Model and Commands during application initialization so the
 *  initially visible Dialog acquires modal isolation, scroll locking, focus
 *  management, stack registration, and runtime-owned cleanup.
 */
(config: InitConfig): UpdateReturn
```

### close

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L467)

```
/** Programmatically closes the dialog. */
(model: Dialog.Model): UpdateReturn
```

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L492)

```
/**
 * Returns the framework-managed description id, the
 *  `-dialog-description` suffix on `model.id`.
 * 
 *  The primary path is spreading `RenderInfo`'s `description` onto your
 *  description element (`h.p([...description], [...])`), which carries this id
 *  for you. Reach for this helper only when you need the id as a value outside
 *  `toView`: a Command that calls `getElementById`, a cross-element
 *  `aria-describedby`, or a test. When the description is rendered, set
 *  `ViewInputs.hasDescription` so the dialog points at this id. Do not
 *  hand-roll the id string.
 */
(model: Dialog.Model): string
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L98)

```
/**
 * Creates a closed dialog model from a config. Use `boot` when the Dialog
 *  should open as the application starts.
 */
(config: InitConfig): Dialog.Model
```

### open

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L463)

```
/** Programmatically opens the dialog. */
(model: Dialog.Model): UpdateReturn
```

### titleId

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L480)

```
/**
 * Returns the framework-managed id the dialog's `aria-labelledby` points at,
 *  the `-dialog-title` suffix on `model.id`.
 * 
 *  The primary path is spreading `RenderInfo`'s `title` onto your heading
 *  (`h.h2([...title], [...])`), which carries this id for you. Reach for this
 *  helper only when you need the id as a value outside `toView`: a Command that
 *  calls `getElementById`, a cross-element `aria-describedby`, or a test. Do not
 *  hand-roll the id string.
 */
(model: Dialog.Model): string
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L341)

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L85)

```
/**
 * Configuration for creating a dialog model with `init`. The `id` must be
 *  non-empty and unique within the document: it keys the dialog element, its
 *  ARIA references, and the framework's per-dialog resource cleanup, so a
 *  duplicate or empty id breaks cleanup accounting.
 * 
 *  The dialog derives framework-managed ids from this `id`: `-dialog-title`,
 *  `-dialog-description`, and `-panel` (the animation panel). Spread
 *  `RenderInfo`'s `title` / `description` onto your heading and description
 *  elements rather than constructing those ids yourself.
 */
type InitConfig = Readonly<{
  focusSelector: string
  id: string
  isAnimated: boolean
}>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L536)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 *  - `dialog`: attributes for the native `<dialog>` element. Carries
 *    the id, ARIA labelling and modal state, `open` prop, positioning style, a `cancel` handler
 *    that prevents a file picker's native cancellation from closing the dialog
 *    while mapping `Dom.showDialog`'s Escape signal to `RequestedClose`,
 *    an `OnMount` acquisition that restores modal resources for an initially
 *    visible or development-preserved Dialog, and an `OnUnmount` backstop that
 *    releases framework hygiene (scroll lock, focus trap, background
 *    isolation, return focus) if the element is removed from the DOM while
 *    still open, such as navigating away from a route-keyed subtree.
 *    The consumer MUST render an `h.dialog(...)` element so the framework
 *    can open and close it, and so the unmount backstop can fire.
 *  - `backdrop`: attributes for the backdrop element. Includes the
 *    Animation data attributes and the `OnClick` handler that closes
 *    the dialog on outside-click (suppressed while a leave animation
 *    is in progress).
 *  - `panel`: attributes for the panel element. Includes the panel id
 *    (`${model.id}-panel`) and the Animation data attributes.
 *  - `title`: attributes for the accessible-name heading. Carries the
 *    framework-managed id the dialog's `aria-labelledby` points at. Spread
 *    onto your heading element (`h.h2([...title], [...])`) so labelling
 *    wires up without hand-rolling the id.
 *  - `description`: attributes for the description element. Carries the
 *    framework-managed id referenced by `aria-describedby` when
 *    `ViewInputs.hasDescription` is true. Spread onto your description element
 *    (`h.p([...description], [...])`).
 *  - `initialFocus`: attributes for the element that should receive focus when
 *    the dialog opens. Spread onto that element (`h.input([...initialFocus])`).
 *    A configured `focusSelector` (see `init`) takes precedence, and focus
 *    falls back to the default when no element carries the group.
 *  - `closeButton`: attributes for an in-panel close control such as a Cancel
 *    or dismiss button. Carries the `OnClick` handler that closes the
 *    dialog (suppressed while a leave animation is in progress). Spread
 *    onto your own button so a plain close needs no parent message. Sets
 *    `type="button"` so that a close control inside a `form` element in the
 *    panel closes without also submitting the form. Spread a later `h.Type`
 *    to override it.
 *  - `isVisible`: derived from `isOpen` and the Animation
 *    `transitionState`. The consumer renders backdrop + panel only
 *    while this is true.
 */
type RenderInfo = Readonly<{
  backdrop: ReadonlyArray<ChildAttribute>
  closeButton: ReadonlyArray<ChildAttribute>
  description: ReadonlyArray<ChildAttribute>
  dialog: ReadonlyArray<ChildAttribute>
  initialFocus: ReadonlyArray<ChildAttribute>
  isVisible: boolean
  panel: ReadonlyArray<ChildAttribute>
  title: ReadonlyArray<ChildAttribute>
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L548)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  hasDescription: boolean
  toView: (render: RenderInfo) => Html
}>
```

## Constants

### AcquireResources

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L173)

```
/**
 * Reacquires an initially visible Dialog's framework resources when its
 *  element mounts, including after development Model preservation restores an
 *  open Dialog without replaying initialization Commands. A successful
 *  acquisition also resumes a preserved animation transition from its current
 *  phase.
 */
const AcquireResources: MountDefinitionWithArgs<"AcquireResources", {
  focusSelector: String
  id: String
}, {
  _tag: "SucceededAcquireResources"
} | {
  _tag: "FailedAcquireResources"
}>
```

### CloseDialog

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L210)

```
/**
 * Calls `close()` on the native dialog element and unlocks page scroll when
 *  the close released the resources `ShowDialog` installed. A close that runs
 *  before the show has installed them leaves the lock alone. When the show
 *  then fails, it releases the lock itself. When the show succeeds, update
 *  closes the dialog again. If the dialog element is gone by the time the
 *  close runs, the Command calls `Dom.releaseDialogResources` instead. That
 *  releases the scroll lock, focus trap, return focus, and stack entry if the
 *  dialog still holds them. The background is restored before return focus.
 */
const CloseDialog: CommandDefinitionWithArgs<"CloseDialog", {
  id: String
}, Effect<{
  _tag: "CompletedCloseDialog"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L35)

```
/** Union of all messages the dialog component can produce. */
const Message: MessageUnion<{
  CompletedCloseDialog: {}
  CompletedReleaseDialogResources: {}
  FailedAcquireResources: {}
  FailedShowDialog: {}
  GotAnimationMessage: {
    message: MessageUnion<{
      CompletedWaitForPaint: {}
      EndedAnimation: {}
      Hid: {}
      Showed: {}
    }>
  }
  RequestedClose: {}
  RequestedOpen: {}
  SucceededAcquireResources: {}
  SucceededShowDialog: {}
  Unmounted: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L22)

```
/** Schema for the dialog component's state, tracking its unique ID, open/closed status, animation support, and animation lifecycle phase. */
const Model: Struct<{
  animation: Struct<{
    id: String
    isShowing: Boolean
    transitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
  }>
  id: String
  isAnimated: Boolean
  isOpen: Boolean
  maybeFocusSelector: Option<String>
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L65)

```
/** Union of out-messages the dialog component can produce. */
const OutMessage: MessageUnion<{
  Closed: {}
  Opened: {}
}>
```

### ReleaseDialogResources

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L227)

```
/**
 * Releases the framework hygiene the dialog holds while open (scroll lock,
 *  focus trap, return focus, stack entry, background isolation) when the
 *  element unmounts without a purposeful close. Calling it after
 *  `CloseDialog` released those resources is a no-op.
 */
const ReleaseDialogResources: CommandDefinitionWithArgs<"ReleaseDialogResources", {
  id: String
}, Effect<{
  _tag: "CompletedReleaseDialogResources"
}, never, never>>
```

### ShowDialog

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L161)

```
/**
 * Opens the native dialog element through `Dom.showDialog`, then locks page
 *  scroll when that call acquired the dialog resources. `Dom.showDialog`
 *  makes the background inert while leaving DevTools available as a separate
 *  developer overlay. It calls `show()` rather than `showModal()` so DevTools
 *  can stay interactive, layers the dialog with a high z-index, and traps
 *  focus. For an unhandled Escape on the topmost Dialog, the
 *  helper dispatches a `CustomEvent` named `cancel`; the Dialog view maps that
 *  signal to `RequestedClose` while suppressing native `cancel` events. The
 *  Dialog component supplies its own backdrop. If the dialog element is gone
 *  by the time the show runs, the Command reports `FailedShowDialog` without
 *  taking the scroll lock. The update function then closes the Model. The
 *  acquisition becomes uninterruptible after the committed element is found,
 *  so modal resources and the scroll lock cannot split. A concurrent
 *  lifecycle acquisition reuses the resources already held by the id.
 */
const ShowDialog: CommandDefinitionWithArgs<"ShowDialog", {
  focusSelector: String
  id: String
}, Effect<{
  _tag: "SucceededShowDialog"
} | {
  _tag: "FailedShowDialog"
}, never, never>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/dialog/index.ts#L557)

```
/**
 * Renders a headless modal dialog backed by the native `<dialog>` element.
 *  `ShowDialog` and the dialog's Mount open it through `Dom.showDialog`,
 *  isolate the background, trap focus, and handle Escape on the topmost
 *  dialog. The component supplies its own backdrop.
 */
const view: SubmodelView<Dialog.Model, {
  _tag: "RequestedOpen"
} | {
  _tag: "RequestedClose"
} | {
  _tag: "SucceededShowDialog"
} | {
  _tag: "FailedShowDialog"
} | {
  _tag: "SucceededAcquireResources"
} | {
  _tag: "FailedAcquireResources"
} | {
  _tag: "CompletedCloseDialog"
} | {
  _tag: "Unmounted"
} | {
  _tag: "CompletedReleaseDialogResources"
} | {
  _tag: "GotAnimationMessage"
  message: {
    _tag: "Showed"
  } | {
    _tag: "Hid"
  } | {
    _tag: "CompletedWaitForPaint"
  } | {
    _tag: "EndedAnimation"
  }
}, Readonly<{
  hasDescription: boolean
  toView: (render: RenderInfo) => Html
}>>
```
