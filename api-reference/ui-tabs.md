---
url: https://foldkit.dev/api-reference/ui-tabs
title: "Ui/Tabs"
description: "API documentation for the Ui/Tabs module."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
---

# Ui/Tabs

## Functions

### create

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L382)

```
/**
 * Pairs the tabs `view` and `update` behind a single Value-typed entry
 *  point. Declare once at module scope so consumers receive
 *  `tab.value: Value` in `toView` and the `Selected` OutMessage without an
 *  `as` cast:
 * 
 *  ```ts
 *  const DemoTabs = Tabs.create<DemoTab>()
 * 
 *  // In view (selectedValue is the parent-owned active tab):
 *  h.submodel({ view: DemoTabs.view, viewInputs: { selectedValue, ... }, ... })
 * 
 *  // In the parent update, pass DemoTabs.update to Update.foldChild and
 *  // fold the Selected OutMessage into your Model.
 *  ```
 * 
 *  The internal view stays typed `ReadonlyArray<string>`; consumers can
 *  pass a `ReadonlyArray<MyUnion>` (assignable) and the fenced cast inside
 *  `create` types `TabInfo.value` as `MyUnion`.
 */
<Value extends string = string>(): Bundle<Value>
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L88)

```
/**
 * Creates an initial tabs model from a config. Focus follows the selected
 *  tab until the user navigates in `Manual` mode, so `maybeFocusedIndex`
 *  starts `None`. Defaults to automatic activation.
 */
(config: InitConfig): Tabs.Model
```

## Types

### ActivationMode

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L22)

```
/** Controls whether tabs activate on focus (`Automatic`) or require an explicit selection (`Manual`). */
type ActivationMode = Literals<readonly ["Automatic", "Manual"]>
```

### Bundle

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L356)

```
/**
 * The `view` and `update` pair that `Tabs.create` returns, bound to one
 *  `Value` type. Name it to annotate a value that holds a created bundle,
 *  such as a field on a config object or a function parameter that takes
 *  the bundle rather than calling `create` itself.
 */
type Bundle = Readonly<{
  update: (model: Model, message: Message) => Update.ReturnWithOutMessage<Model, Message, OutMessage<Value>>
  view: SubmodelView<Model, Message, ViewInputs<Value>>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L80)

```
/** Configuration for creating a tabs model with `init`. */
type InitConfig = Readonly<{
  activationMode: ActivationMode
  id: string
}>
```

### OutMessage

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L65)

```
/**
 * Generic over `Value extends string` so consumers using
 *  `Tabs.create<MyUnion>()` receive `value: MyUnion` in the
 *  `Selected` OutMessage. Defaults to `string`.
 */
type OutMessage = Selected<Value>
```

### RenderInfo

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L158)

```
/**
 * Render-time payload published to the consumer's `toView`.
 * 
 *  - `tablist`: ARIA + role attributes for the wrapping tablist element.
 *  - `tabs`: one entry per tab in `viewInputs.tabs`, in the same order, with
 *    the tab button's attribute bundle, the panel's attribute bundle,
 *    and derived state.
 *  - `activeIndex`: the index of `viewInputs.selectedValue` within
 *    `viewInputs.tabs`, convenient when the consumer wants to render only the
 *    active panel (vs all panels with `Hidden` for transitions).
 */
type RenderInfo = Readonly<{
  activeIndex: number
  tablist: ReadonlyArray<ChildAttribute>
  tabs: ReadonlyArray<TabInfo<Value>>
}>
```

### Selected

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L57)

```
type Selected = Readonly<{
  _tag: "Selected"
  index: number
  value: Value
}>
```

### TabInfo

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L139)

```
/**
 * Per-tab render info passed to the consumer's `toView`. Generic over
 *  `Value extends string`: when `Tabs.create<MyUnion>()` is declared,
 *  `tab.value` is typed `MyUnion` so the consumer can switch on it without
 *  casting.
 */
type TabInfo = Readonly<{
  index: number
  isActive: boolean
  isDisabled: boolean
  isFocused: boolean
  panel: ReadonlyArray<ChildAttribute>
  tab: ReadonlyArray<ChildAttribute>
  value: Value
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L182)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 *  Generic over `Value extends string` so consumers using
 *  `Tabs.create<MyUnion>()` receive `tab.value: MyUnion` in `toView`
 *  and `(value: MyUnion, index) => boolean` in `isTabDisabled`, without
 *  casting.
 * 
 *  - `selectedValue`: the active tab, read straight from the parent Model.
 *    `aria-selected`, the `data-selected` marker, and which panel is active
 *    all derive from it.
 *  - `panelMount`: defaults to `ActiveOnly`, where the consumer renders only
 *    the active panel. Set `All` when the consumer keeps every panel mounted
 *    and hides inactive ones, so every tab retains its panel relationship.
 */
type ViewInputs = Readonly<{
  ariaLabel: string
  isTabDisabled: (value: Value, index: number) => boolean
  orientation: Orientation
  panelMount: PanelMount
  selectedValue: Value
  tabs: ReadonlyArray<Value>
  toView: (render: RenderInfo<Value>) => Html
}>
```

## Constants

### FocusTab

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L101)

```
/** Moves focus to the tab at the given index. */
const FocusTab: CommandDefinitionWithArgs<"FocusTab", {
  id: String
  index: Number
}, Effect<{
  _tag: "CompletedFocusTab"
}, never, never>>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L41)

```
/** Union of all messages the tabs component can produce. */
const Message: MessageUnion<{
  CompletedFocusTab: {}
  FocusedTab: {
    index: Number
  }
  SelectedTab: {
    index: Number
    value: String
  }
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L30)

```
/**
 * Schema for the tabs component's private interaction state. The active
 *  tab is owned by the parent and passed in via `ViewInputs.selectedValue`,
 *  so it is not stored here. `maybeFocusedIndex` is the roving-tabindex
 *  cursor: `None` means keyboard focus follows the selected tab, and `Manual`
 *  activation stores `Some(index)` while focus diverges from the selection.
 */
const Model: Struct<{
  activationMode: Literals<readonly ["Automatic", "Manual"]>
  id: String
  maybeFocusedIndex: Option<Number>
}>
```

### Orientation

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L18)

```
/** Controls the tab list layout direction and which arrow keys navigate between tabs. */
const Orientation: Literals<readonly ["Horizontal", "Vertical"]>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L65)

```
/**
 * Union of OutMessages the tabs component can produce. The parent's
 *  `Update.foldChild` config handles them through `foldOutMessage`.
 */
const OutMessage: MessageUnion<{
  Selected: {
    index: Number
    value: String
  }
}>
```

### PanelMount

const

[source](https://github.com/foldkit/foldkit/blob/b415a3e22be572abb1785010e23bd3e57f2e24f2/packages/ui/src/tabs/index.ts#L167)

```
/**
 * Describes whether a consumer renders only the active tab panel or keeps
 *  every panel mounted and hides inactive ones. Tab-to-panel references follow
 *  the rendered panel strategy.
 */
const PanelMount: Literals<readonly ["ActiveOnly", "All"]>
```
