---
url: https://foldkit.dev/api-reference/html
title: "Html"
description: "API documentation for the Html module."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

# Html

## Functions

### childAttributes

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/childAttribute.ts#L61)

```
/**
 * Captures the current boundary's dispatcher and wraps each attribute
 *  so handlers inside it route through that boundary's wrapping chain at
 *  event-fire time, even when the attribute is later spread into a
 *  parent's element in a different boundary.
 * 
 *  Submodels call this when publishing attribute groups to a consumer's
 *  `toView` slot callback:
 * 
 *  ```ts
 *  // Inside a SubmodelView running in the child's boundary:
 *  return viewInputs.toView({
 *    checkbox: childAttributes([
 *      h.OnClick(Toggled()),
 *      h.Role('checkbox'),
 *    ]),
 *    ...
 *  })
 *  ```
 * 
 *  Without this binding step the consumer's element constructor would
 *  process `h.OnClick(Toggled())` using the parent's dispatcher (because
 *  the consumer's `toView` runs in the parent's boundary), bypassing the
 *  Submodel's `toParentMessage`.
 */
<Attribute>(attributes: readonly Array<Attribute>): readonly Array<Readonly<{
  __childAttribute: true
  attribute: unknown
  boundaryMappers: readonly Array<(message: unknown) => unknown>
  dispatch: DispatchSync
  resolveUnmount: (message: unknown) => () => void
}>>
```

### createKeyedLazy

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/lazy.ts#L127)

```
/**
 * Creates a keyed memoization map for view functions rendered in a loop. Each
 *  key gets its own independent cache slot. On each render, only entries whose
 *  function reference, dispatch, or arguments have changed by reference are
 *  recomputed.
 * 
 *  Like `createLazy`, each key's cached VNode must be rendered at a single
 *  position in the tree. If the same item needs to appear in multiple
 *  positions, create one keyed lazy per position.
 */
(): (key: PropertyKey, fn: (args: Args) => VNode | null, args: Args) => VNode | null
```

### createLazy

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/lazy.ts#L104)

```
/**
 * Creates a memoization slot for a view function. On each render, if the
 *  function reference, dispatch, and all arguments are referentially equal
 *  (`===`) to the previous call, the cached VNode is returned without
 *  re-running the view function. Snabbdom's `patchVnode` short-circuits when
 *  it sees the same VNode reference, so both VNode construction and subtree
 *  diffing are skipped.
 * 
 *  Dispatch is part of the cache key because event handlers in the cached
 *  VNode close over the dispatch active when the VNode was built. Returning
 *  a VNode built under a different dispatch would silently misroute every
 *  event from that subtree.
 * 
 *  The cached VNode must be rendered at a single position in the tree.
 *  Snabbdom tracks the real DOM through each VNode's mutable `.elm` field
 *  and assumes one VNode per position. Rendering the same cached VNode at
 *  two positions causes patches to collide and can duplicate or misplace
 *  DOM nodes. If the same content needs to appear in multiple positions,
 *  create one slot per position.
 */
(): (fn: (args: Args) => VNode | null, args: Args) => VNode | null
```

## Types

### Attribute

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/index.ts#L446)

