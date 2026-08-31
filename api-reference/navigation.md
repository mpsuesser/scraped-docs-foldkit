---
url: https://foldkit.dev/api-reference/navigation
title: "Navigation"
description: "API documentation for the Navigation module."
access_date: 2026-08-31T07:29:25.100Z
current_date: 2026-08-31T07:29:25.100Z
---

# Navigation

## Functions

### back

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/navigation/index.ts#L20)

```
/** Navigates back in browser history. */
(): Effect<void>
```

### forward

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/navigation/index.ts#L24)

```
/** Navigates forward in browser history. */
(): Effect<void>
```

### load

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/navigation/index.ts#L28)

```
/** Performs a full page navigation to the given href. */
(href: string): Effect<void>
```

### openUrl

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/navigation/index.ts#L33)

```
/**
 * Opens the given href in a new browsing context (tab or window, at the browser's discretion).
 *  The current page is unchanged. Subject to popup blockers when not called from a user-gesture handler.
 */
(href: string): Effect<void>
```

### pushUrl

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/navigation/index.ts#L6)

```
/** Pushes a new URL to browser history and triggers Foldkit's URL change handling. */
(url: string): Effect<void>
```

### replaceUrl

function

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/navigation/index.ts#L13)

```
/** Replaces the current URL in browser history and triggers Foldkit's URL change handling. */
(url: string): Effect<void>
```

## Constants

### UrlRequest

const

[source](https://github.com/foldkit/foldkit/blob/65afa5c34cc05b5e6423161420faed831c9a5bd9/packages/foldkit/src/navigation/urlRequest.ts#L7)

```
/** Union of `Internal` and `External` URL request types. */
const UrlRequest: TaggedUnion<{
  External: {
    href: String
  }
  Internal: {
    url: Struct<{
      hash: Option<String>
      host: String
      pathname: String
      port: Option<String>
      protocol: String
      search: Option<String>
    }>
  }
}>
```
