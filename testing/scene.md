---
url: https://foldkit.dev/testing/scene
title: "Scene"
description: "Drive the rendered VNode tree with accessible locators, dispatch interactions, resolve lifecycle results, and assert on the resulting HTML."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

## Testing Through the View

`Scene` runs update and view together. It clicks buttons, types into inputs, presses keys, and checks the rendered VNode tree. The view runs after every step, so the test sees both state transitions and their rendered result.

Scene operates on VNodes directly. It needs no DOM, jsdom, or browser.

Import the steps you need from `foldkit/scene`. Use named imports when the file contains only Scene tests. If a file contains both Story and Scene tests, import the namespaces from `foldkit` so `Story.given` and `Scene.given` stay distinct.

## Locators

Locators find elements by role, label, visible text, and other user-facing properties. A `Locator` resolves to one match. Interactions and assertions also accept a raw CSS selector when no accessible query fits.

```
import {
  altText,
  displayValue,
  label,
  placeholder,
  role,
  selector,
  testId,
  text,
  title,
} from 'foldkit/scene'

role('button', { name: 'Submit' })
label('Email')
text('Welcome back')
placeholder('Search...')
altText('Company logo')
title('Close dialog')
testId('cart-summary')
displayValue('alice@example.com')
selector('.fallback-class')
```

| Locator | Finds | Example |
| --- | --- | --- |
| `role(role, options?)` | Elements by ARIA role (explicit or implicit). Options narrow by accessible name and ARIA state. | `role('button', { name: 'Save' })` |
| `label(text)` | Form controls by their aria-label or associated <label> text. | `label('Email')` |
| `placeholder(text)` | Inputs by their placeholder attribute. | `placeholder('Search...')` |
| `text(text)` | Elements by visible text content. | `text('Welcome back')` |
| `altText(text)` | Images and similar elements by their alt attribute. | `altText('Profile photo')` |
| `title(text)` | Elements by their title attribute (tooltip text). | `title('Delete')` |
| `testId(id)` | Elements by data-testid: the escape hatch for tests. | `testId('cart-item-3')` |
| `displayValue(value)` | Form controls by their current value. | `displayValue('US')` |
| `selector(css)` | Elements by CSS selector. Use when no accessible query fits. | `selector('.chart-legend')` |

### The role Locator

`role` is the usual starting point. Its optional second argument narrows the match by accessible name or ARIA state.

```
import { role } from 'foldkit/scene'

// Match by role alone
role('button')

// Narrow by accessible name (exact match)
role('button', { name: 'Save' })

// Narrow by accessible name (regex match)
role('option', { name: /PM/ })

// Narrow by heading level
role('heading', { level: 2 })

// Narrow by ARIA state
role('checkbox', { checked: true })
role('button', { pressed: true, disabled: false })
```

| Option | Type | Matches |
| --- | --- | --- |
| `name` | `string \| RegExp` | Accessible name (aria-label, aria-labelledby, label\[for\], or text content). Strings match exactly; regular expressions match against the full name. |
| `level` | `number` | Heading level (for role: "heading") |
| `checked` | `boolean \| 'mixed'` | aria-checked or the checked attribute |
| `selected` | `boolean` | aria-selected |
| `pressed` | `boolean \| 'mixed'` | aria-pressed |
| `expanded` | `boolean` | aria-expanded |
| `disabled` | `boolean` | aria-disabled or the disabled attribute |

### Scoping

`within(parent, child)` scopes a single locator to a parent element. `inside(parent, ...steps)` scopes a whole block of steps. Every assertion or interaction inside the block resolves within the parent’s subtree. Use `within` for one-off scoped queries; use `inside` when several steps share the same scope. Nested `inside` calls compose.

