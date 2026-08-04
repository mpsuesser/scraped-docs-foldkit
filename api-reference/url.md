---
url: https://foldkit.dev/api-reference/url
title: "Url"
description: "API documentation for the Url module."
access_date: 2026-08-04T03:55:42.119Z
current_date: 2026-08-04T03:55:42.119Z
---

# Url

## Functions

### fromString

function

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/url/index.ts#L110)

```
/** Parses a URL string into a `Url`, returning `Option.None` if invalid. */
(str: string): Option<Url.Url>
```

### toString

function

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/url/index.ts#L112)

```
/** Serializes a `Url` back to a string. */
(url: Url.Url): string
```

## Constants

### Url

const

[source](https://github.com/foldkit/foldkit/blob/f314f3fb4452439d3c0143188dbcebf8f11e2c00/packages/foldkit/src/url/index.ts#L13)

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
