---
url: https://foldkit.dev/ui/calendar
title: "Calendar"
description: "Accessible inline calendar grid with 2D keyboard navigation, locale-aware headers, and min/max/disabled-date constraints."
access_date: 2026-08-03T18:23:29.850Z
current_date: 2026-08-03T18:23:29.850Z
---

# Calendar

## Overview

An accessible inline calendar grid built to the WAI-ARIA grid pattern. Calendar manages the 2D keyboard navigation state machine and renders a 6×7 grid of days with full screen-reader support. Use it standalone for scheduling UIs and event calendars, or as the foundation of a date picker.

The calendar heading is a button: clicking it switches the day grid into a 3×4 months grid. Clicking the year heading from there switches into a paged 3×4 years grid (prev/next page through 12-year windows). Selecting a year drills back to the months grid for that year; selecting a month drills back to the days grid for that month.

Calendar uses the Submodel pattern: initialize with `Calendar.init()`, store the Model in your parent, delegate Messages via `Calendar.update()`, and render with `Calendar.view()`. The update function returns `[Model, Commands, Option<OutMessage>]`. The parent owns the selected date: store it in your Model, pass it back as `maybeSelectedDate`, and fold the `SelectedDate` OutMessage into that field. The OutMessage lets the parent handle meaningful events, for example date selection or month changes.

See it in an app

Check out how Calendar is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/calendar.ts).

## Examples

A basic calendar with today highlighted. Click a day to select it, or tab into the grid and use the arrow keys. Navigation follows the full WAI-ARIA pattern including Home/End, PageUp/Down, and Shift+PageUp/Down for year jumps.

Sun

Mon

Tue

Wed

Thu

Fri

Sat

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Effect, Match as M, Option } from 'effect'
import { Calendar, Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Calendar as UiCalendar } from '@foldkit/ui'

// Add a field to your Model for the Calendar Submodel, plus a field the
// parent owns for the selected date. The calendar no longer stores the
// selection; the parent holds it and passes it back in as `maybeSelectedDate`.
const Model = S.Struct({
  calendarDemo: UiCalendar.Model,
  maybeSelectedDate: S.Option(Calendar.CalendarDate),
  // ...your other fields
})

// Fetch `today` once at the app boundary via flags so init stays pure:
const Flags = S.Struct({
  today: Calendar.CalendarDate,
  // ...your other flags
})

const flags = Effect.gen(function* () {
  const today = yield* Calendar.today.local
  return { today /* ...your other flags */ }
})

// In your init function, pass the flags-resolved today into UiCalendar.init.
// `initialViewDate` seeds the month the calendar opens onto (pass your
// initial selection to open on it). The parent owns the selection itself:
const init = (flags: Flags) => [
  {
    calendarDemo: UiCalendar.init({
      id: 'calendar-demo',
      today: flags.today,
      initialViewDate: flags.today,
    }),
    maybeSelectedDate: Option.none(),
    // ...your other fields
  },
  [],
]

// Embed the Calendar Message in your parent Message for navigation and
// keyboard routing:
const GotCalendarMessage = m('GotCalendarMessage', {
  message: UiCalendar.Message,
})

