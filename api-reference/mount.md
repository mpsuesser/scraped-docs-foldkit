---
url: https://foldkit.dev/api-reference/mount
title: "Mount"
description: "API documentation for the Mount module."
access_date: 2026-08-31T07:29:25.100Z
current_date: 2026-08-31T07:29:25.100Z
---

# Mount

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/mount/index.ts#L247)

### defineStream

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/mount/index.ts#L404)

## Types

### MountAction

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/mount/index.ts#L46)

```
/**
 * A named, type-constrained per-element side effect, optionally carrying the
 *  args used to construct it. The runtime invokes `f` with the live `Element`
 *  when the element mounts, and dispatches each Message emitted by the
 *  returned Stream. The Stream's scope is tied to the element's lifetime: when
 *  the element unmounts, the runtime interrupts the fiber, which closes the
 *  Stream's scope and runs any registered `acquireRelease` finalizers.
 * 
 *  Authors don't construct this shape directly. `Mount.define` builds it from
 *  an `execute` returning `Effect<Message>` for the one-shot case; only
 *  `Mount.defineStream` exposes the raw Stream shape for continuous-event
 *  cases.
 */
type MountAction = Readonly<{
  args: Record<string, unknown>
  f: (element: Element) => Stream.Stream<Message, E>
  name: string
}>
```

### MountDefinition

type

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/mount/index.ts#L80)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/mount/index.ts#L53)

```
/** A Mount definition for a Mount with no declared args. Call as `Definition()` to produce a MountAction. */
interface MountDefinitionNoArgs {
  [MountDefinitionTypeId]: typeof MountDefinitionTypeId
  name: Name
}
```

### MountDefinitionWithArgs

interface

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/mount/index.ts#L63)

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

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/mount/index.ts#L28)

```
/** Type-level brand for MountDefinition values. */
const MountDefinitionTypeId: unique symbol
```

### mapMessage

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/mount/index.ts#L459)

```
/**
 * Lifts a `MountAction` from one Message universe to another by mapping its
 *  dispatched Messages through a transform. Used by Submodel components to
 *  emit lifecycle action results into the parent's Message union via the
 *  consumer-supplied `toParentMessage` lift. Preserves `name` and `args`.
 */
const mapMessage: (f: (message: A) => B) => (action: MountAction<A, E>) => MountAction<B, E>
```
