---
url: https://foldkit.dev/api-reference/field-validation-rule
title: "FieldValidation/Rule"
description: "API documentation for the FieldValidation/Rule module."
access_date: 2026-08-17T05:12:55.977Z
current_date: 2026-08-17T05:12:55.977Z
---

# FieldValidation/Rule

## Functions

### email

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L52)

```
/** Creates a `Rule` that checks if a string is a valid email format. */
(message: RuleMessage<string>): Rule<string>
```

### endsWith

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L86)

```
/** Creates a `Rule` that checks if a string ends with a specified suffix. */
(
  suffix: string,
  message?: RuleMessage<string>
): Rule<string>
```

### equals

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L104)

```
/** Creates a `Rule` that checks if a string exactly matches an expected value. */
(
  expected: string,
  message?: RuleMessage<string>
): Rule<string>
```

### fromSchema

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L153)

```
/**
 * Creates a `Rule` that passes when the value decodes through `schema`.
 * 
 *  Use it to reuse a Schema you already maintain (a domain codec, a refined or
 *  branded type you decode to on submit) as a field rule, so the rule stays in
 *  sync with the schema instead of duplicating its logic. It does nothing a
 *  custom rule can't, so prefer the dedicated rules for plain checks. Decoding
 *  is synchronous: the schema must decode without running an effect.
 */
<A, I>(
  schema: Codec<A, I>,
  message: RuleMessage<I>
): Rule<I>
```

### includes

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L95)

```
/** Creates a `Rule` that checks if a string contains a specified substring. */
(
  substring: string,
  message?: RuleMessage<string>
): Rule<string>
```

### maxItems

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L136)

```
/** Creates a `Rule` that checks an array holds at most `max` items. */
(
  max: number,
  message?: RuleMessage<readonly Array<unknown>>
): Rule<readonly Array<unknown>>
```

### maxLength

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L35)

```
/** Creates a `Rule` that checks if a string does not exceed a maximum length. */
(
  max: number,
  message?: RuleMessage<string>
): Rule<string>
```

### minItems

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L127)

```
/** Creates a `Rule` that checks an array holds at least `min` items. */
(
  min: number,
  message?: RuleMessage<readonly Array<unknown>>
): Rule<readonly Array<unknown>>
```

### minLength

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L26)

```
/** Creates a `Rule` that checks if a string meets a minimum length. */
(
  min: number,
  message?: RuleMessage<string>
): Rule<string>
```

### oneOf

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L113)

```
/** Creates a `Rule` that checks if a string is one of a specified set of allowed values. */
(
  values: readonly Array<string>,
  message?: RuleMessage<string>
): Rule<string>
```

### pattern

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L44)

```
/** Creates a `Rule` that checks if a string matches a regular expression. */
(
  regex: RegExp,
  message: RuleMessage<string>
): Rule<string>
```

### resolveMessage

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L20)

```
/** Resolves a `RuleMessage` to a concrete string, applying it to the value when the message is a function. */
<A>(
  message: RuleMessage<A>,
  value: A
): string
```

### startsWith

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L77)

```
/** Creates a `Rule` that checks if a string begins with a specified prefix. */
(
  prefix: string,
  message?: RuleMessage<string>
): Rule<string>
```

### url

function

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L63)

```
/**
 * Creates a `Rule` that checks if a string is a valid URL format.
 * 
 *  By default the URL must include an `http://` or `https://` protocol.
 *  Pass `{ requireProtocol: false }` to accept bare domains.
 */
(options: Readonly<{
  message: RuleMessage<string>
  requireProtocol: boolean
}>): Rule<string>
```

## Types

### Rule

type

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L17)

```
/** A tuple of a predicate and error message used for field validation. */
type Rule = readonly [Predicate.Predicate<A>, RuleMessage<A>]
```

### RuleMessage

type

[source](https://github.com/foldkit/foldkit/blob/da6a434e6b200b073e4fbbeae5154287927c7c8d/packages/foldkit/src/fieldValidation/rule.ts#L14)

```
/** An error message for a rule: either a static string, or a function that receives the invalid value. */
type RuleMessage = string | (value: A) => string
```
