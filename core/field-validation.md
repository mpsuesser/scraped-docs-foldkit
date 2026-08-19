---
url: https://foldkit.dev/core/field-validation
title: "Field Validation"
description: "Per-field form validation in Foldkit using a four-state discriminated union. Built-in validation rules, Effect-TS powered, no impossible states."
access_date: 2026-08-19T19:38:38.072Z
current_date: 2026-08-19T19:38:38.072Z
---

# Field Validation

Foldkit models field validation as data in your Model, not scattered logic across event handlers. Each field is a four-state discriminated union: `NotValidated`, `Validating`, `Valid`, and `Invalid`. This makes it impossible to render a success indicator while an error exists, or show a spinner when validation is already complete.

## Defining a Field

`makeRules` takes an options object and returns a `Rules` bundle. `Field(valueSchema)` builds the four-state Schema you put in your Model.

```
import { Schema as S } from 'effect'
import { Field, Rule, makeRules } from 'foldkit/fieldValidation'

// Optional: no `required` option. The rule applies when the user fills it in.
const usernameRules = makeRules({
  rules: [Rule.minLength(3, 'Must be at least 3 characters')],
})

// Required: empty values become `Invalid` with the given message.
const emailRules = makeRules({
  required: 'Email is required',
  rules: [Rule.email('Please enter a valid email address')],
})

// Non-string fields work too. The value Schema is what the control holds,
// so a multi-select holds an array. Annotate the value type on `makeRules`.
const interestsRules = makeRules<ReadonlyArray<string>>({
  required: 'Pick at least one interest',
  rules: [Rule.maxItems(5, 'Choose up to five')],
})

const Model = S.Struct({
  username: Field(S.String),
  email: Field(S.String),
  interests: Field(S.Array(S.String)),
})
type Model = typeof Model.Type
```

Every state carries the current `value`. `Invalid` also carries a non-empty `errors` array.

Variant

Meaning

`NotValidated`

Validation has not run.

`Validating`

An async check is in flight.

`Valid`

Every applicable rule passed.

`Invalid`

One or more rules failed, with the errors.

The Schema you pass `Field` should match what the control actually holds as the user edits, not the type you parse it into: `Field(S.String)` for text inputs, `Field(S.Array(S.String))` for a multi-select. A scalar like a checkbox’s boolean usually stays plain `S.Boolean` in the Model; wrap it in `Field` only when it needs the validation lifecycle. Values you reach by parsing text, like numbers and dates, stay `Field(S.String)`: a half-typed entry is still a string, so parse it into its domain type on submit. Validation rules stay separate, in the `Rules` bundle.

