---
url: https://foldkit.dev/elm/foldkit-vs-elm-side-by-side
title: "Foldkit vs Elm: Side by Side"
description: "A side-by-side comparison of the same pixel art editor built in both Foldkit and Elm. Same architecture, different host: ports vs Commands, decoders vs Schema, and what each side gives up."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Foldkit vs Elm: Side by Side

## Overview

This comparison uses the same [pixel art editor](https://foldkit.dev/example-apps/pixel-art) in Foldkit and Elm. Both versions include grid drawing, undo and redo, brush, fill, and eraser tools, mirror modes, localStorage persistence, PNG export, keyboard shortcuts, and an application history panel.

Foldkit applies the Elm Architecture in TypeScript on top of Effect. The familiar pieces remain: a Model, Messages, a pure update function, a view, and side effects returned for the Runtime to execute. The differences come from the host languages, their interop models, and the tools each framework builds around the architecture.

Elm is the source of this architecture, and its language provides guarantees TypeScript cannot reproduce. Foldkit trades some of those guarantees for direct access to the TypeScript, Effect, browser, and npm ecosystems.

Read them both

The Foldkit version is in the [examples gallery](https://foldkit.dev/example-apps/pixel-art). The [Elm version source](https://github.com/foldkit/foldkit/tree/main/comparisons/pixel-art-elm) is an Elm 0.19 application with no npm dependencies.

## The Architecture You Already Know

Most concepts translate directly:

Elm

Foldkit

State

`Model`

Model declared with Schema

Events

`Msg`

custom type

Message Schema union

Transitions

`update : Msg -> Model -> ( Model, Cmd Msg )`

`update(model, message): [Model, Command[]]`

Side effects

`Cmd Msg`

Command for Message-driven work, plus other lifecycle primitives

Event streams

`Sub Msg`

Subscription backed by an Effect Stream

Boot data

Flags decoded or accepted by

`init`

[Flags](https://foldkit.dev/core/init-and-flags)

Schema supplied at boot

JS interop

Flags, ports, and custom elements

Direct JavaScript APIs,

[Mount](https://foldkit.dev/core/mount)

, and CustomElement

Nested state

Nested Elm Architecture composition

[Submodel](https://foldkit.dev/core/submodel)

helpers for the same pattern

### Elm Msg

The Elm application has 21 `Msg` variants:

```
type Msg
    = PressedCell Int Int
    | EnteredCell Int Int
    | LeftCanvas
    | ReleasedMouse
    | SelectedColor Int
    | SelectedTool Tool
    | SelectedGridSize Int
    | ToggledMirrorHorizontal
    | ToggledMirrorVertical
    | ClickedUndo
    | ClickedRedo
    | ClickedHistoryStep Int
    | ClickedRedoStep Int
    | ClickedClear
    | ClickedExport
    | FailedExportPng String
    | DismissedErrorDialog
    | ConfirmedGridSizeChange
    | DismissedGridSizeDialog
    | SelectedPaletteTheme Int
    | ToggledThemePicker
```

### Foldkit Message union

The current Foldkit application has 25 parent Messages:

```
const PressedCell = m('PressedCell', { x: S.Number, y: S.Number })
const EnteredCell = m('EnteredCell', { x: S.Number, y: S.Number })
const LeftCanvas = m('LeftCanvas')
const ReleasedMouse = m('ReleasedMouse')
const SelectedColor = m('SelectedColor', { colorIndex: PaletteIndex })
const SelectedTool = m('SelectedTool', { tool: Tool })
const SelectedGridSize = m('SelectedGridSize', { size: S.Number })
const ToggledMirrorHorizontal = m('ToggledMirrorHorizontal')
const ToggledMirrorVertical = m('ToggledMirrorVertical')
const ClickedUndo = m('ClickedUndo')
const ClickedRedo = m('ClickedRedo')
const ClickedHistoryStep = m('ClickedHistoryStep', { stepIndex: S.Number })
const ClickedRedoStep = m('ClickedRedoStep', { stepIndex: S.Number })
const ClickedClear = m('ClickedClear')
const ClickedExport = m('ClickedExport')
const SucceededExportPng = m('SucceededExportPng')
const FailedExportPng = m('FailedExportPng', { error: S.String })
const GotErrorDialogMessage = m('GotErrorDialogMessage', {
  message: Dialog.Message,
})
const ConfirmedGridSizeChange = m('ConfirmedGridSizeChange')
const GotGridSizeConfirmDialogMessage = m('GotGridSizeConfirmDialogMessage', {
  message: Dialog.Message,
})
const GotThemeListboxMessage = m('GotThemeListboxMessage', {
  message: Listbox.Message,
})
const GotToolRadioGroupMessage = m('GotToolRadioGroupMessage', {
  message: RadioGroup.Message,
})
const GotGridSizeRadioGroupMessage = m('GotGridSizeRadioGroupMessage', {
  message: RadioGroup.Message,
})
const GotPaletteRadioGroupMessage = m('GotPaletteRadioGroupMessage', {
  message: RadioGroup.Message,
})
const CompletedSaveCanvas = m('CompletedSaveCanvas')

const Message = S.Union([
  PressedCell,
  EnteredCell,
  LeftCanvas,
  ReleasedMouse,
  SelectedColor,
  SelectedTool,
  SelectedGridSize,
  ToggledMirrorHorizontal,
  ToggledMirrorVertical,
  ClickedUndo,
  ClickedRedo,
  ClickedHistoryStep,
  ClickedRedoStep,
  ClickedClear,
  ClickedExport,
  SucceededExportPng,
  FailedExportPng,
  GotErrorDialogMessage,
  GotThemeListboxMessage,
  GotToolRadioGroupMessage,
  GotGridSizeRadioGroupMessage,
  GotPaletteRadioGroupMessage,
  ConfirmedGridSizeChange,
  GotGridSizeConfirmDialogMessage,
  CompletedSaveCanvas,
])
type Message = typeof Message.Type
```

The count differs because the component and effect boundaries differ. Foldkit has `SucceededExportPng` and `CompletedSaveCanvas` for Command completion, plus six `Got*Message` wrappers for two Dialogs, one Listbox, and three RadioGroups. The Elm version hand-rolls those controls and represents their application-facing events with four direct Msgs: `ToggledThemePicker`, `SelectedPaletteTheme`, `DismissedErrorDialog`, and `DismissedGridSizeDialog`.

Both unions are the total input domain of their application update functions. Foldkit’s child wrapper Messages lead to another Message union and update function inside each Submodel.

Elm custom-type values and Foldkit Schema values both have runtime tags. Schema also supplies runtime decoding and encoding when a Message is deliberately used at an external boundary. That does not make an application Message union a wire protocol automatically. A network boundary still needs an explicit Schema and compatibility policy.

## The Update Function

The update functions have the same shape.

### Elm update

```
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        PressedCell x y ->
            case model.tool of
                Brush ->
                    ( { model
                        | grid = applyBrush x y model
                        , undoStack = Grid.pushHistory model.grid model.undoStack
                        , redoStack = []
                        , isDrawing = True
                      }
                    , Cmd.none
                    )

                Fill ->
                    withSave
                        { model
                            | grid = Grid.floodFill x y model.selectedColorIndex model.grid
                            , undoStack = Grid.pushHistory model.grid model.undoStack
                            , redoStack = []
                        }

                Eraser ->
                    -- ...
        ClickedUndo ->
            case model.undoStack of
                [] ->
                    ( model, Cmd.none )

                previousGrid :: olderGrids ->
                    withSave
                        { model
                            | grid = previousGrid
                            , undoStack = olderGrids
                            , redoStack = model.grid :: model.redoStack
                        }

        -- ... 19 more branches

withSave : Model -> ( Model, Cmd Msg )
withSave model =
    ( model, saveCanvas (encodeSavedCanvas model) )
```

### Foldkit update

```
export const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    withUpdateReturn,
    M.tagsExhaustive({
      PressedCell: ({ x, y }) =>
        M.value(model.tool).pipe(
          withUpdateReturn,
          M.when('Brush', () => [
            evo(model, {
              grid: () => applyBrush(model, x, y),
              undoStack: () => pushHistory(model.undoStack, model.grid),
              redoStack: () => [],
              isDrawing: () => true,
            }),
            [],
          ]),
          M.when('Fill', () => {
            const nextModel = evo(model, {
              grid: () => applyFill(model, x, y),
              undoStack: () => pushHistory(model.undoStack, model.grid),
              redoStack: () => [],
            })
            return [nextModel, [saveCanvas(nextModel)]]
          }),
          // ...
        ),
      ClickedUndo: () =>
        Array.match(model.undoStack, {
          onEmpty: () => [model, []],
          onNonEmpty: nonEmptyUndoStack => {
            const nextModel = evo(model, {
              grid: () => Array.lastNonEmpty(nonEmptyUndoStack),
              undoStack: () => Array.initNonEmpty(nonEmptyUndoStack),
              redoStack: () => [...model.redoStack, model.grid],
            })
            return [nextModel, [saveCanvas(nextModel)]]
          },
        }),
      // ... 23 more handlers
    }),
  )
```

`case msg of` becomes `M.tagsExhaustive`. Elm record updates become `evo` transformations. `( model, Cmd.none )` becomes `[model, []]`.

Elm enforces exhaustive pattern matching as part of the language. Foldkit obtains the same compile-time failure at a match written with `M.tagsExhaustive`. That is the required Foldkit update style, but TypeScript itself does not prevent someone from writing a non-exhaustive alternative.

Elm record updates and `evo` both preserve references to unchanged nested values. The rendering section shows how each application uses that reference stability.

## The Model: Custom Types vs Schema

### Elm Model (type alias and custom types)

The Elm Model uses custom types and `Maybe`:

```
type Tool
    = Brush
    | Fill
    | Eraser

type MirrorMode
    = MirrorNone
    | MirrorHorizontal
    | MirrorVertical
    | MirrorBoth

type alias Model =
    { grid : Grid
    , undoStack : List Grid
    , redoStack : List Grid
    , selectedColorIndex : Int
    , gridSize : Int
    , tool : Tool
    , mirrorMode : MirrorMode
    , isDrawing : Bool
    , hoveredCell : Maybe Position
    , exportError : Maybe String
    , paletteThemeIndex : Int
    , pendingGridSize : Maybe Int
    , isThemePickerOpen : Bool
    }
```

This hand-rolled UI stores Dialog visibility through the presence of `exportError` and `pendingGridSize`. The theme picker uses a separate `isThemePickerOpen` field.

### Foldkit Model (Schema struct)

The Foldkit Model uses Effect Schema, `Option`, and child Models for its stateful Foldkit UI controls:

```
import { Schema as S } from 'effect'

import { Dialog, Listbox, RadioGroup } from '@foldkit/ui'

export const Model = S.Struct({
  grid: Grid,
  undoStack: S.Array(Grid),
  redoStack: S.Array(Grid),
  selectedColorIndex: PaletteIndex,
  gridSize: S.Number,
  tool: Tool,
  mirrorMode: MirrorMode,
  isDrawing: S.Boolean,
  maybeHoveredCell: S.Option(Position),
  errorDialog: Dialog.Model,
  maybeExportError: S.Option(S.String),
  paletteThemeIndex: S.Number,
  gridSizeConfirmDialog: Dialog.Model,
  maybePendingGridSize: S.Option(S.Number),
  themeListbox: Listbox.Model,
  toolRadioGroup: RadioGroup.Model,
  gridSizeRadioGroup: RadioGroup.Model,
  paletteRadioGroup: RadioGroup.Model,
})
```

A Schema exists at runtime as well as in TypeScript. Foldkit can use it to validate flags and persisted values, encode selected data, and describe Models to framework tooling. The child component Models expose interaction state that the Elm application implements directly in its parent Model and views.

Elm’s type system is sound and its custom types are compact. TypeScript is intentionally less strict, while Schema adds runtime boundary tools that a plain TypeScript type does not have.

## Ports vs Commands

The pixel editor saves to localStorage and exports a PNG. The Elm implementation crosses into JavaScript for both operations. The Foldkit implementation performs them in Commands.

### Elm ports (the effect lives in JavaScript)

The Elm side declares outgoing and incoming ports:

```
port module Main exposing (Msg(..), defaultModel, main, update)

-- The Elm side: ports declare that JavaScript exists, nothing more.

port saveCanvas : Encode.Value -> Cmd msg

port requestExportPng : Encode.Value -> Cmd msg

port exportPngFailed : (String -> msg) -> Sub msg

-- In update: send a request out, receive the failure (if any) back
-- as a Msg through the subscription.

        ClickedExport ->
            ( model, requestExportPng (encodeExportRequest model) )

        FailedExportPng error ->
            ( { model | exportError = Just error }, Cmd.none )
```

The JavaScript side subscribes to them in `index.html`:

```
// The JavaScript side, in index.html. This code is invisible to the
// Elm compiler. If it throws, drifts out of sync with the encoder, or
// forgets to call send(), Elm cannot know.

app.ports.saveCanvas.subscribe(function (data) {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data))
  } catch (error) {
    // Silently fail on storage errors
  }
})

app.ports.requestExportPng.subscribe(function (request) {
  try {
    var canvas = document.createElement('canvas')
    var context = canvas.getContext('2d')
    if (context === null) {
      throw new Error('Canvas 2D context not available')
    }
    // ... paint request.pixels onto the canvas, then download ...
    link.click()
  } catch (error) {
    app.ports.exportPngFailed.send(
      error instanceof Error ? error.message : 'Failed to export image',
    )
  }
})
```

The port declaration gives Elm a typed interface. The JavaScript subscriber remains outside the Elm compiler, so a renamed payload field or a missing `send` call is not checked against the Elm source. A JavaScript exception can still affect the host page; the boundary protects Elm code from directly calling arbitrary JavaScript, not the entire page from JavaScript failures.

The export failure needs an incoming port so JavaScript can send `FailedExportPng` back to Elm. Saving is fire-and-forget in this application.

### Foldkit Commands (the effect lives with the app)

Foldkit runs in the JavaScript ecosystem, so its Commands can use browser APIs and JavaScript libraries directly:

```
const SaveCanvas = Command.define('SaveCanvas', {
  args: {
    grid: Grid,
    gridSize: S.Number,
    paletteThemeIndex: S.Number,
    selectedColorIndex: PaletteIndex,
  },
  messages: [CompletedSaveCanvas],
  execute: ({ grid, gridSize, paletteThemeIndex, selectedColorIndex }) =>
    Effect.gen(function* () {
      const store = yield* KeyValueStore.KeyValueStore
      const data: SavedCanvas = {
        grid,
        gridSize,
        paletteThemeIndex,
        selectedColorIndex,
      }
      yield* store.set(STORAGE_KEY, S.encodeSync(SavedCanvasJsonString)(data))
      return CompletedSaveCanvas()
    }).pipe(
      Effect.catch(() => Effect.succeed(CompletedSaveCanvas())),
      Effect.provide(BrowserKeyValueStore.layerLocalStorage),
    ),
})

const ExportPng = Command.define('ExportPng', {
  args: { grid: Grid, gridSize: S.Number, paletteThemeIndex: S.Number },
  messages: [SucceededExportPng, FailedExportPng],
  execute: ({ grid, gridSize, paletteThemeIndex }) =>
    Effect.gen(function* () {
      const theme = PALETTE_THEMES[paletteThemeIndex] ?? PALETTE_THEMES[0]
      const canvas = document.createElement('canvas')
      const context = canvas.getContext('2d')

      if (Predicate.isNull(context)) {
        return yield* Effect.fail(
          FailedExportPng({ error: 'Canvas 2D context not available' }),
        )
      }

      // ... paint each cell, then click a generated download link

      return SucceededExportPng()
    }).pipe(
      Effect.catchTag('FailedExportPng', error => Effect.succeed(error)),
      Effect.catch(() =>
        Effect.succeed(FailedExportPng({ error: 'Failed to export image' })),
      ),
    ),
})
```

Each Command declares its arguments and result Messages. Its Effect can use typed failures and recovery operators before returning a Message to update. The application still needs to choose meaningful error behavior. Here `ExportPng` reports failure, while `SaveCanvas` intentionally converts storage failure into the same completion Message as success.

The boundary trade-off

Elm prevents application code from calling arbitrary JavaScript and makes interop explicit through ports or custom elements. Foldkit keeps update pure by convention and framework design, while Command bodies can call the host ecosystem directly. TypeScript cannot enforce Elm’s purity boundary.

## JSON: Decoders vs Schema

Both applications restore a saved canvas from boot flags and persist it as JSON.

### Elm decoders and encoders

Elm defines the type, decoder, and encoder separately:

```
init : Decode.Value -> ( Model, Cmd Msg )
init flags =
    case Decode.decodeValue savedCanvasDecoder flags of
        Ok saved ->
            ( { defaultModel
                | grid = saved.grid
                , gridSize = saved.gridSize
                , paletteThemeIndex = saved.paletteThemeIndex
                , selectedColorIndex = saved.selectedColorIndex
              }
            , Cmd.none
            )

        Err _ ->
            ( defaultModel, Cmd.none )

savedCanvasDecoder : Decode.Decoder SavedCanvas
savedCanvasDecoder =
    Decode.map4 SavedCanvas
        (Decode.field "grid" gridDecoder)
        (Decode.field "gridSize" Decode.int)
        (Decode.field "paletteThemeIndex" Decode.int)
        (Decode.field "selectedColorIndex" Decode.int)

gridDecoder : Decode.Decoder Grid
gridDecoder =
    Decode.array (Decode.array (Decode.nullable Decode.int))

-- And the encoder, written by hand in the other direction:

encodeSavedCanvas : Model -> Encode.Value
encodeSavedCanvas model =
    Encode.object
        [ ( "grid", encodeGrid model.grid )
        , ( "gridSize", Encode.int model.gridSize )
        , ( "paletteThemeIndex", Encode.int model.paletteThemeIndex )
        , ( "selectedColorIndex", Encode.int model.selectedColorIndex )
        ]
```

The compiler checks the values each function produces, but the decoder and encoder use independent string field names. A mismatch between `"gridSize"` and `"gridsize"` can compile.

### Foldkit Schema (one definition, both directions)

Foldkit derives both directions from one `SavedCanvas` Schema:

```
// The Schema is the single source of truth. The decoder and the
// encoder both fall out of it. They cannot drift apart.

export const SavedCanvas = S.Struct({
  grid: SavedGrid,
  gridSize: S.Number,
  paletteThemeIndex: S.Number,
  selectedColorIndex: PaletteIndex,
})

export const SavedCanvasJsonString = S.fromJsonString(
  S.toCodecJson(SavedCanvas),
)

export const flags: Effect.Effect<Flags> = Effect.gen(function* () {
  const store = yield* KeyValueStore.KeyValueStore
  const json = yield* Effect.fromOption(
    Option.fromNullishOr(yield* store.get(STORAGE_KEY)),
  )
  const decoded = yield* S.decodeEffect(SavedCanvasJsonString)(json)
  return Flags.make({ maybeSavedCanvas: Option.some(decoded) })
}).pipe(
  Effect.catch(() =>
    Effect.succeed(Flags.make({ maybeSavedCanvas: Option.none() })),
  ),
  Effect.provide(BrowserKeyValueStore.layerLocalStorage),
)

// Saving goes through the same Schema:
// S.encodeSync(SavedCanvasJsonString)(data)
```

The Schema centralizes field names and value constraints. Encoding and decoding therefore evolve from the same definition. Version migrations and fallback behavior still belong to the application.

## Subscriptions

Both frameworks derive external event streams from Model state. The mouse-release listener exists only while the user is drawing.

### Elm subscriptions

```
subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.batch
        [ Browser.Events.onKeyDown (keyboardDecoder model)
        , if model.isDrawing then
            Browser.Events.onMouseUp (Decode.succeed ReleasedMouse)

          else
            Sub.none
        , exportPngFailed FailedExportPng
        ]

keyboardDecoder : Model -> Decode.Decoder Msg
keyboardDecoder model =
    Decode.map5 KeyEvent
        (Decode.field "key" Decode.string)
        (Decode.field "ctrlKey" Decode.bool)
        (Decode.field "metaKey" Decode.bool)
        (Decode.field "shiftKey" Decode.bool)
        (Decode.field "altKey" Decode.bool)
        |> Decode.andThen (shortcutFor model)

-- shortcutFor maps the decoded event to a Msg, or fails the
-- decoder for keys the app does not care about.
```

### Foldkit Subscriptions

```
export const subscriptions = Subscription.make<Model, Message>()(entry => ({
  keyboard: Subscription.persistent(
    Stream.fromEventListener<KeyboardEvent>(document, 'keydown').pipe(
      Stream.mapEffect(handleKeyboardEvent),
      Stream.filter(Option.isSome),
      Stream.map(option => option.value),
    ),
  ),

  mouseRelease: entry(
    { isDrawing: S.Boolean },
    {
      modelToDependencies: model => ({ isDrawing: model.isDrawing }),
      dependenciesToStream: ({ isDrawing }) =>
        Stream.when(
          Stream.fromEventListener(document, 'mouseup').pipe(
            Stream.map(() => ReleasedMouse()),
          ),
          Effect.sync(() => isDrawing),
        ),
    },
  ),
}))
```

Elm’s `Sub.batch` and Foldkit’s Subscription registry both describe the active set after each state transition. The runtime handles setup and teardown.

Elm uses `Browser.Events` for keyboard and mouse input and an incoming port for export failure. Foldkit Subscriptions use Effect Streams, so the application can construct a Stream from browser APIs or a JavaScript client directly. Export failure does not need a Subscription because it is already a declared Command result.

## Rendering Performance

Both implementations use reference-based memoization around the grid. Actual frame time depends on the production build, browser, and device, so this section compares the mechanisms rather than claiming a universal winner.

### Elm Html.Lazy and Html.Keyed

```
toolbarView : Model -> PaletteTheme -> Html Msg
toolbarView model theme =
    div [ class "w-full md:w-44 flex flex-col gap-5 flex-shrink-0" ]
        [ lazy toolSection model.tool
        , lazy mirrorSection model.mirrorMode
        , lazy sizeSection model.gridSize
        , paletteSection model theme
        , lazy clearCanvasSection model.grid
        ]

-- The canvas keys each row and wraps it in lazy5. A row only
-- re-renders when one of its five arguments changes by reference.

            Html.Keyed.node "div"
                [ class "cursor-crosshair select-none w-full aspect-square flex flex-col bg-white" ]
                (Grid.toRows model.grid
                    |> List.map
                        (\( y, row ) ->
                            ( String.fromInt y
                            , lazy5 rowView
                                y
                                row
                                previewColor
                                (rowPreviewPositions y previewPositions)
                                theme.colors
                            )
                        )
                )
```

### Foldkit createLazy and keyed

```
const lazyHeader = createLazy()
const lazyToolPanel = createLazy()
const lazyHistoryPanel = createLazy()
const lazyRow = createKeyedLazy()

// Each args array is compared element-by-element against the previous render.
// If every arg is reference-equal, the view function isn't called at all.
// evo() preserves references for unchanged Model fields, so the check just
// works, and the builder is the same object every render, so passing it
// through the args never invalidates the cache.
export const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: 'Pixel Art',
  body: h.div(
    [],
    [
      lazyHeader(headerView, [h]),
      lazyToolPanel(toolPanelView, [
        model.mirrorMode,
        model.tool,
        model.gridSize,
        model.selectedColorIndex,
        isGridEmpty(model.grid),
        theme,
        model.themeListbox,
        h,
      ]),
      canvasView(model, theme, h),
      lazyHistoryPanel(historyPanelView, [
        model.undoStack,
        model.redoStack,
        currentGrid,
        model.gridSize,
        theme,
        h,
      ]),
    ],
  ),
})
```

`Html.Lazy.lazy` and `createLazy` reuse a previous rendered value when their function inputs remain referentially equal. Elm provides arity-specific helpers such as `lazy` and `lazy5`. Foldkit creates a lazy wrapper at module scope and passes an argument array. `createKeyedLazy` retains a separate cache for each stable key.

### The cell view, twice

Both cell views attach Message values to event attributes:

```
rowView : Int -> Array Cell -> String -> List Int -> List String -> Html Msg
rowView y row previewColor previewColumns paletteColors =
    div [ class "flex flex-1" ]
        (Array.toIndexedList row
            |> List.map
                (\( x, cell ) ->
                    let
                        displayColor =
                            if List.member x previewColumns then
                                previewColor

                            else
                                cellColor paletteColors cell
                    in
                    cellView x y displayColor
                )
        )

cellView : Int -> Int -> String -> Html Msg
cellView x y backgroundColor =
    div
        [ onMouseDown (PressedCell x y)
        , onMouseEnter (EnteredCell x y)
        , style "flex" "1"
        , style "background-color" backgroundColor
        ]
        []
```

```
const rowView = (
  row: ReadonlyArray<Cell>,
  y: number,
  previewColor: HexColor,
  previewPositions: ReadonlyArray<readonly [number, number]>,
  theme: PaletteTheme,
  h: HtmlBuilder<Message>,
): Html =>
  h.div(
    [h.Style({ display: 'flex', flex: '1' })],
    Array.map(row, (cell, x) => {
      const isPreview = previewPositions.some(
        ([previewX, previewY]) => previewX === x && previewY === y,
      )
      const displayColor = isPreview ? previewColor : resolveColor(cell, theme)

      return h.div([
        h.OnMouseDown(PressedCell({ x, y })),
        h.OnMouseEnter(EnteredCell({ x, y })),
        h.Style({ flex: '1', backgroundColor: displayColor }),
      ])
    }),
  )
```

Neither view needs a component instance or a memoized event-handler closure for each cell. The coordinates are stored in the `Msg` or Message value dispatched by the event.

## UI Components

The Elm version implements its Dialogs, RadioGroups, switches, and theme picker in the application. That keeps their state and events visible, but the application also owns their ARIA attributes, keyboard behavior, focus behavior, and transitions.

The Foldkit version uses [Foldkit UI](https://foldkit.dev/ui/overview). Its Dialogs, RadioGroups, and Listbox are Submodels, while Switch is a controlled render helper. Selected values remain in the parent Model. Each stateful component reports changes through OutMessages that the parent folds into its own update.

Elm application

Foldkit application

Dialog, RadioGroup, Switch, Listbox

Implemented in the application

Dialog, Listbox, and RadioGroup Submodels; controlled Switch helper

Accessibility behavior

Implemented and tested by the application

Implemented and tested by Foldkit UI

Selected values

Parent Model

Parent Model

Transient interaction state

Parent Model and view logic

Child Models in the application Model

Composition

Nested architecture written by the app

[Submodel](https://foldkit.dev/core/submodel)

helpers standardize parent-child delegation

The comparison is between the two checked-in applications, not the entire Elm package ecosystem. An Elm application can use community UI packages or organize nested state differently.

## Testing

Both update functions are pure and easy to call directly. Their effect values differ.

### Elm update test (pure, but the Cmd is opaque)

```
suite : Test
suite =
    test "undo restores the previous grid state" <|
        \() ->
            let
                -- The Cmd in each returned tuple is discarded with `_`.
                -- A Cmd is opaque: there is no way to look inside one,
                -- so there is no way to assert that ReleasedMouse
                -- actually triggered a save.
                ( afterPress, _ ) =
                    update (PressedCell 0 0) defaultModel

                ( afterRelease, _ ) =
                    update ReleasedMouse afterPress

                ( afterUndo, _ ) =
                    update ClickedUndo afterRelease
            in
            Expect.all
                [ \model -> Expect.equal (Grid.cellAt 0 0 model.grid) Nothing
                , \model -> Expect.equal model.undoStack []
                , \model -> Expect.equal (List.length model.redoStack) 1
                ]
                afterUndo
```

`Cmd Msg` is opaque, so a direct `elm-test` unit test cannot compare or pattern-match the Command returned by update. The underscores discard it. Removing the save Command from `ReleasedMouse` would not fail this particular unit test.

Program-level tools such as [elm-program-test](https://package.elm-lang.org/packages/avh4/elm-program-test/latest/) provide a higher-level way to simulate supported effects and interactions. That is a different test boundary from inspecting a `Cmd` value directly.

### Foldkit Story test (Commands are assertable values)

```
test('undo restores the previous grid state', () => {
  story(
    update,
    given(emptyModel),
    message(PressedCell({ x: 0, y: 0 })),
    message(ReleasedMouse()),
    Command.resolve(SaveCanvas, CompletedSaveCanvas()),
    model(model => {
      expect(model.grid[0]?.[0]).toEqual(Option.some(0))
      expect(model.undoStack).toHaveLength(1)
    }),
    message(ClickedUndo()),
    Command.resolve(SaveCanvas, CompletedSaveCanvas()),
    model(model => {
      expect(model.grid[0]?.[0]).toEqual(Option.none())
      expect(model.undoStack).toHaveLength(0)
      expect(model.redoStack).toHaveLength(1)
    }),
  )
})
```

A [Story](https://foldkit.dev/testing/story) receives the named Commands returned by update. `Command.resolve` verifies that `SaveCanvas` is pending, supplies `CompletedSaveCanvas`, and dispatches that result Message. Removing the Command makes this Story fail at the resolution step.

[Scene](https://foldkit.dev/testing/scene) adds interaction through Foldkit virtual DOM. It can query by accessible role, label, or text without starting jsdom.

## What You Give Up

Moving from Elm to Foldkit gives up language-level constraints.

**Enforced purity.** Elm code cannot call `Date.now()`, mutate an object, or perform I/O from update. TypeScript can. Foldkit’s architecture, conventions, and tests make the intended boundary visible, but they do not make an impure update impossible to write.

**Elm’s runtime guarantees.** Elm models failure as data and prevents the ordinary null, undefined, and non-exhaustive failures common in JavaScript. Foldkit uses Schema, Effect, and explicit failure Messages, but TypeScript and npm dependencies can still throw or produce invalid values.

**A smaller language and package surface.** Elm has one language, formatter, package manager, and constrained package API. TypeScript plus Effect and browser libraries has a larger set of concepts and more choices.

**A smaller default runtime footprint.** Optimized Elm output is often compact. A Foldkit application includes Foldkit and Effect. The actual production size depends on the application and should be measured from the two builds being considered.

## What You Gain

Foldkit gains direct access to the host ecosystem and additional framework tools.

**JavaScript and npm access.** Browser APIs and compatible JavaScript packages can be imported into a Command, Mount, Subscription, ManagedResource, or CustomElement without a port layer.

**TypeScript integration.** An embedded Foldkit program can share modules and types with its TypeScript host. Elm also embeds cleanly, but host communication crosses flags, ports, or custom elements.

**Schema codecs.** One definition can provide the TypeScript type, runtime validation, and encoding and decoding for an external boundary.

**Inspectable Commands.** Foldkit DevTools records named Commands beside the Messages that produced them, and Story and Scene tests can assert on the same values.

**Effect services and control flow.** Commands can compose retries, timeouts, concurrency, resources, Layers, and typed failures from Effect.

**First-party UI Submodels.** Foldkit UI supplies accessible components built with the same Model, Message, and update architecture as the application.

## Conclusion

Elm and Foldkit share the application model, so the choice turns on the host environment and the guarantees you need.

Choose Elm when its language, compiler, package constraints, and interop model fit the application. Those constraints provide purity and refactoring guarantees that a TypeScript framework cannot reproduce.

Choose Foldkit when the application needs to remain in TypeScript, integrate directly with JavaScript libraries, or use Effect services while retaining the Elm Architecture. Foldkit standardizes that architecture and adds Schema, inspectable Commands, Submodels, DevTools, and testing tools around it.

The [Elm source](https://github.com/foldkit/foldkit/tree/main/comparisons/pixel-art-elm) and [Foldkit source](https://github.com/foldkit/foldkit/tree/main/examples/pixel-art) remain recognizably the same kind of program. Their differences show which guarantees come from Elm the language and which structures Foldkit recreates in TypeScript.
