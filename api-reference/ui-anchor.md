---
url: https://foldkit.dev/api-reference/ui-anchor
title: "Ui/Anchor"
description: "API documentation for the Ui/Anchor module."
access_date: 2026-08-21T00:40:18.371Z
current_date: 2026-08-21T00:40:18.371Z
---

# Ui/Anchor

## Functions

### anchorSetup

function

[source](https://github.com/foldkit/foldkit/blob/619a80e18c76a5d300c7b85921c86db36d1856fc/packages/ui/src/anchor/anchor.ts#L148)

```
/**
 * Positions a floating element relative to its button using Floating UI, then
 *  returns a cleanup function. Designed to be called inside an `OnMount`
 *  action: the consumer wraps the call in `Effect.sync` and stashes the
 *  returned cleanup in the `Mount` result. When `interceptTab` is true
 *  (default), Tab key in portal mode refocuses the button. Set to false for
 *  components like Popover where Tab should navigate naturally within the
 *  panel. When `focusAfterPosition` is true, the element is focused after the
 *  first position computation clears visibility, deferred via
 *  requestAnimationFrame so the element is painted before focus fires.
 *  `focusSelector` optionally targets a descendant (e.g. a calendar grid
 *  inside a popover panel) instead of the panel itself.
 *  The side the element currently sits on is written to `data-placement`, so
 *  CSS can react to it. When `isPlacementLocked` is true, the element keeps the
 *  side that the first positioning picks, `flip` is removed from every later
 *  update, and `data-placement` holds that locked side. Otherwise the attribute
 *  tracks the side each update resolves to, including the ones `flip` moves.
 */
(
  element: Element,
  config: SetupConfig
): () => void
```

### portalToContainingRoot

function

[source](https://github.com/foldkit/foldkit/blob/619a80e18c76a5d300c7b85921c86db36d1856fc/packages/ui/src/anchor/anchor.ts#L97)

```
/**
 * Relocates an element into the shared `foldkit-portal-root` div within its
 *  containing root: the shadow root when mounted inside one, otherwise
 *  `document.body`. Escapes any ancestor stacking context while keeping the
 *  element under that root's scoped styles. Returns a cleanup function that
 *  removes the element from the portal root. Designed to be called from inside
 *  an `OnMount` action: the consumer wraps the call in `Effect.sync` and
 *  stashes the returned cleanup in the `Mount` result.
 */
(element: Element): () => void
```

## Types

### SetupConfig

type

[source](https://github.com/foldkit/foldkit/blob/619a80e18c76a5d300c7b85921c86db36d1856fc/packages/ui/src/anchor/anchor.ts#L124)

```
/**
 * Config for `anchorSetup`.
 * 
 *  - `buttonId`: id of the trigger to position against, resolved through the
 *    element's own root.
 *  - `anchor`: the static positioning options.
 *  - `interceptTab`: returns focus to the trigger on Tab inside a portaled
 *    panel. Defaults to `true`.
 *  - `focusAfterPosition`: focuses the element once the first position
 *    resolves. Defaults to `false`.
 *  - `focusSelector`: focuses this descendant instead of the element itself.
 *    Read only when `focusAfterPosition` is true.
 */
type SetupConfig = Readonly<{
  anchor: AnchorConfig
  buttonId: string
  focusAfterPosition: boolean
  focusSelector: string
  interceptTab: boolean
}>
```

## Constants

### AnchorConfig

const

[source](https://github.com/foldkit/foldkit/blob/619a80e18c76a5d300c7b85921c86db36d1856fc/packages/ui/src/anchor/anchor.ts#L47)

```
/** Static configuration for anchor-based positioning of a floating element relative to a button. */
const AnchorConfig: Struct<{
  gap: optional<Number>
  isPlacementLocked: optional<Boolean>
  offset: optional<Number>
  padding: optional<Union<readonly [
    Number,
    Struct<{
      bottom: optionalKey<Number>
      left: optionalKey<Number>
      right: optionalKey<Number>
      top: optionalKey<Number>
    }>
  ]>>
  placement: optional<Literals<readonly ["top", "right", "bottom", "left", "top-start", "top-end", "right-start", "right-end", "bottom-start", "bottom-end", "left-start", "left-end"]>>
  portal: optional<Boolean>
}>
```

### Padding

const

[source](https://github.com/foldkit/foldkit/blob/619a80e18c76a5d300c7b85921c86db36d1856fc/packages/ui/src/anchor/anchor.ts#L34)

```
/**
 * Schema mirroring `@floating-ui/dom`'s `Padding` type: a uniform number or a
 *  partial per-side object (`top`/`right`/`bottom`/`left`).
 */
const Padding: Union<readonly [
  Number,
  Struct<{
    bottom: optionalKey<Number>
    left: optionalKey<Number>
    right: optionalKey<Number>
    top: optionalKey<Number>
  }>
]>
```

### Placement

const

[source](https://github.com/foldkit/foldkit/blob/619a80e18c76a5d300c7b85921c86db36d1856fc/packages/ui/src/anchor/anchor.ts#L15)

```
/**
 * Schema mirroring `@floating-ui/dom`'s `Placement` literal union: a side
 *  (`top`/`right`/`bottom`/`left`) optionally suffixed with `-start` or `-end`.
 */
const Placement: Literals<readonly ["top", "right", "bottom", "left", "top-start", "top-end", "right-start", "right-end", "bottom-start", "bottom-end", "left-start", "left-end"]>
```
