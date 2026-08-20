---
url: https://foldkit.dev/testing
title: "Testing"
description: "Test Foldkit programs with Story and Scene. Story drives the update loop directly, while Scene drives the rendered view through accessible locators."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Testing

## Story and Scene

Foldkit tests at two boundaries. Story calls update directly. Scene enters through the rendered view. Neither test runs a browser or executes the Effects inside Commands, so both stay deterministic and fast.

Story

Scene

Enters through

A Message

An interaction or lifecycle result

Observes

Model changes, Commands, and OutMessages

Rendered output, Commands, Mounts, and OutMessages

Best suited to

Update logic, edge cases, and Command wiring

User flows, view behavior, and accessibility

Use both. Story proves the state machine behaves correctly. Scene proves that a person can reach that behavior through the view.

Name each file for the boundary it tests:

- `story.test.ts` drives update.
- `scene.test.ts` drives the rendered view.
- When one folder has several tests of the same kind, prefix the subject: `login.story.test.ts`.
- Keep root-level Scene tests for flows that cross pages. Colocate page and Submodel tests with the code they exercise.

The names stay accurate whether update and view live together or in separate files. See [Project Organization](https://foldkit.dev/patterns/project-organization) for the full layout.

## Story

`story` starts from a Model, sends Messages through update, and keeps Commands as data until the test supplies their result Messages. See the [Story](https://foldkit.dev/testing/story) page for the full API.

Story can test a root update or a child update in isolation. The update function is the contract at either level.

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

`scene` renders the view after every step. Locators find elements by role, label, placeholder, and visible text. Interactions invoke the view's event handlers, while cause-named steps supply Subscription, ManagedResource, and CustomElement results. Scene also tracks pending Commands and Mounts. See the [Scene](https://foldkit.dev/testing/scene) page for the full API.

Scene can also start at the root or at a child Submodel. `withViewInputs` adapts a Submodel view that needs ViewInputs, and `expectOutMessage` checks a child's OutMessage directly.

Choose the level by ownership. Test a Submodel's rendering, interactions, Commands, and OutMessages at the Submodel. Test parent folding, lifted Commands, route changes, and parent-computed ViewInputs at the root. Those behaviors cross the boundary and cannot be observed from the child.

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
