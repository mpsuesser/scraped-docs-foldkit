---
url: https://foldkit.dev/api-reference/route
title: "Route"
description: "API documentation for the Route module."
access_date: 2026-08-03T18:13:14.939Z
current_date: 2026-08-03T18:13:14.939Z
---

# Route

## Functions

### int

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L201)

```
/**
 * Creates a parser that captures a URL segment as a named integer field.
 * 
 * Fails if the segment is not a valid integer.
 */
<K extends string>(name: K): Biparser<Record<K, number>>
```

### literal

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L105)

```
/** Creates a parser that matches an exact URL path segment. */
(segment: string): Biparser<{}>
```

### oneOf

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L514)

```
/**
 * Combines multiple parsers, trying each in order until one matches the
 * entire path.
 * 
 * A parser only matches when it consumes every segment, so a route never
 * shadows a longer route that shares its prefix. When several parsers
 * fully match the same URL, the first one wins.
 * 
 * Returns a `Parser` (parse-only) since the union of different route
 * shapes cannot provide a single unified print function.
 */
<Parsers extends readonly Array<ParserInput>>(parsers: Parsers): Parser<InferParsed<Parsers[number]>>
```

### param

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L143)

```
/** Creates a parser for a dynamic URL segment with custom parse and print functions. */
<A>(
  label: string,
  parse: (segment: string) => Effect<A, ParseError>,
  print: (value: A) => string
): Biparser<A>
```

label

: A descriptive name used in error messages.

parse

: Converts a raw URL segment string into the parsed value.

print

: Converts the parsed value back into a URL segment string.

### parseUrlWithFallback

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L723)

```
/**
 * Parses a URL against a parser, falling back to a not-found route if no
 * parser matches.
 */
<A, B>(
  parser: Parser<A>,
  notFoundRouteConstructor: {
    make: (data: {
      path: string
    }) => B
  }
): (url: Url.Url) => A | B
```

parser

: The parser (typically from `oneOf`) to attempt.

notFoundRouteConstructor

: Constructor called with `{ path }` when no route matches.

### query

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L633)

```
/**
 * Adds query parameter parsing to a `Biparser` using an Effect `Schema`.
 * 
 * Produces a `TerminalParser` that cannot be extended with `slash`,
 * since query parameters must appear at the end of a route definition.
 */
<A, I extends ReadonlyRecord<string, unknown>>(schema: Codec<A, I>): (parser: Biparser<B>) => TerminalParser<B & A>
```

schema

: An Effect Schema describing the expected query parameters.

### r

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/schema/index.ts#L72)

```
/**
 * Wraps `Schema.TaggedStruct` to create a route variant you can call directly as a constructor.
 * Use `r` for route types — enabling `Home()` instead of `Home.make()`.
 */
<Tag extends string>(tag: Tag): CallableTaggedStruct<Tag, {}>

<Tag extends string, Fields extends Fields>(
  tag: Tag,
  fields: Fields
): CallableTaggedStruct<Tag, Fields>
```

### rest

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L360)

```
/**
 * Creates a parser that captures all remaining URL segments as a named
 * non-empty array field.
 * 
 * Requires at least one remaining segment. A bare prefix like `/files`
 * does not match; give it its own route alongside the rest route.
 * 
 * A rest route also matches every URL that a more specific route under
 * the same prefix accepts, so in `oneOf` the specific route must come
 * first.
 * 
 * Nothing can follow `rest` in the path, so the result is a
 * `TerminalParser`. It can still be extended with `query`.
 */
<K extends string>(name: K): TerminalParser<Record<K, readonly [string, string]>>
```

### restString

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L427)

```
/**
 * Creates a parser that captures all remaining URL segments as a single
 * named string field, preserving the slashes between them.
 * 
 * Where `rest` yields a `NonEmptyArray` of segments, `restString` rejoins
 * the tail into one raw path string, so a value that itself contains both
 * slashes and dots, like a repository-relative file path
 * `documents/taxes/2024.pdf`, round-trips through the bidirectional router
 * as `{ path: string }` route data.
 * 
 * Requires at least one remaining segment. A bare prefix like `/vault`
 * does not match; give it its own route alongside the restString route.
 * 
 * A restString route also matches every URL that a more specific route under
 * the same prefix accepts, so in `oneOf` the specific route must come first.
 * 
 * Printing requires a normalized path: non-empty, with no leading,
 * trailing, or repeated slashes. Any other value would build a URL that
 * parses back differently, so printing fails instead of building it.
 * 
 * Nothing can follow `restString` in the path, so the result is a
 * `TerminalParser`. It can still be extended with `query`.
 */
<K extends string>(name: K): TerminalParser<Record<K, string>>
```

