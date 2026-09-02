---
url: https://foldkit.dev/api-reference/scene
title: "Scene"
description: "API documentation for the Scene module."
access_date: 2026-09-02T16:31:11.406Z
current_date: 2026-09-02T16:31:11.406Z
---

# Scene

## Functions

### altText

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1236)

```
/** Creates a Locator that finds an element by its `alt` attribute. */
(altValue: string): Locator
```

### blur

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2291)

```
/** Simulates a blur event on the element matching the target. */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### click

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1877)

```
/**
 * Simulates a click on the element matching the target.
 *  Runs click handlers from the target through its ancestors until one stops
 *  propagation, mirroring browser event propagation.
 *  When the target activates a submit button and no click handler prevents
 *  the default, the click falls through to the `submit` handler of its form
 *  owner. Scene honors an explicit `form` attribute before looking for the
 *  nearest ancestor `<form>`.
 */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### contextMenu

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2037)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1248)

```
/** Creates a Locator that finds a form control by its current `value`. */
(valueString: string): Locator
```

### doubleClick

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1975)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2822)

### expectAll

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2878)

### expectHandled

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1646)

```
/**
 * Asserts that the preceding interaction was handled: its event handler
 *  produced a Message.
 * 
 *  This is the assertion behind "the key is consumed here". A Foldkit
 *  handler that returns a Message is what makes `h.OnKeyDownPreventDefault`
 *  call `preventDefault()`, so a handled keydown is one whose browser
 *  default is suppressed: `Space` does not scroll the page and `Enter` does
 *  not submit a surrounding form.
 * 
 *  Reach for this rather than asserting the Message's tag. The tag is the
 *  mechanism a component happens to use; being consumed is the contract, and
 *  it survives renaming the Message.
 * 
 *  An interaction on an element with no handler at all throws from the
 *  interaction step itself, so this distinguishes the narrower case of a
 *  handler that ran and chose to produce nothing.
 * 
 *  Only interaction steps set the outcome. `Command.resolve`, `Mount.resolve`,
 *  and plain `expect` leave it alone, so the value is the last *interaction*
 *  rather than the last step. Keep the assertion next to the interaction it
 *  covers.
 * 
 *  Where the event should have been consumed but was not, this is the
 *  assertion that says so, and it fails until the handler is fixed. Where
 *  falling through is intended, reach for expectIgnored instead,
 *  which a fall-through requires.
 */
(): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### expectIgnored

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1687)

```
/**
 * Asserts that the preceding interaction was ignored: its event handler ran
 *  and produced no Message, so the event falls through and the browser
 *  default stands.
 * 
 *  Required after any interaction that falls through, and it acknowledges
 *  that fall-through as intended. Saying nothing is not an available
 *  position: Scene fails at the next interaction, or at the end of the
 *  scene, on a fall-through nothing acknowledged, because a test that leaves
 *  it unsaid passes whether the interaction is correctly inert or its
 *  handler regressed.
 * 
 *  One acknowledgement covers one fall-through. Two in a row need one each,
 *  and each must come before the next interaction.
 * 
 *  Carries the same adjacency caveat as expectHandled: only
 *  interaction steps set the outcome, so keep the assertion next to the
 *  interaction it covers.
 */
(): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### expectNoOutMessage

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1599)

```
/** Asserts that the latest update-producing Scene step emitted no OutMessages. */
(): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### expectOutMessage

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1541)

```
/**
 * Asserts by structural equality that the latest update-producing Scene step
 *  emitted exactly the expected OutMessage.
 */
<OutMessage>(expected: OutMessage): OutMessageStep<OutMessage>
```

### expectOutMessages

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1576)

```
/**
 * Asserts by structural equality that the latest update-producing Scene
 *  step emitted two or more expected OutMessages in runtime order.
 */
<Expected extends readonly [unknown, unknown, unknown]>(expected: Expected): OutMessagesStep<Expected[number]>
```

### first

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1333)

```
/** Picks the first match from a LocatorAll, producing a single-match Locator. */
(locatorAll: LocatorAll): Locator
```

### focus

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2281)

```
/** Simulates a focus event on the element matching the target. */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### focusEnter

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2301)

