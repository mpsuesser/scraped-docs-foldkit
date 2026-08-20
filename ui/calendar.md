---
url: https://foldkit.dev/ui/calendar
title: "Calendar"
description: "Accessible inline calendar grid with 2D keyboard navigation, locale-aware headers, and min/max/disabled-date constraints."
access_date: 2026-08-20T02:21:49.544Z
current_date: 2026-08-20T02:21:49.544Z
---

## An Inline Calendar You Render

Calendar provides the state machine, ARIA wiring, and derived data for an inline calendar. You provide the markup and styling through `toView`. Use it in scheduling interfaces and event calendars, or as the calendar inside a date picker.

The Calendar moves among three grids:

- **Days:** a fixed 6×7 grid with locale-aware weekday headings.
- **Months:** 12 cells for choosing a month, arranged in three columns.
- **Years:** 12 cells for choosing a year, arranged in three columns and paged in 12-year windows.

The heading moves from Days to Months, then from Months to Years. Selecting a year returns to Months for that year. Selecting a month returns to Days for that month.

Calendar owns navigation state, including the visible month, active grid, and keyboard cursor. The parent owns the selected date. This keeps the domain value in the parent Model while Calendar handles the interaction state around it.

## Wire Calendar into a Parent

Initialize the Calendar with `Calendar.init()`, store its Model in your parent Model, and delegate its Messages with [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child). Render it through `h.submodel` with `Calendar.view`.

Pass the parent-owned selection into `viewInputs.maybeSelectedDate` on every render. When Calendar emits `SelectedDate`, fold that OutMessage into the parent's selected-date field. `Calendar.update` returns `[Model, Commands, Option<OutMessage>]`, so the same fold can handle `ChangedViewMonth` when your application needs month-scoped data.

See it in an app