```
import { click, expect, inside, role, within } from 'foldkit/scene'

// Scope a single locator to a parent element.
within(role('region', { name: 'Sidebar' }), role('link'))

// Scope a block of steps — every assertion and interaction
// resolves within the parent's subtree.
inside(
  role('dialog', { name: 'Confirm' }),
  expect(role('heading')).toHaveText('Delete item?'),
  click(role('button', { name: 'Cancel' })),
)
```

### Multi-Match

For lists and repeated elements, the `all.*` factories return every match. Pick one with `first`, `last`, or `nth(index)`, or narrow the set with `filter`.

```
import { pipe } from 'effect'
import { all, filter, first, last, nth, role } from 'foldkit/scene'

// Multi-match locators return every match.
all.role('row')
all.text('Delete')
all.label('Email')

// Pick one element from the set.
first(all.role('row'))
last(all.role('button', { name: 'Delete' }))
nth(all.role('row'), 2)

// Narrow with filter, then pick.
pipe(all.role('row'), filter({ hasText: 'Alice' }), first)

pipe(
  all.role('row'),
  filter({ has: role('button', { name: 'Delete' }) }),
  first,
)
```

| Filter option | Keeps matches where |
| --- | --- |
| `has` | The element contains a descendant matching the given Locator |
| `hasNot` | The element does not contain a descendant matching the Locator |
| `hasText` | The element’s text content includes the given substring |
| `hasNotText` | The element’s text content does not include the substring |

## Interactions

An interaction invokes the matched element's event handler. If the handler produces a Message, Scene feeds it through update and renders the next view.

```
import {
  blur,
  change,
  click,
  contextMenu,
  doubleClick,
  focus,
  hover,
  keydown,
  label,
  pointerDown,
  pointerUp,
  role,
  submit,
  type,
} from 'foldkit/scene'

click(role('button', { name: 'Log out' }))
doubleClick(role('button', { name: 'Expand' }))
contextMenu(role('row', { name: 'Quarterly report' }))
pointerDown(role('button', { name: 'Toggle' }))
pointerUp(role('button', { name: 'Toggle' }))
hover(role('menuitem', { name: 'File' }))
focus(label('Email'))
blur(label('Email'))
type(label('Email'), 'alice@example.com')
change(label('Country'), 'US')
submit(role('form'))
keydown(label('Search'), 'Enter')
```

| Step | Invokes |
| --- | --- |
| `click(target)` | `OnClick` (bubbles to ancestors) |
| `doubleClick(target)` | `OnDoubleClick` (bubbles to ancestors) |
| `contextMenu(target)` | `OnContextMenu` (bubbles to ancestors) |
| `pointerDown(target, options?)` | `OnPointerDown` with optional `{ pointerType, button, screenX, screenY }` (bubbles to ancestors) |
| `pointerUp(target, options?)` | `OnPointerUp` with optional `{ pointerType, screenX, screenY }` (bubbles to ancestors) |
| `hover(target)` | `OnMouseEnter` (falls back to `OnMouseOver`) |
| `focus(target)` | `OnFocus` |
| `blur(target)` | `OnBlur` |
| `type(target, text)` | `OnInput` with the given text |
| `change(target, value)` | `OnChange` with the given value, for `<select>` and similar |
| `keydown(target, key, modifiers?)` | `OnKeyDown` or `OnKeyDownPreventDefault` with optional `{ shiftKey, ctrlKey, altKey, metaKey }` |
| `submit(target)` | `OnSubmit` |

`tap(fn)` runs a function for side effects (like ad-hoc assertions on raw VNodes or accumulated Commands) without breaking the step chain.

## Assertions

`expect(locator)` creates an inline assertion step against a single element. Every matcher has a `.not` variant that inverts the assertion.

Property, state, and accessibility matchers require the Locator to match an element, including their `.not` variants. Use `.toBeAbsent()` or `.not.toExist()` when the intended assertion is that no element matches.

