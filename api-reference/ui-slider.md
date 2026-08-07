---
url: https://foldkit.dev/api-reference/ui-slider
title: "Ui/Slider"
description: "API documentation for the Ui/Slider module."
access_date: 2026-08-07T15:54:22.903Z
current_date: 2026-08-07T15:54:22.903Z
---

# Ui/Slider

## Functions

### fractionOfValue

function

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L174)

```
/**
 * Computes the fraction (0–1) of a value between min and max. Returns 0 when
 *  the range has zero width.
 */
(
  value: number,
  min: number,
  max: number
): number
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L129)

```
/**
 * Creates an initial slider model from a config. The value lives in the
 *  parent Model; initialize it there and snap it with snapAndClamp.
 */
(config: InitConfig): Slider.Model
```

### snapAndClamp

function

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L162)

```
/**
 * Snaps a value to the nearest step and clamps it into `[min, max]`. Exported
 *  so a parent can conform the value it owns to the slider's range, for example
 *  when seeding the initial value or reacting to an external update.
 */
(
  value: number,
  min: number,
  max: number,
  step: number
): number
```

### subscriptionsForRoot

function

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L382)

### update

function

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L232)

```
/**
 * Processes a slider message and returns the next model, commands, and an
 *  optional out-message for the parent. The value lives in the parent Model:
 *  the view supplies the current value on the messages that need it, and value
 *  changes surface as `ChangedValue` rather than mutating this Model.
 */
(
  model: Slider.Model,
  message: {
    _tag: "CancelledDrag"
  } | {
    _tag: "PressedThumb"
    originValue: number
  } | {
    _tag: "PressedPointer"
    originValue: number
    value: number
  } | {
    _tag: "MovedDragPointer"
    value: number
  } | {
    _tag: "ReleasedDragPointer"
  } | {
    _tag: "PressedKeyboardNavigation"
    direction: "Max" | "Min" | "StepDecrement" | "StepIncrement" | "PageDecrement" | "PageIncrement"
    value: number
  }
): UpdateReturn
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L120)

```
/** Configuration for creating a slider model with `init`. */
type InitConfig = Readonly<{
  id: string
  max: number
  min: number
  step: number
}>
```

### SliderAttributes

type

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L508)

```
/**
 * Attribute groups the slider component provides to the consumer's `toView`
 *  callback. Each bundle carries the boundary's captured dispatch, so the
 *  consumer can spread it directly into element attributes without manual
 *  Message wrapping.
 */
type SliderAttributes = Readonly<{
  filledTrack: ReadonlyArray<ChildAttribute>
  hiddenInput: ReadonlyArray<ChildAttribute>
  label: ReadonlyArray<ChildAttribute>
  root: ReadonlyArray<ChildAttribute>
  thumb: ReadonlyArray<ChildAttribute>
  track: ReadonlyArray<ChildAttribute>
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L518)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  ariaLabel: string
  ariaLabelledBy: string
  formatValue: (value: number) => string
  getTrackRoot: () => Document | ShadowRoot
  isDisabled: boolean
  name: string
  toView: (attributes: SliderAttributes) => Html
  value: number
}>
```

## Constants

### CancelledDrag

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L63)

```
/** Escape was pressed during a drag. Restores the value from the drag origin. */
const CancelledDrag: CallableTaggedStruct<"CancelledDrag", {}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L79)

```
/** Union of all messages the slider component can produce. */
const Message: S.Union<[typeof PressedThumb, typeof PressedPointer, typeof MovedDragPointer, typeof ReleasedDragPointer, typeof CancelledDrag, typeof PressedKeyboardNavigation]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L34)

```
/**
 * Schema for the slider component's private interaction state. The current
 *  value is owned by the parent and passed in via `ViewInputs.value`, so it is
 *  not stored here. `min`/`max`/`step` are configuration the drag subscription
 *  reads to map pointer positions into values. `dragState` tracks the active
 *  drag phase and captures the pre-drag value so Escape can restore it.
 */
const Model: Struct<{
  dragState: Union<readonly [
    CallableTaggedStruct<"Idle", {}>,
    CallableTaggedStruct<"Dragging", {
      originValue: Number
    }>
  ]>
  id: String
  max: Number
  min: Number
  step: Number
}>
```

### MovedDragPointer

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L59)

```
/**
 * The pointer moved during a drag, producing a new snapped value from the
 *  cursor position within the track.
 */
const MovedDragPointer: CallableTaggedStruct<"MovedDragPointer", {
  value: Number
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L114)

```
/** Union of all out-messages the slider component can emit to its parent. */
const OutMessage: Union<readonly [
  CallableTaggedStruct<"ChangedValue", {
    value: Number
  }>
]>
```

### PressedKeyboardNavigation

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L66)

```
/**
 * The user pressed a keyboard navigation key on the focused thumb. The view
 *  supplies `value`, the current value, to compute the next one from.
 */
