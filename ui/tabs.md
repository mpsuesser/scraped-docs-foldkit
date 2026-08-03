---
url: https://foldkit.dev/ui/tabs
title: "Tabs"
description: "Accessible tabbed interface with keyboard navigation."
access_date: 2026-08-03T19:45:20.723Z
current_date: 2026-08-03T19:45:20.723Z
---

## Tabs

## Overview

Tab panel navigation with roving tabindex keyboard support, horizontal and vertical orientation, and automatic or manual activation modes. Tabs renders a tab list with buttons and corresponding panels. Only the active panel is visible.

Tabs is a Submodel that keeps its own keyboard-focus state, but the parent owns the active tab. Store the active value in your Model, pass it in as `selectedValue`, and fold the `Selected` OutMessage back into that field in your `GotTabsMessage` handler.

What `Tabs.create<Value>()` returns is typed [`Tabs.Bundle<Value>`](https://foldkit.dev/ui/selection-submodels#bundle-type), for the cases where a created bundle has to be named rather than called directly.

See it in an app

Check out how Tabs is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/tabs.ts).

Each item its own URL?

Tabs switches content within one page and applies the `tablist` / `tab` / `tabpanel` roles. For URL-driven navigation where each section has its own URL, reach for [Nav](https://foldkit.dev/ui/nav), which uses `aria-current="page"` instead.

## Examples

### Horizontal

Declare the tabs component once at module scope with `Tabs.create<Value>()` to lift the tab type through `view` and `update` without casting. Pass the typed `tabs` array and a `toView` callback that receives one `TabInfo<Value>` per tab (with attribute bundles for the tab button and its panel).

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Match as M, Option } from 'effect'
import { Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Tabs } from '@foldkit/ui'

const Framework = S.Literals(['Foldkit', 'React', 'Elm'])
type Framework = typeof Framework.Type

// Add fields to your Model for the Tabs Submodel and the active tab. The
// Submodel keeps private keyboard-focus state; the parent owns the active
// tab value and passes it back in as selectedValue.
const Model = S.Struct({
  tabs: Tabs.Model,
  activeFramework: Framework,
  // ...your other fields
})

// In your init function, initialize the Tabs Submodel with a unique id and
// pick the starting active tab:
const init = () => [
  {
    tabs: Tabs.init({ id: 'framework-tabs' }),
    activeFramework: 'Foldkit',
    // ...your other fields
  },
  [],
]

// Embed the Tabs Message in your parent Message:
const GotTabsMessage = m('GotTabsMessage', {
  message: Tabs.Message,
})

// Declare a typed Tabs factory once at module scope. The Value generic
// types tab.value in toView so the consumer can switch on it without
// casting:
const FrameworkTabs = Tabs.create<Framework>()

const frameworks: ReadonlyArray<Framework> = ['Foldkit', 'React', 'Elm']

const descriptions: Record<Framework, string> = {
  Foldkit: 'Model-View-Update with Effect.',
  React: 'Component-based with hooks.',
  Elm: 'The original MVU architecture.',
}

// Inside your update function's M.tagsExhaustive({...}), delegate to
// FrameworkTabs.update. The OutMessage's \`Selected\` carries the chosen
// value (typed as \`Framework\`) and its index. Fold the value into your
// own Model so it flows back in as selectedValue.
GotTabsMessage: ({ message }) => {
  const [nextTabs, commands, maybeOutMessage] = FrameworkTabs.update(
    model.tabs,
    message,
  )
  const mappedCommands = Command.mapMessages(commands, message =>
    GotTabsMessage({ message }),
  )

  return Option.match(maybeOutMessage, {
    onNone: () => [evo(model, { tabs: () => nextTabs }), mappedCommands],
    onSome: M.type<Tabs.OutMessage<Framework>>().pipe(
      M.tagsExhaustive({
        Selected: ({ value }) => [
          // The child has emitted \`Selected\`. Commit the child's next
          // interaction state and store the selected value as the new
          // active tab. In this arm the parent can also update its own
          // state or dispatch Commands, for example route to a new URL,
          // persist the selection, or trigger a panel content fetch.
          evo(model, {
            tabs: () => nextTabs,
            activeFramework: () => value,
          }),
          mappedCommands,
        ],
      }),
    ),
  })
}

