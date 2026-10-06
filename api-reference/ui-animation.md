---
url: https://foldkit.dev/api-reference/ui-animation
title: "Ui/Animation"
description: "API documentation for the Ui/Animation module."
access_date: 2026-10-06T02:21:45.877Z
current_date: 2026-10-06T02:21:45.877Z
---

# Ui/Animation

## Functions

### defaultLeaveCommand

function

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/update.ts#L172)

```
/**
 * Creates the standard leave Command for the Model's current transition. Use
 *  this when handling `StartedLeaveAnimating` unless the component needs its
 *  own settlement strategy.
 */
(model: Animation.Model): Command.Command<Message>
```

### hide

function

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/update.ts#L157)

```
/** Programmatically starts the leave lifecycle. */
(model: Animation.Model): Update.Return<Model, Message>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/schema.ts#L61)

```
/** Creates an initial animation model from a config. Defaults to hidden. */
(config: InitConfig): Animation.Model
```

### show

function

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/update.ts#L153)

```
/** Programmatically starts the enter lifecycle. */
(model: Animation.Model): Update.Return<Model, Message>
```

### toggle

function

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/update.ts#L161)

```
/** Toggles the animation between its shown and hidden states. */
(model: Animation.Model): Update.Return<Model, Message>
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/update.ts#L53)

