---
url: https://foldkit.dev/api-reference/url
title: "Url"
description: "API documentation for the Url module."
access_date: 2026-08-12T01:45:44.345Z
current_date: 2026-08-12T01:45:44.345Z
---

# Url

## Functions

### fromString

function

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/url/index.ts#L110)

```
/** Parses a URL string into a `Url`, returning `Option.None` if invalid. */
(str: string): Option<Url.Url>
```

### toString

function

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/url/index.ts#L112)

```
/** Serializes a `Url` back to a string. */
(url: Url.Url): string
```

## Constants

### Url

const

[source](https://github.com/foldkit/foldkit/blob/a614c5ed5a5c2fb65f8e9128d4ba3bb83d1f550d/packages/foldkit/src/url/index.ts#L13)

```
/** Schema representing a parsed URL with protocol, host, port, pathname, search, and hash fields. */
const Url: Struct<{
  hash: Option<String>
  host: String
  pathname: String
  port: Option<String>
  protocol: String
  search: Option<String>
}>
```
