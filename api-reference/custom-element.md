---
url: https://foldkit.dev/api-reference/custom-element
title: "CustomElement"
description: "API documentation for the CustomElement module."
access_date: 2026-09-12T22:55:23.086Z
current_date: 2026-09-12T22:55:23.086Z
---

# CustomElement

## Functions

### define

function

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/foldkit/src/customElement/index.ts#L195)

```
/**
 * Define a typed binding for a custom element. The returned spec describes
 * the element's properties and events with Schema, and exposes a
 * `.withMessage<Message>()` factory that yields a typed `ElementBuilder` for
 * the consumer's Message universe.
 * 
 * Property changes diff across renders; declared `CustomEvent` details are
 * decoded against their Schema before the runtime converts them to Messages.
 * A detail the Schema rejects is reported to the console and dispatches no
 * Message.
 */
<Tag extends string, Properties extends Record<string, Top>, Events extends Record<string, EventSchema>>(config: CustomElementConfig<Tag, Properties, Events>): CustomElementSpec<Tag, Properties, Events>
```

## Types

### Builder

type

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/foldkit/src/customElement/index.ts#L104)

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

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/foldkit/src/customElement/index.ts#L60)

```
/**
 * Typed call site for a defined custom element. The element constructor
 *  itself is callable; each declared property gets a PascalCase factory
 *  method, and each declared event gets an `On{PascalCase}` factory method.
 *  The attribute array accepts ChildAttribute alongside
 *  `Attribute<Message>`, like every html element builder, so a Submodel's
 *  published attribute groups can be spread into a custom element.
 */
type ElementBuilder = (attributes?: ReadonlyArray<Attribute<Message> | ChildAttribute>, children?: ReadonlyArray<Child>) => Html & PropertyFactories<Message, Properties> & EventFactories<Message, Events>
```

### EventSchema

type

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/foldkit/src/customElement/index.ts#L34)

```
/**
 * Constraint on a declared event's `detail` Schema. The runtime decodes
 * `detail` synchronously inside the DOM event handler, where there is no
 * Effect context to draw from, so a Schema requiring decoding services
 * cannot describe an event payload.
 */
type EventSchema = Schema.Codec<unknown, unknown, never, unknown>
```

## Interfaces

### CustomElementConfig

interface

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/foldkit/src/customElement/index.ts#L72)

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

[source](https://github.com/foldkit/foldkit/blob/a124b3451a885f2e58b4477adcd04b2ef9e4795a/packages/foldkit/src/customElement/index.ts#L88)

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
