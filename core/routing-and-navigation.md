---
url: https://foldkit.dev/core/routing-and-navigation
title: "Routing and Navigation"
description: "Type-safe routing with bidirectional parser combinators in Foldkit. URLs parse into typed routes and build back. No string matching, built on Effect-TS."
access_date: 2026-08-03T18:13:14.939Z
current_date: 2026-08-03T18:13:14.939Z
---

# Routing & Navigation

Foldkit uses a bidirectional routing system where you define routes once and use them for both parsing URLs and building URLs. No more keeping route matchers and URL builders in sync. This page introduces the pieces in the order you reach for them; the [Route API reference](https://foldkit.dev/api-reference/route) has the exhaustive catalog of combinators, and the [Route Transition API reference](https://foldkit.dev/api-reference/route-transition) covers the transition helpers.

## The Biparser Approach

Most routers make you define routes twice: once for matching URLs, and again for generating them. This leads to duplication and bugs when they get out of sync.

Foldkit’s routing is based on biparsers: parsers that work in both directions. A single route definition handles:

- `/people/42` → `PersonRoute { personId: 42 }` (parsing)
- `PersonRoute { personId: 42 }` → `/people/42` (building)

This symmetry means if you can parse a URL into data, you can always build that data back into the same URL.

## Defining Routes

Routes are defined as tagged unions using [Effect Schema](https://effect.website/docs/schema/introduction/). Each route variant carries the data extracted from the URL.

```
import { Schema as S } from 'effect'
import { r } from 'foldkit/route'

const HomeRoute = r('Home')
const PeopleRoute = r('People', { searchText: S.Option(S.String) })
const PersonRoute = r('Person', { personId: S.Number })
const NotFoundRoute = r('NotFound', { path: S.String })

const AppRoute = S.Union([HomeRoute, PeopleRoute, PersonRoute, NotFoundRoute])

type AppRoute = typeof AppRoute.Type
```

- `HomeRoute`: no parameters
- `PersonRoute`: holds a `personId: number`
- `PeopleRoute`: holds an optional `searchText: Option<string>`
- `NotFoundRoute`: holds the unmatched `path: string`

## Building Routers

Routers are built by composing small primitives. Each primitive is a biparser that handles one part of the URL.

```
import { Schema as S, pipe } from 'effect'
import { Route } from 'foldkit'
import { int, literal, slash } from 'foldkit/route'

// Matches: /
const homeRouter = pipe(Route.root, Route.mapTo(HomeRoute))

// Matches: /people or /people?searchText=alice
const peopleRouter = pipe(
  literal('people'),
  Route.query(
    S.Struct({
      searchText: S.OptionFromOptional(S.String),
    }),
  ),
  Route.mapTo(PeopleRoute),
)

// Matches: /people/42
const personRouter = pipe(
  literal('people'),
  slash(int('personId')),
  Route.mapTo(PersonRoute),
)
```

The primitives:

- `literal('people')`: matches the exact segment `people`
- `int('personId')`: captures an integer parameter
- `string('name')`: captures a string parameter
- `schemaSegment('personId', PersonId)`: captures a segment decoded through a Schema
- `rest('path')`: captures all remaining segments
- `restString('path')`: captures all remaining segments as one path string
- `slash(...)`: chains path segments together
- `Route.query(Schema)`: adds query parameter parsing
- `Route.mapTo(RouteType)`: converts parsed data into a typed route

## Parsing URLs

Combine routers with `Route.oneOf` and create a parser with a fallback for unmatched URLs.

```
import { Route, Runtime } from 'foldkit'
import { evo } from 'foldkit/struct'
import { Url } from 'foldkit/url'

// Combine routers. A route matches only when it consumes the whole URL.
const routeParser = Route.oneOf(
  personRouter, // /people/:id
  peopleRouter, // /people?search=...
  homeRouter, // /
)

// Create a parser with a fallback for unmatched URLs
const urlToAppRoute = Route.parseUrlWithFallback(routeParser, NotFoundRoute)

// In your init function, parse the initial URL:
const init: Runtime.RoutingApplicationInit<Model, Message> = (url: Url) => {
  return [{ route: urlToAppRoute(url) }, []]
}

// In your update function, handle URL changes:
ChangedUrl: ({ url }) => [
  evo(model, {
    route: () => urlToAppRoute(url),
  }),
  [],
]
```

A router only matches when it consumes the entire URL, so routes that share a prefix do not conflict. `/people` and `/people/:id` can appear in any order. When several routes fully match the same URL, the first one wins. That only happens when route shapes overlap, like a `literal('new')` page next to a `string('username')` profile: `/users/new` satisfies both, so list the literal route first.

## Building URLs

Here’s where the biparser pays off. The same router that parses URLs can build them:

```
// Building URLs from route data - same router, opposite direction!

const homeUrl = homeRouter()
console.log(homeUrl)
// '/'

const peopleUrl = peopleRouter({ searchText: Option.none() })
console.log(peopleUrl)
// '/people'

const searchUrl = peopleRouter({
  searchText: Option.some('alice'),
})
console.log(searchUrl)
// '/people?searchText=alice'

const personUrl = personRouter({ personId: 42 })
console.log(personUrl)
// '/people/42'

// Use in your view to create type-safe links:
a([Href(personRouter({ personId: person.id }))], [person.name])
```

TypeScript ensures you provide the correct data. If `personRouter` expects `{ personId: number }`, you can’t accidentally pass a string or forget the parameter.

## Query Parameters

Query parameters use [Effect Schema](https://effect.website/docs/schema/introduction/) for validation. This gives you type-safe parsing, optional parameters, and automatic encoding/decoding.

```
import { Schema as S, pipe } from 'effect'
import { Route } from 'foldkit'
import { literal } from 'foldkit/route'

// Query parameters use Effect Schema for validation
const searchRouter = pipe(
  literal('search'),
  Route.query(
    S.Struct({
      q: S.OptionFromOptional(S.String),
      page: S.OptionFromOptional(S.FiniteFromString),
      sort: S.OptionFromOptional(S.Literals(['Asc', 'Desc'])),
    }),
  ),
  Route.mapTo(SearchRoute),
)

// Parsing /search?q=hello&page=2&sort=asc gives you:
// → SearchRoute { q: Some('hello'), page: Some(2), sort: Some('Asc') }

// Building
const searchUrl = searchRouter({
  q: Option.some('hello'),
  page: Option.some(2),
  sort: Option.none(),
})
console.log(searchUrl)
// '/search?q=hello&page=2'
```

`S.OptionFromOptional` makes parameters optional. Missing params become `Option.none()`. `S.FiniteFromString` automatically parses string query values into numbers.

For a complete routing example, see the [Routing example](https://foldkit.dev/example-apps/routing). For a deeper look at query parameters (custom schema transforms, lenient parsing, and bidirectional URL sync), see the [Query Sync example](https://foldkit.dev/example-apps/query-sync).

## Schema Segments

`int` and `string` capture a segment as a bare `number` or `string`. When a segment is really a domain id, `schemaSegment` decodes it through an [Effect Schema](https://effect.website/docs/schema/introduction/) instead, so the route carries the schema’s type. A branded `PersonId` flows straight into the Model, where it can’t be passed anywhere a different id or a bare `number` is expected.

```
import { Schema as S, pipe } from 'effect'
import { Route } from 'foldkit'
import { literal, r, schemaSegment, slash } from 'foldkit/route'

// A branded id: structurally a number, but its own type. The brand stops it
// from being mixed up with another number, like an OrderId or a count.
const PersonId = S.FiniteFromString.pipe(S.brand('PersonId'))
type PersonId = typeof PersonId.Type

const PersonRoute = r('Person', { personId: PersonId })

// int('personId') captures a bare number. schemaSegment decodes the segment
// through the schema, so the route carries a PersonId instead.
//
// Parses: /people/42  →  PersonRoute { personId: PersonId(42) }
const personRouter = pipe(
  literal('people'),
  slash(schemaSegment('personId', PersonId)),
  Route.mapTo(PersonRoute),
)

// Builds: /people/42. The brand is required, so a bare number or a different
// id type is a compile error.
const personUrl = personRouter({ personId: PersonId.make(42) })
```

Whether a segment decodes is the route’s match test, and the decoded value is what the route carries when it passes. `int` already works this way: it claims `/users/42` but not `/users/banana`. `schemaSegment` generalizes that to any rule a schema can express, from a UUID pattern to a fixed set of string literals. Refine a `ProductId` to a UUID and the route matches a real one but declines `/products/banana`, so a malformed id falls through to the next route in `oneOf` (or to not-found) rather than reaching a component that has to handle it. Refinement and a brand compose, so one segment is both validated and carried as a distinct type.

```
import { Schema as S, pipe } from 'effect'
import { Route } from 'foldkit'
import { literal, r, schemaSegment, slash } from 'foldkit/route'

// A refinement, not a transform: the value stays a string, but the route only
// matches when the segment is actually a UUID. The brand rides along, so the
// model carries a ProductId distinct from any other string.
const UUID_PATTERN =
  /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i
const ProductId = S.String.check(S.isPattern(UUID_PATTERN)).pipe(
  S.brand('ProductId'),
)
type ProductId = typeof ProductId.Type

const ProductRoute = r('Product', { productId: ProductId })

// Matches /products/<uuid>. /products/banana does not match, so in oneOf it
// falls through to the next route, or to not-found.
const productRouter = pipe(
  literal('products'),
  slash(schemaSegment('productId', ProductId)),
  Route.mapTo(ProductRoute),
)

// Building still round-trips: a ProductId prints straight back into the path.
const productUrl = productRouter({
  productId: ProductId.make('3f2504e0-4f89-41d3-9a0c-0305e82c3301'),
})
```

The schema’s encoded form must be a single segment string, and `schemaSegment` runs it both ways: it decodes when parsing and encodes when building, so the route still round-trips. For values that span several segments use `rest`, and for values in the query string use `Route.query`.

## Rest Segments

Some routes carry a whole path as data: a file tree, a documentation page, a breadcrumb trail. `rest` captures every remaining segment as a named field, the feature other routers call catch-all or splat routes. The parsed value is a non-empty array of strings, so the route schema declares the field with `S.NonEmptyArray(S.String)`.

```
import { Schema as S, pipe } from 'effect'
import { Route } from 'foldkit'
import { literal, r, rest, slash } from 'foldkit/route'

const FilesIndexRoute = r('FilesIndex')
const FilesRoute = r('Files', { path: S.NonEmptyArray(S.String) })

// Matches: /files
const filesIndexRouter = pipe(literal('files'), Route.mapTo(FilesIndexRoute))

// Matches: /files/documents/taxes/2024.pdf
// path: ['documents', 'taxes', '2024.pdf']
const filesRouter = pipe(
  literal('files'),
  slash(rest('path')),
  Route.mapTo(FilesRoute),
)

// Builds: /files/documents/taxes
const taxesUrl = filesRouter({ path: ['documents', 'taxes'] })
```

`rest` requires at least one segment, so the bare prefix `/files` does not match the rest route. Give the prefix its own route, like `FilesIndexRoute` above. The two never overlap: one matches exactly `/files`, the other matches anything beneath it.

A specific route under the same prefix is different. The rest route also matches every URL that `literal('files'), slash(literal('shared'))` accepts, so in `oneOf` the specific route must come first.

Nothing can follow `rest` in the path, so `slash` cannot extend it. TypeScript rejects the composition. `query` can still follow, since query parameters live after the path.

When the path itself is the value, `restString` captures the same tail as a single string, slashes included, so the route schema declares the field with `S.String`. A repository-relative file path like `20-upgrade/teach/the-elm-architecture.md` round-trips as one value instead of an array of segments.

```
import { Schema as S, pipe } from 'effect'
import { Route } from 'foldkit'
import { literal, r, restString, slash } from 'foldkit/route'

const VaultIndexRoute = r('VaultIndex')
const VaultNoteRoute = r('VaultNote', { path: S.String })

// Matches: /vault
const vaultIndexRouter = pipe(literal('vault'), Route.mapTo(VaultIndexRoute))

// Matches: /vault/20-upgrade/teach/the-elm-architecture.md
// path: '20-upgrade/teach/the-elm-architecture.md'
const vaultNoteRouter = pipe(
  literal('vault'),
  slash(restString('path')),
  Route.mapTo(VaultNoteRoute),
)

// Builds: /vault/20-upgrade/teach/the-elm-architecture.md
const noteUrl = vaultNoteRouter({
  path: '20-upgrade/teach/the-elm-architecture.md',
})
```

Everything above about `rest` applies to `restString` as well: it requires at least one segment, a more specific route under the same prefix must come first in `oneOf`, and nothing can follow it in the path. Building requires a normalized path, non-empty with no leading, trailing, or repeated slashes. Any other value would build a URL that parses back differently, so the build fails instead.

The [Routing example](https://foldkit.dev/example-apps/routing) uses a rest route to drive a small file browser, building breadcrumb and directory links from the captured segments.

## Route View Identity

Each route arm delegates to its own view function, and view functions are identity boundaries: the build brands the VNodes a function returns with that function’s identity, and the differ replaces a position whose identity changed instead of patching it. Navigating from one route to another therefore tears down the old page and builds the new one fresh, with no keys and no wrapper elements. The identity is stamped by `@foldkit/vite-plugin`, which `create-foldkit-app` includes by default. Do not build a Foldkit app without it:

```
import { Match as M } from 'effect'
import type { Document, HtmlBuilder } from 'foldkit/html'

const view = (model: Model, h: HtmlBuilder<Message>): Document => {
  const routeContent = M.value(model.route).pipe(
    M.tagsExhaustive({
      Products: () => productsView(model, h),
      Cart: () => cartView(model, h),
      Checkout: () => checkoutView(model, h),
      NotFound: ({ path }) => notFoundView(path, h),
    }),
  )

  return {
    title: `${model.route._tag} | Shop`,
    body: h.div(
      [],
      [
        h.header([], [navigationView(model.route, h)]),
        h.main([], [routeContent]),
      ],
    ),
  }
}
```

Route views are the most common branch, but the same protection applies to any control flow that selects between view functions. See [Keying](https://foldkit.dev/best-practices/keying) in Best Practices for the full identity model, list keys, and the edges that remain manual.

## Navigation

Foldkit provides navigation Commands for programmatically changing the URL. These are returned from your update function like any other Command.

```
import { Effect, Schema as S } from 'effect'
import { Command, Navigation } from 'foldkit'
import { m } from 'foldkit/message'

const CompletedNavigateInternal = m('CompletedNavigateInternal')
const CompletedReplaceUrl = m('CompletedReplaceUrl')
const CompletedGoBack = m('CompletedGoBack')
const CompletedGoForward = m('CompletedGoForward')
const CompletedLoadExternal = m('CompletedLoadExternal')
const CompletedOpenUrl = m('CompletedOpenUrl')

const Message = S.Union([
  CompletedNavigateInternal,
  CompletedReplaceUrl,
  CompletedGoBack,
  CompletedGoForward,
  CompletedLoadExternal,
  CompletedOpenUrl,
])
type Message = typeof Message.Type

const NavigateInternal = Command.define('NavigateInternal', {
  args: { url: S.String },
  messages: [CompletedNavigateInternal],
  execute: ({ url }) =>
    Navigation.pushUrl(url).pipe(Effect.as(CompletedNavigateInternal())),
})

const ReplaceUrl = Command.define('ReplaceUrl', {
  args: { url: S.String },
  messages: [CompletedReplaceUrl],
  execute: ({ url }) =>
    Navigation.replaceUrl(url).pipe(Effect.as(CompletedReplaceUrl())),
})

const GoBack = Command.define('GoBack', {
  messages: [CompletedGoBack],
  execute: Navigation.back().pipe(Effect.as(CompletedGoBack())),
})

const GoForward = Command.define('GoForward', {
  messages: [CompletedGoForward],
  execute: Navigation.forward().pipe(Effect.as(CompletedGoForward())),
})

const LoadExternal = Command.define('LoadExternal', {
  args: { href: S.String },
  messages: [CompletedLoadExternal],
  execute: ({ href }) =>
    Navigation.load(href).pipe(Effect.as(CompletedLoadExternal())),
})

const OpenUrl = Command.define('OpenUrl', {
  args: { url: S.String },
  messages: [CompletedOpenUrl],
  execute: ({ url }) =>
    Navigation.openUrl(url).pipe(Effect.as(CompletedOpenUrl())),
})
```

- `Navigation.pushUrl`: adds a new entry to browser history
- `Navigation.replaceUrl`: replaces the current history entry (no back button)
- `Navigation.back` / `Navigation.forward`: navigate through browser history
- `Navigation.load`: full page load (for external URLs)
- `Navigation.openUrl`: opens an external URL in a new browsing context (tab or window), leaving the current page untouched

When a link is clicked in your application, the `routing.onUrlRequest` handler receives either an Internal or External request. Handle Internal links with `pushUrl` and External links with `load`:

```
import { Effect, Match as M, Schema as S, pipe } from 'effect'
import { Command, Navigation, Route, Url } from 'foldkit'
import { m } from 'foldkit/message'
import { int, literal, r, slash } from 'foldkit/route'
import { evo } from 'foldkit/struct'

// ROUTE

const HomeRoute = r('Home')
const PersonRoute = r('Person', { personId: S.Number })
const NotFoundRoute = r('NotFound', { path: S.String })
const AppRoute = S.Union([HomeRoute, PersonRoute, NotFoundRoute])
type AppRoute = typeof AppRoute.Type

const homeRouter = pipe(Route.root, Route.mapTo(HomeRoute))
const personRouter = pipe(
  literal('people'),
  slash(int('personId')),
  Route.mapTo(PersonRoute),
)
const routeParser = Route.oneOf(personRouter, homeRouter)
const urlToAppRoute = Route.parseUrlWithFallback(routeParser, NotFoundRoute)

// MODEL

const Model = S.Struct({ route: AppRoute })
type Model = typeof Model.Type

// MESSAGE

const CompletedNavigateInternal = m('CompletedNavigateInternal')
const CompletedLoadExternal = m('CompletedLoadExternal')
const ClickedLink = m('ClickedLink', { request: Navigation.UrlRequest })
const ChangedUrl = m('ChangedUrl', { url: Url.Url })

const Message = S.Union([
  CompletedNavigateInternal,
  CompletedLoadExternal,
  ClickedLink,
  ChangedUrl,
])
type Message = typeof Message.Type

// COMMAND

const NavigateInternal = Command.define('NavigateInternal', {
  args: { url: S.String },
  messages: [CompletedNavigateInternal],
  execute: ({ url }) =>
    Navigation.pushUrl(url).pipe(Effect.as(CompletedNavigateInternal())),
})

const LoadExternal = Command.define('LoadExternal', {
  args: { href: S.String },
  messages: [CompletedLoadExternal],
  execute: ({ href }) =>
    Navigation.load(href).pipe(Effect.as(CompletedLoadExternal())),
})

// UPDATE

const update = (model: Model, message: Message) =>
  M.value(message).pipe(
    M.withReturnType<
      readonly [Model, ReadonlyArray<Command.Command<Message>>]
    >(),
    M.tagsExhaustive({
      CompletedNavigateInternal: () => [model, []],
      CompletedLoadExternal: () => [model, []],

      ClickedLink: ({ request }) =>
        M.value(request).pipe(
          M.tagsExhaustive({
            Internal: ({
              url,
            }): readonly [Model, ReadonlyArray<Command.Command<Message>>] => [
              model,
              [NavigateInternal({ url: Url.toString(url) })],
            ],
            External: ({
              href,
            }): readonly [Model, ReadonlyArray<Command.Command<Message>>] => [
              model,
              [LoadExternal({ href })],
            ],
          }),
        ),

      ChangedUrl: ({ url }) => [
        evo(model, {
          route: () => urlToAppRoute(url),
        }),
        [],
      ],
    }),
  )
```

After `pushUrl` or `replaceUrl` changes the URL, Foldkit automatically calls your `routing.onUrlChange` handler with the new URL. This is where you parse the URL into a route and update your model.

## Cold Loads and the Initial Route

`onUrlChange` fires when the URL changes after boot. On a cold load (a direct visit, a bookmark, a reload) there is no change to report: `init` receives the initial URL, parses it, and seeds the Model with the starting route. Foldkit does not synthesize a `ChangedUrl` for it, because the initial route is starting state, not a transition.

Don’t wire route fetches into navigation alone.

A fetch Command returned only from the `ChangedUrl` handler fires on every in-app navigation and never on a cold load. During development you reach every route by clicking from the home page, so everything works. Then a user reloads on a sub-route or follows a bookmark and lands on a Model stuck in its initial state, with no fetch in flight.

Both code paths resolve a URL into a route, and both should produce the same route-driven Commands. Factor those Commands into one helper and call it from both places:

```
import { Match as M, Option } from 'effect'
import { Command, Runtime } from 'foldkit'
import { evo } from 'foldkit/struct'
import { Url } from 'foldkit/url'

// Route-driven Commands live in one helper...
const commandsForRoute = (
  route: AppRoute,
): ReadonlyArray<Command.Command<Message>> =>
  M.value(route).pipe(
    M.withReturnType<ReadonlyArray<Command.Command<Message>>>(),
    M.tag('People', ({ searchText }) => [
      FetchPeople({ searchText: Option.getOrElse(searchText, () => '') }),
    ]),
    M.orElse(() => []),
  )

// ...which init calls for the cold load...
const init: Runtime.RoutingApplicationInit<Model, Message> = (url: Url) => {
  const route = urlToAppRoute(url)
  return [{ route }, commandsForRoute(route)]
}

// ...and the ChangedUrl handler calls for in-app navigation:
ChangedUrl: ({ url }) => {
  const route = urlToAppRoute(url)
  return [evo(model, { route: () => route }), commandsForRoute(route)]
}
```

When the route-driven state lives in a Submodel, the same factoring follows the Submodel boundary instead of a shared helper: the Submodel’s `init(route)` seeds its state and returns the boot Commands for the cold load, and its `informRouteChanged` helper covers later transitions. [Informing Submodels](https://foldkit.dev/patterns/informing-submodels) shows that shape, and the [Routing example](https://foldkit.dev/example-apps/routing) runs on it.

## Route Transitions

The shared helper above answers what a route needs, so its Commands fire on every navigation that lands on the route. For `FetchPeople` that is the point: every search text is a new query. Other Commands should run once when the user arrives, loading a filter catalog, starting a poll, recording a page view. For those the route alone cannot answer the real question: did this navigation enter the route, or was the application already there?

The `Transition` namespace in `foldkit/route` answers it. A `Transition.Transition` carries both halves of the question: the route the application was on and the route it is on now. `Transition.make(previousRoute, nextRoute)` builds the navigation case, `Transition.coldLoad(nextRoute)` builds the cold load, where there is no previous route, and `Transition.isEntering` asks the question: a transition enters a route when the next route carries the tag and the previous route did not, and a cold load counts as an entry. Navigating within a route, between two ids of one detail route or two search texts of one list route, is not an entry.

The route union is inferred from the transition argument and the tag is checked against it, so a misspelled route name fails to compile. Build the transition in the same two places that resolve a URL into a route: `init` holds no route yet, so it builds the cold load, and the `ChangedUrl` handler transitions from the route the Model still holds:

```
import { Command, Runtime } from 'foldkit'
import { Transition } from 'foldkit/route'
import { evo } from 'foldkit/struct'
import { Url } from 'foldkit/url'

// Entry-only Commands ask about the transition, not the route alone...
const commandsForTransition = (
  transition: Transition.Transition<AppRoute>,
): ReadonlyArray<Command.Command<Message>> =>
  Transition.isEntering(transition, 'People') ? [FetchPeopleFilters()] : []

// ...init builds the cold load transition, which counts as an entry...
const init: Runtime.RoutingApplicationInit<Model, Message> = (url: Url) => {
  const route = urlToAppRoute(url)
  return [{ route }, commandsForTransition(Transition.coldLoad(route))]
}

// ...and the ChangedUrl handler transitions from the route the Model holds:
ChangedUrl: ({ url }) => {
  const nextRoute = urlToAppRoute(url)
  return [
    evo(model, { route: () => nextRoute }),
    commandsForTransition(Transition.make(model.route, nextRoute)),
  ]
}
```

A predicate answers whether, not which. When the entry Command needs the route’s payload, `Transition.entered(transition, tag)` returns the entered route narrowed to the tag, so a detail id arrives typed:

```
import { Option } from 'effect'
import { Command } from 'foldkit'
import { Transition } from 'foldkit/route'

type Commands = ReadonlyArray<Command.Command<Message>>

const commandsForTransition = (
  transition: Transition.Transition<AppRoute>,
): Commands =>
  Option.match(Transition.entered(transition, 'Person'), {
    onNone: () => [],
    onSome: ({ personId }) => [FetchPerson({ personId })],
  })
```

Every helper that takes a tag answers for one named route. When several routes have entry Commands, ask the transition which route it entered instead: `Transition.enteredAny` returns the entered route in a `Some`, whichever route that was, and `Option.none()` when the transition stayed within one route. Match on the result to dispatch every entry policy in one place:

```
import { Match as M, Option } from 'effect'
import { Command } from 'foldkit'
import { Transition } from 'foldkit/route'

type Commands = ReadonlyArray<Command.Command<Message>>

const commandsForTransition = (
  transition: Transition.Transition<AppRoute>,
): Commands =>
  Option.match(Transition.enteredAny(transition), {
    onNone: () => [],
    onSome: M.type<AppRoute>().pipe(
      M.withReturnType<Commands>(),
      M.tag('People', () => [FetchPeopleFilters()]),
      M.tag('Person', ({ personId }) => [FetchPerson({ personId })]),
      M.orElse(() => []),
    ),
  })
```

Entering has a mirror. `Transition.exited(transition, tag)` returns the route the transition left, narrowed to the tag, and `Transition.exitedAny` is its whichever-route form. Exits are for one-shot Commands on the way out, saving a draft, recording that a visit ended. They are not for tearing down things that live while a route is active: listeners, timers, and handles belong to a [Subscription](https://foldkit.dev/core/subscriptions) or [ManagedResource](https://foldkit.dev/core/managed-resources) condition on the Model, which also ends them when the route state disappears for reasons other than navigation.

The last case is staying. `Transition.stayed(transition, tag)` returns both sides of a within-route navigation, narrowed to the tag: `Some({ previousRoute, nextRoute })` when the transition stayed on that route, `Option.none()` when it entered it, left it, or never touched it. A cold load stays nowhere. Reach for it when the previous payload matters, comparing a detail id or diffing query parameters; when only the next value matters, the `ChangedUrl` handler already has the next route. Staying has no whichever-route form: without a tag the two sides could not narrow to the same route variant together, so matching on one would leave the other typed as the whole union.

```
import { Array, Option } from 'effect'
import { Command } from 'foldkit'
import { Transition } from 'foldkit/route'

type Commands = ReadonlyArray<Command.Command<Message>>

// Leaving a route is a fact too: one-shot Commands on the way out...
const commandsOnExit = (
  transition: Transition.Transition<AppRoute>,
): Commands =>
  Option.match(Transition.exited(transition, 'Person'), {
    onNone: () => [],
    onSome: ({ personId }) => [RecordVisitEnded({ personId })],
  })

// ...and stayed hands you both sides of a within-route change:
const commandsOnPersonChange = (
  transition: Transition.Transition<AppRoute>,
): Commands =>
  Option.match(Transition.stayed(transition, 'Person'), {
    onNone: () => [],
    onSome: ({ previousRoute, nextRoute }) =>
      previousRoute.personId === nextRoute.personId
        ? []
        : [FetchPerson({ personId: nextRoute.personId })],
  })

// Transition helpers compose by concatenation
const commandsForTransition = (
  transition: Transition.Transition<AppRoute>,
): Commands =>
  Array.flatten([
    commandsOnExit(transition),
    commandsOnPersonChange(transition),
  ])
```

Because a cold load counts as an entry, `init` and the `ChangedUrl` handler share one load-on-entry policy: reloading on `/people` runs the same entry Commands as clicking there from the home page. Transition helpers compose by concatenation, as above; a handler that mixes entry, exit, and per-navigation Commands flattens their results into one batch.

The [Route Transition API reference](https://foldkit.dev/api-reference/route-transition) lists every helper with its full signature.