```
import { all, expect, expectAll, label, role } from 'foldkit/scene'

// Single-element assertions
expect(role('heading')).toExist()
expect(role('heading')).toHaveText('Welcome')
expect(role('heading')).toHaveText(/^Welcome/)
expect(role('heading')).toContainText('Welcome')
expect(role('dialog')).toBeAbsent()
expect(role('status')).toBeVisible()
expect(role('status')).toBeEmpty()
expect(role('region')).toHaveAccessibleName('User session')
expect(label('Email')).toHaveValue('alice@example.com')
expect(role('button', { name: 'Submit' })).toBeDisabled()
expect(role('button')).not.toBeDisabled()

// Multi-match assertions — count-based
expectAll(all.role('row')).toHaveCount(3)
expectAll(all.role('alert')).toBeEmpty()
```

| Matcher | Asserts that the element |
| --- | --- |
| `.toExist()` | Is present in the tree |
| `.toBeAbsent()` | Is not present in the tree |
| `.toBeVisible()` | Is not hidden via the hidden attribute, aria-hidden, display: none, or visibility: hidden |
| `.toBeEmpty()` | Has no text content or child elements |
| `.toHaveText(value)` | Has text content equal to the given string or matching the given regex |
| `.toContainText(value)` | Has text content including the given substring or matching the regex |
| `.toHaveAccessibleName(name)` | Has the given accessible name (resolves aria-labelledby, aria-label, label\[for\], native host-language sources, accessible text content, and title) |
| `.toHaveAccessibleDescription(description)` | Has the given accessible description (resolves aria-describedby) |
| `.toBeDisabled()` | Has aria-disabled or the disabled attribute |
| `.toBeEnabled()` | Is not disabled |
| `.toBeChecked()` | Has aria-checked="true" or the checked attribute |
| `.toHaveValue(value)` | Has the given current form-control value |
| `.toHaveAttr(name, value)` | Has the given attribute set to the given value |
| `.toHaveId(id)` | Has the given id |
| `.toHaveClass(name)` | Has the given CSS class |
| `.toHaveStyle(name, value)` | Has the given inline style property |

Accessible-name and accessible-description matching excludes hidden descendant content. A hidden node directly referenced by `aria-labelledby` or `aria-describedby` contributes its full subtree text.

For `LocatorAll` (from `all.*`), use `expectAll(locatorAll)` for count-based assertions:

| Matcher | Asserts that |
| --- | --- |
| `.toHaveCount(n)` | The locator matches exactly n elements |
| `.toBeEmpty()` | The locator matches zero elements |

## Handled and Ignored Interactions

Scene makes every fall-through interaction explicit. When a handler runs and returns `Option.none()`, follow the interaction with `expectIgnored()`. When it produces a Message, `expectHandled()` verifies that outcome. If another interaction starts or the scene ends before a fall-through is acknowledged, Scene fails and names the event and target. Each acknowledgement covers one interaction.

An element with no handler for that event throws immediately. That is different from a handler that deliberately returns `Option.none()`. For example: a read-only widget may keep its input handler but ignore edits. `expectIgnored()` proves the handler is still present and deliberately inert. `Command.expectNone()` or `expectNoOutMessage()` cannot distinguish that from a deleted handler.

Use `expectHandled()` to prove that `h.OnKeyDownPreventDefault` consumed a key. A handled `Space` does not scroll the page, and a handled `Enter` does not submit a surrounding form. This checks the user-facing contract without depending on the Message name.

The outcome belongs to the most recent interaction. `Command.resolve`, `Mount.resolve`, and a plain `expect` do not replace it. Keep `expectHandled()` or `expectIgnored()` next to the interaction it covers.

## Commands

When update returns Commands, Scene keeps them pending until the test supplies their result Messages. Update declares the work. The test declares what happened.

