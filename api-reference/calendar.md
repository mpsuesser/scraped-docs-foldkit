---
url: https://foldkit.dev/api-reference/calendar
title: "Calendar"
description: "API documentation for the Calendar module."
access_date: 2026-08-20T21:25:20.391Z
current_date: 2026-08-20T21:25:20.391Z
---

# Calendar

## Functions

### dayOfWeek

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/info.ts#L60)

```
/** Returns the day of the week for a calendar date. */
(self: {
  day: number
  month: number
  year: number
}): "Sunday" | "Monday" | "Tuesday" | "Wednesday" | "Thursday" | "Friday" | "Saturday"
```

### daysInMonth

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L37)

```
/**
 * Returns the number of days in a given month of a given year.
 * Leap-year-aware: February returns 29 in leap years, 28 otherwise.
 */
(
  year: number,
  month: number
): number
```

### firstOfMonth

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/info.ts#L89)

```
/** Returns the first day of the month containing `self`. */
(self: {
  day: number
  month: number
  year: number
}): {
  day: number
  month: number
  year: number
}
```

### fromDateInZone

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L155)

```
/**
 * Constructs a `CalendarDate` from a JavaScript `Date` object, reading the
 * year/month/day in a specific IANA timezone (e.g. `"America/New_York"`).
 */
(
  date: Date,
  timeZone: string
): {
  day: number
  month: number
  year: number
}
```

### fromDateLocal

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L141)

```
/**
 * Constructs a `CalendarDate` from a JavaScript `Date` object, reading the
 * year/month/day in the browser's local timezone.
 */
(date: Date): {
  day: number
  month: number
  year: number
}
```

### isLeapYear

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L20)

```
/**
 * Determines if a year is a leap year in the Gregorian calendar.
 * 
 * A year is a leap year if it is divisible by 4, except for century years
 * (divisible by 100) which must also be divisible by 400. So 2000 is a leap
 * year but 1900 is not.
 */
(year: number): boolean
```

### lastOfMonth

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/info.ts#L107)

```
/**
 * Returns the last day of the month containing `self`. Leap-year aware
 * for February.
 */
(self: {
  day: number
  month: number
  year: number
}): {
  day: number
  month: number
  year: number
}
```

### make

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L115)

```
/**
 * Constructs a `CalendarDate`, validating via Schema.
 * Throws a `ParseError` if the combination is not a real calendar date
 * (e.g. February 30, month 13, day 0).
 */
(
  year: number,
  month: number,
  day: number
): {
  day: number
  month: number
  year: number
}
```

### toDateLocal

function

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L186)

```
/**
 * Converts a `CalendarDate` to a JavaScript `Date` object representing
 * midnight at the start of that day in the browser's local timezone.
 * 
 * Note: `Date` objects always carry a time and timezone component. This
 * function intentionally pins the time to local midnight. For cross-timezone
 * use, pass the result through an `Intl.DateTimeFormat` with an explicit
 * timezone, or keep working with `CalendarDate`.
 */
(calendarDate: {
  day: number
  month: number
  year: number
}): Date
```

## Constants

### CalendarDate

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L67)

```
/**
 * A calendar date — year, month, day. No time, no timezone.
 * 
 * Models the same concept as Java's `LocalDate` or TC39's `Temporal.PlainDate`.
 * Useful when you need to represent a date without a clock attached —
 * birthdays, deadlines, form date inputs, event calendars.
 * 
 * Validation ensures the date is a real calendar date: months are 1-12 and
 * days are within the month's actual length. Leap-year-aware, so February 30
 * is rejected and February 29 is only accepted in leap years.
 */
const CalendarDate: Struct<{
  day: Int
  month: Int
  year: Int
}>
```

### CalendarDateFromIsoString

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L211)

```
/**
 * Schema codec between an ISO 8601 date string (`YYYY-MM-DD`) and a
 * `CalendarDate`. Useful for form inputs, JSON serialization, URL query
 * parameters, and hidden form input values.
 * 
 * Decoding accepts only zero-padded ISO dates. Invalid calendar dates like
 * `2026-02-30` decode the string shape but fail the `CalendarDate` filter.
 */
const CalendarDateFromIsoString: decodeTo<Struct<{
  day: Int
  month: Int
  year: Int
}>, String, never, never>
```

