---
url: https://foldkit.dev/api-reference/ui-calendar
title: "Ui/Calendar"
description: "API documentation for the Ui/Calendar module."
access_date: 2026-08-11T02:16:28.996Z
current_date: 2026-08-11T02:16:28.996Z
---

# Ui/Calendar

## Functions

### dropToDays

function

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L293)

```
/**
 * Returns the calendar to Days mode regardless of current depth. Useful for
 * standalone (non-popovered) consumers that want to wire their own back-out
 * gesture. Popovered consumers like `DatePicker` don't need this. Escape
 * closes the popover, and the calendar resets to Days on next open.
 * 
 * Reconciles `maybeFocusedDate` to a date inside the visible (`viewYear`,
 * `viewMonth`). Months/Years navigation can leave the cursor on a date
 * outside the days grid (paged-away year, etc.), which would otherwise
 * cause `aria-activedescendant` to point at a non-rendered cell and the
 * next ArrowLeft to jump to the cursor's stale year.
 */
(model: Calendar.Model): Calendar.Model
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L164)

```
/**
 * Creates an initial calendar model. The selected date is owned by the
 * parent and passed in via `ViewInputs.maybeSelectedDate`. The view month
 * defaults to `initialViewDate` (pass the parent's selected date to open onto
 * it), or today when omitted.
 */
(config: InitConfig): Calendar.Model
```

### selectDate

function

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L212)

```
/**
 * Programmatically selects a date on the calendar, committing it as the
 * chosen value and moving the cursor onto it. Use this in controlled-mode
 * handlers (when the view's `onSelectedDate` callback is provided) to write
 * the selection back to the calendar's internal state.
 * 
 * Equivalent to dispatching `ClickedDay({ date })` through `update`.
 */
(
  model: Calendar.Model,
  date: {
    day: number
    month: number
    year: number
  }
): UpdateReturn
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L553)

## Types

### CalendarAttributes

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L876)

```
/**
 * Discriminated union of attribute groups and derived data the calendar
 * component provides to the consumer's `toView` callback. The variant
 * matches `model.viewMode`. Pattern-match on `_tag` with `M.tagsExhaustive`
 * to render each mode.
 */
type CalendarAttributes = DaysModeAttributes | MonthsModeAttributes | YearsModeAttributes
```

### ColumnHeader

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L793)

```
/** A column header for the day grid's first row (day-of-week labels). */
type ColumnHeader = Readonly<{
  attributes: ReadonlyArray<ChildAttribute>
  name: string
}>
```

### DayCell

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L780)

```
/** Information about a single day cell in the rendered calendar grid. */
type DayCell = Readonly<{
  buttonAttributes: ReadonlyArray<ChildAttribute>
  cellAttributes: ReadonlyArray<ChildAttribute>
  date: CalendarDate
  isDisabled: boolean
  isFocused: boolean
  isInViewMonth: boolean
  isSelected: boolean
  isToday: boolean
  label: string
}>
```

### DaysModeAttributes

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L835)

```
/** Attributes provided to the consumer when rendering the day grid. */
type DaysModeAttributes = Readonly<{
  _tag: "Days"
  columnHeaders: ReadonlyArray<ColumnHeader>
  grid: ReadonlyArray<ChildAttribute>
  headerRow: ReadonlyArray<ChildAttribute>
  heading: Readonly<{
    id: string
    text: string
  }>
  headingButton: ReadonlyArray<ChildAttribute>
  nextMonthButton: ReadonlyArray<ChildAttribute>
  previousMonthButton: ReadonlyArray<ChildAttribute>
  root: ReadonlyArray<ChildAttribute>
  weeks: ReadonlyArray<Week>
}>
```

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L149)

```
/** Configuration for creating a calendar model with `init`. */
type InitConfig = Readonly<{
  disabledDates: ReadonlyArray<CalendarDate>
  disabledDaysOfWeek: ReadonlyArray<Calendar.DayOfWeek>
  id: string
  initialViewDate: CalendarDate
  locale: Calendar.LocaleConfig
  maxDate: CalendarDate
  minDate: CalendarDate
  today: CalendarDate
}>
```

### MonthCell

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L810)

```
/**
 * Information about a single month cell in the rendered months grid.
 * `label` is the locale-aware full month name (e.g. "September"); `shortLabel`
 * is the locale-aware abbreviation (e.g. "Sep"). Render whichever fits the
 * cell. Never substring `label` to abbreviate, since that's not safe across
 * locales.
 */
type MonthCell = Readonly<{
  buttonAttributes: ReadonlyArray<ChildAttribute>
  cellAttributes: ReadonlyArray<ChildAttribute>
  isCurrentMonth: boolean
  isDisabled: boolean
  isFocused: boolean
  isSelected: boolean
  label: string
  month: number
  shortLabel: string
}>
```

