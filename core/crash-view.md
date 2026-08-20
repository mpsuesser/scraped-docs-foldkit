---
url: https://foldkit.dev/core/crash-view
title: "Crash View"
description: "Replace the default Runtime crash screen and report unrecoverable failures without treating expected Effect failures as crashes."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Crash View

## Runtime Failure

When an unrecoverable error escapes update, view, or a Command, Foldkit stops the application and renders a crash view. No later Message will run. Recoverable failures should become Messages instead, so update can decide what the person sees next.

The default crash view shows the error message and a reload button. To replace it, pass `crash.view` to `makeApplication`. The function receives a `CrashContext`, followed by `h`. The context has three fields:

- `error` is the error that stopped the application.
- `model` is the Model at the time of the crash.
- `message` is the Message being processed, wrapped in `Option`. It is `None` when the initial render crashes.

```
import { Runtime } from 'foldkit'
import { Document, HtmlBuilder } from 'foldkit/html'

const crashView = (
  { error }: Runtime.CrashContext<Model, Message>,
  h: HtmlBuilder<never>,
): Document => ({
  title: 'Something went wrong',
  body: h.div(
    [h.Class('min-h-screen flex items-center justify-center bg-red-50 p-8')],
    [
      h.div(
        [
          h.Class(
            'max-w-md w-full bg-cream rounded-lg border border-red-200 p-8 text-center',
          ),
        ],
        [
          h.h1(
            [h.Class('text-red-600 text-2xl font-semibold mb-4')],
            ['Something went wrong'],
          ),
          h.p([h.Class('text-gray-700 mb-6')], [error.message]),
          h.button(
            [
              h.Class(
                'bg-red-600 text-white px-6 py-2.5 rounded-md text-sm font-normal cursor-pointer',
              ),
              h.Attribute('onclick', 'location.reload()'),
            ],
            ['Reload'],
          ),
        ],
      ),
    ],
  ),
})

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  crash: { view: crashView },
  container: document.getElementById('root'),
})

Runtime.run(application)
```

The builder is `HtmlBuilder<never>` because Foldkit can no longer dispatch Messages. Event helpers such as `h.OnClick` therefore fail to compile instead of creating handlers that cannot run.

For an action that does not need Foldkit, set a raw DOM event attribute. For example: `h.Attribute('onclick', 'location.reload()')` reloads the page through the browser.

Only in crash.view

In a normal Foldkit app, always use `OnClick` with Messages, never raw DOM event attributes. `crash.view` is the one exception because the runtime is no longer running.

If the custom crash view throws, Foldkit renders the default crash screen with both errors.

## Crash Reporting

Use `crash.report` to send the failure to Sentry or another reporting service. It receives the same `CrashContext` as `crash.view`.

```
import { Option } from 'effect'
import { Runtime } from 'foldkit'

import * as Sentry from '@sentry/browser'

const application = Runtime.makeApplication({
  Model,
  init,
  update,
  view,
  crash: {
    report: ({ error, model, message }) => {
      Sentry.captureException(error, {
        extra: { model, message: Option.getOrUndefined(message) },
      })
    },
  },
  container: document.getElementById('root'),
})

Runtime.run(application)
```

Foldkit calls `crash.report` synchronously and does not await work it starts. If the reporter must flush a buffer or make a request, start that work inside the callback.

Reporting runs before the crash view renders. If `crash.report` throws, Foldkit logs that error and still renders the crash view.

See the [crash-view example](https://foldkit.dev/example-apps/crash-view) for a working demonstration.
