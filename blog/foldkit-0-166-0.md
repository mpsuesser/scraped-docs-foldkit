---
url: https://foldkit.dev/blog/foldkit-0-166-0
title: "Foldkit 0.165.0 and 0.166.0"
description: "Effect 4 stable, an experimental Query module for remote data, and improvements to child lifecycles, DevTools, and Vite."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

[← Blog](https://foldkit.dev/blog)

# Foldkit 0.165.0 and 0.166.0

October 4, 2026 · Devin Jameson

Foldkit 0.165.0 and 0.166.0 bring Effect 4 stable and an experimental Query module for remote data. They also improve child lifecycles, DevTools, and Vite.

## Query

The experimental Query module lets you define a Submodel that manages remote data through loading, failure, and refresh. The parent update decides when to load or refresh, and existing data stays available during a refresh.

With an existing `Post` Schema and `fetchPosts` Effect, include the Query's Model and Messages in the parent:

**Defining the posts Submodel**

```typescript
import { Schema } from 'effect'
import { Query } from 'foldkit/experimental'
import { defineMessageUnion } from 'foldkit/message'

const postsQuery = Query.define({
  name: 'Posts',
  data: Schema.Array(Post),
  error: Schema.String,
  execute: fetchPosts,
})

const Model = Schema.Struct({
  posts: postsQuery.Model,
})
type Model = typeof Model.Type

const Message = defineMessageUnion({
  GotPostsMessage: { message: postsQuery.Message },
  ClickedRefreshPosts: {},
})
type Message = typeof Message.Type
```

The parent starts the first load in init and requests a refresh when the button is clicked. Its `GotPostsMessage` branch passes each fetch result back to Query:

**Loading and refreshing posts**

```typescript
import { Update } from 'foldkit'

const posts = postsQuery.lift<Model, Message>({
  parentField: 'posts',
  toParentMessage: message => Message.GotPostsMessage({ message }),
})

const init = () => {
  const model = Model.make({ posts: postsQuery.init() })

  return posts.loadIfMissing(model)
}

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    GotPostsMessage: ({ message }) => posts.fold(model, message),
    ClickedRefreshPosts: () => posts.revalidateOrLoad(model),
  })
```

The view reads an `AsyncData` value. `matchData` keeps rendering the posts while a refresh runs:

**Rendering posts during a refresh**

```typescript
import { Array } from 'effect'
import { AsyncData } from 'foldkit'
import { type Document, type HtmlBuilder } from 'foldkit/html'

const view = (model: Model, h: HtmlBuilder<Message>): Document => ({
  title: 'Posts',
  body: h.div(
    [],
    [
      h.button([h.OnClick(Message.ClickedRefreshPosts())], ['Refresh']),
      AsyncData.matchData(postsQuery.read(model.posts), {
        onEmpty: () => h.p([], ['Loading posts…']),
        onFailure: error => h.p([], [error]),
        onData: posts =>
          h.ul(
            [],
            Array.map(posts, post => h.keyed('li')(post.id, [], [post.title])),
          ),
      }),
    ],
  ),
})
```

Add request arguments to create a KeyedQuery, which keeps separate results for resources such as posts by ID. The new [API Cache Query example](https://foldkit.dev/example-apps/api-cache-query) shows a list of posts, details by post ID, and statistics refreshed by a Subscription.

Query is available from `foldkit/experimental`. See the [Query guide](https://foldkit.dev/core/query) for the API.

Thank you to [@rodygosset](https://github.com/rodygosset) for [contributing Query](https://github.com/foldkit/foldkit/pull/1425) and the example!

## Effect 4 stable

Foldkit 0.165.0 updates the framework and its companion packages to Effect `4.0.0`. New SPA, SSG, and SSR projects from `create-foldkit-app` use Effect 4 stable too.

The LiveStore example is temporarily unavailable while its pinned upstream snapshot still uses prerelease Effect imports.

## More in these releases

- `Subscription.lift` now supports optional child Models, as `ManagedResource.lift` already does. Foldkit stops the child's Subscriptions when that child is absent.
- DevTools replaces Effect `Redacted` values with placeholders in its UI and MCP responses. The [DevTools MCP guide](https://foldkit.dev/ai/mcp) explains what a connected client can inspect.
- Foldkit's Vite plugin now bundles installed libraries that depend on Foldkit during server rendering, avoiding a second Foldkit copy. Thank you to [@filipfalcon](https://github.com/filipfalcon) for [reporting](https://github.com/foldkit/foldkit/issues/1562) and [fixing the issue](https://github.com/foldkit/foldkit/pull/1564)!
- The Vite plugin prevents duplicate Effect instances in development, and the Foldkit Oxlint plugin supports `effect-oxlint` 0.4. Thank you to [@birbprophet](https://github.com/birbprophet) for the [Vite](https://github.com/foldkit/foldkit/pull/1566) and [Oxlint](https://github.com/foldkit/foldkit/pull/1569) fixes!
- Documentation examples now have titles and copy controls. Long examples can be expanded when needed.

## Upgrading

Upgrade your Effect packages to `4.0.0` together. Move HTTP, persistence, RPC, reactivity, and CLI imports to their stable paths.

In 0.166.0, rename the child accessor in `Subscription.lift` and `ManagedResource.lift` to `read`. The Subscription reader now returns an `Option`; use `Option.some` for an always-present child. DevTools 0.166.0 requires Foldkit 0.166.0 or newer.

The full [0.165.0](https://github.com/foldkit/foldkit/releases/tag/foldkit%400.165.0) and [0.166.0](https://github.com/foldkit/foldkit/releases/tag/foldkit%400.166.0) release notes cover the remaining migration details.

Thanks to everyone building with Foldkit!

Devin
