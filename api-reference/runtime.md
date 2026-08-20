---
url: https://foldkit.dev/api-reference/runtime
title: "Runtime"
description: "API documentation for the Runtime module."
access_date: 2026-08-20T02:21:49.544Z
current_date: 2026-08-20T02:21:49.544Z
---

# Runtime

## Functions

### defaultSlowCallback

function

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L453)

```
(context: SlowContext<unknown, unknown>): void
```

### embed

function

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L4478)

```
/**
 * Starts a Foldkit runtime under a host-controlled lifecycle and returns an
 * `EmbedHandle`. This is the entry point for embedding a Foldkit app inside
 * another application: the host pushes values in through the handle's inbound
 * Ports, listens to outbound Ports, and calls `dispose` when it unmounts the
 * app. The host never touches the Model or dispatches Messages directly; the
 * Schema-typed Ports are the whole boundary.
 * 
 * Works with programs from both `makeApplication` and `makeElement`; for a
 * widget on a page the host owns, `makeElement` is the natural fit.
 * 
 * A program can be embedded once at a time (it owns one container). After
 * `dispose`, the same container can be embedded again with a fresh program.
 * 
 * ```ts
 * const handle = Runtime.embed(element)
 * 
 * handle.ports.stepChanged.send(5)
 * const unsubscribe = handle.ports.countChanged.subscribe(count => {
 *   console.log(count)
 * })
 * 
 * handle.dispose()
 * ```
 */
<P extends Readonly<{
  inbound: Readonly<Record<string, Inbound<any, any>>>
  outbound: Readonly<Record<string, Outbound<any, any>>>
}> | undefined = undefined, Resources = never, Kind extends "Application" | "Element" = "Application" | "Element">(program: MakeRuntimeReturn<P, void, Resources, Kind>): EmbedHandle<P>

<P extends Readonly<{
  inbound: Readonly<Record<string, Inbound<any, any>>>
  outbound: Readonly<Record<string, Outbound<any, any>>>
}> | undefined, Flags, Resources, Kind extends "Application" | "Element">(
  program: MakeRuntimeReturn<P, Flags, Resources, Kind>,
  options: RunOptions<Flags, Resources>
): EmbedHandle<P>
```

### hydrate

function

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L4422)

```
<P extends Readonly<{
  inbound: Readonly<Record<string, Inbound<any, any>>>
  outbound: Readonly<Record<string, Outbound<any, any>>>
}> | undefined, Flags, Resources>(
  program: MakeRuntimeReturn<P, Flags, Resources, "Application">,
  options: HydrateOptions
): void
```

### makeApplication

function

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L3752)

### makeElement

function

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L4057)

```
/**
 * Creates a Foldkit app scoped to its container and returns a runtime that
 * can be passed to `run`.
 * 
 * Unlike `makeApplication`, the `view` returns `Html` directly rather than a
 * `Document`, and the runtime never touches the document `<head>`. This lets a
 * Foldkit app be embedded at a node (a widget on a page it does not own)
 * without clobbering the host page's `title`, `canonical`, or `og:url`. Use
 * `makeApplication` when the app owns the page and should manage those tags, and
 * `makeElement` when it is one component among others on a page it does not
 * control. Embedded apps do not own the URL bar, so `makeElement` has no
 * `routing` config.
 */
<Model, Message extends {
  _tag: string
}, Flags, Resources = never, ManagedResourceServices = never, P extends Readonly<{
  inbound: Readonly<Record<string, Inbound<any, any>>>
  outbound: Readonly<Record<string, Outbound<any, any>>>
}> | undefined = undefined>(config: ElementConfigWithFlags<Model, Message, Flags, Resources, ManagedResourceServices, P>): MakeRuntimeReturn<P, void, Resources, "Element">

<Model, Message extends {
  _tag: string
}, Resources = never, ManagedResourceServices = never, P extends Readonly<{
  inbound: Readonly<Record<string, Inbound<any, any>>>
  outbound: Readonly<Record<string, Outbound<any, any>>>
}> | undefined = undefined>(config: ElementConfig<Model, Message, Resources, ManagedResourceServices, P>): MakeRuntimeReturn<P, void, Resources, "Element">
```

### run

function

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L4362)

