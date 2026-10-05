---
url: https://foldkit.dev/best-practices/immutability
title: "Immutability"
description: "Update Models immutably with modifyFields, preserving references for unchanged branches and keeping state transitions predictable."
access_date: 2026-10-05T07:06:39.496Z
current_date: 2026-10-05T07:06:39.496Z
---

# Immutability

## Updating the Model with modifyFields

`update` returns a new Model instead of mutating the current one. Foldkit provides `modifyFields` for these immutable field updates. It wraps Effect's `Struct.evolve` with stricter key checking, so removing or renaming a Model field produces errors at every stale update site.

Using modifyFields

Each property in the transform object receives that field's current value and returns its next value. Omitted properties remain unchanged.

Use the current value when the change depends on it, such as incrementing a count. Use `() => value` when a Message payload or Command result replaces the field. Keep the transformer deterministic. Time, randomness, browser state, and other outside values belong behind a [Command](https://foldkit.dev/core/commands).
