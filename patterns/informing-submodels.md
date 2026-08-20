---
url: https://foldkit.dev/patterns/informing-submodels
title: "Informing Submodels"
description: "Relay a change a Submodel does not own (a URL, a server push, an auth change) through a helper it exposes, so it can update its own state in response."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Informing Submodels

## Parent-Owned Changes

A Submodel sometimes needs to react to a change it does not own. The URL may change, a server may push new data, or a sibling field may establish a new constraint. The parent observes that change, but the child still owns the state transition it causes.

Export an `inform*` helper from the child. The helper runs an internal child Message through update and returns the next child Model and Commands. The parent folds that helper with `Update.foldChild`, so it never imports or constructs the internal Message.

Use `inform*` when the child must derive a transition or return Commands. Use a silent [`reflect*` helper](https://foldkit.dev/core/submodel#reflecting-external-state) when the child only needs to conform one of its values to an external source.

The example below uses routing. A People Submodel owns its search input, results, and recent searches. The root owns the Route. When the URL resolves to a People Route, the root calls `People.informRouteChanged`.

Prerequisite

This page builds on the [Submodels](https://foldkit.dev/core/submodel) pattern. Read that first if the `Got*Message` wrapping convention is unfamiliar.

## The Child

People declares an internal `ChangedRoute` Message carrying only `PeopleRoute`, the part of the application Route that the feature understands.

Update copies the route query into the input, records it in recent searches, and returns `FetchPeople` for the new results.

`informRouteChanged` is the public entry point. It calls `update(model, ChangedRoute({ route }))`, keeping the Message constructor private.

```
import { Match as M, Option, Schema as S, String } from 'effect'
import { Command } from 'foldkit'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { PeopleRoute } from '../route'

// MESSAGE

const Person = S.Struct({ id: S.Number, name: S.String, role: S.String })

const ChangedSearchInput = m('ChangedSearchInput', { value: S.String })
const SubmittedSearch = m('SubmittedSearch')
const ChangedRoute = m('ChangedRoute', { route: PeopleRoute })
const SucceededFetchPeople = m('SucceededFetchPeople', {
  query: S.String,
  people: S.Array(Person),
})

export const Message = S.Union([
  ChangedSearchInput,
  SubmittedSearch,
  ChangedRoute,
  SucceededFetchPeople,
])
export type Message = typeof Message.Type

// UPDATE

export const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    M.tagsExhaustive({
      ChangedSearchInput: ({ value }) => [
        evo(model, { searchInput: () => value }),
        [],
      ],

      SubmittedSearch: () => [
        model,
        [
          PushSearchUrl({
            searchText: Option.liftPredicate(
              model.searchInput,
              String.isNonEmpty,
            ),
          }),
        ],
      ],

      ChangedRoute: ({ route }) => {
        const searchText = Option.getOrElse(route.searchText, () => '')
        return [
          evo(model, {
            searchInput: () => searchText,
            searchHistory: searchHistory =>
              addSearchToHistory(searchHistory, searchText),
            results: () => SearchLoading(),
          }),
          [FetchPeople({ searchText })],
        ]
      },

      SucceededFetchPeople: ({ query, people }) => [
        evo(model, { results: () => SearchLoaded({ query, people }) }),
        [],
      ],
    }),
  )

export const informRouteChanged = (model: Model, route: PeopleRoute) =>
  update(model, ChangedRoute({ route }))
```

Not an OutMessage

`ChangedRoute` moves from parent to child through an `inform*` helper. An [OutMessage](https://foldkit.dev/core/submodel#surfacing-facts) moves a fact from child to parent.

## The Parent

The root defines one fold for regular People Messages and another for `informRouteChanged`. Both folds use the same `read`, `write`, and `toParentMessage` boundary. The `ChangedUrl` handler stores the next Route, then composes the relevant child step with `Update.combine`.

```
import { Match as M, Option } from 'effect'
import { Update } from 'foldkit'
import { evo } from 'foldkit/struct'

import { People } from './page'

const foldPeople = Update.foldChild({
  update: People.update,
  read: (model: Model) => Option.some(model.peoplePage),
  write: (model, nextPeoplePage) =>
    evo(model, { peoplePage: () => nextPeoplePage }),
  toParentMessage: message => GotPeopleMessage({ message }),
})

const foldPeopleRouteChanged = Update.foldChild({
  update: People.informRouteChanged,
  read: (model: Model) => Option.some(model.peoplePage),
  write: (model, nextPeoplePage) =>
    evo(model, { peoplePage: () => nextPeoplePage }),
  toParentMessage: message => GotPeopleMessage({ message }),
})

const setRoute =
  (nextRoute: AppRoute): Update.Step<Model, Message> =>
  model => [evo(model, { route: () => nextRoute }), []]

export const update = (model: Model, message: Message): UpdateReturn =>
  M.value(message).pipe(
    M.tagsExhaustive({
      ChangedUrl: ({ url }) => {
        const nextRoute = urlToAppRoute(url)

        const routeSteps = M.value(nextRoute).pipe(
          M.withReturnType<ReadonlyArray<Update.Step<Model, Message>>>(),
          M.tag('People', peopleRoute => [foldPeopleRouteChanged(peopleRoute)]),
          M.orElse(() => []),
        )

        return Update.combine(model, [setRoute(nextRoute), ...routeSteps])
      },

      GotPeopleMessage: ({ message }) => foldPeople(model, message),
    }),
  )
```

Multiple Submodels

When several page Submodels react to routing, match the next Route and return the `informRouteChanged` step for the page that owns that Route.

Cold loads

`informRouteChanged` handles later URL changes. On a cold load, root init parses the initial URL and passes the People Route to child init. See [Cold Loads and the Initial Route](https://foldkit.dev/core/routing-and-navigation#cold-loads).

The [Routing example](https://foldkit.dev/example-apps/routing) contains the complete search flow. [Routing and Navigation](https://foldkit.dev/core/routing-and-navigation) covers the parser that produces these Routes.

The same pattern works when a [Subscription](https://foldkit.dev/core/subscriptions) or [Command](https://foldkit.dev/core/commands) tells the parent about a change the child must process. The cause changes, but ownership does not.
