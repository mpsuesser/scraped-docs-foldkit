---
url: https://foldkit.dev/api-reference/http
title: "Http"
description: "API documentation for the Http module."
access_date: 2026-11-17T22:33:54.784Z
current_date: 2026-11-17T22:33:54.784Z
---

# Http

## Constants

### layer

const

[source](https://github.com/foldkit/foldkit/blob/9154209bd6e4776a34f909c8c9795680a8cbac71/packages/foldkit/src/http/http.ts#L46)

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
