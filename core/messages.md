---
url: https://foldkit.dev/core/messages
title: "Messages"
description: "Define the facts update can handle as a Schema-backed Message union, with naming conventions for user actions, Command results, and Submodel wrappers."
access_date: 2026-09-02T07:05:07.578Z
current_date: 2026-09-02T07:05:07.578Z
---

# Messages

## Facts, Not Instructions

A Message records something that happened in the application. It does not prescribe the response. The update function decides what the fact means for the current Model.

`ClickedIncrement` does not mean “add one.” It records that the user clicked the increment button. In this counter, update adds one. A later version may return a Command that obtains the next value elsewhere. The Message remains a stable account of the event.

The counter has three Messages:

```
import { Schema } from 'effect'
import { defineMessageUnion } from 'foldkit/message'

// MESSAGE

// defineMessageUnion() declares the union and its callable constructors together

const Message = defineMessageUnion({
  ClickedDecrement: {},
  ClickedIncrement: {},
  ClickedReset: {},
})
type Message = typeof Message.Type
```

Messages use verb-first, past-tense names such as `ClickedIncrement`, not `Increment` or `ADD_COUNT`. Prefixes make their causes easy to scan. `Clicked*` records clicks, and `Updated*` records input changes. Command results use `Succeeded*` or `Failed*` when the distinction matters, and `Completed*` otherwise. `Got*` is reserved for results lifted from a child [Submodel](https://foldkit.dev/core/submodel).

The `defineMessageUnion()` helper declares the whole union in one place. Each key becomes a callable constructor on the union, so `Message.ClickedIncrement()` creates the value and `Message.match(message, handlers)` handles every variant exhaustively. Do not destructure the constructors. Keeping `Message` or `OutMessage` at the call site makes the owning domain explicit.

Name the cause

A Message says what happened, not what update intends to do next. That keeps the same fact useful when the application’s response changes.

Messages describe what happened. The [update function](https://foldkit.dev/core/update) defines every resulting state transition.
