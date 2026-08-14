---
url: https://foldkit.dev/api-reference/ui-nav
title: "Ui/Nav"
description: "API documentation for the Ui/Nav module."
access_date: 2026-08-14T21:53:32.516Z
current_date: 2026-08-14T21:53:32.516Z
---

# Ui/Nav

## Functions

### view

function

[source](https://github.com/foldkit/foldkit/blob/ac3a34fd6f1be16714aba1e4e18e20e8b31bbf16/packages/ui/src/nav/index.ts#L64)

```
/**
 * Renders a navigation landmark whose items are links, marking the current
 *  destination with `aria-current="page"`. Stateless: the current item is
 *  derived from `isItemCurrent`, which the consumer drives from the URL, the
 *  single source of truth for which page is showing.
 * 
 *  Use this for URL-driven navigation where each section is a separate
 *  route with its own URL, deep-linking, and browser-history behavior.
 *  Reach for `Tabs` instead when switching content within a single page;
 *  `Tabs` carries the `tablist`/`tab`/`tabpanel` semantics, which are
 *  wrong for URL-driven navigation.
 * 
 *  Headless: `toView` owns all markup, spreading the `nav` bundle onto an
 *  `h.nav` and each `item.link` bundle onto an `h.a`. Keyboard support is the
 *  browser's native link handling (Tab to move, Enter to follow); no roving
 *  tabindex is imposed, since that is a composite-widget pattern, not a
 *  navigation-landmark one.
 */
<Value extends string = string>(viewInputs: ViewInputs<Value>): Html
```

## Types

### ItemInfo

type

[source](https://github.com/foldkit/foldkit/blob/ac3a34fd6f1be16714aba1e4e18e20e8b31bbf16/packages/ui/src/nav/index.ts#L14)

```
/**
 * Per-item render info passed to the consumer's `toView`. Generic over
 *  `Value extends string`: the item value is typed as the consumer's union
 *  (typically a route tag) so `toView` can switch on it without casting.
 */
type ItemInfo = Readonly<{
  index: number
  isCurrent: boolean
  link: ReadonlyArray<ChildAttribute>
  value: Value
}>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/ac3a34fd6f1be16714aba1e4e18e20e8b31bbf16/packages/ui/src/nav/index.ts#L28)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 *  - `nav`: ARIA attributes for the wrapping navigation landmark. Carries
 *    `aria-label` only; the `<nav>` element already supplies the navigation
 *    role, so spread this onto an `h.nav`.
 *  - `items`: one entry per `viewInputs.items`, in the same order, each with
 *    the anchor's attribute bundle and derived current state.
 */
type RenderInfo = Readonly<{
  items: ReadonlyArray<ItemInfo<Value>>
  nav: ReadonlyArray<ChildAttribute>
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/ac3a34fd6f1be16714aba1e4e18e20e8b31bbf16/packages/ui/src/nav/index.ts#L36)

```
/**
 * View inputs for `Nav.view`. Generic over `Value extends string` so the
 *  consumer's section/route union flows into `toView`, `toHref`, and
 *  `isItemCurrent` without casting.
 */
type ViewInputs = Readonly<{
  ariaLabel: string
  isItemCurrent: (value: Value, index: number) => boolean
  items: ReadonlyArray<Value>
  toHref: (value: Value, index: number) => string
  toView: (render: RenderInfo<Value>) => Html
}>
```
