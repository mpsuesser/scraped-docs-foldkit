---
url: https://foldkit.dev/core/view-transitions
title: "View Transitions"
description: "Animated renders via the View Transitions API, with direction-aware types and shared-element morphs."
access_date: 2026-08-03T18:13:14.939Z
current_date: 2026-08-03T18:13:14.939Z
---

# View Transitions

## Overview

The browser’s View Transitions API animates between two DOM states. The browser snapshots the page, you mutate the DOM inside a callback, and CSS animates from the old snapshot to the new state: cross-fades, slides, and shared-element morphs, all off the main thread.

The API expects the DOM update to happen inside `document.startViewTransition(callback)`. Foldkit owns the render schedule, so application code has no correct place to make that call. The `viewTransition` option on `makeApplication` and `makeElement` closes the gap: when a render qualifies, the runtime calls `startViewTransition` itself and performs that render inside the callback. It defaults to `undefined`, so nothing animates until an application passes a predicate.

## Animating Route Changes

Pass a predicate. Before each render it decides whether that render should animate. It is a total function over your Message union, so animation is opted into one Message at a time. Returning `true` only for `ChangedUrl` animates navigation and leaves every other Message, whatever your application has, rendering plainly:

```
import { Runtime } from 'foldkit'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  container: document.getElementById('root'),
  routing: {
    onUrlRequest: request => ClickedLink({ request }),
    onUrlChange: url => ChangedUrl({ url }),
  },
  viewTransition: ({ message }) => message._tag === 'ChangedUrl',
})

Runtime.run(application)
```

Return `false` for a plain render, exactly as cheap as before. Return `true` to wrap the render in a transition. With no extra CSS, the browser cross-fades the whole page.

The context carries two Models alongside `message`, because a transition is between two states. `previousModel` is the Model behind the DOM on screen, the state the browser is about to snapshot. `model` is the one the pending render will paint: `update` has already run by the time the predicate is asked.

## Direction-Aware Types

Return `{ types }` instead of `true` to tag the transition. Direction is the pair, so hand the two routes to [Route.Transition](https://foldkit.dev/api-reference/route) and match on what the navigation entered:

```
import { Match as M, Option } from 'effect'
import { Runtime } from 'foldkit'
import { Transition } from 'foldkit/route'

export const viewTransition: Runtime.ViewTransitionConfig<Model, Message> = ({
  previousModel,
  model,
  message,
}) => {
  if (message._tag !== 'ChangedUrl') {
    return false
  }

  const transition = Transition.make(previousModel.route, model.route)

  // Every arm past the guard is a navigation, so none of them return `false`.
  // `true` is "animate, with no direction to declare": an untyped transition
  // still cross-fades, it just does not slide. Moving between two artworks
  // enters no route at all, which is `onNone`. Adding a route to AppRoute is a
  // compile error here until it is given a direction.
  return Option.match(Transition.enteredAny(transition), {
    onNone: () => true,
    onSome: M.type<AppRoute>().pipe(
      M.withReturnType<Runtime.ViewTransitionDecision>(),
      M.tagsExhaustive({
        Artwork: () => ({ types: ['to-artwork-detail'] }),
        Gallery: () => ({ types: ['to-gallery'] }),
        NotFound: () => true,
      }),
    ),
  })
}
```

CSS scopes animations to the active types with `:active-view-transition-type(...)`, so drilling into a detail page slides one way and returning slides back:

```
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 300ms;
}

:root:active-view-transition-type(to-artwork-detail)::view-transition-old(
    root
  ) {
  animation-name: slide-out-to-left;
}

:root:active-view-transition-type(to-artwork-detail)::view-transition-new(
    root
  ) {
  animation-name: slide-in-from-right;
}

:root:active-view-transition-type(to-gallery)::view-transition-old(root) {
  animation-name: slide-out-to-right;
}

:root:active-view-transition-type(to-gallery)::view-transition-new(root) {
  animation-name: slide-in-from-left;
}
```

## Shared Elements

Give an element a `viewTransitionName` with `h.Style` and the browser pairs elements that share a name across the old and new states, morphing position, size, and shape between them. A gallery card growing into a detail hero needs nothing more than the same name on both sides:

```
import type { Html, HtmlBuilder } from 'foldkit/html'

const artworkCardView = (artwork: Artwork, h: HtmlBuilder<Message>): Html =>
  h.a(
    [h.Href(artworkRouter({ artworkId: artwork.id }))],
    [
      h.div(
        [
          h.Class('aspect-square rounded-xl'),
          h.Style({ viewTransitionName: `artwork-${artwork.id}` }),
        ],
        [],
      ),
    ],
  )

// The hero keeps the card's aspect ratio. A transition interpolates the two
// snapshots while it interpolates the box, so a square growing into a wider
// box visibly stretches mid-flight.
const artworkHeroView = (artwork: Artwork, h: HtmlBuilder<Message>): Html =>
  h.div(
    [
      h.Class('aspect-square w-full rounded-2xl'),
      h.Style({ viewTransitionName: `artwork-${artwork.id}` }),
    ],
    [],
  )
```

Names must be unique

A `viewTransitionName` may appear at most once per snapshot. Derive names from stable Model identifiers, the same way you key list items.

## When Transitions Are Skipped

The predicate decides which renders animate, and the runtime falls through to a plain synchronous render whenever a transition cannot or should not run:

- The browser does not support the View Transitions API. Browsers that support the API but not transition types run the transition untyped.
- The user has set `prefers-reduced-motion: reduce`.
- The render is not a Live render: DevTools time-travel replays, crash views, and the initial render never animate.
- The predicate returns `false`.
- `document.startViewTransition` refuses to start, which some browsers do for a document that is not fully active. The frame renders plainly rather than losing its patch.

A transition already running is skipped when a later qualifying render supersedes it, so a second navigation landing mid-animation cuts to the new one instead of queueing behind it. Its DOM update still happens, so nothing is lost. A render the predicate declines does not skip a running transition; it simply paints underneath.

Two scheduling details are worth knowing. First, the runtime coalesces a burst of Messages into one render frame, so the predicate sees the last dirtying Message before the frame. That is the right Message for navigation, which is what transitions are for. Second, the runtime never blocks on the transition, so follow-up renders paint immediately. The render runs inside the transition’s DOM update phase, and renders that land mid-animation are picked up live by the new-state snapshot.

See the [view-transitions example](https://foldkit.dev/example-apps/view-transitions) for direction-aware route slides, a shared-element gallery, and a filter input that never animates.
