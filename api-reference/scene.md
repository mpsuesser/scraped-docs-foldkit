---
url: https://foldkit.dev/api-reference/scene
title: "Scene"
description: "API documentation for the Scene module."
access_date: 2026-08-07T02:46:48.844Z
current_date: 2026-08-07T02:46:48.844Z
---

# Scene

## Functions

### altText

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1240)

```
/** Creates a Locator that finds an element by its `alt` attribute. */
(altValue: string): Locator
```

### blur

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1799)

```
/** Simulates a blur event on the element matching the target. */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### click

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1391)

```
/**
 * Simulates a click on the element matching the target.
 *  When the element has no click handler, the event bubbles up to the
 *  nearest ancestor with one, mirroring browser event propagation.
 *  When the element is a submit button (`<button>` with no type or
 *  `type="submit"`, `<input type="submit">`, `<input type="image">`) with no
 *  click handler in its ancestor chain, the click falls through to the
 *  `submit` handler of the nearest ancestor `<form>`.
 */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### contextMenu

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1545)

```
/**
 * Simulates a contextmenu event on the element matching the target.
 *  When the element has no contextmenu handler, the event bubbles up to the
 *  nearest ancestor with one, mirroring browser event propagation.
 */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### displayValue

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1252)

```
/** Creates a Locator that finds a form control by its current `value`. */
(valueString: string): Locator
```

### doubleClick

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1483)

```
/**
 * Simulates a double-click on the element matching the target.
 *  When the element has no dblclick handler, the event bubbles up to the
 *  nearest ancestor with one, mirroring browser event propagation.
 */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### expect

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L2308)

### expectAll

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L2364)

### expectNoOutMessage

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1272)

```
/**
 * Asserts that the OutMessage is None.
 * 
 *  The tracked OutMessage is the third element of the most recent update
 *  result that had one. An update branch that returns a two-tuple leaves
 *  the previous value latched in place, so keep every branch of an
 *  OutMessage-returning update on the three-tuple shape, returning
 *  `Option.none()` when there is nothing to report.
 */
(): (simulation: SceneSimulation<Model, Message, Option.Option<OutMessage>>) => SceneSimulation<Model, Message, Option.Option<OutMessage>>
```

### expectOutMessage

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1244)

```
/**
 * Asserts that the OutMessage is Some with the expected value.
 * 
 *  The tracked OutMessage is the third element of the most recent update
 *  result that had one. An update branch that returns a two-tuple leaves
 *  the previous value latched in place, so keep every branch of an
 *  OutMessage-returning update on the three-tuple shape, returning
 *  `Option.none()` when there is nothing to report.
 */
<Expected>(expected: Expected): (simulation: SceneSimulation<Model, Message, Option.Option<OutMessage>>) => SceneSimulation<Model, Message, Option.Option<OutMessage>>
```

### first

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1339)

```
/** Picks the first match from a LocatorAll, producing a single-match Locator. */
(locatorAll: LocatorAll): Locator
```

### focus

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1789)

```
/** Simulates a focus event on the element matching the target. */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### getAllByAltText

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1138)

```
/** Finds all elements with the given `alt` attribute. */
(altValue: string): (html: VNode) => readonly Array<VNode>
```

### getAllByDisplayValue

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1156)

```
/** Finds all form controls whose current value matches. */
(displayValueString: string): (html: VNode) => readonly Array<VNode>
```

### getAllByLabel

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1052)

```
/**
 * Finds every element with the given label text. Applies the same four
 *  resolution strategies as `getByLabel` (`aria-label`, `<label for="id">`,
 *  `<label>` nesting, `aria-labelledby`) and returns deduplicated matches.
 */
(labelValue: string): (html: VNode) => readonly Array<VNode>
```

### getAllByPlaceholder

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1129)

```
/** Finds all elements with the given placeholder attribute. */
(placeholderValue: string): (html: VNode) => readonly Array<VNode>
```

### getAllByRole

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L956)

```
/** Finds all elements with the given ARIA role and optional matching options. */
(
  role: string,
  options?: Readonly<{
    checked: boolean | "mixed"
    disabled: boolean
    expanded: boolean
    level: number
    name: string | RegExp
    pressed: boolean | "mixed"
    selected: boolean
  }>
): (html: VNode) => readonly Array<VNode>
```

### getAllByTestId

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1150)

```
/** Finds all elements with the given `data-testid` attribute. */
(testIdValue: string): (html: VNode) => readonly Array<VNode>
```

### getAllByText

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1115)

```
/**
 * Finds all elements matching the given text content.
 *  Includes nested ancestors — a `<div><p>hi</p></div>` with text "hi" yields both.
 */
