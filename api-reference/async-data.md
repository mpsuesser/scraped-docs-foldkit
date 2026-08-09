---
url: https://foldkit.dev/api-reference/async-data
title: "AsyncData"
description: "API documentation for the AsyncData module."
access_date: 2026-08-09T21:03:38.321Z
current_date: 2026-08-09T21:03:38.321Z
---

# AsyncData

## Functions

### Schema

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L165)

```
/**
 * Builds the six-state `AsyncData` Schema for the given data and error
 *  Schemas (value-first). Put `schema` in your Model; use the returned
 *  constructors when you want ones typed to this instance's `A` and `E`.
 */
<A, AI, E, EI>(
  dataSchema: Codec<A, AI>,
  errorSchema: Codec<E, EI>
): AsyncDataSchema<A, AI, E, EI>
```

### fail

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L117)

```
/** Bare-value alias for `Failure({ error })`, mirroring `Result.fail`. */
<E, A = never>(error: E): AsyncData<A, E>
```

### fromOptionOrIdle

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L518)

```
/**
 * Collapses a possibly-missing cache entry to an AsyncData: `Some(entry)`
 *  is the entry, `None` is `Idle`. The keyed-cache read idiom, where a key
 *  that was never fetched is simply absent and absence means nothing was
 *  requested yet:
 * 
 *  ```ts
 *  const notes = AsyncData.fromOptionOrIdle(HashMap.get(model.notesByNotebook, notebookId))
 *  ```
 */
<A, E>(maybeEntry: Option<AsyncData<A, E>>): AsyncData<A, E>
```

### getData

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L480)

```
/**
 * Returns the data as an `Option`, spanning all three data-bearing states:
 *  `Some` for `Success`, `Refreshing`, and `Stale`, `None` otherwise.
 *  Payload-named because it deliberately spans three tags.
 */
<A, E>(self: AsyncData<A, E>): Option<A>
```

### getError

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L490)

```
/**
 * Returns the error as an `Option`, spanning both error-bearing states:
 *  `Some` for `Failure` and `Stale`, `None` otherwise. Symmetric with
 *  `getData`.
 */
<A, E>(self: AsyncData<A, E>): Option<E>
```

### hasData

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L552)

```
/**
 * Returns `true` when the state holds data: `Success`, `Refreshing`, or
 *  `Stale`.
 */
<A, E>(self: AsyncData<A, E>): boolean
```

### hasError

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L557)

```
/**
 * Returns `true` when the state carries an error: `Failure` or `Stale`.
 *  The error-channel twin of `hasData`.
 */
<A, E>(self: AsyncData<A, E>): boolean
```

### isAsyncData

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L577)

```
/**
 * Type guard on `unknown`: checks that the value has a `_tag` belonging to
 *  the six `AsyncData` states.
 */
(input: unknown): unknown
```

### isFailure

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L537)

```
/** Returns `true` only for the `Failure` state, not `Stale`. Refinement. */
<A, E>(self: AsyncData<A, E>): unknown
```

### isIdle

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L523)

```
/** Returns `true` only for the `Idle` state. Refinement. */
<A, E>(self: AsyncData<A, E>): unknown
```

### isLoading

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L528)

```
/**
 * Returns `true` only for the empty `Loading` state, not `Refreshing`.
 *  Refinement.
 */
<A, E>(self: AsyncData<A, E>): unknown
```

### isPending

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L563)

```
/**
 * Returns `true` when a request is in flight: `Loading` or `Refreshing`
 *  only. `Stale` is not pending because its fetch already failed. Use for a
 *  spinner regardless of held data.
 */
<A, E>(self: AsyncData<A, E>): boolean
```

### isRefreshing

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L532)

```
/** Returns `true` only for the `Refreshing` state. Refinement. */
<A, E>(self: AsyncData<A, E>): unknown
```

### isStale

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L542)

```
/**
 * Returns `true` only for the `Stale` state: a failed refresh holding the
 *  last good data. Refinement.
 */
<A, E>(self: AsyncData<A, E>): unknown
```

