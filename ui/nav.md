---
url: https://foldkit.dev/ui/nav
title: "Nav"
description: "URL-driven navigation between routes with aria-current page semantics."
access_date: 2026-08-03T18:55:49.002Z
current_date: 2026-08-03T18:55:49.002Z
---

# Nav

## Overview

URL-driven navigation between sections, where each section is a separate route with its own URL, deep-linking, and browser-history behavior. Nav renders a navigation landmark whose items are links, marking the current destination with `aria-current="page"`.

Switching content without changing the URL?

Reach for [Tabs](https://foldkit.dev/ui/tabs) instead. Nav is for URL-driven navigation; Tabs is for swapping panels inside a single page.

See it in an app

The [UI showcase](https://github.com/foldkit/foldkit/tree/main/examples/ui-showcase/src) builds its own sidebar with Nav, so every route marks the active link with aria-current.

## Examples

### Basic

Nav is stateless. There is no `Nav.Model` and no `Nav.update`: the current item is derived from the URL via `isItemCurrent`. Pass the item values, a `toHref` that maps each to its route, and a `toView` callback that receives one `ItemInfo<Value>` per item.

[Home](?section=home)

[Search](?section=search)

[Library](?section=library)

[Profile](?section=profile)

myapp.dev/?section=home

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, Message, update,
// and view definitions.
import { Match as M } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'

import { Nav } from '@foldkit/ui'

// Nav is stateless: the current destination comes from the URL, so there is
// no Nav.Model to store and no Nav.update to delegate to. Your app already
// holds the active route in its Model:
const Model = S.Struct({
  route: AppRoute,
  // ...your other fields
})

// The nav items are the sections you navigate between. Use the route tags as
// the item values so toView can switch on them without casting:
type Section = 'Home' | 'Search' | 'Library' | 'Profile'

const sections: ReadonlyArray<Section> = [
  'Home',
  'Search',
  'Library',
  'Profile',
]

// Map each section to its URL with your routers, and decide which section is
// current from the active route. A section can own a whole family of routes,
// so this is a predicate rather than an equality check:
const sectionToHref = (section: Section): string =>
  M.value(section).pipe(
    M.when('Home', () => homeRouter()),
    M.when('Search', () => searchRouter()),
    M.when('Library', () => libraryRouter()),
    M.when('Profile', () => profileRouter()),
    M.exhaustive,
  )

const isSectionCurrent =
  (route: AppRoute) =>
  (section: Section): boolean =>
    sectionToRouteTag(section) === route._tag

// Inside your view function, render the nav. Spread the nav bundle onto an
// h.nav landmark and each item.link bundle onto an h.a. The current item
// carries aria-current="page" and a data-current attribute for styling.
// Browser-native Tab and Enter handle keyboard navigation; Foldkit's runtime
// turns the link clicks into route changes:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  Nav.view<Section>({
    items: sections,
    ariaLabel: 'Primary',
    toHref: sectionToHref,
    isItemCurrent: isSectionCurrent(model.route),
    toView: ({ nav, items }) =>
      h.nav(
        [...nav, h.Class('flex gap-2')],
        items.map(item =>
          h.a(
            [
              ...item.link,
              h.Class(
                'px-4 py-2 rounded-lg text-gray-500 data-[current]:bg-gray-100 data-[current]:text-gray-900',
              ),
            ],
            [h.span([], [item.value])],
          ),
        ),
      ),
  })
```

## Styling

Nav is headless. The `toView` callback owns all markup, spreading the attribute bundles onto your own elements. Style the current item with the `data-current` attribute, for example `data-[current]:bg-gray-100`.

Attribute

Condition

`data-current`

Present on the anchor of the current item.

## Keyboard Interaction

Nav items are plain links, so keyboard support is the browser’s native link handling. There is no roving tabindex: every link is in the tab order, which is the correct pattern for a navigation landmark (roving tabindex belongs to composite widgets like [Tabs](https://foldkit.dev/ui/tabs)).

Key

Description

`Tab`

Move focus to the next link. Native browser behavior.

`Shift + Tab`

Move focus to the previous link. Native browser behavior.

`Enter`

Follow the focused link. Native browser behavior.

## Accessibility

The `nav` bundle carries `aria-label`, and the wrapping `<nav>` element supplies the navigation landmark role. The current item’s anchor receives `aria-current="page"`, the value the ARIA Authoring Practices Guide recommends for marking the current page within a set of navigation links.

## Nav vs Tabs

Both render a set of selectable items, but the semantics differ. [Tabs](https://foldkit.dev/ui/tabs) applies `role="tablist"`, `role="tab"`, and `role="tabpanel"`: it switches content within one page, owns the active index in its Model, and uses roving tabindex with arrow keys. Nav applies the navigation landmark with `aria-current="page"`: it derives the current item from the URL, holds no state of its own, and leaves each link in the tab order. If each item has its own URL and the browser back button should move between items, use Nav.

## API Reference

### ViewInputs

Configuration object passed to `Nav.view()`.

Name

Type

Default

Description

`items`

`ReadonlyArray<Value>`

—

The nav item values, in display order. When typed via

`Nav.view<MyUnion>()`

,

`Value`

is your union (typically a route tag) and each

`ItemInfo.value`

is typed as

`MyUnion`

.

`ariaLabel`

`string`

—

Accessible label for the navigation landmark. Distinguishes this nav from other landmarks on the page (e.g. “Primary”, “Footer”).

`toHref`

`(value: Value, index: number) => string`

—

Maps each item to its URL. Build these with your routers so the links deep-link and the runtime can drive history.

`isItemCurrent`

`(value: Value, index: number) => boolean`

—

Decides which item is the current page from the active route. A predicate (not equality) so one section can own a whole family of routes.

`toView`

`(render: RenderInfo<Value>) => Html`

—

Callback that receives the

`nav`

attribute bundle and one

`ItemInfo<Value>`

per item. Returns the composed layout.

### RenderInfo

Payload delivered to the `toView` callback each render.

Name

Type

Default

Description

`nav`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the navigation landmark. Includes

`aria-label`

. Spread it onto an

`h.nav`

, which already supplies the navigation role.

`items`

`ReadonlyArray<ItemInfo<Value>>`

—

One entry per

`items`

, in the same order. See ItemInfo below.

### ItemInfo

Each entry in `RenderInfo.items`. Carries the value, derived current state, and the anchor attribute bundle.

Name

Type

Default

Description

`value`

`Value`

—

The item value. Typed as your

`Value`

union when declared via

`Nav.view<Value>()`

.

`index`

`number`

—

Position in the

`items`

array.

`isCurrent`

`boolean`

—

Whether this item is the current page per

`isItemCurrent`

.

`link`

`ReadonlyArray<ChildAttribute>`

—

Spread onto the anchor element. Includes

`href`

, plus

`aria-current="page"`

and

`data-current`

on the current item.