(
  target: string,
  options?: Readonly<{
    exact: boolean
  }>
): (html: VNode) => readonly Array<VNode>
```

### getAllByTitle

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1144)

```
/** Finds all elements with the given `title` attribute. */
(titleValue: string): (html: VNode) => readonly Array<VNode>
```

### getByAltText

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1093)

```
/** Finds the first element with the given `alt` attribute. */
(altValue: string): (html: VNode) => Option<VNode>
```

### getByDisplayValue

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1168)

```
/**
 * Finds the first form control whose current value matches. Checks the `value`
 *  attribute on inputs, textareas, and selects.
 */
(displayValue: string): (html: VNode) => Option<VNode>
```

### getByLabel

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1008)

```
/**
 * Finds the first element with the given label text. Checks `aria-label`
 *  first, then `<label for="id">` association, then `<label>` nesting,
 *  then `aria-labelledby` reverse lookup.
 */
(labelValue: string): (html: VNode) => Option<VNode>
```

### getByPlaceholder

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L997)

```
/** Finds the first element with the given placeholder attribute. */
(placeholderValue: string): (html: VNode) => Option<VNode>
```

### getByRole

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L942)

```
/**
 * Finds the first element with the given ARIA role and optional matching options.
 *  Supports `name` (accessible name), `level` (heading level), `checked`,
 *  `selected`, `pressed`, `expanded`, and `disabled` state filters.
 */
(
  role: string,
  options?: Readonly<{
    checked: boolean | "mixed"
    disabled: boolean
    expanded: boolean
    level: number
    name: string | RegExp
    pressed: boolean | "mixed"
    selected: boolean
  }>
): (html: VNode) => Option<VNode>
```

### getByTestId

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1105)

```
/** Finds the first element with the given `data-testid` attribute. */
(testIdValue: string): (html: VNode) => Option<VNode>
```

### getByText

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L971)

```
/**
 * Finds the most specific element matching the given text content.
 *  Skips text VNodes (sel undefined) — only returns actual DOM elements.
 */
(
  target: string,
  options?: Readonly<{
    exact: boolean
  }>
): (html: VNode) => Option<VNode>
```

### getByTitle

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1099)

```
/** Finds the first element with the given `title` attribute. */
(titleValue: string): (html: VNode) => Option<VNode>
```

### given

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L617)

```
/** Sets the initial Model for a scene test. */
<Model>(model: Model): GivenStep<Model>
```

### hover

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1770)

```
/**
 * Simulates a hover (mouseenter) on the element matching the target.
 *  Dispatches the `mouseenter` handler, falling back to `mouseover`.
 */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### inside

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1342)

```
/**
 * Scopes a sequence of steps to a parent element. Every Locator referenced by
 *  child steps (assertions, interactions) resolves within the parent's subtree.
 *  Use this when several steps share the same scope. For a single scoped query,
 *  prefer `within(parent, child)` directly. Nested `inside` calls compose scopes
 *  via `within(outer, inner)`.
 */
<Model, Message, OutMessage = undefined>(
  parent: Locator,
  steps: readonly Array<NoInfer<SceneStep<Model, Message, OutMessage>>>
): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### label

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1236)

```
/** Creates a Locator that finds an element by aria-label. */
(labelValue: string): Locator
```

### last

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1346)

```
/** Picks the last match from a LocatorAll, producing a single-match Locator. */
(locatorAll: LocatorAll): Locator
```

### placeholder

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1229)

```
/** Creates a Locator that finds an element by placeholder attribute. */
(placeholderValue: string): Locator
```

### pointerDown

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1624)

```
/**
 * Simulates a pointerdown event on the element matching the target.
 *  When the element has no pointerdown handler, the event bubbles up to
 *  the nearest ancestor with one, mirroring browser event propagation.
 *  Defaults to `pointerType: 'mouse'`, `button: 0`, and `screenX/screenY: 0`.
 */
