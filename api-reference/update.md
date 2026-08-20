---
url: https://foldkit.dev/api-reference/update
title: "Update"
description: "API documentation for the Update module."
access_date: 2026-08-20T02:21:49.544Z
current_date: 2026-08-20T02:21:49.544Z
---

# Update

## Functions

### refresh

function

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L135)

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

### ChildFold

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L163)

```
/**
 * The four capabilities that fold one child Submodel's update into the
 *  parent, for a child without an OutMessage channel.
 * 
 *  - `update`: the child update function to run.
 *  - `read`: the getter half of the lens onto the child: reads the child
 *    Model from the parent Model. Returns an `Option` because a child
 *    may not be mounted (for example a page behind a route or a keyed
 *    collection miss); a single always-present field wraps in
 *    `Option.some`.
 *  - `write`: the setter half of the lens: writes the updated child
 *    Model back into the parent Model.
 *  - `toParentMessage`: lifts a child Message into the parent's Message,
 *    the same contract `h.submodel` takes for the view half. Always the
 *    child's `Got*` wrapper: `message => GotSearchMessage({ message })`.
 */
type ChildFold = Readonly<{
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  update: (childModel: ChildModel, input: Input) => Return<ChildModel, ChildMessage, R>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildFoldWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L237)

```
/**
 * ChildFold for a child whose update returns
 *  ReturnWithOutMessage, adding the fifth capability:
 * 
 *  - `foldOutMessage`: folds the child's OutMessage into the parent as a
 *    Step. The Step receives the parent Model with the child
 *    already written back, and its Commands follow the child's in the
 *    returned batch. Match on the OutMessage tag inside
 *    (`M.tagsExhaustive`), and build a multi-step fold with
 *    combine. Takes an optional second parameter, a
 *    FoldContext of lifters bound to `toParentMessage`, for a
 *    Command the Step returns whose result is the child's Message.
 */
type ChildFoldWithOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => Step<ParentModel, ParentMessage, R>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  update: (childModel: ChildModel, input: Input) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, R>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildFoldWithParentOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L272)

```
/**
 * ChildFoldWithOutMessage for a parent that is itself a
 *  Submodel, so the fold's result carries the parent's own OutMessage
 *  channel as a third tuple element. Adds:
 * 
 *  - `toParentOutMessage`: lifts the child's OutMessage into the
 *    parent's own OutMessage; `None` passes nothing upward. When the
 *    child returns no OutMessage the fold's third element is `None`.
 *  - `foldOutMessage` stays available for a parent that also updates
 *    its own state from the child's OutMessage, and is optional here.
 * 
 *  A parent Submodel embedding a child with no OutMessage channel needs
 *  no config at all: spread the plain fold into its return,
 *  `[...foldStartDate(model, message), Option.none()]`.
 */
type ChildFoldWithParentOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => Step<ParentModel, ParentMessage, R>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  toParentOutMessage: (outMessage: ChildOutMessage) => Option.Option<ParentOutMessage>
  update: (childModel: ChildModel, input: Input) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, R>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildStepFold

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L532)

```
/**
 * ChildFold for an entry point that takes nothing but the child
 *  Model, such as `Dialog.close` or a Submodel's `informRouteChanged` that
 *  derives everything it needs from its own state. There is no `input`, so
 *  foldChildStep returns the Step itself rather than a dual
 *  Fold.
 */
type ChildStepFold = Readonly<{
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  update: (childModel: ChildModel) => Return<ChildModel, ChildMessage, R>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildStepFoldWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L549)

```
/**
 * ChildStepFold for an entry point whose return carries the child's
 *  OutMessage channel, adding `foldOutMessage`. It behaves exactly as it does
 *  in ChildFoldWithOutMessage, down to the optional second parameter,
 *  a FoldContext of lifters bound to `toParentMessage`.
 */
type ChildStepFoldWithOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => Step<ParentModel, ParentMessage, R>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  update: (childModel: ChildModel) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, R>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### Commands

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L16)

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

### Fold

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L329)

```
/**
 * The dual function foldChild returns. Data-first runs the
 *  fold now (`fold(model, input)` returns a Return); data-last
 *  builds a composable Step (`fold(input)`, for
 *  combine).
 */
