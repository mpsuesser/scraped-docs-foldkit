---
url: https://foldkit.dev/api-reference/ui-animation
title: "Ui/Animation"
description: "API documentation for the Ui/Animation module."
access_date: 2026-09-18T04:36:53.681Z
current_date: 2026-09-18T04:36:53.681Z
---

# Ui/Animation

## Functions

### defaultLeaveCommand

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/update.ts#L134)

```
/** Creates the standard leave-phase command that waits for CSS animations on the element to settle. Use this when handling the `StartedLeaveAnimating` OutMessage for components that don't need custom leave behavior. */
(model: Animation.Model): Command.Command<Message>
```

### hide

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/update.ts#L121)

```
/** Programmatically starts the leave lifecycle. */
(model: Animation.Model): Update.Return<Model, Message>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/schema.ts#L58)

```
/** Creates an initial animation model from a config. Defaults to hidden. */
(config: InitConfig): Animation.Model
```

### show

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/update.ts#L117)

```
/** Programmatically starts the enter lifecycle. */
(model: Animation.Model): Update.Return<Model, Message>
```

### toggle

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/update.ts#L125)

```
/** Toggles the animation between its shown and hidden states. */
(model: Animation.Model): Update.Return<Model, Message>
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/update.ts#L46)

```
/**
 * Processes an Animation Message and returns the next Model, optional
 *  Commands, and an optional OutMessage. `Showed` and `Hid` start a transition
 *  but cannot finish one, so direct calls with either Message return a plain
 *  update result.
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
  } | {
    _tag: "EndedAnimation"
  }
): UpdateReturn
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/schema.ts#L52)

```
/** Configuration for creating an animation model with `init`. */
type InitConfig = Readonly<{
  id: string
  isShowing: boolean
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/index.ts#L32)

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

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/schema.ts#L30)

```
/** Union of all messages the animation component can produce. */
const Message: MessageUnion<{
  CompletedWaitForPaint: {}
  EndedAnimation: {}
  Hid: {}
  Showed: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/schema.ts#L19)

```
/** Schema for the animation component's state, tracking its unique ID, visibility intent, and lifecycle phase. */
const Model: Struct<{
  id: String
  isShowing: Boolean
  transitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/schema.ts#L43)

```
const OutMessage: MessageUnion<{
  StartedLeaveAnimating: {}
  TransitionedOut: {}
}>
```

### TransitionState

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/schema.ts#L7)

```
/** Schema for the animation lifecycle state, tracking enter/leave phases. */
const TransitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
```

### WaitForAnimationSettled

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/update.ts#L30)

```
/** Waits for all CSS animations on the element to settle. Covers both CSS transitions and CSS keyframe animations. */
const WaitForAnimationSettled: CommandDefinitionWithArgs<"WaitForAnimationSettled", {
  id: String
}, Effect<{
  _tag: "EndedAnimation"
}, never, never>>
```

### WaitForPaint

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/update.ts#L25)

```
/** Waits for paint via double-rAF before the enter/leave lifecycle advances. */
const WaitForPaint: CommandDefinitionNoArgs<"WaitForPaint", Effect<{
  _tag: "CompletedWaitForPaint"
}, never, never>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/7a3180b90d4eff772fa22928f31009e7dda9ff8f/packages/ui/src/animation/index.ts#L52)

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
} | {
  _tag: "EndedAnimation"
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
  element: "symbol" | "object" | "portal" | "div" | "a" | "abbr" | "address" | "area" | "article" | "aside" | "audio" | "b" | "base" | "bdi" | "bdo" | "blockquote" | "body" | "br" | "button" | "canvas" | "caption" | "cite" | "code" | "col" | "colgroup" | "data" | "datalist" | "dd" | "del" | "details" | "dfn" | "dialog" | "dl" | "dt" | "em" | "embed" | "fieldset" | "figcaption" | "figure" | "footer" | "form" | "h1" | "h2" | "h3" | "h4" | "h5" | "h6" | "head" | "header" | "hgroup" | "hr" | "html" | "i" | "iframe" | "img" | "input" | "ins" | "kbd" | "label" | "legend" | "li" | "link" | "main" | "map" | "mark" | "menu" | "meta" | "meter" | "nav" | "noscript" | "ol" | "optgroup" | "option" | "output" | "p" | "picture" | "pre" | "progress" | "q" | "rp" | "rt" | "ruby" | "s" | "samp" | "script" | "search" | "section" | "select" | "slot" | "small" | "source" | "span" | "strong" | "style" | "sub" | "summary" | "sup" | "table" | "tbody" | "td" | "template" | "tfoot" | "th" | "thead" | "time" | "title" | "tr" | "track" | "u" | "ul" | "var" | "video" | "wbr" | "filter" | "animate" | "animateMotion" | "animateTransform" | "circle" | "clipPath" | "defs" | "desc" | "ellipse" | "feBlend" | "feColorMatrix" | "feComponentTransfer" | "feComposite" | "feConvolveMatrix" | "feDiffuseLighting" | "feDisplacementMap" | "feDistantLight" | "feDropShadow" | "feFlood" | "feFuncA" | "feFuncB" | "feFuncG" | "feFuncR" | "feGaussianBlur" | "feImage" | "feMerge" | "feMergeNode" | "feMorphology" | "feOffset" | "fePointLight" | "feSpecularLighting" | "feSpotLight" | "feTile" | "feTurbulence" | "foreignObject" | "g" | "image" | "line" | "linearGradient" | "marker" | "mask" | "metadata" | "mpath" | "path" | "pattern" | "polygon" | "polyline" | "radialGradient" | "rect" | "set" | "stop" | "svg" | "switch" | "text" | "textPath" | "tspan" | "use" | "view" | "annotation" | "annotation-xml" | "maction" | "math" | "merror" | "mfrac" | "mi" | "mmultiscripts" | "mn" | "mo" | "mover" | "mpadded" | "mphantom" | "mprescripts" | "mroot" | "mrow" | "ms" | "mspace" | "msqrt" | "mstyle" | "msub" | "msubsup" | "msup" | "mtable" | "mtd" | "mtext" | "mtr" | "munder" | "munderover" | "semantics" | "menclose" | "mfenced" | "mglyph" | "mlabeledtr" | "mlongdiv" | "mscarries" | "mscarry" | "msgroup" | "msline" | "msrow" | "mstack"
}>>
```
