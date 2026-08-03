---
url: https://foldkit.dev/api-reference/ui-dialog
title: "Ui/Dialog"
description: "API documentation for the Ui/Dialog module."
access_date: 2026-08-03T17:27:13.509Z
current_date: 2026-08-03T17:27:13.509Z
---

# Ui/Dialog

## Functions

### close

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L363)

```
/** Programmatically closes the dialog. */
(model: Dialog.Model): UpdateReturn
```

### descriptionId

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L386)

```
/**
 * Returns the framework-managed id the dialog's `aria-describedby` points at,
 *  the `-dialog-description` suffix on `model.id`.
 * 
 *  The primary path is spreading `RenderInfo`'s `description` onto your
 *  description element (`h.p([...description], [...])`), which carries this id
 *  for you. Reach for this helper only when you need the id as a value outside
 *  `toView`: a Command that calls `getElementById`, a cross-element
 *  `aria-describedby`, or a test. Do not hand-roll the id string.
 */
(model: Dialog.Model): string
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L141)

```
/** Creates an initial dialog model from a config. Defaults to closed and non-animated. */
(config: InitConfig): Dialog.Model
```

### open

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L359)

```
/** Programmatically opens the dialog. */
(model: Dialog.Model): UpdateReturn
```

### titleId

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L376)

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

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L255)

```
/** Processes a dialog message and returns the next model and commands. */
(
  model: Dialog.Model,
  message: {
    _tag: "RequestedOpen"
  } | {
    _tag: "RequestedClose"
  } | {
    _tag: "CompletedShowDialog"
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
  }
): UpdateReturn
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L128)

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
  isOpen: boolean
}>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L423)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 *  - `dialog`: attributes for the native `<dialog>` element. Carries
 *    the id, ARIA labelling, `open` prop, positioning style, the
 *    `OnCancel` handler that wires Escape to `RequestedClose`, and an
 *    `OnUnmount` backstop that releases framework hygiene (scroll lock,
 *    focus trap, return focus) if the element is removed from the DOM
 *    while still open, such as navigating away from a route-keyed subtree.
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
 *    framework-managed id the dialog's `aria-describedby` points at. Spread
 *    onto your description element (`h.p([...description], [...])`).
 *  - `initialFocus`: attributes for the element that should receive focus when
 *    the dialog opens. Spread onto that element (`h.input([...initialFocus])`).
 *    A configured `focusSelector` (see `init`) takes precedence, and focus
 *    falls back to the default when no element carries the group.
 *  - `closeButton`: attributes for an in-panel close control such as a Cancel
 *    or dismiss button. Carries the `OnClick` handler that closes the
 *    dialog (suppressed while a leave animation is in progress). Spread
 *    onto your own button so a plain close needs no parent message.
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

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L435)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  toView: (render: RenderInfo) => Html
}>
```

## Constants

### CloseDialog

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L190)

```
/** Calls `close()` on the native dialog element and unlocks page scroll. */
const CloseDialog: CommandDefinitionWithArgs<"CloseDialog", {
  id: String
}, Effect<{
  _tag: "CompletedCloseDialog"
}, never, never>>
```

### Closed

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L108)

```
/**
 * Sent once the dialog has transitioned to closed. Programmatic
 *  `Dialog.close` on an already-closed model is a no-op that does not
 *  re-emit; calling close while a leave animation is in progress is
 *  also a no-op.
 */
const Closed: CallableTaggedStruct<"Closed", {}>
```

### CompletedCloseDialog

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L48)

```
/** Sent when the close-dialog command completes. */
const CompletedCloseDialog: CallableTaggedStruct<"CompletedCloseDialog", {}>
```

### CompletedReleaseDialogResources

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L57)

```
/** Sent when the release-dialog-resources command completes. */
const CompletedReleaseDialogResources: CallableTaggedStruct<"CompletedReleaseDialogResources", {}>
```

### CompletedShowDialog

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L46)

```
/** Sent when the show-dialog command completes. */
const CompletedShowDialog: CallableTaggedStruct<"CompletedShowDialog", {}>
```

### GotAnimationMessage

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L61)

```
/** Wraps an Animation submodel message for delegation. */
const GotAnimationMessage: CallableTaggedStruct<"GotAnimationMessage", {
  message: Union<[CallableTaggedStruct<"Showed", {}>, CallableTaggedStruct<"Hid", {}>, CallableTaggedStruct<"CompletedWaitForPaint", {}>, CallableTaggedStruct<"EndedAnimation", {}>]>
}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L66)

