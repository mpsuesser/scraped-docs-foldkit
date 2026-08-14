---
url: https://foldkit.dev/api-reference/canvas
title: "Canvas"
description: "API documentation for the Canvas module."
access_date: 2026-08-14T23:11:22.406Z
current_date: 2026-08-14T23:11:22.406Z
---

# Canvas

## Functions

### view

function

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/view.ts#L83)

```
/**
 * A virtual DOM `<canvas>` element backed by a declarative scene description.
 * The insert hook captures the 2D context and paints the initial scene; the
 * postpatch hook re-paints on every render. The canvas is a pure function of
 * `shapes`. Same shapes produce the same pixels.
 * 
 * The trailing `h` fixes `Message` to the calling view's frame; the pointer
 * handlers dispatch through that frame at fire time, so passing the builder
 * that built the surrounding view is what makes the handler types truthful.
 */
<Message>(
  config: ViewConfig<NoInfer<Message>>,
  _h: HtmlBuilder<Message>
): Html
```

## Types

### Shape

type

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L159)

```
/**
 * A drawable scene-graph node. Composing `Group` recursively builds a tree;
 * the painter walks the tree depth-first, applying transforms via
 * `ctx.save` / `ctx.restore`.
 */
type Shape = typeof Rect.Type | typeof Circle.Type | typeof Path.Type | typeof Text.Type | Group
```

### ViewConfig

type

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/view.ts#L19)

```
/**
 * Configuration for `Canvas.view`. Pointer handlers are optional and
 * receive a `Point` already translated to the canvas's internal coordinate
 * space (the `width` and `height` passed here), independent of how the
 * canvas is sized in CSS.
 */
type ViewConfig = Readonly<{
  className: string
  height: number
  onPointerDown: (point: Point) => Message
  onPointerMove: (point: Point) => Message
  onPointerUp: (point: Point) => Message
  shapes: ReadonlyArray<Shape>
  width: number
}>
```

## Interfaces

### Group

interface

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L145)

```
/**
 * A scene-graph node that applies a 2D transform and global alpha to a list of
 * child shapes. Transforms compose multiplicatively when groups are nested.
 * 
 * Defined via the `interface` form so the recursion (`shapes: ReadonlyArray<Shape>`
 * where `Shape` includes `Group` itself) resolves under TypeScript's lazy
 * interface evaluation. The matching runtime Schema uses `S.suspend`.
 */
interface Group {
  _tag: "Group"
  opacity: number
  rotate: number
  scale: Canvas.Point
  shapes: readonly Array<Shape>
  translate: Canvas.Point
}
```

## Constants

### BezierTo

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L31)

```
/** Draw a cubic Bezier curve from the cursor through two control points to an end point. */
const BezierTo: CallableTaggedStruct<"BezierTo", {
  cp1x: Number
  cp1y: Number
  cp2x: Number
  cp2y: Number
  x: Number
  y: Number
}>
```

### Circle

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L99)

```
/** A filled or stroked circle. */
const Circle: CallableTaggedStruct<"Circle", {
  fill: optional<String>
  lineWidth: optional<Number>
  radius: Number
  stroke: optional<String>
  x: Number
  y: Number
}>
```

### Close

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L43)

```
/** Close the current path by drawing a line back to its starting point. */
const Close: CallableTaggedStruct<"Close", {}>
```

### Group

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L145)

```
/** Construct a `Group` shape that wraps its children in a transformed scope. */
const Group: CallableTaggedStruct<"Group", {
  opacity: optional<Number>
  rotate: optional<Number>
  scale: optional<Struct<{
    x: Number
    y: Number
  }>>
  shapes: $Array<Schema<Shape>>
  translate: optional<Struct<{
    x: Number
    y: Number
  }>>
}>
```

### LineCap

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L59)

```
/** Stroke cap style: how the ends of an open stroked subpath are rendered. */
const LineCap: Literals<readonly ["Butt", "Round", "Square"]>
```

### LineJoin

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L64)

```
/** Stroke join style: how two connected stroked segments meet. */
const LineJoin: Literals<readonly ["Miter", "Round", "Bevel"]>
```