(
  target: string | Locator,
  options?: Readonly<{
    button: number
    clientX: number
    clientY: number
    pointerType: string
    screenX: number
    screenY: number
  }>
): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### pointerUp

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1708)

```
/**
 * Simulates a pointerup event on the element matching the target.
 *  When the element has no pointerup handler, the event bubbles up to
 *  the nearest ancestor with one, mirroring browser event propagation.
 *  Defaults to `pointerType: 'mouse'` and `screenX/screenY: 0`.
 */
(
  target: string | Locator,
  options?: Readonly<{
    pointerType: string
    screenX: number
    screenY: number
  }>
): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### role

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1220)

```
/**
 * Creates a Locator that finds an element by ARIA role. Supports matching on
 *  `name`, `level`, `checked`, `selected`, `pressed`, `expanded`, and `disabled`.
 */
(
  roleValue: string,
  options?: Readonly<{
    checked: boolean | "mixed"
    disabled: boolean
    expanded: boolean
    level: number
    name: string | RegExp
    pressed: boolean | "mixed"
    selected: boolean
  }>
): Locator
```

### selector

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1263)

```
/**
 * Creates a Locator that wraps a CSS selector. Escape hatch for cases
 *  where no accessible attribute is available.
 */
(css: string): Locator
```

### submit

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1900)

```
/** Simulates form submission on the element matching the target. */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### tap

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1293)

```
/** Runs a function for side effects (e.g. assertions) without breaking the step chain. */
<Model, Message, OutMessage = undefined>(f: (simulation: SceneSimulation<Model, Message, OutMessage>) => void): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### testId

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1248)

```
/** Creates a Locator that finds an element by its `data-testid` attribute. */
(testIdValue: string): Locator
```

### text

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1256)

```
/** Creates a Locator that finds the most specific element matching the given text content. */
(
  target: string,
  options?: Readonly<{
    exact: boolean
  }>
): Locator
```

### textContent

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L758)

```
/** Extracts all text content from a VNode tree, depth-first. */
(vnode: VNode): string
```

### title

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1244)

```
/** Creates a Locator that finds an element by its `title` attribute. */
(titleValue: string): Locator
```

### withViewInputs

function

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L2387)

```
/**
 * Adapts a Submodel view that declares `ViewInputs` to the `(model, h)`
 *  shape `Scene.scene` takes. `defaults` supplies the full `ViewInputs`
 *  once; the returned factory accepts per-test overrides for everything
 *  except `toView`, so tests vary value inputs while the renderer stays
 *  pinned:
 * 
 *  ```ts
 *  const sceneView = Scene.withViewInputs(view, {
 *    value: 5,
 *    toView: testToView,
 *  })
 * 
 *  Scene.scene({ update, view: sceneView() }, ...)
 *  Scene.scene({ update, view: sceneView({ isDisabled: true }) }, ...)
 *  ```
 */
<Model, Message, ViewInputs extends object>(
  view: (model: Model, viewInputs: ViewInputs, h: HtmlBuilder<Message>) => Html,
  defaults: NoInfer<ViewInputs>
): (overrides?: Omit<Partial<NoInfer<ViewInputs>>, "toView">) => (model: Model, h: HtmlBuilder<Message>) => Html
```

## Types

### AnyCommand

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/internal.ts#L28)

```
/**
 * A Command in a test simulation. Carries `name` and optionally the `args`
 *  the runtime captured at construction. Instance matchers (Command values
 *  produced by calling a Definition) are matched against this shape; the
 *  `effect` field on a real Command is irrelevant for matching, so we only
 *  retain `name + args`.
 * 
 *  `messageMappers` is the Command's message-mapping chain (from
 *  `Command.mapMessage`/`mapMessages`). It rides along on pending Commands so
 *  `resolve` can apply the parent's wrapping to a substitute result Message
 *  without the test restating it. Absent on synthetic matchers built for
 *  formatting; treated as empty then.
 * 
 *  `key` is present on Commands built with an interruptible `Command.define`:
 *  such Commands may stay pending across Messages (they model long-running
 *  work). `interruptsKey` is present on Interrupt Commands: resolving one
 *  drops every pending Command holding that key, mirroring the runtime
 *  guarantee that an interrupted Command's result Message never dispatches.
 */