```
/**
 * Union of all HTML, SVG, and MathML attributes a virtual DOM element can carry.
 * 
 *  When a Submodel publishes attribute groups to a consumer's `toView`
 *  slot, those attributes are wrapped via childAttributes into
 *  ChildAttribute, a distinct type that carries the Submodel's
 *  own dispatcher. Element constructors accept the union
 *  `ReadonlyArray<Attribute<Message> | ChildAttribute>`, so consumers
 *  can spread published bundles directly into their own attribute
 *  arrays.
 */
type Attribute = Data.TaggedEnum<{
  Accept: {
    value: string
  }
  Accesskey: {
    value: string
  }
  Action: {
    value: string
  }
  AlignmentBaseline: {
    value: string
  }
  Allow: {
    value: string
  }
  AllowDrop: {}
  Alt: {
    value: string
  }
  AriaActiveDescendant: {
    value: string
  }
  AriaAtomic: {
    value: boolean
  }
  AriaAutocomplete: {
    value: string
  }
  AriaBusy: {
    value: boolean
  }
  AriaChecked: {
    value: boolean | "mixed"
  }
  AriaColcount: {
    value: number
  }
  AriaColindex: {
    value: number
  }
  AriaColspan: {
    value: number
  }
  AriaControls: {
    value: string
  }
  AriaCurrent: {
    value: string
  }
  AriaDescribedBy: {
    value: string
  }
  AriaDescription: {
    value: string
  }
  AriaDetails: {
    value: string
  }
  AriaDisabled: {
    value: boolean
  }
  AriaErrorMessage: {
    value: string
  }
  AriaExpanded: {
    value: boolean
  }
  AriaFlowto: {
    value: string
  }
  AriaHasPopup: {
    value: string
  }
  AriaHidden: {
    value: boolean
  }
  AriaInvalid: {
    value: boolean
  }
  AriaKeyshortcuts: {
    value: string
  }
  AriaLabel: {
    value: string
  }
  AriaLabelledBy: {
    value: string
  }
  AriaLevel: {
    value: number
  }
  AriaLive: {
    value: string
  }
  AriaModal: {
    value: boolean
  }
  AriaMultiSelectable: {
    value: boolean
  }
  AriaOrientation: {
    value: string
  }
  AriaOwns: {
    value: string
  }
  AriaPlaceholder: {
    value: string
  }
  AriaPosinset: {
    value: number
  }
  AriaPressed: {
    value: string
  }
  AriaReadonly: {
    value: boolean
  }
  AriaRelevant: {
    value: string
  }
  AriaRequired: {
    value: boolean
  }
  AriaRoleDescription: {
    value: string
  }
  AriaRowcount: {
    value: number
  }
  AriaRowindex: {
    value: number
  }
  AriaRowspan: {
    value: number
  }
  AriaSelected: {
    value: boolean
  }
  AriaSetsize: {
    value: number
  }
  AriaSort: {
    value: string
  }
  AriaValuemax: {
    value: number
  }
  AriaValuemin: {
    value: number
  }
  AriaValuenow: {
    value: number
  }
  AriaValuetext: {
    value: string
  }
  Attribute: {
    key: string
    value: string
  }
  Autocapitalize: {
    value: string
  }
  Autocomplete: {
    value: string
  }
  Autocorrect: {
    value: string
  }
  Autofocus: {
    value: boolean
  }
  Autoplay: {
    value: boolean
  }
  BaselineShift: {
    value: string
  }
  Charset: {
    value: string
  }
  Checked: {
    value: boolean
  }
  CiteAttr: {
    value: string
  }
  Class: {
    value: string
  }
  ClipPath: {
    value: string
  }
  ClipPathUnits: {
    value: string
  }
  ClipRule: {
    value: string
  }
  Color: {
    value: string
  }
  Cols: {
    value: number
  }
  Colspan: {
    value: number
  }
  ContentAttr: {
    value: string
  }
  Contenteditable: {
    value: string
  }
  Controls: {
    value: boolean
  }
  Crossorigin: {
    value: string
  }
  Cursor: {
    value: string
  }
  Cx: {
    value: string
  }
  Cy: {
    value: string
  }
  D: {
    value: string
  }
  DataAttribute: {
    key: string
    value: string
  }
  Datetime: {
    value: string
  }
  Decoding: {
    value: string
  }
  Dir: {
    value: string
  }
  Disabled: {
    value: boolean
  }
  Display: {
    value: string
  }
  DominantBaseline: {
    value: string
  }
  Download: {
    value: string
  }
  Draggable: {
    value: boolean
  }
  Dx: {
    value: string
  }
  Dy: {
    value: string
  }
  Enctype: {
    value: string
  }
  EnterKeyHint: {
    value: string
  }
  Fetchpriority: {
    value: string
  }
  Fill: {
    value: string
  }
  FillOpacity: {
    value: string
  }
  FillRule: {
    value: string
  }
  Filter: {
    value: string
  }
  FilterUnits: {
    value: string
  }
  FontFamily: {
    value: string
  }
  FontSize: {
    value: string
  }
  FontStyle: {
    value: string
  }
  FontWeight: {
    value: string
  }
  For: {
    value: string
  }
  Formaction: {
    value: string
  }
  FormAttr: {
    value: string
  }
  Formenctype: {
    value: string
  }
  Formmethod: {
    value: string
  }
  Formnovalidate: {
    value: boolean
  }
  Formtarget: {
    value: string
  }
  Fr: {
    value: string
  }
  Fx: {
    value: string
  }
  Fy: {
    value: string
  }
  GradientTransform: {
    value: string
  }
  GradientUnits: {
    value: string
  }
  Headers: {
    value: string
  }
  Height: {
    value: string
  }
  Hidden: {
    value: boolean
  }
  High: {
    value: number
  }
  Href: {
    value: string
  }
  Hreflang: {
    value: string
  }
  HttpEquiv: {
    value: string
  }
  Id: {
    value: string
  }
  ImageRendering: {
    value: string
  }
  Inert: {
    value: boolean
  }
  InnerHTML: {
    value: string
  }
  InputMode: {
    value: string
  }
  Integrity: {
    value: string
  }
  Ismap: {
    value: boolean
  }
  Key: {
    value: string
  }
  LabelAttr: {
    value: string
  }
  Lang: {
    value: string
  }
  LengthAdjust: {
    value: string
  }
  LetterSpacing: {
    value: string
  }
  List: {
    value: string
  }
  Loading: {
    value: string
  }
  Loop: {
    value: boolean
  }
  Low: {
    value: number
  }
  MarkerEnd: {
    value: string
  }
  MarkerHeight: {
    value: string
  }
  MarkerMid: {
    value: string
  }
  MarkerStart: {
    value: string
  }
  MarkerUnits: {
    value: string
  }
  MarkerWidth: {
    value: string
  }
  Mask: {
    value: string
  }
  MaskContentUnits: {
    value: string
  }
  MaskUnits: {
    value: string
  }
  Max: {
    value: string
  }
  Maxlength: {
    value: number
  }
  Method: {
    value: string
  }
  Min: {
    value: string
  }
  Minlength: {
    value: number
  }
  Multiple: {
    value: boolean
  }
  Muted: {
    value: boolean
  }
  Name: {
    value: string
  }
  Novalidate: {
    value: boolean
  }
  Offset: {
    value: string
  }
  OnAnimationEnd: {
    message: Message
  }
  OnAnimationIteration: {
    message: Message
  }
  OnAnimationStart: {
    message: Message
  }
  OnBlur: {
    message: Message
  }
  OnCancel: {
    message: Message
  }
  OnChange: {
    f: (value: string) => Message
  }
  OnClick: {
    message: Message
  }
  OnClickFocus: {
    focusSelector: string
    message: Message
  }
  OnContextMenu: {
    message: Message
  }
  OnCopy: {
    message: Message
  }
  OnCopyText: {
    text: string
  }
  OnCustomEvent: {
    f: (event: CustomEvent<any>) => Message
    name: string
  }
  OnCut: {
    message: Message
  }
  OnCutText: {
    message: Message
    text: string
  }
  OnDoubleClick: {
    message: Message
  }
  OnDrag: {
    message: Message
  }
  OnDragEnd: {
    message: Message
  }
  OnDragEnter: {
    message: Message
  }
  OnDragLeave: {
    message: Message
  }
  OnDragOver: {
    message: Message
  }
  OnDragStart: {
    message: Message
  }
  OnDrop: {
    message: Message
  }
  OnDropFiles: {
    f: (files: ReadonlyArray<File>) => Message
  }
  OnEnded: {
    message: Message
  }
  OnError: {
    message: Message
  }
  OnFileChange: {
    f: (files: ReadonlyArray<File>) => Message
  }
  OnFocus: {
    message: Message
  }
  OnInput: {
    f: (value: string) => Message
  }
  OnKeyDown: {
    f: (key: string, modifiers: KeyboardModifiers) => Message
  }
  OnKeyDownFocus: {
    f: (key: string, modifiers: KeyboardModifiers) => Option.Option<Readonly<{
      focusSelector: string
      message: Message
    }>>
  }
  OnKeyDownPreventDefault: {
    f: (key: string, modifiers: KeyboardModifiers) => Option.Option<Message>
  }
  OnKeyPress: {
    f: (key: string, modifiers: KeyboardModifiers) => Message
  }
  OnKeyUp: {
    f: (key: string, modifiers: KeyboardModifiers) => Message
  }
  OnKeyUpPreventDefault: {
    f: (key: string, modifiers: KeyboardModifiers) => Option.Option<Message>
  }
  OnLoad: {
    message: Message
  }
  OnMount: {
    action: MountAction<Message, any>
  }
  OnMouseDown: {
    message: Message
  }
  OnMouseEnter: {
    message: Message
  }
  OnMouseLeave: {
    message: Message
  }
  OnMouseMove: {
    message: Message
  }
  OnMouseOut: {
    message: Message
  }
  OnMouseOver: {
    message: Message
  }
  OnMouseUp: {
    message: Message
  }
  OnPaste: {
    message: Message
  }
  OnPastePreventDefault: {
    f: (text: string) => Option.Option<Message>
  }
  OnPause: {
    message: Message
  }
  OnPlay: {
    message: Message
  }
  OnPointerDown: {
    f: (pointerType: string, button: number, screenX: number, screenY: number, timeStamp: number, clientX: number, clientY: number) => Option.Option<Message>
  }
  OnPointerLeave: {
    f: (pointerType: string) => Option.Option<Message>
  }
  OnPointerMove: {
    f: (screenX: number, screenY: number, pointerType: string) => Option.Option<Message>
  }
  OnPointerUp: {
    f: (screenX: number, screenY: number, pointerType: string, timeStamp: number) => Option.Option<Message>
  }
  OnReset: {
    message: Message
  }
  OnScroll: {
    f: (scrollTop: number) => Message
  }
  OnSelect: {
    message: Message
  }
  OnSubmit: {
    message: Message
  }
  OnTimeUpdate: {
    message: Message
  }
  OnToggle: {
    f: (isOpen: boolean) => Message
  }
  OnTouchCancel: {
    message: Message
  }
  OnTouchEnd: {
    message: Message
  }
  OnTouchMove: {
    message: Message
  }
  OnTouchStart: {
    message: Message
  }
  OnTransitionEnd: {
    message: Message
  }
  OnUnmount: {
    message: Message
  }
  OnVolumeChange: {
    message: Message
  }
  OnWheel: {
    message: Message
  }
  Opacity: {
    value: string
  }
  Open: {
    value: boolean
  }
  Optimum: {
    value: number
  }
  Orient: {
    value: string
  }
  Overflow: {
    value: string
  }
  PaintOrder: {
    value: string
  }
  PathLength: {
    value: string
  }
  Pattern: {
    value: string
  }
  PatternContentUnits: {
    value: string
  }
  PatternTransform: {
    value: string
  }
  PatternUnits: {
    value: string
  }
  Ping: {
    value: string
  }
  Placeholder: {
    value: string
  }
  Playsinline: {
    value: boolean
  }
  PointerEvents: {
    value: string
  }
  Points: {
    value: string
  }
  Popover: {
    value: string
  }
  Popovertarget: {
    value: string
  }
  Popovertargetaction: {
    value: string
  }
  Poster: {
    value: string
  }
  Preload: {
    value: string
  }
  PreserveAspectRatio: {
    value: string
  }
  PrimitiveUnits: {
    value: string
  }
  Prop: {
    key: string
    value: unknown
  }
  R: {
    value: string
  }
  Readonly: {
    value: boolean
  }
  Referrerpolicy: {
    value: string
  }
  RefX: {
    value: string
  }
  RefY: {
    value: string
  }
  Rel: {
    value: string
  }
  Required: {
    value: boolean
  }
  Reversed: {
    value: boolean
  }
  Role: {
    value: string
  }
  Rotate: {
    value: string
  }
  Rows: {
    value: number
  }
  Rowspan: {
    value: number
  }
  Rx: {
    value: string
  }
  Ry: {
    value: string
  }
  Sandbox: {
    value: string
  }
  Scope: {
    value: string
  }
  Selected: {
    value: boolean
  }
  ShapeRendering: {
    value: string
  }
  Size: {
    value: number
  }
  Sizes: {
    value: string
  }
  Span: {
    value: number
  }
  Spellcheck: {
    value: boolean
  }
  SpreadMethod: {
    value: string
  }
  Src: {
    value: string
  }
  Srcdoc: {
    value: string
  }
  Srcset: {
    value: string
  }
  Start: {
    value: number
  }
  Step: {
    value: string
  }
  StopColor: {
    value: string
  }
  StopOpacity: {
    value: string
  }
  Stroke: {
    value: string
  }
  StrokeDasharray: {
    value: string
  }
  StrokeDashoffset: {
    value: string
  }
  StrokeLinecap: {
    value: string
  }
  StrokeLinejoin: {
    value: string
  }
  StrokeMiterlimit: {
    value: string
  }
  StrokeOpacity: {
    value: string
  }
  StrokeWidth: {
    value: string
  }
  Style: {
    value: Record<string, string>
  }
  Tabindex: {
    value: number
  }
  Target: {
    value: string
  }
  TextAnchor: {
    value: string
  }
  TextDecoration: {
    value: string
  }
  TextLength: {
    value: string
  }
  TextRendering: {
    value: string
  }
  Title: {
    value: string
  }
  Transform: {
    value: string
  }
  Translate: {
    value: string
  }
  Type: {
    value: string
  }
  Usemap: {
    value: string
  }
  Value: {
    value: string
  }
  VectorEffect: {
    value: string
  }
  ViewBox: {
    value: string
  }
  Visibility: {
    value: string
  }
  Width: {
    value: string
  }
  WordSpacing: {
    value: string
  }
  Wrap: {
    value: string
  }
  WritingMode: {
    value: string
  }
  X: {
    value: string
  }
  X1: {
    value: string
  }
  X2: {
    value: string
  }
  Xmlns: {
    value: string
  }
  Y: {
    value: string
  }
  Y1: {
    value: string
  }
  Y2: {
    value: string
  }
}>
```

