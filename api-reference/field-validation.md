---
url: https://foldkit.dev/api-reference/field-validation
title: "FieldValidation"
description: "API documentation for the FieldValidation module."
access_date: 2026-08-07T15:31:08.863Z
current_date: 2026-08-07T15:31:08.863Z
---

# FieldValidation

## Functions

### allValid

function

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L217)

```
/**
 * Returns true when every field is acceptable per its rules, by `isValid`.
 *  Each pair is a field's `[state, rules]`. The pairs in one call share a
 *  value type, so a call gates fields of a single type; for a form that mixes
 *  types, call `allValid` once per type and combine the results with `&&`.
 *  Use for form-level submit gates.
 */
<A>(pairs: readonly Array<readonly [
  Field<A>,
  Readonly<{
    isEmpty: (value: A) => boolean
    requiredMessage: Option<RuleMessage<A>>
    rules: readonly Array<Rule<A>>
  }>
]>): boolean
```

### anyInvalid

function

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L223)

```
/**
 * Returns true when any state in the input has tag `Invalid`. Use for
 *  "this step/section has errors" affordances, independent of rules.
 */
(states: readonly Array<Field<unknown>>): boolean
```

### isInvalid

function

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L209)

```
/**
 * Returns true when the state's tag is `Invalid`. Tag-only predicate;
 *  unlike `!isValid(rules)(state)`, this does not treat `NotValidated`
 *  or `Validating` as errors. Use for "has the user seen a rule failure
 *  on this field?" affordances like red borders or per-step error
 *  indicators.
 */
(state: Field<unknown>): boolean
```

### isRequired

function

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L201)

```
/**
 * Returns true when the rules mark the field as required. Useful for
 *  rendering affordances like a `*` next to required field labels.
 */
<A>(rules: Rules<A>): boolean
```

### isValid

function

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L187)

```
/**
 * Returns true when the field's current state is acceptable given its
 *  rules. For required fields, only `Valid` returns `true`. For optional
 *  fields, `Valid` or `NotValidated` both return `true`. `Invalid` and
 *  `Validating` always return `false`.
 * 
 *  The name is distinct from the `Valid` tag on purpose: `isValid`
 *  answers "is this state an acceptable result?", which for an optional
 *  field is broader than `_tag === 'Valid'`. For pattern-matching on the
 *  state itself, check the `_tag` directly.
 */
<A>(rules: Rules<A>): (state: Field<A>) => boolean
```

### makeRules

function

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L115)

```
/**
 * Creates a `Rules` bundle from options. The value type defaults to `string`;
 *  for other field values, annotate it: `makeRules<ReadonlyArray<Tag>>({ ... })`.
 */
<A = string>(options: MakeRulesOptions<A>): Rules<A>
```

### validate

function

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L131)

```
/**
 * Validates a new value and returns the next field state.
 * 
 *  For required fields, an empty value produces `Invalid` with the
 *  required message. For optional fields, an empty value produces
 *  `NotValidated`. Non-empty values run through the field's rules;
 *  the first failure becomes `Invalid`, otherwise the result is `Valid`.
 */
<A>(rules: Rules<A>): (value: A) => Field<A>
```

### validateAll

function

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L154)

```
/**
 * Like `validate` but collects every failing rule into the
 *  `Invalid` state's errors array instead of stopping at the first.
 */
<A>(rules: Rules<A>): (value: A) => Field<A>
```

## Types

### Field

type

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L24)

```
/** The four-state union that represents a field's value in the Model. */
type Field = NotValidated<A> | Validating<A> | Valid<A> | Invalid<A>
```

### Invalid

type

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L17)

```
/** The `Invalid` state: one or more rules failed. Carries a non-empty `errors` array. */
type Invalid = Readonly<{
  _tag: "Invalid"
  errors: Array.NonEmptyReadonlyArray<string>
  value: A
}>
```

### MakeRulesOptions

type

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L90)

```
/** Options accepted by `makeRules`. */
type MakeRulesOptions = Readonly<{
  isEmpty: (value: A) => boolean
  required: RuleMessage<A>
  rules: ReadonlyArray<Rule<A>>
}>
```

### NotValidated

type

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L8)

```
/** The `NotValidated` state: user hasn't interacted yet. */
type NotValidated = Readonly<{
  _tag: "NotValidated"
  value: A
}>
```

### Rules

type

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L83)

```
/**
 * A field's validation rules: the required message (if any), the list of rules,
 *  and an empty predicate. Produced by `makeRules`; consumed by the module's
 *  operations (`validate`, `validateAll`, `isValid`, `isRequired`). The fields
 *  are accessible but treating them as stable is discouraged. Prefer the
 *  operations so internal shape changes don't break callers.
 */
type Rules = Readonly<{
  isEmpty: (value: A) => boolean
  requiredMessage: Option.Option<RuleMessage<A>>
  rules: ReadonlyArray<Rule<A>>
}>
```

### Valid

type

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L14)

```
/** The `Valid` state: every rule passed. */
type Valid = Readonly<{
  _tag: "Valid"
  value: A
}>
```

### Validating

type

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L11)

```
/** The `Validating` state: async validation is in flight. */
type Validating = Readonly<{
  _tag: "Validating"
  value: A
}>
```

## Constants

### Field

const

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L24)

```
/**
 * Builds the four-state `Field` Schema for a value of the given Schema. Put the
 *  result in your Model. The value Schema should match what the control
 *  actually holds as the user edits, not the type you parse it into:
 *  `Field(S.String)` for text inputs, `Field(S.Array(S.String))` for a
 *  multi-select. A scalar like a checkbox's boolean usually stays plain
 *  `S.Boolean` in the Model; wrap it in `Field` only when it needs the
 *  validation lifecycle. Validation rules stay separate, in a `makeRules`
 *  bundle.
 */
const Field: (valueSchema: Codec<A, I>) => Union<readonly [
  TaggedStruct<"NotValidated", {
    value: Codec<A, I, never, never>
  }>,
  TaggedStruct<"Validating", {
    value: Codec<A, I, never, never>
  }>,
  TaggedStruct<"Valid", {
    value: Codec<A, I, never, never>
  }>,
  TaggedStruct<"Invalid", {
    errors: NonEmptyArray<String>
    value: Codec<A, I, never, never>
  }>
]>
```

### Invalid

const

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L17)

```
/** Constructs an `Invalid` state. */
const Invalid: (field: Readonly<{
  errors: Array.NonEmptyReadonlyArray<string>
  value: A
}>) => Invalid<A>
```

### NotValidated

const

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L8)

```
/** Constructs a `NotValidated` state. */
const NotValidated: (field: Readonly<{
  value: A
}>) => NotValidated<A>
```

### Valid

const

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L14)

```
/** Constructs a `Valid` state. */
const Valid: (field: Readonly<{
  value: A
}>) => Valid<A>
```

### Validating

const

[source](https://github.com/foldkit/foldkit/blob/b86a18681e6b60cd0669cc2a28c02e913d2b4600/packages/foldkit/src/fieldValidation/fieldValidation.ts#L11)

```
/** Constructs a `Validating` state. */
const Validating: (field: Readonly<{
  value: A
}>) => Validating<A>
```