### LineTo

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L16)

```
/** Draw a straight line from the cursor to a point. */
const LineTo: CallableTaggedStruct<"LineTo", {
  x: Number
  y: Number
}>
```

### MoveTo

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L11)

```
/** Move the path cursor to a point without drawing. */
const MoveTo: CallableTaggedStruct<"MoveTo", {
  x: Number
  y: Number
}>
```

### Path

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L111)

```
/** A path built from a sequence of `PathInstruction`s. */
const Path: CallableTaggedStruct<"Path", {
  fill: optional<String>
  instructions: $Array<Union<readonly [
    CallableTaggedStruct<"MoveTo", {
      x: Number
      y: Number
    }>,
    CallableTaggedStruct<"LineTo", {
      x: Number
      y: Number
    }>,
    CallableTaggedStruct<"QuadTo", {
      cpx: Number
      cpy: Number
      x: Number
      y: Number
    }>,
    CallableTaggedStruct<"BezierTo", {
      cp1x: Number
      cp1y: Number
      cp2x: Number
      cp2y: Number
      x: Number
      y: Number
    }>,
    CallableTaggedStruct<"Close", {}>
  ]>>
  lineCap: optional<Literals<readonly ["Butt", "Round", "Square"]>>
  lineJoin: optional<Literals<readonly ["Miter", "Round", "Bevel"]>>
  lineWidth: optional<Number>
  stroke: optional<String>
}>
```

### PathInstruction

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L48)

```
/** A single drawing instruction within a `Path` shape. */
const PathInstruction: Union<readonly [
  CallableTaggedStruct<"MoveTo", {
    x: Number
    y: Number
  }>,
  CallableTaggedStruct<"LineTo", {
    x: Number
    y: Number
  }>,
  CallableTaggedStruct<"QuadTo", {
    cpx: Number
    cpy: Number
    x: Number
    y: Number
  }>,
  CallableTaggedStruct<"BezierTo", {
    cp1x: Number
    cp1y: Number
    cp2x: Number
    cp2y: Number
    x: Number
    y: Number
  }>,
  CallableTaggedStruct<"Close", {}>
]>
```

### Point

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L6)

```
/** A 2D point in canvas-local coordinates. */
const Point: Struct<{
  x: Number
  y: Number
}>
```

### QuadTo

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L21)

```
/** Draw a quadratic Bezier curve from the cursor through a control point to an end point. */
const QuadTo: CallableTaggedStruct<"QuadTo", {
  cpx: Number
  cpy: Number
  x: Number
  y: Number
}>
```

### Rect

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L86)

```
/** An axis-aligned rectangle. */
const Rect: CallableTaggedStruct<"Rect", {
  fill: optional<String>
  height: Number
  lineWidth: optional<Number>
  stroke: optional<String>
  width: Number
  x: Number
  y: Number
}>
```

### Text

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L123)

```
/** A single line of text drawn with a font, fill, and optional stroke. */
const Text: CallableTaggedStruct<"Text", {
  align: optional<Literals<readonly ["Left", "Center", "Right", "Start", "End"]>>
  baseline: optional<Literals<readonly ["Top", "Middle", "Bottom", "Alphabetic", "Hanging", "Ideographic"]>>
  content: String
  fill: optional<String>
  font: optional<String>
  lineWidth: optional<Number>
  stroke: optional<String>
  x: Number
  y: Number
}>
```

### TextAlign

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L69)

```
/** Horizontal alignment of a `Text` shape relative to its anchor x coordinate. */
const TextAlign: Literals<readonly ["Left", "Center", "Right", "Start", "End"]>
```

### TextBaseline

const

[source](https://github.com/foldkit/foldkit/blob/86c1608ac269f98af7f09d6e4e7bcfead03b9ddb/packages/foldkit/src/canvas/shape.ts#L74)

```
/** Vertical alignment of a `Text` shape relative to its anchor y coordinate. */
const TextBaseline: Literals<readonly ["Top", "Middle", "Bottom", "Alphabetic", "Hanging", "Ideographic"]>
```