### ChildAttribute

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/childAttribute.ts#L27)

```
/**
 * An attribute carrying a handler that dispatches through a Submodel
 *  boundary's wrapping chain. Published by Submodels (typically Foldkit's
 *  `Ui.*` components) for a parent to spread into its own element
 *  attribute arrays. The parent does not know or care which child
 *  produced these; the runtime routes each handler through the
 *  originating Submodel's wrap chain at event-fire time.
 * 
 *  `resolveUnmount` snapshots the boundary's wrapping chain at the time the
 *  group was published (child boundary alive) so `OnUnmount` can dispatch a
 *  root message from a destroy hook that fires after the boundary has been
 *  torn down. `boundaryMappers` snapshots the same chain as a pure list of
 *  `toParentMessage` lifts (innermost first) so `OnMount` can stamp it on the
 *  mount marker; the Scene test harness folds it to replay the lift.
 * 
 *  Created via childAttributes. Element constructors accept
 *  `ChildAttribute` alongside `Attribute<Message>` in their attribute
 *  arrays.
 */
type ChildAttribute = Readonly<{
  __childAttribute: true
  attribute: unknown
  boundaryMappers: ReadonlyArray<(message: unknown) => unknown>
  dispatch: DispatchSync
  resolveUnmount: (message: unknown) => () => void
}>
```

