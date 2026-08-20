---
url: https://foldkit.dev/core/view-memoization
title: "View Memoization"
description: "Skip stable view subtrees with createLazy and createKeyedLazy, choose cache keys by entity identity, and profile before adding memoization."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# View Memoization

## Skipping Stable Subtrees

Every Model change calls view again. Foldkit builds the next VNode tree and diffs it against the previous tree. Most views need no extra optimization, but a large subtree that rarely changes can repeat the same construction and diff work on every render.

Foldkit provides two memoization helpers:

- `createLazy` caches a view rendered at one position.
- `createKeyedLazy` gives one view function a separate cache for each list item, entity, or call site.

Each helper caches the VNode returned by a view function. On a later live render, Foldkit reuses it when the function and every argument still have the same references. The DOM differ sees the same VNode object and skips the subtree.

## createLazy

`createLazy` creates one memoization slot. Declare it at module scope, then use it to wrap an expensive subtree rendered at one position.

```
import { type HtmlBuilder, createLazy } from 'foldkit/html'

// Define the view function at module level for a stable reference.
// If defined inside the view, a new function is created each render,
// defeating the cache.
const statsView = (
  revenue: number,
  orderCount: number,
  topProducts: ReadonlyArray<string>,
  h: HtmlBuilder<Message>,
) =>
  h.div(
    [],
    [
      h.h2([], ['Dashboard']),
      h.p([], [`Revenue: $${revenue}`]),
      h.p([], [`Orders: ${orderCount}`]),
      h.ul(
        [],
        topProducts.map(name => h.li([], [name])),
      ),
    ],
  )

// Create the lazy slot at module level. One slot per view.
const lazyStats = createLazy()

// In your view, wrap the call with the lazy slot.
// If revenue, orderCount, and topProducts are the same references
// as last render, the cached VNode is returned instantly.
// both VNode construction and subtree diffing are skipped.
// The builder travels through the args array like any other argument.
// The runtime hands every render the same builder object, so it never
// invalidates the cache.
const view = (model: Model, h: HtmlBuilder<Message>) => ({
  title: 'Dashboard',
  body: h.div(
    [],
    [
      headerView(model, h),
      lazyStats(statsView, [
        model.revenue,
        model.orderCount,
        model.topProducts,
        h,
      ]),
      sidebarView(model, h),
    ],
  ),
})
```

Both the view function and the lazy slot must stay at module scope. Defining either inside view creates a new reference on every render, so the cache always misses.

Arguments are compared by reference, not by value. This works with [evo](https://foldkit.dev/best-practices/immutability#immutable-updates): an unchanged Model branch keeps its reference, so a lazy view receiving that branch can reuse its VNode.

## createKeyedLazy

`createKeyedLazy` stores an independent memoization slot for every key. Use it when one view function renders several positions, such as rows in a list.

```
import { Array, Option } from 'effect'
import { type HtmlBuilder, createKeyedLazy } from 'foldkit/html'

// Define the per-item view at module level
const contactView = (
  name: string,
  email: string,
  isSelected: boolean,
  h: HtmlBuilder<Message>,
) =>
  h.li(
    [],
    [
      h.span([], [name]),
      h.span([], [email]),
      ...(isSelected ? [h.span([], ['✓'])] : []),
    ],
  )

// Create the keyed lazy map at module level.
// Each key gets its own independent cache slot.
const lazyContact = createKeyedLazy()

// When rendering a list, only items whose args changed are recomputed.
// If you select a different contact, only the previously-selected
// and newly-selected items re-render. All others return cached VNodes.
const contactListView = (
  contacts: ReadonlyArray<Contact>,
  maybeSelectedId: Option.Option<string>,
  h: HtmlBuilder<Message>,
) =>
  h.ul(
    [],
    Array.map(contacts, contact => {
      const isSelected = Option.exists(
        maybeSelectedId,
        selectedId => selectedId === contact.id,
      )

      return lazyContact(contact.id, contactView, [
        contact.name,
        contact.email,
        isSelected,
        h,
      ])
    }),
  )
```

When one item changes, its slot misses while unchanged items return their cached VNodes. The parent view still traverses the list, but it does not rebuild or diff each unchanged item subtree.

One slot per position

A cached VNode can appear at only one position in the tree. Foldkit records its DOM element on the VNode object. Reusing that object at two positions can duplicate or move the wrong DOM node. Give each position its own key. For example: use separate keys for desktop and mobile instances of the same navigation view.

## Keying by Entity Identity

A keyed lazy also separates entities rendered at the same position. A blog page can cache each post by slug, so returning to a previous post can reuse its VNode instead of rebuilding it.

Use the same stable Model identifier for memoization and [DOM identity](https://foldkit.dev/best-practices/keying#keys-and-view-identity). A post addressed by `post.slug` uses `post.slug`. A row keyed with `todo.id` uses `todo.id`. One identifier then names the entity in both systems.

```
import { type HtmlBuilder, createKeyedLazy } from 'foldkit/html'

// One view function serves every post. Turning the compiled markdown into a
// rendered page is the expensive part, so it belongs behind the memo.
const postView = (
  post: Post,
  copiedSnippets: CopiedSnippets,
  h: HtmlBuilder<Message>,
) =>
  h.article(
    [],
    [
      h.h1([], [post.frontmatter.title]),
      docPage(post.document, post.slug).view(copiedSnippets, h),
    ],
  )

// One slot per post, keyed by the same slug the route already uses to give
// the post its DOM identity.
const lazyPostView = createKeyedLazy()

// Navigating between posts moves between slots instead of overwriting one.
// Coming back to a post you already read returns its cached VNode.
const view = (
  post: Post,
  copiedSnippets: CopiedSnippets,
  h: HtmlBuilder<Message>,
) => lazyPostView(post.slug, postView, [post, copiedSnippets, h])
```

Keys can also identify fixed call sites. If one view function renders in two places, give those positions distinct keys instead of maintaining two `createLazy` slots.

Keys are never evicted

`createKeyedLazy` keeps every key it has seen for the lifetime of the page. Use it with a bounded set, such as an entity registry, route table, or fixed set of call sites. A search query or paged cursor can produce unbounded keys and grow the cache without limit.

## When to Use Lazy Views

Consider a lazy view when:

- A large subtree changes less often than its parent.
- A long list usually changes only a few items.
- Profiling shows that building or diffing a view is expensive.

Do not add lazy views to every function. Small views and inputs that change on every render receive no useful cache hits. Confirm the repeated work with the [slow warnings](https://foldkit.dev/core/slow-warnings) and a profiler first.

How it works under the hood

Foldkit's differ compares the old and new VNode by reference before diffing. When they are the same object, it skips attribute comparison, child reconciliation, and DOM updates.

The active dispatch is also part of the internal cache key because event handlers close over it. A DevTools replay uses a non-live dispatch, so Foldkit rebuilds the subtree before returning to the live application even when the function and arguments are unchanged.