```
/**
 * Starts a Foldkit runtime that owns the page for the page's whole lifetime,
 *  with HMR support for development. The first render builds the DOM fresh in
 *  the container, replacing whatever is there. On a server-rendered page use
 *  `hydrate` instead, which adopts the existing DOM. To start a runtime under a
 *  host-controlled lifecycle, use `embed`.
 */
<P extends Readonly<{
  inbound: Readonly<Record<string, Inbound<any, any>>>
  outbound: Readonly<Record<string, Outbound<any, any>>>
}> | undefined, Resources, Kind extends "Application" | "Element">(program: MakeRuntimeReturn<P, void, Resources, Kind>): void

<P extends Readonly<{
  inbound: Readonly<Record<string, Inbound<any, any>>>
  outbound: Readonly<Record<string, Outbound<any, any>>>
}> | undefined, Flags, Resources, Kind extends "Application" | "Element">(
  program: MakeRuntimeReturn<P, Flags, Resources, Kind>,
  options: RunOptions<Flags, Resources>
): void
```

## Types

### ApplicationConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1197)

```
/** Configuration for `makeApplication` without Flags or URL routing. */
type ApplicationConfig = BaseApplicationConfig<Model, Message, Resources, ManagedResourceServices, P> & Readonly<{
  init: () => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
}>
```

### ApplicationConfigWithFlags

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1170)

```
/** Configuration for `makeApplication` with Flags but no URL routing. */
type ApplicationConfigWithFlags = BaseApplicationConfig<Model, Message, Resources, ManagedResourceServices, P> & FlagsSchemaConfig<Flags> & Readonly<{
  init: (flags: Flags) => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
}>
```

### ApplicationInit

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1307)

```
/** The `init` function type for a `makeApplication` app without URL routing. */
type ApplicationInit = Flags extends void
  ? () => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
  : (flags: Flags) => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
```

### CrashConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L608)

```
/**
 * Configuration for crash handling, with custom crash UI and/or crash
 *  reporting. The crash view renders after the dispatch loop has stopped, so
 *  its builder's Message is `never` and no handler is expressible. Reload or
 *  navigate with a raw DOM attribute instead.
 */
type CrashConfig = Readonly<{
  report: (context: CrashContext<Model, Message>) => void
  view: (context: CrashContext<Model, Message>, h: HtmlBuilder<never>) => Document
}>
```

### CrashContext

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L598)

```
/**
 * Context provided to crash.view and crash.report when the runtime encounters
 *  an unrecoverable error. `message` is the Message being processed when the
 *  crash occurred, present as an `Option` because a crash during the initial
 *  render has no triggering Message.
 */
type CrashContext = Readonly<{
  error: Error
  message: Option.Option<Message>
  model: Model
}>
```

### DevToolsConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L178)

```
/**
 * DevTools configuration.
 * 
 * Pass `false` to disable DevTools entirely.
 * 
 * - `show`: `'Development'` (default) enables in dev mode only, `'Always'` enables in all environments including production.
 * - `position`: Where the badge and panel appear. Defaults to `'BottomRight'`.
 * - `mode`: `'TimeTravel'` (default) enables full time-travel debugging. `'Inspect'` allows browsing state snapshots without pausing the app. Pass `{ development, production }` to use different modes per environment. Useful when DevTools is shown in production (`show: 'Always'`) and you want `'TimeTravel'` only in local development.
 * - `banner`: Optional text shown as a banner at the top of the panel.
 * - `excludeFromHistory`: Message `_tag` values whose dispatches should not be recorded in DevTools history. The Messages still drive `update` and the runtime as usual; they just don't appear in the history panel and don't pay the per-Message diff cost. Use for high-frequency Messages (animation frames, pointer moves, scroll events) that would flood history without adding insight.
 * - `maxEntries`: Maximum number of recorded Messages retained in history before the oldest is evicted. Defaults to 100. Clamped to the range 20-500: smaller values keep the panel snappy under high message rates, larger values give you more scroll-back. Each retained entry stores a full Model snapshot, so memory cost scales linearly with both `maxEntries` and your Model size.
 * - `keyframeInterval`: Number of recorded Messages between full Model snapshots. Defaults to 31. Time-travel to an index replays `update` forward from the nearest earlier keyframe, so this is a memory/time tradeoff: smaller values store more snapshots (more memory) but make each jump cheaper, down to `1` where every jump is a constant-time snapshot lookup with no replay. Reach for a denser interval when the app has a heavy `update` and time-travel jumps feel sluggish. Clamped to a minimum of 1. Forced to 1 automatically when `excludeFromHistory` is active, since excluded Messages are never replayed.
 */
type DevToolsConfig = false | Readonly<{
  banner: string
  excludeFromHistory: ReadonlyArray<string>
  keyframeInterval: number
  maxEntries: number
  Message: Schema.Codec<any, any, unknown, unknown>
  mode: DevToolsModeConfig
  position: DevToolsPosition
  show: Visibility
}>
```

