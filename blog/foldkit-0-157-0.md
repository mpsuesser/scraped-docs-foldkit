---
url: https://foldkit.dev/blog/foldkit-0-157-0
title: "Foldkit 0.156.0 and 0.157.0"
description: "Two new recommended @foldkit/oxlint-plugin rules, less boilerplate when matching refined unions, plus updates to Story, Scene, Mount, @foldkit/ui, and Foldkit's Vite plugin."
access_date: 2026-09-02T16:31:11.406Z
current_date: 2026-09-02T16:31:11.406Z
---

[← Blog](https://foldkit.dev/blog)

# Foldkit 0.156.0 and 0.157.0

September 2, 2026 · Devin Jameson

Foldkit 0.157.0 and 0.156.0 are out.

0.156.0 was a small release.

- HoverIntent now closes without the pointer grace delay when focus leaves the trigger and panel.
- Parents can programmatically close HoverIntent with `HoverIntent.close`, for example after a user selects a menu item.

Now for 0.157.0. The appetizer is improved type safety in Story and Scene tests. The main dish is `@foldkit/oxlint-plugin` updates, with a side of more ergonomic OutMessage matching.

## Better Foldkit linting

The recommended `@foldkit/oxlint-plugin` preset gained two rules.

`foldkit/no-impure-call-at-decision-time` reports time and randomness calls made while update is deciding which Commands to return, instead of when those Commands execute.

```
const Save = Command.define('Save', {
  args: { createdAt: Schema.Number },
  messages: [Message.CompletedSave],
  execute: ({ createdAt }) =>
    Effect.succeed(Message.CompletedSave({ createdAt })),
})

const save = (model: Model): Update.Return<Model, Message> => ({
  model,
  // Wrong: Date.now() runs while update is deciding which Commands to return.
  commands: [Save({ createdAt: Date.now() })],
})
```

Move the read into the Command's Effect and return the value in its result Message:

```
const Save = Command.define('Save', {
  messages: [Message.CompletedSave],
  // Right: the clock is read only when the Command executes.
  execute: Clock.currentTimeMillis.pipe(
    Effect.map(createdAt => Message.CompletedSave({ createdAt })),
  ),
})

const save = (model: Model): Update.Return<Model, Message> => ({
  model,
  commands: [Save()],
})
```

`foldkit/prefer-effect-module-names` reports abbreviated and trailing-underscore Effect import aliases, and auto-fixes aliases when it can identify the exported module name.

Existing applications using the preset may produce new lint errors after upgrading.

Across the preset, rules now recognize aliases for Foldkit, `@foldkit/ui`, Effect, and child Message imports. They ignore local shadows and no longer mistake unrelated `message` fields for Submodel Messages.

The preset already reported parent code that constructs a child Message. In 0.157.0, the rule also catches that code when the child import is aliased. For example, a parent should not decide that opening a Dialog means dispatching its `RequestedOpen` Message:

```
const foldDialogOpen = Update.foldChildStep({
  // Wrong: the parent constructs one of Dialog's internal Messages.
  update: (dialog: Dialog.Model) =>
    Dialog.update(dialog, Dialog.Message.RequestedOpen()),
  read: readDialog,
  write: writeDialog,
  toParentMessage: toGotDialogMessage,
  foldOutMessage: foldDialogOutMessage,
})
```

Dialog exposes `open`, so the parent can fold that operation without reaching into the child's Messages. Parents should drive child behavior through update functions exposed by the child:

```
const foldDialogOpen = Update.foldChildStep({
  // Right: the parent calls an update function owned by Dialog.
  update: Dialog.open,
  read: readDialog,
  write: writeDialog,
  toParentMessage: toGotDialogMessage,
  foldOutMessage: foldDialogOutMessage,
})
```

This lets the parent call a child-owned update function without depending on the child's internal Messages. `Update.foldChildStep` still handles the child Model, Commands, and OutMessages. Animation in `@foldkit/ui` now exposes `show`, `hide`, and `toggle` update functions for the same reason.

## Refined union matching

Foldkit union matchers now accept a structurally refined union as their optional second type argument. For a Listbox whose value is refined to `Plan`, this lets `Listbox.OutMessage.match` keep `value` typed as `Plan` without a separate Effect Match pipeline.

Before:

```
// Before: preserving Plan requires a separate Effect Match pipeline.
const foldListboxOutMessage = Match.type<Listbox.OutMessage<Plan>>().pipe(
  Match.withReturnType<Update.Step<Model, Message>>(),
  Match.tagsExhaustive({
    Selected:
      ({ value }) =>
      model => ({ model: evo(model, { maybePlan: () => Option.some(value) }) }),
  }),
)
```

After:

```
// After: Foldkit's matcher preserves Plan directly.
const foldListboxOutMessage = Listbox.OutMessage.match<
  Update.Step<Model, Message>,
  Listbox.OutMessage<Plan>
>({
  Selected:
    ({ value }) =>
    model => ({ model: evo(model, { maybePlan: () => Option.some(value) }) }),
})
```

The new form is shorter, remains exhaustive, and keeps `value` typed as `Plan` in the `Selected` handler.

Other changes:

- Story and Scene now type-check Message and OutMessage steps against the update under test, with a new `Story.steps` API for reusable Story sequences.
- Mounts can use the new `viewStateChanges` Stream to stop DOM interaction while DevTools shows a historical view.
- Textarea content must now use `h.Value`.
- `Machine.unreachableStates` and `Machine.deadTransitions` can now account for states entered through persistence or deep links.
- Dialog now falls back to the first focusable element—or the dialog itself—when its requested focus target is missing or cannot receive focus.
- DevTools overlay dependencies are now preloaded, so Vite does not reload on first use.
- The former Manifesto page is now [Why Foldkit](https://foldkit.dev/get-started/why-foldkit).

Thank you to [@rjdellecese](https://github.com/rjdellecese) for proposing the Effect module naming convention and Mount view-state API, and to [@artile](https://github.com/artile) for reporting the Dialog focus issue. Thank you also to [@armancharan](https://github.com/armancharan) for adding `vitest.config.ts` typechecking across the repo and [@filipfalcon](https://github.com/filipfalcon) for the Vite preload fix!

The full [0.156.0](https://github.com/foldkit/foldkit/releases/tag/foldkit%400.156.0) and [0.157.0](https://github.com/foldkit/foldkit/releases/tag/foldkit%400.157.0) release notes cover every package and migration detail.

Thanks to everyone building with Foldkit!

Devin