type AnyCommand = Readonly<{
  args: Record<string, unknown>
  interruptsKey: string
  key: string
  messageMappers: ReadonlyArray<(message: unknown) => unknown>
  name: string
}>
```

### AnyMount

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/internal.ts#L134)

```
/**
 * A Mount lifecycle event in a test simulation. Carries `name` and
 *  optionally the `args` used to construct the MountAction. Mirrors
 *  `AnyCommand` for Mount matchers.
 */
type AnyMount = Readonly<{
  args: Record<string, unknown>
  name: string
}>
```

### Locator

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1183)

```
/**
 * A deferred element query that resolves against a VNode tree. Callable as a
 *  function (`locator(html)`) so it composes directly in `flow` and `pipe` chains.
 *  Used by interaction steps (`click`, `type`, `submit`, `keydown`) to
 *  target elements by accessible attributes instead of CSS selectors.
 */
type Locator = (html: VNode) => Option.Option<VNode> & Readonly<{
  description: string
}>
```

### LocatorAll

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1189)

```
/**
 * A deferred multi-element query that resolves to all matching VNodes.
 *  Produced by `all*` locator factories and by `filter(...)`. Convert to a
 *  single-match `Locator` via `first`, `last`, or `nth(n)`.
 */
type LocatorAll = (html: VNode) => ReadonlyArray<VNode> & Readonly<{
  description: string
}>
```

### MountMatcher

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/internal.ts#L147)

```
/**
 * Pattern for matching a pending Mount in test assertions. A Definition
 *  matches by name only ("a Mount with this identity is in the rendered
 *  tree"); an Instance matches by name AND structural-equal args ("a Mount
 *  with this identity AND these args"). Choose the form per assertion based
 *  on whether the test cares about the args value.
 * 
 *  Two modes only: name-only or name + full args. Partial-args matching is
 *  intentionally unsupported, mirroring `CommandMatcher`.
 */
type MountMatcher = MountDefinition<string, unknown> | AnyMount
```

### MountResolver

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/internal.ts#L189)

```
/**
 * A Mount matcher (Definition or Instance) paired with the raw result Message
 *  to resolve it with. Mirrors `Resolver` for Commands. When the mount lives
 *  inside a Submodel, the boundary's own `toParentMessage` chain (snapshotted at
 *  render time) is applied to the result, so pass the child's raw result
 *  Message, not a parent-wrapped one.
 */
type MountResolver = readonly [MountMatcher, ResultMessage]
```

### PendingMount

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/internal.ts#L124)

```
/**
 * A pending Mount in a Scene simulation. Identified by `name` and an
 *  `occurrence` index that disambiguates same-named mounts in the rendered
 *  tree (e.g. two open popovers each contributing an `AnchorPopover`). The
 *  occurrence is the 0-based position among same-named markers in
 *  tree-traversal order. `args` carries the runtime values used to construct
 *  the MountAction when its definition declared an args record.
 */
type PendingMount = Readonly<{
  args: Record<string, unknown>
  messageMappers: ReadonlyArray<(message: unknown) => unknown>
  name: string
  occurrence: number
}>
```

### Resolver

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/internal.ts#L180)

```
/**
 * A Command matcher (Definition or Instance) paired with the raw result
 *  Message to resolve a pending Command with. Definition matchers resolve by
 *  name; an Instance matcher resolves only the pending Command whose name AND
 *  args match. The matched Command's own message-mapping chain is applied to
 *  the result, so pass the child's raw result Message, not a parent-wrapped
 *  one. The result Message must belong to the matched Command.
 */
type Resolver = readonly [Matcher, ResultMessageForMatcher<Matcher>]
```

### SceneSimulation

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L165)

```
/**
 * An immutable test simulation that includes the rendered VNode tree.
 *  The Model and Message are intentionally opaque. Scene tests assert
 *  through the view, not the model. Use Story for model-level assertions.
 */
type SceneSimulation = Readonly<{
  commands: ReadonlyArray<AnyCommand>
  html: VNode
  mounts: ReadonlyArray<PendingMount>
  outMessage: OutMessage
}>
```

### SceneStep

type

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L181)

```
/** A single step in a scene: either a `given` step or a scene simulation transform. */
type SceneStep = GivenStep<NoInfer<Model>> | (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

## Constants

### Command

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1161)