### MonthsModeAttributes

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L851)

```
/**
 * Attributes provided to the consumer when rendering the months grid. The
 * 12 cells are pre-built in calendar (locale-ordered). The consumer arranges
 * them in whatever grid layout they prefer (3×4 is the typical choice).
 */
type MonthsModeAttributes = Readonly<{
  _tag: "Months"
  cells: ReadonlyArray<MonthCell>
  grid: ReadonlyArray<ChildAttribute>
  heading: Readonly<{
    id: string
    text: string
  }>
  headingButton: ReadonlyArray<ChildAttribute>
  root: ReadonlyArray<ChildAttribute>
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L887)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  The Calendar dispatches its own `ClickedDay` message on date commit
 *  and emits a `SelectedDate` OutMessage. Consumers handle date
 *  selection by pattern-matching the OutMessage in their
 *  `GotCalendarMessage` handler.
 */
type ViewInputs = Readonly<{
  daysHeadingButtonLabel: string
  maybeSelectedDate: Option.Option<CalendarDate>
  monthsHeadingButtonLabel: string
  nextMonthLabel: string
  nextYearsPageLabel: string
  previousMonthLabel: string
  previousYearsPageLabel: string
  toView: (attributes: CalendarAttributes) => Html
}>
```

### Week

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L800)

```
/**
 * A single week row in the day grid, carrying its own row attributes (role,
 * aria-rowindex) alongside its 7 day cells.
 */
type Week = Readonly<{
  attributes: ReadonlyArray<ChildAttribute>
  cells: ReadonlyArray<DayCell>
}>
```

### YearCell

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L823)

```
/** Information about a single year cell in the rendered years grid. */
type YearCell = Readonly<{
  buttonAttributes: ReadonlyArray<ChildAttribute>
  cellAttributes: ReadonlyArray<ChildAttribute>
  isCurrentYear: boolean
  isDisabled: boolean
  isFocused: boolean
  isSelected: boolean
  label: string
  year: number
}>
```

### YearsModeAttributes

type

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L862)

```
/**
 * Attributes provided to the consumer when rendering the years grid. The
 * 12 cells span one paged window; prev/next buttons page by 12 years.
 */
type YearsModeAttributes = Readonly<{
  _tag: "Years"
  cells: ReadonlyArray<YearCell>
  grid: ReadonlyArray<ChildAttribute>
  heading: Readonly<{
    id: string
    text: string
  }>
  nextPageButton: ReadonlyArray<ChildAttribute>
  previousPageButton: ReadonlyArray<ChildAttribute>
  root: ReadonlyArray<ChildAttribute>
}>
```

## Constants

### BlurredGrid

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L90)

```
/** Sent when the grid container loses DOM focus. */
const BlurredGrid: CallableTaggedStruct<"BlurredGrid", {}>
```

### ChangedViewMonth

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L126)

```
/**
 * Emitted when the visible month changes due to navigation. Consumers of an
 * inline calendar may use this to load month-scoped data (holidays, events).
 * A click that commits a date in a different month emits `SelectedDate`, not
 * `ChangedViewMonth`. The parent infers the month change from the date.
 */
const ChangedViewMonth: CallableTaggedStruct<"ChangedViewMonth", {
  month: Int
  year: Int
}>
```

### ClickedDay

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L60)

```
/** Sent when the user clicks a day cell in the grid. */
const ClickedDay: CallableTaggedStruct<"ClickedDay", {
  date: Struct<{
    day: Int
    month: Int
    year: Int
  }>
}>
```

### ClickedHeading

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L75)

```
/**
 * Sent when the user clicks the calendar heading. Zooms out one mode
 * level: Days → Months, Months → Years. Terminal in Years mode.
 */
const ClickedHeading: CallableTaggedStruct<"ClickedHeading", {}>
```

### ClickedNextMonthButton

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L72)

```
/**
 * Sent when the user clicks the next-month navigation button in Days
 * mode. (The Years mode prev/next-page buttons dispatch `PagedYears`.)
 */
const ClickedNextMonthButton: CallableTaggedStruct<"ClickedNextMonthButton", {}>
```

### ClickedPreviousMonthButton

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L69)

```
/**
 * Sent when the user clicks the previous-month navigation button in Days
 * mode. (The Years mode prev/next-page buttons dispatch `PagedYears`.)
 */
const ClickedPreviousMonthButton: CallableTaggedStruct<"ClickedPreviousMonthButton", {}>
```

### CompletedFocusGrid

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L96)

```
/** Sent when a FocusGrid command completes. */
const CompletedFocusGrid: CallableTaggedStruct<"CompletedFocusGrid", {}>
```

