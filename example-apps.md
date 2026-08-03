---
url: https://foldkit.dev/example-apps
title: "Example Apps"
description: "Working Foldkit example apps: counter, forms, routing, auth, websocket chat, and more. Each demonstrates Effect-TS and Elm Architecture patterns."
access_date: 2026-08-03T18:55:49.002Z
current_date: 2026-08-03T18:55:49.002Z
---

# Examples

Each example is available as a starter template via [Create Foldkit App](https://github.com/foldkit/foldkit/tree/main/packages/create-foldkit-app). Pick one that matches what you’re building, or start with Counter and work your way up. See [Getting Started](https://foldkit.dev/get-started/getting-started) to get up and running.

Example

Description

[Counter](https://foldkit.dev/example-apps/counter)

The classic counter example. Increment, decrement, and reset a number.

[Counters](https://foldkit.dev/example-apps/counters)

A dynamic list of Counter Submodels. Add and remove rows; each row is an independent Submodel embedded via h.submodel, with per-instance routing via a wrapper Message.

[Todo](https://foldkit.dev/example-apps/todo)

A todo list with local storage persistence. Add, complete, and delete tasks.

[Stopwatch](https://foldkit.dev/example-apps/stopwatch)

A stopwatch with start, stop, and reset. Demonstrates time-based subscriptions.

[Crash View](https://foldkit.dev/example-apps/crash-view)

Custom crash fallback UI. Demonstrates crash.view and crash.report with a crash button and reload.

[Slow Warnings](https://foldkit.dev/example-apps/slow-warnings)

Interactive lab for triggering slow update, view, patch, and Subscription dependency warnings with default thresholds and a visible warning log.

[Form](https://foldkit.dev/example-apps/form)

Form handling with field validation, error states, and async submission.

[Weather](https://foldkit.dev/example-apps/weather)

Look up weather by zip code. Demonstrates HTTP requests and loading states.

[API Cache](https://foldkit.dev/example-apps/api-cache)

Query caching without a query client. Demonstrates stale-while-revalidate, request deduplication, invalidation, and interval refetching.

[Charting](https://foldkit.dev/example-apps/charting)

Live dashboard for public Foldkit telemetry from GitHub and npm. Demonstrates HTTP Commands, async state, an ECharts Mount adapter, and a Subscription that turns chart clicks back into Messages.

[Routing](https://foldkit.dev/example-apps/routing)

Client-side routing with URL parameters, nested routes, rest segments, and navigation.

[Route Transitions](https://foldkit.dev/example-apps/route-transitions)

Live log of every navigation, narrated by the Transition helpers. Entering the gallery loads the catalog once, the stayed helper refetches a painting only when its id changes, and the exited helper saves a draft when you leave the studio.

[Interrupting Commands](https://foldkit.dev/example-apps/interrupting-commands)

Simulated file uploads driven by interruptible Commands. Cancel a single upload, cancel every upload in flight, or restart a cancelled one; each cancellation resolves through a keyed interrupt registry and an outcome-carrying result Message.

[View Transitions](https://foldkit.dev/example-apps/view-transitions)

Animated route changes with the View Transitions API. Direction-aware slides via transition types and a shared-element morph from gallery card to detail hero.

[Query Sync](https://foldkit.dev/example-apps/query-sync)

Filterable dinosaur table where every control syncs to URL query parameters. Schema transforms enforce valid states. Invalid params gracefully fall back.

[Snake](https://foldkit.dev/example-apps/snake)

The classic snake game. Keyboard input, game loop, and collision detection.

[Auth](https://foldkit.dev/example-apps/auth)

Authentication flow with Submodels, OutMessage, protected routes, and session management.

[Shopping Cart](https://foldkit.dev/example-apps/shopping-cart)

E-commerce app with product listing, cart management, and checkout flow.

[State Machine](https://foldkit.dev/example-apps/state-machine)

Checkout workflow powered by the experimental state machine module. Guards skip Shipping for digital orders, gate Place order behind a complete review, and parse promo codes into applied discounts.

[Pixel Art](https://foldkit.dev/example-apps/pixel-art)

Pixel art editor showcasing undo/redo with immutable snapshots, time-travel history, UI components (RadioGroup, Switch, Listbox, Dialog, Button), createLazy view optimization, Subscriptions, Commands with error handling, and localStorage persistence via Flags.

[Job Application](https://foldkit.dev/example-apps/job-application)

Multi-step form with async email validation, cross-field date constraints, file uploads, and per-step error indicators.

[WebSocket Chat](https://foldkit.dev/example-apps/websocket-chat)

Managed resources with WebSocket integration. Connection lifecycle, reconnection, and message streaming.

[Managed Resource Layer](https://foldkit.dev/example-apps/managed-resource-layer)

Layer-backed ManagedResource that starts a ComputeEngine service from an Effect Layer, exposes it to Commands, and runs Layer finalizers when the Model turns it off.

[Kanban](https://foldkit.dev/example-apps/kanban)

Drag-and-drop kanban board with cross-column reordering, keyboard navigation, fractional indexing, and screen reader announcements.

[Map](https://foldkit.dev/example-apps/map)

Interactive MapLibre GL map with locations, search, and "find my location". Demonstrates OnMount integration with a third-party DOM library, plus a Subscription bridging map move and marker click events back to the Model.

[Canvas Art](https://foldkit.dev/example-apps/canvas-art)

Click the canvas to spawn bouncing balls. Demonstrates declarative 2D rendering with Canvas.view, animation-frame Subscriptions, and pointer events translated to canvas-local coordinates.

[Generative Art](https://foldkit.dev/example-apps/generative-art)

Move the mouse to stir a Perlin-noise flow field, click to bloom prismatic particle bursts. Demonstrates Canvas.view with hundreds of evolving Path strokes per frame, Effect Random for spawning, and tunable simulation knobs wired through Messages.

[Web Components](https://foldkit.dev/example-apps/web-components)

QR code designer wiring two real third-party web components into Foldkit with CustomElement.define. A hex color picker from vanilla-colorful emits color-changed CustomEvents that flow back as Messages, and the sl-qr-code element from Shoelace accepts typed properties. The picker and the QR never touch each other directly; they share state through the Model.

[Embedding](https://foldkit.dev/example-apps/embedding)

A Foldkit widget embedded in a plain TypeScript host page through Runtime.embed. The host seeds initial state with Flags, pushes a step value in through an inbound Port, mirrors the count the widget emits through an outbound Port, and mounts and unmounts the widget with dispose. All communication crosses one Schema-typed handle; the host never touches the Model.

[UI Showcase](https://foldkit.dev/example-apps/ui-showcase)

Interactive showcase of every Foldkit UI component with styled examples, routing, and component state management.

[Personal Blog](https://foldkit.dev/example-apps/personal-blog)

Blog whose prose lives in markdown files. The @foldkit/markdown Vite plugin compiles each file into a typed document at build time, the app restyles the fold with per-node view overrides, and directive islands place a live Counter Submodel and a Note callout between paragraphs.

[Typing Terminal](https://foldkit.dev/example-apps/typing-terminal)

A production real-time multiplayer typing speed game. Full stack Effect app with RPC backend and Foldkit frontend.

[Race your friends →](https://typingterminal.com)
