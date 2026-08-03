---
url: https://foldkit.dev/ui/virtual-list
title: "Virtual List"
description: "Virtualization primitive for large lists. Only items inside the viewport plus an overscan buffer are mounted; spacers above and below keep the scrollbar physically correct."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

# VirtualList

## Overview

A virtualization component for large lists. Only items inside the viewport plus an overscan buffer are mounted. Spacer divs above and below the visible slice keep the scrollbar physically correct. The demo below manages ten thousand items; only the rows currently visible exist in the DOM.

See it in an app

Check out how VirtualList is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/virtualList.ts).

## Example

Items live in your Model, not the component, and pass through `ViewConfig.items` on each render. The parent owns the data and can swap, filter, sort, or paginate freely without sending Messages to the list. Each item must be keyed via `itemToKey` so the VDOM matches rows by data identity, not by position, when the visible slice shifts.

### Basic

Every row uses the same height, configured at init through `rowHeightPx`. The component divides scroll math by that constant. Prefer this path when row heights are stable.

10,000 activity events

- 
- S

  Sarah Chen

  merged

  PR #1

  1m ago
- M

  Marcus Davies

  opened

  issue #14

  2h ago
- P

  Priya Patel

  commented on

  PR #27

  5h ago
- A

  Alex Kim

  approved

  PR #40

  7h ago
- J

  Jordan Lee

  closed

  issue #53

  9h ago
- S

  Sam Rivera

  reopened

  issue #66

  12h ago
- B

  Ben Carter

  requested review on

  PR #79

  14h ago
- M

  Mira Patel

  pushed to

  fix/dialog-focus

  16h ago
- L

  Lucy Hong

  merged

  PR #105

  18h ago
- C

  Casey Park

  opened

  issue #118

  21h ago
- R

  Robin Adams

  commented on

  PR #131

  23h ago
- T

  Tomás Reyes

  approved

  PR #144

  1d ago
- 

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, view, and subscription definitions.
import { Effect, Schema as S } from 'effect'
import { Command, Subscription } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { VirtualList } from '@foldkit/ui'

// Add a field to your Model for the VirtualList Submodel. The list items
// stay in your domain Model (your own `activities`, `messages`, `rows`,
// whatever you call them); only scroll and measurement state live here:
const Model = S.Struct({
  activityList: VirtualList.Model,
  // ...your other fields, including the items array you want to render
})

// In your init function, give the list a unique id and a row height in
// pixels. All rows share this height:
const init = () => [
  {
    activityList: VirtualList.init({
      id: 'activity-list',
      rowHeightPx: 56,
    }),
    // ...your other fields
  },
  [],
]

// Embed the VirtualList Message in your parent Message:
const GotActivityListMessage = m('GotActivityListMessage', {
  message: VirtualList.Message,
})

// Inside your update function's M.tagsExhaustive({...}), delegate to
// VirtualList.update:
GotActivityListMessage: ({ message }) => {
  const [nextList, commands] = VirtualList.update(model.activityList, message)

  return [
    evo(model, { activityList: () => nextList }),
    Command.mapMessages(commands, message =>
      GotActivityListMessage({ message }),
    ),
  ]
}

// Wire the VirtualList container subscription into your app's
// subscriptions. This powers scroll tracking and container resize
// observation:
const activityListSubscriptions = Subscription.lift({
  activityListEvents: VirtualList.subscriptions.containerEvents,
})<Model, Message>({
  toChildModel: model => model.activityList,
  toParentMessage: message => GotActivityListMessage({ message }),
})

const subscriptions = Subscription.aggregate<Model, Message>()(
  activityListSubscriptions,
  // ...your other subscription records
)

// Inside your view, render the list. Pass `items` from your Model, key
// each row by a stable identifier (the data id, not its array position),
// and give the container a fixed height. Note `h-96` below: without a
// fixed height the container grows to fit its children and never scrolls.
// The component sets only `overflow: auto` inline; everything else is
// yours:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'activity-list',
    model: model.activityList,
    view: VirtualList.view<Activity>(),
    viewInputs: {
      items: model.activities,
      itemToKey: activity => String(activity.id),
      itemToView: activity =>
        h.div(
          [h.Class('grid grid-cols-[2rem_1fr_5rem] items-center gap-3 px-4')],
          [
            h.div(
              [
                h.Class(
                  'flex h-7 w-7 items-center justify-center rounded-full',
                ),
              ],
              [activity.initial],
            ),
            h.span([h.Class('truncate text-sm')], [activity.label]),
            h.span(
              [h.Class('text-right text-xs text-gray-500 tabular-nums')],
              [activity.timeAgo],
            ),
          ],
        ),
      containerClassName:
        'h-96 w-full rounded-lg bg-white ring-1 ring-gray-200',
    },
    toParentMessage: message => GotActivityListMessage({ message }),
  })

