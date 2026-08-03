---
url: https://foldkit.dev/testing
title: "Testing"
description: "Test Foldkit programs with Story and Scene. Story simulates the update loop. Scene tests features through the rendered view with accessible locators."
access_date: 2026-08-03T18:23:29.850Z
current_date: 2026-08-03T18:23:29.850Z
---

# Testing

## Overview

The Elm Architecture makes testing straightforward. The update function is pure. Given a Model and a Message, it always returns the same result. No DOM, no HTTP calls, no timers. Just a function that takes data and returns data.

Foldkit ships two testing primitives. `Story` tests the state machine: you send Messages directly through update, resolve Commands inline, and assert on the Model. `Scene` tests features through the rendered view (for example clicking buttons, typing into inputs, or pressing keys) using accessible locators. Both are pure, deterministic, and fast.

Use Story for update logic, edge cases, and Command wiring. Use Scene for user flows, view rendering, and accessibility. A well-tested Foldkit app uses both.

Name test files for their test style, beside the code under test: `story.test.ts` for Story tests (which drive `update`) and `scene.test.ts` for Scene tests (which drive the rendered view). The name describes how the test works, not a source file, so it stays correct whether `update` and `view` live in `main.ts` or in their own files. When one folder holds more than one test of a kind (sibling pages, component variants), prefix with the subject, like `login.story.test.ts`. Both styles colocate at any level, including inside [Submodel](https://foldkit.dev/core/submodel) folders. A root-level `scene.test.ts` then holds only the flows that span pages, and a large suite splits those by flow into `checkout.scene.test.ts`, `cart.scene.test.ts`, and so on. See [Project Organization](https://foldkit.dev/project-organization) for the full layout.

## Story

`story` simulates the update loop. Each step reads like a sentence: send a Message, resolve a Command, check the Model. See the [Story](https://foldkit.dev/testing/story) page for the full API.

Story tests are flexible about testing level. Because Story sends Messages directly to `update` and asserts on the Model, testing a child’s update in isolation is valid: the function signature is the contract, and it works the same whether the parent calls it or the test does.

```
import { Command, given, message, model, story } from 'foldkit/story'
import { expect, test } from 'vitest'

test('delayed reset: count resets after the delay fires', () => {
  story(
    update,
    given({ count: 5 }),
    message(ClickedResetAfterDelay()),
    Command.expectExact(DelayReset),
    Command.resolve(DelayReset, CompletedDelayReset()),
    model(model => {
      expect(model.count).toBe(0)
    }),
  )
})
```

## Scene

`scene` exercises the view. Locators find elements the way users do: by role, label, or placeholder. Interactions dispatch Messages through the rendered event handlers, and Messages from the other lifecycle causes enter through their own cause-named steps: `Subscription.emit`, `ManagedResource.acquire` / `release` / `failAcquire`, and `CustomElement.emit`. Inline assertions check the HTML between steps. Scene also tracks the Mount lifecycle: the side effects declared by `OnMount` attributes in the view must be acknowledged via `Mount.resolve`, mirroring how Commands are resolved, and a Mount that later unmounts is acknowledged with `Mount.expectEnded`. See the [Scene](https://foldkit.dev/testing/scene) page for the full API.

Scene runs at any level. `Submodel.defineView` produces a plain `(model, h) => Html` function, so a child view drops into `scene` unmodified. The `toParentMessage` wrap lives at the parent’s `h.submodel` call site, not in the child’s signature. A Submodel that declares `ViewInputs` takes a second argument, which `withViewInputs` supplies: pass the view and its default inputs once, then vary value inputs per test while the renderer stays pinned; `packages/ui/src/slider/scene.test.ts` is the canonical example. `scene` also accepts a child `update` that returns the three-tuple with an OutMessage, and `expectOutMessage` and `expectNoOutMessage` assert on it directly.

Pick the level by coupling. Enter at the [Submodel](https://foldkit.dev/core/submodel) for behavior the Submodel owns: how it renders, how it responds to interaction, which Commands it returns. Enter at the root for behavior that crosses the boundary, which covers how the parent folds an OutMessage, a Command the parent lifts, a route change, and view inputs the parent computes. Those are invisible from below, so asserting on them in a child Scene would test a path production never runs. (The OutMessage itself is assertable at the child level with `expectOutMessage`; what the parent does with it is not.) The reverse is just as real: driving the whole app through the root to check one page’s rendering buries the assertion in unrelated setup.

```
import {
  Command,
  click,
  expect,
  given,
  inside,
  label,
  role,
  scene,
  text,
  type,
} from 'foldkit/scene'
import { test } from 'vitest'

test('type a zip code, click get weather, see the forecast', () => {
  scene(
    { update, view },
    given(model),

    type(label('Zip code'), '90210'),
    click(role('button', { name: 'Get Weather' })),
    expect(role('button', { name: 'Loading...' })).toExist(),

    // Instance form: locks in the zipCode the runtime captured.
    Command.expectExact(FetchWeather({ zipCode: '90210' })),
    Command.resolve(
      FetchWeather,
      SucceededFetchWeather({ weather: beverlyHillsWeather }),
    ),
    inside(
      role('article'),
      expect(text('Beverly Hills, California')).toExist(),
      expect(text('72\u00B0F')).toExist(),
      expect(text('Clear sky')).toExist(),
    ),
  )
})
```