### Document

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/index.ts#L163)

```
/**
 * A view's complete output for the runtime: title, body, and optional document
 *  metadata. The runtime applies `title` to `document.title`, syncs `lang` and
 *  `dir` to the `<html>` element, syncs `canonical` to `<link rel="canonical">`
 *  (creating it if absent), syncs `ogUrl` to `<meta property="og:url">`
 *  (creating it if absent), and patches `body` into the application container.
 * 
 *  When `canonical` is omitted, it defaults to the current URL (origin +
 *  pathname + search). When `ogUrl` is omitted, it falls back to `canonical`.
 * 
 *  `lang` and `dir` have no default. When either is omitted the runtime does not
 *  touch that attribute, leaving whatever value it currently holds, so a view
 *  that never sets it leaves the served HTML in place. Drive them from the Model
 *  when the app switches language at runtime. The served HTML still decides what
 *  a crawler sees on first paint, because the runtime can only sync after the
 *  first render.
 * 
 *  This is the return type of a `makeApplication` view, which owns the document. An
 *  app embedded at a node should use `makeElement` instead, whose view returns
 *  `Html` and never touches the `<head>` or the `<html>` element.
 */
type Document = Readonly<{
  body: Html
  canonical: string
  dir: TextDirection
  lang: string
  ogUrl: string
  title: string
}>
```