// Programmatic scrolling. Returns [Model, Commands] in the same shape as
// update. Stale completions are version-cancelled, so rapid successive
// calls do not fight each other:
const [nextList, commands] = VirtualList.scrollToIndex(model.activityList, 500)
```

### Variable row heights

Pass an `itemToRowHeightPx` callback on `ViewConfig` and rows take the height the callback returns for each item. The component walks the items at render time to compute cumulative offsets for the visible slice and the spacers. Use this for tables with wrapping cells, taller detail rows, or any list where heights differ.

Programmatic scrolling for variable-height lists uses `scrollToIndexVariable`, which walks the heights to compute the target `scrollTop`. Pass the same `items` and `itemToRowHeightPx` you pass to `view` so the math agrees.

Mixed-height rows: every fourth row is taller and shows a summary

- 
- S

  Sarah Chen

  merged

  PR #1

  CI passing across all browsers

  Resolved the flake in the snapshot suite and confirmed the migration step runs idempotently against staging.

  ci/run-4892

  1m ago
- M

  Marcus Davies

  opened

  issue #14

  2h ago
- P

  Priya Patel

  commented on

  PR #27

  5h ago
- A

  Alex Kim

  approved

  PR #40

  7h ago
- J

  Jordan Lee

  closed

  issue #53

  Failure trace attached

  Captured the steps to reproduce, attached the failing trace, and tagged the owning team for triage.

  traces/failure-7c2e

  9h ago
- S

  Sam Rivera

  reopened

  issue #66

  12h ago
- B

  Ben Carter

  requested review on

  PR #79

  14h ago
- M

  Mira Patel

  pushed to

  fix/dialog-focus

  16h ago
- L

  Lucy Hong

  merged

  PR #105

  Tracking upstream change

  Linked the upstream regression and added reproduction context so the next reviewer has everything in one place.

  tracker/issue-218

  18h ago
- C

  Casey Park

  opened

  issue #118

  21h ago
- 

```
// Pseudocode walkthrough for variable-height rows. Builds on the basic
// example: same Model, init, Message, update, subscription wiring. The
// difference is in the view and in how `scrollToIndex` is called. Fit the
// excerpts into your own definitions.
import type { HtmlBuilder } from 'foldkit/html'

import { VirtualList } from '@foldkit/ui'

// Model and init are unchanged from the basic example. Pass any
// `rowHeightPx` to `init`; it remains the uniform default for the
// `scrollToIndex` initial-apply path on the first measurement, and the
// fallback for any item the variable callback doesn't cover:
const init = () => [
  {
    activityList: VirtualList.init({
      id: 'activity-list',
      rowHeightPx: 56,
    }),
    // ...your other fields
  },
  [],
]

// Provide an `itemToRowHeightPx` callback on `view`. Each row wrapper is
// sized to the height the callback returns for that item. Slice and spacer
// math walk the items via a prefix-sum to find the visible window. Tests
// with 10k items at 60Hz scroll well within budget; larger lists may need
// a prefix-sum cache if you can profile the regression:
const view = (h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'activity-list',
    model: model.activityList,
    view: VirtualList.view<Activity>(),
    viewInputs: {
      items: model.activities,
      itemToKey: activity => String(activity.id),
      itemToRowHeightPx: (activity, index) => (activity.hasSummary ? 104 : 56),
      itemToView: activity =>
        activity.hasSummary
          ? h.div(
              [
                h.Class(
                  'grid grid-cols-[2rem_1fr_5rem] items-start gap-3 px-4 py-3',
                ),
              ],
              [
                h.div([h.Class('h-7 w-7 rounded-full')], [activity.initial]),
                h.div(
                  [],
                  [
                    h.span([], [activity.label]),
                    h.div(
                      [h.Class('mt-1 text-xs text-gray-500')],
                      [activity.summary],
                    ),
                  ],
                ),
                h.span([h.Class('text-right text-xs')], [activity.timeAgo]),
              ],
            )
          : h.div(
              [
                h.Class(
                  'grid grid-cols-[2rem_1fr_5rem] items-center gap-3 px-4',
                ),
              ],
              [
                h.div([h.Class('h-7 w-7 rounded-full')], [activity.initial]),
                h.span([], [activity.label]),
                h.span([h.Class('text-right text-xs')], [activity.timeAgo]),
              ],
            ),
      containerClassName:
        'h-96 w-full rounded-lg bg-white ring-1 ring-gray-200',
    },
    toParentMessage: message => GotActivityListMessage({ message }),
  })

