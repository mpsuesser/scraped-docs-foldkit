---
url: https://foldkit.dev/api-reference/update
title: "Update"
description: "API documentation for the Update module."
access_date: 2026-09-12T18:49:33.387Z
current_date: 2026-09-12T18:49:33.387Z
---

# Update

## Functions

### refresh

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L197)

```
/**
 * Turns a Refreshable into an update step that revalidates one
 *  cache: read the entry, ask `revalidate` whether it should transition,
 *  and only when it says yes write the transitioned state and emit the
 *  load Command. When `revalidate` returns `None` (a missing entry, or a
 *  state with nothing to revalidate) the step returns `{ model }`: same
 *  Model, no Command. A handler can list every affected cache, and only the
 *  caches that currently hold data reload.
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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L228)

```
/**
 * The four capabilities that fold one child Submodel's update into the
 *  parent, for a child whose update cannot emit an OutMessage.
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

### ChildFoldWithDerivedParentOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L341)

```
/**
 * ChildFoldWithOutMessage for a parent that derives its own
 *  OutMessage while folding the child's. The returned
 *  StepWithOutMessage receives the parent Model with the child already
 *  written back. Use this shape when no child OutMessage is forwarded one to
 *  one, so the fold needs no `toParentOutMessage` adapter.
 */
type ChildFoldWithDerivedParentOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => StepWithOutMessage<NoInfer<ParentModel>, OutMessageStepMessage, ParentOutMessage, OutMessageStepRequirements>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  toParentOutMessage: never
  update: (childModel: ChildModel, input: Input) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, ChildRequirements>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildFoldWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L303)

```
/**
 * ChildFold for a child whose update returns
 *  ReturnWithOutMessage, adding the fifth capability:
 * 
 *  - `foldOutMessage`: folds the child's OutMessage into the parent as a
 *    Step. The Step receives the parent Model with the child
 *    already written back, and its Commands follow the child's in the
 *    returned batch. Match on the OutMessage tag through its union matcher,
 *    and build a multi-step fold with combine. Takes an optional
 *    second parameter, a
 *    FoldContext of lifters bound to `toParentMessage`, for a
 *    Command the Step returns whose result is the child's Message. Parent Model
 *    inference comes from `read` and `write`; the child wrapper and OutMessage
 *    Step infer their Message and service requirements independently, and the
 *    resulting Fold requires their unions.
 */
type ChildFoldWithOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => Step<NoInfer<ParentModel>, OutMessageStepMessage, OutMessageStepRequirements>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  update: (childModel: ChildModel, input: Input) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, ChildRequirements>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildFoldWithParentOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L396)

```
/**
 * ChildFoldWithOutMessage for a parent that is itself a
 *  Submodel, so the fold can return the parent's own OutMessage. Adds:
 * 
 *  - `toParentOutMessage`: lifts the child's OutMessage into the
 *    parent's own OutMessage. Return `undefined` for a named child variant
 *    that stops at this parent. When the child returns no OutMessage, the fold
 *    omits `outMessage`.
 *  - `foldOutMessage` stays available for a parent that also updates
 *    its own state from the child's OutMessage, and is optional here.
 *    It may emit a derived parent OutMessage. That OutMessage replaces the
 *    one-to-one lift for the dispatch. When the Step emits nothing, the lift
 *    runs as usual.
 * 
 *  Use this shape only when at least one child OutMessage should continue to
 *  the current Submodel's parent. If every child OutMessage stops here, use
 *  ChildFoldWithDerivedParentOutMessage when the fold derives its own
 *  OutMessage, or ChildFoldWithOutMessage when it does not. When
 *  provided, `foldOutMessage` still handles each variant locally, including
 *  variants that continue upward.
 */
type ChildFoldWithParentOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => StepWithOutMessage<NoInfer<ParentModel>, OutMessageStepMessage, DerivedParentOutMessage, OutMessageStepRequirements>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  toParentOutMessage: (outMessage: ChildOutMessage) => ParentOutMessage | undefined
  update: (childModel: ChildModel, input: Input) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, ChildRequirements>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildStepFold

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L723)

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

### ChildStepFoldWithDerivedParentOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L775)

```
/**
 * ChildStepFoldWithOutMessage for a parent that derives its own
 *  OutMessage while folding the child's. This is the no-argument counterpart
 *  to ChildFoldWithDerivedParentOutMessage.
 */
type ChildStepFoldWithDerivedParentOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => StepWithOutMessage<NoInfer<ParentModel>, OutMessageStepMessage, ParentOutMessage, OutMessageStepRequirements>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  toParentOutMessage: never
  update: (childModel: ChildModel) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, ChildRequirements>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildStepFoldWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L741)

```
/**
 * ChildStepFold for an entry point whose return carries the child's
 *  OutMessage channel, adding `foldOutMessage`. It behaves exactly as it does
 *  in ChildFoldWithOutMessage, down to the optional second parameter,
 *  a FoldContext of lifters bound to `toParentMessage`, and combines
 *  the child update and OutMessage Step Message and service requirements.
 */
type ChildStepFoldWithOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => Step<NoInfer<ParentModel>, OutMessageStepMessage, OutMessageStepRequirements>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  update: (childModel: ChildModel) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, ChildRequirements>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### ChildStepFoldWithParentOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L823)

```
/**
 * ChildStepFoldWithOutMessage for a parent that is itself a
 *  Submodel. `toParentOutMessage` turns the child's OutMessage into the
 *  parent's OutMessage. Return `undefined` for a named child variant that
 *  stops at this parent. `foldOutMessage` remains available when the parent
 *  also updates its own state from the child's OutMessage. A derived
 *  OutMessage from that Step replaces the one-to-one lift for the dispatch.
 *  When the Step emits nothing, the lift runs as usual.
 * 
 *  Use this shape only when at least one child OutMessage should continue to
 *  the current Submodel's parent. If every child OutMessage stops here, use
 *  ChildStepFoldWithDerivedParentOutMessage when the fold derives its
 *  own OutMessage, or ChildStepFoldWithOutMessage when it does not.
 *  When provided, `foldOutMessage` still handles each variant locally,
 *  including variants that continue upward.
 */
type ChildStepFoldWithParentOutMessage = Readonly<{
  foldOutMessage: (outMessage: ChildOutMessage, context: FoldContext<ChildMessage, ParentMessage>) => StepWithOutMessage<NoInfer<ParentModel>, OutMessageStepMessage, DerivedParentOutMessage, OutMessageStepRequirements>
  read: (model: ParentModel) => Option.Option<ChildModel>
  toParentMessage: (message: ChildMessage) => ParentMessage
  toParentOutMessage: (outMessage: ChildOutMessage) => ParentOutMessage | undefined
  update: (childModel: ChildModel) => ReturnWithOutMessage<ChildModel, ChildMessage, ChildOutMessage, ChildRequirements>
  write: (model: ParentModel, nextChildModel: ChildModel) => ParentModel
}>
```

### Commands

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L16)

```
/**
 * The Commands collection an update return may include. The collection keeps
 *  the order in which the update returned them, but the runtime forks the
 *  Commands independently. `R` is the services the Commands need and defaults
 *  to `never` for applications without resources.
 * 
 *  Name an alias when a module reuses the same Message and service types:
 * 
 *  ```ts
 *  export type Commands = Update.Commands<Message, AppServices>
 *  ```
 */
type Commands = ReadonlyArray<Command<Message, never, R>>
```

### Fold

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L461)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L280)

```
/**
 * The lifters a `foldOutMessage` receives as its second parameter,
 *  already bound to the fold config's `toParentMessage`.
 * 
 *  The fold already lifts the Commands returned by the child's `update`.
 *  Use these lifters for a Command returned by the parent's OutMessage Step
 *  when that Command still produces the child's Message. For example, the
 *  parent may handle a child's `Requested*` fact by returning a child Command
 *  built with routing context only the parent holds.
 * 
 *  The lifters apply the same lift the fold gives the child's own
 *  Commands, so the Step writes no `Command.mapMessage` call and keeps
 *  no second copy of the wrapper, and the mapping stays recorded on the
 *  Command for `Story.Command.resolve` and `Scene.Command.resolve`.
 * 
 *  The annotated standalone const takes both parameters, so pass the
 *  OutMessage value to its union matcher:
 * 
 *  ```ts
 *  const foldLoginOutMessage = (
 *    outMessage: Login.OutMessage,
 *    { liftCommand }: Update.FoldContext<Login.Message, Message>,
 *  ) =>
 *    Login.OutMessage.match<Update.Step<Model, Message>>(outMessage, {
 *      RequestedMagicLink:
 *        ({ email }) =>
 *        model => ({
 *          model,
 *          commands: [
 *            liftCommand(
 *              Login.SendMagicLink({ email, redirectRoute: model.route }),
 *            ),
 *          ],
 *        }),
 *    })
 *  ```
 */
type FoldContext = Readonly<{
  liftCommand: (command: Command<ChildMessage, E, R>) => Command<ParentMessage, E, R>
  liftCommands: (commands: ReadonlyArray<Command<ChildMessage, E, R>>) => ReadonlyArray<Command<ParentMessage, E, R>>
}>
```

### FoldWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L470)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L174)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L37)