### Html

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/index.ts#L134)

```
/**
 * A virtual DOM element. Constructed synchronously by the element factories
 *  returned from html. The runtime patches a `VNode` (or `null` to
 *  render nothing) into the application container.
 */
type Html = VNode | null
```

### HtmlBuilder

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/index.ts#L4459)

```
/**
 * The typed Html builder a view builds DOM with: all HTML, SVG, and MathML
 * element constructors, attribute constructors, a `keyed` helper for keyed
 * elements, `empty` for rendering nothing, and `submodel` for embedding a
 * child Submodel.
 * 
 * A builder cannot be constructed by application code. The runtime supplies
 * it to each view alongside the model, typed by the Message universe of the
 * frame that view renders in: the app's Message for the root view, the
 * Submodel's own Message inside a `Submodel.defineView`. Used in the frame
 * that supplied it, a handler built with it carries exactly the Messages that
 * frame's dispatcher can route, so a Message from another universe (the
 * classic case: a shared helper building an app-level Message inside a
 * Submodel) is a compile error at the handler call site.
 * 
 * The type scopes where a builder is obtained, not where it is used. Carrying
 * one into another frame, by storing it or handing the builder itself to a
 * child through `viewInputs`, still compiles and still builds handlers that
 * frame cannot route. Thread `h` as a parameter; never store it.
 * 
 * Pass `h` along as an ordinary parameter when extracting view helpers; a
 * memoized helper receives it through the `createLazy` args array. When a
 * child must render markup that belongs to an ancestor, the ancestor passes a
 * renderer that already closed over its own builder, not the builder, so the
 * handlers resolve in the ancestor's boundary. Where no builder is in scope
 * at all, typically module scope, use inertHtml.
 */
type HtmlBuilder = MessageUniverse<Message> & HtmlElements<Message> & HtmlAttributes<Message> & Readonly<{
  empty: null
  keyed: (tagName: TagName) => (key: PropertyKey, attributes?: ReadonlyArray<Attribute<Message> | ChildAttribute>, children?: ReadonlyArray<Child>) => Html
  submodel: (config: SubmodelConfig<View, Message>) => Html
}>
```

