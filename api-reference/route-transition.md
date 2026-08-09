---
url: https://foldkit.dev/api-reference/route-transition
title: "Route/Transition"
description: "API documentation for the Route/Transition module."
access_date: 2026-08-09T19:15:24.712Z
current_date: 2026-08-09T19:15:24.712Z
---

# Route/Transition

## Functions

### coldLoad

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L28)

```
/**
 * Builds the Transition for a cold load (a direct visit, a
 *  bookmark, a reload). There is no previous route, so isEntering
 *  counts the transition as an entry.
 */
<Route>(nextRoute: Route): Transition<Route>
```

### entered

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L62)

```
/**
 * The route a transition entered, narrowed to the given tag: `Some` of
 *  the entered route when this transition entered it, and `None`
 *  otherwise. The entry Command for a detail route usually needs the
 *  route's payload, and this returns it typed. enteredAny is the
 *  form that answers for every route at once.
 */
<Route extends Readonly<{
  _tag: string
}>, Tag extends string>(
  transition: Transition<Route>,
  routeTag: Tag
): Option<Extract<Route, {
  _tag: Tag
}>>
```

### enteredAny

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L40)

```
/**
 * The route a transition entered, whichever route that was:
 *  `Some(nextRoute)` when the next route's tag differs from the previous
 *  route's (a cold load counts as an entry), `None` when the transition
 *  stayed within one route, for example between two ids of one detail
 *  route. Match on the result to dispatch entry Commands for many routes
 *  in one place. For a single named route, entered narrows to its
 *  tag and isEntering is the plain predicate form.
 */
<Route extends Readonly<{
  _tag: string
}>>(__namedParameters: Transition<Route>): Option<Route>
```

### exited

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L92)

```
/**
 * The route a transition left, narrowed to the given tag: `Some` of the
 *  exited route when this transition left it, and `None` otherwise.
 *  exitedAny is the form that answers for every route at once.
 */
<Route extends Readonly<{
  _tag: string
}>, Tag extends string>(
  transition: Transition<Route>,
  routeTag: Tag
): Option<Extract<Route, {
  _tag: Tag
}>>
```

### exitedAny

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L80)

```
/**
 * The route a transition left, whichever route that was:
 *  `Some(previousRoute)` when the previous route's tag differs from the
 *  next route's, `None` on a cold load or when the transition stayed
 *  within one route. For one-shot Commands on the way out, saving a
 *  draft, recording that a visit ended. Things that live while a route is
 *  active, listeners, timers, handles, belong to a Subscription or
 *  ManagedResource condition on the Model instead, which also ends them
 *  when the route state disappears for reasons other than navigation.
 *  exited narrows to a single tag.
 */
<Route extends Readonly<{
  _tag: string
}>>(__namedParameters: Transition<Route>): Option<Route>
```

### isEntering

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L140)

```
/**
 * Predicate for "this transition entered the given route": the next
 *  route carries the tag and the previous route did not (a cold load
 *  counts as an entry). Navigating within the same route, for example
 *  between two ids of one detail route, is not an entry. The boolean
 *  view of entered: when the entry Command needs the route's
 *  payload, use that instead. The route union is inferred from the
 *  transition argument, so the tag is checked against the union's
 *  tags.
 */
<Route extends Readonly<{
  _tag: string
}>, Tag extends string>(
  transition: Transition<Route>,
  routeTag: Tag
): boolean
```

### make

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L17)

```
/**
 * Builds the Transition for an in-app navigation, from the route
 *  the Model still holds to the route the new URL parsed to. Arguments
 *  read in travel order: previous first, next second.
 */
<Route>(
  previousRoute: Route,
  nextRoute: Route
): Transition<Route>
```

### stayed

function

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L111)

```
/**
 * Both sides of a within-route navigation: `Some` of the previous and
 *  next routes, narrowed to the given tag, when the transition stayed on
 *  that route, and `None` when it entered it, left it, or never touched
 *  it. A cold load stays nowhere. For reacting to payload changes within
 *  one route, a detail id, a search text, when the previous value
 *  matters; when it does not, the ChangedUrl handler already has the next
 *  route. There is no tagless counterpart to this the way
 *  enteredAny answers for entered: without a tag the two
 *  sides of a stay could not narrow to the same route variant together,
 *  so matching on one would leave the other typed as the whole union.
 */
<Route extends Readonly<{
  _tag: string
}>, Tag extends string>(
  transition: Transition<Route>,
  routeTag: Tag
): Option<Readonly<{
  nextRoute: Extract<Route, {
    _tag: Tag
  }>
  previousRoute: Extract<Route, {
    _tag: Tag
  }>
}>>
```

## Types

### Transition

type

[source](https://github.com/foldkit/foldkit/blob/bea1da58f00cc4d43ce880c6e5faca862870ce2d/packages/foldkit/src/route/transition.ts#L9)

```
/**
 * A route change: where the application was and where it is now.
 *  `maybePreviousRoute` is `None` on the first render (a cold load), which
 *  isEntering treats as an entry. Build one with coldLoad
 *  in `init` and with make wherever the application handles its
 *  ChangedUrl Message, then hand it to per-cache step functions so `init`
 *  and navigation share one loading policy.
 */
type Transition = Readonly<{
  maybePreviousRoute: Option.Option<Route>
  nextRoute: Route
}>
```
