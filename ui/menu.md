---
url: https://foldkit.dev/ui/menu
title: "Menu"
description: "Accessible dropdown menu with keyboard navigation."
access_date: 2026-08-03T18:23:29.850Z
current_date: 2026-08-03T18:23:29.850Z
---

# Menu

## Overview

A dropdown menu for actions, like a macOS context menu. Menu is fire-and-forget: each activation is an action, not a choice that persists (use Listbox for selection, where the parent owns the selected value). It supports typeahead search, drag-to-select, keyboard navigation, grouped items, and anchor positioning.

For programmatic control in update functions, use the factory’s `open(model)`, `close(model)`, and `selectItem(model, item, index)` methods. Each returns the same `[Model, Commands, Option<OutMessage>]` tuple as `update`.

What `Menu.create<Item>()` returns is typed [`Menu.Bundle<Item>`](https://foldkit.dev/ui/selection-submodels#bundle-type), for the cases where a created bundle has to be named rather than called directly.

See it in an app

Check out how Menu is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/menu.ts).

## Examples

### Basic

Pair `view` and `update` behind `Menu.create<Item>()` at module scope. The factory threads your item union through both, so `Selected({ value, index })` carries the picked value directly. Menu closes automatically after selection.

Row actions

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Effect, Match as M, Option } from 'effect'
import { Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Menu } from '@foldkit/ui'

// Add a field to your Model for the Menu Submodel:
const Model = S.Struct({
  menu: Menu.Model,
  // ...your other fields
})

// In your init function, initialize the Menu Submodel with a unique id:
const init = () => [
  {
    menu: Menu.init({ id: 'actions' }),
    // ...your other fields
  },
  [],
]

// Embed the Menu Message in your parent Message:
const GotMenuMessage = m('GotMenuMessage', {
  message: Menu.Message,
})

type Action = 'Edit' | 'Duplicate' | 'Archive' | 'Delete'
const actions: ReadonlyArray<Action> = [
  'Edit',
  'Duplicate',
  'Archive',
  'Delete',
]

// Pair view and update behind a single Item-typed factory at module scope:
const ActionMenu = Menu.create<Action>()

// Inside your update function's M.tagsExhaustive({...}), delegate to
// ActionMenu.update. The OutMessage's `Selected` carries the picked item
// directly (typed as `Action`):
GotMenuMessage: ({ message }) => {
  const [nextMenu, commands, maybeOutMessage] = ActionMenu.update(
    model.menu,
    message,
  )
  const mappedCommands = Command.mapMessages(commands, message =>
    GotMenuMessage({ message }),
  )

  return Option.match(maybeOutMessage, {
    onNone: () => [evo(model, { menu: () => nextMenu }), mappedCommands],
    onSome: M.type<Menu.OutMessage<Action>>().pipe(
      M.tagsExhaustive({
        Selected: ({ value }) => {
          // The child has emitted `Selected`. The body commits the
          // child's next state as usual. In this arm the parent can
          // also update its own state or dispatch its own Commands,
          // for example transition a page, mutate domain state, or
          // trigger a downstream Command.
          return [evo(model, { menu: () => nextMenu }), mappedCommands]
        },
      }),
    ),
  })
}

// Inside your view function, render the menu via the factory's view. The
// `buttonContent` below names the trigger. When the trigger is icon-only,
// give it a name with `ariaLabel`, or point `ariaLabelledBy` at a visible
// label element (target the trigger id with `Menu.buttonId('actions')` for a
// native `<label for>`). Either attribute is only emitted when provided, so
// the trigger never carries a dangling `aria-labelledby`.
const view = (h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'menu',
    model: model.menu,
    view: ActionMenu.view,
    viewInputs: {
      // ariaLabel: 'Options',
      items: actions,
      buttonContent: h.span([], ['Options']),
      buttonClassName: 'rounded-lg border px-3 py-2 cursor-pointer',
      itemsClassName: 'rounded-lg border shadow-lg',
      itemToConfig: (action, { isActive }) => ({
        className: isActive ? 'bg-blue-100' : '',
        content: h.div([h.Class('px-3 py-2')], [action]),
      }),
      isItemDisabled: action => action === 'Archive',
      backdropClassName: 'fixed inset-0',
      anchor: { placement: 'bottom-start', gap: 4, padding: 8 },
    },
    toParentMessage: message => GotMenuMessage({ message }),
  })
