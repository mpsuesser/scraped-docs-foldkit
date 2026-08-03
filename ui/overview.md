---
url: https://foldkit.dev/ui/overview
title: "Foldkit UI"
description: "Headless, accessible UI components for Foldkit: dialog, menu, tabs, listbox, and more. Built for The Elm Architecture with Effect-TS."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

# Foldkit UI

## What is Foldkit UI?

Foldkit UI is a set of headless, accessible UI components. Each component is renderless. You provide the markup and styling through a toView callback, and Foldkit UI provides the accessibility attributes, keyboard navigation, and (where applicable) state management.

## Installation

`@foldkit/ui` is a separate package from `foldkit`. Projects scaffolded by `create-foldkit-app` from an example that uses UI components already include it. Its peer dependencies are `foldkit` and `effect`, which every Foldkit project already has. Add it to any other project with:

```sh
npm install @foldkit/ui
```

## Two categories

Foldkit UI components fall into two categories, distinguished by whether they carry state.

Stateful [Submodels](https://foldkit.dev/core/submodel) (Menu, Listbox, Combobox, Calendar, Dialog, Popover, among others) manage their own Model, Message, update, and OutMessage. You embed them via h.submodel and handle their events by pattern-matching the OutMessage in your update.

Stateless render helpers (Button, Input, Textarea, Select, Radio Group, Checkbox, Switch, Disclosure, Fieldset, Nav) are called directly with a ViewConfig and your builder, and return Html. They bundle ARIA and data attributes onto consumer-rendered DOM. No Model, no h.submodel wiring. The controlled helpers dispatch the Messages returned by their config callbacks, and the builder you pass is what determines the Message type those callbacks must return, so there is no type argument to write. The “Kind” column in the table below marks which is which.

## Components

Component

Kind

Description

[Button](https://foldkit.dev/ui/button)

Helper

Accessible button with consistent ARIA attributes and data-attribute hooks for styling.

[Input](https://foldkit.dev/ui/input)

Helper

Text input with ARIA label/description linking and data-attribute hooks.

[Textarea](https://foldkit.dev/ui/textarea)

Helper

Multi-line text input with ARIA label/description linking and data-attribute hooks.

[Checkbox](https://foldkit.dev/ui/checkbox)

Helper

Toggle with accessible labeling, keyboard support, indeterminate state, and optional form integration.

[Fieldset](https://foldkit.dev/ui/fieldset)

Helper

Groups related form controls with a legend and description. Disabled state propagates to all children.

[Radio Group](https://foldkit.dev/ui/radio-group)

Helper

Controlled radio options with roving tabindex, keyboard navigation, and per-option label/description linking.

[Switch](https://foldkit.dev/ui/switch)

Helper

On/off toggle with accessible labeling, keyboard support, and optional form integration.

[Slider](https://foldkit.dev/ui/slider)

Submodel

Numeric range input with pointer drag, keyboard step / page / home / end navigation, and ARIA slider semantics.

[Select](https://foldkit.dev/ui/select)

Helper

Native select wrapper with ARIA label/description linking and data-attribute hooks.

[Listbox](https://foldkit.dev/ui/listbox)

Submodel

Custom select dropdown with persistent selection, keyboard navigation, and typeahead search.

[Combobox](https://foldkit.dev/ui/combobox)

Submodel

Autocomplete input with filtering, keyboard navigation, and custom rendering.

[Dialog](https://foldkit.dev/ui/dialog)

Submodel

Modal dialog using native <dialog> with focus trapping, backdrop, and scroll locking.

[Menu](https://foldkit.dev/ui/menu)

Submodel

Dropdown menu with keyboard navigation, typeahead search, and aria-activedescendant focus.

[Popover](https://foldkit.dev/ui/popover)

Submodel

Floating panel with arbitrary content and natural Tab navigation.

[Disclosure](https://foldkit.dev/ui/disclosure)

Helper

Show/hide toggle for building collapsible sections like FAQs and accordions.

[Tabs](https://foldkit.dev/ui/tabs)

Submodel

Tabbed interface with keyboard navigation, Home/End support, and wrapping.

[Nav](https://foldkit.dev/ui/nav)

Helper

Stateless, URL-driven navigation landmark whose items are links, marking the current destination with aria-current="page".

[Drag and Drop](https://foldkit.dev/ui/drag-and-drop)

Submodel

Sortable lists and cross-container movement with pointer tracking, keyboard navigation, auto-scrolling, and screen reader announcements.

[File Drop](https://foldkit.dev/ui/file-drop)

Submodel

File input with drag-and-drop support, configurable accept patterns, and multiple-file mode. Emits typed OutMessages for received files and non-file drops.

[Calendar](https://foldkit.dev/ui/calendar)

Submodel

Inline calendar grid with 2D keyboard navigation, locale-aware headers, min/max constraints, and disabled-date support. Foundation for date pickers.

[Date Picker](https://foldkit.dev/ui/date-picker)

Submodel

Input paired with a popover Calendar. Inherits the calendar’s constraint and keyboard-navigation support, with programmatic open/close and setters.

[Animation](https://foldkit.dev/ui/animation)

Submodel

Coordinates CSS enter/leave animations via a state machine and data attributes. Works with both CSS transitions and CSS keyframe animations. Sends an OutMessage when the leave animation completes.

## Showcase

The [UI Showcase](https://foldkit.dev/example-apps/ui-showcase) example demonstrates every component with styled, interactive examples. It’s a good reference for how to wire up component state, handle Messages, and compose views.
