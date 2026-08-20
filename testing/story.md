---
url: https://foldkit.dev/testing/story
title: "Story"
description: "Drive update with Messages, inspect the Model and Commands, and supply Command results without executing their Effects."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Story

## Testing the Update Loop

`Story` calls update with the Model and Messages you provide. Commands stay as data until the test supplies their result Messages. You can test a complete state transition without a DOM, a timer, or a running Effect.

The entire test is one `story` call. Start from a Model, send a Message, resolve any Commands, and assert on the next Model.

## The API

Import the steps you need from `foldkit/story`.

- Top-level steps start the test, send Messages, and inspect results: `story`, `given`, `message`, `model`, `expectOutMessage`, and `expectNoOutMessage`.
- The `Command` namespace handles pending Commands: `resolve`, `resolveAll`, `resolveAllExact`, `expectHas`, `expectExact`, and `expectNone`.

Use named imports when the file contains only Story tests. If a file contains both Story and Scene tests, import the namespaces from `foldkit` so `Story.given` and `Scene.given` stay distinct.

```
import { Array } from 'effect'
import {
  Command,
  expectOutMessage,
  given,
  message,
  model,
  story,
} from 'foldkit/story'

// Set the initial Model.
given(model)

// Send a Message. Commands stay pending.
message(ClickedSubmit())

// Resolve one Command with its result. Pass a Definition to match by name,
// or a Command instance to match by name AND args.
Command.resolve(FetchWeather, SucceededFetchWeather({ data }))
Command.resolve(
  FetchWeather({ zipCode: '90210' }),
  SucceededFetchWeather({ data }),
)

// Resolve many Commands at once. Each entry resolves exactly one matching
// dispatch in declaration order.
Command.resolveAll(
  [FocusInput, CompletedFocusInput()],
  [ScrollToTop, CompletedScrollToTop()],
)

// Resolve and assert a batch. Every listed resolver must match in this call,
// and no actual Command may remain unresolved.
message(ClickedSubmit())
Command.resolveAllExact(
  [FocusInput, CompletedFocusInput()],
  [ScrollToTop, CompletedScrollToTop()],
)

// For N identical responses, compose with Array.makeBy.
Command.resolveAll(
  ...Array.makeBy(3, () => [AnimationTick, CompletedTick()] as const),
)

// Assert on the Model.
model(model => {
  expect(model.count).toBe(0)
})

// Assert these Commands were produced. Definition matchers match by name only;
// instance matchers (FetchWeather({ zipCode: '90210' })) match by name AND args.
Command.expectHas(FetchWeather)
Command.expectHas(FetchWeather({ zipCode: '90210' }))

// Assert exactly these Commands were produced (mix Definition and instance).
Command.expectExact(FetchWeather, SaveBoard)
Command.expectExact(FetchWeather({ zipCode: '90210' }), SaveBoard)

// Assert no Commands were produced.
Command.expectNone()

// Assert on the OutMessage.
expectOutMessage(SucceededLogin({ session }))

// Run the test story. Throws on unresolved Commands.
story(
  update,
  given(model),
  message(ClickedSubmit()),
  Command.expectHas(FetchData),
  Command.resolve(FetchData, SucceededFetch({ data })),
  model(model => {
    expect(model.status).toBe('loaded')
  }),
)
```

A Command matcher accepts either a Definition or an instance. A Definition matches by name. An instance also checks structurally equal arguments.

Use `FetchWeather` when the test only cares that the Command was dispatched. Use `FetchWeather({ zipCode: '90210' })` when the zip code is part of the contract.

Mount lifecycle is a Scene concern

Story does not render the view, so the OnMount lifecycle is not observable from a Story test. Tests that need to acknowledge mounts use Scene's `Mount.resolve` and the related steps; see the [Scene](https://foldkit.dev/testing/scene) page.

## Your First Test

