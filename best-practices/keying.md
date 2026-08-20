---
url: https://foldkit.dev/best-practices/keying
title: "Keying"
description: "Use stable Model identifiers to preserve identity for mapped list items and entities rendered at one position."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Keying

## Keys and View Identity

Keys identify Model entities across renders. Write a key when one view function renders several entities, either in a dynamic collection or at one fixed position over time. Use a stable identifier such as an id or slug.

Foldkit handles the other kind of identity for you. The `@foldkit/vite-plugin` build stamps the VNodes returned by each view function. The same function at one position patches the existing DOM; a different function replaces it. Focus, uncontrolled input values, scroll position, and other element-owned state therefore stay with the view that owns them.

Coming from React

A Foldkit view function plays the role a component type plays during reconciliation. The same function patches; a different function replaces. Keys still identify list items and entities, but the build handles branch identity for every branching syntax.

The syntax that selects a view function does not matter. `if`/`else`, ternaries, and Effect `Match` all use the identity of the function that produced the subtree:

```
import type { Html, HtmlBuilder } from 'foldkit/html'

const searchView = (model: Model, h: HtmlBuilder<Message>): Html =>
  h.div(
    [h.Class('search')],
    [model.mode === 'Editing' ? editorView(model, h) : summaryView(model, h)],
  )
```

`editorView` and `summaryView` have different identities, so switching replaces the subtree even when both return the same root tag. Inline same-tag elements inside one surrounding function share that function's identity and patch in place. If switching an inline branch must reset DOM state, extract its arms into named view functions.

## Keys for Model Entities

### Mapped List Items

Mapped rows all come from the same function, so view identity cannot distinguish them. Key each item's root by a stable Model identifier, never by its array position:

```
import type { Html, HtmlBuilder } from 'foldkit/html'

const entryListView = (
  entries: ReadonlyArray<Entry>,
  h: HtmlBuilder<Message>,
): Html =>
  h.ul(
    [],
    entries.map(entry =>
      h.keyed('li')(
        entry.id,
        [],
        [
          h.input([
            h.Value(entry.text),
            h.OnInput(text => EditedEntry({ id: entry.id, text })),
          ]),
        ],
      ),
    ),
  )
```

### One Position, Different Entities

A detail page may render every article through `articlePageView`. The function identity stays the same when navigation selects another article, so key the root by the article id or slug:

```
import type { Html, HtmlBuilder } from 'foldkit/html'

const articlePageView = (article: Article, h: HtmlBuilder<Message>): Html =>
  h.keyed('article')(
    article.slug,
    [],
    [h.h1([], [article.title]), h.p([], [article.body])],
  )
```

The key makes the old article and the new article different entities. Local DOM state cannot carry from one into the other.

### Stable Identity, Not Displayed Data

Key by what an entity is, never by what it currently shows. A key derived from displayed data changes whenever the content changes. Each edit then tears down the node and discards focus, text selection, and other element-owned state:

```
import type { Html, HtmlBuilder } from 'foldkit/html'

// ❌ Displayed data is not identity
const reviewPanelKeyedByData = (model: Model, h: HtmlBuilder<Message>): Html =>
  h.keyed('div')(
    `${model.isCardSelected}:${model.isTermsAccepted}`,
    [],
    [reviewContentView(model, h)],
  )

// ✅ The same panel remains the same entity
const reviewPanel = (model: Model, h: HtmlBuilder<Message>): Html =>
  h.div([], [reviewContentView(model, h)])
```

Always build with the plugin

Identity is stamped by `@foldkit/vite-plugin`, which `create-foldkit-app` includes by default. Do not build a Foldkit app without it.