const PressedKeyboardNavigation: CallableTaggedStruct<"PressedKeyboardNavigation", {
  direction: Literals<readonly ["StepDecrement", "StepIncrement", "PageDecrement", "PageIncrement", "Min", "Max"]>
  value: Number
}>
```

### PressedPointer

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L53)

```
/**
 * The user pressed the track. Starts a drag and snaps the value to the
 *  cursor position. Ignored while already dragging, which absorbs the bubble
 *  from a thumb press so the value is not shifted. `originValue` is the current
 *  value the drag restores to on Escape.
 */
const PressedPointer: CallableTaggedStruct<"PressedPointer", {
  originValue: Number
  value: Number
}>
```

### PressedThumb

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L48)

```
/**
 * The user pressed the thumb. Starts a drag without changing the value. The
 *  view supplies `originValue`, the current value, so Escape can restore it.
 */
const PressedThumb: CallableTaggedStruct<"PressedThumb", {
  originValue: Number
}>
```

### ReleasedDragPointer

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L61)

```
/** The pointer was released during a drag. Commits the current value. */
const ReleasedDragPointer: CallableTaggedStruct<"ReleasedDragPointer", {}>
```

### reflectRange

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L330)

```
/**
 * Reflects an externally-driven range onto the slider. Use this when min/max
 *  derive from external state (e.g. a bounded buffer whose first/last index
 *  shifts over time). The parent owns the value, so conform it to the new range
 *  in the same update with snapAndClamp.
 */
const reflectRange: Reflect<Model, Readonly<{
  max: number
  min: number
}>>
```

### subscriptions

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L479)

```
/** Default drag subscriptions, with the track looked up via `document`. */
const subscriptions: {
  dragEscape: EntryWithoutKeepAlive<Slider.Model, {
    _tag: "CancelledDrag"
  } | {
    _tag: "PressedThumb"
    originValue: number
  } | {
    _tag: "PressedPointer"
    originValue: number
    value: number
  } | {
    _tag: "MovedDragPointer"
    value: number
  } | {
    _tag: "ReleasedDragPointer"
  } | {
    _tag: "PressedKeyboardNavigation"
    direction: "Max" | "Min" | "StepDecrement" | "StepIncrement" | "PageDecrement" | "PageIncrement"
    value: number
  }, {
    dragActivity: "Idle" | "Active"
  }, never> & SubscriptionBrand
  dragPointer: EntryWithoutKeepAlive<Slider.Model, {
    _tag: "CancelledDrag"
  } | {
    _tag: "PressedThumb"
    originValue: number
  } | {
    _tag: "PressedPointer"
    originValue: number
    value: number
  } | {
    _tag: "MovedDragPointer"
    value: number
  } | {
    _tag: "ReleasedDragPointer"
  } | {
    _tag: "PressedKeyboardNavigation"
    direction: "Max" | "Min" | "StepDecrement" | "StepIncrement" | "PageDecrement" | "PageIncrement"
    value: number
  }, {
    dragActivity: "Idle" | "Active"
    id: string
    max: number
    min: number
  }, never> & SubscriptionBrand
}
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/40ccffe57f29bc329cd37443055b46a8f33a7ce2/packages/ui/src/slider/index.ts#L540)

```
/**
 * Renders an accessible slider by building ARIA attribute groups and
 *  delegating layout to the consumer's `toView` callback. Follows the
 *  WAI-ARIA slider pattern: role="slider" on the thumb, aria-valuemin /
 *  aria-valuemax / aria-valuenow, keyboard navigation by step / page / home /
 *  end. Pointer drag is handled by the component's drag subscriptions.
 */
const view: SubmodelView<Slider.Model, {
  _tag: "CancelledDrag"
} | {
  _tag: "PressedThumb"
  originValue: number
} | {
  _tag: "PressedPointer"
  originValue: number
  value: number
} | {
  _tag: "MovedDragPointer"
  value: number
} | {
  _tag: "ReleasedDragPointer"
} | {
  _tag: "PressedKeyboardNavigation"
  direction: "Max" | "Min" | "StepDecrement" | "StepIncrement" | "PageDecrement" | "PageIncrement"
  value: number
}, Readonly<{
  ariaLabel: string
  ariaLabelledBy: string
  formatValue: (value: number) => string
  getTrackRoot: () => Document | ShadowRoot
  isDisabled: boolean
  name: string
  toView: (attributes: SliderAttributes) => Html
  value: number
}>>
```
