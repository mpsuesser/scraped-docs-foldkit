---
url: https://foldkit.dev/core/runtime
title: "Runtime"
description: "Mount and run a Foldkit application in the browser."
access_date: 2026-08-03T19:40:01.169Z
current_date: 2026-08-03T19:40:01.169Z
---

# Runtime

## Overview

A Foldkit app lives in two files. `src/main.ts` holds the pure definitions: Model, Messages, update, init, and view. `src/entry.ts` imports them, creates a runtime with `makeApplication`, and calls `Runtime.run`. `entry.ts` is the only place runtime side effects happen, which keeps `main.ts` importable from tests.

## makeApplication

`makeApplication` creates a Foldkit runtime for an app that owns the page. It handles both single-page apps and full applications with routing. The difference is whether you provide a `routing` config. To mount an app scoped to a node without owning the page, use `makeElement` (below).

### Without routing

Without a `routing` config, the program doesn't manage the URL bar. This is the default for most programs.

```
import { Runtime } from 'foldkit'

import { Model, init, update, view } from './main'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  container: document.getElementById('root'),
})

Runtime.run(application)
```

### With routing

With a `routing` config, the program manages the URL bar. The init function receives the current URL so it can set the initial route.

```
import { Runtime } from 'foldkit'

import { ChangedUrl, ClickedLink, Model, init, update, view } from './main'

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
})

Runtime.run(application)
```

The `routing` config has two handlers: `onUrlRequest` is called when a link is clicked (giving you a chance to handle internal vs external links), and `onUrlChange` is called when the URL changes (so you can update your Model with the new route). See the [Routing & Navigation](https://foldkit.dev/core/routing-and-navigation) guide for a full walkthrough.

Your `view` function returns a `Document` rather than bare HTML: the body to render, plus the document-level state the runtime keeps in sync. `makeApplication` owns that state and reapplies it on every render, so the tab title, the `<html>` language and direction, and the canonical and og:url tags all track your Model. The [View](https://foldkit.dev/core/view#the-document) page lists every field and what to put in it.

## makeElement

`makeApplication` assumes it owns the page, reapplying on every render whatever document state the view declares. That is what you want for an app that owns its tab, but not for a widget embedded on a page you do not control, where it would clobber the host page metadata.

Use `makeElement` to mount a Foldkit app scoped to its container. Its `view` returns `Html` directly rather than a `Document`, so there is no title to discard, and the runtime never touches the document `<head>` or the `<html>` element. Everything else (Model, `init`, `update`, Commands, Subscriptions, flags, crash handling) works exactly as it does with `makeApplication`. Embedded apps do not own the URL bar, so `makeElement` has no `routing` config.

```
import { Runtime } from 'foldkit'

import { Model, init, update, view } from './main'

const element = Runtime.makeElement({
  Model,
  init,
  update,
  view,
  container: document.getElementById('widget'),
})

Runtime.run(element)
```

## embed

`makeElement` mounts a self-contained app in a container. `Runtime.embed` goes further for a widget embedded in a host application, whether that host is React or anything else. The host starts the runtime, seeds it with Flags, exchanges values through Schema-typed Ports, and tears it down with `dispose`. The handle is the whole boundary: the host never reads the Model or dispatches Messages.

The [Embedding](https://foldkit.dev/core/embedding) guide has the full walkthrough.
