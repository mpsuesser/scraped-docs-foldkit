---
url: https://foldkit.dev/testing/scene
title: "Scene"
description: "Test features through the rendered view with Scene. Click buttons, type into inputs, and assert on the HTML using accessible locators."
access_date: 2026-08-03T17:27:13.509Z
current_date: 2026-08-03T17:27:13.509Z
---

# Scene

## Testing Through the View

`Scene` tests features through the rendered view. Where [Story](https://foldkit.dev/testing/story) sends Messages directly to update, Scene clicks buttons, types into inputs, presses keys, and asserts on the rendered VNode tree. The view function runs on every step, so if it crashes or renders the wrong thing, the test catches it.

Scene operates on the VNode tree directly. No DOM, no jsdom, no browser. Tests are pure, deterministic, and fast.

Import the steps you need from `foldkit/scene`. A test file usually needs only one of the two testing modules, so named imports keep the call sites short. When a single file tests both a story and a scene, import the namespaces instead (`import { Scene, Story } from 'foldkit'`) so `given` and `given` stay distinguishable.

## Locators

Locators find elements the way users find them: by role, by label, by visible text. Each factory returns a `Locator` that resolves to a single match; interactions and assertions accept either a Locator or a raw CSS selector string.

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

Locator

Finds

Example

`role(role, options?)`

Elements by ARIA role (explicit or implicit). Options narrow by accessible name and ARIA state.

`role('button', { name: 'Save' })`

`label(text)`

Form controls by their aria-label or associated <label> text.

`label('Email')`

`placeholder(text)`

Inputs by their placeholder attribute.

`placeholder('Search...')`

`text(text)`

Elements by visible text content.

`text('Welcome back')`

`altText(text)`

Images and similar elements by their alt attribute.

`altText('Profile photo')`

`title(text)`

Elements by their title attribute (tooltip text).

`title('Delete')`

`testId(id)`

Elements by data-testid: the escape hatch for tests.

`testId('cart-item-3')`

`displayValue(value)`

Form controls by their current value.

`displayValue('US')`

`selector(css)`

Elements by CSS selector. Use when no accessible query fits.

`selector('.chart-legend')`

### The role Locator

`role` is the most common locator. It accepts a second argument of state options that narrow the match. All options are optional:

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

Option

Type

Matches

`name`

`string | RegExp`

Accessible name (aria-label, aria-labelledby, label[for], or text content). Strings match exactly; regular expressions match against the full name.

`level`

`number`

Heading level (for role: "heading")

`checked`

`boolean | 'mixed'`

aria-checked or the checked attribute

`selected`

`boolean`

aria-selected

`pressed`

`boolean | 'mixed'`

aria-pressed

`expanded`

`boolean`

aria-expanded

`disabled`

`boolean`

aria-disabled or the disabled attribute

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

For lists and repeated elements, the `all.*` factories (`all.role`, `all.text`, `all.label`, and so on, one per single-match factory) return a `LocatorAll` that resolves to every match. Pick one with `first`, `last`, or `nth(index)`, or narrow with `filter`:

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

Filter option

Keeps matches where

`has`

The element contains a descendant matching the given Locator

`hasNot`

The element does not contain a descendant matching the Locator

`hasText`

The element’s text content includes the given substring

`hasNotText`

The element’s text content does not include the substring

## Interactions

Interactions exercise the view by invoking event handlers on matched elements. Each one captures the dispatched Message, feeds it through update, and re-renders. They accept either a Locator or a CSS selector string.

```
import {
  blur,
  change,
  click,
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

Step

Invokes

`click(target)`

`OnClick`

(bubbles to ancestors)

`doubleClick(target)`

`OnDoubleClick`

(bubbles to ancestors)

`pointerDown(target, options?)`

`OnPointerDown`

with optional

`{ pointerType, button, screenX, screenY }`

(bubbles to ancestors)

`pointerUp(target, options?)`

`OnPointerUp`

with optional

`{ pointerType, screenX, screenY }`

(bubbles to ancestors)

`hover(target)`

`OnMouseEnter`

(falls back to

`OnMouseOver`

)

`focus(target)`

`OnFocus`

`blur(target)`

`OnBlur`

`type(target, text)`

`OnInput`

with the given text

`change(target, value)`

`OnChange`

with the given value, for

`<select>`

and similar

`keydown(target, key, modifiers?)`

`OnKeyDown`

or

`OnKeyDownPreventDefault`

with optional

`{ shiftKey, ctrlKey, altKey, metaKey }`

`submit(target)`

`OnSubmit`

`tap(fn)` runs a function for side effects (like ad-hoc assertions on raw VNodes or accumulated Commands) without breaking the step chain.

## Assertions

`expect(locator)` creates an inline assertion step against a single element. Every matcher has a `.not` variant that inverts the assertion.

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

Matcher

Asserts that the element

`.toExist()`

Is present in the tree

`.toBeAbsent()`

Is not present in the tree

`.toBeVisible()`

Is not hidden via the hidden attribute, aria-hidden, display: none, or visibility: hidden

`.toBeEmpty()`

Has no text content or child elements

`.toHaveText(value)`

Has text content equal to the given string or matching the given regex

`.toContainText(value)`

Has text content including the given substring or matching the regex

`.toHaveAccessibleName(name)`

Has the given accessible name (resolves aria-labelledby, aria-label, label[for], text content)

`.toHaveAccessibleDescription(description)`

Has the given accessible description (resolves aria-describedby)

`.toBeDisabled()`

Has aria-disabled or the disabled attribute

`.toBeEnabled()`

Is not disabled

`.toBeChecked()`

Has aria-checked="true" or the checked attribute

`.toHaveValue(value)`

Has the given current form-control value

`.toHaveAttr(name, value)`

Has the given attribute set to the given value

`.toHaveId(id)`

Has the given id

`.toHaveClass(name)`

Has the given CSS class

`.toHaveStyle(name, value)`

Has the given inline style property

For `LocatorAll` (from `all.*`), use `expectAll(locatorAll)` for count-based assertions:

Matcher

Asserts that

`.toHaveCount(n)`

The locator matches exactly n elements

`.toBeEmpty()`

The locator matches zero elements

## Commands

When `update` returns Commands (see [Commands](https://foldkit.dev/core/commands)), Scene tracks each as pending until the test resolves it with the result Message its Effect would resolve to at runtime. `update` declares the Command, the test declares its outcome.

Command tracking has a few semantics worth knowing:

- Pending Commands accumulate in the order `update` returns them, across as many steps as the test takes.
- Resolving a Command feeds its result Message through `update`; new Commands produced by that update join the pending list.
- `Command.resolveAll` walks cascades within the batch. If resolving Command A produces Command B and B’s resolver is in the same call, B resolves without a separate step.
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

Step

Effect

`Command.resolve(Def, ResultMessage)`

Resolves the first pending Command with the given name by feeding

`ResultMessage`

through update. For a child Submodel Command, pass the child’s raw result Message; resolve replays the Command’s own

`mapMessages`

wrapping automatically.

`Command.resolveAll([Def, ResultMessage], ...)`

Resolves a batch of pending Commands, walking cascades. Each entry resolves exactly one matching dispatch in declaration order; compose with Array.makeBy for N identical responses.

`Command.expectExact(A, B)`

The pending Commands are exactly A and B (order-independent).

`Command.expectHas(A)`

A is among the pending Commands (subset check).

`Command.expectNone()`

There are no pending Commands.

Prefer `Command.expectExact` as the default. It catches bugs where an interaction produces unexpected Commands. Use `Command.expectHas` when you only care about a subset of the pending Commands.

Each matcher accepts either a Command Definition (matches by name) or a Command instance (matches by name AND structural-equal args). Pass a Definition when the test only cares that the Command was dispatched; pass an instance when the args are part of what the test is verifying. `Command.expectExact(FetchWeather({ zipCode: '90210' }))` fails if the runtime dispatched `FetchWeather({ zipCode: '99999' })`, where the same call with just `FetchWeather` would pass.

## Mounts

When a rendered view contains an `OnMount` attribute (see [Mount](https://foldkit.dev/core/mount)), Scene tracks the mount as pending until the test acknowledges it with the result Message its Effect would resolve to at runtime. The mechanic mirrors Command resolution: the view declares the Mount, the test declares its outcome.

Many UI components in `@foldkit/ui` declare mounts internally (popovers positioning their panels, modal components portaling backdrops to the body, components that hand the live element to a third-party library). When the test renders any of these, the same `OnMount` shows up in the VNode tree, and Scene treats it as a pending mount. Acknowledging it advances the test through the same path the user takes: the view renders, the mount fires, the result Message updates the Model.

Mount tracking has a few semantics worth knowing:

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

Step

Effect

`Mount.resolve(Def, ResultMessage)`

Resolves the first pending mount with the given name by feeding

`ResultMessage`

through update. For a mount inside a child Submodel, resolve replays the boundary lift automatically, so you pass the child’s raw result Message.

`Mount.resolveAll([Def, ResultMessage], ...)`

Resolves a batch of pending mounts in order.

`Mount.expectExact(A, B)`

The pending mounts are exactly A and B (order-independent, by name).

`Mount.expectHas(A)`

A is among the pending mounts (subset check).

`Mount.expectNone()`

There are no pending mounts.

`Mount.expectEnded(A)`

A has disappeared from the rendered tree. Required for every Mount that fires and then unmounts during the scene, regardless of whether it was resolved first; otherwise the scene throws at the end.

UI components export their Mount definitions (`Popover.AnchorPopover`, `Listbox.AnchorListbox`, and so on) so consumer tests can name them in `Mount.resolve`.

## Subscriptions

Messages caused by a DOM event enter a scene through interactions, and Messages caused by a Command or Mount enter through `resolve`. A Message whose real cause is a [Subscription](https://foldkit.dev/core/subscriptions) (a timer tick, a WebSocket frame, a global listener) has no element in the rendered tree, so `Subscription.emit(message)` feeds it through update directly and re-renders like any other step.

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

Reach for it only when the Message's cause lives outside the rendered tree. If the Message has a DOM affordance, click the actual button instead; the interaction proves the handler wiring that `emit` skips. Like interactions, `emit` throws if unresolved Commands, unresolved Mounts, or unacknowledged unmounts are pending.

## Managed Resources

A [ManagedResource](https://foldkit.dev/core/managed-resources) dispatches lifecycle Messages through its declared hooks: `onAcquired(value)` when acquisition succeeds, `onAcquireError(error)` when it fails, and `onReleased()` after release. The `ManagedResource` steps declare those outcomes the way `Command.resolve` declares a Command result, feeding the hook's Message through update.

Each step checks the current Model against the entry's `modelToMaybeRequirements` gate first, mirroring the runtime's `None` to `Some` and `Some` to `None` transitions: `acquire` and `failAcquire` throw unless the Model requests the resource (`Some`), and `release` throws while it still does. A scene therefore has to drive the Model transition through real steps before declaring the lifecycle outcome. The runtime's third transition, the `Some` to `Some` re-acquire when the requirements change structurally (which dispatches `onReleased` and then `onAcquired` while the Model still requests the resource), has no step yet.

Unlike Commands and Mounts, these steps leave nothing pending: each dispatches its Message through update immediately, so there is nothing to resolve or acknowledge at the end of the scene. The steps are also never required. Scene only sees `update` and `view`, not the application's ManagedResources record, so driving the Model into the requesting state without ever calling `acquire` is legal and ends the scene in the in-flight state, the same state the runtime sits in while its own `acquire` Effect runs. The gates work the other way around: any step you do call throws immediately when the current Model contradicts the lifecycle outcome it declares.

`acquire` takes exactly the arguments the entry's `onAcquired` declares. A handler that consumes the acquired value, like `socket => Connected({ socketId: socket.socketId })`, requires the value here (what the entry's `acquire` Effect would have produced). A handler that ignores it, like `() => Connected()`, takes none, so a test never fabricates a resource value nobody reads.

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

Step

Effect

`ManagedResource.acquire(entry, ...args)`

Feeds

`onAcquired(...args)`

through update. The Model must request the resource.

`ManagedResource.failAcquire(entry, error)`

Feeds

`onAcquireError(error)`

through update. The Model must request the resource.

`ManagedResource.release(entry)`

Feeds

`onReleased()`

through update. The Model must no longer request the resource.

## Custom Elements

A [CustomElement](https://foldkit.dev/core/custom-element) converts declared CustomEvents into Messages through its `On*` event attributes. `CustomElement.emit(spec, target, eventName, detail)` dispatches such an event on a rendered element: the event name and detail are typed by the spec's event Schemas, and the Message comes out of the same mapping the browser event would run. The element must be in the rendered tree with the event's attribute attached; a missing element or missing handler throws.

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

When the update under test is a Submodel's three-tuple update, Scene tracks its `Option<OutMessage>` the same way Story does. `expectOutMessage(expected)` asserts the OutMessage is `Some(expected)`; `expectNoOutMessage()` asserts there is none.

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

The tracked value is the third element of the most recent update result that had one. An update branch that returns a two-tuple leaves the previous value in place, so keep every branch of an OutMessage-returning update on the three-tuple shape, returning `Option.none()` when there is nothing to report.

## Submodels with ViewInputs

A Submodel that declares `ViewInputs` has a `(model, viewInputs, h)` view, which does not match the `(model, h)` shape `scene` takes. `withViewInputs(view, defaults)` closes the gap: pass the view and its full default inputs once, and the returned factory produces a scene view.

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

The factory's overrides accept every `ViewInputs` field except `toView`, so tests vary value inputs while the renderer stays pinned. The published Submodels in `packages/ui/src/` are tested exactly this way; `packages/ui/src/slider/scene.test.ts` is the canonical example.

## A Complete Scene

Here’s a Scene test for a weather app. The user types a zip code, clicks Get Weather, sees a loading state, and then the forecast appears:

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

Every interaction targets an element the way a user would: by label, by role, by placeholder. Every assertion reads like a sentence. Commands are resolved inline, just like in Story.

## Story vs Scene

Story and Scene are complementary. Story tests the state machine: does this sequence of Messages produce the right Model? Scene tests the contract: does this feature work from the user’s perspective?

Use Story for update logic, edge cases, and Command wiring. Use Scene for user flows, view rendering, and accessibility. A well-tested app uses both.