// Inside your update function's M.tagsExhaustive({...}), delegate
// navigation, focus, and picker-mode transitions to UiCalendar.update.
// Its third tuple element is `Option<OutMessage>`. When the user
// commits a date (click, Enter, or Space) it carries `SelectedDate({ date })`.
// `ChangedViewMonth` fires when navigation shifts the visible month
// without selecting a date.
GotCalendarMessage: ({ message }) => {
  const [nextCalendar, commands, maybeOutMessage] = UiCalendar.update(
    model.calendarDemo,
    message,
  )
  const mappedCommands = Command.mapMessages(commands, message =>
    GotCalendarMessage({ message }),
  )

  return Option.match(maybeOutMessage, {
    onNone: () => [
      evo(model, { calendarDemo: () => nextCalendar }),
      mappedCommands,
    ],
    onSome: M.type<UiCalendar.OutMessage>().pipe(
      M.tagsExhaustive({
        SelectedDate: ({ date }) => [
          // The child has emitted `SelectedDate`. The body commits
          // the child's next state as usual. This is where the parent
          // lifts the committed date into its own field. That field is
          // then passed back to the calendar as `maybeSelectedDate`, so
          // the parent stays the single source of truth for the selection.
          evo(model, {
            calendarDemo: () => nextCalendar,
            maybeSelectedDate: () => Option.some(date),
          }),
          mappedCommands,
        ],
        ChangedViewMonth: () => [
          // The child has emitted `ChangedViewMonth`. The body commits
          // the child's next state as usual. In this arm the parent
          // can also update its own state or dispatch its own
          // Commands, for example prefetch month data, fire analytics,
          // or trigger a downstream Command.
          evo(model, { calendarDemo: () => nextCalendar }),
          mappedCommands,
        ],
      }),
    ),
  })
}

