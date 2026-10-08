---
url: https://foldkit.dev/api-reference/experimental-query
title: "Experimental/Query"
description: "API documentation for the Experimental/Query module."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

# Experimental/Query

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/experimental/query/define.ts#L45)

```
/**
 * Defines a Submodel that fetches data and retains it in the application
 * Model. Add `args` to define a KeyedQuery; omit them to define a
 * Query.
 * 
 *  Ships from `foldkit/experimental/query`; expect breaking changes while the API settles.
 */
<Name extends string, A, AI, E, EI, Fields extends SyncFields, R = never>(config: KeyedQueryConfig<Name, A, AI, E, EI, Fields, R>): KeyedQuery<Name, A, AI, E, EI, Fields, R>

<Name extends string, A, AI, E, EI, R = never>(config: Readonly<{
  data: Codec<A, AI, never, never>
  error: Codec<E, EI, never, never>
  execute: Effect<A, E, R>
  name: Name
}> & {
  args: undefined
  toKey: undefined
}): Query<Name, A, AI, E, EI, R>
```

## Interfaces

### KeyedQuery

interface

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/experimental/query/keyedQuery.ts#L166)

```
/**
 * Submodel for fetching and retaining `AsyncData` values by argument key.
 * 
 *  Ships from `foldkit/experimental/query`; expect breaking changes while the API settles.
 */
interface KeyedQuery {
  Fetch: CommandDefinitionWithArgs<`Fetch${Name}`, {
    args: Struct<Fields>
    generation: Number
  }, Effect<{
    _tag: "CompletedFetch"
    args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
    generation: number
    result: Result<A, E>
  }, never, R>>
  init: () => {
    entries: HashMap<string, {
      args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
      data: AsyncData<A, E>
      generation: number
    }>
    generation: number
  }
  lift: LiftKeyedQuery<{
    entries: HashMap<string, {
      args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
      data: AsyncData<A, E>
      generation: number
    }>
    generation: number
  }, {
    _tag: "CompletedFetch"
    args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
    generation: number
    result: Result<A, E>
  }, View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>, R>
  loadIfMissing: Fold<{
    entries: HashMap<string, {
      args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
      data: AsyncData<A, E>
      generation: number
    }>
    generation: number
  }, {
    _tag: "CompletedFetch"
    args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
    generation: number
    result: Result<A, E>
  }, View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>, R>
  Message: MessageUnion<{
    CompletedFetch: {
      args: Struct<Fields>
      generation: Number
      result: Result<Codec<A, AI, never, never>, Codec<E, EI, never, never>>
    }
  }>
  Model: Struct<{
    entries: HashMap<String, Struct<{
      args: Struct<Fields>
      data: Codec<AsyncData<A, E>, AsyncDataEncoded<AI, EI>, never, never>
      generation: Number
    }>>
    generation: Number
  }>
  read: (model: {
    entries: HashMap<string, {
      args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
      data: AsyncData<A, E>
      generation: number
    }>
    generation: number
  }, args: KeyedArgs<Fields>) => AsyncData<A, E>
  reset: (model: {
    entries: HashMap<string, {
      args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
      data: AsyncData<A, E>
      generation: number
    }>
    generation: number
  }) => Update.Return<KeyedQueryModel<A, AI, E, EI, Fields>["Type"], KeyedQueryMessage<A, AI, E, EI, Fields>["Type"]>
  revalidate: Fold<{
    entries: HashMap<string, {
      args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
      data: AsyncData<A, E>
      generation: number
    }>
    generation: number
  }, {
    _tag: "CompletedFetch"
    args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
    generation: number
    result: Result<A, E>
  }, View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>, R>
  revalidateOrLoad: Fold<{
    entries: HashMap<string, {
      args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
      data: AsyncData<A, E>
      generation: number
    }>
    generation: number
  }, {
    _tag: "CompletedFetch"
    args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
    generation: number
    result: Result<A, E>
  }, View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>, R>
  run: (args: KeyedArgs<Fields>) => Effect<AsyncData<A, E>, never, R>
  update: (model: {
    entries: HashMap<string, {
      args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
      data: AsyncData<A, E>
      generation: number
    }>
    generation: number
  }, message: {
    _tag: "CompletedFetch"
    args: View<Fields, "Type", TypeOptionalKeys<Fields>, TypeMutableKeys<Fields>>
    generation: number
    result: Result<A, E>
  }) => Update.Return<KeyedQueryModel<A, AI, E, EI, Fields>["Type"], KeyedQueryMessage<A, AI, E, EI, Fields>["Type"]>
}
```

### Query

interface

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/foldkit/src/experimental/query/query.ts#L72)

```
/**
 * Submodel for fetching and retaining one `AsyncData` value.
 * 
 *  Ships from `foldkit/experimental/query`; expect breaking changes while the API settles.
 */
interface Query {
  Fetch: CommandDefinitionWithArgs<`Fetch${Name}`, {
    generation: Number
  }, Effect<{
    _tag: "CompletedFetch"
    generation: number
    result: Result<A, E>
  }, never, R>>
  init: () => {
    data: AsyncData<A, E>
    generation: number
  }
  lift: LiftQuery<{
    data: AsyncData<A, E>
    generation: number
  }, {
    _tag: "CompletedFetch"
    generation: number
    result: Result<A, E>
  }, R>
  loadIfMissing: (model: {
    data: AsyncData<A, E>
    generation: number
  }) => Update.Return<QueryModel<A, AI, E, EI>["Type"], QueryMessage<A, AI, E, EI>["Type"], R>
  Message: MessageUnion<{
    CompletedFetch: {
      generation: Number
      result: Result<Codec<A, AI, never, never>, Codec<E, EI, never, never>>
    }
  }>
  Model: Struct<{
    data: Codec<AsyncData<A, E>, AsyncDataEncoded<AI, EI>, never, never>
    generation: Number
  }>
  read: (model: {
    data: AsyncData<A, E>
    generation: number
  }) => AsyncData<A, E>
  reset: (model: {
    data: AsyncData<A, E>
    generation: number
  }) => Update.Return<QueryModel<A, AI, E, EI>["Type"], QueryMessage<A, AI, E, EI>["Type"]>
  revalidate: (model: {
    data: AsyncData<A, E>
    generation: number
  }) => Update.Return<QueryModel<A, AI, E, EI>["Type"], QueryMessage<A, AI, E, EI>["Type"], R>
  revalidateOrLoad: (model: {
    data: AsyncData<A, E>
    generation: number
  }) => Update.Return<QueryModel<A, AI, E, EI>["Type"], QueryMessage<A, AI, E, EI>["Type"], R>
  run: Effect<AsyncData<A, E>, never, R>
  update: (model: {
    data: AsyncData<A, E>
    generation: number
  }, message: {
    _tag: "CompletedFetch"
    generation: number
    result: Result<A, E>
  }) => Update.Return<QueryModel<A, AI, E, EI>["Type"], QueryMessage<A, AI, E, EI>["Type"]>
}
```