### DayOfWeek

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/info.ts#L10)

```
/**
 * Schema for days of the week, Sunday through Saturday. Tagged union preferred
 * over 0-6 numeric indices to avoid magic numbers at call sites.
 */
const DayOfWeek: Literals<readonly ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]>
```

### Equivalence

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L36)

```
/** Value-based equivalence for calendar dates. */
const Equivalence: Equivalence_.Equivalence<CalendarDate>
```

### LocaleConfig

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/locale.ts#L40)

```
/**
 * Locale configuration for rendering calendar dates. Contains only data —
 * month/day names and the first day of the week. Formatting functions
 * (`formatLong`, `formatShort`, `formatAriaLabel`) are separate exports that
 * take a `LocaleConfig` as input.
 * 
 * Day names are always stored Sunday-first in the config; `firstDayOfWeek`
 * controls how the view rotates them at render time.
 */
const LocaleConfig: Struct<{
  dayNames: Tuple<readonly [String, String, String, String, String, String, String]>
  firstDayOfWeek: Literals<readonly ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]>
  monthNames: Tuple<readonly [String, String, String, String, String, String, String, String, String, String, String, String]>
  shortDayNames: Tuple<readonly [String, String, String, String, String, String, String]>
  shortMonthNames: Tuple<readonly [String, String, String, String, String, String, String, String, String, String, String, String]>
}>
```

### Order

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L27)

```
/**
 * Total ordering over calendar dates. Uses lexicographic comparison on
 * `year`, then `month`, then `day` — which matches calendar chronology
 * because the struct fields are already in the right order.
 */
const Order: Order_.Order<CalendarDate>
```

### addDays

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/arithmetic.ts#L76)

```
/**
 * Adds `n` days to a calendar date. Negative `n` subtracts days.
 * Handles month and year rollovers correctly in both directions.
 */
const addDays: (n: number) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### addMonths

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/arithmetic.ts#L127)

```
/**
 * Adds `n` months to a calendar date. Negative `n` subtracts months.
 * 
 * Clamps the day to the last valid day of the resulting month when the
 * original day would exceed it. So `addMonths(make(2026, 1, 31), 1)` returns
 * February 28, 2026 (not March 3).
 */
const addMonths: (n: number) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### addYears

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/arithmetic.ts#L169)

```
/**
 * Adds `n` years to a calendar date. Handles leap-year edge cases by clamping
 * day-of-month when the target year's month is shorter (February 29 in a
 * leap year + 1 year = February 28).
 */
const addYears: (n: number) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### between

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L169)

```
/** Returns `true` when `self` is within the inclusive range `[minimum, maximum]`. */
const between: (options: {
  maximum: CalendarDate
  minimum: CalendarDate
}) => (self: {
  day: number
  month: number
  year: number
}) => boolean
```

### clamp

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L209)

```
/**
 * Clamps `self` to the inclusive range `[minimum, maximum]`. Returns
 * `minimum` when `self` is before it, `maximum` when `self` is after it,
 * and `self` itself otherwise.
 */
const clamp: (options: {
  maximum: CalendarDate
  minimum: CalendarDate
}) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### daysSince

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/arithmetic.ts#L231)

```
/**
 * Returns the number of days from `start` until `self`, positive when `self`
 * is after `start`, negative when before, zero when equal. Matches
 * `Temporal.PlainDate.since`.
 */
const daysSince: (start: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => number
```

### daysUntil

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/arithmetic.ts#L205)

```
/**
 * Returns the number of days from `self` until `end`, positive when `end` is
 * after `self`, negative when before, zero when equal. Matches `Temporal.PlainDate.until`.
 */
const daysUntil: (end: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => number
```

### defaultEnglishLocale

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/locale.ts#L55)

```
/**
 * Default English (United States) locale. Picker components default to this
 * when no locale is passed via ViewConfig. Consumers who want a different
 * locale pass their own `LocaleConfig`.
 */
const defaultEnglishLocale: LocaleConfig
```

### endOfWeek

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/info.ts#L142)