// Inside your view function, render the calendar. The `toView` callback
// receives a discriminated `CalendarAttributes` whose variant matches the
// calendar's current `viewMode`. Pattern-match on `_tag` to render the
// day grid, the months grid, or the years grid:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: model.calendarDemo.id,
    model: model.calendarDemo,
    view: UiCalendar.view,
    viewInputs: {
      // The parent-owned selection. The selected-day marker derives from it.
      maybeSelectedDate: model.maybeSelectedDate,
      toView: attributes =>
        M.value(attributes).pipe(
          M.tagsExhaustive({
            Days: days =>
              h.div(
                [
                  ...days.root,
                  h.Class('flex flex-col gap-3 rounded-xl border p-4'),
                ],
                [
                  h.div(
                    [h.Class('flex items-center justify-between')],
                    [
                      h.button(
                        [...days.previousMonthButton, h.Class('rounded px-2')],
                        ['‹'],
                      ),
                      // The heading is a button: clicking it switches to the
                      // months grid for fast navigation. Pair the text with a
                      // chevron so the button reads as interactive at rest.
                      h.button(
                        [
                          h.Id(days.heading.id),
                          ...days.headingButton,
                          h.Class(
                            'inline-flex items-center gap-2 rounded px-2 text-sm font-semibold',
                          ),
                        ],
                        [days.heading.text, ' ▾'],
                      ),
                      h.button(
                        [...days.nextMonthButton, h.Class('rounded px-2')],
                        ['›'],
                      ),
                    ],
                  ),
                  h.div(
                    [...days.grid, h.Class('flex flex-col gap-1 outline-none')],
                    [
                      h.div(
                        [...days.headerRow, h.Class('grid grid-cols-7 gap-1')],
                        days.columnHeaders.map(header =>
                          h.div(
                            [
                              ...header.attributes,
                              h.Class('text-center text-xs uppercase'),
                            ],
                            [header.name],
                          ),
                        ),
                      ),
                      ...days.weeks.map(week =>
                        h.div(
                          [
                            ...week.attributes,
                            h.Class('grid grid-cols-7 gap-1'),
                          ],
                          week.cells.map(cell =>
                            h.div(
                              // `group` lets day buttons style themselves from
                              // parent state via group-data-[today],
                              // group-data-[selected], etc.
                              [
                                ...cell.cellAttributes,
                                h.Class(
                                  'group flex items-center justify-center',
                                ),
                              ],
                              [
                                h.button(
                                  [
                                    ...cell.buttonAttributes,
                                    h.Class(
                                      'h-9 w-9 rounded-full text-sm group-data-[today]:ring-1 group-data-[selected]:bg-accent-600 group-data-[selected]:text-white group-data-[outside-month]:text-gray-400 group-data-[disabled]:opacity-40',
                                    ),
                                  ],
                                  [cell.label],
                                ),
                              ],
                            ),
                          ),
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            // The months grid renders 12 cells (one per month). Clicking the
            // heading again drills further into the years grid.
            Months: months =>
              h.div(
                [
                  ...months.root,
                  h.Class('flex flex-col gap-3 rounded-xl border p-4'),
                ],
                [
                  h.div(
                    [h.Class('flex items-center justify-center')],
                    [
                      h.button(
                        [
                          h.Id(months.heading.id),
                          ...months.headingButton,
                          h.Class(
                            'inline-flex items-center gap-2 rounded px-2 text-sm font-semibold',
                          ),
                        ],
                        [months.heading.text, ' ▾'],
                      ),
                    ],
                  ),
                  h.div(
                    [
                      ...months.grid,
                      h.Class('grid grid-cols-3 gap-1 outline-none'),
                    ],
                    months.cells.map(cell =>
                      h.div(
                        [
                          ...cell.cellAttributes,
                          h.Class('group flex items-center justify-center'),
                        ],
                        [
                          h.button(
                            [
                              ...cell.buttonAttributes,
                              h.Class(
                                'h-12 w-full rounded-md text-sm group-data-[selected]:bg-accent-600 group-data-[selected]:text-white group-data-[disabled]:opacity-40',
                              ),
                            ],
                            [cell.shortLabel],
                          ),
                        ],
                      ),
                    ),
                  ),
                ],
              ),
            // The years grid renders 12 cells (one paged window). Prev/next
            // page through 12-year windows; clicking a year drills back to
            // the months grid for that year.
            Years: years =>
              h.div(
                [
                  ...years.root,
                  h.Class('flex flex-col gap-3 rounded-xl border p-4'),
                ],
                [
                  h.div(
                    [h.Class('flex items-center justify-between')],
                    [
                      h.button(
                        [...years.previousPageButton, h.Class('rounded px-2')],
                        ['‹'],
                      ),
                      h.h2(
                        [
                          h.Id(years.heading.id),
                          h.Class('text-sm font-semibold'),
                        ],
                        [years.heading.text],
                      ),
                      h.button(
                        [...years.nextPageButton, h.Class('rounded px-2')],
                        ['›'],
                      ),
                    ],
                  ),
                  h.div(
                    [
                      ...years.grid,
                      h.Class('grid grid-cols-3 gap-1 outline-none'),
                    ],
                    years.cells.map(cell =>
                      h.div(
                        [
                          ...cell.cellAttributes,
                          h.Class('group flex items-center justify-center'),
                        ],
                        [
                          h.button(
                            [
                              ...cell.buttonAttributes,
                              h.Class(
                                'h-12 w-full rounded-md text-sm group-data-[selected]:bg-accent-600 group-data-[selected]:text-white group-data-[disabled]:opacity-40',
                              ),
                            ],
                            [cell.label],
                          ),
                        ],
                      ),
                    ),
                  ),
                ],
              ),
          }),
        ),
    },
    toParentMessage: message => GotCalendarMessage({ message }),
  })
```

## Styling

Calendar is headless. Your `toView` callback controls all markup and styling. The attribute groups carry ARIA and event wiring; data attributes on day cells let you style state variants with CSS selectors like `data-[today]:` and `group-data-[selected]:`.

Attribute

Condition

`data-today`

Present on the cell representing "today": the day cell in Days mode, the current month cell in Months mode, the current year cell in Years mode.

`data-selected`

Present on the calendar's currently-centered cell: the selected date in Days mode, the centered month (viewMonth) in Months mode, the centered year (viewYear) in Years mode.

`data-focused`

Present on the cell at the keyboard cursor position while the grid has DOM focus.

`data-outside-month`

(Days mode only.) Present on cells that fall outside the currently-viewed month (leading/trailing grid rows).

`data-disabled`

Present on cells disabled by min/max, disabledDaysOfWeek, or disabledDates.

## Keyboard Interaction

The grid container receives DOM focus, and navigation happens via `aria-activedescendant`. Screen readers announce the focused cell without moving browser focus. Disabled dates are skipped during navigation with a bounded cap so fully-disabled ranges terminate cleanly.

Key

Description

`ArrowLeft / ArrowRight`

Move the focus cursor by one cell. Days: ±1 day. Months: ±1 month (wraps across years). Years: ±1 year (wraps across pages).

`ArrowUp / ArrowDown`

Move the focus cursor by one row. Days: ±1 week (7 days). Months: ±1 row (3 months). Years: ±1 row (3 years).

`Home / End`

(Days mode only.) Move focus to the start / end of the current week (based on locale.firstDayOfWeek).

`PageUp / PageDown`

Days: ±1 month. Months: ±1 year. Years: ±1 window (12 years).

`Shift + PageUp / Shift + PageDown`

(Days mode only.) Move focus by one year.

`Enter / Space`

Commit the focus cursor. Days: select the date. Months: jump the calendar to that month and drill back to Days. Years: jump to that year and drill back to Months.

## Accessibility

The grid renders with `role="grid"` and an explicit `aria-label` that leads with a non-numeric word ("Calendar, April 2026") so VoiceOver doesn't pattern-match the grid's row position into a date literal. Each row has `role="row"`, column headers have `role="columnheader"`, and day cells have `role="gridcell"` with `aria-selected` set on the chosen date. Day buttons carry a full accessible name via `aria-label` (e.g. "Monday, April 13, 2026"), and disabled days get `aria-disabled="true"`.

## API Reference

### InitConfig

Configuration object passed to `Calendar.init()`.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the calendar instance.

`today`

`CalendarDate`

—

The current calendar date. Typically fetched at the app boundary via Calendar.today.local and threaded through flags.

`initialViewDate`

`CalendarDate`

—

Seeds the month the calendar opens onto. When set, the view starts on the month containing this date. The parent owns the selection itself; pass your initial selected date here to open onto it.

`locale`

`LocaleConfig`

`defaultEnglishLocale`

Month and day names plus the first day of the week. Import from foldkit/calendar.

`minDate`

`CalendarDate`

—

Earliest selectable date. Dates before minDate are marked disabled and skipped by keyboard navigation.

`maxDate`

`CalendarDate`

—

Latest selectable date. Dates after maxDate are marked disabled and skipped by keyboard navigation.

`disabledDaysOfWeek`

`ReadonlyArray<DayOfWeek>`

`[]`

Days of the week to disable (e.g. ["Saturday", "Sunday"] for weekday-only selection).

`disabledDates`

`ReadonlyArray<CalendarDate>`

`[]`

Explicit list of disabled dates (e.g. holidays). Pre-compute for complex rules.

### Model

The calendar state managed as a Submodel field in your parent Model.

Name

Type

Default

Description

`id`

`string`

—

The calendar instance ID.

`today`

`CalendarDate`

—

Cached "today" used for the data-today highlight and as the fallback focus target.

`viewYear`

`number`

—

The year currently rendered in the grid.

`viewMonth`

`number`

—

The month (1-12) currently rendered in the grid.

`maybeFocusedDate`

`Option<CalendarDate>`

—

The keyboard cursor position, referenced by aria-activedescendant on the grid.

`isGridFocused`

`boolean`

—

Whether the grid container has DOM focus. Used to apply focused styling only when visually appropriate.

`locale`

`LocaleConfig`

—

The locale for month/day names and first day of the week.

`maybeMinDate`

`Option<CalendarDate>`

—

Lower bound for selectable dates.

`maybeMaxDate`

`Option<CalendarDate>`

—

Upper bound for selectable dates.

`disabledDaysOfWeek`

`ReadonlyArray<DayOfWeek>`

—

Days of the week disabled across every month.

`disabledDates`

`ReadonlyArray<CalendarDate>`

—

Explicit dates marked as disabled.

`viewMode`

`'Days' | 'Months' | 'Years'`

`'Days'`

Which grid the calendar is currently showing. Drives the

`_tag`

of the CalendarAttributes passed to

`toView`

.

### ViewConfig

Configuration object passed to `Calendar.view()`.

Name

Type

Default

Description

`model`

`Calendar.Model`

—

The calendar state from your parent Model.

`maybeSelectedDate`

`Option<CalendarDate>`

—

The parent-owned selected date. The calendar reads it to render the selected-day marker; it does not store the selection itself. Fold the SelectedDate OutMessage into this field and pass it back on every render.

`toParentMessage`

`(childMessage: Calendar.Message) => ParentMessage`

—

Wraps Calendar Messages in your parent Message type for Submodel delegation (navigation, keyboard, picker-mode transitions).

`toView`

`(attributes: CalendarAttributes) => Html`

—

Callback that receives a discriminated CalendarAttributes whose variant matches the calendar viewMode (Days, Months, or Years). Pattern-match on _tag to render each grid.

`previousMonthLabel`

`string`

`'Previous month'`

Accessible label for the previous-month navigation button (Days mode).

`nextMonthLabel`

`string`

`'Next month'`

Accessible label for the next-month navigation button (Days mode).

`previousYearsPageLabel`

`string`

`'Previous 12 years'`

Accessible label for the previous-page button in the years grid.

`nextYearsPageLabel`

`string`

`'Next 12 years'`

Accessible label for the next-page button in the years grid.

`daysHeadingButtonLabel`

`string`

`'Switch to month picker'`

Accessible label for the heading button in Days mode. Clicked to drill into the months grid.

`monthsHeadingButtonLabel`

`string`

`'Switch to year picker'`

Accessible label for the heading button in Months mode. Clicked to drill into the years grid.

### CalendarAttributes

Attribute groups and derived data provided to the `toView` callback.

Name

Type

Default

Description

`_tag`

`'Days' | 'Months' | 'Years'`

—

Discriminator matching model.viewMode. Use M.tagsExhaustive to render each variant. The fields below describe the union of variants. Only the fields documented for the current _tag are present.

`root`

`ReadonlyArray<Attribute<Message>>`

—

(All modes.) Spread onto the outermost wrapper. Includes the root id.

`grid`

`ReadonlyArray<Attribute<Message>>`

—

(All modes.) Spread onto the grid container. Includes role="grid", tabindex, aria-label, aria-activedescendant, and keyboard/focus handlers.

`heading`

`{ id: string; text: string }`

—

(All modes.) Heading id and text. In Days mode the text is "September 2019"; in Months mode "2019"; in Years mode "2016–2027" (the visible window).

`headingButton`

`ReadonlyArray<Attribute<Message>>`

—

(Days, Months only.) Spread onto a

`<button>`

wrapping heading.text. Clicking dispatches ClickedHeading and drills one level deeper. Years mode is terminal and omits this field.

`previousMonthButton / nextMonthButton`

`ReadonlyArray<Attribute<Message>>`

—

(Days only.) Prev/next month navigation. Click handlers dispatch ClickedPreviousMonthButton / ClickedNextMonthButton.

`previousPageButton / nextPageButton`

`ReadonlyArray<Attribute<Message>>`

—

(Years only.) Page through 12-year windows. Click handlers dispatch PagedYears with direction -1 or 1.

`headerRow / columnHeaders`

`ReadonlyArray<Attribute<Message>> + ReadonlyArray<ColumnHeader<Message>>`

—

(Days only.) Row attributes (role="row") and seven column headers (role="columnheader") in locale-aware order.

`weeks`

`ReadonlyArray<Week<Message>>`

—

(Days only.) Six week rows. Each Week carries its own row attributes (role="row", aria-rowindex) and seven DayCells. DayCells carry cellAttributes (role="gridcell", aria-colindex), buttonAttributes (type="button", aria-label, click), the day label string, and state flags (isToday, isSelected, isFocused, isInViewMonth, isDisabled).

`cells`

`ReadonlyArray<MonthCell<Message>> | ReadonlyArray<YearCell<Message>>`

—

(Months, Years.) Twelve cells. In Months mode each cell carries the month number (1-12), the full localized name (label, e.g. "September"), and the localized abbreviation (shortLabel, e.g. "Sep"). Render whichever fits, never substring label to abbreviate. In Years mode each cell carries a year from the current 12-year window. Both expose cellAttributes (role="gridcell", aria-selected), buttonAttributes (click dispatches SelectedMonth/SelectedYear), and state flags (isSelected, isFocused, isCurrentMonth/isCurrentYear, isDisabled).

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Parents pattern-match on the OutMessage in their own update handler.

Name

Type

Default

Description

`SelectedDate`

`{ date: CalendarDate }`

—

Emitted when the user commits a date (click / Enter / Space). Pattern-match the third tuple element of Calendar.update in your GotCalendarMessage handler to lift the date into domain state.

`ChangedViewMonth`

`{ year: number; month: number }`

—

Emitted when navigation changes the visible month (prev/next buttons, heading-drill selection of a different month, arrow keys crossing a month boundary, or a commit that crosses a month). Useful for inline-calendar consumers loading month-scoped data like holidays or availability.

### Programmatic Helpers

Helpers you call from your own update handlers to drive the calendar imperatively: writing back the selection in controlled mode, focusing the grid, or updating constraints when they derive from other Model state.

The four `reflect*` helpers are how you implement cross-field date validation. Constraints are set at init time and updated via these helpers. They do not live on ViewConfig, because the update function needs them for keyboard-navigation disabled-skipping and commit-time validation. For an end date that must be on or after a start date, call `reflectMinDate(endCalendar, maybeStartDate)` in the handler that processes the start date change, where `maybeStartDate` is the parent-owned start-date field.

Name

Type

Default

Description

`selectDate`

`(model: Model, date: CalendarDate) => [Model, Commands, Option<OutMessage>]`

—

Commits the given date and moves the cursor onto it, emitting SelectedDate. Use for a programmatic selection that should behave like a user pick. To move the view to a date without selecting it (opening onto an externally-sourced value), use focusDate.

`focusDate`

`(model: Model, date: CalendarDate) => Model`

—

Moves the view and cursor to a date without changing the selection (which the parent owns). Use it to navigate to a known date, for example when the parent sets its value externally (a URL, a saved draft) so opening the calendar shows that month. Returns the model directly: no Command, no OutMessage.

`FocusGrid`

`(args: { id: string }) => Command`

—

A Command constructor, dispatched as FocusGrid({ id }), that focuses the calendar grid container. Parent components like DatePicker dispatch it to hand focus to the grid's keyboard layer after opening.

`dropToDays`

`(model: Model) => Model`

—

Returns the calendar to Days mode regardless of current depth (Days, Months, or Years). Useful for standalone consumers that want to wire their own back-out gesture; popovered consumers like DatePicker call this internally on open and close so the picker always reopens at the day grid.

`reflectMinDate`

`(model: Model, maybeMinDate: Option<CalendarDate>) => Model`

—

Reflects the minimum selectable date onto the model without emitting an OutMessage. Pass Option.none() to remove the minimum. Use for cross-field validation when the minimum derives from other Model state. Does not reconcile the current selection if it falls below the new minimum.

`reflectMaxDate`

`(model: Model, maybeMaxDate: Option<CalendarDate>) => Model`

—

Reflects the maximum selectable date onto the model without emitting an OutMessage. Pass Option.none() to remove the maximum. Does not reconcile the current selection.

`reflectDisabledDates`

`(model: Model, disabledDates: ReadonlyArray<CalendarDate>) => Model`

—

Reflects the list of individually-disabled dates (e.g. holidays) onto the model. Pass an empty array to clear.

`reflectDisabledDaysOfWeek`

`(model: Model, disabledDaysOfWeek: ReadonlyArray<DayOfWeek>) => Model`

—

Reflects the list of disabled days of the week (e.g. ["Saturday", "Sunday"]) onto the model. Pass an empty array to clear.
