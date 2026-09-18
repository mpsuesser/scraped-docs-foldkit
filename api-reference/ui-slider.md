---
url: https://foldkit.dev/api-reference/ui-slider
title: "Ui/Slider"
description: "API documentation for the Ui/Slider module."
access_date: 2026-09-18T16:33:34.359Z
current_date: 2026-09-18T16:33:34.359Z
---

# Ui/Slider

## Functions

### fractionOfValue

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L144)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L99)

```
/**
 * Creates an initial slider model from a config. The value lives in the
 *  parent Model; initialize it there and snap it with snapAndClamp.
 */
(config: InitConfig): Slider.Model
```

### snapAndClamp

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L132)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L346)

### update

function

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L210)

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L90)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L476)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L486)

```
/** Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field. */
type ViewInputs = Readonly<{
  ariaLabel: string
  ariaLabelledBy: string
  formatValue: (value: number) => string
  getTrackRoot: () => Document | ShadowRoot
  isDisabled: boolean
  isReadOnly: boolean
  name: string
  toView: (attributes: SliderAttributes) => Html
  value: number
}>
```

## Constants

### Message

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L47)

```
/** Union of all messages the slider component can produce. */
const Message: MessageUnion<{
  CancelledDrag: {}
  MovedDragPointer: {
    value: Number
  }
  PressedKeyboardNavigation: {
    direction: Literals<readonly ["StepDecrement", "StepIncrement", "PageDecrement", "PageIncrement", "Min", "Max"]>
    value: Number
  }
  PressedPointer: {
    originValue: Number
    value: Number
  }
  PressedThumb: {
    originValue: Number
  }
  ReleasedDragPointer: {}
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L34)

```
/**
 * Schema for the slider component's private interaction state. The current
 *  value is owned by the parent and passed in via `ViewInputs.value`, so it is
 *  not stored here. `min`/`max`/`step` are configuration the drag subscription
 *  reads to map pointer positions into values. `dragState` tracks the active
 *  drag phase and captures the pre-drag value so Escape can restore it.
 */
const Model: Struct<{
  dragState: TaggedUnion<{
    Dragging: {
      originValue: Number
    }
    Idle: {}
  }>
  id: String
  max: Number
  min: Number
  step: Number
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L82)

```
/** Union of all out-messages the slider component can emit to its parent. */
const OutMessage: MessageUnion<{
  ChangedValue: {
    value: Number
  }
}>
```

### reflectRange

const

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L294)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L443)

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

[source](https://github.com/foldkit/foldkit/blob/71e9b47d6caba59367961fe61f55a882af1faa04/packages/ui/src/slider/index.ts#L522)

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
  isReadOnly: boolean
  name: string
  toView: (attributes: SliderAttributes) => Html
  value: number
}>>
```