```
/**
 * Steps that operate on the pending Commands of a scene simulation.
 *  Destructure as `const { Command } = Scene` for concise call sites.
 */
const Command: {
  expectExact: (matchers: readonly Array<CommandMatcher>) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  expectHas: (matchers: readonly Array<CommandMatcher>) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  expectNone: () => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  resolve: (definition: ResolvableCommandDefinition<Name, ResultMessage>, resultMessage: ResultMessage) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  resolveAll: (resolvers: {
    [K in string | number | symbol]: Resolver<Matchers[K]>
  }) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  resolveAllExact: (resolvers: {
    [K in string | number | symbol]: Resolver<Matchers[K]>
  }) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
}
```

### CustomElement

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1231)

```
/**
 * Steps that model CustomEvents arriving from a rendered custom element.
 *  Destructure as `const { CustomElement } = Scene` for concise call sites.
 */
const CustomElement: {
  emit: (spec: CustomElementSpec<string, Record<string, Top>, Events>, target: string | Locator, eventName: Name, detail: Type<Events[Name]>) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
}
```

### ManagedResource

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1216)

```
/**
 * Steps that model the lifecycle Messages of a ManagedResource. The test
 *  declares the lifecycle outcome the way `Scene.Command.resolve` declares a
 *  Command result, and each step checks the current Model against the
 *  entry's `modelToMaybeRequirements` gate first, mirroring the runtime's
 *  None to Some and Some to None transitions. The Some to Some re-acquire
 *  transition has no step yet. Destructure as
 *  `const { ManagedResource } = Scene` for concise call sites.
 */
const ManagedResource: {
  acquire: (entry: SceneManagedResourceEntry<EntryModel, EntryMessage, Value, (args: Args) => EntryMessage>, args: NoInfer<Args>) => (simulation: SceneSimulation<EntryModel, Message, OutMessage>) => SceneSimulation<EntryModel, Message, OutMessage>
  failAcquire: (entry: SceneManagedResourceEntry<EntryModel, EntryMessage, Value>, error: unknown) => (simulation: SceneSimulation<EntryModel, Message, OutMessage>) => SceneSimulation<EntryModel, Message, OutMessage>
  release: (entry: SceneManagedResourceEntry<EntryModel, EntryMessage, Value>) => (simulation: SceneSimulation<EntryModel, Message, OutMessage>) => SceneSimulation<EntryModel, Message, OutMessage>
}
```

### Mount

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1184)

```
/**
 * Steps that operate on the pending Mounts of a scene simulation.
 *  Destructure as `const { Mount } = Scene` for concise call sites.
 */
const Mount: {
  expectEnded: (matchers: readonly Array<MountMatcher>) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  expectExact: (matchers: readonly Array<MountMatcher>) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  expectHas: (matchers: readonly Array<MountMatcher>) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  expectNone: () => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  resolve: (matcher: Readonly<{
    args: Record<string, unknown>
    name: string
  }> | MountDefinition<Name, ResultMessage>, resultMessage: ResultMessage) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
  resolveAll: (resolvers: {
    [K in string | number | symbol]: MountResolver<R[K]>
  }) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
}
```

### Subscription

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1202)

```
/**
 * Steps that model Messages arriving from a Subscription.
 *  Destructure as `const { Subscription } = Scene` for concise call sites.
 */
const Subscription: {
  emit: (message: MessageInput) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
}
```

### all

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L149)

```
/**
 * Multi-match Locator factories. Each returns a `LocatorAll` that resolves
 *  to every matching VNode. Convert to a single `Locator` via `first`,
 *  `last`, or `nth(n)`, or narrow via `filter`.
 */
const all: {
  altText: (altValue: string) => LocatorAll
  displayValue: (valueString: string) => LocatorAll
  label: (labelValue: string) => LocatorAll
  placeholder: (placeholderValue: string) => LocatorAll
  role: (roleValue: string, options?: Readonly<{
    checked: boolean | "mixed"
    disabled: boolean
    expanded: boolean
    level: number
    name: string | RegExp
    pressed: boolean | "mixed"
    selected: boolean
  }>) => LocatorAll
  selector: (css: string) => LocatorAll
  testId: (testIdValue: string) => LocatorAll
  text: (target: string, options?: Readonly<{
    exact: boolean
  }>) => LocatorAll
  title: (titleValue: string) => LocatorAll
}
```

