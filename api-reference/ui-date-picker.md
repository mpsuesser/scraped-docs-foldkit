---
url: https://foldkit.dev/api-reference/ui-date-picker
title: "Ui/DatePicker"
description: "API documentation for the Ui/DatePicker module."
access_date: 2026-08-13T05:04:07.016Z
current_date: 2026-08-13T05:04:07.016Z
---

# Ui/DatePicker

## Functions

### clear

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L299)

```
/** Programmatically clears the selected date. */
(model: DatePicker.Model): UpdateReturn
```

### close

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L292)

```
/** Programmatically closes the date picker. Use this in domain-event handlers. */
(model: DatePicker.Model): UpdateReturn
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L123)

```
/**
 * Creates an initial date picker model from a config. The selected date is
 * owned by the parent; pass its current value as `initialViewDate` to open the
 * calendar onto that month. The calendar and popover submodels are created
 * with derived ids so their DOM elements stay addressable. The popover is
 * opened in `contentFocus` mode so focus lands on the calendar grid instead of
 * the panel.
 */
(config: InitConfig): DatePicker.Model
```

### open

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L289)

```
/**
 * Programmatically opens the date picker, updating the model and returning
 * focus and popover commands. Use this in domain-event handlers.
 */
(model: DatePicker.Model): UpdateReturn
```

### selectDate

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L295)

```
/** Programmatically selects a date, committing it and closing the popover. Emits a `SelectedDate` OutMessage just like a user-initiated selection. */
(
  model: DatePicker.Model,
  date: {
    day: number
    month: number
    year: number
  }
): UpdateReturn
```

### triggerId

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L390)

```
/**
 * Returns the bare DOM id of the date picker trigger button, derived from
 *  the date picker's base id. The trigger is the embedded Popover's button,
 *  so the id is suffixed `-popover-button`. Use this to associate an external
 *  label with the trigger via a native `<label for={DatePicker.triggerId(id)}>`
 *  or an `aria-labelledby` reference.
 */
(id: string): string
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L244)

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L105)

```
/** Configuration for creating a date picker model with `init`. */
type InitConfig = Readonly<{
  disabledDates: ReadonlyArray<CalendarDate>
  disabledDaysOfWeek: ReadonlyArray<Calendar.DayOfWeek>
  id: string
  initialViewDate: CalendarDate
  isAnimated: boolean
  locale: Calendar.LocaleConfig
  maxDate: CalendarDate
  minDate: CalendarDate
  today: CalendarDate
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L402)

```
/**
 * Per-render view inputs passed to `view` via `h.submodel`'s `viewInputs` field.
 * 
 *  The DatePicker emits a `SelectedDate({ date })` OutMessage when the
 *  user commits a date. Consumers pattern-match this in their
 *  `GotDatePickerMessage` handler (third tuple element of
 *  `DatePicker.update`'s return) to lift the date into domain state.
 */
type ViewInputs = Readonly<{
  anchor: AnchorConfig
  ariaLabel: string
  ariaLabelledBy: string
  attributes: ReadonlyArray<ChildAttribute>
  backdropAttributes: ReadonlyArray<ChildAttribute>
  backdropClassName: string
  className: string
  isDisabled: boolean
  maybeSelectedDate: Option.Option<CalendarDate>
  name: string
  panelAttributes: ReadonlyArray<ChildAttribute>
  panelClassName: string
  toCalendarView: (attributes: UiCalendar.CalendarAttributes) => Html
  triggerAttributes: ReadonlyArray<ChildAttribute>
  triggerClassName: string
  triggerContent: (maybeDate: Option.Option<CalendarDate>) => Html
}>
```

## Constants

### ChangedViewMonth

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L78)

```
/**
 * Emitted when the visible month changes (propagated from the embedded
 * Calendar). Useful for month-scoped data loading.
 */
const ChangedViewMonth: CallableTaggedStruct<"ChangedViewMonth", {
  month: Int
  year: Int
}>
```

### Cleared

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L46)

```
/** Sent when the user clears the selected date. Does not close the popover. */
const Cleared: CallableTaggedStruct<"Cleared", {}>
```

### ClearedDate

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L92)

```
/**
 * Emitted when the user clears the selected date. The parent clears its own
 * value field.
 */
const ClearedDate: CallableTaggedStruct<"ClearedDate", {}>
```

### Closed

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L52)

```
/**
 * Sent when the popover should close. Delegates to Popover which returns
 * focus to the trigger button.
 */
