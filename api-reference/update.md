---
url: https://foldkit.dev/api-reference/update
title: "Update"
description: "API documentation for the Update module."
access_date: 2026-08-04T03:55:42.119Z
current_date: 2026-08-04T03:55:42.119Z
---

# Update

## Functions

### refresh

function

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/update/update.ts#L135)

```
/**
 * Turns a Refreshable into an update step that revalidates one
 *  cache: read the entry, ask `revalidate` whether it should transition,
 *  and only when it says yes write the transitioned state and emit the
 *  load Command. When `revalidate` returns `None` (a missing entry, or a
 *  state with nothing to revalidate) the step returns `[model, []]`: same
 *  Model, no Command. That one rule is what makes blanket revalidation
 *  safe, because only the caches that actually hold data reload.
 * 
 *  ```ts
 *  const refreshAllNotes = refresh({
 *    read: model => Option.some(model.allNotes),
 *    revalidate: AsyncData.revalidate,
 *    write: (model, nextAllNotes) => evo(model, { allNotes: () => nextAllNotes }),
 *    load: LoadAllNotes(),
 *  })
 *  ```
 */
<Model, Message, A, E, R = never>(refreshable: Refreshable<Model, Message, A, E, R>): Step<Model, Message, R>
```

## Types

### Commands

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/update/update.ts#L16)

```
/**
 * The Commands half of an update return: every Command the update wants
 *  the runtime to run, in order. `R` is the services the Commands need
 *  and defaults to `never` for applications without resources.
 * 
 *  Each update module pins its concrete types once and uses the alias
 *  throughout; the root update and every Submodel define their own:
 * 
 *  ```ts
 *  export type Commands = Update.Commands<Message, AppServices>
 *  ```
 */
type Commands = ReadonlyArray<Command<Message, never, R>>
```

### Refreshable

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/update/update.ts#L112)

```
/**
 * The four capabilities that make one cache field revalidatable.
 * 
 *  - `read`: gets the field's AsyncData out of the Model. Returns an
 *    `Option` because keyed caches miss (`HashMap.get`); single fields
 *    wrap in `Option.some`.
 *  - `revalidate`: decides whether and how the entry transitions.
 *    Usually exactly `AsyncData.revalidate` (refresh after a mutation:
 *    only `Success` and `Stale` move to `Refreshing`). Pass
 *    `AsyncData.revalidateOrLoad` instead for load-on-entry semantics.
 *  - `write`: puts the transitioned entry back into the Model.
 *  - `load`: the Command that refetches the data.
 */
type Refreshable = Readonly<{
  load: Command<Message, never, R>
  read: (model: Model) => Option.Option<AsyncData<A, E>>
  revalidate: (current: AsyncData<A, E>) => Option.Option<AsyncData<A, E>>
  write: (model: Model, next: AsyncData<A, E>) => Model
}>
```

### Return

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/update/update.ts#L30)

```
/**
 * The pair every update function returns: the next Model and the
 *  Commands to run.
 * 
 *  Each update module pins its concrete types once and aliases the
 *  result, the root update and every Submodel alike:
 * 
 *  ```ts
 *  export type UpdateReturn = Update.Return<Model, Message>
 *  export const withUpdateReturn = M.withReturnType<UpdateReturn>()
 *  ```
 */
type Return = readonly [Model, Commands<Message, R>]
```

### ReturnWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/update/update.ts#L40)

```
/**
 * The return shape of an update that also surfaces an OutMessage to its
 *  parent. The third element is an `Option`: the update always returns
 *  the channel, and `None` means there is nothing for the parent this
 *  time. Named for the shape, not the caller: a Submodel without an
 *  OutMessage channel returns a plain Return.
 */
type ReturnWithOutMessage = readonly [Model, Commands<Message, R>, Option.Option<OutMessage>]
```

### Step

type

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/update/update.ts#L50)

```
/**
 * One self-contained edit to the Model paired with the Commands to run:
 *  the unit combine composes. A step that needs arguments is a
 *  function returning a Step (`(noteId: NoteId) => Step<...>`).
 */
type Step = (model: Model) => Return<Model, Message, R>
```

## Constants

### combine

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/update/update.ts#L79)

```
/**
 * Composes a list of update steps into one. Each step runs against the
 *  Model the previous step produced, and every step's Commands are
 *  concatenated into a single batch, in step order.
 * 
 *  Dual: call it data-first with the Model to run the steps now
 *  (`combine(model, steps)` returns a Return), or data-last with
 *  only the steps to build a composable Step that runs later
 *  (`combine(steps)`, for a `pipe` or a nested step list).
 * 
 *  Steps only ever accumulate Commands; a step cannot cancel or replace
 *  another step's Commands, and no Command runs during the fold. The
 *  runtime runs the batch after update returns. `combine([])` returns
 *  `[model, []]`.
 * 
 *  ```ts
 *  SucceededUpdateNote: ({ note }) =>
 *    combine(model, [
 *      replaceNoteInCaches(note),
 *      refreshNote(note.id),
 *      refreshAllNotes,
 *      refreshNotebookNotes(note.maybeNotebookId),
 *      ...(hasMoved ? [refreshNotebookNotes(previousNotebookId)] : []),
 *      showToast('Success', `Updated ${note.title}`),
 *    ])
 *  ```
 */
const combine: (steps: readonly Array<Step<Model, Message, R>>) => Step<Model, Message, R>
```
