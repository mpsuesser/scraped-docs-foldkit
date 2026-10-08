---
url: https://foldkit.dev/core/server-rendering
title: "Server Rendering"
description: "Render the same application to HTML for request-time SSR or build-time SSG, then hydrate it in place through a validated build-id and Flags handoff."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

## Overview

Experimental

Server rendering ships from `foldkit/experimental/server` while its API and operational contract settle. It will move to `foldkit/server` once Foldkit can make a stable compatibility commitment to both. It may change in any Foldkit release and has not yet had broad production exposure.

Pin the exact version you deploy and test upgrades against your own SSR or SSG host. For regulated or security-critical workloads, wait for the stable export, or have your security and deployment setup reviewed independently first.

The lowest-risk use today is statically generated public content, which is how this site uses it. Please try it and report what breaks.

Foldkit renders on the server with the same program the browser runs. `renderToString` resolves `init`, runs the pure view, and returns HTML. `Runtime.hydrate` then adopts matching HTML in place and rebuilds mismatches. The same init, view, update, and Model work whether the HTML was rendered during a build (SSG) or while handling a request (SSR).

One program gives Foldkit one rendering pipeline with two delivery policies:

- **Static site generation (SSG):** a build script renders a finite set of URLs and writes HTML files.
- **Server-side rendering (SSR):** a server renders a URL when its request arrives.

An application can use either policy, or use SSG for some URLs and SSR for others. The application code does not need a second rendering API.

For an application with Flags, here is the handoff from server input to a live application:

```
SERVER OR BUILD                      BROWSER

request or build input                 live Foldkit app
         |                                     ^
         v                                     |
       Flags                           adopts matching DOM
         |                                     ^
         v                                     |
        init                                same view
         |                                     ^
         v                                     |
       Model                          equivalent Model
         |                                     ^
         v                                     |
        view                               same init
         |                                     ^
         v                                     |
HTML + serialized Flags --------> Runtime.hydrate reads Flags
```

Once the live application takes over, it behaves like any other Foldkit application. Routing, update, Commands, and Subscriptions run in the browser. A handled navigation does not ask the delivery host to render another document, though the application's Commands may still request data. The server renders the document again only on a full page load, such as a reload or a link the runtime does not handle.

For SSG, the build script takes the host's place. It writes the response to a file that a static server or CDN delivers later.

## The server entry

A server entry connects the application to its host. It exports a `renderPage` function that accepts a Web `Request` and returns a `Promise<EntryResult>`:

Server entry

The outer `Promise` keeps `renderPage` callable from Vite, build scripts, serverless functions, and the emitted `fetch` handler. Those hosts do not need to provide the application's Effect requirements. The entry uses Effect internally; the host sees only the `Promise`.

The entry is application code. Keep it in `src/` (`src/entry.server.ts` in the examples), not in the host's directory. It imports the application's `init`, `view`, and `Flags`, so the server build must compile it with those application imports.

An SSR build also requires `ssr.clientEntry` next to `ssr.serverEntry`, such as `'/src/entry.ts'`. The client entry names the browser script the generated document loads. Import the application's stylesheets from that script so Vite includes them in the browser build.

### Module resolution

The client and server are separate module graphs. Within each graph, the view and the Foldkit runtime that calls it must resolve to one `foldkit` module instance. The HTML builder tracks a render in module-level state. If one render uses two Foldkit copies, the view writes to one copy while the runtime reads the other. The render fails instead of producing the wrong page. Duplicate monorepo installs and aliases that split one graph are common causes.

In server builds and in every server environment of the dev server, `@foldkit/vite-plugin` bundles `foldkit`, `@foldkit/ui`, and `@foldkit/devtools`, plus every installed package whose `dependencies` or `peerDependencies` include `foldkit` or an `@foldkit/*` package, such as `@foldkit/markdown`. Those packages then run against the one Foldkit copy inside the server bundle. In a Node server environment of the dev server, these packages run through Vite's module runner instead of Node's own import. The plugin finds them by crawling from the application's `package.json`. The crawl follows:

