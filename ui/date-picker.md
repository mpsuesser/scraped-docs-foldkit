---
url: https://foldkit.dev/ui/date-picker
title: "Date Picker"
description: "Accessible date picker that wraps Calendar in a Popover. Focus choreography, click-outside dismissal, and hidden form input for native form submission."
access_date: 2026-08-03T19:09:41.518Z
current_date: 2026-08-03T19:09:41.518Z
---

# Date Picker

## Overview

An accessible date picker that wraps `Calendar` in a `Popover`. Consumers provide the trigger button face and the calendar grid layout. DatePicker handles focus choreography (opening focuses the grid, closing returns focus to the trigger), open/close state, and an optional hidden form input for native form submission.

DatePicker uses the Submodel pattern: initialize with `DatePicker.init()`, store the Model in your parent, delegate Messages via `DatePicker.update()`, and render with `DatePicker.view()`. The update function returns `[Model, Commands, Option<OutMessage>]`. The [OutMessage](https://foldkit.dev/core/submodel#surfacing-facts) carries `SelectedDate({ date })` when the user commits a date, `ClearedDate` when the user clears it, and `ChangedViewMonth` when navigation shifts the visible month. The parent owns the selected date: store it in your Model, pass it back as `maybeSelectedDate`, and fold `SelectedDate` and `ClearedDate` into that field. Pattern-match the third tuple element of `DatePicker.update` in your `GotDatePickerMessage` handler to react. For programmatic control in update functions, use `DatePicker.open(model)` and `DatePicker.close(model)` which return `[Model, Commands]` directly.

The calendar heading inside the popover is a button: clicking it switches the day grid into a 3x4 months grid; clicking the year heading from there switches into a paged 3x4 years grid. Selecting a year drills back to the months grid for that year; selecting a month drills back to the days grid for that month. Re-opening the popover always shows the day grid.

See it in an app

Check out how DatePicker is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/datePicker.ts).

## Examples

A date picker constrained to a one-year window around today via `minDate` and `maxDate`. Click the trigger to open, pick a date, click the heading to drill into a months grid (and again to drill into a years grid), or navigate with the full WAI-ARIA grid keyboard pattern. Press Enter to commit, Escape to dismiss.