### KeyboardModifiers

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/index.ts#L117)

```
/** Modifier key state extracted from a `KeyboardEvent`. */
type KeyboardModifiers = Readonly<{
  altKey: boolean
  ctrlKey: boolean
  metaKey: boolean
  shiftKey: boolean
}>
```

### TagName

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/index.ts#L173)

```
/** Union of all valid HTML, SVG, and MathML tag names. */
type TagName = "a" | "abbr" | "address" | "area" | "article" | "aside" | "audio" | "b" | "base" | "bdi" | "bdo" | "blockquote" | "body" | "br" | "button" | "canvas" | "caption" | "cite" | "code" | "col" | "colgroup" | "data" | "datalist" | "dd" | "del" | "details" | "dfn" | "dialog" | "div" | "dl" | "dt" | "em" | "embed" | "fieldset" | "figcaption" | "figure" | "footer" | "form" | "h1" | "h2" | "h3" | "h4" | "h5" | "h6" | "head" | "header" | "hgroup" | "hr" | "html" | "i" | "iframe" | "img" | "input" | "ins" | "kbd" | "label" | "legend" | "li" | "link" | "main" | "map" | "mark" | "menu" | "meta" | "meter" | "nav" | "noscript" | "object" | "ol" | "optgroup" | "option" | "output" | "p" | "picture" | "portal" | "pre" | "progress" | "q" | "rp" | "rt" | "ruby" | "s" | "samp" | "script" | "search" | "section" | "select" | "slot" | "small" | "source" | "span" | "strong" | "style" | "sub" | "summary" | "sup" | "table" | "tbody" | "td" | "template" | "textarea" | "tfoot" | "th" | "thead" | "time" | "title" | "tr" | "track" | "u" | "ul" | "var" | "video" | "wbr" | "animate" | "animateMotion" | "animateTransform" | "circle" | "clipPath" | "defs" | "desc" | "ellipse" | "feBlend" | "feColorMatrix" | "feComponentTransfer" | "feComposite" | "feConvolveMatrix" | "feDiffuseLighting" | "feDisplacementMap" | "feDistantLight" | "feDropShadow" | "feFlood" | "feFuncA" | "feFuncB" | "feFuncG" | "feFuncR" | "feGaussianBlur" | "feImage" | "feMerge" | "feMergeNode" | "feMorphology" | "feOffset" | "fePointLight" | "feSpecularLighting" | "feSpotLight" | "feTile" | "feTurbulence" | "filter" | "foreignObject" | "g" | "image" | "line" | "linearGradient" | "marker" | "mask" | "metadata" | "mpath" | "path" | "pattern" | "polygon" | "polyline" | "radialGradient" | "rect" | "set" | "stop" | "svg" | "switch" | "symbol" | "text" | "textPath" | "tspan" | "use" | "view" | "annotation" | "annotation-xml" | "math" | "maction" | "menclose" | "merror" | "mfenced" | "mfrac" | "mglyph" | "mi" | "mlabeledtr" | "mlongdiv" | "mmultiscripts" | "mn" | "mo" | "mover" | "mpadded" | "mphantom" | "mprescripts" | "mroot" | "mrow" | "ms" | "mscarries" | "mscarry" | "msgroup" | "msline" | "mspace" | "msqrt" | "msrow" | "mstack" | "mstyle" | "msub" | "msubsup" | "msup" | "mtable" | "mtd" | "mtext" | "mtr" | "munder" | "munderover" | "semantics"
```