- Pending Commands accumulate in the order `update` returns them, across as many steps as the test takes.
- Resolving a Command feeds its result Message through `update`; new Commands produced by that update join the pending list.
- `Command.resolveAll` walks cascades within the batch. If resolving Command A produces Command B and B’s resolver is in the same call, B resolves without a separate step.
- `Command.resolveAllExact` walks the same cascades while requiring every listed resolver to match within that call and every actual Command to be resolved.
- Interactions throw if there are unresolved Commands when they try to dispatch a Message.
- `scene` throws at the end if any Command remains unresolved.

```
import { Command, click, role } from 'foldkit/scene'

// Single Command. Click a button, acknowledge its Command result.
click(role('button', { name: 'Get Weather' }))
Command.expectExact(FetchWeather)
Command.resolve(FetchWeather, SucceededFetchWeather({ weather }))

// Lock in args. Pass a Command instance instead of a Definition to match by
// name AND args. Catches regressions where the Command fires with wrong inputs.
Command.expectExact(FetchWeather({ zipCode: '90210' }))

// Multiple Commands. Resolve a batch in one step; cascading Commands resolve too.
click(role('button', { name: 'Sign In' }))
Command.expectExact(RequestAuthentication, TrackSignInAttempt)
Command.resolveAll(
  [RequestAuthentication, SucceededRequestAuthentication({ session })],
  [TrackSignInAttempt, CompletedTrackSignInAttempt()],
)

// Resolve the batch while asserting every listed Command was dispatched.
click(role('button', { name: 'Sign In' }))
Command.resolveAllExact(
  [RequestAuthentication, SucceededRequestAuthentication({ session })],
  [TrackSignInAttempt, CompletedTrackSignInAttempt()],
)

// Subset assertion. Use when you only care that a particular Command is pending.
// Definition or instance: instance form locks in the args.
Command.expectHas(FetchWeather)
Command.expectHas(FetchWeather({ zipCode: '90210' }))

// Negative assertion. Useful before a transition that should produce no Commands.
Command.expectNone()

// Submodel Command. When the Command lives in a child component, resolve it
// with the child's raw result Message. resolve replays the Command's own
// mapMessages wrapping automatically, so you never restate the lift.
Command.resolve(
  Search.FetchSuggestions,
  Search.SucceededFetchSuggestions({ suggestions }),
)
```

| Step | Effect |
| --- | --- |
| `Command.resolve(Def, ResultMessage)` | Resolves the first pending Command with the given name by feeding `ResultMessage` through update. For a child Submodel Command, pass the child’s raw result Message; resolve replays the Command’s own `mapMessages` wrapping automatically. |
| `Command.resolveAll([Def, ResultMessage], ...)` | Resolves a batch of pending Commands, walking cascades. Each entry resolves exactly one matching dispatch in declaration order and unmatched entries carry forward; compose with Array.makeBy for N identical responses. |
| `Command.resolveAllExact([Def, ResultMessage], ...)` | Resolves a batch and throws unless every listed resolver matches within the call and no actual Commands remain unresolved. Repeated Definition entries consume repeated dispatches in declaration order. |
| `Command.expectExact(A, B)` | The pending Commands are exactly A and B (order-independent). |
| `Command.expectHas(A)` | A is among the pending Commands (subset check). |
| `Command.expectNone()` | There are no pending Commands. |

Prefer `Command.expectExact` as the default. It catches bugs where an interaction produces unexpected Commands. Use `Command.expectHas` when you only care about a subset of the pending Commands.

Each matcher accepts either a Command Definition (matches by name) or a Command instance (matches by name AND structural-equal args). Pass a Definition when the test only cares that the Command was dispatched; pass an instance when the args are part of what the test is verifying. `Command.expectExact(FetchWeather({ zipCode: '90210' }))` fails if the runtime dispatched `FetchWeather({ zipCode: '99999' })`, where the same call with just `FetchWeather` would pass.

## Mounts

When a rendered view contains an `OnMount` attribute, Scene keeps that Mount pending until the test supplies its result Message. If the mounted element later leaves the tree, the test must also acknowledge the unmount with `Mount.expectEnded`.

