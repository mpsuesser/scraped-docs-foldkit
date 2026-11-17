---
url: https://foldkit.dev/api-reference/url
title: "Url"
description: "API documentation for the Url module."
access_date: 2026-11-17T22:33:54.784Z
current_date: 2026-11-17T22:33:54.784Z
---

# Url

## Functions

### fromString

function

[source](https://github.com/foldkit/foldkit/blob/9154209bd6e4776a34f909c8c9795680a8cbac71/packages/foldkit/src/url/index.ts#L110)

```
/** Parses a URL string into a `Url`, returning `Option.None` if invalid. */
(str: string): Option<Url.Url>
```

### toString

function

[source](https://github.com/foldkit/foldkit/blob/9154209bd6e4776a34f909c8c9795680a8cbac71/packages/foldkit/src/url/index.ts#L113)

```
/** Serializes a `Url` back to a string. */
(url: Url.Url): string
```

## Constants

### Url

const

[source](https://github.com/foldkit/foldkit/blob/9154209bd6e4776a34f909c8c9795680a8cbac71/packages/foldkit/src/url/index.ts#L13)

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
