---
url: https://foldkit.dev/core/custom-element
title: "CustomElement"
description: "Bind native web components to Foldkit with CustomElement.define. Declare properties and events with Schema once and get a typed builder back."
access_date: 2026-08-19T19:38:38.072Z
current_date: 2026-08-19T19:38:38.072Z
---

# CustomElement

## Overview

Native web components are browser elements with hyphenated tags, JavaScript properties, and `CustomEvent`s. They can render into a shadow DOM and own their internal keyboard, pointer, and display behavior.

`CustomElement.define` gives that declarative interface a typed Foldkit binding. Properties become PascalCase factories on an element builder, events become `On{PascalCase}` factories, and the builder renders beside standard elements such as `h.div` and `h.button`. Properties flow from the Model into the element; event details return as Messages.

A declarative boundary

The browser owns the custom element's implementation. Foldkit owns the typed property and event wiring. The view remains a pure function from Model to VNode, with no manual property assignment or separate Mount.

## Defining a Binding

Foldkit defines the typed binding, but the browser still needs the element's class. Third-party packages commonly register it through a side-effect import. For example: importing `vanilla-colorful/hex-color-picker.js` calls `customElements.define` for `<hex-color-picker>`. A custom element you author follows the same browser registration step.

`CustomElement.define` takes one config object with the element's `tag`, a record of JavaScript `properties`, and a record of custom `events`. Each property and event payload is described with Schema. The resulting spec is independent of any application Message type, so it can be exported and shared.

Inside a view, call `.withMessage(h)` on the spec. The view builder acts as a type witness that binds event handlers to the current Message universe. The runtime builder is reusable after that binding.

```
import { Schema as S } from 'effect'
import { CustomElement } from 'foldkit'
import type { Html, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import 'vanilla-colorful/hex-color-picker.js'

import '@shoelace-style/shoelace/dist/components/qr-code/qr-code.js'

// The two side-effect imports above register each custom element with
// the browser. Foldkit does not call customElements.define for you;
// most third-party packages do it as a side effect on import. If you
// author the class yourself, you call customElements.define once next
// to the class.

// Declare a typed Foldkit binding for each element. Properties become
// PascalCase factories on the builder, events become On{PascalCase}
// factories, all checked against the declared Schema.

const hexColorPicker = CustomElement.define({
  tag: 'hex-color-picker',
  properties: {
    color: S.String,
  },
  events: {
    'color-changed': S.Struct({ value: S.String }),
  },
})

const qrCode = CustomElement.define({
  tag: 'sl-qr-code',
  properties: {
    value: S.String,
    fill: S.String,
    background: S.String,
    size: S.Number,
  },
  events: {},
})

const ChangedFillColor = m('ChangedFillColor', { value: S.String })

const Message = S.Union([ChangedFillColor])
type Message = typeof Message.Type

// Inside a view, mint typed builders with `withMessage(h)`. The view's
// builder fixes the Message universe, so the elements' event factories can
// only produce Messages the surrounding frame dispatches.
//
// Use the builders inline next to standard elements. Property factories
// write JS properties through Snabbdom's propsModule. Event factories
// convert kebab-case CustomEvents into Messages. The picker and the QR
// never talk directly; they share state through the Model.

export const designerView = (
  model: { content: string; fillColor: string },
  h: HtmlBuilder<Message>,
): Html => {
  const fillPicker = hexColorPicker.withMessage(h)
  const qr = qrCode.withMessage(h)

  return h.div(
    [h.Class('flex gap-6')],
    [
      fillPicker([
        fillPicker.Color(model.fillColor),
        fillPicker.OnColorChanged(detail =>
          ChangedFillColor({ value: detail.value }),
        ),
      ]),
      qr([
        qr.Value(model.content),
        qr.Fill(model.fillColor),
        qr.Background('#ffffff'),
        qr.Size(200),
      ]),
    ],
  )
}
```

The bound element builder is callable. Pass attributes, including generated property and event factories, as the first argument and children as the second. Either argument can be omitted when it is not needed. Schema keeps both directions typed: a property factory accepts its declared value, while an event factory receives its declared `detail` and returns a Message.

## Properties and Events

Each property name becomes a PascalCase factory. For example: `value` becomes `Value`, and `isDisabled` becomes `IsDisabled`. The factory writes a JavaScript property on the live element through `propsModule`, not an HTML attribute. This allows values such as arrays and objects, and Foldkit only writes a property again when its value changes across renders. When primitive properties are removed, booleans reset to `false`, strings to `''`, and numbers to `0`.

Each kebab-case event name becomes an `On{PascalCase}` factory. For example: `color-changed` becomes `OnColorChanged`. Its callback receives the typed `detail` from a `CustomEvent` and returns the Message Foldkit dispatches.

Validation runs at define time

`CustomElement.define` validates the custom tag, property names, and event names when the binding is created. It also rejects generated factory-name collisions. For example: an `onClick` property and a `click` event would both produce `OnClick`, so the definition throws before the view can render it.

## When to Reach for CustomElement

Use CustomElement when the foreign element exposes a declarative contract: JavaScript properties go in and `CustomEvent`s come out. The element owns its rendering and internal state, while the application shares state through the Model. Most web component libraries built with tools such as Lit or Stencil fit this shape.

Use [Mount](https://foldkit.dev/core/mount) when the foreign API is imperative. Instantiating a map in a container, calling methods on an editor, or pairing setup with a destructor requires the live `Element` and an element-scoped Effect. The [Map example](https://foldkit.dev/example-apps/map) shows that lifecycle.

The [Web Components example](https://foldkit.dev/example-apps/web-components) shows the declarative alternative. A `<hex-color-picker>` emits color changes as Messages, while `<sl-qr-code>` receives typed properties diffed from the Model. The elements never coordinate directly; update connects them through application state.