### isSuccess

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L547)

```
/**
 * Returns `true` only for the `Success` state, not `Refreshing` or
 *  `Stale`. Refinement.
 */
<A, E>(self: AsyncData<A, E>): unknown
```

### loadIfMissing

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L644)

```
/**
 * The first-visit load transition: the cold no-data states (`Idle`,
 *  `Failure`) start a fresh `Loading`; every other state yields `None`, so
 *  loaded data is kept without revalidation and a request in flight is not
 *  restarted. The load-only sibling of `revalidateOrLoad` and `revalidate`,
 *  and the state-machine form of TanStack Query's `staleTime: Infinity`:
 *  fetch on first visit, keep the cache afterwards.
 */
<A, E>(self: AsyncData<A, E>): Option<AsyncData<A, E>>
```

### revalidate

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L630)

```
/**
 * The loaded-only revalidation transition: `Success` and `Stale` move to
 *  `Refreshing`, every other state yields `None`. Unlike
 *  `revalidateOrLoad` there is no cold-start `Loading`, so only caches
 *  that actually hold data revalidate; this is the generic refresher path
 *  after a mutation.
 */
<A, E>(self: AsyncData<A, E>): Option<AsyncData<A, E>>
```

### revalidateOrLoad

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L610)

```
/**
 * The revalidate-on-entry transition: revalidates loaded data and loads
 *  cold data. The data-bearing loaded states (`Success`, `Stale`) move to
 *  `Refreshing`; the cold no-data states (`Idle`, `Failure`) start a fresh
 *  `Loading`; the already-pending states (`Loading`, `Refreshing`) yield
 *  `None` so the request in flight is not restarted. `None` means no
 *  transition, and no load Command, is needed.
 */
<A, E>(self: AsyncData<A, E>): Option<AsyncData<A, E>>
```

### succeed

function

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L113)

```
/** Bare-value alias for `Success({ data })`, mirroring `Result.succeed`. */
<A, E = never>(data: A): AsyncData<A, E>
```

## Types

### AsyncData

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L55)

```
/**
 * The six-state union for an asynchronously loaded value: `Idle | Loading
 *  | Refreshing(data) | Failure(error) | Stale(error, data) | Success(data)`.
 *  Value-first type parameters, matching `Result<A, E>` and `Exit<A, E>`.
 * 
 *  `Refreshing` and `Stale` make stale-while-revalidate and
 *  stale-on-failure first-class states: both hold the previous good `data`,
 *  `Refreshing` while a reload is in flight, `Stale` after a reload failed.
 * 
 *  Data-presence classification, used throughout the module:
 * 
 *  - "has data" states: `Success`, `Refreshing`, `Stale`.
 *  - "no data" states: `Idle`, `Loading`, `Failure`.
 *  - "pending" states (a request in flight): `Loading`, `Refreshing` only.
 *    `Stale` is not pending; its fetch already failed.
 * 
 *  Naming rule: predicates are tag-named (`isSuccess`, `isFailure`) but
 *  getters are payload-named (`getData`, `getError`) because `getData` spans
 *  three tags (`Success`, `Refreshing`, `Stale`) and `getError` spans two
 *  (`Failure`, `Stale`). Handler keys are tag-named where dispatch is per tag
 *  (`match`) and channel-named where one handler covers multiple tags
 *  (`matchData`, `mapBoth`).
 */
type AsyncData = Idle | Loading | Refreshing<A> | Failure<E> | Stale<A, E> | Success<A>
```

### AsyncDataEncoded

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L124)

```
/**
 * The encoded (wire) form of a `AsyncData<A, E>` whose data encodes to
 *  `AI` and whose error encodes to `EI`.
 */
type AsyncDataEncoded = Readonly<{
  _tag: "Idle"
}> | Readonly<{
  _tag: "Loading"
}> | Readonly<{
  _tag: "Refreshing"
  data: AI
}> | Readonly<{
  _tag: "Failure"
  error: EI
}> | Readonly<{
  _tag: "Stale"
  data: AI
  error: EI
}> | Readonly<{
  _tag: "Success"
  data: AI
}>
```