const Closed: CallableTaggedStruct<"Closed", {}>
```

### GotCalendarMessage

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L33)

```
/** Wraps a Calendar submodel message for delegation. */
const GotCalendarMessage: CallableTaggedStruct<"GotCalendarMessage", {
  message: Union<readonly [
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
}>
```

### GotPopoverMessage

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L37)

```
/** Wraps a Popover submodel message for delegation. */
const GotPopoverMessage: CallableTaggedStruct<"GotPopoverMessage", {
  message: Union<[
    CallableTaggedStruct<"RequestedOpen", {}>,
    CallableTaggedStruct<"RequestedClose", {}>,
    CallableTaggedStruct<"BlurredPanel", {}>,
    CallableTaggedStruct<"PressedPointerOnButton", {
      button: Number
      pointerType: String
    }>,
    CallableTaggedStruct<"CompletedFocusPanel", {}>
  ]>
}>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L55)

```
/** Union of all messages the date picker component can produce. */
const Message: S.Union<[typeof GotCalendarMessage, typeof GotPopoverMessage, typeof RequestedSelectDate, typeof Cleared, typeof Opened, typeof Closed]>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L23)

```
/**
 * Schema for the date picker component's private interaction state. The
 * selected date is owned by the parent and passed in via
 * `ViewInputs.maybeSelectedDate`. This holds the embedded Calendar submodel
 * (the visible grid) and the embedded Popover submodel (the open/close +
 * transition layer).
 */
const Model: Struct<{
  calendar: Struct<{
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
  id: String
  popover: Struct<{
    animation: Struct<{
      id: String
      isShowing: Boolean
      transitionState: Literals<readonly ["Idle", "EnterStart", "EnterAnimating", "LeaveStart", "LeaveAnimating"]>
    }>
    contentFocus: Boolean
    id: String
    isAnimated: Boolean
    isModal: Boolean
    isOpen: Boolean
    maybeLastButtonPointerType: Option<String>
  }>
}>
```

### Opened

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L49)

```
/**
 * Sent when the popover should open. Triggers focus-grid on the embedded
 * Calendar so keyboard focus lands inside the grid instead of the panel.
 */
const Opened: CallableTaggedStruct<"Opened", {}>
```

### OutMessage

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L95)

```
/** Union of out-messages the date picker can produce. */
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
  }>,
  CallableTaggedStruct<"ClearedDate", {}>
]>
```

### RequestedSelectDate

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L42)

```
/**
 * Sent when the user commits a date via click or keyboard. Updates the
 * selected date, syncs the calendar, and closes the popover.
 */
const RequestedSelectDate: CallableTaggedStruct<"RequestedSelectDate", {
  date: Struct<{
    day: Int
    month: Int
    year: Int
  }>
}>
```

### SelectedDate

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L86)

```
/**
 * Emitted when the user commits a date selection (propagated from the
 * embedded Calendar). The popover has already closed; the parent stores the
 * committed date and passes it back in as `maybeSelectedDate`.
 */
const SelectedDate: CallableTaggedStruct<"SelectedDate", {
  date: Struct<{
    day: Int
    month: Int
    year: Int
  }>
}>
```

### focusDate

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L306)

```
/**
 * Moves the embedded calendar's view and cursor to a date without changing
 *  the selection (which the parent owns). Use it to navigate the picker onto a
 *  known date, for example after the parent sets its value externally (a URL
 *  parameter, a saved draft) so opening the picker shows that month. Returns
 *  the model directly because it produces no commands and no OutMessage.
 */
const focusDate: Reflect<Model, CalendarDate>
```

### reflectDisabledDates

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L350)

```
/**
 * Reflects the list of individually-disabled dates onto the embedded
 * calendar. Pass an empty array to clear. Does NOT reconcile the current
 * selection.
 */
const reflectDisabledDates: Reflect<Model, ReadonlyArray<CalendarDate>>
```

### reflectDisabledDaysOfWeek

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L365)

```
/**
 * Reflects the days of the week that are disabled onto the embedded calendar
 * (e.g. weekends). Pass an empty array to clear. Does NOT reconcile the
 * current selection.
 */
const reflectDisabledDaysOfWeek: Reflect<Model, ReadonlyArray<Calendar.DayOfWeek>>
```

### reflectMaxDate

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L336)

```
/**
 * Reflects the maximum selectable date onto the embedded calendar. Pass
 * `Option.none()` to remove the maximum. Does NOT reconcile the current
 * selection.
 */