```
/** Union of all messages the dialog component can produce. */
const Message: S.Union<[typeof RequestedOpen, typeof RequestedClose, typeof CompletedShowDialog, typeof CompletedCloseDialog, typeof Unmounted, typeof CompletedReleaseDialogResources, typeof GotAnimationMessage]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L29)

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

### Opened

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L102)

```
/**
 * Sent once the dialog has transitioned to open. Fires after `update`
 *  has processed `RequestedOpen` and `isOpen` reflects the new state.
 *  Programmatic `Dialog.open` on an already-open model is a no-op that
 *  does not re-emit.
 */
const Opened: CallableTaggedStruct<"Opened", {}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L111)

```
/** Union of out-messages the dialog component can produce. */
const OutMessage: Union<readonly [CallableTaggedStruct<"Opened", {}>, CallableTaggedStruct<"Closed", {}>]>
```

### ReleaseDialogResources

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L205)

```
/**
 * Releases the framework hygiene the dialog holds while open (scroll lock,
 *  focus trap, return focus, stack entry) when the element unmounts without a
 *  purposeful close. Idempotent: a no-op if the dialog already released its
 *  resources through `CloseDialog`.
 */
const ReleaseDialogResources: CommandDefinitionWithArgs<"ReleaseDialogResources", {
  id: String
}, Effect<{
  _tag: "CompletedReleaseDialogResources"
}, never, never>>
```

### RequestedClose

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L44)

```
/** Sent when the dialog should close (Escape key, backdrop click, or programmatic). */
const RequestedClose: CallableTaggedStruct<"RequestedClose", {}>
```

### RequestedOpen

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L42)

```
/** Sent when the dialog should open. Triggers the ShowDialog command. */
const RequestedOpen: CallableTaggedStruct<"RequestedOpen", {}>
```

### ShowDialog

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L176)

```
/**
 * Locks page scroll and opens the native dialog element through
 *  `Dom.showDialog`, which calls `show()` (not native `showModal()`) so other
 *  high-z-index overlays stay interactive. It layers the dialog with a high
 *  z-index, traps focus, and dispatches a `cancel` event on Esc. The Dialog
 *  component supplies its own backdrop.
 */
const ShowDialog: CommandDefinitionWithArgs<"ShowDialog", {
  focusSelector: String
  id: String
}, Effect<{
  _tag: "CompletedShowDialog"
}, never, never>>
```

### Unmounted

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L55)

```
/**
 * Sent when the native `<dialog>` element is removed from the DOM, the classic
 *  case being navigation away from a route-keyed subtree that contains the
 *  dialog. When the dialog still holds framework resources, `update` triggers
 *  the hygiene-only `ReleaseDialogResources` command and resets the model to a
 *  clean closed state. Does not emit `Closed` or run any consumer close
 *  Commands: it is a backstop, not the purposeful close.
 */
const Unmounted: CallableTaggedStruct<"Unmounted", {}>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/ui/src/dialog/index.ts#L443)

```
/**
 * Renders a headless dialog component backed by the native `<dialog>`
 *  element. `ShowDialog` opens it through `Dom.showDialog`, which uses `show()`
 *  (not native `showModal()`) with a high z-index, a focus trap, a
 *  component-supplied backdrop, and a `cancel` event dispatched on Esc.
 */
const view: SubmodelView<Dialog.Model, {
  _tag: "RequestedOpen"
} | {
  _tag: "RequestedClose"
} | {
  _tag: "CompletedShowDialog"
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
  toView: (render: RenderInfo) => Html
}>>
```