type Fold = (model: ParentModel, input: Input) => Return<ParentModel, ParentMessage, R>
```

### FoldContext

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L217)

```
/**
 * The lifters a `foldOutMessage` receives as its second parameter,
 *  already bound to the fold config's `toParentMessage`.
 * 
 *  The fold lifts the Commands the child's `update` returns on its own.
 *  This covers the other case: a Command the parent returns on the
 *  child's behalf from the OutMessage Step, whose result Message is the
 *  child's and therefore still needs wrapping, such as a parent handling
 *  a child's `Requested*` fact by returning the child's Command that
 *  fulfills it, built with context only the parent holds.
 * 
 *  The lifters apply the same lift the fold gives the child's own
 *  Commands, so the Step writes no `Command.mapMessage` call and keeps
 *  no second copy of the wrapper, and the mapping stays recorded on the
 *  Command for `Story.Command.resolve` and `Scene.Command.resolve`.
 * 
 *  The annotated standalone const takes both parameters, so the match
 *  moves from `M.type` to `M.value` on the OutMessage:
 * 
 *  ```ts
 *  const foldLoginOutMessage: (
 *    outMessage: Login.OutMessage,
 *    context: Update.FoldContext<Login.Message, Message>,
 *  ) => Update.Step<Model, Message> = (outMessage, { liftCommand }) =>
 *    M.value(outMessage).pipe(
 *      M.withReturnType<Update.Step<Model, Message>>(),
 *      M.tagsExhaustive({
 *        RequestedMagicLink: ({ email }) => model => [
 *          model,
 *          [
 *            liftCommand(
 *              Login.SendMagicLink({ email, redirectRoute: model.route }),
 *            ),
 *          ],
 *        ],
 *      }),
 *    )
 *  ```
 */
type FoldContext = Readonly<{
  liftCommand: (command: Command<ChildMessage, E, R>) => Command<ParentMessage, E, R>
  liftCommands: (commands: ReadonlyArray<Command<ChildMessage, E, R>>) => ReadonlyArray<Command<ParentMessage, E, R>>
}>
```

### FoldWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L338)

```
/**
 * Fold for a ChildFoldWithParentOutMessage: the
 *  data-first form returns a ReturnWithOutMessage and the
 *  data-last form builds a StepWithOutMessage, so the fold slots
 *  directly into a parent that is itself a Submodel.
 */
type FoldWithOutMessage = (model: ParentModel, input: Input) => ReturnWithOutMessage<ParentModel, ParentMessage, ParentOutMessage, R>
```

### Refreshable

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L112)

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

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L30)

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

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L40)

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

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L50)

```
/**
 * One self-contained edit to the Model paired with the Commands to run:
 *  the unit combine composes. A step that needs arguments is a
 *  function returning a Step (`(noteId: NoteId) => Step<...>`).
 */
type Step = (model: Model) => Return<Model, Message, R>
```

### StepWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L321)

```
/**
 * Step for an update that also surfaces an OutMessage to its
 *  parent: maps a Model to a ReturnWithOutMessage over the same
 *  Model.
 */
