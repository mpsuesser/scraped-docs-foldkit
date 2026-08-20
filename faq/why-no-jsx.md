---
url: https://foldkit.dev/faq/why-no-jsx
title: "Why no JSX?"
description: "Why Foldkit uses a typed function-call DSL instead of JSX, with side-by-side comparisons of buttons, inputs, and conditional rendering."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

Foldkit views use plain TypeScript function calls instead of JSX. They do not need a JSX transform or a JSX runtime. Foldkit applications do still use the required `@foldkit/vite-plugin`, which transforms application functions to assign view identity and provides Model-preserving hot reload. The choice on this page is about the view authoring syntax, not whether application code passes through build tooling.

## Familiarity and Type Safety

JSX has a familiarity advantage. Developers who have used it for years recognize an element tree immediately, while a function-call DSL takes time to learn.

Foldkit chooses a different advantage. Each view receives an `HtmlBuilder<Message>`, so ordinary TypeScript generics connect the view's Message union to every event attribute it creates. Attribute constructors also carry their value types, and children are explicit arrays of `Html | string`.

Whether that syntax feels easier to read depends on the reader. The reason Foldkit uses it is more specific: the builder preserves a local Message constraint that lowercase JSX cannot express.

## DSL Basics

The builder passed to a view contains a function for each HTML element: `h.div`, `h.button`, `h.p`, and `h.input`. Attributes are an array of typed constructors. Children, when an element has any, are a second array. Each element returns `Html`.

Event attributes either take a Message value or translate a curated event payload into one. Because the builder is typed for the view's Message union, a handler cannot dispatch a Message from some other part of the application.