```

### Animated

Pass `isAnimated: true` at init for animation coordination.

Row actions

```
// Pseudocode walkthrough using the same Model, Messages, and update as
// the basic menu; only init and view change. Each labeled block below is
// an excerpt.
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'

import { Menu } from '@foldkit/ui'

// Only init and view differ from the basic menu: init adds isAnimated, the
// view uses data-[closed] selectors for enter/leave transitions.

// In your init function, set isAnimated: true to coordinate CSS transitions:
const init = () => [
  {
    menu: Menu.init({ id: 'actions', isAnimated: true }),
    // ...your other fields
  },
  [],
]

// Embed the Menu Message in your parent Message:
const GotMenuMessage = m('GotMenuMessage', {
  message: Menu.Message,
})

// Pair view and update behind a single Item-typed factory at module scope:
const ActionMenu = Menu.create<Action>()

// Inside your view function, use data-[closed] for enter/leave transitions:
const view = (h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'menu',
    model: model.menu,
    view: ActionMenu.view,
    viewInputs: {
      items: actions,
      buttonContent: h.span([], ['Options']),
      buttonClassName: 'rounded-lg border px-3 py-2 cursor-pointer',
      itemsClassName:
        'rounded-lg border shadow-lg transition duration-150 ease-out data-[closed]:opacity-0 data-[closed]:scale-95',
      itemToConfig: (action, { isActive }) => ({
        className: isActive ? 'bg-blue-100' : '',
        content: h.div([h.Class('px-3 py-2')], [action]),
      }),
      backdropClassName: 'fixed inset-0',
      anchor: { placement: 'bottom-start', gap: 4, padding: 8 },
    },
    toParentMessage: message => GotMenuMessage({ message }),
  })
```

## Styling

Menu is headless. The `itemToConfig` callback controls all item markup. Group items with `itemGroupKey` and `groupToHeading`.

The items panel is portaled to the document body and positioned relative to the trigger button with Floating UI. Ancestor stacking contexts and overflow clipping no longer apply, so a clipped container or a sibling overlay wrapper cannot hide an open menu. The panel still stacks at the document level: give it a z-index above elevated content like sticky headers or toasts, as the demos on this page do with `z-10`. Pass `anchor: { portal: false }` to keep the panel inside the wrapper instead.

When `isAnimated` is true, enter/leave animations flow through the [Animation](https://foldkit.dev/ui/animation) module. Style with CSS transitions or CSS keyframe animations. Animation advances once every animation on the element has settled.

Attribute

Condition

`data-open`

Present on the button when the menu is open.

`data-active`

Present on the highlighted menu item.

`data-disabled`

Present on disabled menu items.

`data-closed`

Present during close animation.

`data-placement`

Present on the items panel, set to the side it currently sits on: top, right, bottom, or left. Fixed to the first resolved side when isPlacementLocked is true.

## Keyboard Interaction

Menu uses `aria-activedescendant`. Focus stays on the items container while arrow keys update the highlighted item. Typeahead search accumulates characters for 350ms.

Key

Description

`Enter / Space`

Opens the menu (from button) or selects the active item.

`Arrow Down`

Opens with first item active (from button) or moves to next item.

`Arrow Up`

Opens with last item active (from button) or moves to previous item.

`Home / End`

Moves to the first / last item.

`Escape`

Closes the menu and returns focus to the button.

`Type a character`

Typeahead search: jumps to the matching item.

## Accessibility

The button receives `aria-haspopup="menu"` and `aria-expanded`. The items container receives `role="menu"` with `aria-activedescendant`. Each item receives `role="menuitem"`.

Give the trigger an accessible name. For a visible label, wire a native `<label for>` that targets the trigger id with `Menu.buttonId(id)` rather than hardcoding the `-button` convention. The `for` association makes the trigger properly labeled: assistive technology announces it by the visible label text, and clicking the label opens the menu. That is why it is the recommended pattern.

Two ViewConfig fields cover the cases a `<label for>` does not. Pass `ariaLabel` for an icon-only trigger with no visible label, or `ariaLabelledBy` when the element that names the trigger is not a `<label>` you can point `for` at.

## API Reference

### InitConfig

Configuration object passed to `Menu.init()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the menu instance.

