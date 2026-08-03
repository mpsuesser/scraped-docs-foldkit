---
url: https://foldkit.dev/core/messages
title: "Messages"
description: "Type-safe events that drive state changes in Foldkit. Messages replace React event handlers with a declarative, traceable pattern."
access_date: 2026-08-03T19:40:01.169Z
current_date: 2026-08-03T19:40:01.169Z
---

# Messages

## Overview

A Message is a fact about something that happened in your application. Not an instruction to do something, just a record of what occurred.

In the [restaurant analogy](https://foldkit.dev/core/architecture#the-restaurant-analogy), “table 3 asked for the check” is a Message. It doesn’t tell the waiter what to do: maybe they bring the check immediately, maybe they offer dessert first. The waiter (the update function) decides. The Message stays the same either way.

`ClickedIncrement` doesn’t say “add one to the count.” It says “the user clicked the increment button.” The update function decides what that means. Maybe today it adds one. Maybe tomorrow it fetches a new count from a server. The Message stays the same.

The counter has three Messages:

```
import { Schema as S } from 'effect'
import { m } from 'foldkit/message'

// MESSAGE

// m() gives you a Message type with a callable constructor
const ClickedDecrement = m('ClickedDecrement')
const ClickedIncrement = m('ClickedIncrement')
const ClickedReset = m('ClickedReset')

const Message = S.Union([ClickedDecrement, ClickedIncrement, ClickedReset])
type Message = typeof Message.Type
```

By convention, Messages follow a verb-first, past-tense naming pattern: `ClickedIncrement`, not `Increment` or `ADD_COUNT`. The verb prefix functions as a category marker, e.g. `Clicked*` for button clicks, `Updated*` for input changes, `Succeeded*` and `Failed*` for Command results, and `Got*` for [Submodel results](https://foldkit.dev/core/submodel).

The `m()` helper creates a `TaggedStruct` with a callable constructor. `m('ClickedIncrement')` gives you a type you can pattern match on and a function you can call to create instances: `ClickedIncrement()`.

Actions without the boilerplate

Messages are similar to Redux action types, but more ergonomic with Effect Schema. Instead of string constants and action creators, you get type inference and pattern matching for free.

Messages describe what happened. But who decides what to do about it? That’s the job of the update function: the single place where your application’s state transitions live.