```
/** Simulates focus entering the subtree of the element matching the target. */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### focusLeave

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2311)

```
/** Simulates focus leaving the subtree of the element matching the target. */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### getAllByAltText

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1134)

```
/** Finds all elements with the given `alt` attribute. */
(altValue: string): (html: VNode) => readonly Array<VNode>
```

### getAllByDisplayValue

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1152)

```
/** Finds all form controls whose current value matches. */
(displayValueString: string): (html: VNode) => readonly Array<VNode>
```

### getAllByLabel

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1048)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1125)

```
/** Finds all elements with the given placeholder attribute. */
(placeholderValue: string): (html: VNode) => readonly Array<VNode>
```

### getAllByRole

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L952)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1146)

```
/** Finds all elements with the given `data-testid` attribute. */
(testIdValue: string): (html: VNode) => readonly Array<VNode>
```

### getAllByText

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1111)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1140)

```
/** Finds all elements with the given `title` attribute. */
(titleValue: string): (html: VNode) => readonly Array<VNode>
```

### getByAltText

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1089)

```
/** Finds the first element with the given `alt` attribute. */
(altValue: string): (html: VNode) => Option<VNode>
```

### getByDisplayValue

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1164)

```
/**
 * Finds the first form control whose current value matches. Checks the `value`
 *  attribute on inputs, textareas, and selects.
 */
(displayValue: string): (html: VNode) => Option<VNode>
```

### getByLabel

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1004)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L993)

```
/** Finds the first element with the given placeholder attribute. */
(placeholderValue: string): (html: VNode) => Option<VNode>
```

### getByRole

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L938)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1101)

```
/** Finds the first element with the given `data-testid` attribute. */
(testIdValue: string): (html: VNode) => Option<VNode>
```

### getByText

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L967)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1095)

```
/** Finds the first element with the given `title` attribute. */
(titleValue: string): (html: VNode) => Option<VNode>
```

### given

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L941)

```
/** Sets the initial Model for a scene test. */
<Model>(model: Model): GivenStep<Model>
```

### hover

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2262)

```
/**
 * Simulates a hover (mouseenter) on the element matching the target.
 *  Dispatches the `mouseenter` handler, falling back to `mouseover`.
 */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### inside

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1801)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1232)

```
/** Creates a Locator that finds an element by aria-label. */
(labelValue: string): Locator
```

### last

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1340)

```
/** Picks the last match from a LocatorAll, producing a single-match Locator. */
(locatorAll: LocatorAll): Locator
```

### placeholder

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1225)

```
/** Creates a Locator that finds an element by placeholder attribute. */
(placeholderValue: string): Locator
```

### pointerDown

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2116)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2200)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1216)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1259)

```
/**
 * Creates a Locator that wraps a CSS selector. Escape hatch for cases
 *  where no accessible attribute is available.
 */
(css: string): Locator
```

### submit

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2412)

```
/** Simulates form submission on the element matching the target. */
(target: string | Locator): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### tap

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1711)

```
/** Runs a function for side effects (e.g. assertions) without breaking the step chain. */
<Model, Message, OutMessage = undefined>(f: (simulation: SceneSimulation<Model, Message, OutMessage>) => void): (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### testId

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1244)

```
/** Creates a Locator that finds an element by its `data-testid` attribute. */
(testIdValue: string): Locator
```

### text

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1252)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L754)

```
/** Extracts all text content from a VNode tree, depth-first. */
(vnode: VNode): string
```

### title

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1240)

```
/** Creates a Locator that finds an element by its `title` attribute. */
(titleValue: string): Locator
```

### withViewInputs

function

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2901)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/internal.ts#L28)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/internal.ts#L133)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1179)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1185)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/internal.ts#L146)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/internal.ts#L203)

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

### OutMessageStep

type

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L192)

```
/** A typed OutMessage assertion step. */
type OutMessageStep = Readonly<{
  _tag: "OutMessageStep"
  expected: OutMessage
}>
```

### OutMessagesStep

type

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L198)

```
/** A typed OutMessage sequence assertion step. */
type OutMessagesStep = Readonly<{
  _tag: "OutMessagesStep"
  expected: readonly [OutMessage, OutMessage, ...ReadonlyArray<OutMessage>]
}>
```

### PendingMount

type

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/internal.ts#L123)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/internal.ts#L194)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L168)

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
  outMessage: OutMessage | undefined
}>
```

