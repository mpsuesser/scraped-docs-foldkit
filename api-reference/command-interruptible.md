---
url: https://foldkit.dev/api-reference/command-interruptible
title: "Command/Interruptible"
description: "API documentation for the Command/Interruptible module."
access_date: 2026-08-08T21:58:00.646Z
current_date: 2026-08-08T21:58:00.646Z
---

# Command/Interruptible

## Interfaces

### DefinitionNoArgs

interface

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/foldkit/src/command/interruptible/interruptible.ts#L180)

```
/**
 * An interruptible Command definition with no declared args; its key is the
 *  Command name. Call as `Definition()` to produce a Command instance; use
 *  `Definition.Interrupt` to build the Command that stops it.
 */
interface DefinitionNoArgs {
  [CommandDefinitionTypeId]: typeof CommandDefinitionTypeId
  Interrupt: InterruptDefinitionNoArgs<Name>
  name: Name
}
```

### DefinitionWithArgs

interface

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/foldkit/src/command/interruptible/interruptible.ts#L194)

```
/**
 * An interruptible Command definition with declared args and a key derived
 *  from them, namespaced by the Command name. Call as `Definition(args)` to
 *  produce a Command instance; use `Definition.Interrupt` to build the
 *  Command that stops every holder of a specific key.
 */
interface DefinitionWithArgs {
  [CommandDefinitionTypeId]: typeof CommandDefinitionTypeId
  Interrupt: InterruptDefinitionWithArgs<Name, KeyArgs>
  name: Name
}
```

### DefinitionWithArgsNameKeyed

interface

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/foldkit/src/command/interruptible/interruptible.ts#L217)

```
/**
 * An interruptible Command definition with declared args but no `toKey`, so
 *  its key is the Command name, exactly like the no-args form. Call as
 *  `Definition(args)` to produce a Command instance; use `Definition.Interrupt`
 *  to build the Command that stops it. Reach for this when a Command takes args
 *  yet at most one invocation is meaningfully in flight, so its invocations need
 *  nothing to distinguish them.
 */
interface DefinitionWithArgsNameKeyed {
  [CommandDefinitionTypeId]: typeof CommandDefinitionTypeId
  Interrupt: InterruptDefinitionNoArgs<Name>
  name: Name
}
```

### InterruptDefinitionNoArgs

interface

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/foldkit/src/command/interruptible/interruptible.ts#L149)

```
/**
 * An Interrupt Command definition derived from an interruptible Command
 *  definition with no declared args. Call as `Definition.Interrupt(toMessage)`
 *  to produce a Command that interrupts every holder of the definition's key.
 */
interface InterruptDefinitionNoArgs {
  [CommandDefinitionTypeId]: typeof CommandDefinitionTypeId
  name: `${Name}.Interrupt`
}
```

### InterruptDefinitionWithArgs

interface

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/foldkit/src/command/interruptible/interruptible.ts#L163)

```
/**
 * An Interrupt Command definition derived from an interruptible Command
 *  definition with declared args. Call as
 *  `Definition.Interrupt(keyArgs, toMessage)` to produce a Command that
 *  interrupts every holder of the key derived from `keyArgs`.
 */
interface InterruptDefinitionWithArgs {
  [CommandDefinitionTypeId]: typeof CommandDefinitionTypeId
  name: `${Name}.Interrupt`
}
```

## Constants

### Interrupted

const

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/foldkit/src/command/interruptible/interruptible.ts#L9)

```
/**
 * At least one in-flight Command held the interrupt key, and every holder
 *  has been stopped. Their declared result Messages are guaranteed never to
 *  dispatch.
 */
const Interrupted: CallableTaggedStruct<"Interrupted", {}>
```

### NotFound

const

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/foldkit/src/command/interruptible/interruptible.ts#L19)

```
/**
 * No Command holds the interrupt key: every target already completed (its
 *  result Message dispatched or will dispatch) or was never dispatched. The
 *  two cases are indistinguishable by design.
 */
const NotFound: CallableTaggedStruct<"NotFound", {}>
```

### Outcome

const

[source](https://github.com/foldkit/foldkit/blob/ea9c4f39b10a4db2bad82ee3dff3cb0ea0fa6bd7/packages/foldkit/src/command/interruptible/interruptible.ts#L29)

```
/**
 * The result of an Interrupt Command: Interrupted when at least one
 *  holder was stopped, NotFound when nothing held the key.
 *  Interruption itself cannot fail.
 */
const Outcome: Union<readonly [CallableTaggedStruct<"Interrupted", {}>, CallableTaggedStruct<"NotFound", {}>]>
```
