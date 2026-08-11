---
url: https://foldkit.dev/ui/selection-submodels
title: "Selection Submodels"
description: "How Foldkit UI components expose create<Item>() factories that pair view and update behind one type parameter so Item types cannot drift between the rendered list and the selection handler."
access_date: 2026-08-11T02:16:28.996Z
current_date: 2026-08-11T02:16:28.996Z
---

## Selection Submodels

## Overview

Foldkit UI ships five Submodels for selecting one or more values from a set: [Listbox](https://foldkit.dev/ui/listbox), [Combobox](https://foldkit.dev/ui/combobox), [Tabs](https://foldkit.dev/ui/tabs), [Menu](https://foldkit.dev/ui/menu), and [RadioGroup](https://foldkit.dev/ui/radio-group). For example, a Listbox of plans, a Combobox of cities, a Tabs of view modes, a Menu of actions, or a RadioGroup of pricing plans.

Each exposes a `create<Item>()` factory that pairs the view and update behind a single type parameter, so the value type is fixed at the binding site and flows into the OutMessage.

A Listbox over a literal-union `Plan` type:

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Match as M, Option } from 'effect'
import { Update } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Listbox } from '@foldkit/ui'

const Plan = S.Literals(['Free', 'Pro', 'Enterprise'])
type Plan = typeof Plan.Type

// Declare a typed Listbox once at module scope. \`view\` and \`update\` are
// bound to \`Plan\`: \`items\` is typed as \`ReadonlyArray<Plan>\` and the
// OutMessage carries \`value: Plan\`.
const PlanListbox = Listbox.create<Plan>()

// Add a field to your Model for the Listbox Submodel, plus a field for
// the selected value your app actually cares about. Using the \`Plan\`
// Schema keeps the field literal-typed end to end:
const Model = S.Struct({
  maybePlan: S.Option(Plan),
  listbox: Listbox.Model,
  // ...your other fields
})

// In your init function, initialize the Listbox Submodel with a unique id:
const init = () => [
  {
    maybePlan: Option.none(),
    listbox: Listbox.init({ id: 'plan' }),
    // ...your other fields
  },
  [],
]

// Wrap Listbox's Messages so they can flow through your update:
const GotListboxMessage = m('GotListboxMessage', {
  message: Listbox.Message,
})

// At module scope, fold the OutMessage into your own Model. When the user
// commits a selection it carries \`Selected({ value })\` where \`value: Plan\`.
// The arm returns an Update.Step over the parent Model, which already has the
// next Listbox Model written back:
const foldListboxOutMessage = M.type<Listbox.OutMessage<Plan>>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    Selected:
      ({ value }) =>
      model => [evo(model, { maybePlan: () => Option.some(value) }), []],
  }),
)

// Update.foldChild wires the child into the parent: it delegates keyboard
// navigation, typeahead, and open/close to PlanListbox.update, writes the next
// Listbox Model back, maps the Submodel's Commands into your Message type, and
// hands any OutMessage to foldOutMessage.
const foldListbox = Update.foldChild({
  update: PlanListbox.update,
  read: (model: Model) => Option.some(model.listbox),
  write: (model, nextListbox) => evo(model, { listbox: () => nextListbox }),
  toParentMessage: message => GotListboxMessage({ message }),
  foldOutMessage: foldListboxOutMessage,
})

// Inside your update function's M.tagsExhaustive({...}), call the fold:
GotListboxMessage: ({ message }) => foldListbox(model, message)

const plans: ReadonlyArray<Plan> = ['Free', 'Pro', 'Enterprise']

// Inside your view function, embed the Listbox via h.submodel using
// \`PlanListbox.view\`. Associate a visible label with the trigger via a native
// \`<label for>\`: target the trigger id with \`Listbox.buttonId('plan')\`. The
// \`for\` association gives the trigger both its accessible name and
// click-to-focus, so ariaLabelledBy is not needed here.
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.div(
    [],
    [
      h.label([h.For(Listbox.buttonId('plan'))], ['Plan']),
      h.submodel({
        slotId: 'plan',
        model: model.listbox,
        view: PlanListbox.view,
        viewInputs: {
          // \`items\` must be ReadonlyArray<Plan>. The factory's <Plan> parameter constrains the shape.
          items: plans,
          // The parent owns the selection and passes it in. Single-select
          // takes an Option: None when nothing is selected yet.
          maybeSelectedValue: model.maybePlan,
          buttonContent: h.span(
            [],
            [Option.getOrElse(model.maybePlan, () => 'Select a plan')],
          ),
          buttonClassName: 'w-full rounded-lg border px-3 py-2 text-left',
          itemsClassName: 'rounded-lg border shadow-lg',
          itemToConfig: (plan, { isSelected, isActive }) => ({
            className: isActive ? 'bg-blue-100' : '',
            content: h.div(
              [h.Class('flex items-center gap-2 px-3 py-2')],
              [
                isSelected ? h.span([], ['✓']) : h.span([h.Class('w-4')]),
                h.span([], [plan]),
              ],
            ),
          }),
          backdropClassName: 'fixed inset-0',
          anchor: { placement: 'bottom-start', gap: 4, padding: 8 },
        },
        toParentMessage: message => GotListboxMessage({ message }),
      }),
    ],
  )