This applies to Mounts declared inside `@foldkit/ui` components too. Popovers, dialogs, and other components export their Mount Definitions so a consumer test can name and resolve them.

- Pending mounts persist across re-renders. Resolving a mount does not re-pend it on the next render.
- Every mount that fires and unmounts during a scene must be acknowledged with `Mount.expectEnded`, even if it was already resolved. `resolve` handles a mount’s result Message; `expectEnded` handles its unmount. Unacknowledged unmounts throw at the end of the scene.
- Same-named mounts in the tree are disambiguated by occurrence. `Mount.resolve` resolves the first pending occurrence; a second call resolves the next.
- Interactions throw if there are unresolved mounts or unacknowledged unmounts when they try to dispatch a Message. Same contract as Commands.
- `scene` throws at the end if any mount remains unresolved.

```
import { Mount, click, role } from 'foldkit/scene'

import { Listbox, Popover } from '@foldkit/ui'

// Single Mount. Open a popover, acknowledge its anchor mount.
click(role('button', { name: 'Open' }))
Mount.expectExact(Popover.AnchorPopover)
Mount.resolve(Popover.AnchorPopover, Popover.CompletedAnchorPopover())

// Multiple Mounts. Opening a modal Listbox renders both the items container
// (positioning) and a backdrop (portaled to body), so two Mounts fire.
click(role('button', { name: 'Pick a fruit' }))
Mount.expectExact(Listbox.AnchorListbox, Listbox.PortalListboxBackdrop)
Mount.resolveAll(
  [Listbox.AnchorListbox, Listbox.CompletedAnchorListbox()],
  [Listbox.PortalListboxBackdrop, Listbox.CompletedPortalListboxBackdrop()],
)

// Subset assertion. Use when you only care that a particular mount is pending.
Mount.expectHas(Listbox.AnchorListbox)

// Negative assertion. Useful before a transition that should produce no mounts.
Mount.expectNone()

// Acknowledge an unmount. Required for every Mount that fires and then
// unmounts during the scene, regardless of whether it was resolved first.
// The scene throws at the end for any unacknowledged unmount.
Mount.expectEnded(Popover.AnchorPopover)

// When the mount lives inside a child Submodel, resolve replays the
// Submodel boundary's own lift, so you pass the child's raw result Message.
Mount.resolve(Popover.AnchorPopover, Popover.CompletedAnchorPopover())
```

| Step | Effect |
| --- | --- |
| `Mount.resolve(Def, ResultMessage)` | Resolves the first pending mount with the given name by feeding `ResultMessage` through update. For a mount inside a child Submodel, resolve replays the boundary lift automatically, so you pass the child’s raw result Message. |
| `Mount.resolveAll([Def, ResultMessage], ...)` | Resolves a batch of pending mounts in order. |
| `Mount.expectExact(A, B)` | The pending mounts are exactly A and B (order-independent, by name). |
| `Mount.expectHas(A)` | A is among the pending mounts (subset check). |
| `Mount.expectNone()` | There are no pending mounts. |
| `Mount.expectEnded(A)` | A has disappeared from the rendered tree. Required for every Mount that fires and then unmounts during the scene, regardless of whether it was resolved first; otherwise the scene throws at the end. |

UI components export their Mount definitions (`Popover.AnchorPopover`, `Listbox.AnchorListbox`, and so on) so consumer tests can name them in `Mount.resolve`.

## Subscriptions

A Subscription Message has no element to interact with. `Subscription.emit(message)` feeds a timer tick, WebSocket frame, global listener result, or other Subscription output through update and renders the next view.

