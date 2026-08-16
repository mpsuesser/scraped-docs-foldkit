---
url: https://foldkit.dev/api-reference/http
title: "Http"
description: "API documentation for the Http module."
access_date: 2026-08-16T19:09:52.991Z
current_date: 2026-08-16T19:09:52.991Z
---

# Http

## Constants

### layer

const

[source](https://github.com/foldkit/foldkit/blob/7014ef6d904c897802df3797298255bb3ce65498/packages/foldkit/src/http/http.ts#L46)

```
/**
 * A Fetch-backed `HttpClient` Layer with trace header propagation disabled by
 * default.
 * 
 * Effect's `HttpClient` records an `http.client` span for every request and,
 * by default, writes that span's context onto the request as `traceparent`
 * and `b3` headers. That default is tuned for servers, where propagating
 * trace context to your own downstream services is desirable. In a browser
 * the same headers make otherwise CORS-simple requests non-simple, triggering
 * preflights against plain APIs and dev proxies that never expect them.
 * Foldkit apps run in the browser, so this Layer defaults propagation off and
 * requests stay CORS-simple.
 * 
 * Local observability is unaffected: the `http.client` span with method, URL,
 * and status attributes is still recorded, and in a Foldkit app it nests
 * under the runtime's Command span. An app doing distributed tracing can
 * re-enable propagation per Command with
 * `Effect.provideService(HttpClient.TracerPropagationEnabled, true)`, or use
 * `FetchHttpClient.layer` from Effect directly.
 */
const layer: Layer.Layer<HttpClient.HttpClient>
```
