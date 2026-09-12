---
url: https://foldkit.dev/api-reference/experimental-server
title: "Experimental/Server"
description: "API documentation for the Experimental/Server module."
access_date: 2026-09-12T18:49:33.387Z
current_date: 2026-09-12T18:49:33.387Z
---

# Experimental/Server

## Functions

### acceptsHtml

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L77)

```
(acceptHeader: string | undefined): boolean
```

### classifyRequest

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L298)

```
(
  requestUrl: string,
  fetchDestination?: string
): RequestClassification
```

### handleRequest

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/fetch.ts#L81)

```
(
  request: Request,
  options: HandleRequestOptions
): Promise<Response>
```

### injectIntoTemplate

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/template.ts#L728)

```
(
  template: string,
  rendered: RenderedApplication,
  options?: Readonly<{
    containerId: string
  }>
): string
```

### isHostSettledMethod

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L400)

```
(method: string): boolean
```

### renderToString

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1366)

### resolveRequestUrl

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L166)

```
(
  requestTarget: string,
  origin: string
): string | undefined
```

### resolvesToIndexHtml

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L344)

```
(requestUrl: string): boolean
```

### toResponse

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/entry.ts#L96)

```
(
  template: string,
  result: EntryResult,
  options?: Readonly<{
    containerId: string
  }>
): Response
```

### varyWith

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L116)

```
(
  existing: string | undefined,
  fieldName: string
): string
```

### varyWithAccept

function

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L146)

```
(existing: string | undefined): string
```

## Types

### ApplicationConfig

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1118)

```
/**
 * Server-side subset of a non-routing `makeApplication` config.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type ApplicationConfig = Readonly<{
  init: () => InitReturn<Model, Message>
  view: (model: Model, h: HtmlBuilder<Message>) => Document
}>
```

### ApplicationConfigWithFlags

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1108)

```
/**
 * Server-side subset of a non-routing `makeApplication` config with Flags.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type ApplicationConfigWithFlags = Readonly<{
  Flags: Schema.Codec<Flags, any, never, never>
  init: (flags: Flags) => InitReturn<Model, Message>
  view: (model: Model, h: HtmlBuilder<Message>) => Document
}>
```

### EntryModule

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/entry.ts#L85)

```
/**
 * The module shape a Foldkit server entry exports for hosts to call: one Web
 * `Request` in, one delivery result out.
 * 
 * The outer boundary is a `Promise` so Vite, build scripts, serverless
 * functions, and long-running HTTP hosts can call it without owning the
 * application's Effect requirements. The entry may use Effect internally and
 * settles that work before returning. Rendering itself must run in the same
 * Foldkit module instance as the application's view because the HTML builder's
 * render frame is module-local.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type EntryModule = Readonly<{
  renderPage: (request: Request) => Promise<EntryResult>
}>
```

### EntryResult

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/entry.ts#L71)

```
/**
 * The result of handling one request through a Foldkit server entry.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type EntryResult = Rendered | Responded
```

### HandleRequestOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/fetch.ts#L17)

```
/**
 * How handleRequest renders a page request.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type HandleRequestOptions = Readonly<{
  containerId: string
  renderPage: (request: Request) => Promise<EntryResult>
  template: string
}>
```

### HydratableRenderOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1140)

```
/**
 * Render options for output a client will hydrate, which is the default. The
 *  build id is required here: hydration compares it against the client's own
 *  before adopting any DOM, and a page carrying none has no such protection.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type HydratableRenderOptions = CommonRenderOptions & Readonly<{
  buildId: string
  isHydratable: true
}>
```

### InjectIntoTemplateOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/template.ts#L677)

```
/**
 * Options for injectIntoTemplate.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type InjectIntoTemplateOptions = Readonly<{
  containerId: string
}>
```

### RenderError

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1070)

```
/**
 * Union of the failures renderToString can produce.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RenderError = MissingBuildId | InvalidUrl | FlagsEncodeError | SerializationError | InvalidRuntimeId | InvalidHydrationRoot
```

### RenderFlagsOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1191)

```
/**
 * Render options for a Flags application, adding the per-request Flags.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RenderFlagsOptions = RenderOptions & Readonly<{
  flags: Flags
}>
```

### RenderOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1176)

```
/**
 * Options for renderToString. `runtimeId` names the application in the
 *  root stamp and Flags payload; it defaults to `'app'` and must be non-empty.
 *  A nondefault `runtimeId` changes the root stamp and the keys used for Flags,
 *  Model, and scroll preservation. It does not permit a second hydratable
 *  application in one document.
 * 
 *  A hydratable render (the default) requires `buildId`. Pass
 *  `isHydratable: false` for static markup that nothing will hydrate, which
 *  takes no build id.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RenderOptions = HydratableRenderOptions | StaticRenderOptions
```

### RenderUrlFlagsOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1201)

```
/**
 * Render options for a routing Flags application: the request URL plus the
 *  per-request Flags.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RenderUrlFlagsOptions = RenderUrlOptions & RenderFlagsOptions<Flags>
```

### RenderUrlOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1182)

```
/**
 * Render options for a routing application, adding the request URL.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RenderUrlOptions = RenderOptions & Readonly<{
  url: string
}>
```

### Rendered

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/entry.ts#L23)

```
/**
 * A server entry result whose application markup still needs to be placed in
 * the host's HTML template.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type Rendered = Readonly<{
  _tag: "Rendered"
  application: RenderedApplication
  headers: HeadersInit
  status: number
}>
```

### RenderedApplication

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L997)