// Inside your view function, embed the tabs via h.submodel and pass the
// parent-owned active tab as selectedValue:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'framework-tabs',
    model: model.tabs,
    view: FrameworkTabs.view,
    viewInputs: {
      tabs: frameworks,
      selectedValue: model.activeFramework,
      ariaLabel: 'Framework comparison',
      toView: ({ tablist, tabs, activeIndex }) =>
        h.div(
          [],
          [
            h.div(
              [...tablist, h.Class('flex')],
              tabs.map(tab =>
                h.button(
                  [
                    ...tab.tab,
                    h.Class(
                      'px-4 py-2 rounded-t-lg border data-[selected]:bg-white data-[selected]:border-b-0',
                    ),
                  ],
                  [h.span([], [tab.value])],
                ),
              ),
            ),
            ...tabs
              .filter(tab => tab.index === activeIndex)
              .map(tab =>
                h.div(
                  [...tab.panel, h.Class('p-6 border rounded-b-lg')],
                  [h.p([], [descriptions[tab.value]])],
                ),
              ),
          ],
        ),
    },
    toParentMessage: message => GotTabsMessage({ message }),
  })
```

### Vertical

Pass `orientation: 'Vertical'` to switch to up/down arrow navigation.

```
// Pseudocode walkthrough using the same Model, init, Message, and update
// as the basic tabs; only the view config changes to set orientation and
// use flex + flex-col for layout.
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'

import { Tabs } from '@foldkit/ui'

const GotTabsMessage = m('GotTabsMessage', {
  message: Tabs.Message,
})

const Framework = S.Literals(['Foldkit', 'React', 'Elm'])
type Framework = typeof Framework.Type

const FrameworkTabs = Tabs.create<Framework>()
const frameworks: ReadonlyArray<Framework> = ['Foldkit', 'React', 'Elm']

const descriptions: Record<Framework, string> = {
  Foldkit: 'Model-View-Update with Effect.',
  React: 'Component-based with hooks.',
  Elm: 'The original MVU architecture.',
}

// Inside your view function, set orientation to 'Vertical' and use flex +
// flex-col for layout:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: 'framework-tabs',
    model: model.tabs,
    view: FrameworkTabs.view,
    viewInputs: {
      tabs: frameworks,
      selectedValue: model.activeFramework,
      ariaLabel: 'Framework comparison',
      orientation: 'Vertical',
      toView: ({ tablist, tabs, activeIndex }) =>
        h.div(
          [h.Class('flex')],
          [
            h.div(
              [...tablist, h.Class('flex flex-col')],
              tabs.map(tab =>
                h.button(
                  [
                    ...tab.tab,
                    h.Class(
                      'px-4 py-2 text-left rounded-l-lg border mr-[-1px] data-[selected]:bg-white data-[selected]:border-r-0',
                    ),
                  ],
                  [h.span([], [tab.value])],
                ),
              ),
            ),
            ...tabs
              .filter(tab => tab.index === activeIndex)
              .map(tab =>
                h.div(
                  [...tab.panel, h.Class('flex-1 p-6 border rounded-r-lg')],
                  [h.p([], [descriptions[tab.value]])],
                ),
              ),
          ],
        ),
    },
    toParentMessage: message => GotTabsMessage({ message }),
  })