Each entry in the `rules` array is a `Rule`: a `[predicate, errorMessage]` tuple. Error messages can be static strings or functions that receive the invalid value. Foldkit ships built-in rules for common cases; see [Custom Rules](#custom-rules) to write your own.

Operations are free module functions that take a `Rules` bundle as their first argument. `Rules` itself has no methods; the sections below introduce each operation.

To construct a state directly (e.g. initial Model values, async Command results), use the module-level constructors: `NotValidated`, `Validating`, `Valid`, `Invalid`.

### Conditional Rules

A `Rules` bundle is just data, so build it from model state via a plain function.

```
import { Rule, makeRules, validate } from 'foldkit/fieldValidation'

// A function that builds the bundle from whatever state it depends on.
const companyNameRules = (accountType: 'Personal' | 'Business') =>
  makeRules({
    ...(accountType === 'Business' && {
      required: 'Required for business accounts',
    }),
    rules: [Rule.maxLength(100)],
  })

const validateCompanyName = (
  accountType: 'Personal' | 'Business',
  value: string,
) => validate(companyNameRules(accountType))(value)
```

## Applying Validation

Call `validate(rules)(value)` to validate a value against a bundle of rules. It returns one of the four `Field` variants, failing fast at the first rule that fails. Use it in your update function with `evo` to set the field state.

```
import { Match as M } from 'effect'
import { validate } from 'foldkit/fieldValidation'
import { evo } from 'foldkit/struct'

const validateUsername = validate(usernameRules)

const update = (model: Model, message: Message) =>
  M.value(message).pipe(
    M.tagsExhaustive({
      ChangedUsername: ({ value }) => [
        evo(model, {
          username: () => validateUsername(value),
        }),
        [],
      ],
    }),
  )
```

Empty values follow the bundle’s requiredness before any rules run. An empty required value becomes `Invalid` with the required message; an empty optional value becomes `NotValidated`. A non-empty value becomes `Valid` when every rule passes or `Invalid` when one fails.

Use `validateAll(rules)` when you want to collect every failing rule into the `errors` array rather than stopping at the first failure. Requiredness behaves the same way in both functions.

## Displaying Validation State

Match exhaustively on the four tags to derive border colors, status indicators, and error messages. For a single-field submit gate, use `isValid(rules)(state)`. If the rules are required, only `Valid` passes; if they are optional, `NotValidated` also passes. `Validating` and `Invalid` never pass.

For a form-level gate, pass `[state, rules]` pairs to `allValid`. A single call gates fields of one value type, so a form that mixes types calls `allValid` per type and combines the results with `&&`.

```
import { Array, Match as M } from 'effect'
import { type Field, allValid } from 'foldkit/fieldValidation'
import type { HtmlBuilder } from 'foldkit/html'

const borderClass = (field: Field<string>) =>
  M.value(field).pipe(
    M.tagsExhaustive({
      NotValidated: () => 'border-gray-300',
      Validating: () => 'border-accent-300',
      Valid: () => 'border-accent-500',
      Invalid: () => 'border-red-500',
    }),
  )

const statusIndicator = (field: Field<string>, h: HtmlBuilder<Message>) =>
  M.value(field).pipe(
    M.tagsExhaustive({
      NotValidated: () => h.empty,
      Validating: () => h.span([], ['Checking...']),
      Valid: () => h.span([], ['✓']),
      Invalid: ({ errors }) => h.div([], [Array.headNonEmpty(errors)]),
    }),
  )

// `allValid` gates fields of one value type per call; required rules demand
// `Valid`, optional rules also accept `NotValidated`. For a form that mixes
// value types, call `allValid` per type and combine with `&&`.
const isFormValid = (model: Model): boolean =>
  allValid([
    [model.username, usernameRules],
    [model.email, emailRules],
  ])
```

Because `Field` is a discriminated union, the exhaustive match ensures you handle every state.

Use `isInvalid(state)` or `anyInvalid(states)` when you specifically need to know whether validation has produced errors. They check for the `Invalid` tag. A required `NotValidated` field and a `Validating` field are not invalid, but they still fail an `isValid` submit gate.

## Async Validation

For server-side checks like “Is this email taken?”, use the `Validating` state as a bridge: run sync `validate` first, then transition to `Validating`, fire a Command, and handle the result message.

```
import { Effect, Match as M, Number, Schema as S } from 'effect'
import { Command } from 'foldkit'
import { Invalid, Valid, Validating, validate } from 'foldkit/fieldValidation'
import { evo } from 'foldkit/struct'

const validateEmail = validate(emailRules)

const CheckEmailAvailable = Command.define('CheckEmailAvailable', {
  args: { email: S.String, validationId: S.Number },
  messages: [CompletedCheckEmailAvailable],
  execute: ({ email, validationId }) =>
    Effect.gen(function* () {
      const isAvailable = yield* apiCheckEmail(email)
      return CompletedCheckEmailAvailable({
        validationId,
        field: isAvailable
          ? Valid({ value: email })
          : Invalid({
              value: email,
              errors: ['This email is already taken'],
            }),
      })
    }).pipe(
      Effect.catch(() =>
        Effect.succeed(
          CompletedCheckEmailAvailable({
            validationId,
            field: Invalid({
              value: email,
              errors: ['Could not check this email. Try again.'],
            }),
          }),
        ),
      ),
    ),
})

const update = (model: Model, message: Message) =>
  M.value(message).pipe(
    M.tagsExhaustive({
      ChangedEmail: ({ value }) => {
        const syncResult = validateEmail(value)
        const validationId = Number.increment(model.emailValidationId)

        return M.value(syncResult).pipe(
          M.tag('Valid', () => [
            evo(model, {
              email: () => Validating({ value }),
              emailValidationId: () => validationId,
            }),
            [CheckEmailAvailable({ email: value, validationId })],
          ]),
          M.orElse(() => [
            evo(model, {
              email: () => syncResult,
              emailValidationId: () => validationId,
            }),
            [],
          ]),
        )
      },

      CompletedCheckEmailAvailable: ({ validationId, field }) => {
        if (validationId === model.emailValidationId) {
          return [evo(model, { email: () => field }), []]
        } else {
          return [model, []]
        }
      },
    }),
  )
```

The `validationId` pattern prevents race conditions. Each keystroke increments the ID, and the result handler only applies if the ID still matches. Responses from superseded requests are silently discarded.

## Custom Rules

A `Rule` is a `[predicate, errorMessage]` tuple. Write your own by pairing any predicate with an error message (a static string, or a function that receives the value).

```
import { Rule } from 'foldkit/fieldValidation'

const noConsecutiveSpaces: Rule.Rule<string> = [
  value => !/  /.test(value),
  'Cannot contain consecutive spaces',
]

const hasUppercase: Rule.Rule<string> = [
  value => /[A-Z]/.test(value),
  'Must contain at least one uppercase letter',
]

// Messages can be functions that receive the failing value:
const noTrailingWhitespace: Rule.Rule<string> = [
  value => value === value.trimEnd(),
  value => `Remove the trailing whitespace from "${value}"`,
]
```

Custom rules compose with built-in ones in the same `rules` array.

## Cross-Field Validation

A `Rule` only sees a single value. For checks that compare fields against each other (like “confirm password must match password”), handle the logic directly in your update function where you have access to the full model.

```
import { Match as M } from 'effect'
import {
  type Field,
  Invalid,
  Rule,
  makeRules,
  validate,
} from 'foldkit/fieldValidation'
import { evo } from 'foldkit/struct'

const passwordRules = makeRules({
  required: 'Password is required',
  rules: [Rule.minLength(8, 'Must be at least 8 characters')],
})

const validatePassword = validate(passwordRules)

const validateConfirmPassword = (
  password: string,
  confirmPassword: string,
): Field<string> => {
  const result = validatePassword(confirmPassword)
  if (result._tag === 'Valid' && result.value !== password) {
    return Invalid({
      value: confirmPassword,
      errors: ['Passwords must match'],
    })
  }
  return result
}

const update = (model: Model, message: Message) =>
  M.value(message).pipe(
    M.tagsExhaustive({
      ChangedPassword: ({ value }) => [
        evo(model, {
          password: () => validatePassword(value),
          confirmPassword: confirmPassword =>
            confirmPassword._tag === 'NotValidated'
              ? confirmPassword
              : validateConfirmPassword(value, confirmPassword.value),
        }),
        [],
      ],

      ChangedConfirmPassword: ({ value }) => [
        evo(model, {
          confirmPassword: () =>
            validateConfirmPassword(model.password.value, value),
        }),
        [],
      ],
    }),
  )
```

Keep cross-field logic in update only when the check genuinely needs more than one value. Anything expressible as `[predicate, errorMessage]` over a single value fits better as a [custom rule](#custom-rules).

## Built-in Rules

Requiredness is not a rule. It is a `makeRules` option: pass `required: message` to make the field required, or omit it for an optional field. By default, Foldkit treats an empty string or empty array as missing. Whitespace and every other value, including the boolean `false`, count as present.

Pass an `isEmpty` predicate to `makeRules` when your control needs a different definition of empty. To require that a checkbox is checked, use a custom rule such as `[(checked) => checked, message]`; unchecked is a present but invalid boolean value, not an absent one.

Rule

Description

`Rule.minLength(min, message?)`

Minimum character count

`Rule.maxLength(max, message?)`

Maximum character count

`Rule.pattern(regex, message?)`

Matches a regular expression

`Rule.email(message?)`

Valid email format

`Rule.url(options?)`

Valid URL format

`Rule.startsWith(prefix, message?)`

Begins with a prefix

`Rule.endsWith(suffix, message?)`

Ends with a suffix

`Rule.includes(substring, message?)`

Contains a substring

`Rule.equals(expected, message?)`

Exact string match

`Rule.oneOf(values, message?)`

Value is in a set of allowed strings

For array-valued fields like a multi-select, validate with `Rule.minItems(min, message?)` and `Rule.maxItems(max, message?)`. A required array field already treats the empty array as missing, so reach for these when you need a specific count.

### Rules from a Schema

When a value is already modeled by a Schema, a domain codec, or a refined or branded type, `Rule.fromSchema(schema, message)` turns it into a rule, so the field stays in sync with that Schema instead of duplicating its checks. It does nothing a custom rule can’t, so reach for it only when you already maintain the Schema; for plain checks the dedicated rules above are clearer.

Its sweet spot is values where “valid” means “decodes”. The Schema can transform a string into a different type, like a `Calendar.CalendarDateFromIsoString` codec that parses a date the string-shaped rules can’t check, or refine and brand it, like a `Slug`. Either way the field reuses the one Schema as its rule, so the check can’t drift from the type you already maintain:

```
import { Schema as S } from 'effect'
import { Calendar } from 'foldkit'
import { Field, Rule, makeRules } from 'foldkit/fieldValidation'

// A transform Schema: parses a string into a CalendarDate.
const EventDate = Calendar.CalendarDateFromIsoString

// A refinement Schema: brands a string that matches the pattern.
const Slug = S.String.check(S.isPattern(/^[a-z0-9-]+$/)).pipe(S.brand('Slug'))
type Slug = typeof Slug.Type

// Reuse each Schema as a rule, so the rule can't drift from the Schema.
const eventDateRules = makeRules({
  required: 'Event date is required',
  rules: [Rule.fromSchema(EventDate, 'Enter a real date as YYYY-MM-DD')],
})

const slugRules = makeRules({
  required: 'Slug is required',
  rules: [Rule.fromSchema(Slug, 'Use lowercase letters, numbers, and hyphens')],
})

// Each Field wraps S.String, the raw value the control holds.
const Model = S.Struct({
  eventDate: Field(S.String),
  slug: Field(S.String),
})
type Model = typeof Model.Type
```

See the full [API reference](https://foldkit.dev/api-reference/field-validation) for details on every export. For a complete working example with sync validation, async server checks, and form submission gating, see the [Form example](https://foldkit.dev/example-apps/form). For sync-only validation with OutMessage context, see the [Auth example](https://github.com/foldkit/foldkit/tree/main/examples/auth/src/page/loggedOut/page/login.ts).
