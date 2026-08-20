---
url: https://foldkit.dev/core/preserve-scroll
title: "Preserve Scroll"
description: "Restore window scroll position across Vite HMR reloads. Covers when restoration runs and its window-only scope."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Preserve Scroll

## Development Reloads

When Vite reloads the page after a file change, the browser normally returns to the top. That makes editing content far down a page frustrating.

Foldkit saves `window.scrollX` and `window.scrollY` just before the reload. After the restored view renders, it scrolls the window back to that position.

Scroll preservation runs only when Vite HMR is active. Set `preserveScroll` to `false` when the application implements its own development scroll restoration.

## Window Scroll Only

Only the window scroll offset is preserved. Scroll positions of nested `overflow` containers are not.

Restoration applies to document-owning apps built with `makeApplication`. An embedded `makeElement` app does not control the host page's scroll, so Foldkit disables the feature for it.

Foldkit restores the offset as soon as the first view renders. Later layout shifts can still move the final position. For example: images without dimensions may load after restoration and push content farther down the page.
