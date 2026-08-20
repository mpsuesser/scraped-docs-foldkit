---
url: https://foldkit.dev/example-apps
title: "Examples"
description: "Browse working Foldkit applications that cover state, forms, routing, caching, authentication, server rendering, UI components, third-party integrations, and more."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Examples

Each example is available as a starter template via [Create Foldkit App](https://github.com/foldkit/foldkit/tree/main/packages/create-foldkit-app). Pick one that matches what you’re building, or start with Counter and work your way up. See [Getting Started](https://foldkit.dev/get-started/getting-started) to get up and running.

Example

Description

[Counter](https://foldkit.dev/example-apps/counter)

The classic counter example. Increment, decrement, and reset a number.

[Counters](https://foldkit.dev/example-apps/counters)

Add and remove independent Counter Submodels in a dynamic list. Each row is embedded through h.submodel and routed through a wrapper Message.

[Todo](https://foldkit.dev/example-apps/todo)

A todo list persisted in localStorage. Add, complete, and delete tasks.

[Stopwatch](https://foldkit.dev/example-apps/stopwatch)

A stopwatch with start, stop, and reset. Demonstrates a time-based Subscription.

[Crash View](https://foldkit.dev/example-apps/crash-view)

A custom crash fallback with a button that crashes the application and an action that reloads it. Demonstrates crash.view and crash.report.

[Slow Warnings](https://foldkit.dev/example-apps/slow-warnings)

Trigger slow update, view, patch, and Subscription dependency warnings at their default thresholds, then inspect them in a visible log.

[Form](https://foldkit.dev/example-apps/form)

A form with field validation, error states, and asynchronous submission.

[Weather](https://foldkit.dev/example-apps/weather)

Look up weather by ZIP code. Demonstrates HTTP requests and loading states.

[API Cache](https://foldkit.dev/example-apps/api-cache)

Query caching without a query client. Demonstrates stale-while-revalidate, request deduplication, invalidation, and interval refetching.

[Charting](https://foldkit.dev/example-apps/charting)

A live dashboard for public Foldkit telemetry from GitHub and npm. Demonstrates HTTP Commands, asynchronous state, an ECharts Mount adapter, and a Subscription that turns chart clicks into Messages.

[Routing](https://foldkit.dev/example-apps/routing)

A client-routed application with URL parameters, nested routes, rest segments, and navigation.

[Route Transitions](https://foldkit.dev/example-apps/route-transitions)

A live log shows which Transition helper handles each navigation. Entering the gallery loads its catalog once, staying on a painting refetches only when its id changes, and leaving the studio saves a draft.

[Interrupting Commands](https://foldkit.dev/example-apps/interrupting-commands)

Simulated file uploads driven by interruptible Commands. Cancel one upload, cancel every upload in flight, or restart a cancelled upload through a keyed interrupt registry and an outcome-carrying result Message.

[View Transitions](https://foldkit.dev/example-apps/view-transitions)

Animated route changes with the View Transitions API. Transition types control direction-aware slides, and a shared element morphs from gallery card to detail hero.

[Query Sync](https://foldkit.dev/example-apps/query-sync)

A filterable dinosaur table where every control syncs to URL query parameters. Schema transforms accept valid states and replace invalid parameters with declared defaults.

[Snake](https://foldkit.dev/example-apps/snake)

The classic snake game. Keyboard input, game loop, and collision detection.

[Auth](https://foldkit.dev/example-apps/auth)

An authentication flow with Submodels, OutMessage, protected routes, and session management.

[Shopping Cart](https://foldkit.dev/example-apps/shopping-cart)

An e-commerce application with a product listing, cart management, and checkout flow.

[State Machine](https://foldkit.dev/example-apps/state-machine)

A checkout workflow powered by the experimental state machine module. Guards skip Shipping for digital orders, gate Place order behind a complete review, and parse promo codes into applied discounts.

[Pixel Art](https://foldkit.dev/example-apps/pixel-art)

A pixel art editor with immutable undo and redo, time-travel history, Foldkit UI components, lazy views, Subscriptions, Commands that report errors as Messages, and localStorage persistence through Flags.

[Job Application](https://foldkit.dev/example-apps/job-application)

A multi-step form with asynchronous email validation, cross-field date constraints, file uploads, and per-step error indicators.

[WebSocket Chat](https://foldkit.dev/example-apps/websocket-chat)

A ManagedResource owns a WebSocket connection lifecycle. Demonstrates connection state, reconnection, and frames entering update as Messages.

[Managed Resource Layer](https://foldkit.dev/example-apps/managed-resource-layer)

A layer-backed ManagedResource starts a ComputeEngine service from an Effect Layer, exposes it to Commands, and runs Layer finalizers when the Model turns it off.

[Kanban](https://foldkit.dev/example-apps/kanban)

A drag-and-drop kanban board with cross-column reordering, keyboard navigation, fractional indexing, and screen reader announcements.

[Map](https://foldkit.dev/example-apps/map)

An interactive MapLibre GL map with locations, search, and "find my location." Demonstrates a Mount integration with a third-party DOM library, plus a Subscription that turns map movement and marker clicks into Messages.

[Canvas Art](https://foldkit.dev/example-apps/canvas-art)

Click the canvas to spawn bouncing balls. Demonstrates declarative 2D rendering with Canvas.view, animation-frame Subscriptions, and pointer events translated to canvas-local coordinates.

[Generative Art](https://foldkit.dev/example-apps/generative-art)

Move the mouse to stir a Perlin-noise flow field, then click to bloom prismatic particle bursts. Demonstrates Canvas.view with hundreds of evolving Path strokes per frame, Effect Random for spawning, and simulation controls wired through Messages.

[Web Components](https://foldkit.dev/example-apps/web-components)

A QR code designer integrates two third-party web components through CustomElement.define. The color picker emits CustomEvents as Messages, the QR element receives typed properties, and both communicate through the Model.

[Embedding](https://foldkit.dev/example-apps/embedding)

A Foldkit widget embedded in a plain TypeScript host page through Runtime.embed. The host seeds initial state with Flags, pushes a step value in through an inbound Port, mirrors the count the widget emits through an outbound Port, and mounts and unmounts the widget with dispose. All communication crosses one Schema-typed handle; the host never touches the Model.

[Static Site Generation](https://foldkit.dev/example-apps/ssg)

A build script renders every route to static HTML through a server entry, and the client hydrates the served markup in place. The same init, view, and Model produce the build output and the running application.

[Server-Side Rendering](https://foldkit.dev/example-apps/ssr)

A server renders each request into HTML using Flags read from a cookie, and the client hydrates with the exact values the server used. Reload the page and your latest count arrives already in the markup, before any JavaScript runs.

[UI Showcase](https://foldkit.dev/example-apps/ui-showcase)

An interactive showcase of every Foldkit UI component, with styled routed demos and their parent and Submodel wiring.

[Personal Blog](https://foldkit.dev/example-apps/personal-blog)

A blog whose prose lives in Markdown files. The @foldkit/markdown Vite plugin compiles each file into a typed document, per-node view overrides style the result, and directive islands place a live Counter Submodel and Note callout between paragraphs.

[Typing Terminal](https://foldkit.dev/example-apps/typing-terminal)

A production real-time multiplayer typing speed game. A full-stack Effect application with an RPC backend and Foldkit frontend.

[Race your friends →](https://typingterminal.com)