`isAnimated`

`boolean`

`false`

Enables animation coordination.

`isModal`

`boolean`

`false`

Locks page scroll and marks other elements inert when open.

### ViewConfig

Configuration object passed to `Menu.view()`.

Name

Type

Default

Description

`model`

`Menu.Model`

—

The menu state from your parent Model.

`toParentMessage`

`(childMessage: Menu.Message) => ParentMessage`

—

Wraps Menu Messages in your parent Message type for Submodel delegation.

`items`

`ReadonlyArray<Item>`

—

The list of menu items.

`itemToConfig`

`(item, context) => ItemConfig`

—

Maps each item to its className and content. The context provides isActive and isDisabled.

`buttonContent`

`Html`

—

Content rendered inside the trigger button.

`isItemDisabled`

`((item, index) => boolean) | undefined`

—

Disables individual menu items.

`itemToSearchText`

`((item, index) => string) | undefined`

—

Optional override for the string typeahead matches against. Defaults to the item itself; override when items carry searchable text distinct from their display content.

`isButtonDisabled`

`boolean | undefined`

—

Disables the trigger button entirely. The menu cannot be opened while true.

`itemGroupKey`

`((item, index) => string) | undefined`

—

Groups contiguous items by key.

`groupToHeading`

`((groupKey) => GroupHeading | undefined) | undefined`

—

Renders a heading for each group.

`anchor`

`AnchorConfig | undefined`

—

Floating positioning config: placement, gap, offset, padding, isPlacementLocked, and portal. The items panel is always anchored to the button; when omitted, the panel uses bottom-start placement. Portaled to the document body by default; pass portal: false to keep the panel inside the wrapper.

`buttonClassName`

`string | undefined`

—

CSS class for the trigger button.

`buttonAttributes`

`ReadonlyArray<ChildAttribute> | undefined`

—

Extra attributes spread onto the trigger button alongside its built-in click/keyboard handlers and aria-* attributes.

`itemsClassName`

`string | undefined`

—

CSS class for the items container (the panel root).

`itemsAttributes`

`ReadonlyArray<ChildAttribute> | undefined`

—

Extra attributes spread onto the items container.

`itemsScrollClassName`

`string | undefined`

—

CSS class for the inner scrollable wrapper around the item list. Useful for setting max-height/overflow without restyling the panel root.

`itemsScrollAttributes`

`ReadonlyArray<ChildAttribute> | undefined`

—

Extra attributes spread onto the inner scrollable wrapper.

`backdropClassName`

`string | undefined`

—

CSS class for the backdrop.

`backdropAttributes`

`ReadonlyArray<ChildAttribute> | undefined`

—

Extra attributes spread onto the backdrop element.

`groupClassName`

`string | undefined`

—

CSS class applied to each group wrapper.

`groupAttributes`

`ReadonlyArray<ChildAttribute> | undefined`

—

Extra attributes spread onto each group wrapper.

`separatorClassName`

`string | undefined`

—

CSS class applied to the separator rendered between adjacent groups.

`separatorAttributes`

`ReadonlyArray<ChildAttribute> | undefined`

—

Extra attributes spread onto each group separator.

`className`

`string | undefined`

—

CSS class applied to the outer Menu root element.

`attributes`

`ReadonlyArray<ChildAttribute> | undefined`

—

Extra attributes spread onto the outer Menu root element.

`ariaLabel`

`string`

—

Accessible name for the trigger button. Use for an icon-only trigger with no visible label. Applied as aria-label, and takes precedence over ariaLabelledBy.

`ariaLabelledBy`

`string`

—

Id of an external element that labels the trigger button, applied as aria-labelledby. Pair with a visible label element.

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Pattern-match on the OutMessage in your update handler.

Name

Type

Default

Description

`Selected`

`{ value: Item; index: number }`

—

Emitted when a menu item is selected. Carries both the value (typed as your

`Item`

union via

`Menu.create<Item>()`

) and its index into the items array supplied at view time. Menu closes itself on selection; the parent does not need to dispatch Menu.close. Pattern-match the third tuple element of Menu.update in your GotMenuMessage handler to dispatch the corresponding domain action.
