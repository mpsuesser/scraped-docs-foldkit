---
url: https://foldkit.dev/api-reference/submodel
title: "Submodel"
description: "API documentation for the Submodel module."
access_date: 2026-08-09T19:15:24.712Z
current_date: 2026-08-09T19:15:24.712Z
---

# Submodel

## Functions

### defineView

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/html/submodel.ts#L88)

```
/**
 * Defines the view function of a Submodel, a child component embedded
 *  via `h.submodel`.
 * 
 *  The runtime supplies the view's `h` builder, typed by the Submodel's
 *  own Message. It is the only builder whose handlers route through this
 *  Submodel's boundary, so a Message from another Message universe is a
 *  type error at the handler call site. A helper this view delegates to
 *  takes `h` as an ordinary parameter; markup owned by an ancestor arrives
 *  pre-built through a `viewInputs` slot callback, which the runtime
 *  executes in the ancestor's boundary.
 * 
 *  Use this ONLY for views that will be embedded via `h.submodel`. Plain
 *  view functions (page-level render functions, helper render functions
 *  that compose Html, etc.) don't need to be defined this way. Write
 *  them as ordinary `(model, h) => Html` functions.
 * 
 *  Explicit type arguments are required because the Model and ViewInputs
 *  parameters cannot drive inference on their own.
 * 
 *  `Message` defaults to `never` so that omitting them fails loudly. Nothing in
 *  the parameter list can infer it, so without the default it would widen to
 *  `unknown` and the builder would accept any Message at all, which is exactly
 *  the confusion this type is meant to prevent.
 */
<Model, Message = never, ViewInputs = void>(fn: [ViewInputs] extends [void]
  ? (model: Model, h: HtmlBuilder<Message>) => VNode | null
  : (model: Model, viewInputs: ViewInputs, h: HtmlBuilder<Message>) => VNode | null): SubmodelView<Model, Message, ViewInputs>
```

## Types

### Config

type

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/html/submodel.ts#L173)

```
/**
 * Configuration for embedding a child Submodel into a parent's view.
 * 
 *  - `slotId`: unique identifier for this Submodel instance under the
 *    current boundary. Name the slot semantically (e.g.
 *    `'sidebar-group'`). For lists, use a stable per-item id (typically
 *    `entry.id`), not the array index. If the same model is rendered in
 *    two DOM positions (desktop + mobile, master + detail), each slot
 *    needs its own id (e.g. `'desktop-sidebar-group'`,
 *    `'mobile-sidebar-group'`). Two `h.submodel` calls inside the same
 *    parent boundary with the same `slotId` throw at view-build time,
 *    including across `createLazy`/`createKeyedLazy` cache hits.
 *  - `view`: the child's `SubmodelView`. Must be branded via
 *    defineView so `h.submodel` can infer the child's Message
 *    type. Unbranded plain functions fail to type-check here.
 *  - `model`: the child's model, inferred from `view`'s first parameter.
 *    Compared by `===` when the boundary is wrapped in a memoizing
 *    helper such as `createKeyedLazy`.
 *  - `viewInputs`: optional second-argument data passed to `view`,
 *    inferred from `view`'s second parameter. Function values AT THE TOP
 *    LEVEL of `viewInputs` (slot callbacks like `toView`) are
 *    auto-wrapped to execute in the parent's boundary so handlers the
 *    consumer builds inside them dispatch through the parent's wrapping
 *    chain. Function values nested below the top level (e.g.
 *    `viewInputs: { config: { onSubmit } }`) throw at view-build time
 *    with a path-based error like `viewInputs.config.onSubmit`. The
 *    check is runtime-only (TypeScript cannot distinguish a
 *    user-declared nested callback from a data value whose prototype
 *    carries methods), so a misuse compiles cleanly and surfaces the
 *    first time the boundary renders. Keep slot callbacks at the top
 *    level of `viewInputs`.
 *  - `toParentMessage`: function that lifts a child message into the
 *    current boundary's Message type. The argument is typed as the
 *    child's Message via the view's brand, and the return is typed as
 *    the embedding builder's Message, so destructuring is correctly
 *    typed without annotation and a lift into the wrong Message union
 *    fails to compile. For per-instance identifiers, capture them in a
 *    closure
 *    (`(message) => GotEntryMessage({ entryId: entry.id, message })`).
 * 
 *  High-level events the parent handles declaratively flow through
 *  each Submodel's `OutMessage`. The parent's `GotChildMessage`
 *  handler unpacks the third tuple element of the child's `update`
 *  return and pattern-matches on `Option<OutMessage>`. See `Menu`,
 *  `Listbox`, etc., for examples.
 */
type Config = Readonly<{
  model: ViewModelOf<View>
  slotId: string
  toParentMessage: (message: ViewMessageOf<View>) => ParentMessage
  view: View
}> & [ViewInputsOf<View>] extends [void]
  ? Readonly<{
    viewInputs: never
  }>
  : Readonly<{
    viewInputs: ViewInputsOf<View>
  }>
```

### Reflect

type

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/submodel/submodel.ts#L21)

```
/**
 * Data-first / data-last signature for a `reflect*` setter built with
 *  `Function.dual`.
 * 
 *  A `reflect*` helper conforms a Submodel to a value that originated
 *  outside it (a URL, a server push, restored storage, a sibling field),
 *  without emitting an OutMessage. It is the inbound complement to
 *  OutMessage's outbound direction: the world is the source of truth, so
 *  the Submodel mirrors it silently and never announces the change back.
 * 
 *  Being dual, it reads two ways. Data-first sets the field and returns the
 *  model; data-last returns `(model) => model`, which slots point-free into
 *  an `evo` callback:
 * 
 *  ```ts
 *  // data-first
 *  const next = Slider.reflectRange(model.priceSlider, rangeFromUrl)
 *  // data-last, point-free in evo
 *  evo(model, { priceSlider: Slider.reflectRange(rangeFromUrl) })
 *  ```
 */
type Reflect = (model: Model, value: Value) => Model
```

### Reflect2

type

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/submodel/submodel.ts#L29)

```
/**
 * Two-argument variant of Reflect, for setters that resolve a
 *  value against a companion argument (for example, a setter that finds a
 *  value's index within a list of options before storing it).
 */
type Reflect2 = (model: Model, a: A, b: B) => Model
```

### View

type

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/html/submodel.ts#L53)

```
/**
 * A view function branded with the Message type it dispatches. Build
 *  one with defineView:
 * 
 *  ```ts
 *  export const view = defineView<Counter.Model, Counter.Message>(
 *    (model, h) => h.button([h.OnClick(Increment())], ['+']),
 *  )
 *  ```
 * 
 *  When `ViewInputs` is provided, the view takes `viewInputs` as its
 *  second argument and the builder moves to third position:
 * 
 *  ```ts
 *  export const view = defineView<
 *    CommandMenu.Model,
 *    CommandMenu.Message,
 *    ViewInputs
 *  >((model, viewInputs, h) =>
 *    viewInputs.toView({
 *      isOpen: model.isOpen,
 *      buttonAttributes: [...],
 *      menuAttributes: [...],
 *      items: menuItemSlots(model, viewInputs.items),
 *    }),
 *  )
 *  ```
 * 
 *  Required at the `h.submodel` call site so unbranded plain functions
 *  fail to type-check there.
 */
type View = [ViewInputs] extends [void]
  ? (model: Model, h: HtmlBuilder<Message>) => VNode | null
  : (model: Model, viewInputs: ViewInputs, h: HtmlBuilder<Message>) => VNode | null & {
  __submodelMessage: Message
}
```