### attr

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L798)

```
/** Reads an attribute or prop value from a VNode. */
const attr: (vnode: VNode, name: string) => Option<string>
```

### change

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1810)

```
/**
 * Simulates a change event on the element matching the target.
 *  Dual: `change(target, value)` or `change(value)` for data-last piping.
 */
const change: (target: string | Locator, value: string) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### changeFiles

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1839)

```
/**
 * Simulates a file input change event on the element matching the target.
 *  For use with `OnFileChange` attributes. The handler receives a synthetic
 *  event with `target.files` set to the provided files array.
 *  Dual: `changeFiles(target, files)` or `changeFiles(files)` for data-last piping.
 */
const changeFiles: (target: string | Locator, files: readonly Array<File>) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### dropFiles

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1870)

```
/**
 * Simulates a drop event with files on the element matching the target.
 *  For use with `OnDropFiles` attributes. The handler receives a synthetic
 *  event with `dataTransfer.files` set to the provided files array and a
 *  no-op `preventDefault`.
 *  Dual: `dropFiles(target, files)` or `dropFiles(files)` for data-last piping.
 */
const dropFiles: (target: string | Locator, files: readonly Array<File>) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### filter

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1383)

```
/**
 * Filters a LocatorAll's matches. Supports `has`/`hasNot` to keep entries
 *  that do/don't contain a matching descendant, and `hasText`/`hasNotText`
 *  to keep entries whose text content does/doesn't include a substring.
 */
const filter: (locatorAll: LocatorAll, options: FilterOptions) => LocatorAll
```

### find

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L740)

```
/** Finds the first VNode matching the CSS selector. */
const find: (html: VNode, selectorString: string) => Option<VNode>
```

### findAll

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L750)

```
/** Finds all VNodes matching the CSS selector. */
const findAll: (html: VNode, selectorString: string) => readonly Array<VNode>
```

### keydown

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1939)

```
/**
 * Simulates a keydown event on the element matching the target.
 *  Dual: `keydown(target, key, modifiers?)` or `keydown(key, modifiers?)` for data-last piping.
 */
const keydown: (target: string | Locator, key: string) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### nth

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1350)

```
/** Picks the nth match (0-indexed) from a LocatorAll, producing a Locator. */
const nth: (locatorAll: LocatorAll, index: number) => Locator
```

### scene

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L2403)

```
/** Executes a scene test. Throws if any Commands remain unresolved. */
const scene: (config: Readonly<{
  update: (model: Model, message: Message) => readonly [Model, ReadonlyArray<AnyCommand>, OutMessage]
  view: (model: Model, h: HtmlBuilder<Message>) => Html | Document
}>, steps: readonly Array<SceneStep<Model, Message, OutMessage>>) => void
```

### sceneMatchers

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/matchers.ts#L18)

```
/** Custom Vitest matchers for scene testing. Register with `expect.extend(Scene.sceneMatchers)`. */
const sceneMatchers: {
  toBeAbsent: unknown
  toBeChecked: unknown
  toBeDisabled: unknown
  toBeEmpty: unknown
  toBeEnabled: unknown
  toBeVisible: unknown
  toContainText: unknown
  toExist: unknown
  toHaveAttr: unknown
  toHaveClass: unknown
  toHaveHandler: unknown
  toHaveHook: unknown
  toHaveId: unknown
  toHaveStyle: unknown
  toHaveText: unknown
  toHaveValue: unknown
}
```

### type

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/scene.ts#L1912)

```
/**
 * Simulates typing a value into the input matching the target.
 *  Dual: `type(target, value)` or `type(value)` for data-last piping.
 */
const type: (target: string | Locator, value: string) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### within

const

[source](https://github.com/foldkit/foldkit/blob/5926900b5d5136e2f6ba527737386fc6b4619da7/packages/foldkit/src/test/query.ts#L1269)

```
/**
 * Creates a scoped Locator that finds the child within the parent.
 *  Composes via `Option.flatMap` — the parent is resolved first, then
 *  the child is searched within the parent's subtree.
 */
const within: (parent: Locator, child: Locator) => Locator
```