```
/** Returns the end-of-week date — six days after the start of the week. */
const endOfWeek: (firstDayOfWeek: "Sunday" | "Monday" | "Tuesday" | "Wednesday" | "Thursday" | "Friday" | "Saturday") => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### formatAriaLabel

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/locale.ts#L188)

```
/**
 * Renders an accessibility label for a calendar date, suitable for
 * `aria-label` on a grid cell. Example: `"Monday, January 15, 2026"`.
 */
const formatAriaLabel: (locale: Calendar.LocaleConfig) => (self: {
  day: number
  month: number
  year: number
}) => string
```

### formatLong

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/locale.ts#L155)

```
/** Renders a calendar date in long form. Example: `"January 15, 2026"`. */
const formatLong: (locale: Calendar.LocaleConfig) => (self: {
  day: number
  month: number
  year: number
}) => string
```

### formatShort

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/locale.ts#L167)

```
/** Renders a calendar date in short form. Example: `"Jan 15, 2026"`. */
const formatShort: (locale: Calendar.LocaleConfig) => (self: {
  day: number
  month: number
  year: number
}) => string
```

### isAfter

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L81)

```
/** Returns `true` when `self` is strictly after `that`. */
const isAfter: (that: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => boolean
```

### isAfterOrEqual

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L103)

```
/** Returns `true` when `self` is after or equal to `that`. */
const isAfterOrEqual: (that: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => boolean
```

### isBefore

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L70)

```
/** Returns `true` when `self` is strictly before `that`. */
const isBefore: (that: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => boolean
```

### isBeforeOrEqual

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L92)

```
/** Returns `true` when `self` is before or equal to `that`. */
const isBeforeOrEqual: (that: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => boolean
```

### isCalendarDate

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/calendarDate.ts#L98)

```
/**
 * Type guard for `CalendarDate`. Returns true when `value` is a valid
 * calendar date struct.
 */
const isCalendarDate: (value: unknown) => unknown
```

### isEqual

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L51)

```
/** Returns `true` when two calendar dates represent the same day. */
const isEqual: (that: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => boolean
```

### max

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L138)

```
/** Returns the later of two calendar dates. */
const max: (that: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### min

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/comparison.ts#L126)

```
/** Returns the earlier of two calendar dates. */
const min: (that: {
  day: number
  month: number
  year: number
}) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### startOfWeek

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/info.ts#L126)

```
/**
 * Returns the start-of-week date for the week containing `self`, where
 * `firstDayOfWeek` specifies which day begins the week. Returns `self`
 * itself when it already falls on `firstDayOfWeek`.
 */
const startOfWeek: (firstDayOfWeek: "Sunday" | "Monday" | "Tuesday" | "Wednesday" | "Thursday" | "Friday" | "Saturday") => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### subtractDays

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/arithmetic.ts#L100)

```
/** Subtracts `n` days from a calendar date. Equivalent to `addDays(self, -n)`. */
const subtractDays: (n: number) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### subtractMonths

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/arithmetic.ts#L144)

```
/** Subtracts `n` months from a calendar date. Equivalent to `addMonths(self, -n)`. */
const subtractMonths: (n: number) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### subtractYears

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/arithmetic.ts#L181)

```
/** Subtracts `n` years from a calendar date. Equivalent to `addYears(self, -n)`. */
const subtractYears: (n: number) => (self: {
  day: number
  month: number
  year: number
}) => {
  day: number
  month: number
  year: number
}
```

### today

const

[source](https://github.com/foldkit/foldkit/blob/a0c2b4e65417f7d9c98b4dbc290093be950345c2/packages/foldkit/src/calendar/today.ts#L29)

```
/**
 * Effect-based accessors for the current calendar date. Uses Effect's `Clock`
 * service under the hood, so tests can freeze time with `TestClock` and
 * production uses the real system clock.
 * 
 * This is the **only** impurity boundary in the calendar module — every
 * other function in this module is referentially transparent.
 */
const today: {
  inZone: (timeZone: string) => Effect<{
    day: number
    month: number
    year: number
  }, never, never>
  local: Effect<{
    day: number
    month: number
    year: number
  }, never, never>
}
```