// Programmatic scrolling for variable-height lists uses
// `scrollToIndexVariable`, which walks the heights to compute the target
// `scrollTop`. Pass the same `items` and `itemToRowHeightPx` you pass to
// `view` so the math agrees:
const itemToRowHeightPx = (activity, index) => (activity.hasSummary ? 104 : 56)

const [nextList, commands] = VirtualList.scrollToIndexVariable(
  model.activityList,
  model.activities,
  itemToRowHeightPx,
  500,
)

// `scrollToIndex` (uniform) and `scrollToIndexVariable` (variable) are
// independent: pick the one that matches how `view` is rendering. Mixing
// them produces inconsistent scroll targets.
```

## Subscriptions

VirtualList exposes a single subscription, `containerEvents`, that listens for `scroll` events on the container and observes its size with `ResizeObserver`. Wire it into your app's subscriptions alongside the rest of the framework subscriptions.

## Styling

The container needs a constrained height for virtualization to work. Without it, the container grows to fit children and never scrolls. Pass `className` or `attributes` on `ViewConfig` to apply the height through your styling system. The component sets only `overflow: auto` inline; the rest is yours.

VirtualList exposes two data attributes for styling and test selectors: `data-virtual-list-id` on the scrollable container and `data-virtual-list-item-index` on each rendered row.

Attribute

Condition

`data-virtual-list-id`

Present on the scrollable container. Carries the id from InitConfig so subscriptions and tests can find the right element.

`data-virtual-list-item-index`

Present on each rendered row wrapper. Carries the data index of the item being rendered (0-based) so tests and consumer styling can address a specific row.

## Accessibility

The container is rendered as `<ul>` and each row as `<li>`. The top and bottom spacer `<li>` elements carry `role="presentation"` so they do not contribute to the list. Each rendered row carries `aria-setsize` (total item count) and `aria-posinset` (1-based logical position), so screen readers announce "row 5,234 of 10,000" rather than the much smaller count of mounted rows. No consumer wiring required.

## API Reference

### InitConfig

Configuration object passed to `VirtualList.init()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the virtual list instance. Applied to the scrollable container and used by the subscription to attach scroll and resize listeners.

`rowHeightPx`

`number`

—

Height in pixels of every row. All rows share this height; the value drives spacer math, slice math, and the inline height on row wrappers.

`initialScrollTop`

`number`

`0`

Initial scroll position in pixels. When non-zero, the first MeasuredContainer message issues an apply-scroll Command so the DOM and model agree from the first frame.

### ViewConfig

Configuration object passed to `VirtualList.view()`.

Name

Type

Default

Description

`model`

`VirtualList.Model`

—

The virtual list state from your parent Model.

`items`

`ReadonlyArray<Item>`

—

The full item array. Items live in your Model, not the component's; pass them fresh on each render. Swap, filter, sort, or paginate freely without sending Messages to the list.

`itemToKey`

`(item: Item, index: number) => string`

—

Returns a stable identifier for an item. Used to key rendered rows so the VDOM matches by data identity rather than by position when the visible slice shifts.

`itemToView`

`(item: Item, index: number) => Html`

—

Renders one row's contents. The framework wraps your output in a row-height grid container; use flex or grid with align-items: center inside to vertically center your content.

`itemToRowHeightPx`

`(item: Item, index: number) => number`

—

Optional. When provided, the list renders with variable-height rows: each row wrapper takes the height returned for its item, and slice and spacer math walks the items to compute cumulative offsets. When absent, every row uses model.rowHeightPx. Prefer the uniform path when row heights are stable.

`overscan`

`number`

`5`

Number of rows mounted above and below the visible viewport. Higher values smooth out fast scroll at the cost of mounting more DOM. react-window uses 1 and react-virtualized uses 3; pick a value that suits the row mount cost.

`rowElement`

`TagName`

`'li'`

HTML tag for each row wrapper. Defaults to li (since the container is rendered as ul). Override only when you also wrap the list in something whose children aren't expected to be li.

`containerClassName`

`string | undefined`

—

CSS class applied to the scrollable container. The container needs a constrained height (e.g. h-96) for virtualization to work.

`containerAttributes`

`ReadonlyArray<ChildAttribute> | undefined`

—

Additional attributes spread onto the scrollable container. Pass extra Style({...}) entries for CSS like overscroll-behavior or scroll-margin, data attributes, or any other ChildAttribute.