### SceneStep

type

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L205)

```
/**
 * A single step in a scene: a `given` step, typed Message or OutMessage step,
 *  or scene simulation transform.
 */
type SceneStep = GivenStep<NoInfer<Model>> | Readonly<{
  _tag: "SubscriptionMessageStep"
  message: NoInfer<Message>
}> | Readonly<{
  _tag: "OutMessageStep"
  expected: NoInfer<OutMessage>
}> | Readonly<{
  _tag: "OutMessagesStep"
  expected: readonly [NoInfer<OutMessage>, NoInfer<OutMessage>, ...ReadonlyArray<NoInfer<OutMessage>>]
}> | (simulation: SceneSimulation<any, any, any>) => SceneSimulation<any, any, any>
```

### SubscriptionMessageStep

type

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L186)

```
/** A typed Subscription Message step. */
type SubscriptionMessageStep = Readonly<{
  _tag: "SubscriptionMessageStep"
  message: Message
}>
```

## Constants

### Command

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1463)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1533)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1518)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1486)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L1504)

```
/**
 * Steps that model Messages arriving from a Subscription.
 *  Destructure as `const { Subscription } = Scene` for concise call sites.
 */
const Subscription: {
  emit: (message: Message) => SubscriptionMessageStep<Message>
}
```

### all

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L152)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L794)

```
/** Reads an attribute or prop value from a VNode. */
const attr: (vnode: VNode, name: string) => Option<string>
```

### change

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2322)

```
/**
 * Simulates a change event on the element matching the target.
 *  Dual: `change(target, value)` or `change(value)` for data-last piping.
 */
const change: (target: string | Locator, value: string) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### changeFiles

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2351)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2382)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1375)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L738)

```
/** Finds the first VNode matching the CSS selector. */
const find: (html: VNode, selectorString: string) => Option<VNode>
```

### findAll

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L746)

```
/** Finds all VNodes matching the CSS selector. */
const findAll: (html: VNode, selectorString: string) => readonly Array<VNode>
```

### keydown

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2451)

```
/**
 * Simulates a keydown event on the element matching the target.
 *  Dual: `keydown(target, key, modifiers?)` or `keydown(key, modifiers?)` for data-last piping.
 */
const keydown: (target: string | Locator, key: string) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### nth

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1344)

```
/** Picks the nth match (0-indexed) from a LocatorAll, producing a Locator. */
const nth: (locatorAll: LocatorAll, index: number) => Locator
```

### scene

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2919)

```
/**
 * Executes a scene test. Throws if any Commands or Mounts remain
 *  unresolved, any unmount is unacknowledged, or any interaction fell
 *  through unacknowledged.
 */
const scene: (config: Readonly<{
  update: (model: Model, message: Message) => Readonly<{
    commands: ReadonlyArray<AnyCommand>
    model: Model
    outMessage: OutMessage
  }>
  view: (model: Model, h: HtmlBuilder<Message>) => Html | Document
}>, steps: readonly Array<SceneStep<NoInfer<Model>, NoInfer<Message>, NoInfer<OutMessage>>>) => void
```

### sceneMatchers

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/matchers.ts#L19)

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

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/scene.ts#L2424)

```
/**
 * Simulates typing a value into the input matching the target.
 *  Dual: `type(target, value)` or `type(value)` for data-last piping.
 */
const type: (target: string | Locator, value: string) => (simulation: SceneSimulation<Model, Message, OutMessage>) => SceneSimulation<Model, Message, OutMessage>
```

### within

const

[source](https://github.com/foldkit/foldkit/blob/009aa88d012fa1ac31d90ec469acba8af67c4249/packages/foldkit/src/test/query.ts#L1265)

```
/**
 * Creates a scoped Locator that finds the child within the parent.
 *  Composes via `Option.flatMap` — the parent is resolved first, then
 *  the child is searched within the parent's subtree.
 */
const within: (parent: Locator, child: Locator) => Locator
```
