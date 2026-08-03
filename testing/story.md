---
url: https://foldkit.dev/testing/story
title: "Story"
description: "Test the state machine with Story. Send Messages, resolve Commands inline, and assert on the Model. Pure, deterministic, fast."
access_date: 2026-08-03T18:13:14.939Z
current_date: 2026-08-03T18:13:14.939Z
---

# Story

## Testing the Update Loop

The Elm Architecture makes testing straightforward. The update function is pure. Given a Model and a Message, it always returns the same result. No DOM, no HTTP calls, no timers. Just a function that takes data and returns data.

`Story` tests the state machine. You send Messages through update, resolve Commands inline, and assert on the Model. The entire test is one `story` call. No mocking libraries, no fake timers, no setup or teardown.

## The API

Import the steps you need from `foldkit/story`. The top-level steps are `story`, `given`, `message`, `model`, `expectOutMessage`, and `expectNoOutMessage`. Command resolution and assertions live under the `Command` namespace: `Command.resolve`, `Command.resolveAll`, `Command.expectHas`, `Command.expectExact`, and `Command.expectNone`.

A test file usually needs only one of the two testing modules, so named imports keep the call sites short. When a single file tests both a story and a scene, import the namespaces instead (`import { Scene, Story } from 'foldkit'`) so `Story.given` and `Scene.given` stay distinguishable.

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

Command matchers (`expectHas`, `expectExact`, and `resolve`) accept either a Command Definition (matches by name) or a Command instance (matches by name AND structural-equal args). Pass a Definition when the test only cares that the Command was dispatched. Pass an instance like `FetchWeather({ zipCode: '90210' })` when the args are part of what the test is verifying. Strict matching catches regressions where a Command fires with wrong inputs, which a name-only match would silently pass.

Mount lifecycle is a Scene concern

Story does not render the view, so the OnMount lifecycle is not observable from a Story test. Tests that need to acknowledge mounts use Scene's `Mount.resolve` and the related steps; see the [Scene](https://foldkit.dev/testing/scene) page.

## Your First Test

Here’s a test for the delayed reset from the [Commands](https://foldkit.dev/core/commands) page. When the user clicks reset, a one-second delay fires, then the count resets to zero:

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

The test reads as a story. Start from a Model with count 5. Send `ClickedResetAfterDelay()`. Verify that update returned a `DelayReset` Command. Resolve it with `CompletedDelayReset()`. Verify the count is 0. Every step is visible. The simulation called update, resolved the Command with the Message you provided, fed that Message back through update, and arrived at the final state.

## Multi-Step Flows

Real apps have multi-step user stories. `Command.resolve` and `Command.resolveAll` let you resolve Commands inline at any point in the story. This keeps the resolution next to the step that produced the Command, so the test reads chronologically:

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

Every `message` is a user action: “the user submitted the form.” Every `Command.resolve` or `Command.resolveAll` is world-building: “the weather API succeeded.” Every `model` is a scene check: “the weather is showing.”

Resolvers are a queue

Each entry in `resolveAll` resolves exactly one matching dispatch in declaration order. `[FetchCount, m1], [FetchCount, m2], [FetchCount, m3]` reads as three responses to three dispatches. For N identical responses, compose with `Array.makeBy(n, () => [Def, message])`. Resolvers carry across calls: unused entries can match later dispatches, and a new entry replaces any leftover resolvers sharing its Definition or Instance shape (latest wins).

Unresolved Commands

`message` throws if there are pending Commands from a previous step. Resolve all Commands before sending the next Message. `story` throws at the end if any Commands remain unresolved. Every Command your update function produces must be accounted for.

## Testing Side Effects

The simulation tests the state machine. Messages go in, Model changes come out, Commands are resolved declaratively. It does not run the actual Effects inside Commands.

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

Two levels, clean separation. The simulation proves the state machine wires correctly. `Effect.provide` proves the side effect works. If the state machine sends the right Command, and the Command does the right thing, the program works.