type StepWithOutMessage = (model: Model) => ReturnWithOutMessage<Model, Message, OutMessage, R>
```

## Constants

### combine

const

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L79)

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

### foldChild

const

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L412)

```
/**
 * Folds a child Submodel's update into the parent: the update half of
 *  embedding a child, complementing `h.submodel` on the view half. Give
 *  it the facts that vary per child (a ChildFold, or a
 *  ChildFoldWithOutMessage when the child's update returns
 *  OutMessages) and it returns a dual Fold:
 * 
 *  ```ts
 *  const foldSearch = Update.foldChild({
 *    update: Search.update,
 *    read: (model: Model) => Option.some(model.search),
 *    write: (model, nextSearch) => evo(model, { search: () => nextSearch }),
 *    toParentMessage: message => GotSearchMessage({ message }),
 *  })
 * 
 *  // in the parent update
 *  GotSearchMessage: ({ message }) => foldSearch(model, message),
 *  ```
 * 
 *  The fold runs `update` against the child Model `read` returns, writes
 *  the child back, and lifts the child's Commands through
 *  `toParentMessage`. When `read` returns `None` the fold returns
 *  `[model, []]`: a Message for an unmounted child is a no-op. When the
 *  child's update returns an OutMessage, `foldOutMessage` runs against
 *  the Model with the child already written back, and its Commands
 *  follow the child's in the returned batch.
 * 
 *  `foldOutMessage` takes an optional second parameter, a
 *  FoldContext carrying `liftCommand` and `liftCommands` bound to
 *  this config's `toParentMessage`. Reach for it when the Step returns a
 *  Command that produces the child's Message, such as an animating
 *  component's overridable leave Command.
 * 
 *  A parent that is itself a Submodel passes a
 *  ChildFoldWithParentOutMessage and receives a
 *  FoldWithOutMessage, whose results carry the parent's own
 *  OutMessage channel as a third element.
 * 
 *  An entry point that takes nothing but the child Model, such as
 *  `Dialog.close`, has no input to pass: fold it with
 *  foldChildStep, which returns the Step directly.
 * 
 *  `update` closes over per-dispatch context, and the data-last form
 *  composes with combine, here to put a navigation Command ahead
 *  of the child's:
 * 
 *  ```ts
 *  const enterJoinedRoom = (roomId: string, player: Player): UpdateStep =>
 *    Update.combine([
 *      model => [model, [NavigateToRoom({ roomId })]],
 *      Update.foldChild({
 *        update: (room: Room.Model, joinedPlayer: Player) =>
 *          Room.informJoined(room, joinedPlayer, { roomId }),
 *        read: readRoom,
 *        write: writeRoom,
 *        toParentMessage: toGotRoomMessage,
 *      })(player),
 *    ])
 *  ```
 */
const foldChild: (childFold: ChildFoldWithParentOutMessage<ParentModel, ParentMessage, ChildModel, Input, ChildMessage, ChildOutMessage, ParentOutMessage, R>) => FoldWithOutMessage<ParentModel, ParentMessage, Input, ParentOutMessage, R>
```

### foldChildStep

const

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/update/update.ts#L616)

```
/**
 * Folds a child entry point that takes nothing but the child Model, and
 *  returns the Step directly. Everything else matches
 *  foldChild: the child is read, updated, and written back, its
 *  Commands are lifted through `toParentMessage`, a `None` from `read` makes
 *  the Step a no-op, and `foldOutMessage` runs against the Model with the
 *  child already written back.
 * 
 *  Reach for it wherever a Submodel exposes a no-argument entry point, so the
 *  call site composes with combine as a plain Step and never invents
 *  an input the child does not take:
 * 
 *  ```ts
 *  const foldMobileMenuDialogClose = Update.foldChildStep({
 *    update: Dialog.close,
 *    read: readMobileMenuDialog,
 *    write: writeMobileMenuDialog,
 *    toParentMessage: toGotMobileMenuDialogMessage,
 *    foldOutMessage: foldMobileMenuDialogOutMessage,
 *  })
 * 
 *  // in the parent update
 *  Update.combine(model, [writeRouteFields, foldMobileMenuDialogClose])
 *  ```
 * 
 *  `foldOutMessage` takes the same optional second parameter `foldChild`'s
 *  does, a FoldContext carrying `liftCommand` and `liftCommands` bound
 *  to this config's `toParentMessage`, for a Command the Step returns whose
 *  result is the child's Message.
 * 
 *  A parent that is itself a Submodel, and so needs its own OutMessage
 *  channel on the result, uses foldChild.
 */
const foldChildStep: (childFold: ChildStepFoldWithOutMessage<ParentModel, ParentMessage, ChildModel, ChildMessage, ChildOutMessage, R>) => Step<ParentModel, ParentMessage, R>
```
