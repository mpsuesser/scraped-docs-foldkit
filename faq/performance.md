---
url: https://foldkit.dev/faq/performance
title: "Performance"
description: "Foldkit’s rendering cost model, TodoMVC benchmark results, development-mode overhead, and the tools for measuring and memoizing expensive views."
access_date: 2026-11-17T22:33:54.784Z
current_date: 2026-11-17T22:33:54.784Z
---

# Performance

## Overview

This page compares Foldkit with peer frameworks in a keyed table workload, measured on October 30, 2026. The Foldkit results use [commit `a664f983`](https://github.com/foldkit/foldkit/commit/a664f9834f0398807d748071cc402a991861a870), an unreleased build based on 0.170.0.

## Benchmarks

### Workload

Foldkit uses [js-framework-benchmark](https://github.com/krausest/js-framework-benchmark) for its public rendering comparison. The suite creates, updates, selects, swaps, removes, and clears table rows. Every entry in this comparison is keyed: a row's DOM identity follows its id when the rows move.

The [measured Foldkit implementation](https://github.com/foldkit/foldkit/blob/6991555d65d9fd6e5199acbf3693a31b05f72898/internal/js-framework-benchmark/src/main.ts) uses keyed rows with per-row view memoization. Framework entries use production builds.

### Operation timings

Values are median milliseconds; lower is faster. Vanilla is the upstream `vanillajs-lite` direct DOM implementation, and React is `react-hooks`. Partial update changes every tenth row of 1,000. Append adds 1,000 rows to an existing 1,000.

| Operation (ms) | Vanilla reference | Solid 1.9.3 | React 19.2.0 | Svelte 5.42.1 | Vue 3.5.39 | Elm 0.19.1 | Foldkit unreleased |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Create 1,000 rows | 28.0 | 30.0 | 36.4 | 30.8 | 33.7 | 36.7 | 48.1 |
| Replace 1,000 rows | 29.9 | 34.2 | 42.8 | 35.3 | 37.2 | 40.2 | 49.0 |
| Update every tenth row | 16.8 | 17.5 | 23.7 | 18.2 | 20.1 | 22.3 | 42.9 |
| Select a row | 3.9 | 5.3 | 8.9 | 8.2 | 6.1 | 13.4 | 9.1 |
| Swap two rows | 21.0 | 22.0 | 138.4 | 22.1 | 22.7 | 41.5 | 29.1 |
| Remove one row | 20.4 | 18.8 | 18.4 | 23.5 | 20.1 | 19.8 | 16.9 |
| Create 10,000 rows | 301.2 | 344.2 | 520.4 | 348.3 | 362.8 | 676.8 | 480.9 |
| Append 1,000 rows | 32.6 | 35.1 | 40.5 | 34.9 | 37.4 | 46.6 | 53.9 |
| Clear 1,000 rows | 12.5 | 14.7 | 24.0 | 14.8 | 18.6 | 18.4 | 34.0 |

Among these seven entries, Foldkit had the slowest median in five of nine operations and the lowest median for removing one row. Selection was close to React, at 9.1 ms versus 8.9 ms. For creating 10,000 rows and swapping, Foldkit's median was lower than React and Elm, and higher than Solid, Svelte, and Vue.

These table operations measure rendering workloads. They do not establish how a complete application performs, or isolate the cost of Foldkit's architecture.

### Memory

Values are median MiB across five samples, measured after garbage collection with the browser's memory measurement API. Memory tests ran separately in seven isolated headless Chrome instances, one per entry, under normal system memory pressure. Each create/clear sample performs five cycles of creating and clearing 1,000 rows.

| Measurement (MiB) | Vanilla reference | Solid 1.9.3 | React 19.2.0 | Svelte 5.42.1 | Vue 3.5.39 | Elm 0.19.1 | Foldkit unreleased |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Ready | 0.56 | 0.60 | 1.19 | 0.67 | 0.87 | 0.69 | 2.26 |
| After creating 1,000 rows | 1.70 | 2.68 | 4.43 | 2.88 | 3.92 | 3.65 | 6.21 |
| After five create/clear cycles | 0.67 | 0.77 | 1.98 | 0.99 | 1.20 | 1.06 | 2.86 |

Foldkit had the highest median memory use in all three measurements. After creating 1,000 rows, it used 6.21 MiB, compared with 2.68 MiB for Solid and 4.43 MiB for React.

### Measurement

The timings come from Chrome traces and include scripting and rendering. The harness applies 4× CPU throttling to partial update, selection, swapping, and clearing, and 2× to removal. The other operations run without throttling. See the [operation definitions at the tested revision](https://github.com/krausest/js-framework-benchmark/blob/ca8715d9ae84916fd24de3a79457d8c328f9b766/webdriver-ts/src/benchmarksCommon.ts).

The timing batch used harness commit `ca8715d9`, Chrome for Testing 156.0.8078.75 in a visible window, and an Apple M1 Max with 64 GiB of memory running macOS 26.6.2 on AC power. Each operation has 15 samples, except selection, which the upstream runner samples 25 times. Each sample starts from a fresh page and follows the harness's warmup procedure. Frameworks run sequentially within each operation.

These are local measurements, separate from the [official upstream results](https://krausest.github.io/js-framework-benchmark/index.html). The [raw samples and run metadata](https://github.com/foldkit/foldkit/tree/6991555d65d9fd6e5199acbf3693a31b05f72898/internal/js-framework-benchmark/results/2026-10-30) record the entry versions, build details, and commands.

## Where the time goes

Every state change in a Foldkit app flows through one pipeline, so the cost model fits in a few sentences:

### Dispatch and Rendering

- A dispatched Message runs `update` synchronously, right on the dispatching stack, in arrival order. The runtime compares the returned Model to the previous one by reference. If nothing changed, nothing renders.
- Renders coalesce. Any number of Messages arriving between frames mark at most one pending frame; the next animation frame renders once with the latest Model. Animation-frame Subscription ticks can advance that render into the frame that delivers them. A burst of two hundred Messages inside one frame costs two hundred update calls and a single render.
- A render is `view` plus diff plus patch: build the new virtual tree, diff it against the previous one, and touch only the DOM that changed.
- Command results always arrive asynchronously. A synchronous burst that exhausts its time budget defers its remaining Messages to a new task, yielding to the browser.

Rendering can refuse an eligible commit three times while Messages are deferred. Animation frames and asynchronous ViewTransition update callbacks share that allowance. The next blocked opportunity processes the finite prefix of Messages already queued and commits plainly, skipping a selected or pending transition if necessary. This incorporates queued controlled-input changes before restoring input values. Messages enqueued during catch-up form a separate suffix and wait for the posted task. A large prefix can occupy a long task; the refusal bound supplies no wall-clock or browser-paint deadline.

### Model Size and Change Detection

By default, the production hot path does no work proportional to Model size. No serialization, no deep comparison, no freezing, no snapshots. Change detection is reference equality, and `modifyFields` preserves unchanged branches by reference, so the cost of a Message is the cost of your `update` logic, and the cost of a frame is the cost of the subtrees that actually changed.

## What the abstractions cost

The cost model above puts the cost of a Message on your `update` logic. What the update-side abstractions add on top of that is small, and worth spelling out, because they are the first thing people suspect when an app feels slow:

### Child Folds and Model Updates

- `Update.foldChild` adds a function call and an arity check over the wiring you would otherwise write by hand. Inside, it does exactly what the hand-written handler does: read the child out of the Model, run its update, write it back, and lift the child’s Commands through `toParentMessage`.
- `modifyFields` is Effect’s `Struct.evolve` at runtime; the wrapper adds key checking at the type level only. It walks the struct’s own enumerable keys once and applies a transform only where you provided one, copying everything else by reference. The cost is proportional to the Model’s top-level field count, the same as an object spread, and that reference preservation is what lets [createLazy](https://foldkit.dev/core/view-memoization) hit its `===` check.

### Exhaustive Message Matching

`Message.match` is the one whose cost scales with your app. It dispatches directly on `_tag`, but the handler object and one closure per arm are still allocated on every dispatch. Because the arms close over `model`, they cannot be hoisted to module scope.

Keep `Message.match` anyway. Its signature requires a handler for every tag in the union, so adding a Message variant fails to compile in every update that does not handle it. It also avoids the matcher pipeline that Effect `Match` builds for more general pattern matching. An `if`/`else` chain on `_tag` gives up exhaustiveness for less allocation. If a profile ever puts a genuinely hot update at the top, that trade is available, but it is the last change to reach for rather than the first.

## Development mode is not production

### Development Diagnostics

The default development configuration adds several systems:

- [Freeze Model](https://foldkit.dev/core/freeze-model) deep-freezes the Model after every update to catch accidental mutation at the write site.
- [DevTools](https://foldkit.dev/core/devtools) records each Message with Model snapshots and diffs for time travel.
- The [Slow Warnings](https://foldkit.dev/core/slow-warnings) can time update, subscriptions, view, and patch work against phase budgets.
- Model preservation encodes the Model so state survives development reloads.

### Measure a Production Build

Model freezing and preservation run only under the dev server. DevTools and Slow Warnings are disabled in production by default, but `show: 'Always'` enables them there. Measure a production build with the diagnostic settings you intend to deploy. An animation-heavy app dispatching Messages at 60Hz can pay extra development costs on every update. If DevTools is enabled, use `excludeFromHistory` to skip history recording for high-frequency Messages like frame ticks and pointer moves.

## The optimization toolkit

When something is slow, work through this list in order:

- Let the [Slow Warnings](https://foldkit.dev/core/slow-warnings) tell you which synchronous phase is actually slow before optimizing anything.
- Memoize expensive subtrees with [createLazy and createKeyedLazy](https://foldkit.dev/core/view-memoization). The keyed cache releases entries absent from a completed render.
- [Key](https://foldkit.dev/best-practices/keying) mapped list items by stable Model ids, the one identity only your data can provide, so the differ moves nodes instead of rebuilding them. Branching views need no keys when built with `@foldkit/vite-plugin`: view functions carry identity, and the differ replaces DOM when a position’s identity changes. Without the plugin, branch identity falls back to positional-plus-key semantics and each branch point needs an explicit key.
- Cache expensive derived data on the Model when memoization cannot cover it. The view recomputes a derived value on every render whether or not its inputs changed; update can compute it once, in the branches that change those inputs. The price is a derived field every such branch must keep in sync, so reach for this after `createLazy`, not before.
- Render long lists with [Virtual List](https://foldkit.dev/ui/virtual-list) so only visible items mount.

## Bundle size and code splitting

### Benchmark Bundle

The package is ESM-only, marked side-effect-free, and exposed through subpath exports, so bundlers tree-shake everything an app does not import. The table app measured with Foldkit `a664f983` contained about 306 kB of JavaScript, or 100 kB gzipped. That includes the table app, the Foldkit runtime, its vendored differ, and Effect. The application also uses Effect for Commands, Schemas, and data manipulation. The exact byte counts and build hash are in the run metadata.

### What Splits Today

What splits today:

- Heavy dependencies behind dynamic `import()`. This site loads the Monaco editor that way, only on the playground.
- Independent apps on one page via [embedding](https://foldkit.dev/core/embedding), each with its own bundle and lifecycle.

### Route Splitting

What does not split today: the routes of a single app. A Foldkit program is one statically composed Model, update, and view, so the code for every page ships in the initial bundle. Elm shares this property for the same reason. Splitting a program by route would take design work against the single-Model architecture, not a configuration flag.

For first-paint and SEO concerns, see [Server Rendering](https://foldkit.dev/core/server-rendering). Rendering a route on the server or at build time puts real content in the response before the bundle loads.