### DevToolsMode

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L137)

```
/**
 * Controls DevTools interaction mode.
 * 
 * - `'Inspect'`: Messages stream in and clicking a row shows its state snapshot without pausing the app.
 * - `'TimeTravel'`: Clicking a row pauses the app at that historical state. Resume to continue.
 */
type DevToolsMode = "Inspect" | "TimeTravel"
```

### DevToolsModeConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L144)

```
/**
 * Mode value for the DevTools panel. Either a single mode used in every
 *  environment, or an object selecting different modes for development and
 *  production. Use the object form to keep `'TimeTravel'` for local debugging
 *  while shipping the safer `'Inspect'` mode to users. `'TimeTravel'` in
 *  production pauses the user's app when a history row is clicked.
 */
type DevToolsModeConfig = DevToolsMode | Readonly<{
  development: DevToolsMode
  production: DevToolsMode
}>
```

### DevToolsPosition

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L123)

```
/** Position of the DevTools badge and panel on screen. */
type DevToolsPosition = "BottomRight" | "BottomLeft" | "TopRight" | "TopLeft"
```

### ElementConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1290)

```
/** Configuration for `makeElement` without Flags. */
type ElementConfig = BaseElementConfig<Model, Message, Resources, ManagedResourceServices, P> & Readonly<{
  init: () => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
}>
```

### ElementConfigWithFlags

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1260)

```
/** Configuration for `makeElement` with Flags. */
type ElementConfigWithFlags = BaseElementConfig<Model, Message, Resources, ManagedResourceServices, P> & FlagsSchemaConfig<Flags> & Readonly<{
  flags: Effect.Effect<Flags, never, NoInfer<Resources>>
  init: (flags: Flags) => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
}>
```

### ElementCrashConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1222)

```
/**
 * Configuration for crash handling in a `makeElement` app. The crash view
 *  returns `Html`, not a `Document`, because a scoped app never owns the
 *  document `<head>`.
 */
type ElementCrashConfig = Readonly<{
  report: (context: CrashContext<Model, Message>) => void
  view: (context: CrashContext<Model, Message>, h: HtmlBuilder<never>) => Html
}>
```

### ElementInit

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1358)

```
/**
 * The `init` function type for a `makeElement` app. A scoped app never owns
 *  the URL, so its `init` has the same shape as a non-routing
 *  `ApplicationInit`: argless, or receiving Flags when `Flags` is set.
 */
type ElementInit = ApplicationInit<Model, Message, Flags, Resources, ManagedResourceServices>
```

### EmbedHandle

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1463)

```
/**
 * The handle returned by `embed`. The host talks to the embedded app only
 * through it: `ports.<name>.send` pushes values in, `ports.<name>.subscribe`
 * listens to values the app emits, and `dispose` shuts the runtime down.
 * 
 * `dispose` is idempotent. It interrupts the runtime and runs all cleanup:
 * Subscriptions, ManagedResources, Mounts, listeners, and in-flight Commands
 * stop, and the rendered DOM is removed with the container element restored
 * empty in its place, ready for a fresh `embed`.
 */
type EmbedHandle = Readonly<{
  dispose: () => void
  ports: PortHandles<P>
}>
```

### HydrateOptions

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L4384)

```
/** Options for hydrate. */
type HydrateOptions = Readonly<{
  buildId: string
}>
```

### InboundPortHandle

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1407)

```
/**
 * Host-side handle for one inbound Port. `send` validates the value by
 *  decoding it against the Port's Schema: on success the decoded value enters
 *  the app through the Port's Subscription; on failure nothing reaches the
 *  app, the failure is logged, and the returned `Exit` carries the
 *  `SchemaError`. Sends after `dispose` are no-ops.
 */
type InboundPortHandle = Readonly<{
  send: (value: Encoded) => Exit.Exit<void, Schema.SchemaError>
}>
```

### InboundPortHandles

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1421)

```
/**
 * The inbound half of `PortHandles`: one `InboundPortHandle` per declared
 *  inbound Port, keyed by Port name.
 */
type InboundPortHandles = InboundPorts extends Readonly<Record<string, Inbound<any, any>>>
  ? {
    readonly [Name in keyof InboundPorts]: InboundPorts[Name] extends Inbound<any, infer Encoded>
      ? InboundPortHandle<Encoded>
      : never
  }
  : unknown
```