### AsyncDataSchema

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L136)

```
/**
 * What the `Schema` factory returns: the six-state Union codec to embed in
 *  a Model plus the Schema-bound constructors for the four parameterized
 *  states. Combinators are not part of this bag; they are module-level free
 *  functions over any `AsyncData<A, E>` value.
 */
type AsyncDataSchema = Readonly<{
  Failure: (payload: Readonly<{
    error: E
  }>) => AsyncData<A, E>
  Idle: typeof Idle
  Loading: typeof Loading
  Refreshing: (payload: Readonly<{
    data: A
  }>) => AsyncData<A, E>
  schema: S.Codec<AsyncData<A, E>, AsyncDataEncoded<AI, EI>>
  Stale: (payload: Readonly<{
    data: A
    error: E
  }>) => AsyncData<A, E>
  Success: (payload: Readonly<{
    data: A
  }>) => AsyncData<A, E>
}>
```

### Failure

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L25)

```
/** The `Failure` state: request failed, showing the failure. */
type Failure = Readonly<{
  _tag: "Failure"
  error: E
}>
```

### Idle

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L16)

```
/** The `Idle` state: nothing requested yet. */
type Idle = Readonly<{
  _tag: "Idle"
}>
```

### Loading

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L19)

```
/** The `Loading` state: first request in flight, no prior data. */
type Loading = Readonly<{
  _tag: "Loading"
}>
```

### Refreshing

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L22)

```
/** The `Refreshing` state: reloading while holding the previous good data. */
type Refreshing = Readonly<{
  _tag: "Refreshing"
  data: A
}>
```

### Stale

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L29)

```
/**
 * The `Stale` state: the last refresh failed, still holding the previous
 *  good data. The failed-refresh mirror of `Refreshing`.
 */
type Stale = Readonly<{
  _tag: "Stale"
  data: A
  error: E
}>
```

### Success

type

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L32)

```
/** The `Success` state: request succeeded, data present. */
type Success = Readonly<{
  _tag: "Success"
  data: A
}>
```

## Constants

### Failure

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L25)

```
/**
 * Constructs a `Failure` state carrying the error only. Plain value
 *  builder, generic in `E`; use the `Schema` factory's `Failure` for a
 *  Schema-bound constructor.
 */
const Failure: (payload: Readonly<{
  error: E
}>) => AsyncData<never, E>
```

### Idle

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L16)

```
/**
 * Constructs the `Idle` state. A parameter-free callable Schema, so it can
 *  also serve as a Union member.
 */
const Idle: CallableTaggedStruct<"Idle", {}>
```

### Loading

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L19)

```
/**
 * Constructs the `Loading` state. A parameter-free callable Schema, so it
 *  can also serve as a Union member.
 */
const Loading: CallableTaggedStruct<"Loading", {}>
```

### Refreshing

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L22)

```
/**
 * Constructs a `Refreshing` state holding the previous good data. Plain
 *  value builder, generic in `A`; use the `Schema` factory's `Refreshing`
 *  for a Schema-bound constructor.
 */
const Refreshing: (payload: Readonly<{
  data: A
}>) => AsyncData<A, never>
```

### Stale

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L29)

```
/**
 * Constructs a `Stale` state carrying both the refresh error and the
 *  last good data. Plain value builder, generic in `A` and `E`; use the
 *  `Schema` factory's `Stale` for a Schema-bound constructor.
 */
const Stale: (payload: Readonly<{
  data: A
  error: E
}>) => AsyncData<A, E>
```

### Success

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L32)

```
/**
 * Constructs a `Success` state holding the data. Plain value builder,
 *  generic in `A`; use the `Schema` factory's `Success` for a Schema-bound
 *  constructor.
 */
const Success: (payload: Readonly<{
  data: A
}>) => AsyncData<A, never>
```

### all

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L757)