For a complete introduction, see [View](https://foldkit.dev/core/view).

## Side-by-Side Examples

These comparisons use React, the JSX environment most readers know. JSX syntax itself does not define event behavior.

### Button Events

A button with a click handler in JSX:

```
function SaveButton({
  isSaving,
  onSave,
}: {
  isSaving: boolean
  onSave: () => void
}) {
  return (
    <button type="button" disabled={isSaving} onClick={onSave}>
      Save
    </button>
  )
}
```

The same button in the Foldkit DSL:

```
import { Schema as S } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'

const ClickedSave = m('ClickedSave')

const Message = S.Union([ClickedSave])
type Message = typeof Message.Type

const saveButton = (isSaving: boolean, h: HtmlBuilder<Message>) =>
  h.button(
    [h.Type('button'), h.Disabled(isSaving), h.OnClick(ClickedSave())],
    ['Save'],
  )
```

In JSX, `onClick` can run whatever callback the parent supplied. Foldkit's `OnClick` accepts a Message value. The Runtime dispatches that Message, update handles it, and DevTools can record it. The event stays on the same explicit data path as every other state change.

### Input Values

An email input in JSX:

```
function EmailInput({
  email,
  onChange,
}: {
  email: string
  onChange: (value: string) => void
}) {
  return (
    <input
      type="email"
      value={email}
      placeholder="you@example.com"
      onChange={e => onChange(e.target.value)}
    />
  )
}
```

The same input in the DSL:

```
import { Schema as S } from 'effect'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'

const InputtedEmail = m('InputtedEmail', { value: S.String })

const Message = S.Union([InputtedEmail])
type Message = typeof Message.Type

const emailInput = (email: string, h: HtmlBuilder<Message>) =>
  h.input([
    h.Type('email'),
    h.Value(email),
    h.Placeholder('you@example.com'),
    h.OnInput(value => InputtedEmail({ value })),
  ])
```

The React handler receives an event object, so the example extracts `e.target.value` before calling the application callback. Foldkit's `OnInput` extracts the string first. Its translator only receives the value it needs and returns a Message.

The DSL provides [typed handlers for the standard HTML event surface](https://foldkit.dev/api-reference/html#type-Html/Attribute). For example: `OnPointerDown` passes the pointer type, button, screen and client coordinates, and timestamp; `OnFileChange` passes a `ReadonlyArray<File>`; and `OnKeyDown` passes the key with typed Shift, Control, Alt, and Meta state.

Those curated payloads are intentionally narrower than the browser's event objects. `OnPointerDown` does not expose pen pressure, `OnInput` does not expose `isComposing`, and the pointer handlers do not expose a touch list. Foldkit does not currently provide a general native-event decoder. When an interaction needs another native field, a [`Mount.defineStream`](https://foldkit.dev/core/mount) can attach a listener to the live element and emit Messages for its lifetime.

Third-party web component events use a different path. [`CustomElement.define`](https://foldkit.dev/core/custom-element) describes each `CustomEvent` payload with Schema and creates typed `On*` attributes for it.

### Conditional Rendering

Four-way dispatch in JSX:

```
import { Match as M, Schema as S } from 'effect'

const Idle = S.TaggedStruct('Idle', {})
const Loading = S.TaggedStruct('Loading', {})
const Failed = S.TaggedStruct('Failed', { error: S.String })
const Loaded = S.TaggedStruct('Loaded', { greeting: S.String })

const Status = S.Union([Idle, Loading, Failed, Loaded])
type Status = typeof Status.Type

function Greeting({ status }: { status: Status }) {
  return (
    <div>
      {M.value(status).pipe(
        M.tagsExhaustive({
          Idle: () => null,
          Loading: () => <p>Loading…</p>,
          Failed: ({ error }) => <p>Sorry: {error}</p>,
          Loaded: ({ greeting }) => <p>{greeting}</p>,
        }),
      )}
    </div>
  )
}
```

The same dispatch in the DSL:

```
import { Match as M, Schema as S } from 'effect'
import { inertHtml as ih } from 'foldkit/html'

const Idle = S.TaggedStruct('Idle', {})
const Loading = S.TaggedStruct('Loading', {})
const Failed = S.TaggedStruct('Failed', { error: S.String })
const Loaded = S.TaggedStruct('Loaded', { greeting: S.String })

const Status = S.Union([Idle, Loading, Failed, Loaded])
type Status = typeof Status.Type

const greetingView = (status: Status) =>
  ih.div(
    [],
    [
      M.value(status).pipe(
        M.tagsExhaustive({
          Idle: () => ih.empty,
          Loading: () => ih.p([], ['Loading…']),
          Failed: ({ error }) => ih.p([], [\`Sorry: ${error}\`]),
          Loaded: ({ greeting }) => ih.p([], [greeting]),
        }),
      ),
    ],
  )
```

Both examples use Effect's Match over a tagged Schema union, so both are exhaustive. Adding another status without adding an arm fails to compile in either version.

The syntactic difference is where the matched value goes. JSX embeds it between braces inside the element. The DSL places the resulting `Html` directly in the children array. The exhaustive match does the same job in both.

## The JSX Type Constraint

A JSX runtime for Foldkit would be technically possible. A JSX transform turns `<div class="x">hi</div>` into a function call, and an adapter could map the resulting props object onto Foldkit's element factories.

The difficult part is preserving the Message guarantee. A Foldkit view receives an `HtmlBuilder<Message>` value, so the view's Message type flows through normal generic function calls. TypeScript checks a lowercase JSX tag such as `<div>` against the `JSX.IntrinsicElements` type selected for that source file. That lookup cannot take a type argument from the view function surrounding the tag.

The available designs all weaken the reason for adding JSX:

- Type event props broadly, and a view can dispatch a Message that does not belong to its union.
- Configure JSX around one application Message type, and files containing multiple Submodel Message unions lose the local guarantee.
- Use capitalized generic components derived from the view's builder, and the result no longer has the lowercase HTML syntax developers expect from JSX.

A JSX layer would also become a second authoring surface. Documentation, examples, event mappings, and future view features would need to cover both syntaxes.

The decision is not that function calls are universally easier to read. It is that the DSL expresses the view's local Message constraint directly, while lowercase JSX does not. Foldkit keeps that guarantee instead of adding a more familiar syntax that weakens it.