### MakeRuntimeReturn

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1386)

```
/**
 * A configured Foldkit runtime returned by `makeApplication` or `makeElement`.
 *  Pass it to `run` to start a page-owning app, or to `embed` to start it under
 *  a host-controlled lifecycle handle. `ports` is the Ports record from the
 *  config (or `undefined` when the config declared none); it types the
 *  `EmbedHandle` that `embed` returns. `Flags` and `Resources` carry the
 *  fresh-boot requirements from `makeApplication` to `run` and `embed`; they
 *  have no runtime representation.
 */
type MakeRuntimeReturn = Readonly<{
  [RuntimeBootTypeId]: Readonly<{
    Flags: (flags: Flags) => Flags
    Kind: Kind
    Resources: (resources: Resources) => Resources
  }>
  ports: P
  runtimeId: string
  start: (hmrModel?: unknown) => Effect.Effect<void>
}>
```

### OutboundPortHandle

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1415)

```
/**
 * Host-side handle for one outbound Port. `subscribe` registers a listener
 *  for the encoded values the app emits with `Port.emit` and returns an
 *  unsubscribe function. Multiple listeners receive each value in
 *  registration order.
 */
type OutboundPortHandle = Readonly<{
  subscribe: (listener: (value: Encoded) => void) => () => void
}>
```

### OutboundPortHandles

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1435)

```
/**
 * The outbound half of `PortHandles`: one `OutboundPortHandle` per declared
 *  outbound Port, keyed by Port name.
 */
type OutboundPortHandles = OutboundPorts extends Readonly<Record<string, Outbound<any, any>>>
  ? {
    readonly [Name in keyof OutboundPorts]: OutboundPorts[Name] extends Outbound<any, infer Encoded>
      ? OutboundPortHandle<Encoded>
      : never
  }
  : unknown
```

### PortHandles

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1449)

```
/**
 * The `ports` field of an `EmbedHandle`: one `InboundPortHandle` or
 *  `OutboundPortHandle` per declared Port, keyed by Port name.
 */
type PortHandles = P extends Ports
  ? InboundPortHandles<P["inbound"]> & OutboundPortHandles<P["outbound"]>
  : unknown
```

### RoutingApplicationConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1144)

```
/** Configuration for `makeApplication` with URL routing but no Flags. */
type RoutingApplicationConfig = BaseApplicationConfig<Model, Message, Resources, ManagedResourceServices, P> & Readonly<{
  init: (url: Url) => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
  routing: RoutingConfig<Message>
}>
```

### RoutingApplicationConfigWithFlags

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1115)

```
/** Configuration for `makeApplication` with Flags and URL routing. */
type RoutingApplicationConfigWithFlags = BaseApplicationConfig<Model, Message, Resources, ManagedResourceServices, P> & FlagsSchemaConfig<Flags> & Readonly<{
  init: (flags: Flags, url: Url) => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
  routing: RoutingConfig<Message>
}>
```

### RoutingApplicationInit

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1330)

```
/** The `init` function type for a `makeApplication` app with URL routing, receives the current URL and optional Flags. */
type RoutingApplicationInit = Flags extends void
  ? (url: Url) => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
  : (flags: Flags, url: Url) => readonly [Model, ReadonlyArray<Command<Message, never, Resources | ManagedResourceServices>>]
```

### RoutingConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L589)

```
/** Configuration for URL routing with handlers for URL requests and URL changes. */
type RoutingConfig = Readonly<{
  onUrlChange: (url: Url) => Message
  onUrlRequest: (request: UrlRequest) => Message
}>
```

### RunOptions

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L1374)

```
/**
 * Client-only startup input for an application that declares Flags. Pass it
 *  to `run` or `embed`; `hydrate` instead decodes the exact Flags value
 *  embedded by the server render.
 */
type RunOptions = Readonly<{
  flags: Effect.Effect<Flags, never, Resources>
}>
```

### SlowConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L315)

```
/**
 * Slow-phase warning configuration.
 * 
 * By default, all phases are enabled in development with Foldkit's default
 * thresholds. Pass `false` to disable warnings entirely. Pass an object to
 * refine those defaults.
 * 
 * - `show`: `'Development'` (default) enables warnings only when Vite HMR is active. `'Always'` enables them in every environment.
 * - `measuredPhases`: Phases to measure. Defaults to every slow warning phase.
 * - `thresholdOverrides`: Per-phase budget overrides. Omitted fields keep defaults; overrides for unmeasured phases are ignored.
 * - `onSlow`: Callback for every measured phase that exceeds its budget. Replaces Foldkit's default `console.warn`; Foldkit will not also warn for tags your callback ignores.
 */
type SlowConfig = false | Readonly<{
  measuredPhases: ReadonlyArray<SlowPhase>
  onSlow: (context: SlowContext<Model, Message>) => void
  show: Visibility
  thresholdOverrides: SlowThresholdOverrides
}>
```

