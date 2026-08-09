---
url: https://foldkit.dev/api-reference/ui-toast-test
title: "Ui/Toast/test"
description: "API documentation for the Ui/Toast/test module."
access_date: 2026-08-09T15:49:55.717Z
current_date: 2026-08-09T15:49:55.717Z
---

# Ui/Toast/test

## Functions

### drainEntry

function

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/toast/test.ts#L54)

```
/**
 * Builds a Story.Command.resolveAll step that drains a single toast
 *  entry's full animation and dismiss lifecycle. Resolving these Commands in
 *  order takes a freshly shown entry from its enter animation through
 *  auto-dismiss and its exit animation, ending with the entry removed from the
 *  stack and the `DismissedToast` OutMessage emitted.
 * 
 *  Showing a toast via `Toast.show` emits a multi-step lifecycle that a Story
 *  test must resolve in full or the story fails on leftover Commands. The
 *  steps are:
 * 
 *  - enter animation: `WaitForPaint` then `CompletedWaitForPaint`
 *  - enter settle: `WaitForAnimationSettled` then `EndedAnimation`
 *  - auto-dismiss: `WaitBeforeDismissal` then
 *    `CompletedWaitBeforeDismissal`
 *  - exit animation: `WaitForPaint` then `CompletedWaitForPaint`
 *  - exit settle: `WaitForAnimationSettled` then `EndedAnimation`
 * 
 *  Each step resolves with the child's raw result Message. `resolveAll` replays
 *  the matched Command's own recorded wrapping, so a parent that embeds the
 *  toast Submodel drains the same way without restating its `Got*` lift.
 */
(__namedParameters: DrainEntryInput): (simulation: StorySimulation<Model, Message, OutMessage>) => StorySimulation<Model, Message, OutMessage>
```

## Types

### DrainEntryInput

type

[source](https://github.com/foldkit/foldkit/blob/3fc75b1b3c8fd9f882c87184de3889d8145cc6e6/packages/ui/src/toast/test.ts#L16)

```
/**
 * Input for drainEntry. `entryId` selects the entry whose lifecycle
 *  to drain. `version` is the auto-dismiss timer version echoed back by
 *  `CompletedWaitBeforeDismissal`; it defaults to `0`, the version a freshly
 *  shown entry carries.
 */
type DrainEntryInput = Readonly<{
  entryId: string
  version: number
}>
```
