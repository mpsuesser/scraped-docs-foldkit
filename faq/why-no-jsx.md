---
url: https://foldkit.dev/faq/why-no-jsx
title: "Why no JSX?"
description: "Why Foldkit uses a typed function-call DSL instead of JSX, with side-by-side comparisons of buttons, inputs, and conditional rendering."
access_date: 2026-08-03T19:45:20.723Z
current_date: 2026-08-03T19:45:20.723Z
---

## Why no JSX?

Foldkit is plain TypeScript. There is no JSX, no transform, no compiler step. The view is built with a typed function-call DSL. Developers coming from JSX often ask why Foldkit doesn’t use it, and whether it could. This is the answer to both.

## Recognition speed vs readability

When a developer says JSX is easier to read, they usually mean they read it faster. That measurement is real. After years of working in JSX, an angle bracket lights up neurons that a function call does not. That is recognition speed. It belongs to the reader, not to the syntax.

Readability is a property of the code: how completely it communicates what it does, what it accepts, and what it can produce. A typed function call wins that comparison. Every attribute is a known constructor. Every event handler returns a known Message type. Children are a typed array, not an opaque variadic.

Familiarity is real, but it is a complaint about ramp-up, not about the syntax. A week into using the DSL, the recognition gap closes and unfamiliarity stops being the bottleneck.

## The DSL in thirty seconds

Each HTML element is a function on the builder `h` that every view receives: `h.div`, `h.button`, `h.p`, `h.input`. Each one returns `Html`. Attributes are passed as an array of typed values, children as an array of `Html | string`. Event handlers like `OnClick` and `OnInput` produce typed Messages. The builder is typed `HtmlBuilder<Message>` for the view's own `Message` union, so every handler built with it is constrained to produce a Message that belongs to that union. The compiler enforces it.

For the full tour of how views work, see [View](https://foldkit.dev/core/view).

## Side by side

### A button with an event

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

The shapes are different. `OnClick` does not take a function. It takes a value of the Message type. That value flows through the entire app, gets logged in DevTools, replays in tests, and lands in `update`. JSX reaches into closures. The DSL hands you a fact.

### An input

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

In JSX you write `(e) => onChange(e.target.value)`. The handler signature leaks the SyntheticEvent shape into your code. In the DSL, `OnInput(value => ...)` extracts the value for you. The handler only cares about the data you actually want.

The DSL ships [typed handlers for the standard HTML event surface](https://foldkit.dev/api-reference/html#type-Html/Attribute). `OnPointerDown` hands you `pointerType, button, screenX, screenY, clientX, clientY`. `OnFileChange` hands you a list of files with metadata. `OnKeyDown` hands you the key and a typed modifier set.

The natural follow-up question is: what if I need a field a typed handler does not expose? Today you cannot reach it through the DSL. The set of handlers is closed. In practice this is rarely the limit you hit, because the curated payloads cover the fields you usually want. The places it does bite are specialized: pen pressure on pointer events for drawing apps, `isComposing` on input events for IME-aware text editors, multi-touch gesture data, and custom events dispatched by third-party widgets. We plan to add a typed escape hatch the way Elm does, with a decoder that fails safely on missing fields, before v1.0.0 ships. Until then, the set is what it is.

### Conditional rendering

Four-way dispatch in JSX:

```
import { Data, Match } from 'effect'

type Status = Data.TaggedEnum<{
  Idle: {}
  Loading: {}
  Failed: { error: string }
  Loaded: { greeting: string }
}>

const Status = Data.taggedEnum<Status>()

function Greeting({ status }: { status: Status }) {
  return (
    <div>
      {Match.value(status).pipe(
        Match.tagsExhaustive({
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

With `Data.TaggedEnum` and `Match` from effect-ts, JSX gets dispatch parity. Both versions are exhaustive. Both fail to compile if you add a fifth variant without handling it. Without those tools, JSX is back to ternaries, `&&`, and extracted helper components, with exhaustiveness on you.

The remaining difference is structural. JSX is an expression syntax, so the match lives inside `{...}` braces inside a wrapping element, and every arm returns a React node. The DSL returns `Html` directly into the children array because the tree already is an array of values. No wrapping required. No expression embedding. The dispatch sits at the same level as everything else in the view.

## Could Foldkit add JSX?

JSX is not magic. A bundler compiles `<div class="x">hi</div>` into a function call like `jsx('div', { class: 'x', children: 'hi' })`, and Foldkit could ship a JSX runtime that maps that props bag onto the same element factories the DSL already uses. Views are plain functions that return virtual nodes. No hooks, no reactivity tracking, no compile-time magic. At runtime, JSX support would be a small adapter.

The blocker is the type system. Everything above about typed handlers rests on one mechanism: each view receives a builder typed `HtmlBuilder<Message>`, parameterizing every element factory by that view's Message union. The runtime supplies the builder for the frame the view renders in, so the union `h` carries is the one the frame's dispatcher routes. TypeScript resolves lowercase JSX tags like `<div>` through `JSX.IntrinsicElements`, a single non-generic interface shared by the whole project. There is no way for `<div onClick={...}>` to infer a Message type from the view function around it.

That leaves three ways to wire it up, and each one gives up something the DSL refuses to give up. Type handlers as `unknown`, and any Message compiles in any view; the guarantee this page advertises is gone. Declare your app’s Message type into `JSX.IntrinsicElements` globally, and the first Submodel breaks it, because a Foldkit codebase routinely has several Message unions live at once and one global type cannot represent that. Or skip lowercase tags entirely and derive capitalized components from the view’s builder, `const { Div, Button } = jsxElements(h)`, which preserves the typing and throws away the familiarity that was the only reason to want JSX.

Even a perfectly typed JSX layer would be a second authoring syntax: a parallel set of docs, examples, and edge cases, with every future view feature needing two mappings. That is a permanent cost paid to soften a ramp-up that closes in a week.

So the precise answer is sharper than a style preference. JSX as syntax is feasible. JSX with the DSL’s guarantee, in the lowercase form people actually want, is not. “Every handler in this subtree produces a Message from this view’s union” is not something JSX’s type model can say. Foldkit isn’t avoiding JSX. It expresses a constraint JSX cannot.