```
/**
 * Combines an iterable or record of values under the `zipWith` lattice.
 *  An iterable collapses to an in-order array of data (empty input yields
 *  `Success({ data: [] })`); a record builds a struct (empty input yields
 *  `Success({ data: {} })`). The highest-ranked no-data state blocks, the
 *  leftmost `Failure` error wins, and any `Stale` in an all-data set makes
 *  the result `Stale` with the leftmost `Stale` error. All inputs must share
 *  one error type; unify with `mapError` first.
 * 
 *  The record form is the multi-resource screen:
 *  `all({ user, orders, prefs })` combines into one value whose data is the
 *  struct of all datas.
 */
const all: (inputs: Inputs) => [Inputs] extends [ReadonlyArray<AsyncData<any, any>>]
  ? AsyncData<{
    [K in keyof Inputs]: [Inputs[K]] extends [AsyncData<infer A, any>]
      ? A
      : never
  }, Inputs[number] extends never
    ? never
    : [Inputs[number]] extends [AsyncData<any, infer E>]
      ? E
      : never>
  : [Inputs] extends [Iterable<AsyncData<infer A, infer E>>]
    ? AsyncData<Array<A>, E>
    : AsyncData<{
      [K in keyof Inputs]: [Inputs[K]] extends [AsyncData<infer A, any>]
        ? A
        : never
    }, Inputs[keyof Inputs] extends never
      ? never
      : [Inputs[keyof Inputs]] extends [AsyncData<any, infer E>]
        ? E
        : never>
```

### flatMap

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L450)

```
/**
 * Treats every data-bearing state (`Success`, `Refreshing`, `Stale`)
 *  exactly like `Success(data)`: returns `f(data)` unchanged, dropping the
 *  tag and any `Stale` error, so a caller can settle in-flight or stale data
 *  into `Success`. The no-data states pass through. Widens the error channel
 *  to `E | E2`, matching `Result.flatMap`.
 */
const flatMap: (f: (data: A) => AsyncData<B, E2>) => (self: AsyncData<A, E>) => AsyncData<B, E2 | E>
```

### getOrElse

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L501)

```
/**
 * Returns the data of any data-bearing state, or `onEmpty()` for the three
 *  no-data states. The fallback is a nullary thunk, mirroring
 *  `Option.getOrElse`, because the no-data states collapse to empty with no
 *  single payload.
 */
const getOrElse: (onEmpty: LazyArg<B>) => (self: AsyncData<A, E>) => B | A
```

### map

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L363)

```
/**
 * Maps the data of all three data-bearing states, preserving each tag so
 *  the `Refreshing` and `Stale` signals survive a pure transform. `Stale`
 *  maps only its `data`, keeping its `error`. The no-data states pass
 *  through unchanged.
 */
const map: (f: (data: A) => B) => (self: AsyncData<A, E>) => AsyncData<B, E>
```

### mapBoth

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L407)

```
/**
 * Maps both channels with channel-named handlers: `onData` spans `Success`,
 *  `Refreshing`, and `Stale`; `onError` spans `Failure` and `Stale`. For
 *  `Stale`, both handlers apply. Tags are preserved.
 */
const mapBoth: (handlers: {
  onData: (data: A) => B
  onError: (error: E) => E2
}) => (self: AsyncData<A, E>) => AsyncData<B, E2>
```

### mapError

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L385)

```
/**
 * Maps the error of the two error-bearing states: `Failure` and `Stale`
 *  transform (`Stale` keeps its `data`), everything else passes through.
 *  Use it to unify heterogeneous error types before a combine.
 */
const mapError: (f: (error: E) => E2) => (self: AsyncData<A, E>) => AsyncData<A, E2>
```

### match

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L218)

