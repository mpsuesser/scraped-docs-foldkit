---
url: https://foldkit.dev/api-reference/command-interruptible
title: "Command/Interruptible"
description: "API documentation for the Command/Interruptible module."
access_date: 2026-09-12T18:49:33.387Z
current_date: 2026-09-12T18:49:33.387Z
---

# Command/Interruptible

## Interfaces

### DefinitionNoArgs

interface

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/command/interruptible/interruptible.ts#L167)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/command/interruptible/interruptible.ts#L181)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/command/interruptible/interruptible.ts#L204)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/command/interruptible/interruptible.ts#L136)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/command/interruptible/interruptible.ts#L150)

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

### Outcome

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/command/interruptible/interruptible.ts#L10)

```
/**
 * The result of interrupting a key. `Outcome.Interrupted` means at least one
 * in-flight Command was stopped and will not dispatch its result Message.
 * `Outcome.NotFound` means no Command held the key. The interrupt operation
 * itself cannot fail.
 */
const Outcome: TaggedUnion<{
  Interrupted: {}
  NotFound: {}
}>
```