```
/**
 * The record an update returns when it cannot emit an OutMessage: the next
 *  Model and any Commands to run.
 * 
 *  Inline the type when a matcher is its only use:
 * 
 *  ```ts
 *  export const update = (model: Model, message: Message) =>
 *    Message.match<Update.Return<Model, Message>>(message, {
 *      ClickedSave: () => ({ model, commands: [Save()] }),
 *      SucceededSave: ({ note }) => ({
 *        model: evo(model, { note: () => note }),
 *      }),
 *    })
 *  ```
 * 
 *  Give it a local `UpdateReturn` alias when another matcher or helper in the
 *  module needs the same type.
 */
type Return = Readonly<{
  commands: Commands<Message, R>
  model: Model
  outMessage: never
}>
```

### ReturnWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L51)

```
/**
 * The return shape of an update that can also surface an OutMessage to its
 *  parent. Omit `commands` when the update statically creates none. Return a
 *  computed Commands collection directly, even when it may be empty. Omit
 *  `outMessage` when the update emitted nothing. A Submodel that cannot emit
 *  an OutMessage returns Return instead.
 */
type ReturnWithOutMessage = Readonly<{
  commands: Commands<Message, R>
  model: Model
  outMessage: OutMessage
}>
```

### Step

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L104)

```
/**
 * One self-contained edit to the Model paired with the Commands to run:
 *  the unit combine composes. A step that needs arguments is a
 *  function returning a Step
 *  (`(noteId: NoteId) => Update.Step<Model, Message>`).
 */
type Step = (model: Model) => Return<Model, Message, R>
```

### StepWithOutMessage

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L111)

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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L140)

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
 *  `{ model }`.
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

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L547)

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
 *  `{ model }`: a Message for an unmounted child is a no-op. When the
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
 *  A parent that is itself a Submodel receives a
 *  FoldWithOutMessage when `foldOutMessage` emits a derived parent
 *  OutMessage. Add `toParentOutMessage` only when at least one child OutMessage
 *  should continue to the current Submodel's parent. When provided,
 *  `foldOutMessage` still handles forwarded variants locally. A derived
 *  OutMessage replaces the one-to-one lift for the dispatch. When the Step
 *  emits nothing, the lift runs as usual.
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
 *      model => ({ model, commands: [NavigateToRoom({ roomId })] }),
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
const foldChild: (childFold: ChildFoldWithParentOutMessage<ParentModel, ParentMessage, ChildModel, Input, ChildMessage, ChildOutMessage, ParentOutMessage, ChildRequirements, OutMessageStepRequirements, OutMessageStepMessage, DerivedParentOutMessage>) => FoldWithOutMessage<ParentModel, ParentMessage | OutMessageStepMessage, Input, ParentOutMessage | DerivedParentOutMessage, ChildRequirements | OutMessageStepRequirements>
```

### foldChildStep

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L911)

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
 *  `foldOutMessage` takes the same optional second parameter as
 *  foldChild: a FoldContext carrying `liftCommand` and
 *  `liftCommands` bound to this config's `toParentMessage`, for a Command the
 *  Step returns whose result is the child's Message.
 * 
 *  A parent that is itself a Submodel receives a
 *  StepWithOutMessage when `foldOutMessage` emits a derived parent
 *  OutMessage. Add `toParentOutMessage` only when at least one child OutMessage
 *  should continue to the current Submodel's parent. When provided,
 *  `foldOutMessage` still handles forwarded variants locally. A derived
 *  OutMessage replaces the one-to-one lift for the dispatch. When the Step
 *  emits nothing, the lift runs as usual.
 */
const foldChildStep: (childFold: ChildStepFoldWithParentOutMessage<ParentModel, ParentMessage, ChildModel, ChildMessage, ChildOutMessage, ParentOutMessage, ChildRequirements, OutMessageStepRequirements, OutMessageStepMessage, DerivedParentOutMessage>) => StepWithOutMessage<ParentModel, ParentMessage | OutMessageStepMessage, ParentOutMessage | DerivedParentOutMessage, ChildRequirements | OutMessageStepRequirements>
```

### withOutMessage

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/update/update.ts#L81)

```
/**
 * Adds a known or optional OutMessage to a plain update return while
 *  preserving its Model and Commands. Use this helper when attaching to an
 *  existing return or when the value has the type `OutMessage | undefined`.
 *  `undefined` means that the operation emitted no OutMessage, so the returned
 *  record omits the property.
 * 
 *  The input must be a Return, so this helper cannot replace an
 *  OutMessage that an update already emitted.
 * 
 *  ```ts
 *  const editorSave = Update.combine(model, [writeDraft, clearErrors])
 * 
 *  return pipe(editorSave, Update.withOutMessage(outMessage))
 *  ```
 * 
 *  When the OutMessage is already known while constructing a new result,
 *  include it directly: `{ model, commands, outMessage }`. If the OutMessage
 *  may be `undefined`, pass the new result first:
 *  `Update.withOutMessage({ model, commands }, outMessage)`.
 */
const withOutMessage: (outMessage: OutMessage | undefined) => (updateReturn: Return<Model, Message, R>) => ReturnWithOutMessage<Model, Message, OutMessage, R>
```