Due date

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Effect, Match as M, Option } from 'effect'
import { Calendar, Command } from 'foldkit'
import type { HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { DatePicker } from '@foldkit/ui'

// Add a field to your Model for the DatePicker Submodel, plus a field the
// parent owns for the selected date. The picker no longer stores the
// selection; the parent holds it and passes it back in as `maybeSelectedDate`.
const Model = S.Struct({
  datePickerDemo: DatePicker.Model,
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

// In your init function, pass the flags-resolved today into DatePicker.init.
// Optional: constrain the selectable range with minDate / maxDate.
const init = (flags: Flags) => [
  {
    datePickerDemo: DatePicker.init({
      id: 'date-picker-demo',
      today: flags.today,
      minDate: flags.today,
      maxDate: Calendar.addMonths(flags.today, 3),
    }),
    maybeSelectedDate: Option.none(),
    // ...your other fields
  },
  [],
]

// Embed the DatePicker Message in your parent Message. DatePicker handles
// Calendar + Popover routing internally. You only need one wrapper:
const GotDatePickerMessage = m('GotDatePickerMessage', {
  message: DatePicker.Message,
})

// Inside your update function's M.tagsExhaustive({...}), delegate
// navigation, focus, and popover messages to DatePicker.update. The
// OutMessage's `SelectedDate` carries the committed date. The popover
// has already closed by the time it fires; lift the date into your
// domain state and pass it back as `maybeSelectedDate`. `ClearedDate`
// fires when the user clears the selection. `ChangedViewMonth` fires when
// calendar navigation shifts the visible month without selecting a date.
GotDatePickerMessage: ({ message }) => {
  const [nextDatePicker, commands, maybeOutMessage] = DatePicker.update(
    model.datePickerDemo,
    message,
  )
  const mappedCommands = Command.mapMessages(commands, message =>
    GotDatePickerMessage({ message }),
  )

  return Option.match(maybeOutMessage, {
    onNone: () => [
      evo(model, { datePickerDemo: () => nextDatePicker }),
      mappedCommands,
    ],
    onSome: M.type<DatePicker.OutMessage>().pipe(
      M.tagsExhaustive({
        SelectedDate: ({ date }) => [
          // The child has emitted `SelectedDate`. The body commits
          // the child's next state as usual. This is where the parent
          // lifts the committed date into its own field, which is then
          // passed back to the picker as `maybeSelectedDate`, so the
          // parent stays the single source of truth for the selection.
          evo(model, {
            datePickerDemo: () => nextDatePicker,
            maybeSelectedDate: () => Option.some(date),
          }),
          mappedCommands,
        ],
        ClearedDate: () => [
          // The user cleared the selection. Reset the parent's field.
          evo(model, {
            datePickerDemo: () => nextDatePicker,
            maybeSelectedDate: () => Option.none(),
          }),
          mappedCommands,
        ],
        ChangedViewMonth: () => [
          // The child has emitted `ChangedViewMonth`. The body commits
          // the child's next state as usual. In this arm the parent
          // can also update its own state or dispatch its own
          // Commands, for example prefetch month data, fire analytics,
          // or trigger a downstream Command.
          evo(model, { datePickerDemo: () => nextDatePicker }),
          mappedCommands,
        ],
      }),
    ),
  })
}

// Inside your view function, embed the DatePicker via h.submodel. The
// `toCalendarView` callback receives a discriminated `CalendarAttributes`
// whose variant matches the calendar's current `viewMode`. Pattern-match
// on `_tag` to render the day grid, the months grid, or the years grid.
//
// The trigger is a form field, so give it an accessible name. Pass
// `ariaLabelledBy` with the id of a visible label element, and render that
// label targeting the trigger id with
// `DatePicker.triggerId('date-picker-demo')` for a native `<label for>`. The
// attribute is only emitted when provided, so the trigger never carries a
// dangling `aria-labelledby`.
const view = (model: Model, h: HtmlBuilder<Message>) => {
  const labelId = 'date-picker-label'

  return h.div(
    [h.Class('flex flex-col gap-1.5')],
    [
      h.label(
        [h.Id(labelId), h.For(DatePicker.triggerId('date-picker-demo'))],
        ['Date'],
      ),
      h.submodel({
        slotId: 'date-picker-demo',
        model: model.datePickerDemo,
        view: DatePicker.view,
        viewInputs: {
          ariaLabelledBy: labelId,
          anchor: { placement: 'bottom-start', gap: 4, padding: 8 },
          // The parent-owned selection. The trigger content, the calendar's
          // selected-day marker, and the hidden form input all derive from it.
          maybeSelectedDate: model.maybeSelectedDate,
          triggerContent: maybeDate =>
            Option.match(maybeDate, {
              onNone: () => h.span([], ['Pick a date']),
              onSome: date =>
                h.span([], [`${date.year}-${date.month}-${date.day}`]),
            }),
          toCalendarView: attributes =>
            M.value(attributes).pipe(
              M.tagsExhaustive({
                Days: days =>
                  h.div(
                    [...days.root, h.Class('flex flex-col gap-3 p-4')],
                    [
                      h.div(
                        [h.Class('flex items-center justify-between')],
                        [
                          h.button(
                            [
                              ...days.previousMonthButton,
                              h.Class('rounded px-2'),
                            ],
                            ['‹'],
                          ),
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
                        [
                          ...days.grid,
                          h.Class('flex flex-col gap-1 outline-none'),
                        ],
                        [
                          h.div(
                            [
                              ...days.headerRow,
                              h.Class('grid grid-cols-7 gap-1'),
                            ],
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
                    [...months.root, h.Class('flex flex-col gap-3 p-4')],
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
                    [...years.root, h.Class('flex flex-col gap-3 p-4')],
                    [
                      h.div(
                        [h.Class('flex items-center justify-between')],
                        [
                          h.button(
                            [
                              ...years.previousPageButton,
                              h.Class('rounded px-2'),
                            ],
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
          // Optional: enable hidden form input for native <form> submission:
          name: 'appointment-date',
        },
        toParentMessage: message => GotDatePickerMessage({ message }),
      }),
    ],
  )
}
```

## Styling

DatePicker is headless. You control the trigger button via `triggerContent` and `triggerClassName`, the popover panel via `panelClassName`, and the calendar grid via the `toCalendarView` callback. Data attributes on day cells let you style state variants with CSS selectors like `group-data-[selected]:` and `group-data-[disabled]:`.

Attribute

Condition

`data-today`

Present on the cell representing "today". The day cell in Days mode, the current month cell in Months mode, the current year cell in Years mode.

`data-selected`

Present on the calendar's currently-centered cell. The selected date in Days mode, the centered month (viewMonth) in Months mode, the centered year (viewYear) in Years mode.

`data-focused`

Present on the cell at the keyboard cursor position while the grid has DOM focus.

`data-outside-month`

(Days mode only.) Present on cells that fall outside the currently-viewed month (leading/trailing grid rows).

`data-disabled`

Present on cells disabled by min/max, disabledDaysOfWeek, or disabledDates.

`data-open`

Present on the trigger button and wrapper while the popover is open.

`data-placement`

Present on the calendar panel, set to the side it currently sits on: top, right, bottom, or left. Fixed to the first resolved side when isPlacementLocked is true.

## Keyboard Interaction

The trigger button opens the popover on Enter, Space, or ArrowDown. Inside the popover, the calendar grid handles the full WAI-ARIA grid keyboard pattern. Escape closes the popover from both the trigger and the grid.

Key

Description

`Enter / Space / ArrowDown`

Open the popover when the trigger button is focused.

`Escape`

Close the popover from the trigger button or from inside the calendar grid.

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

Commit the focus cursor. Days: select the date and close the popover. Months: jump the calendar to that month and drill back to Days. Years: jump to that year and drill back to Months.

## Accessibility

The trigger button uses `aria-expanded` and `aria-controls` to announce the popover relationship. Inside the popover, the calendar grid renders with `role="grid"` and an explicit `aria-label` that leads with a non-numeric word ("Calendar, April 2026") so VoiceOver does not pattern-match the grid's row position into a date literal. `aria-activedescendant` tracks the keyboard cursor; rows carry `role="row"` with `aria-rowindex`; cells carry `role="gridcell"`, `aria-colindex`, and `aria-selected` on the chosen date. Day buttons carry full accessible names via `aria-label` and disabled days get `aria-disabled="true"`. When a hidden form input is enabled via the `name` prop, the selected date is encoded as an ISO string (`YYYY-MM-DD`) for native form submission.

Give the trigger an accessible name. For a visible label, wire a native `<label for>` that targets the trigger id with `DatePicker.triggerId(id)` rather than hardcoding the `-popover-button` convention. The `for` association makes the trigger properly labeled: assistive technology announces it by the visible label text, and clicking the label opens the date picker. That is why it is the recommended pattern.

Two ViewConfig fields cover the cases a `<label for>` does not. Pass `ariaLabel` for an icon-only trigger with no visible label, or `ariaLabelledBy` when the element that names the trigger is not a `<label>` you can point `for` at.

## API Reference

### InitConfig

Configuration object passed to `DatePicker.init()`. Calendar constraints (min/max, disabled dates) are forwarded to the embedded Calendar submodel.

Name

Type

Default

Description

`id`

`string`

—

Unique ID for the date picker instance.

`today`

`CalendarDate`

—

The current calendar date. Typically fetched at the app boundary via Calendar.today.local and threaded through flags.

`initialViewDate`

`CalendarDate`

—

Seeds the month the calendar opens onto. When set, the view starts on the month containing this date. The parent owns the selection itself; pass its current value here to open onto that month.

`isAnimated`

`boolean`

`false`

Enables animation coordination on the popover panel (enter/leave animations).

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

The DatePicker Model. Stored on your parent Model and threaded through `DatePicker.update()` and `DatePicker.view()`.

Name

Type

Default

Description

`id`

`string`

—

The date picker instance ID.

`calendar`

`Calendar.Model`

—

The embedded Calendar submodel. Forwards navigation, focus, locale, and disabled-cell state. The picker delegates Calendar messages and resets the calendar to Days mode every time the popover opens or closes.

`popover`

`Popover.Model`

—

The embedded Popover submodel. Tracks open/close state, animation phase, and focus choreography (opening focuses the calendar grid, closing returns focus to the trigger).

### ViewConfig

Configuration object passed to `DatePicker.view()`.

Name

Type

Default

Description

`model`

`DatePicker.Model`

—

The date picker state from your parent Model.

`maybeSelectedDate`

`Option<CalendarDate>`

—

The parent-owned selected date. Passed through to the calendar (selected-day marker), the trigger content, and the hidden form input. The picker does not store the selection itself; fold the SelectedDate and ClearedDate OutMessages into this field and pass it back on every render.

`toParentMessage`

`(message: DatePicker.Message) => ParentMessage`

—

Wraps DatePicker Messages in your parent Message type for Submodel delegation.

`anchor`

`AnchorConfig`

—

Popover positioning config (placement, gap, offset, padding, isPlacementLocked, and portal). Controls where the calendar panel floats relative to the trigger. Portaled to the document body by default; pass portal: false to keep the panel inside its wrapper.

`triggerContent`

`(maybeDate: Option<CalendarDate>) => Html`

—

Renders the trigger button face. Receives the current selection so you can show the formatted date or a placeholder.

`toCalendarView`

`(attributes: CalendarAttributes) => Html`

—

Renders the calendar grid layout inside the popover panel. Same callback shape as Calendar.view toView. Lay out the attribute groups (for example grid, header, weeks, or cells) however you like.

`isDisabled`

`boolean`

`false`

Disables the trigger button, preventing the popover from opening.

`name`

`string`

—

When provided, renders a hidden

`<input>`

with this name and the selected date encoded as an ISO string (YYYY-MM-DD) for native form submission.

`triggerClassName / triggerAttributes`

`string / ReadonlyArray<Attribute<Message>>`

—

Class name and additional attributes spread onto the trigger button.

`ariaLabel`

`string`

—

Accessible name for the trigger button. Use for an icon-only trigger with no visible label. Applied as aria-label, and takes precedence over ariaLabelledBy.

`ariaLabelledBy`

`string`

—

Id of an external element that labels the trigger button, applied as aria-labelledby. Pair with a visible label element.

`panelClassName / panelAttributes`

`string / ReadonlyArray<Attribute<Message>>`

—

Class name and additional attributes spread onto the popover panel.

`backdropClassName / backdropAttributes`

`string / ReadonlyArray<Attribute<Message>>`

—

Class name and additional attributes spread onto the click-outside backdrop.

### CalendarAttributes

The discriminated union passed to `toCalendarView`. Pattern-match on `_tag` (`'Days' | 'Months' | 'Years'`) with `M.tagsExhaustive` to render each grid. Each variant exposes a different shape: Days carries weeks plus a headingButton; Months carries 12 month cells plus a headingButton; Years carries 12 year cells plus prev/next page buttons. See [the Calendar page's CalendarAttributes section](https://foldkit.dev/ui/calendar) for the full prop table. The type is the same.

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Pattern-match on the OutMessage in your update handler.

Name

Type

Default

Description

`SelectedDate`

`{ date: CalendarDate }`

—

Emitted when the user commits a date (click / Enter / Space). Pattern-match the third tuple element of DatePicker.update in your GotDatePickerMessage handler to lift the date into domain state.

`ClearedDate`

`{}`

—

Emitted when the user clears the selected date (via Cleared or DatePicker.clear). The popover stays open. Fold it into the parent-owned selected-date field by setting it to Option.none().

`ChangedViewMonth`

`{ year: number; month: number }`

—

Emitted when navigation changes the visible month inside the calendar grid.

### Programmatic Helpers

Helpers you call from your own update handlers to drive the date picker imperatively: for writing back the selection in controlled mode, opening/closing on domain events, or updating constraints when they derive from other Model state.

The four `reflect*` helpers are how you implement cross-field date validation. Constraints are set at init time and updated via these helpers. They do not live on ViewConfig, because the update function needs them for keyboard-navigation disabled-skipping and commit-time validation. For an end date that must be on or after a start date, call `reflectMinDate(endDatePicker, maybeStartDate)` in the handler that processes the start date change, where `endDatePicker` is the end date picker's own `DatePicker.Model` and `maybeStartDate` is the parent-owned start-date field.

Name

Type

Default

Description

`selectDate`

`(model: Model, date: CalendarDate) => [Model, Commands, Option<OutMessage>]`

—

Commits the given date and closes the popover, emitting SelectedDate. Use for a programmatic selection equivalent to a user pick. To move the embedded calendar onto a date without selecting it (opening onto an externally-sourced value), use focusDate.

`focusDate`

`(model: Model, date: CalendarDate) => Model`

—

Moves the embedded calendar view and cursor to a date without changing the selection (which the parent owns). Use it to navigate the picker onto a known date, for example after the parent sets its value externally (a URL, a saved draft) so opening the picker shows that month. Returns the model directly: no Command, no OutMessage.

`clear`

`(model: Model) => [Model, Commands, Option<OutMessage>]`

—

Clears the selected date, emitting ClearedDate so the parent resets its own field. Does not close the popover.

`open`

`(model: Model) => [Model, Commands]`

—

Programmatically opens the popover. Use from domain-event handlers when the date picker should open in response to something other than a trigger click.

`close`

`(model: Model) => [Model, Commands]`

—

Programmatically closes the popover.

`reflectMinDate`

`(model: Model, maybeMinDate: Option<CalendarDate>) => Model`

—

Updates the minimum selectable date. Pass Option.none() to remove the minimum. Use for cross-field validation, e.g. an end date picker whose minimum tracks a start date picker's selection. Does not reconcile the current selection if it falls below the new minimum.

`reflectMaxDate`

`(model: Model, maybeMaxDate: Option<CalendarDate>) => Model`

—

Updates the maximum selectable date. Pass Option.none() to remove the maximum. Does not reconcile the current selection.

`reflectDisabledDates`

`(model: Model, disabledDates: ReadonlyArray<CalendarDate>) => Model`

—

Replaces the list of individually-disabled dates (e.g. holidays). Pass an empty array to clear.

`reflectDisabledDaysOfWeek`

`(model: Model, disabledDaysOfWeek: ReadonlyArray<DayOfWeek>) => Model`

—

Replaces the list of disabled days of the week (e.g. ["Saturday", "Sunday"]). Pass an empty array to clear.
