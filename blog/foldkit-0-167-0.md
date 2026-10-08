---
url: https://foldkit.dev/blog/foldkit-0-167-0
title: "Foldkit 0.167.0"
description: "SSR without an HTML template, a Node host adapter, VirtualList improvements for chat and feeds, Submodel updates by key, and browser Stream helpers moved to Dom."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

[← Blog](https://foldkit.dev/blog)

# Foldkit 0.167.0

October 7, 2026 · Devin Jameson

Foldkit 0.167.0 lets your server entry render the whole HTML page and adds a Node host adapter. It also improves VirtualList for chat and feeds, adds `Update.foldChildAt` for updating one Submodel selected by key, and moves existing browser Stream helpers into `Dom`.

## Server rendering and the Node adapter

SSR and SSG builds now render the whole HTML page from the server entry, replacing the separate `index.html` template. That includes the `<html>`, `<head>`, and `<body>` elements. `Server.renderDocument` supplies the default document, which you can customize with extra head content or a default language. Request-time rendering and prerendering use the same renderer.

The new `@foldkit/node` package hosts a built application on Node, serving its static assets and forwarding application requests to its Fetch handler. New SSR projects from `create-foldkit-app` use the adapter.

The [server rendering guide](https://foldkit.dev/core/server-rendering) covers document rendering, the Node adapter, and custom hosts. The [SSR example](https://foldkit.dev/example-apps/ssr) shows the complete setup.

Thank you to [@filipfalcon](https://github.com/filipfalcon) for proposing [rendering the whole HTML page from the server entry](https://github.com/foldkit/foldkit/issues/1390) and the [Node adapter](https://github.com/foldkit/foldkit/issues/1353)!

## VirtualList for chat and feeds

[VirtualList](https://foldkit.dev/ui/virtual-list) now supports rows that change height and lists that follow appended content. A chat can stay at the bottom as new messages arrive, then preserve the reader's place when they scroll up or load older history.

Try appending messages at the bottom, then scroll up and append another. Click to expand a message and change its height, or load older messages above the viewport:

Conversation

24 messages · Click to expand a message

The [VirtualList guide](https://foldkit.dev/ui/virtual-list#end-anchored-dynamic-heights) explains how this demo handles measured rows and older history.

## Updating a Submodel by key

`Update.foldChildAt` runs a child update for one Submodel selected by key. Define the fold once, then pass it the key and child Message. Here, the parent stores Applicant Submodels in an array, each with a stable entry ID:

**Updating an Applicant Submodel by key**

```typescript
import { Array, Option } from 'effect'
import { Update } from 'foldkit'
import { modifyFields } from 'foldkit/struct'

const foldApplicant = Update.foldChildAt({
  update: Applicant.update,
  readAt: (model: Model, entryId: string) =>
    Option.map(
      Array.findFirst(model.applicants, applicant => applicant.id === entryId),
      applicant => applicant.entry,
    ),
  writeAt: (model, entryId, nextEntry) =>
    modifyFields(model, {
      applicants: Array.map(applicant =>
        applicant.id === entryId
          ? modifyFields(applicant, { entry: () => nextEntry })
          : applicant,
      ),
    }),
  toParentMessage: (entryId, message) =>
    Message.GotApplicantMessage({ entryId, message }),
})

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    GotApplicantMessage: ({ entryId, message }) =>
      foldApplicant(model, entryId, message),
  })
```

`readAt` finds the child Model, `writeAt` stores the next one, and `toParentMessage` carries the key when lifting the child's Command results. If `readAt` returns `None`, the parent Model is unchanged.

This replaces a common pattern where each call to `foldChild` closes over an entry's key. The [job-application example](https://foldkit.dev/example-apps/job-application) uses the new helper for education, work-history, and skills Submodels stored in arrays. The [Submodels guide](https://foldkit.dev/core/submodel#fold-child-at) covers the fold and its OutMessage handling.

The Oxlint plugin also recognizes direct-field boundaries declared through `foldChildAt` and catches empty curried `toParentOutMessage` mappers.

Thank you to [@armancharan](https://github.com/armancharan) for [contributing the helper](https://github.com/foldkit/foldkit/pull/1599)!

## Browser Streams in Dom

Browser event, media query, and key-binding Stream helpers move from `Subscription` to the [Dom module](https://foldkit.dev/core/dom). For example, `Subscription.fromEvent` becomes `Dom.streamFromEvent`, making its role as an Effect Stream constructor explicit.

Subscription entry helpers are also renamed to describe what they return:

Before

After

`Subscription.persistent`

`Subscription.persistentEntry`

`Subscription.animationFrame`

`Subscription.animationFrameEntry`

`Port.subscription`

`Port.subscriptionEntry`

## More in this release

- DevTools can copy Model, Message, Command, and Mount payloads as formatted JSON. The copied value includes collapsed fields, so you can share the whole payload without expanding the inspector tree.
- The Vite plugin applies its dependency and bundling rules in every environment, including named Cloudflare Worker environments. This fixes missing build IDs and duplicate runtime instances in those integrations.

Thank you to [@filipfalcon](https://github.com/filipfalcon) for reporting and fixing the [DevTools overlay](https://github.com/foldkit/foldkit/pull/1593) and [Vite environment](https://github.com/foldkit/foldkit/pull/1591) issues!

See the full [0.167.0 release notes](https://github.com/foldkit/foldkit/releases/tag/foldkit%400.167.0) for all changes, package versions, and upgrade guidance.

Thanks to everyone building with Foldkit!

Devin