```

## Styling

Tabs is headless. The `toView` callback owns all tab and panel markup, spreading the attribute bundles from each `TabInfo` onto the consumer's elements. A common styling trick is to use a negative margin (`mb-[-1px]` for horizontal, `mr-[-1px]` for vertical) on the active tab to overlap the panel border.

| Attribute | Condition |
| --- | --- |
| `data-selected` | Present on the active tab button and its panel. |
| `data-disabled` | Present on disabled tab buttons. |

## Keyboard Interaction

Tabs uses roving tabindex: only the focused tab is in the tab order. Arrow direction depends on orientation: left/right for horizontal, up/down for vertical. Disabled tabs are skipped during navigation.

| Key | Description |
| --- | --- |
| `Arrow Right / Down` | Move to the next tab. In Automatic mode, also selects it. |
| `Arrow Left / Up` | Move to the previous tab. In Automatic mode, also selects it. |
| `Home` | Move to the first tab. |
| `End` | Move to the last tab. |
| `Enter / Space` | Select the focused tab (Manual mode only; Automatic selects on focus). |

## Accessibility

The tab list receives `role="tablist"` with `aria-orientation` and `aria-label`. Each tab button gets `role="tab"` with `aria-selected` and `aria-controls` linking to its panel. Panels receive `role="tabpanel"` with `aria-labelledby` pointing back to the tab.

## API Reference

### InitConfig

Configuration object passed to `Tabs.init()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the tabs instance. |
| `activationMode` | `'Automatic' \| 'Manual'` | `'Automatic'` | In Automatic mode, arrow keys select tabs on focus. In Manual mode, arrow keys focus only. Enter or Space is required to select. |

### ViewConfig

Configuration object passed to the view returned by `Tabs.create<Value>()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `model` | `Tabs.Model` | — | The tabs state from your parent Model. |
| `toParentMessage` | `(childMessage: Tabs.Message) => ParentMessage` | — | Wraps Tabs Messages in your parent Message type for Submodel delegation. |
| `tabs` | `ReadonlyArray<Value>` | — | The list of tab values, in display order. When the tabs component is declared via `Tabs.create<MyUnion>()`, `Value` is your union type and each `TabInfo.value` is typed as `MyUnion`. |
| `selectedValue` | `Value` | — | The active tab, owned by the parent Model and passed back in on each render. `aria-selected`, the `data-selected` marker, the active panel, and `RenderInfo.activeIndex` all derive from it. Update it by folding the `Selected` OutMessage in your `GotTabsMessage` handler. |
| `ariaLabel` | `string` | — | Accessible label for the tab list. |
| `toView` | `(render: RenderInfo<Value>) => Html` | — | Callback that receives the `tablist` attribute bundle, one `TabInfo<Value>` per tab, and the current `activeIndex`. Returns the composed layout. |
| `isTabDisabled` | `(value: Value, index: number) => boolean` | — | Disables individual tabs. |
| `orientation` | `'Horizontal' \| 'Vertical'` | `'Horizontal'` | Controls arrow key direction and `aria-orientation`. Horizontal uses left/right, vertical uses up/down. |

### RenderInfo

Payload delivered to the `toView` callback each render.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `tablist` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the tab list container. Includes `role="tablist"`, `aria-orientation`, and `aria-label`. |
| `tabs` | `ReadonlyArray<TabInfo<Value>>` | — | One entry per tab in `ViewConfig.tabs`, in the same order. See TabInfo below. |
| `activeIndex` | `number` | — | The currently-active tab index. Convenient when the consumer wants to render only the active panel (vs all panels with `hidden` for transitions). |

### TabInfo

Each entry in `RenderInfo.tabs`. Carries the value, derived state flags, and attribute bundles for the tab button and its panel.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `value` | `Value` | — | The tab value. Typed as your `Value` union when the tabs component is declared via `Tabs.create<Value>()`. |
| `index` | `number` | — | Position in the `tabs` array. |
| `isActive` | `boolean` | — | Whether this tab is currently active. |
| `isFocused` | `boolean` | — | Whether this tab owns the roving tabindex (the one in the tab order). |
| `isDisabled` | `boolean` | — | Whether this tab is disabled via `isTabDisabled`. |
| `tab` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the tab button element. Includes `role="tab"`, `type="button"`, `aria-selected`, `aria-controls`, `tabindex`, the click handler, and the keyboard handler. |
| `panel` | `ReadonlyArray<ChildAttribute>` | — | Spread onto the tab panel element. Includes `role="tabpanel"`, `aria-labelledby` pointing back to the tab, and `tabindex`. |

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Pattern-match on the OutMessage in your update handler.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `Selected` | `{ value: Value; index: number }` | — | Emitted when a tab is committed via click or keyboard. Carries both the tab’s value (typed as your `Value` union via `Tabs.create<Value>()`) and its index. Pattern-match the third tuple element of `Tabs.update` in your `GotTabsMessage` handler. |