Here’s a test for the delayed reset from the [Commands](https://foldkit.dev/core/commands) page. Clicking reset starts a one-second delay. When the delay completes, the count returns to zero.

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

Read it from top to bottom. The Model starts at 5. `ClickedResetAfterDelay()` returns `DelayReset`. The test resolves that Command with `CompletedDelayReset()`, which update handles by setting the count to 0.

## Multi-Step Flows

Keep each Command result next to the Message that caused it. `Command.resolve`, `Command.resolveAll`, and `Command.resolveAllExact` let a longer test stay chronological:

```
import { Command, given, message, model, story } from 'foldkit/story'
import { expect, test } from 'vitest'

test('weather search: success then failure', () => {
  story(
    update,
    given(model),

    message(UpdatedZipCodeInput({ value: '90210' })),
    model(model => {
      expect(model.zipCode).toBe('90210')
    }),
    message(SubmittedWeatherForm()),
    // Instance form: locks in the zipCode the runtime captured.
    Command.expectHas(FetchWeather({ zipCode: '90210' })),
    Command.resolve(
      FetchWeather,
      SucceededFetchWeather({ weather: beverlyHillsWeather }),
    ),
    model(model => {
      expect(model.weather._tag).toBe('WeatherSuccess')
      expect(model.weather.data.temperature).toBe(72)
    }),

    message(UpdatedZipCodeInput({ value: '00000' })),
    model(model => {
      expect(model.zipCode).toBe('00000')
    }),
    message(SubmittedWeatherForm()),
    Command.expectHas(FetchWeather({ zipCode: '00000' })),
    Command.resolve(FetchWeather, FailedFetchWeather({ error: 'Not found' })),
    model(model => {
      expect(model.weather._tag).toBe('WeatherFailure')
    }),
  )
})
```

The test does not run an HTTP request. It declares that `FetchWeather` succeeded, feeds the resulting Message through update, and checks the Model that the view will render.

Resolvers are a queue

Each `resolveAll` entry resolves one matching dispatch in declaration order. Three `[FetchCount, message]` entries resolve three `FetchCount` dispatches. For N identical responses, use `Array.makeBy(n, () => [Definition, message])`.

Unused resolvers can match later dispatches. A new resolver replaces any leftover resolver with the same Definition or instance shape.

Exact resolution

Use `resolveAllExact` when the resolver list is also a claim about which Commands were dispatched. It walks cascades like `resolveAll`, but every listed entry must match one dispatch within that call and no actual Command may remain unresolved. Repeated entries sharing a Definition consume repeated dispatches in declaration order. Supplied resolvers never carry forward.

Unresolved Commands

`message` throws if there are pending Commands from a previous step. Resolve all Commands before sending the next Message. `story` throws at the end if any Commands remain unresolved. Every Command your update function produces must be accounted for.

## Testing Side Effects

Story tests the state machine. It does not run the Effect inside a Command.

To test that a Command’s Effect works correctly (for example, that an HTTP request parses the response right), test it separately with `Effect.provide` and a mock service layer:

```
import { Effect, Layer, Match as M, String } from 'effect'
import { HttpClient, HttpClientResponse } from 'effect/unstable/http'
import { expect, test } from 'vitest'

test('fetchWeather returns SucceededFetchWeather on success', async () => {
  const mockClient = HttpClient.make(request =>
    Effect.sync(() => {
      const responseData = M.value(request.url).pipe(
        M.when(String.includes('geocoding'), () => ({
          results: [
            {
              name: 'Beverly Hills',
              latitude: 34.07362,
              longitude: -118.40036,
              admin1: 'California',
            },
          ],
        })),
        M.when(String.includes('forecast'), () => ({
          current: {
            time: '2026-03-10T01:30',
            interval: 900,
            temperature_2m: 72.4,
            relative_humidity_2m: 45,
            wind_speed_10m: 9.8,
            weather_code: 0,
          },
        })),
        M.orElse(url => {
          throw new Error(`Unexpected request URL: ${url}`)
        }),
      )
      return HttpClientResponse.fromWeb(
        request,
        new Response(JSON.stringify(responseData), { status: 200 }),
      )
    }),
  )

  const message = await fetchWeather('90210').effect.pipe(
    Effect.provide(Layer.succeed(HttpClient.HttpClient, mockClient)),
    Effect.runPromise,
  )

  expect(message._tag).toBe('SucceededFetchWeather')
})
```

The two tests cover different contracts. Story checks which Command update returns and what update does with its result Message. The Effect test checks the work the Command executes.
