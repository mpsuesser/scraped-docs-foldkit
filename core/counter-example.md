---
url: https://foldkit.dev/core/counter-example
title: "Counter Example"
description: "Build and trace a minimal Counter through its Model, Message Schema, update, view, init, and Runtime wiring."
access_date: 2026-10-05T07:06:39.496Z
current_date: 2026-10-05T07:06:39.496Z
---

# A Simple Counter Example

## See the Whole Loop

This counter puts the core loop from [Architecture](https://foldkit.dev/core/architecture) into one small application. Its Model holds the count. Its Messages record button clicks. Its update function decides the next count, and its view renders the result.

The example uses two files. `src/main.ts` holds the pure application definitions: Model, Messages, update, init, and view. Larger applications can split those definitions into focused modules. `src/entry.ts` remains the runtime boundary, so tests can import the application without starting it as a side effect.

Counter main.ts

The entry imports those definitions and passes them to `Runtime.makeApplication`. `Runtime.run` then starts the application in the selected container.

Counter entry.ts

Read the example once for its shape. The next four pages examine the [Model](https://foldkit.dev/core/model), [Messages](https://foldkit.dev/core/messages), [update](https://foldkit.dev/core/update), and [view](https://foldkit.dev/core/view) in order. Later pages extend the same counter with a delayed reset, automatic counting, and saved state to introduce side effects and ongoing work.

Start with the Model, the single data structure that describes the application right now.