```
/**
 * The server render of one request: the body markup and the `Document` head
 *  fields for the host to place into its HTML template. Hydratable output
 *  contains a stamped root and, when the application declares Flags, its
 *  payload script. Static output carries no handoff markers.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RenderedApplication = Readonly<{
  canonical: string
  dir: "ltr" | "rtl" | "auto"
  html: string
  lang: string
  ogUrl: string
  title: string
}>
```

### RequestClassification

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L281)

```
/**
 * How a host should read a request that no static file answered.
 * 
 * `PathAsset`: the path names an asset, whatever the request headers say, so a
 * refusal is the same for every client and needs no `Vary`.
 * 
 * `DestinationAsset`: the path could be a page, and only the request's
 * `Sec-Fetch-Dest` says otherwise. A refusal here depends on that header, so it
 * has to declare it in `Vary` before a shared cache may store it: a cross-site
 * script request would otherwise seed a cached 404 for a real page.
 * 
 * `Page`: nothing marks it as an asset, so the usual `Accept` negotiation
 * decides.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RequestClassification = "PathAsset" | "DestinationAsset" | "Page"
```

### Responded

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/entry.ts#L53)

```
/**
 * A server entry result that bypasses application rendering and returns a
 * complete Web `Response`, such as a redirect or another non-page response to
 * a page request. A dedicated data API belongs on a separate backend rather
 * than the render host; this is for responses the page request itself
 * resolves without rendering the shell.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type Responded = Readonly<{
  _tag: "Responded"
  response: Response
}>
```

### ResponseOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/entry.ts#L13)

```
/**
 * Optional HTTP metadata for a rendered application.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type ResponseOptions = Readonly<{
  headers: HeadersInit
  status: number
}>
```

### RoutingApplicationConfig

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1098)

```
/**
 * Server-side subset of a routing `makeApplication` config without Flags.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RoutingApplicationConfig = Readonly<{
  init: (url: Url) => InitReturn<Model, Message>
  routing: unknown
  view: (model: Model, h: HtmlBuilder<Message>) => Document
}>
```

### RoutingApplicationConfigWithFlags

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1086)

```
/**
 * Server-side subset of a routing `makeApplication` config with Flags. The
 *  full application config is structurally assignable; `container`, `update`,
 *  and `subscriptions` play no part in a server render.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type RoutingApplicationConfigWithFlags = Readonly<{
  Flags: Schema.Codec<Flags, any, never, never>
  init: (flags: Flags, url: Url) => InitReturn<Model, Message>
  routing: unknown
  view: (model: Model, h: HtmlBuilder<Message>) => Document
}>
```

### StaticRenderOptions

type

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/server.ts#L1158)

```
/**
 * Render options for static markup nothing will hydrate. No build id applies,
 *  because no client will compare one.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
type StaticRenderOptions = CommonRenderOptions & Readonly<{
  buildId: undefined
  isHydratable: false
}>
```

## Constants

### FOLDKIT_APP_ATTRIBUTE

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/hydrationMarker.ts#L5)

```
/**
 * Attribute stamped on a hydratable server-rendered application root. Its
 *  nonempty value is the runtime id used to pair the root with its Flags
 *  payload and scope Model and scroll preservation. `makeApplication` locates
 *  the root; `Runtime.hydrate` adopts it.
 */
const FOLDKIT_APP_ATTRIBUTE: "data-foldkit-app"
```

### FOLDKIT_FLAGS_ATTRIBUTE

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/hydrationMarker.ts#L11)

```
/**
 * Attribute on the JSON script tag carrying the Schema-encoded flags the
 *  server rendered with. A hydrating runtime decodes this payload instead of
 *  running the client `flags` Effect, so both sides call `init` with the
 *  same value.
 */
const FOLDKIT_FLAGS_ATTRIBUTE: "data-foldkit-flags"
```

### HOST_METHOD_ANSWERS

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/host.ts#L386)

```
/**
 * The answer a host gives for the methods it cannot hand to a server entry.
 * 
 * The WHATWG `Request` constructor rejects `CONNECT`, `TRACE`, and `TRACK`, so
 * an entry can never be handed one: forwarding it turns a malformed request
 * into a 500. A host refuses them itself, with `405` and an `Allow` header
 * naming what it does forward.
 * 
 * On Node only `TRACE` reaches this rule. The HTTP parser rejects `TRACK` with
 * a 400 before any handler runs, and `CONNECT` arrives on its own event rather
 * than as an ordinary request. Both are named anyway, so a host on another
 * runtime refuses them rather than handing one to `new Request`.
 * 
 * Every other method reaches the entry, `OPTIONS` included. A preflight is a
 * question about a resource, so the application answers it: the entry can allow
 * one origin for one route and refuse it for another, which no host-level
 * policy could express. The Vite dev host applies Vite's CORS option only to
 * Vite-owned modules and assets. An application preflight therefore reaches
 * the entry rather than middleware a deployed host has no counterpart for.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
const HOST_METHOD_ANSWERS: Readonly<{
  allow: string
  refusedStatus: number
}>
```

### Rendered

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/entry.ts#L23)

```
/**
 * Constructs a rendered server entry result with optional HTTP status and
 * headers.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
const Rendered: (application: RenderedApplication, options?: Readonly<{
  headers: HeadersInit
  status: number
}>) => Rendered
```

### Responded

const

[source](https://github.com/foldkit/foldkit/blob/ba0d6f141325a66ba6608cdd5957c3667ed5ba37/packages/foldkit/src/experimental/server/entry.ts#L53)

```
/**
 * Constructs a complete Web `Response` server entry result.
 * 
 *  Ships from `foldkit/experimental/server`; expect breaking changes while the API settles.
 */
const Responded: (response: Response) => Responded
```