```
/**
 * Handles all six states exhaustively, passing each handler its unwrapped
 *  payload. `onStale` alone receives the whole `{ error, data }` payload
 *  object because `Stale` carries two fields. Use `matchData` when the view
 *  does not care which of the data-bearing states it is rendering.
 */
const match: (handlers: Readonly<{
  onFailure: (error: E) => F
  onIdle: Function.LazyArg<B>
  onLoading: Function.LazyArg<C>
  onRefreshing: (data: A) => D
  onStale: (payload: Readonly<{
    data: A
    error: E
  }>) => G
  onSuccess: (data: A) => H
}>) => (self: AsyncData<A, E>) => B | C | D | F | G | H
```

### matchData

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L279)

```
/**
 * Collapses the six states to the three channels a view usually renders:
 *  `onData` spans the data-bearing states (`Success`, `Refreshing`,
 *  `Stale`), `onFailure` receives the `Failure` error, and `onEmpty` covers
 *  `Idle` and `Loading` together. A `Stale` renders through `onData` so its
 *  data stays on screen. Use `matchDataSplitEmpty` when `Idle` and `Loading`
 *  render differently, and `match` when the stale error or the `Refreshing`
 *  signal matters.
 */
const matchData: (handlers: {
  onData: (data: A) => D
  onEmpty: Function.LazyArg<B>
  onFailure: (error: E) => C
}) => (self: AsyncData<A, E>) => B | C | D
```

### matchDataSplitEmpty

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L318)

```
/**
 * Like `matchData`, but the collapsed `onEmpty` channel is split in two:
 *  `onIdle` handles `Idle` and `onLoading` handles `Loading`, for views that
 *  render nothing requested yet differently from a request in flight.
 *  `onData` and `onFailure` behave exactly as in `matchData`.
 */
const matchDataSplitEmpty: (handlers: {
  onData: (data: A) => F
  onFailure: (error: E) => D
  onIdle: Function.LazyArg<B>
  onLoading: Function.LazyArg<C>
}) => (self: AsyncData<A, E>) => B | C | D | F
```

### orElse

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L588)

```
/**
 * Returns `self` when it holds data (`Success`, `Refreshing`, or `Stale`),
 *  otherwise `that()`. The recovery and cache-fallback combinator: recover
 *  an `Idle`, `Loading`, or `Failure` into a secondary source without
 *  `match`.
 */
const orElse: (that: LazyArg<AsyncData<A, E>>) => (self: AsyncData<A, E>) => AsyncData<A, E>
```

### settle

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L834)

```
/**
 * Folds a settled `Result` into the previous state, keeping the last good
 *  data: a `Result` success becomes `Success`, and a `Result` failure
 *  becomes `Stale({ error, data })` when `self` holds data, else a bare
 *  `Failure`. The primary way to fold a fetch back into the Model, and the
 *  reason a failed refresh keeps data on screen: without `settle`, every
 *  fetch needs a success arm and a failure arm that hand-assemble
 *  `Success`, `Stale`, and `Failure` from the previous state. When a
 *  failure should deliberately drop the previous data, match on the
 *  `Result` directly and build the `Failure` explicitly.
 */
const settle: (result: Result<A, E>) => (self: AsyncData<A, E>) => AsyncData<A, E>
```

### zipWith

const

[source](https://github.com/foldkit/foldkit/blob/0262c9be1039984c9507a67da72bbc0f380ffcd8/packages/foldkit/src/asyncData/asyncData.ts#L662)

```
/**
 * Combines two values under the two-tier lattice
 *  `Failure > Loading > Idle > Stale > Refreshing > Success`. If either
 *  input is a no-data state, the highest-ranked such state wins with no
 *  combination (`self`'s error wins when both are `Failure`). Otherwise both
 *  hold data: combine with `f` and tag the result with the highest-ranked
 *  data state present (`self`'s error wins when both are `Stale`).
 * 
 *  Because the combined value needs both inputs' data, a single no-data
 *  input collapses the whole combine to that state:
 *  `zipWith(Idle(), Success({ data }), f)` yields `Idle`.
 */
const zipWith: (that: AsyncData<B, E>, f: (selfData: A, thatData: B) => C) => (self: AsyncData<A, E>) => AsyncData<C, E>
```
