---
url: https://foldkit.dev/api-reference/http
title: "Http"
description: "API documentation for the Http module."
access_date: 2026-08-04T23:13:41.872Z
current_date: 2026-08-04T23:13:41.872Z
---

# Http

## Constants

### layer

const

[source](https://github.com/foldkit/foldkit/blob/c4822d9ea91727f767b8b52f61a81fcc8ae6a260/packages/foldkit/src/http/http.ts#L46)

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