### SlowContext

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L262)

```
/** Tagged union of every slow-phase context passed to `slow.onSlow`. */
type SlowContext = SlowViewContext<Model, Message> | SlowUpdateContext<Model, Message> | SlowPatchContext<Model, Message> | SlowSubscriptionDependenciesContext<Model>
```

### SlowPatchContext

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L244)

```
/** Context provided when DOM patching exceeds its configured time budget. */
type SlowPatchContext = Readonly<{
  _tag: "Patch"
  durationMs: number
  message: Option.Option<Message>
  model: Model
  thresholdMs: number
}>
```

### SlowSubscriptionDependenciesContext

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L253)

```
/** Context provided when subscription dependency extraction exceeds its configured time budget. */
type SlowSubscriptionDependenciesContext = Readonly<{
  _tag: "SubscriptionDependencies"
  durationMs: number
  model: Model
  subscriptionKey: string
  thresholdMs: number
}>
```

### SlowThresholdOverrides

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L278)

```
/** Budget overrides for slow warning phases. Omitted fields use Foldkit defaults. */
type SlowThresholdOverrides = Readonly<{
  Patch: number
  SubscriptionDependencies: number
  Update: number
  View: number
}>
```

### SlowUpdateContext

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L234)

```
/** Context provided when update exceeds its configured time budget. */
type SlowUpdateContext = Readonly<{
  _tag: "Update"
  durationMs: number
  message: Message
  nextModel: Model
  previousModel: Model
  thresholdMs: number
}>
```

### SlowViewContext

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L225)

```
/** Context provided when view construction exceeds its configured time budget. */
type SlowViewContext = Readonly<{
  _tag: "View"
  durationMs: number
  message: Option.Option<Message>
  model: Model
  thresholdMs: number
}>
```

### ViewTransitionConfig

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/viewTransition.ts#L36)

```
/**
 * Decides, per render, whether the DOM update should run inside a View
 *  Transition. The predicate is total over the Message union, so animation is
 *  opted into per Message: return `true` for the ones worth animating and
 *  `false` for everything else.
 */
type ViewTransitionConfig = (context: ViewTransitionContext<Model, Message>) => ViewTransitionDecision
```

### ViewTransitionContext

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/viewTransition.ts#L18)

```
/**
 * Context passed to the `viewTransition` predicate before each live render.
 * 
 *  A View Transition animates between two states, so the predicate is given
 *  both. `previousModel` is the Model behind the DOM on screen, the state the
 *  browser is about to snapshot. `model` is the Model the pending render will
 *  paint. Direction is the pair: comparing the two says where the application
 *  is going and where it came from, with no route history kept in the Model.
 * 
 *  `previousModel` is always a real painted state rather than an `Option`. The
 *  predicate only runs once a Message has dirtied the Model, and the initial
 *  render completes before any Message is processed.
 * 
 *  `message` is the Message that dirtied the Model for this frame. The runtime
 *  coalesces a burst of dispatches into one render frame, so the predicate
 *  sees the last dirtying Message before the frame.
 */
type ViewTransitionContext = Readonly<{
  message: Message
  model: Model
  previousModel: Model
}>
```

### ViewTransitionDecision

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/viewTransition.ts#L28)

```
/**
 * What the `viewTransition` predicate returns for a render. `false` renders
 *  plainly. `true` wraps the render in `document.startViewTransition`.
 *  `{ types }` additionally tags the transition so CSS can scope animations
 *  via `:active-view-transition-type(...)`.
 */
type ViewTransitionDecision = boolean | Readonly<{
  types: ReadonlyArray<string>
}>
```

### Visibility

type

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L130)

```
/** Controls when a feature is shown. */
type Visibility = "Development" | "Always"
```

## Constants

### SlowPhase

const

[source](https://github.com/foldkit/foldkit/blob/ce6e12fe473f674bed6fcf9964c1b3edf81d4a13/packages/foldkit/src/runtime/runtime.ts#L269)

```
/** Phase names measured by the slow warning runtime option. */
const SlowPhase: Literals<readonly ["Update", "View", "Patch", "SubscriptionDependencies"]>
```
