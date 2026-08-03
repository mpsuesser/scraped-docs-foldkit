---
url: https://foldkit.dev/core/http
title: "Http"
description: "Provide a Fetch-backed HttpClient to a Command with trace header propagation disabled by default, so browser requests stay CORS-simple instead of triggering preflights."
access_date: 2026-08-03T17:27:13.509Z
current_date: 2026-08-03T17:27:13.509Z
---

# Http

## Overview

The `Http` module has a single export, `Http.layer`: Effect’s Fetch-backed `HttpClient` with trace header propagation disabled by default. It is the browser-correct client to provide to an HTTP [Command](https://foldkit.dev/core/commands). Yield `HttpClient.HttpClient` inside the Command and provide `Http.layer` at the edge of its Effect.

The snippet on this page imports from `effect/unstable/http`. In the Effect v4 beta that Foldkit pins, the `HttpClient` modules live under the unstable namespace, so that import path is expected and matches what Foldkit projects use.

## Why propagation is off

Effect’s `HttpClient` records an `http.client` span for every request and, by default, writes that span’s context onto the request as `traceparent` and `b3` headers. That default is tuned for servers, where propagating trace context to your own downstream services is desirable. In a browser those same headers make an otherwise CORS-simple request non-simple, so a plain API or a same-origin dev proxy suddenly sees a preflight. `Http.layer` defaults propagation off, so requests stay CORS-simple.

Local observability is unaffected. Disabling propagation removes only the outgoing `traceparent` and `b3` request headers: the `http.client` span with its method, URL, and status attributes is still recorded, and in a Foldkit app it still nests under the runtime’s Command span.

## Providing it in a Command

Provide `Http.layer` per Command with `Effect.provide`. It is a thin wrapper around `fetch`, so building it on each invocation costs nothing and the Command stays self-contained. When an app grows many HTTP Commands, or shares a derived client across modules, provide it once through [resources](https://foldkit.dev/core/resources) instead.

```
import { Effect, Match as M, Schema as S } from 'effect'
import { HttpClient, HttpClientRequest } from 'effect/unstable/http'
import { Command, Http } from 'foldkit'
import { m } from 'foldkit/message'

const ClickedFetchCount = m('ClickedFetchCount')
const SucceededFetchCount = m('SucceededFetchCount', {
  count: S.Number,
})
const FailedFetchCount = m('FailedFetchCount', {
  error: S.String,
})

const CountResponse = S.Struct({ count: S.Number })

const FetchCount = Command.define('FetchCount', {
  messages: [SucceededFetchCount, FailedFetchCount],
  execute: Effect.gen(function* () {
    const client = yield* HttpClient.HttpClient
    const response = yield* client.execute(HttpClientRequest.get('/api/count'))

    if (response.status !== 200) {
      return yield* Effect.fail('API request failed')
    }

    const { count } = yield* S.decodeUnknownEffect(CountResponse)(
      yield* response.json,
    )
    return SucceededFetchCount({ count })
  }).pipe(
    Effect.catch(error =>
      Effect.succeed(FailedFetchCount({ error: String(error) })),
    ),
    Effect.provide(Http.layer),
  ),
})

const update = (
  model: Model,
  message: Message,
): readonly [Model, ReadonlyArray<Command.Command<Message>>] =>
  M.value(message).pipe(
    M.withReturnType<
      readonly [Model, ReadonlyArray<Command.Command<Message>>]
    >(),
    M.tagsExhaustive({
      ClickedFetchCount: () => [model, [FetchCount()]],
      SucceededFetchCount: ({ count }) => [{ count }, []],
      FailedFetchCount: () => [model, []],
    }),
  )
```

## Customizing the client

`Http.layer` folds one overridable default into `FetchHttpClient.layer`, so every seam Effect’s client exposes still works through it. Supply a custom `fetch` by also providing `FetchHttpClient.Fetch`. An app doing distributed tracing can re-enable propagation for one Command with `Effect.provideService(HttpClient.TracerPropagationEnabled, true)`. Add auth headers, a base URL, or retries by transforming the client with `HttpClient.mapRequest` at the call site. You would write your own layer only for a transport that is not `fetch`, which is unusual in the browser.

## Full API surface

The [Http API reference](https://foldkit.dev/api-reference/http) lists the export with its full signature.