### schemaSegment

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L265)

```
/**
 * Creates a parser that captures a URL segment and decodes it through an
 * Effect `Schema`, producing a named field of the schema's decoded type.
 * 
 * Decodes the raw segment when parsing and encodes the value back when
 * printing, so branded ids, refined strings, and string-literal unions
 * round-trip through the route. The decoded type flows straight into the
 * route value, so a model carries `UserId` rather than a bare `string`.
 * 
 * The schema's encoded form must be a single URL segment string. For shapes
 * that span multiple segments or live in the query string, use `rest` or
 * `query`.
 */
<K extends string, A, I extends string>(
  name: K,
  schema: Codec<A, I>
): Biparser<Record<K, A>>
```

name

: The field name the decoded value is captured under.

schema

: A codec whose encoded form is a single URL segment string.

### string

function

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L181)

```
/** Creates a parser that captures a URL segment as a named string field. */
<K extends string>(name: K): Biparser<Record<K, string>>
```

## Types

### Biparser

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L43)

```
/**
 * A bidirectional parser that can both parse URL segments into a value
 * and print a value back to URL segments.
 */
type Biparser = {
  parse: (segments: ReadonlyArray<string>, search?: string) => Effect.Effect<ParseResult<A>, ParseError>
  print: (value: A, state: PrintState) => Effect.Effect<PrintState, ParseError>
}
```

### ExtendableBiparser

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L86)

```
/**
 * A `Biparser` that can still be extended with `slash`. Terminal parsers
 * (`query`, `rest`) do not qualify, since nothing can follow them in
 * the path.
 */
type ExtendableBiparser = Biparser<A> & {
  __terminal: never
  Cannot use slash after a terminal parser - nothing can follow query or rest: never
}
```

### ParseResult

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L32)

```
/** The result of parsing: a tuple of the parsed value and remaining URL segments. */
type ParseResult = [A, ReadonlyArray<string>]
```

### Parser

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L476)

```
/**
 * A parse-only parser with no print/build capabilities.
 * 
 * Returned by `oneOf`, which combines multiple parsers whose print
 * types may differ and therefore cannot be unified into a single `Biparser`.
 */
type Parser = {
  parse: (segments: ReadonlyArray<string>, search?: string) => Effect.Effect<ParseResult<A>, ParseError>
}
```

### Router

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L67)

```
/**
 * A parser with a `build` method that can reconstruct URLs from parsed values.
 * 
 * Created by applying `mapTo` to a `Biparser`, binding it to a tagged
 * type constructor so parsed values carry a discriminant tag and URLs can be
 * built from tag payloads.
 * 
 * Routers are callable — `homeRouter()` or `personRouter({ id: 42 })` —
 * and also expose `.build` as an alias and `.parse` for URL matching.
 */
type Router = BuildFn<A> & {
  build: BuildFn<A>
  parse: (segments: ReadonlyArray<string>, search?: string) => Effect.Effect<ParseResult<A>, ParseError>
}
```

### TerminalParser

type

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L79)

```
/**
 * A `Biparser` that has been terminated (e.g. by `query` or `rest`)
 * and cannot be extended with `slash`.
 */
type TerminalParser = Biparser<A> & {
  __terminal: true
}
```

## Constants

### mapTo

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L546)

```
/**
 * Converts a `Biparser` into a `Router` by mapping parsed values to a
 * tagged type constructor.
 * 
 * The resulting `Router` can both parse URLs into tagged route values and
 * build URLs from route payloads.
 */
const mapTo: (appRouteConstructor: {
  make: () => T
}) => (parser: Biparser<{}>) => Router<T>
```

### root

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L323)

```
/**
 * A parser that matches the root path with no remaining segments.
 * 
 * Succeeds only when the URL path is exactly `/`.
 */
const root: Biparser<{}>
```

### slash

const

[source](https://github.com/foldkit/foldkit/blob/7790ab1a71e346cfe02b0290e24666d900054b3b/packages/foldkit/src/route/parser.ts#L586)

```
/**
 * Composes two `Biparser`s sequentially, combining their parsed values.
 * 
 * Cannot be used after a terminal parser (`query` or `rest`).
 * Composing with a terminal second parser yields a `TerminalParser`,
 * so terminality survives the composition.
 */
const slash: (parserB: TerminalParser<A>) => (parserA: ExtendableBiparser<B>) => TerminalParser<B & A>
```