## Constants

### TextDirection

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/index.ts#L139)

```
/**
 * Text direction for the document root, applied to `dir` on the `<html>`
 *  element. `Auto` defers to the browser's first-strong-character heuristic.
 */
const TextDirection: Literals<readonly ["Ltr", "Rtl", "Auto"]>
```

### inertHtml

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/html/index.ts#L4542)

```
/**
 * Builder whose Message universe is empty. Its Message type is `never`, so
 * every event-handler constructor is uncallable and nothing built with it can
 * dispatch a Message. Elements, attributes, and keying behave exactly as they
 * do on any other builder, and the markup itself is free to vary with runtime
 * data. Inert describes what the result can do, not how it is computed.
 * 
 * Inert to Foldkit's dispatch, not to the browser. A raw DOM attribute still
 * does whatever the browser makes of it, which is exactly how the default
 * crash view gets a working reload button:
 * `h.Attribute('onclick', 'location.reload()')`. What `never` rules out is a
 * Message reaching `update`, not every possible behavior.
 * 
 * Use it where no builder is in scope, which in practice means module scope
 * and the frameless renders the framework performs itself:
 * 
 * ```ts
 * import { inertHtml as ih } from 'foldkit/html'
 * 
 * const PagefindBody = ih.DataAttribute('pagefind-body', '')
 * ```
 * 
 * Attributes it produces are `Attribute<never>`, so they flow into any
 * Message universe by covariance. That makes it the right builder for
 * library code emitting handler-free attribute bundles for arbitrary apps.
 * 
 * Inside a view, use the view's own `h` parameter. A view already holds a
 * builder, and reaching past it for this one is the habit that made a
 * caller-chosen Message type possible in the first place.
 */
const inertHtml: HtmlBuilder<never>
```