```
import { Subscription, click, role } from 'foldkit/scene'

// A Message whose real cause is a Subscription has no element in the
// rendered tree. Emit it directly; the step feeds it through update and
// re-renders like any other step.
Subscription.emit(Ticked())
Subscription.emit(ReceivedServerFrame({ frame }))

// Don't emit a Message that has a DOM affordance. Click the actual
// button instead; the interaction proves the handler wiring that emit skips.
click(role('button', { name: 'Log out' }))
```

Use it only when the Message's cause lives outside the rendered tree. If the Message comes from a button, click the button. That interaction verifies the handler wiring that `emit` would skip. `emit` throws while Commands, Mounts, or unacknowledged unmounts are pending.

## Managed Resources

A [ManagedResource](https://foldkit.dev/core/managed-resources) reports three lifecycle outcomes: acquisition succeeded, acquisition failed, or release completed. The `ManagedResource` steps feed the corresponding hook Message through update.

Drive the Model into the matching state before declaring an outcome. `acquire` and `failAcquire` throw unless `modelToMaybeRequirements` returns `Some`. `release` throws until it returns `None`.

These steps dispatch immediately and leave nothing pending. They are optional because Scene sees update and view, not the application's ManagedResources record. A scene may end while the Model still requests a resource, just as the runtime can wait for its acquire Effect to finish. If a lifecycle step contradicts the Model, it throws immediately.

Pass `acquire` exactly the arguments that `onAcquired` accepts. If `onAcquired` reads the acquired value, the test supplies it. If the hook ignores the value, the test supplies nothing.

```
import { ManagedResource, click, expect, role } from 'foldkit/scene'

// The test declares the lifecycle outcome, the way Command.resolve declares
// a Command result. acquire takes exactly the arguments the entry's
// onAcquired declares (here the acquired socket value; none for a handler
// that ignores it) and requires the current Model to request the resource,
// so drive that transition through real steps first.
click(role('button', { name: 'Open feed' }))
ManagedResource.acquire(resources.feedSocket, { socketId: 'sock-1' })
expect(role('status')).toHaveText('Connected')

// release requires the Model to no longer request the resource, mirroring
// the runtime's Some to None transition.
click(role('button', { name: 'Close feed' }))
ManagedResource.release(resources.feedSocket)

// failAcquire feeds onAcquireError through update. It runs under the same
// gate as acquire: the Model must request the resource.
click(role('button', { name: 'Open feed' }))
ManagedResource.failAcquire(resources.feedSocket, new Error('offline'))
```

| Step | Effect |
| --- | --- |
| `ManagedResource.acquire(entry, ...args)` | Feeds `onAcquired(...args)` through update. The Model must request the resource. |
| `ManagedResource.failAcquire(entry, error)` | Feeds `onAcquireError(error)` through update. The Model must request the resource. |
| `ManagedResource.release(entry)` | Feeds `onReleased()` through update. The Model must no longer request the resource. |

Scene does not model a `Some` to structurally different `Some` reacquisition. At runtime, that transition releases and reacquires the resource while the Model continues to request it. There is no Scene step for that transition yet.

## Custom Elements

A [CustomElement](https://foldkit.dev/core/custom-element) maps declared CustomEvents to Messages through its `On*` attributes. `CustomElement.emit(spec, target, eventName, detail)` dispatches one of those events on a rendered element. The spec's event Schemas type the event name and detail.

The target must be in the rendered tree with that event handler attached. A missing element or handler throws.

```
import { Schema as S } from 'effect'
import { CustomElement, Scene } from 'foldkit'

const hexColorPicker = CustomElement.define({
  tag: 'hex-color-picker',
  properties: {
    color: S.String,
  },
  events: {
    'color-changed': S.Struct({ value: S.String }),
  },
})

// Dispatches a CustomEvent the element's spec declares. The event name and
// detail are typed by the spec's event Schemas, and the element's
// OnColorChanged mapping converts the detail into a Message.
Scene.CustomElement.emit(
  hexColorPicker,
  Scene.selector('hex-color-picker'),
  'color-changed',
  { value: '#ff0000' },
)
Scene.expect(Scene.role('status')).toHaveText('#ff0000')
```

## OutMessages

When the update under test returns a Submodel's three-tuple, Scene tracks its `Option<OutMessage>`. `expectOutMessage(expected)` asserts `Some(expected)`. `expectNoOutMessage()` asserts `None`.

```
import {
  Subscription,
  click,
  expectNoOutMessage,
  expectOutMessage,
  given,
  role,
  scene,
} from 'foldkit/scene'

// A Submodel's update returns [Model, Commands, Option<OutMessage>]. Scene
// tracks the third element, so a page-level test asserts what the child
// announced to its parent.
scene(
  { update, view },
  given(initialModel),
  click(role('button', { name: 'Log out' })),
  expectOutMessage(RequestedLogout()),
  Subscription.emit(CompletedAction()),
  expectNoOutMessage(),
)
```

Scene keeps the third element of the most recent update result that included one. A two-tuple branch leaves the previous OutMessage in place. Keep every branch of an OutMessage-returning update on the three-tuple shape, and return `Option.none()` when there is nothing to report.

## Submodels with ViewInputs

A Submodel with ViewInputs has a `(model, viewInputs, h)` view. Scene expects `(model, h)`. `withViewInputs(view, defaults)` adapts the view once and returns a factory for Scene views.

```
import { inertHtml as ih } from 'foldkit/html'
import { expect, given, role, scene, withViewInputs } from 'foldkit/scene'

import { Slider } from '@foldkit/ui'

const model = Slider.init({ id: 'volume', min: 0, max: 10, step: 1 })

// Defaults supply the full ViewInputs once. The returned factory produces
// a (model, h) view for scene, taking per-test overrides for
// everything except toView.
const sceneView = withViewInputs(Slider.view, {
  value: 5,
  toView: attributes =>
    ih.div(
      [...attributes.root],
      [ih.div([...attributes.track]), ih.div([...attributes.thumb])],
    ),
})

// Vary value inputs per test while the renderer stays pinned.
scene(
  { update: Slider.update, view: sceneView() },
  given(model),
  expect(role('slider')).toHaveAttr('aria-valuenow', '5'),
)

scene(
  { update: Slider.update, view: sceneView({ isDisabled: true }) },
  given(model),
  expect(role('slider')).toHaveAttr('aria-disabled', 'true'),
)
```

Each test can override any ViewInputs field except `toView`, so values vary while the renderer stays fixed. See `packages/ui/src/slider/scene.test.ts` for a complete example.

## A Complete Scene

Here’s a Scene test for a weather app. The user types a zip code, requests the weather, sees the loading state, and then sees the forecast.

```
import {
  Command,
  click,
  expect,
  given,
  inside,
  label,
  role,
  scene,
  text,
  type,
} from 'foldkit/scene'
import { test } from 'vitest'

test('type a zip code, click get weather, see the forecast', () => {
  scene(
    { update, view },
    given(model),

    type(label('Zip code'), '90210'),
    click(role('button', { name: 'Get Weather' })),
    expect(role('button', { name: 'Loading...' })).toExist(),

    // Instance form: locks in the zipCode the runtime captured.
    Command.expectExact(FetchWeather({ zipCode: '90210' })),
    Command.resolve(
      FetchWeather,
      SucceededFetchWeather({ weather: beverlyHillsWeather }),
    ),
    inside(
      role('article'),
      expect(text('Beverly Hills, California')).toExist(),
      expect(text('72\u00B0F')).toExist(),
      expect(text('Clear sky')).toExist(),
    ),
  )
})
```

The interactions use a label, a role, and a placeholder. The Command result stays beside the interaction that produced it.

## Story vs Scene

Use Story when the Message sequence and resulting Model are the contract. Use Scene when the interaction and rendered result are the contract. The [Testing overview](https://foldkit.dev/testing) shows where each kind of test belongs in a project.
