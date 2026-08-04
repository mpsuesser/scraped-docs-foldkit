---
url: https://foldkit.dev/api-reference/custom-element
title: "CustomElement"
description: "API documentation for the CustomElement module."
access_date: 2026-08-04T23:13:41.872Z
current_date: 2026-08-04T23:13:41.872Z
---

# CustomElement

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/c4822d9ea91727f767b8b52f61a81fcc8ae6a260/packages/foldkit/src/customElement/index.ts#L160)

```
/**
 * Define a typed binding for a custom element. The returned spec describes
 * the element's properties and events with Schema, and exposes a
 * `.withMessage<Message>()` factory that yields a typed `ElementBuilder` for
 * the consumer's Message universe.
 * 
 * Property changes diff across renders; declared `CustomEvent`s are
 * converted to Messages by the runtime.
 */
<Tag extends string, Properties extends Record<string, Top>, Events extends Record<string, Top>>(config: CustomElementConfig<Tag, Properties, Events>): CustomElementSpec<Tag, Properties, Events>
```

## Types

### Builder

type

[source](https://github.com/foldkit/foldkit/blob/c4822d9ea91727f767b8b52f61a81fcc8ae6a260/packages/foldkit/src/customElement/index.ts#L90)

```
/**
 * The typed builder for a given spec and Message universe. Equivalent to
 *  the value `Spec.withMessage<Message>()` returns, expressed as a type
 *  consumers can name without reaching for `ReturnType<typeof ...>`.
 */
type Builder = Spec extends CustomElementSpec<string, infer Properties, infer Events>
  ? ElementBuilder<Message, Properties, Events>
  : never
```

### ElementBuilder

type

[source](https://github.com/foldkit/foldkit/blob/c4822d9ea91727f767b8b52f61a81fcc8ae6a260/packages/foldkit/src/customElement/index.ts#L46)

```
/**
 * Typed call site for a defined custom element. The element constructor
 *  itself is callable; each declared property gets a PascalCase factory
 *  method, and each declared event gets an `On{PascalCase}` factory method.
 */
type ElementBuilder = (attributes?: ReadonlyArray<Attribute<Message>>, children?: ReadonlyArray<Child>) => Html & PropertyFactories<Message, Properties> & EventFactories<Message, Events>
```

## Interfaces

### CustomElementConfig

interface

[source](https://github.com/foldkit/foldkit/blob/c4822d9ea91727f767b8b52f61a81fcc8ae6a260/packages/foldkit/src/customElement/index.ts#L58)

```
/** Configuration accepted by `CustomElement.define`. */
interface CustomElementConfig {
  events: Events
  properties: Properties
  tag: Tag
}
```

### CustomElementSpec

interface

[source](https://github.com/foldkit/foldkit/blob/c4822d9ea91727f767b8b52f61a81fcc8ae6a260/packages/foldkit/src/customElement/index.ts#L74)

```
/**
 * A defined custom element, untyped on Message at definition time so the
 *  spec can be exported and shared across modules. Call `.withMessage(h)`
 *  with a view's builder to mint a typed `ElementBuilder` bound to that
 *  frame's Message universe. The builder argument is a type witness: it
 *  fixes `Message` to the frame the element renders in, so the spec cannot
 *  be bound to a Message universe its events will not dispatch into.
 */
interface CustomElementSpec {
  events: Events
  properties: Properties
  tag: Tag
  withMessage: (h: HtmlBuilder<Message>) => ElementBuilder<Message, Properties, Events>
}
```
