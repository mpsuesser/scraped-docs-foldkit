---
url: https://foldkit.dev/core/mount
title: "Mount"
description: "OnMount: the single mount-time DOM hook for integrating third-party libraries with paired cleanup. Keeps imperative work confined to the seam where the virtual DOM meets the real one."
access_date: 2026-08-20T02:21:49.544Z
current_date: 2026-08-20T02:21:49.544Z
---

## Overview

Most Foldkit code is declarative. The [view](https://foldkit.dev/core/view) is a pure function from Model to Html. It does not reach into the DOM, hold element references, or run side effects.

Mount is the escape hatch for work whose cause is a particular element existing in the DOM. `OnMount` supplies the live `Element`, starts the work when that element enters the DOM, and tears it down when the element leaves.

Use `Mount.define` for work that produces one Message when it starts. The returned `Effect<Message>` emits that Message, then its scope remains open until unmount so cleanup registered with `Effect.acquireRelease` runs at the right time. Use `Mount.defineStream` when listeners or observers on the element must emit a continuing `Stream<Message>`.

Both forms require at least one declared result Message. When no result needs to change the Model, return a descriptive `Completed*` Message and leave the Model unchanged in update. The Message keeps the effect visible to DevTools, Scene tests, and replay.

Functional core, imperative shell

The view describes what should be on screen. `OnMount` handles imperative work at the boundary between the virtual DOM and a live element. Results still return to update as Messages, and setup stays paired with cleanup inside the Mount's Effect or Stream.

Mounts surface in tests

Scene records every `OnMount` in the rendered view as a pending Mount. A test must acknowledge or resolve each one with a declared result Message. See [Scene](https://foldkit.dev/testing/scene) for the full contract.

## When to Reach for Mount

Choose a lifecycle primitive by what causes the work:

- A [Command](https://foldkit.dev/core/commands) runs because update just handled a Message. Use it for one-time work such as navigation, network requests, storage, analytics, or focusing an input after `OpenedDialog`.
- A Mount runs because an element exists, and the work needs that live `Element`. Use it to measure geometry, portal a node, attach an element observer, or instantiate a library in a specific container.
- A [Subscription](https://foldkit.dev/core/subscriptions) listens to an external event source while dependencies derived from the Model remain active. Timers, document events, system theme changes, and WebSocket messages fit this shape.
- A [ManagedResource](https://foldkit.dev/core/managed-resources) owns a stateful handle whose lifetime follows a Model condition and whose operations are performed by Commands.
- A [CustomElement](https://foldkit.dev/core/custom-element) renders a native web component with declarative properties and `CustomEvent` s. Use Mount only when the foreign element requires an imperative API instead.

Check the cause, not the timing

Work that happens when an element appears is not necessarily caused by that element. For example: when `OpenedDialog` makes a search input visible, focusing it is still caused by the Message. Return a `FocusInput` Command from that Message's update handler. Reach for Mount only when the work is inseparable from the live element and its lifetime.

## Side Effects on Mount

A Mount follows the lifetime of a DOM node, not a VNode. Foldkit reconstructs VNodes on every render, but the differ reuses an existing DOM node when its tag and identity still match. A reused node keeps its Mount running. A replaced node closes the old Mount's scope, runs its finalizers, and starts a fresh Mount on the new node.

View-function identity and stable keys keep that lifecycle attached to the right logical element. This matters in mapped lists, where an unkeyed reorder can reuse the same DOM position for different data. Key each item by a stable Model identifier so its DOM node and Mount move together.

Portal-to-body is a small example. When an overlay enters the DOM, its Mount moves the live element to `document.body` so it can escape clipping ancestors. When the element unmounts, the paired release removes it.

```
import { Effect } from 'effect'
import { Mount } from 'foldkit'
import type { Html, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'

const CompletedPortalToBody = m('CompletedPortalToBody')

// Portal-to-body is a per-instance lifecycle effect that uses the element
// directly. The Effect's acquireRelease moves the element to document.body
// at mount and removes it on unmount. The work is pure DOM manipulation on
// the element Mount provides, idempotent and safe to re-run during
// DevTools time-travel.

const PortalToBody = Mount.define(
  'PortalToBody',
  CompletedPortalToBody,
)(element =>
  Effect.gen(function* () {
    yield* Effect.acquireRelease(
      Effect.sync(() => document.body.appendChild(element)),
      () => Effect.sync(() => element.remove()),
    )
    return CompletedPortalToBody()
  }),
)

const overlayView = (h: HtmlBuilder<Message>): Html =>
  h.div([h.Class('fixed inset-0 bg-black/50'), h.OnMount(PortalToBody())])
```

Two rules for Mount work

First, the factory must use the live element. If it does not read or write that element, a Message or Model condition is probably the real cause. Second, the work must be safe to repeat whenever that element is inserted again. DOM measurement, paired DOM manipulation, observers, and element-owned library instances fit these rules.

Attach one Mount per element

A VNode has one `insert` and `destroy` hook. If the same element receives `[h.OnMount(A), h.OnMount(B)]`, the second action silently replaces the first and `A` never runs. Combine both behaviors in one Mount and register both releases in its scope.

Mounts re-run during DevTools time-travel

DevTools re-renders historical Models. Elements inserted during replay run their Mounts again, and elements removed during replay run their finalizers. Keep Mount work replay-safe and local to the element. External mutations such as network calls, storage writes, and analytics belong in Commands.

## Per-Instance Args

Mount factories often need an input that differs by element instance, such as an initial scroll position, chart data, or a stable host id. Declare a Schema record of `args` with the same shape used by [Commands](https://foldkit.dev/core/commands):

```ts
Mount.define(name, args, ...results)(args => element => Effect<Message>)
```

Calling the Definition with an args record creates the MountAction passed to `OnMount`. `Mount.defineStream` supports the same overload and returns a `Stream<Message>` from its factory instead.

Args are only per-instance inputs. Module constants stay in lexical scope, app-wide services come from Foldkit `Resources`, Model-owned handles come from `ManagedResources`, and Effect services remain available through `yield*` inside the factory.

Args surface in DevTools and tests

DevTools shows the args beside the Mount name. Scene tests can target one instance by passing the same args record to `Mount.expectHas` or `Mount.resolve`. See [Scene](https://foldkit.dev/testing/scene) for the Definition and instance matcher contract.

Args are captured at mount

The factory receives the args from the render that inserts the element. Later renders create new MountActions, but a reused DOM node does not run the factory again. Name values for that lifecycle, such as `initialScroll` or `seedValue`, rather than implying that they stay current.

When a later Message changes the Model and should trigger new DOM work, return a Command from that Message's update handler. A Subscription is appropriate when a Model dependency controls the lifetime of an external stream or a paired DOM state, or when a browser event must be handled synchronously, such as calling `preventDefault` inside its listener. Mount args are not reactive properties for either case.

## Third-Party Libraries

Mount is especially useful when a library owns a rendered subtree. Charts, code editors, map renderers, and force-directed graphs all need a real element to render into and a way to release their resources.

Construct the handle in an acquire Effect, return the Mount's result Message, and register teardown with `Effect.acquireRelease`. The Effect can finish after emitting its Message because Foldkit keeps its scope open until the element unmounts.

```
import { Effect, Schema as S } from 'effect'
import { Mount } from 'foldkit'
import type { Html, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'

const SucceededMountChart = m('SucceededMountChart')
const FailedMountChart = m('FailedMountChart', { reason: S.String })

// Mount.define gives the action a name and constrains what Messages it can
// produce, plus an args record so the chart's per-instance data flows through
// declared values rather than a closure. The runtime invokes the bound factory
// on insert, runs the Effect to produce one Message, dispatches it, and closes
// the scope on destroy (firing any acquireRelease finalizers).

const ChartData = S.Array(S.Number)
type ChartData = typeof ChartData.Type

const MountChart = Mount.define(
  'MountChart',
  { data: ChartData },
  SucceededMountChart,
  FailedMountChart,
)(
  ({ data }) =>
    element =>
      Effect.gen(function* () {
        yield* Effect.acquireRelease(
          Effect.tryPromise(() => import('some-chart-library')).pipe(
            Effect.map(({ Chart }) => new Chart(element, { data })),
          ),
          chart => Effect.sync(() => chart.destroy()),
        )
        return SucceededMountChart()
      }).pipe(
        Effect.catch(error =>
          Effect.succeed(
            FailedMountChart({
              reason: error instanceof Error ? error.message : String(error),
            }),
          ),
        ),
      ),
)

const chartView = (data: ChartData, h: HtmlBuilder<Message>): Html =>
  h.div([h.Class('w-[480px] h-[320px]'), h.OnMount(MountChart({ data }))])
```

Construct the handle inside the acquire body

`Effect.acquireRelease` registers the release only after its acquire Effect succeeds. Constructing a chart, map, or other stateful handle before that Effect can leak the handle if interruption happens before registration. Make construction the acquire Effect's success value. For example: use `Effect.sync(() => new Thing(...))`, or put an asynchronous import and construction in the same Effect pipeline. Whatever the release needs must be produced by the acquire Effect.

The Model owns the input data. The library owns its rendered subtree. Foldkit owns the lifecycle.

Unmount interrupts the work

When the element unmounts, Foldkit interrupts the Mount's fiber and runs registered finalizers. Any Messages produced after interruption are discarded, so update never receives a Mount Message for an element that no longer exists.

When a foreign element already exposes a declarative property-and-event API, bind it with [CustomElement](https://foldkit.dev/core/custom-element) instead.
