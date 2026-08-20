---
url: https://foldkit.dev/ui/date-picker
title: "Date Picker"
description: "Accessible date picker that wraps Calendar in a Popover. Focus choreography, click-outside dismissal, and hidden form input for native form submission."
access_date: 2026-08-20T02:21:49.544Z
current_date: 2026-08-20T02:21:49.544Z
---

## Overview

An accessible date picker that wraps `Calendar` in a `Popover`. Consumers provide the trigger button face and the calendar grid layout. DatePicker handles focus choreography (opening focuses the grid, closing returns focus to the trigger), open/close state, and an optional hidden form input for native form submission.

DatePicker uses the Submodel pattern: initialize with `DatePicker.init()`, store the Model in your parent, wire Messages through [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child), and render with `DatePicker.view()`. The update function returns `[Model, Commands, Option<OutMessage>]`. The [OutMessage](https://foldkit.dev/core/submodel#surfacing-facts) carries `SelectedDate({ date })` when the user commits a date, `ClearedDate` when the user clears it, and `ChangedViewMonth` when navigation shifts the visible month. The parent owns the selected date: store it in your Model, pass it back as `maybeSelectedDate`, and fold `SelectedDate` and `ClearedDate` into that field from the fold's `foldOutMessage`. For programmatic control in update functions, use `DatePicker.open(model)` and `DatePicker.close(model)` which return `[Model, Commands]` directly.

The calendar heading inside the popover is a button: clicking it switches the day grid into a 3x4 months grid; clicking the year heading from there switches into a paged 3x4 years grid. Selecting a year drills back to the months grid for that year; selecting a month drills back to the days grid for that month. Re-opening the popover always shows the day grid.

See it in an app

Check out how DatePicker is wired up in a [real Foldkit app](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/datePicker.ts).

## Examples

A date picker constrained to a one-year window around today via `minDate` and `maxDate`. Click the trigger to open, pick a date, click the heading to drill into a months grid (and again to drill into a years grid), or navigate with the full WAI-ARIA grid keyboard pattern. Press Enter to commit, Escape to dismiss.

```
// Pseudocode walkthrough of the Foldkit integration points. Each labeled
// block below is an excerpt. Fit them into your own Model, init, Message,
// update, and view definitions.
import { Effect, Match as M, Option } from 'effect'
import { Calendar, Update } from 'foldkit'
import type { ChildAttribute, Html, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { DatePicker, Calendar as UiCalendar } from '@foldkit/ui'

// Add a field to your Model for the DatePicker Submodel, plus a field the
// parent owns for the selected date. The picker no longer stores the
// selection; the parent holds it and passes it back in as \`maybeSelectedDate\`.
const Model = S.Struct({
  datePickerDemo: DatePicker.Model,
  maybeSelectedDate: S.Option(Calendar.CalendarDate),
  // ...your other fields
})

// Fetch \`today\` once at the app boundary via flags so init stays pure:
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

// At module scope, fold the OutMessage into your own Model. \`SelectedDate\`
// carries the committed date. The popover has already closed by the time it
// fires; lift the date into your domain state and pass it back as
// \`maybeSelectedDate\`. \`ClearedDate\` fires when the user clears the selection.
// \`ChangedViewMonth\` fires when calendar navigation shifts the visible month
// without selecting a date. Each arm returns an Update.Step over the parent
// Model, which already has the next DatePicker Model written back:
const foldDatePickerOutMessage = M.type<DatePicker.OutMessage>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    // The child has emitted \`SelectedDate\`. This is where the parent lifts
    // the committed date into its own field, which is then passed back to
    // the picker as \`maybeSelectedDate\`, so the parent stays the single
    // source of truth for the selection.
    SelectedDate:
      ({ date }) =>
      model => [evo(model, { maybeSelectedDate: () => Option.some(date) }), []],
    // The user cleared the selection. Reset the parent's field.
    ClearedDate: () => model => [
      evo(model, { maybeSelectedDate: () => Option.none() }),
      [],
    ],
    // The child has emitted \`ChangedViewMonth\`. In this arm the parent can
    // update its own state or dispatch its own Commands, for example
    // prefetch month data, fire analytics, or trigger a downstream Command.
    ChangedViewMonth: () => model => [model, []],
  }),
)

// Update.foldChild wires the child into the parent: it delegates navigation,
// focus, and popover messages to DatePicker.update, writes the next DatePicker
// Model back, maps the Submodel's Commands into your Message type, and hands
// any OutMessage to foldOutMessage.
const foldDatePicker = Update.foldChild({
  update: DatePicker.update,
  read: (model: Model) => Option.some(model.datePickerDemo),
  write: (model, nextDatePickerDemo) =>
    evo(model, { datePickerDemo: () => nextDatePickerDemo }),
  toParentMessage: message => GotDatePickerMessage({ message }),
  foldOutMessage: foldDatePickerOutMessage,
})

// Inside your update function's M.tagsExhaustive({...}), call the fold:
GotDatePickerMessage: ({ message }) => foldDatePicker(model, message)

// Class names live at module scope, and each view mode gets its own view
// function below. The calendar panel has no border of its own: the
// DatePicker's popover panel provides it.
const panelClassName = 'flex flex-col gap-3 p-4'

const headerClassName = 'flex items-center justify-between'

const navButtonClassName = 'rounded px-2'

const headingButtonClassName =
  'inline-flex items-center gap-2 rounded px-2 text-sm font-semibold'

const headingTextClassName = 'text-sm font-semibold'

const gridClassName = 'flex flex-col gap-1 outline-none'

const rowClassName = 'grid grid-cols-7 gap-1'

const columnHeaderClassName = 'text-center text-xs uppercase'

// \`group\` lets the button inside style itself from the cell's data attributes.
const cellClassName = 'group flex items-center justify-center'

const dayButtonClassName =
  'h-9 w-9 rounded-full text-sm group-data-[today]:ring-1 group-data-[selected]:bg-accent-600 group-data-[selected]:text-white group-data-[outside-month]:text-gray-400 group-data-[disabled]:opacity-40'

const monthYearGridClassName = 'grid grid-cols-3 gap-1 outline-none'

const monthYearButtonClassName =
  'h-12 w-full rounded-md text-sm group-data-[selected]:bg-accent-600 group-data-[selected]:text-white group-data-[disabled]:opacity-40'

// \`ChildAttribute\` is the type of the attributes a Submodel publishes to its
// consumer. Spread them onto whichever element you want to carry them.
const navButton = (
  attributes: ReadonlyArray<ChildAttribute>,
  label: string,
  h: HtmlBuilder<Message>,
): Html => h.button([...attributes, h.Class(navButtonClassName)], [label])

// The heading is a button: clicking it drills one level deeper (Days into
// Months, Months into Years). Pair the text with a chevron so the button reads
// as interactive at rest.
const headingButton = (
  heading: UiCalendar.DaysModeAttributes['heading'],
  attributes: ReadonlyArray<ChildAttribute>,
  h: HtmlBuilder<Message>,
): Html =>
  h.button(
    [h.Id(heading.id), ...attributes, h.Class(headingButtonClassName)],
    [heading.text, ' ▾'],
  )

// Each Week carries its own row attributes and seven day cells, and each cell
// is a gridcell wrapping a button.
const weekRow = (week: UiCalendar.Week, h: HtmlBuilder<Message>): Html =>
  h.div(
    [...week.attributes, h.Class(rowClassName)],
    week.cells.map(cell =>
      h.div(
        [...cell.cellAttributes, h.Class(cellClassName)],
        [
          h.button(
            [...cell.buttonAttributes, h.Class(dayButtonClassName)],
            [cell.label],
          ),
        ],
      ),
    ),
  )

// One view function per view mode. Each receives the attribute group for its
// own grid, so the fields it reads are exactly the fields that exist.
const daysView = (
  days: UiCalendar.DaysModeAttributes,
  h: HtmlBuilder<Message>,
): Html =>
  h.div(
    [...days.root, h.Class(panelClassName)],
    [
      h.div(
        [h.Class(headerClassName)],
        [
          navButton(days.previousMonthButton, '‹', h),
          headingButton(days.heading, days.headingButton, h),
          navButton(days.nextMonthButton, '›', h),
        ],
      ),
      h.div(
        [...days.grid, h.Class(gridClassName)],
        [
          h.div(
            [...days.headerRow, h.Class(rowClassName)],
            days.columnHeaders.map(header =>
              h.div(
                [...header.attributes, h.Class(columnHeaderClassName)],
                [header.name],
              ),
            ),
          ),
          ...days.weeks.map(week => weekRow(week, h)),
        ],
      ),
    ],
  )

// The months grid renders 12 cells (one per month). Clicking the heading again
// drills further into the years grid.
const monthsView = (
  months: UiCalendar.MonthsModeAttributes,
  h: HtmlBuilder<Message>,
): Html =>
  h.div(
    [...months.root, h.Class(panelClassName)],
    [
      h.div(
        [h.Class('flex items-center justify-center')],
        [headingButton(months.heading, months.headingButton, h)],
      ),
      h.div(
        [...months.grid, h.Class(monthYearGridClassName)],
        months.cells.map(cell =>
          h.div(
            [...cell.cellAttributes, h.Class(cellClassName)],
            [
              h.button(
                [...cell.buttonAttributes, h.Class(monthYearButtonClassName)],
                [cell.shortLabel],
              ),
            ],
          ),
        ),
      ),
    ],
  )

// The years grid renders 12 cells (one paged window). Prev/next page through
// 12-year windows; clicking a year drills back to the months grid for that
// year. Years is terminal, so its heading is text rather than a button.
const yearsView = (
  years: UiCalendar.YearsModeAttributes,
  h: HtmlBuilder<Message>,
): Html =>
  h.div(
    [...years.root, h.Class(panelClassName)],
    [
      h.div(
        [h.Class(headerClassName)],
        [
          navButton(years.previousPageButton, '‹', h),
          h.h2(
            [h.Id(years.heading.id), h.Class(headingTextClassName)],
            [years.heading.text],
          ),
          navButton(years.nextPageButton, '›', h),
        ],
      ),
      h.div(
        [...years.grid, h.Class(monthYearGridClassName)],
        years.cells.map(cell =>
          h.div(
            [...cell.cellAttributes, h.Class(cellClassName)],
            [
              h.button(
                [...cell.buttonAttributes, h.Class(monthYearButtonClassName)],
                [cell.label],
              ),
            ],
          ),
        ),
      ),
    ],
  )

// Inside your view function, embed the DatePicker via h.submodel. The
// \`toCalendarView\` callback receives a discriminated \`CalendarAttributes\`
// whose variant matches the calendar's current \`viewMode\`, so the match hands
// each grid to its own view function.
//
// The trigger is a form field, so give it an accessible name. Pass
// \`ariaLabelledBy\` with the id of a visible label element, and render that
// label targeting the trigger id with
// \`DatePicker.triggerId('date-picker-demo')\` for a native \`<label for>\`. The
// attribute is only emitted when provided, so the trigger never carries a
// dangling \`aria-labelledby\`.
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
                h.span([], [\`${date.year}-${date.month}-${date.day}\`]),
            }),
          toCalendarView: M.type<UiCalendar.CalendarAttributes>().pipe(
            M.tagsExhaustive({
              Days: days => daysView(days, h),
              Months: months => monthsView(months, h),
              Years: years => yearsView(years, h),
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

| Attribute | Condition |
| --- | --- |
| `data-today` | Present on the cell representing "today". The day cell in Days mode, the current month cell in Months mode, the current year cell in Years mode. |
| `data-selected` | Present on the calendar's currently-centered cell. The selected date in Days mode, the centered month (viewMonth) in Months mode, the centered year (viewYear) in Years mode. |
| `data-focused` | Present on the cell at the keyboard cursor position while the grid has DOM focus. |
| `data-outside-month` | (Days mode only.) Present on cells that fall outside the currently-viewed month (leading/trailing grid rows). |
| `data-disabled` | Present on cells disabled by min/max, disabledDaysOfWeek, or disabledDates. |
| `data-open` | Present on the trigger button and wrapper while the popover is open. |
| `data-placement` | Present on the calendar panel, set to the side it currently sits on: top, right, bottom, or left. Fixed to the first resolved side when isPlacementLocked is true. |

## Keyboard Interaction

The trigger button opens the popover on Enter, Space, or ArrowDown. Inside the popover, the calendar grid handles the full WAI-ARIA grid keyboard pattern. Escape closes the popover from both the trigger and the grid.

| Key | Description |
| --- | --- |
| `Enter / Space / ArrowDown` | Open the popover when the trigger button is focused. |
| `Escape` | Close the popover from the trigger button or from inside the calendar grid. |
| `ArrowLeft / ArrowRight` | Move the focus cursor by one cell. Days: ±1 day. Months: ±1 month (wraps across years). Years: ±1 year (wraps across pages). |
| `ArrowUp / ArrowDown` | Move the focus cursor by one row. Days: ±1 week (7 days). Months: ±1 row (3 months). Years: ±1 row (3 years). |
| `Home / End` | (Days mode only.) Move focus to the start / end of the current week (based on locale.firstDayOfWeek). |
| `PageUp / PageDown` | Days: ±1 month. Months: ±1 year. Years: ±1 window (12 years). |
| `Shift + PageUp / Shift + PageDown` | (Days mode only.) Move focus by one year. |
| `Enter / Space` | Commit the focus cursor. Days: select the date and close the popover. Months: jump the calendar to that month and drill back to Days. Years: jump to that year and drill back to Months. |

## Accessibility

The trigger button uses `aria-expanded` and `aria-controls` to announce the popover relationship. Inside the popover, the calendar grid renders with `role="grid"` and an explicit `aria-label` that leads with a non-numeric word ("Calendar, April 2026") so VoiceOver does not pattern-match the grid's row position into a date literal. `aria-activedescendant` tracks the keyboard cursor; rows carry `role="row"` with `aria-rowindex`; cells carry `role="gridcell"`, `aria-colindex`, and `aria-selected` on the chosen date. Day buttons carry full accessible names via `aria-label` and disabled days get `aria-disabled="true"`. When a hidden form input is enabled via the `name` prop, the selected date is encoded as an ISO string (`YYYY-MM-DD`) for native form submission.

Give the trigger an accessible name. For a visible label, wire a native `<label for>` that targets the trigger id with `DatePicker.triggerId(id)` rather than hardcoding the `-popover-button` convention. The `for` association makes the trigger properly labeled: assistive technology announces it by the visible label text, and clicking the label opens the date picker. That is why it is the recommended pattern.

Two ViewConfig fields cover the cases a `<label for>` does not. Pass `ariaLabel` for an icon-only trigger with no visible label, or `ariaLabelledBy` when the element that names the trigger is not a `<label>` you can point `for` at.

## API Reference

### InitConfig

Configuration object passed to `DatePicker.init()`. Calendar constraints (min/max, disabled dates) are forwarded to the embedded Calendar submodel.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for the date picker instance. |
| `today` | `CalendarDate` | — | The current calendar date. Typically fetched at the app boundary via Calendar.today.local and threaded through flags. |
| `initialViewDate` | `CalendarDate` | — | Seeds the month the calendar opens onto. When set, the view starts on the month containing this date. The parent owns the selection itself; pass its current value here to open onto that month. |
| `isAnimated` | `boolean` | `false` | Enables animation coordination on the popover panel (enter/leave animations). |
| `locale` | `LocaleConfig` | `defaultEnglishLocale` | Month and day names plus the first day of the week. Import from foldkit/calendar. |
| `minDate` | `CalendarDate` | — | Earliest selectable date. Dates before minDate are marked disabled and skipped by keyboard navigation. |
| `maxDate` | `CalendarDate` | — | Latest selectable date. Dates after maxDate are marked disabled and skipped by keyboard navigation. |
| `disabledDaysOfWeek` | `ReadonlyArray<DayOfWeek>` | `[]` | Days of the week to disable (e.g. \["Saturday", "Sunday"\] for weekday-only selection). |
| `disabledDates` | `ReadonlyArray<CalendarDate>` | `[]` | Explicit list of disabled dates (e.g. holidays). Pre-compute for complex rules. |

### Model

The DatePicker Model. Stored on your parent Model and threaded through `DatePicker.update()` and `DatePicker.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | The date picker instance ID. |
| `calendar` | `Calendar.Model` | — | The embedded Calendar submodel. Forwards navigation, focus, locale, and disabled-cell state. The picker delegates Calendar messages and resets the calendar to Days mode every time the popover opens or closes. |
| `popover` | `Popover.Model` | — | The embedded Popover submodel. Tracks open/close state, animation phase, and focus choreography (opening focuses the calendar grid, closing returns focus to the trigger). |

### ViewConfig

Configuration object passed to `DatePicker.view()`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `model` | `DatePicker.Model` | — | The date picker state from your parent Model. |
| `maybeSelectedDate` | `Option<CalendarDate>` | — | The parent-owned selected date. Passed through to the calendar (selected-day marker), the trigger content, and the hidden form input. The picker does not store the selection itself; fold the SelectedDate and ClearedDate OutMessages into this field and pass it back on every render. |
| `toParentMessage` | `(message: DatePicker.Message) => ParentMessage` | — | Wraps DatePicker Messages in your parent Message type for Submodel delegation. |
| `anchor` | `AnchorConfig` | — | Popover positioning config (placement, gap, offset, padding, isPlacementLocked, and portal). Controls where the calendar panel floats relative to the trigger. Portaled to the document body by default; pass portal: false to keep the panel inside its wrapper. |
| `triggerContent` | `(maybeDate: Option<CalendarDate>) => Html` | — | Renders the trigger button face. Receives the current selection so you can show the formatted date or a placeholder. |
| `toCalendarView` | `(attributes: CalendarAttributes) => Html` | — | Renders the calendar grid layout inside the popover panel. Same callback shape as Calendar.view toView. Lay out the attribute groups (for example grid, header, weeks, or cells) however you like. |
| `isDisabled` | `boolean` | `false` | Disables the trigger button, preventing the popover from opening. |
| `name` | `string` | — | When provided, renders a hidden `<input>` with this name and the selected date encoded as an ISO string (YYYY-MM-DD) for native form submission. |
| `triggerClassName / triggerAttributes` | `string / ReadonlyArray<Attribute<Message>>` | — | Class name and additional attributes spread onto the trigger button. |
| `ariaLabel` | `string` | — | Accessible name for the trigger button. Use for an icon-only trigger with no visible label. Applied as aria-label, and takes precedence over ariaLabelledBy. |
| `ariaLabelledBy` | `string` | — | Id of an external element that labels the trigger button, applied as aria-labelledby. Pair with a visible label element. |
| `panelClassName / panelAttributes` | `string / ReadonlyArray<Attribute<Message>>` | — | Class name and additional attributes spread onto the popover panel. |
| `backdropClassName / backdropAttributes` | `string / ReadonlyArray<Attribute<Message>>` | — | Class name and additional attributes spread onto the click-outside backdrop. |

### CalendarAttributes

The discriminated union passed to `toCalendarView`. Pattern-match on `_tag` (`'Days' | 'Months' | 'Years'`) with `M.tagsExhaustive` to render each grid. Each variant exposes a different shape: Days carries weeks plus a headingButton; Months carries 12 month cells plus a headingButton; Years carries 12 year cells plus prev/next page buttons. See [the Calendar page's CalendarAttributes section](https://foldkit.dev/ui/calendar) for the full prop table. The type is the same.

### OutMessage

Messages emitted to the parent through the third element of `[Model, Commands, Option<OutMessage>]`. Fold the OutMessage in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) config.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `SelectedDate` | `{ date: CalendarDate }` | — | Emitted when the user commits a date (click / Enter / Space). Fold it in the `foldOutMessage` of your DatePicker fold to lift the date into domain state. |
| `ClearedDate` | `{}` | — | Emitted when the user clears the selected date (via Cleared or DatePicker.clear). The popover stays open. Fold it into the parent-owned selected-date field by setting it to Option.none(). |
| `ChangedViewMonth` | `{ year: number; month: number }` | — | Emitted when navigation changes the visible month inside the calendar grid. |

### Programmatic Helpers

Helpers you call from your own update handlers to drive the date picker imperatively: for writing back the selection in controlled mode, opening/closing on domain events, or updating constraints when they derive from other Model state.

The four `reflect*` helpers are how you implement cross-field date validation. Constraints are set at init time and updated via these helpers. They do not live on ViewConfig, because the update function needs them for keyboard-navigation disabled-skipping and commit-time validation. For an end date that must be on or after a start date, call `reflectMinDate(endDatePicker, maybeStartDate)` in the handler that processes the start date change, where `endDatePicker` is the end date picker's own `DatePicker.Model` and `maybeStartDate` is the parent-owned start-date field.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `selectDate` | `(model: Model, date: CalendarDate) => [Model, Commands, Option<OutMessage>]` | — | Commits the given date and closes the popover, emitting SelectedDate. Use for a programmatic selection equivalent to a user pick. To move the embedded calendar onto a date without selecting it (opening onto an externally-sourced value), use focusDate. |
| `focusDate` | `(model: Model, date: CalendarDate) => Model` | — | Moves the embedded calendar view and cursor to a date without changing the selection (which the parent owns). Use it to navigate the picker onto a known date, for example after the parent sets its value externally (a URL, a saved draft) so opening the picker shows that month. Returns the model directly: no Command, no OutMessage. |
| `clear` | `(model: Model) => [Model, Commands, Option<OutMessage>]` | — | Clears the selected date, emitting ClearedDate so the parent resets its own field. Does not close the popover. |
| `open` | `(model: Model) => [Model, Commands]` | — | Programmatically opens the popover. Use from domain-event handlers when the date picker should open in response to something other than a trigger click. |
| `close` | `(model: Model) => [Model, Commands]` | — | Programmatically closes the popover. |
| `reflectMinDate` | `(model: Model, maybeMinDate: Option<CalendarDate>) => Model` | — | Updates the minimum selectable date. Pass Option.none() to remove the minimum. Use for cross-field validation, e.g. an end date picker whose minimum tracks a start date picker's selection. Does not reconcile the current selection if it falls below the new minimum. |
| `reflectMaxDate` | `(model: Model, maybeMaxDate: Option<CalendarDate>) => Model` | — | Updates the maximum selectable date. Pass Option.none() to remove the maximum. Does not reconcile the current selection. |
| `reflectDisabledDates` | `(model: Model, disabledDates: ReadonlyArray<CalendarDate>) => Model` | — | Replaces the list of individually-disabled dates (e.g. holidays). Pass an empty array to clear. |
| `reflectDisabledDaysOfWeek` | `(model: Model, disabledDaysOfWeek: ReadonlyArray<DayOfWeek>) => Model` | — | Replaces the list of disabled days of the week (e.g. \["Saturday", "Sunday"\]). Pass an empty array to clear. |