### FocusGrid

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L196)

```
/**
 * Focuses the calendar grid container. Parent components like DatePicker
 * dispatch this after opening to hand focus to the grid's keyboard layer.
 */
const FocusGrid: CommandDefinitionWithArgs<"FocusGrid", {
  id: String
}, Effect<{
  _tag: "CompletedFocusGrid"
}, never, never>>
```

### FocusedGrid

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L88)

```
/** Sent when the grid container receives DOM focus. */
const FocusedGrid: CallableTaggedStruct<"FocusedGrid", {}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L99)

```
/** Union of all messages the calendar component can produce. */
const Message: Union<readonly [
  CallableTaggedStruct<"ClickedDay", {
    date: Struct<{
      day: Int
      month: Int
      year: Int
    }>
  }>,
  CallableTaggedStruct<"PressedKeyOnGrid", {
    isShift: Boolean
    key: String
  }>,
  CallableTaggedStruct<"ClickedPreviousMonthButton", {}>,
  CallableTaggedStruct<"ClickedNextMonthButton", {}>,
  CallableTaggedStruct<"ClickedHeading", {}>,
  CallableTaggedStruct<"SelectedMonth", {
    month: Int
  }>,
  CallableTaggedStruct<"SelectedYear", {
    year: Int
  }>,
  CallableTaggedStruct<"PagedYears", {
    direction: Literals<readonly [1, -1]>
  }>,
  CallableTaggedStruct<"FocusedGrid", {}>,
  CallableTaggedStruct<"BlurredGrid", {}>,
  CallableTaggedStruct<"RefreshedToday", {
    today: Struct<{
      day: Int
      month: Int
      year: Int
    }>
  }>,
  CallableTaggedStruct<"CompletedFocusGrid", {}>
]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L40)

```
/**
 * Schema for the calendar component's state. Tracks the visible month/year,
 * the keyboard-focused and user-selected dates, the active view mode, and
 * the configuration that governs navigation (locale, min/max, disabled
 * days).
 */
const Model: Struct<{
  disabledDates: $Array<Struct<{
    day: Int
    month: Int
    year: Int
  }>>
  disabledDaysOfWeek: $Array<Literals<readonly ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]>>
  id: String
  isGridFocused: Boolean
  locale: Struct<{
    dayNames: Tuple<readonly [String, String, String, String, String, String, String]>
    firstDayOfWeek: Literals<readonly ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]>
    monthNames: Tuple<readonly [String, String, String, String, String, String, String, String, String, String, String, String]>
    shortDayNames: Tuple<readonly [String, String, String, String, String, String, String]>
    shortMonthNames: Tuple<readonly [String, String, String, String, String, String, String, String, String, String, String, String]>
  }>
  maybeFocusedDate: Option<Struct<{
    day: Int
    month: Int
    year: Int
  }>>
  maybeMaxDate: Option<Struct<{
    day: Int
    month: Int
    year: Int
  }>>
  maybeMinDate: Option<Struct<{
    day: Int
    month: Int
    year: Int
  }>>
  today: Struct<{
    day: Int
    month: Int
    year: Int
  }>
  viewMode: Literals<readonly ["Days", "Months", "Years"]>
  viewMonth: Int
  viewYear: Int
}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L140)

```
/** Union of the calendar's OutMessages. */
const OutMessage: Union<readonly [
  CallableTaggedStruct<"ChangedViewMonth", {
    month: Int
    year: Int
  }>,
  CallableTaggedStruct<"SelectedDate", {
    date: Struct<{
      day: Int
      month: Int
      year: Int
    }>
  }>
]>
```

### PagedYears

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L84)

```
/**
 * Sent when the user pages the years grid forward or backward by one
 * window. Direction is `1` for next, `-1` for previous.
 */
const PagedYears: CallableTaggedStruct<"PagedYears", {
  direction: Literals<readonly [1, -1]>
}>
```

### PressedKeyOnGrid

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L63)

```
/**
 * Sent when the user presses a key on the grid container. The update maps
 * the key to a navigation or selection action.
 */
const PressedKeyOnGrid: CallableTaggedStruct<"PressedKeyOnGrid", {
  isShift: Boolean
  key: String
}>
```

### RefreshedToday

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L92)

```
/** Sent when a long-lived session's "today" reference should be refreshed. */
const RefreshedToday: CallableTaggedStruct<"RefreshedToday", {
  today: Struct<{
    day: Int
    month: Int
    year: Int
  }>
}>
```

### SelectedDate

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L135)

```
/**
 * Emitted when the user commits a date selection via click or keyboard. The
 * calendar's internal state already reflects the new selection by the time
 * this fires; consumers react by lifting the date into their domain state
 * (closing a popover, advancing a form step, etc.).
 */
