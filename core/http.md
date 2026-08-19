---
url: https://foldkit.dev/core/http
title: "Http"
description: "Provide a Fetch-backed HttpClient to a Command with trace header propagation disabled by default, so browser requests stay CORS-simple instead of triggering preflights."
access_date: 2026-08-19T19:38:38.072Z
current_date: 2026-08-19T19:38:38.072Z
---

# Http

## Overview

The `Http` module has one export: `Http.layer`, a Fetch-backed Effect `HttpClient` Layer with trace-header propagation disabled by default. Provide it to an HTTP [Command](https://foldkit.dev/core/commands), then yield `HttpClient.HttpClient` inside that Command.

The examples import client modules from `effect/unstable/http`. Foldkit currently pins an Effect v4 release candidate, where these modules live under the unstable namespace. That import path is expected.

## Why Propagation Is Off

Effect records an `http.client` span for each request. Its standard Fetch client also propagates the span context through `traceparent` and `b3` request headers. That default suits a server calling downstream services that participate in the same distributed trace.

In a browser, those extra headers can turn an otherwise CORS-simple cross-origin request into a preflighted request. `Http.layer` disables propagation so tracing alone does not change the request's CORS behavior.

Local observability remains intact. The `http.client` span still records request method, URL, and status, and a Foldkit app nests it under the Command span. Only the outgoing trace-context headers are removed.

## Providing It in a Command

Provide `Http.layer` at the edge of the Command's Effect with `Effect.provide`. The Layer is a thin wrapper around the browser's `fetch`, so it can stay local to a self-contained Command. When many HTTP Commands share one configured client, provide it once through [Resources](https://foldkit.dev/core/resources).

The Command remains responsible for status checks, response decoding, and converting failures into declared Messages.

```
import { Effect, Match as M, Schema as S } from 'effect'
import { HttpClient, HttpClientRequest } from 'effect/unstable/http'
import { Command, Http } from 'foldkit'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

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
      SucceededFetchCount: ({ count }) => [
        evo(model, { count: () => count }),
        [],
      ],
      FailedFetchCount: () => [model, []],
    }),
  )
```

## Customizing the Client

`Http.layer` supplies an overridable default to `FetchHttpClient.layer`, so Effect's normal client customization remains available. Provide `FetchHttpClient.Fetch` to substitute a custom `fetch` implementation. Set `HttpClient.TracerPropagationEnabled` to `true` for a Command that participates in distributed tracing.

Transform a yielded client with helpers such as `HttpClient.mapRequest` to add authentication headers or prepend a base URL. Use `HttpClient.retry` or `HttpClient.retryTransient` for request retry policies. A custom Layer is only necessary when the transport is not `fetch` or when the application wants to centralize a configured client.

## Full API Surface

The [Http API reference](https://foldkit.dev/api-reference/http) lists `Http.layer` with its full signature.