const reflectMaxDate: Reflect<Model, Option.Option<CalendarDate>>
```

### reflectMinDate

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L322)

```
/**
 * Reflects the minimum selectable date onto the embedded calendar. Pass
 * `Option.none()` to remove the minimum. Use this when the minimum derives
 * from other Model state (e.g. a start date field whose current selection
 * constrains an end date picker).
 * 
 * Does NOT reconcile the current selection. If a previously-selected date
 * is now below the new minimum, it remains selected. Callers should `clear`
 * or reassign the selection explicitly if their domain requires it.
 */
const reflectMinDate: Reflect<Model, Option.Option<CalendarDate>>
```

### view

const

[source](https://github.com/foldkit/foldkit/blob/71556def2f366e7cb8ecc2caef61d0c08966d6ee/packages/ui/src/datePicker/index.ts#L437)

```
/**
 * Renders an accessible date picker: a trigger button that opens a popover
 * containing an accessible calendar grid. The date picker assembles the
 * embedded Calendar and Popover components into one flat API. Consumers
 * provide the trigger face and the calendar grid layout, DatePicker handles
 * focus choreography, open/close state, and form submission.
 */
const view: SubmodelView<DatePicker.Model, {
  _tag: "Opened"
} | {
  _tag: "Closed"
} | {
  _tag: "GotCalendarMessage"
  message: {
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
  }
} | {
  _tag: "GotPopoverMessage"
  message: {
    _tag: "RequestedOpen"
  } | {
    _tag: "RequestedClose"
  } | {
    _tag: "BlurredPanel"
  } | {
    _tag: "PressedPointerOnButton"
    button: number
    pointerType: string
  } | {
    _tag: "IgnoredMouseClick"
  } | {
    _tag: "SuppressedSpaceScroll"
  } | {
    _tag: "CompletedFocusPanel"
  } | {
    _tag: "CompletedFocusButton"
  } | {
    _tag: "CompletedLockScroll"
  } | {
    _tag: "CompletedUnlockScroll"
  } | {
    _tag: "CompletedInertOthers"
  } | {
    _tag: "CompletedRestoreInert"
  } | {
    _tag: "CompletedAnchorPopover"
  } | {
    _tag: "CompletedPortalPopoverBackdrop"
  } | {
    _tag: "GotAnimationMessage"
    message: {
      _tag: "Showed"
    } | {
      _tag: "Hid"
    } | {
      _tag: "CompletedWaitForPaint"
    } | {
      _tag: "EndedAnimation"
    }
  }
} | {
  _tag: "RequestedSelectDate"
  date: {
    day: number
    month: number
    year: number
  }
} | {
  _tag: "Cleared"
}, Readonly<{
  anchor: {
    gap: number
    isPlacementLocked: boolean
    offset: number
    padding: number
    placement: "top" | "right" | "bottom" | "left" | "top-start" | "top-end" | "right-start" | "right-end" | "bottom-start" | "bottom-end" | "left-start" | "left-end"
    portal: boolean
  }
  ariaLabel: string
  ariaLabelledBy: string
  attributes: readonly Array<Readonly<{
    __childAttribute: true
    attribute: unknown
    boundaryMappers: readonly Array<(message: unknown) => unknown>
    dispatch: DispatchSync
    resolveUnmount: (message: unknown) => () => void
  }>>
  backdropAttributes: readonly Array<Readonly<{
    __childAttribute: true
    attribute: unknown
    boundaryMappers: readonly Array<(message: unknown) => unknown>
    dispatch: DispatchSync
    resolveUnmount: (message: unknown) => () => void
  }>>
  backdropClassName: string
  className: string
  isDisabled: boolean
  maybeSelectedDate: Option<{
    day: number
    month: number
    year: number
  }>
  name: string
  panelAttributes: readonly Array<Readonly<{
    __childAttribute: true
    attribute: unknown
    boundaryMappers: readonly Array<(message: unknown) => unknown>
    dispatch: DispatchSync
    resolveUnmount: (message: unknown) => () => void
  }>>
  panelClassName: string
  toCalendarView: (attributes: CalendarAttributes) => Html
  triggerAttributes: readonly Array<Readonly<{
    __childAttribute: true
    attribute: unknown
    boundaryMappers: readonly Array<(message: unknown) => unknown>
    dispatch: DispatchSync
    resolveUnmount: (message: unknown) => () => void
  }>>
  triggerClassName: string
  triggerContent: (maybeDate: Option<{
    day: number
    month: number
    year: number
  }>) => Html
}>>
```