const SelectedDate: CallableTaggedStruct<"SelectedDate", {
  date: Struct<{
    day: Int
    month: Int
    year: Int
  }>
}>
```

### SelectedMonth

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L78)

```
/**
 * Sent when the user picks a month from the months grid. Jumps the view
 * to that month and returns the calendar to Days mode.
 */
const SelectedMonth: CallableTaggedStruct<"SelectedMonth", {
  month: Int
}>
```

### SelectedYear

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L81)

```
/**
 * Sent when the user picks a year from the years grid. Jumps the view to
 * that year and transitions the calendar to Months mode for further drilling.
 */
const SelectedYear: CallableTaggedStruct<"SelectedYear", {
  year: Int
}>
```

### ViewMode

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L33)

```
/**
 * Which grid the calendar is currently displaying. `Days` is the standard
 * 6×7 day grid; `Months` is a 3×4 month-name grid for fast month jumps;
 * `Years` is a 3×4 year grid paged in 12-year windows for fast year jumps.
 */
const ViewMode: Literals<readonly ["Days", "Months", "Years"]>
```

### focusDate

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L220)

```
/**
 * Moves the calendar's view and cursor to a date without changing the
 *  selection (which the parent owns). Use it to navigate to a known date, for
 *  example when opening a date picker onto its current value so the selected
 *  month is visible. Returns the model directly because it produces no
 *  commands and no OutMessage.
 */
const focusDate: Reflect<Model, CalendarDate>
```

### reflectDisabledDates

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L260)

```
/**
 * Reflects the list of individually-disabled dates onto the model. Pass an
 * empty array to clear. Does NOT reconcile the current selection.
 */
const reflectDisabledDates: Reflect<Model, ReadonlyArray<CalendarDate>>
```

### reflectDisabledDaysOfWeek

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L272)

```
/**
 * Reflects the days of the week that are disabled (e.g. weekends) onto the
 * model. Pass an empty array to clear. Does NOT reconcile the current
 * selection.
 */
const reflectDisabledDaysOfWeek: Reflect<Model, ReadonlyArray<Calendar.DayOfWeek>>
```

### reflectMaxDate

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L249)

```
/**
 * Reflects the maximum selectable date onto the model. Pass `Option.none()`
 * to remove the maximum. Does NOT reconcile the current selection.
 */
const reflectMaxDate: Reflect<Model, Option.Option<CalendarDate>>
```

### reflectMinDate

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L238)

```
/**
 * Reflects the minimum selectable date onto the model. Pass `Option.none()`
 * to remove the minimum. Use this when the minimum derives from other Model
 * state (e.g. a start date field whose current selection constrains an end
 * date picker).
 * 
 * Does NOT reconcile the current selection. If a previously-selected date
 * is now below the new minimum, it remains selected. Callers should clear or
 * reassign the selection explicitly if their domain requires it.
 */
const reflectMinDate: Reflect<Model, Option.Option<CalendarDate>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/c7e01b2f8894f66ad1c3e876a3714c52c01febcf/packages/ui/src/calendar/index.ts#L1383)

```
/**
 * Renders an accessible calendar. Publishes mode-specific ARIA attribute
 *  bundles + derived cell data, then delegates layout to the consumer's
 *  `toView` callback. The variant of `CalendarAttributes` passed to
 *  `toView` matches `model.viewMode`.
 */
const view: SubmodelView<Calendar.Model, {
  _tag: "ClickedDay"
  date: {
    day: number
    month: number
    year: number
  }
} | {
  _tag: "PressedKeyOnGrid"
  isShift: boolean
  key: string
} | {
  _tag: "ClickedPreviousMonthButton"
} | {
  _tag: "ClickedNextMonthButton"
} | {
  _tag: "ClickedHeading"
} | {
  _tag: "SelectedMonth"
  month: number
} | {
  _tag: "SelectedYear"
  year: number
} | {
  _tag: "PagedYears"
  direction: -1 | 1
} | {
  _tag: "FocusedGrid"
} | {
  _tag: "BlurredGrid"
} | {
  _tag: "RefreshedToday"
  today: {
    day: number
    month: number
    year: number
  }
} | {
  _tag: "CompletedFocusGrid"
}, Readonly<{
  daysHeadingButtonLabel: string
  maybeSelectedDate: Option<{
    day: number
    month: number
    year: number
  }>
  monthsHeadingButtonLabel: string
  nextMonthLabel: string
  nextYearsPageLabel: string
  previousMonthLabel: string
  previousYearsPageLabel: string
  toView: (attributes: CalendarAttributes) => Html
}>>
```