```

## The create<Item>() Factory

A call to `Listbox.create<Plan>()` returns an object whose entry points are all bound to `Plan`: `view` accepts `items: ReadonlyArray<Plan>`, `update` returns an OutMessage carrying the picked `Plan`, and the imperative helpers the Submodel exposes (`selectItem`, `open`, and `close` for Listbox and Combobox) accept and emit `Plan` too. Declare the factory once at module scope and use the same bundle at every site that needs it.

There is no inbound reflect helper for the selection: the parent owns it outright and passes it in as `maybeSelectedValue` (`selectedValues` for multi-select), so there is nothing on the Listbox or Combobox to reflect onto. When an external value (a URL parameter, restored storage, a server push) changes the selection, the parent writes its own field. The `reflect*` family lives on the components with configuration the parent feeds in: `reflectMinDate`, `reflectMaxDate`, `reflectDisabledDates`, and `reflectDisabledDaysOfWeek` on Calendar and DatePicker, and `reflectRange` on Slider. See [Reflecting External State](https://foldkit.dev/core/submodel#reflecting-external-state) for the concept.

## Naming What create Returns

Each component exports a `Bundle` type for what its factory returns, taking the same type parameters as the factory itself. `Listbox.Bundle<Plan>` is what `Listbox.create<Plan>()` produces, `Menu.Bundle<Action>` is what `Menu.create<Action>()` produces, and the multi-select variants export their own under `Listbox.Multi.Bundle` and `Combobox.Multi.Bundle`.

Declaring the factory at module scope and using it directly needs no annotation, since inference covers it. Reach for `Bundle` when a created bundle has to be named instead. For example: a config object with a field typed `Combobox.Bundle<City>`, or a view helper whose parameter is `Listbox.Bundle<Plan>` because it receives the bundle rather than calling `create` itself.

The name also matters to consumers that emit their own declarations. Without it, TypeScript has to expand the factory's whole result into the generated `.d.ts` at every use site, and it refuses where that expansion reaches a type the consumer cannot name.

## The Submodel Doesn’t Own Your Selection

A common first question is: if the Listbox is Item-typed, why does my own Model still hold an `Option<Plan>` for the picked value? Isn’t that the same state twice?

It isn’t. The Listbox’s Model is UI state: open vs. closed, which option the keyboard is focused on, the typeahead key buffer. It deliberately does not hold the committed selection, because committed selections are domain truth. The Submodel hands you that truth at the moment the user commits via the OutMessage; your update lifts it into your own Model, where it belongs.

That split is why the Listbox Model can stay generic-free (no `Listbox.Model<Item>`) while `Item` still flows into your code with full type safety. The generic threads through the boundary (`items` in, OutMessage `value` out), and nowhere else. If the selection needs to persist, store it in your Model. If the commit just dispatches a Command (for example a Menu of actions), no Model field is needed.

## Why the Factory Exists

Without the factory, the view and update would each carry their own `Item` type parameter. Nothing would stop a consumer from writing `view: Listbox.view<Plan>()` next to `Listbox.update<Color>(...)`. Two different type arguments at the same call site. The selected item would arrive in the OutMessage typed as one and the update would believe it was the other, and TypeScript would have no way to flag the mismatch.

The factory closes that hole by setting `Item` once. The returned `view` and `update` are bound to the same `Item` because both come from the same factory call.

Internally, each Submodel’s view and update are written against an untyped string value and then cast back to the consumer’s `Item` at the factory boundary. The cast is sound because the value being emitted came from the same `items` array the consumer just supplied. The fence keeps a single `Item` type on both sides of that cast.

## The Factories

Each Submodel exposes a `create<...>()` factory. The shape of the type parameter differs by what the Submodel accepts as items.

### Listbox

`Listbox.create<Item, Value>()` takes two type parameters, or `Listbox.create<Item>()` when relying on the default. `Item` is the shape of the items the consumer supplies. `Value` is the shape of the value the OutMessage carries; it defaults to `Item` when `Item extends string`, else `string`. The two-parameter shape supports object-typed items via an `itemToValue` callback that extracts the stringy identifier from each `Item`.

`Listbox.Multi.create<Item, Value>()` (or `Listbox.Multi.create<Item>()`) is the multi-select variant. Same type-parameter shape; the `Selected` OutMessage carries only the activated `value`, and the parent toggles that value in and out of the selection it owns.

### Combobox

`Combobox.create<Item>()` takes one type parameter and constrains `Item extends string`. Combobox items are typed strings (a literal union, a branded string type, or plain `string`).

`Combobox.Multi.create<Item>()` is the multi-select variant. Same type-parameter shape; the `Selected` OutMessage carries only the activated `value`, and the parent toggles that value in and out of the selection it owns.

### Tabs

`Tabs.create<Value>()` takes one type parameter, `Value extends string`. The view accepts `ReadonlyArray<Value>` as its tab list (a literal union `Value` is assignable to `string`), and the OutMessage carries both the picked `value: Value` and its `index: number`. The single parameter is enough because Tabs values are always inline strings; there is no object form.

### Menu

`Menu.create<Item>()` takes one type parameter, `Item extends string`. The view accepts `ReadonlyArray<Item>` as its menu items, and the OutMessage carries both the picked `value: Item` and its `index: number`. The picked value arrives directly in the OutMessage, so consumers no longer need to look it up from their own items array.