See the complete Calendar integration in the [UI Showcase](https://github.com/foldkit/foldkit/blob/main/examples/ui-showcase/src/ui/view/calendar.ts).

## Try It

This Calendar highlights today and lets the parent store a selected date. Click a day, or tab into the grid and navigate with the keyboard. The snippet shows the parent Model, `Update.foldChild`, OutMessage handling, and all three view modes.

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
import { Calendar, Update } from 'foldkit'
import type { ChildAttribute, Html, HtmlBuilder } from 'foldkit/html'
import { m } from 'foldkit/message'
import { evo } from 'foldkit/struct'

import { Calendar as UiCalendar } from '@foldkit/ui'

// Add a field to your Model for the Calendar Submodel, plus a field the
// parent owns for the selected date. The calendar no longer stores the
// selection; the parent holds it and passes it back in as \`maybeSelectedDate\`.
const Model = S.Struct({
  calendarDemo: UiCalendar.Model,
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

// In your init function, pass the flags-resolved today into UiCalendar.init.
// \`initialViewDate\` seeds the month the calendar opens onto (pass your
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

// At module scope, fold the OutMessage into your own Model. When the user
// commits a date (click, Enter, or Space) it carries \`SelectedDate({ date })\`.
// \`ChangedViewMonth\` fires when navigation shifts the visible month without
// selecting a date. Each arm returns an Update.Step over the parent Model,
// which already has the next Calendar Model written back:
const foldCalendarOutMessage = M.type<UiCalendar.OutMessage>().pipe(
  M.withReturnType<Update.Step<Model, Message>>(),
  M.tagsExhaustive({
    // The child has emitted \`SelectedDate\`. This is where the parent lifts
    // the committed date into its own field. That field is then passed back
    // to the calendar as \`maybeSelectedDate\`, so the parent stays the single
    // source of truth for the selection.
    SelectedDate:
      ({ date }) =>
      model => [evo(model, { maybeSelectedDate: () => Option.some(date) }), []],
    // The child has emitted \`ChangedViewMonth\`. In this arm the parent can
    // update its own state or dispatch its own Commands, for example
    // prefetch month data, fire analytics, or trigger a downstream Command.
    ChangedViewMonth: () => model => [model, []],
  }),
)

// Update.foldChild wires the child into the parent: it delegates navigation,
// focus, and picker-mode transitions to UiCalendar.update, writes the next
// Calendar Model back, maps the Submodel's Commands into your Message type,
// and hands any OutMessage to foldOutMessage.
const foldCalendar = Update.foldChild({
  update: UiCalendar.update,
  read: (model: Model) => Option.some(model.calendarDemo),
  write: (model, nextCalendarDemo) =>
    evo(model, { calendarDemo: () => nextCalendarDemo }),
  toParentMessage: message => GotCalendarMessage({ message }),
  foldOutMessage: foldCalendarOutMessage,
})

// Inside your update function's M.tagsExhaustive({...}), call the fold:
GotCalendarMessage: ({ message }) => foldCalendar(model, message)

// Class names live at module scope, and each view mode gets its own view
// function below.
const panelClassName = 'flex flex-col gap-3 rounded-xl border p-4'

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

// Inside your view function, render the calendar. The \`toView\` callback
// receives a discriminated \`CalendarAttributes\` whose variant matches the
// calendar's current \`viewMode\`, so the match hands each grid to its own view
// function:
const view = (model: Model, h: HtmlBuilder<Message>) =>
  h.submodel({
    slotId: model.calendarDemo.id,
    model: model.calendarDemo,
    view: UiCalendar.view,
    viewInputs: {
      // The parent-owned selection. The selected-day marker derives from it.
      maybeSelectedDate: model.maybeSelectedDate,
      toView: M.type<UiCalendar.CalendarAttributes>().pipe(
        M.tagsExhaustive({
          Days: days => daysView(days, h),
          Months: months => monthsView(months, h),
          Years: years => yearsView(years, h),
        }),
      ),
    },
    toParentMessage: message => GotCalendarMessage({ message }),
  })
```

## Render and Style the Calendar

Calendar is headless. Its `toView` callback receives attribute groups for the current mode. Spread those attributes onto your elements, then add your own classes and structure.

State data attributes let styles follow Calendar state without rebuilding that logic in the view:

| Attribute | Present when |
| --- | --- |
| `data-today` | The cell represents today, today's month, or today's year. |
| `data-selected` | The cell represents the parent-owned selected date in Days mode, `viewMonth` in Months mode, or `viewYear` in Years mode. |
| `data-focused` | The cell is at the keyboard cursor while the grid has DOM focus. |
| `data-outside-month` | A Days-mode cell falls outside the visible month. |
| `data-disabled` | A day violates a date constraint, or a month or year falls entirely outside the configured minimum and maximum. |

For example: `group-data-[selected]:` can style a button from the state attributes on its containing grid cell.

## Keyboard Interaction

The grid container holds DOM focus. `aria-activedescendant` points to the cell at the keyboard cursor, so assistive technology can announce movement without moving focus among the buttons.

In Days mode, navigation clamps to `minDate` and `maxDate` and skips disabled dates. The search is bounded, so a fully disabled range terminates cleanly. Disabled month and year cells cannot be committed.

| Key | Behavior |
| --- | --- |
| `ArrowLeft` / `ArrowRight` | Move one day, month, or year, according to the current mode. |
| `ArrowUp` / `ArrowDown` | Move one row: seven days in Days mode, three months in Months mode, or three years in Years mode. |
| `Home` / `End` | In Days mode, move to the start or end of the current week using `locale.firstDayOfWeek`. |
| `PageUp` / `PageDown` | Move one month in Days mode, one year in Months mode, or one 12-year window in Years mode. |
| `Shift` + `PageUp` / `PageDown` | In Days mode, move one year. |
| `Enter` / `Space` | Select the date in Days mode, open the month in Months mode, or open the year in Years mode. |

## Accessibility

Each mode renders a grid with an accessible name and uses `aria-activedescendant` for its keyboard cursor. The Days grid is labeled with a leading word, such as `Calendar, April 2026`, so VoiceOver does not interpret its row position as a date literal.

Rows use `role="row"`. Weekday headings use `role="columnheader"`. Cells use `role="gridcell"` and expose selection state through `aria-selected`. Day buttons receive full accessible names, such as `Monday, April 13, 2026`, and disabled cells use `aria-disabled="true"`.

## API Reference

### InitConfig

Pass this configuration to `Calendar.init()`:

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | — | Unique ID for this Calendar instance. |
| `today` | `CalendarDate` | — | Current date used for highlighting and as the fallback focus target. Resolve it at the application boundary with `Calendar.today.local`. |
| `initialViewDate` | `CalendarDate` | `today` | Date whose month opens first. Pass the initial parent-owned selection to open on that value. |
| `locale` | `LocaleConfig` | `defaultEnglishLocale` | Month names, weekday names, and the first day of the week. Import the type and default from `foldkit/calendar`. |
| `minDate` | `CalendarDate` | — | Earliest selectable date. Earlier dates are disabled and skipped during Days-mode keyboard navigation. |
| `maxDate` | `CalendarDate` | — | Latest selectable date. Later dates are disabled and skipped during Days-mode keyboard navigation. |
| `disabledDaysOfWeek` | `ReadonlyArray<DayOfWeek>` | `[]` | Weekdays to disable across every month. For example: `['Saturday', 'Sunday']`. |
| `disabledDates` | `ReadonlyArray<CalendarDate>` | `[]` | Individual dates to disable. Precompute the array for more complex rules. |

### Model

Store the Calendar Model as a field in the parent Model. It contains interaction and constraint state, not the selected date.

| Name | Type | Description |
| --- | --- | --- |
| `id` | `string` | Calendar instance ID. |
| `today` | `CalendarDate` | Date used for the today marker and fallback focus. |
| `viewYear` | `number` | Year centered by the Calendar. |
| `viewMonth` | `number` | Month centered by the Calendar, from 1 through 12. |
| `viewMode` | `'Days' \| 'Months' \| 'Years'` | Grid currently displayed. |
| `maybeFocusedDate` | `Option<CalendarDate>` | Keyboard cursor used by `aria-activedescendant`. |
| `isGridFocused` | `boolean` | Whether the grid container has DOM focus. |
| `locale` | `LocaleConfig` | Month names, weekday names, and first day of the week. |
| `maybeMinDate` | `Option<CalendarDate>` | Lower selection bound. |
| `maybeMaxDate` | `Option<CalendarDate>` | Upper selection bound. |
| `disabledDaysOfWeek` | `ReadonlyArray<DayOfWeek>` | Weekdays disabled across every month. |
| `disabledDates` | `ReadonlyArray<CalendarDate>` | Individually disabled dates. |

### ViewInputs

Pass these fields under `viewInputs` when `h.submodel` renders `Calendar.view`. The surrounding `h.submodel` configuration separately receives `slotId`, `model`, `view`, and `toParentMessage`.

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `maybeSelectedDate` | `Option<CalendarDate>` | — | Parent-owned selection used to derive selected-cell state. Pass it on every render. |
| `toView` | `(attributes: CalendarAttributes) => Html` | — | Renders the current grid from the mode-specific attribute bundle. Match on `_tag` with `M.tagsExhaustive`. |
| `previousMonthLabel` | `string` | `'Previous month'` | Accessible label for the previous-month button in Days mode. |
| `nextMonthLabel` | `string` | `'Next month'` | Accessible label for the next-month button in Days mode. |
| `previousYearsPageLabel` | `string` | `'Previous 12 years'` | Accessible label for the previous-page button in Years mode. |
| `nextYearsPageLabel` | `string` | `'Next 12 years'` | Accessible label for the next-page button in Years mode. |
| `daysHeadingButtonLabel` | `string` | `'Switch to month picker'` | Accessible label for the heading button in Days mode. |
| `monthsHeadingButtonLabel` | `string` | `'Switch to year picker'` | Accessible label for the heading button in Months mode. |

### CalendarAttributes

`CalendarAttributes` is a discriminated union. Its `_tag` always matches `model.viewMode`. Every variant contains these fields:

| Name | Type | Description |
| --- | --- | --- |
| `_tag` | `'Days' \| 'Months' \| 'Years'` | Discriminator for exhaustive rendering with `M.tagsExhaustive`. |
| `root` | `ReadonlyArray<ChildAttribute>` | Attributes for the outermost Calendar element, including its ID. |
| `grid` | `ReadonlyArray<ChildAttribute>` | Grid semantics, keyboard and focus handlers, accessible name, and active-descendant state. |
| `heading` | `{ id: string; text: string }` | Heading ID and localized text for the current month, year, or 12-year window. |

The remaining fields depend on the variant.

#### Days

| Name | Type | Description |
| --- | --- | --- |
| `previousMonthButton`, `nextMonthButton` | `ReadonlyArray<ChildAttribute>` | Attributes for the month-navigation buttons. |
| `headingButton` | `ReadonlyArray<ChildAttribute>` | Attributes for the heading button that opens Months mode. |
| `headerRow` | `ReadonlyArray<ChildAttribute>` | Attributes for the weekday-header row. |
| `columnHeaders` | `ReadonlyArray<ColumnHeader>` | Seven locale-ordered weekday headings. Each has `name` and `attributes`. |
| `weeks` | `ReadonlyArray<Week>` | Six rows. Each has `attributes` and seven `DayCell` values. |

Each `DayCell` contains `date`, `label`, `cellAttributes`, `buttonAttributes`, and the state flags `isSelected`, `isFocused`, `isToday`, `isInViewMonth`, and `isDisabled`.

#### Months

| Name | Type | Description |
| --- | --- | --- |
| `headingButton` | `ReadonlyArray<ChildAttribute>` | Attributes for the heading button that opens Years mode. |
| `cells` | `ReadonlyArray<MonthCell>` | Twelve month cells with `month`, localized `label` and `shortLabel`, cell and button attributes, and state flags. |

Use `shortLabel` when space is tight. Do not derive an abbreviation by slicing `label`, which is not safe across locales.

#### Years

| Name | Type | Description |
| --- | --- | --- |
| `previousPageButton`, `nextPageButton` | `ReadonlyArray<ChildAttribute>` | Attributes for paging through 12-year windows. |
| `cells` | `ReadonlyArray<YearCell>` | Twelve year cells with `year`, `label`, cell and button attributes, and state flags. |

Month cells expose `isSelected`, `isFocused`, `isCurrentMonth`, and `isDisabled`. Year cells expose `isSelected`, `isFocused`, `isCurrentYear`, and `isDisabled`.

### OutMessage

Calendar returns an optional OutMessage as the third element of `[Model, Commands, Option<OutMessage>]`. Match on its tag in the `foldOutMessage` of your [`Update.foldChild`](https://foldkit.dev/core/submodel#fold-child) configuration.

| Name | Payload | Emitted when |
| --- | --- | --- |
| `SelectedDate` | `{ date: CalendarDate }` | The user commits a date by click, `Enter`, or `Space`. If the date is outside the visible month, Calendar also moves the view, but emits only `SelectedDate`. |
| `ChangedViewMonth` | `{ year: number; month: number }` | Navigation changes the visible month without committing a date. This includes month buttons, cross-month keyboard movement, and choosing a different month or year while drilling. |

Use `SelectedDate` to update the parent-owned selection. Use `ChangedViewMonth` when an inline Calendar needs month-scoped data such as availability, holidays, or events.

### Programmatic Helpers

Call these helpers from parent update handlers when a domain event needs to drive Calendar state.

| Name | Type | Behavior |
| --- | --- | --- |
| `selectDate` | `(model: Model, date: CalendarDate) => [Model, Commands, Option<OutMessage>]` | Moves the view and cursor to the date and emits `SelectedDate`. Fold the OutMessage to update the parent-owned selection. |
| `focusDate` | `(model: Model, date: CalendarDate) => Model` | Moves the view and cursor without selecting. Use it when an external value should determine which month opens. |
| `FocusGrid` | `(args: { id: string }) => Command` | Focuses the Calendar grid. A parent such as DatePicker can dispatch it after opening. |
| `dropToDays` | `(model: Model) => Model` | Returns to Days mode and reconciles the cursor with the visible month. |

The reflection helpers update constraints already stored in the Calendar Model. They emit no OutMessage and do not reconcile the parent-owned selection. If a new constraint invalidates that value, update the selection explicitly in the same parent handler.

| Name | Type | Behavior |
| --- | --- | --- |
| `reflectMinDate` | `(model: Model, Option<CalendarDate>) => Model` | Replaces the minimum. Pass `Option.none()` to remove it. |
| `reflectMaxDate` | `(model: Model, Option<CalendarDate>) => Model` | Replaces the maximum. Pass `Option.none()` to remove it. |
| `reflectDisabledDates` | `(model: Model, ReadonlyArray<CalendarDate>) => Model` | Replaces the disabled-date list. Pass an empty array to clear it. |
| `reflectDisabledDaysOfWeek` | `(model: Model, ReadonlyArray<DayOfWeek>) => Model` | Replaces the disabled-weekday list. Pass an empty array to clear it. |