- The application's `dependencies` and `devDependencies`.
- The `dependencies` of each package it bundles.
- The `devDependencies` of a bundled package that is a private workspace package.

A package the crawl does not reach stays external. For example: a peer the application does not declare, or a package reached only through a package that does not depend on Foldkit. Such a package loads a second Foldkit copy from `node_modules` at runtime. Declare it in the application's `package.json`, or add it to `resolve.noExternal`, which Vite applies to every environment.

### Rendering a route

A delivery host runs the built `fetch` handler. It does not import the application and render it directly. One `vite build` emits `dist/server/fetch.js` whose default export is `{ fetch }`. The [SSR example](https://foldkit.dev/example-apps/ssr) starts that module with `node scripts/serve.ts`. A Worker can default-export the same module.

`renderToString` accepts the server-relevant subset of a `makeApplication` config. That subset contains `init` and `view`, plus `Flags` and `routing` when the application declares them. A full application config satisfies the subset, so an entry can pass it unchanged.

The `container`, `update`, `subscriptions`, and `managedResources` fields do not participate in server rendering. The server runs the view once over the Model returned by `init`. There is no DOM to attach to and no Message to dispatch.

For a routing application, pass the request URL so `init` receives the same value it receives from `window.location` in the browser:

Using renderToString with a URL

```typescript
Server.renderToString(config, {
  url: request.url,
  flags,
})
```

`request.url` is the public URL. The Vite dev host preserves its configured `base` prefix and the browser's query string when middleware routes the request.

## The result contract

### Entry results

A server entry returns one of two variants:

- `Server.Rendered(application, options)` supplies the rendered application to the entry's document renderer. Its options can carry an HTTP status and headers.
- `Server.Responded(response)` bypasses document rendering with a complete Web `Response`. Use it for redirects and any request that does not render a page.

`Server.toResponse(document, result)` turns either variant into the Web `Response` the host sends. Supply a function that produces a complete document, or an HTML template when the host owns one. A `Rendered` result defaults to status 200 and a UTF-8 HTML content type. A `Responded` result passes through unchanged without calling the document renderer.

The render host serves pages, not a data API. Put JSON endpoints on a separate backend, such as an Effect `HttpApi` service.

### Rendered application

The rendered application contains the body markup and the Document's initial head state:

RenderedApplication type

```typescript
type RenderedApplication = Readonly<{
  html: string
  title: string
  lang?: string
  dir?: 'ltr' | 'rtl' | 'auto'
  canonical?: string
  ogUrl?: string
}>
```

### The document renderer

An SSR build calls the server entry's `renderDocument` export with the rendered application and its browser assets. The default `Server.renderDocument` supplies UTF-8 and viewport metadata, the application's Document fields, stylesheets, module preloads, and the client script. Request-time rendering and prerendering use the same renderer.

The plugin supplies `assets.entryScript`, `assets.stylesheets`, and `assets.modulePreloads`. Import stylesheets from the client script; Vite emits their production URLs and keeps lazy imports out of the initial document. Use a root-relative `clientEntry` and an absolute-path or full-URL Vite `base`. Relative bases cannot locate the same assets consistently on nested routes and are rejected.

Wrap the default renderer to set a default language or add trusted author-owned head markup. For example, this entry adds a favicon:

Custom document head

```typescript
import { Server } from 'foldkit/experimental'

export const renderDocument: Server.DocumentRenderer = (application, assets) =>
  Server.renderDocument(application, assets, {
    head: '<link rel="icon" href="/favicon.svg">',
  })
```

The `head` option accepts HTML. Escape request-derived values before interpolating them. The application's language overrides the renderer's default `lang` option.

### Custom HTML templates

For a host that owns an HTML template, `injectIntoTemplate` places the rendered application in its placeholder. The template must contain exactly one `<div id="root"></div>` placeholder with no other attributes or whitespace inside it, and its head must contain exactly one `<title>`. Pass `containerId` when the placeholder uses another id. A missing or duplicate placeholder or title produces an error that names the problem.

The injector also writes the language, text direction, canonical URL, and Open Graph URL into corresponding shell elements. For a Vite development host whose template comes from another build pipeline, omit `clientEntry` and `ssr.build`.

### Protocol validation

`RenderedApplication` is public so a host can transport or wrap it. Its `html` field remains protocol data. Pass the value returned by `renderToString` to `injectIntoTemplate` unchanged.

Hydratable HTML must parse as one top-level element with one nonempty application stamp and build stamp. It may be followed by one matching top-level JSON Flags script. Static HTML may contain one element, text, or comment root, or no body output.

Foldkit rejects extra top-level content, ambiguous handoff markers, and source that the HTML parser drops, splits, moves, or reconstructs. It does not insert markup when parsing changes which nodes belong to the application.

### Application ownership

`runtimeId` pairs one hydratable root with its Flags payload. It also keys the preserved Model and scroll position. A nondefault id changes that pairing. It does not create another document owner.

A document may contain one hydratable Foldkit root. `injectIntoTemplate` refuses to insert a hydratable render when the template already contains one, even when the ids differ. `Runtime.hydrate` refuses and contains a page assembled elsewhere when it finds more than one stamped root. An explicit container does not override this rule.

Two roots with the same id would also read the same Flags and share Model and scroll preservation. Foldkit reports that collision specifically, but distinct ids do not make multiple page-owning applications valid.

The root stamp must have a nonempty id and name the document's single stamped root in the body light DOM. A requested root in `<head>`, a shadow tree, a detached subtree, or another document is refused and the page is contained before startup. When the configured container resolves to an element, it must be that root or one of its descendants.

A page-owning `makeApplication` controls the document title, language, text direction, canonical URL, and Open Graph URL. It also installs document-wide navigation listeners. With two applications, the last render would own the metadata and the first listener would handle every link. Render one application per page.

Static body output carries no handoff stamp. It may coexist with the document's one hydratable application. Each call to `injectIntoTemplate` still applies that render's `Document` head fields, so insertion order decides which render supplies the initial page metadata.

### Supported templates and roots

The placeholder's location and the view's root are part of the contract. Browsers move or drop markup that appears in an invalid parser context. Foldkit supports a short list of predictable contexts and refuses the rest. Each error names the rejected tag:

- The placeholder must reach `<body>` through `div`, `main`, `section`, `article`, `aside`, `header`, or `footer`. A placeholder inside `<form>`, `<table>`, `<select>`, SVG or MathML content, or `<template>` content is rejected.
- Rendered markup cannot declare a shadow root through `<template shadowrootmode>` or the older `shadowroot` attribute. Parsing moves that content out of the light DOM, so the browser tree and the hydration tree would differ. Attach shadow roots from a custom element instead.
- A view cannot be rooted at `<html>`, `<head>`, `<body>`, or `<frameset>`. `renderToString` rejects those roots for static and hydratable output because the document parser drops, merges, or replaces them. Root the view at an ordinary element such as `<div>` or `<main>`. Set the title, language, and text direction through the `Document` returned by the view.

## The hydration handoff

### Handoff markers

A hydratable render carries these markers:

- The application root has `data-foldkit-app`. Its value is the `runtimeId`.
- The root also has `data-foldkit-build`. Its value identifies the deployment that rendered the page. The client removes it as it takes the root over, so its absence is the signal that the client runtime owns the page; a browser test can wait for `[data-foldkit-build]` to disappear. A refused handoff keeps the stamp and adds `data-foldkit-refused`.
- An application with Flags emits a `<script type="application/json" data-foldkit-flags="...">`. It carries the Schema-encoded Flags that produced the server Model. The attribute value matches the root's `runtimeId`.
- Keyed elements carry `data-foldkit-key`. Elements with build-assigned view identity carry `data-foldkit-identity`. Both values are deterministic, non-cryptographic fingerprints. Neither marker contains the original key, which may hold an account id or email address, or the build's source path. Hydration compares each fingerprint and removes the marker as it adopts the element. A render with `isHydratable: false` emits neither marker.
	A fingerprint is a public comparison token, not a secret or an authentication check. A reader can compute the fingerprint of a guessed key or view identity and test for a match. An attacker can also construct two values with the same fingerprint. Key by values that are safe to publish. Hydratable keys must be strings or numbers other than `NaN`.

Conceptually, the handoff appears next to the rendered root:

Hydration handoff markup

```html
<main data-foldkit-app="app"><!-- rendered view --></main>
<script type="application/json" data-foldkit-flags="app">
  { "initialCount": 2 }
</script>
```

The script type makes the payload data rather than executable JavaScript. Foldkit escapes values that could close the script element. Hydration then parses and Schema-decodes the text. Flags are public HTML, not a place for secrets.

### Opting in from the client entry

The client opts into the handoff in its entry (`src/entry.ts` in the examples):

Hydrating the application

```typescript
Runtime.hydrate(application)
```

`Runtime.run` always builds the DOM from scratch. An application with Flags supplies its client-only Flags Effect at that boundary:

Runtime.run with flags

```typescript
Runtime.run(application, { flags })
```

`Runtime.hydrate` accepts no client Flags producer. It reads the serialized Flags, calls the same `init`, and adopts matching server DOM nodes. Element identity, focus, scroll position, and media state survive while listeners and Mounts attach.

A mismatched subtree is rebuilt from its nearest parent. Rebuilding discards the DOM identity and browser state that adoption preserves. Development logs a warning that points to nondeterministic Flags, `init`, or view output. Production rebuilds silently, so test hydration before shipping.

Calling `hydrate` declares that a complete server handoff exists. If the handoff is invalid, startup stops before Foldkit adopts DOM and the page is put out of reach. [What a refusal does](#what-a-refusal-does) describes that state. This is safer than booting a different client Model over the server's HTML.

Use `run` from a separate client entry when the page must also support a fresh SPA boot.

`isHydratable` defaults to `true` for SSR and SSG. Set `isHydratable: false` only for static markup that no client will hydrate. The output then carries no application stamp, build id, Flags payload, key marker, or identity marker. `Runtime.hydrate` refuses it.

### Flags and what only the browser knows

Hydration requires the server and browser to build the same first Model. Embedded Flags let the browser call `init` with the values the server used.

Request-time SSR can derive Flags from the request, including the URL, headers, and cookies. Build-time SSG writes one file for every visitor, so its Flags must be universal and fixed at build time.

Flags are public

Every serialized Flag ships in the page's HTML. Never place credentials, private tokens, or other secrets in Flags.

Browser-only facts do not belong in hydratable SSG Flags. For example: a theme stored in `localStorage`, the viewport width, and browser feature detection are unknown during the build. Start with a neutral Model on both sides. Load browser facts through a boot-time Command or Subscription after hydration.

When a preference must affect the server HTML, make it request-visible, such as through a cookie, and use request-time SSR for that URL.

### The build id

The build id does not make hydration correct. It makes hydration refuse when it would otherwise be incorrect.

The server stamps the id on the rendered root. The client bundle carries the same value. Hydration compares them before it accesses the Flags payload text or adopts DOM. Different ids stop startup; matching ids allow hydration to continue.

Most structural mismatches are safe because Foldkit rebuilds the affected subtree. The dangerous case is markup that has the same shape but a different meaning. For example: an old page may place `<input name="email">` where the new build places `<input name="ssn">`. Without a build check, hydration could preserve text entered before startup and submit it under the new field name.

Flags create the same risk. A payload belongs to the deployment that rendered it. A new Schema may accept the old data even when its values now mean something different.

`@foldkit/vite-plugin` generates one opaque id when a Vite app build coordinates the client and server artifacts. It compiles that id into Foldkit in both artifacts, so `Runtime.hydrate(application)` and `Server.renderToString(config, options)` use it without application forwarding.

Set an explicit override only when the artifacts build in separate jobs, or when the id should name a deployment in another system. Use the plugin's `buildId` option or the `FOLDKIT_BUILD_ID` environment variable, and give every job the same value:

Build id override

```typescript
// vite.config.ts: give separate build jobs the same deployment id.
foldkit({ buildId: process.env.DEPLOYMENT_ID })
```

Whatever value you pick, three things have to be true:

- It is public. The id appears in the HTML sent to every visitor, so it must not contain a secret.
- It identifies one deployment. Reusing an override makes a stale page look current and produces no warning. A unique CI deployment id is a good source. A commit SHA or release tag is enough only when every deployment carrying it has identical rendering inputs.
- It reaches both artifacts. One Vite app build handles this automatically. Separate build jobs must receive the same explicit override.

A hydratable render without a compiled or explicit id fails with `MissingBuildId`. `Runtime.hydrate` refuses hydration on the same terms. A static render with `isHydratable: false` needs none.

The development server generates an opaque id for its own client and server transforms. A second dev server receives a different id, so a page from one session is not accepted by the other.

### Why view identity cannot replace the build id

A view identity names a module path and function. It does not capture imported constants, configuration, or caller arguments.

View identity also ships in the client bundle. Adding a source hash would expose a digest of that source to every visitor. A reader could test candidates for a low-entropy server-only value by hashing each one, even when the client build removed the value itself. An opaque build id detects skew without hashing source files.

## Request-time SSR

### Development host

In development, enable the Vite host in `vite.config.ts`:

Vite SSR configuration

```typescript
foldkit({
  ssr: {
    clientEntry: '/src/entry.ts',
    serverEntry: '/src/entry.server.ts',
  },
})
```

Vite continues to serve the client entry, HMR, and assets. Requests that reach Foldkit become Web `Request` values and pass to `renderPage`. The returned Web `Response` provides the status, headers, and body.

A development reload does not exercise hydration. Foldkit restores the Model but rebuilds the DOM under the root. That DOM came from code that predates the edit. Refresh the page manually to test hydration itself. The stamped root remains required during a development reload; without it, startup fails as it would on a fresh load.

### Production host

In production, the host is built alongside the client. Set `ssr.build` and `ssr.clientEntry` in the plugin and `vite build` produces both. The server bundle is a Web `fetch` handler: Node and Workers both run it. Static files stay the platform's job. Build-time Vite `transformIndexHtml` hooks do not run for this script-input build. Put document tags in `renderDocument` and import CSS from `clientEntry`.

The [SSR example](https://github.com/foldkit/foldkit/tree/main/examples/ssr) starts that handler on Node:

SSR build configuration

```typescript
foldkit({
  buildId,
  ssr: {
    serverEntry: '/src/entry.server.ts',
    clientEntry: '/src/entry.ts',
    build: true,
  },
})
```

`@foldkit/node` is the standard Node host. It reads `foldkit.build.json` to find the client assets and Fetch handler, serves static files for `GET` and `HEAD`, then sends the remaining requests to the handler. `port` selects the listening port. `origin` is the public origin that Node uses to turn each raw request target into a Web `Request` URL.

Start the Node adapter

```typescript
import { Config, Option, String } from 'effect'

import { NodeRuntime } from '@effect/platform-node'
import * as Node from '@foldkit/node'

const port = Config.withDefault(Config.Port('PORT'), 3000)
const origin = Config.option(Config.String('ORIGIN')).pipe(
  Config.map(Option.filter(String.isNonEmpty)),
)

NodeRuntime.runMain(Node.serve({ port, origin }))
```

The manifest records output paths relative to the Vite root. When the server output is outside that root, pass `rootDirectory` so the adapter can resolve both output directories; it refuses an outside-root server path without it.

Set `basePath` to the same root-relative `base` used by Vite, such as `/app/`. The adapter serves static files only through that path and preserves it when it calls the Fetch handler.

Caching personalized responses

When Flags depend on the request, such as a cookie, authorization header, or locale, the rendered HTML belongs to that visitor. Set `cache-control` and `vary` so a shared cache cannot serve it to someone else. The SSR example uses `private, no-store` and `vary: cookie` because its initial count comes from a cookie.

## Build-time SSG

### Built-in prerendering

Generation is part of the build. `ssr.build.prerender` builds the browser bundle and the server entry, then calls `renderPage` once for every path the entry lists and passes each result to `renderDocument` before writing it as a file, all inside one `vite build`:

SSG build configuration

```typescript
foldkit({
  buildId,
  ssr: {
    serverEntry: '/src/entry.server.ts',
    clientEntry: '/src/entry.ts',
    build: { prerender: true },
  },
})
```

An `ssr.build` build emits no unfilled HTML template. The generated document comes from `renderDocument`; SSG writes `index.html` only for paths that `prerender` generates.

### Generating more pages

To generate more pages from an `ssr.build` output, call its `fetch` handler with a `Request` for each path. The handler returns a complete document, so the loop needs no template. For example, this loop generates two routes whose server entry is known to return rendered HTML.

SSG render loop over the fetch handler

The `fetch` response does not say whether the entry returned `Rendered` or a complete `Responded` response. A 200 `Responded` result could carry headers that the loop would lose when it writes only the body. Use this loop only for routes whose entry is known to return rendered HTML, and check that your static host can reproduce any response metadata you need. Foldkit's built-in `prerender` can reject a `Responded` result before writing a file.

### Browser-only generation

This website does not set `ssr.build`, so its generation loop has no built `fetch` handler. It calls `renderPage` and injects each result into the browser build's template.

SSG render loop over a browser build

```typescript
for (const path of prerenderPaths) {
  const request = new Request(\`https://example.com${path}\`)
  const result = await serverEntry.renderPage(request)

  if (result._tag === 'Responded') {
    throw new Error(\`Cannot write a Response for ${path} as static HTML\`)
  }

  const html = Server.injectIntoTemplate(template, result.application)
  await writeRoute(path, html)
}
```

The browser-only loop must keep a copy of the built template outside `dist/client`. Generating `/` replaces `dist/client/index.html` with a rendered page. On a later run, use the saved copy if that file no longer contains `<div id="root"></div>`. Reading the file at the start of each run is not enough: after the first run, it is already a page, and `injectIntoTemplate` cannot find the placeholder.

A static HTML file cannot preserve a redirect, a 404, or per-response headers. This is why built-in `prerender` refuses `Responded`, non-200 statuses, and explicit headers rather than writing their bodies as ordinary pages.

The [SSG example](https://github.com/foldkit/foldkit/tree/main/examples/ssg) is the minimal reference. This website is the production-scale reference. Its prerender host uses the same `renderPage(Request)` contract, seeds route content through universal Flags, and writes every route as hydratable static HTML.

## Deploying

A deployed SSG build is a directory of static files. Any static host or CDN can serve it as is. The hydration handoff already lives in the HTML.

A build that `@foldkit/vite-plugin` owns writes `foldkit.build.json` beside the server bundle. It names the two output directories, the server entry, and every generated path. An SSR host can serve those files and send requests that match no file to the server. A static-only SSG host serves the generated files and leaves other paths as misses.

A deployed SSR application needs a host that serves the built client assets and calls `fetch` for page requests. The build writes no fallback document. Send requests that match no file to `fetch`; do not enable a single-page-application fallback that answers those requests with a file. `@foldkit/node` provides that policy for Node and derives the output directories from `foldkit.build.json`.

### Custom hosts

Use a custom host when the deployment needs routing or delivery behavior that `@foldkit/node` does not own. The adapter does not choose a `Cache-Control` policy or application 404 behavior. A custom Node host can make those choices around the same Fetch handler.

A custom host must validate a raw request target against its configured origin before static-file lookup, serve static files only for `GET` and `HEAD`, and send the Vite-base `index.html` path, such as `/app/index.html` for `/app/`, to the Fetch handler instead of a generated file. Static lookup must also stay under that base path, so an unprefixed application request reaches the Fetch handler. The [adapter source](https://github.com/foldkit/foldkit/tree/main/packages/node/src/node.ts) is a reference for that boundary.

### Reading completed build metadata

A deployment integration that runs Vite in process can read the `foldkit:build` plugin's `api` after `await builder.buildApp()` succeeds. Its `getBuildMetadata()` method returns a frozen, serializable `FoldkitBuildMetadata` snapshot with absolute `root`, `clientDirectory`, `serverDirectory`, and emitted `serverEntry` paths, plus the same `manifest` data written to disk. These paths follow the resolved Vite environments, including host overrides.