```
/**
 * Processes an Animation Message and returns the next Model, optional
 *  Commands, and an optional OutMessage. `Showed` and `Hid` start a transition
 *  but cannot finish one, so direct calls with either Message return a plain
 *  update result. Results from an earlier transition generation leave the Model
 *  unchanged.
 */
(
  model: Animation.Model,
  message: {
    _tag: "Showed"
  } | {
    _tag: "Hid"
  }
): Update.Return<Model, Message>

(
  model: Animation.Model,
  message: {
    _tag: "Showed"
  } | {
    _tag: "Hid"
  } | {
    _tag: "CompletedWaitForPaint"
    generation: number
  } | {
    _tag: "EndedAnimation"
    generation: number
  }
): UpdateReturn
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/schema.ts#L55)

```
/** Configuration for creating an animation model with `init`. */
type InitConfig = Readonly<{
  id: string
  isShowing: boolean
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/index.ts#L32)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  animateSize: boolean
  attributes: ReadonlyArray<ChildAttribute>
  className: string
  content: Html
  element: Exclude<TagName, "textarea">
}>
```

## Constants

### Message

const

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/schema.ts#L32)

```
/** Union of all messages the animation component can produce. */
const Message: MessageUnion<{
  CompletedWaitForPaint: {
    generation: Number
  }
  EndedAnimation: {
    generation: Number
  }
  Hid: {}
  Showed: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/schema.ts#L20)

```
/**
 * Schema for Animation state, including the transition generation used to reject
 *  stale Command results.
 */
const Model: Struct<{
  id: String
  isShowing: Boolean
  transitionGeneration: Number
  transitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/schema.ts#L46)

```
/** Union of the facts Animation reports to its parent. */
const OutMessage: MessageUnion<{
  StartedLeaveAnimating: {
    generation: Number
  }
  TransitionedOut: {}
}>
```

### TransitionState

const

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/schema.ts#L7)

```
/** Schema for the animation lifecycle state, tracking enter/leave phases. */
const TransitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
```

### WaitForAnimationSettled

const

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/update.ts#L36)

```
/**
 * Waits for all CSS transitions and keyframe animations on the element to
 *  settle, then reports the transition generation that scheduled the wait.
 */
const WaitForAnimationSettled: CommandDefinitionWithArgs<"WaitForAnimationSettled", {
  generation: Number
  id: String
}, Effect<{
  _tag: "EndedAnimation"
  generation: number
}, never, never>>
```

### WaitForPaint

const

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/update.ts#L26)

```
/**
 * Waits for paint via double-rAF, then reports the transition generation that
 *  scheduled the wait.
 */
const WaitForPaint: CommandDefinitionWithArgs<"WaitForPaint", {
  generation: Number
}, Effect<{
  _tag: "CompletedWaitForPaint"
  generation: number
}, never, never>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/03baf1666c9902e94a755e4ad1b0c44207453f9f/packages/ui/src/animation/index.ts#L52)

```
/**
 * Renders a headless animation wrapper that coordinates CSS transitions and
 *  CSS keyframe animations via data attributes.
 * 
 *  Data attributes reflect the current lifecycle phase:
 *  - `data-closed`: element is in its hidden/initial state
 *  - `data-enter`: enter animation is active
 *  - `data-leave`: leave animation is active
 *  - `data-transition`: any animation is active
 */
const view: SubmodelView<Animation.Model, {
  _tag: "Showed"
} | {
  _tag: "Hid"
} | {
  _tag: "CompletedWaitForPaint"
  generation: number
} | {
  _tag: "EndedAnimation"
  generation: number
}, Readonly<{
  animateSize: boolean
  attributes: readonly Array<Readonly<{
    __childAttribute: true
    attribute: unknown
    boundaryMappers: readonly Array<(message: unknown) => unknown>
    dispatch: DispatchSync
    resolveMountDispatch: Readonly<{
      owner: MountRenderOwner
      resolve: () => unknown
    }>
    resolveUnmount: (message: unknown) => () => void
  }>>
  className: string
  content: Html
  element: "symbol" | "object" | "portal" | "div" | "a" | "abbr" | "address" | "area" | "article" | "aside" | "audio" | "b" | "base" | "bdi" | "bdo" | "blockquote" | "body" | "br" | "button" | "canvas" | "caption" | "cite" | "code" | "col" | "colgroup" | "data" | "datalist" | "dd" | "del" | "details" | "dfn" | "dialog" | "dl" | "dt" | "em" | "embed" | "fieldset" | "figcaption" | "figure" | "footer" | "form" | "h1" | "h2" | "h3" | "h4" | "h5" | "h6" | "head" | "header" | "hgroup" | "hr" | "html" | "i" | "iframe" | "img" | "input" | "ins" | "kbd" | "label" | "legend" | "li" | "link" | "main" | "map" | "mark" | "menu" | "meta" | "meter" | "nav" | "noscript" | "ol" | "optgroup" | "option" | "output" | "p" | "picture" | "pre" | "progress" | "q" | "rp" | "rt" | "ruby" | "s" | "samp" | "script" | "search" | "section" | "select" | "slot" | "small" | "source" | "span" | "strong" | "style" | "sub" | "summary" | "sup" | "table" | "tbody" | "td" | "template" | "tfoot" | "th" | "thead" | "time" | "title" | "tr" | "track" | "u" | "ul" | "var" | "video" | "wbr" | "animate" | "animateMotion" | "animateTransform" | "circle" | "clipPath" | "defs" | "desc" | "ellipse" | "feBlend" | "feColorMatrix" | "feComponentTransfer" | "feComposite" | "feConvolveMatrix" | "feDiffuseLighting" | "feDisplacementMap" | "feDistantLight" | "feDropShadow" | "feFlood" | "feFuncA" | "feFuncB" | "feFuncG" | "feFuncR" | "feGaussianBlur" | "feImage" | "feMerge" | "feMergeNode" | "feMorphology" | "feOffset" | "fePointLight" | "feSpecularLighting" | "feSpotLight" | "feTile" | "feTurbulence" | "filter" | "foreignObject" | "g" | "image" | "line" | "linearGradient" | "marker" | "mask" | "metadata" | "mpath" | "path" | "pattern" | "polygon" | "polyline" | "radialGradient" | "rect" | "set" | "stop" | "svg" | "switch" | "text" | "textPath" | "tspan" | "use" | "view" | "annotation" | "annotation-xml" | "maction" | "math" | "merror" | "mfrac" | "mi" | "mmultiscripts" | "mn" | "mo" | "mover" | "mpadded" | "mphantom" | "mprescripts" | "mroot" | "mrow" | "ms" | "mspace" | "msqrt" | "mstyle" | "msub" | "msubsup" | "msup" | "mtable" | "mtd" | "mtext" | "mtr" | "munder" | "munderover" | "semantics" | "menclose" | "mfenced" | "mglyph" | "mlabeledtr" | "mlongdiv" | "mscarries" | "mscarry" | "msgroup" | "msline" | "msrow" | "mstack"
}>>
```
