---
url: https://foldkit.dev/api-reference/mount
title: "Mount"
description: "API documentation for the Mount module."
access_date: 2026-09-02T16:31:11.406Z
current_date: 2026-09-02T16:31:11.406Z
---

# Mount

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L325)

### defineStream

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L499)

## Types

### MountAction

type

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L66)

```
/**
 * A named, type-constrained per-element side effect, optionally carrying the
 *  args used to construct it. The runtime invokes `f` with the live `Element`
 *  and required view-state Stream when the element mounts. A Mount acquired by
 *  a live render keeps live dispatch, while one acquired by a historical
 *  render uses no-op dispatch. When resume reuses a replay-created element,
 *  the runtime releases its historical Mount before starting the live action.
 *  Otherwise the Stream's scope is tied to the element's lifetime: when the
 *  element unmounts, the runtime interrupts the fiber, which closes the
 *  Stream's scope and runs any registered `acquireRelease` finalizers.
 * 
 *  Authors don't construct this shape directly. `Mount.define` builds it from
 *  an `execute` returning `Effect<Message>` for the one-shot case; only
 *  `Mount.defineStream` exposes the raw Stream shape for continuous-event
 *  cases.
 */
type MountAction = Readonly<{
  args: Record<string, unknown>
  f: (element: Element, viewStateChanges: Stream.Stream<ViewState>) => Stream.Stream<Message, E>
  name: string
}>
```

### MountDefinition

type

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L109)

```
/**
 * A Mount definition created with `Mount.define` or `Mount.defineStream`.
 *  Union over the no-args and with-args shapes; consumers that only need
 *  name/identity can accept this.
 */
type MountDefinition = MountDefinitionNoArgs<Name, ResultMessage> | MountDefinitionWithArgs<Name, any, ResultMessage>
```

## Interfaces

### MountDefinitionNoArgs

interface

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L76)

```
/** A Mount definition for a Mount with no declared args. Call as `Definition()` to produce a MountAction. */
interface MountDefinitionNoArgs {
  [MountDefinitionTypeId]: typeof MountDefinitionTypeId
  name: Name
}
```

### MountDefinitionWithArgs

interface

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L89)

```
/** A Mount definition for a Mount with declared args. Call as `Definition(args)` to produce a MountAction. */
interface MountDefinitionWithArgs {
  [MountDefinitionTypeId]: typeof MountDefinitionTypeId
  name: Name
}
```

## Constants

### MountDefinitionTypeId

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L45)

```
/** Type-level brand for MountDefinition values. */
const MountDefinitionTypeId: unique symbol
```

### ViewState

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L29)

```
/**
 * The state of the DOM currently owned by the Foldkit renderer. `Live` means
 *  it represents the current live Model. `Paused` means time travel has
 *  installed a historical view while the live application continues running.
 */
const ViewState: Literals<readonly ["Live", "Paused"]>
```

### liveViewStateChanges

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L162)

```
/**
 * A never-ending view-state Stream for renderers without time travel.
 *  It emits `Live` immediately and never completes. Custom renderers and
 *  low-level MountAction wrappers can pass it as the required second argument
 *  to `MountAction.f` when the rendered view is always live.
 */
const liveViewStateChanges: Stream.Stream<ViewState>
```

### mapMessage

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/mount/index.ts#L562)

```
/**
 * Lifts a `MountAction` from one Message universe to another by mapping its
 *  dispatched Messages through a transform. Used by Submodel components to
 *  emit lifecycle action results into the parent's Message union via the
 *  consumer-supplied `toParentMessage` lift. Preserves `name` and `args`.
 */
const mapMessage: (f: (message: A) => B) => (action: MountAction<A, E>) => MountAction<B, E>
```
