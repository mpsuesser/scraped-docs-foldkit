---
url: https://foldkit.dev/core/http
title: "Http"
description: "Provide a Fetch-backed HttpClient to Commands while keeping browser requests CORS-simple by disabling trace header propagation unless it is required."
access_date: 2026-09-20T01:01:06.971Z
current_date: 2026-09-20T01:01:06.971Z
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
import { Effect, Schema } from 'effect'
import { HttpClient, HttpClientRequest } from 'effect/unstable/http'
import { Command, Http, type Update } from 'foldkit'
import { defineMessageUnion } from 'foldkit/message'
import { modifyFields } from 'foldkit/struct'

const Message = defineMessageUnion({
  ClickedFetchCount: {},
  SucceededFetchCount: { count: Schema.Number },
  FailedFetchCount: { error: Schema.String },
})

const CountResponse = Schema.Struct({ count: Schema.Number })

const FetchCount = Command.define('FetchCount', {
  messages: [Message.SucceededFetchCount, Message.FailedFetchCount],
  execute: Effect.gen(function* () {
    const client = yield* HttpClient.HttpClient
    const response = yield* client.execute(HttpClientRequest.get('/api/count'))

    if (response.status !== 200) {
      return yield* Effect.fail('API request failed')
    }

    const { count } = yield* Schema.decodeUnknownEffect(CountResponse)(
      yield* response.json,
    )
    return Message.SucceededFetchCount({ count })
  }).pipe(
    Effect.catch(error =>
      Effect.succeed(Message.FailedFetchCount({ error: String(error) })),
    ),
    Effect.provide(Http.layer),
  ),
})

const update = (model: Model, message: Message) =>
  Message.match<Update.Return<Model, Message>>(message, {
    ClickedFetchCount: () => ({ model, commands: [FetchCount()] }),
    SucceededFetchCount: ({ count }) => ({
      model: modifyFields(model, { count: () => count }),
    }),
    FailedFetchCount: () => ({ model }),
  })
```

## Customizing the Client

`Http.layer` supplies an overridable default to `FetchHttpClient.layer`, so Effect's normal client customization remains available. Provide `FetchHttpClient.Fetch` to substitute a custom `fetch` implementation. Set `HttpClient.TracerPropagationEnabled` to `true` for a Command that participates in distributed tracing.

Transform a yielded client with helpers such as `HttpClient.mapRequest` to add authentication headers or prepend a base URL. Use `HttpClient.retry` or `HttpClient.retryTransient` for request retry policies. A custom Layer is only necessary when the transport is not `fetch` or when the application wants to centralize a configured client.

## Full API Surface

The [Http API reference](https://foldkit.dev/api-reference/http) lists `Http.layer` with its full signature.
