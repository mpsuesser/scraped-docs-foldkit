---
url: https://foldkit.dev/example-apps/job-application
title: "Job Application"
description: "A multi-step form with asynchronous email validation, cross-field date constraints, file uploads, and per-step error indicators."
access_date: 2026-09-05T19:30:16.678Z
current_date: 2026-09-05T19:30:16.678Z
---

[All Examples](https://foldkit.dev/example-apps)

# Job Application

A multi-step form with asynchronous email validation, cross-field date constraints, file uploads, and per-step error indicators.

Validation

Multi-step

UI Components

[Launch Playground](https://foldkit.dev/playground/job-application)

[View source on GitHub](https://github.com/foldkit/foldkit/tree/main/examples/job-application/src)

/

```
import { Crypto, Effect, Schema } from 'effect'
import { Calendar, Runtime } from 'foldkit'

import { BrowserCrypto } from '@effect/platform-browser'
import { Menu, Tabs } from '@foldkit/ui'

import { Message } from './message'
import { Model, Submission } from './model'
import {
  Attachments,
  CoverLetter,
  Education,
  PersonalInfo,
  Skills,
  WorkHistory,
} from './step'
import { update } from './update'
import { view } from './view/view'

// FLAGS

export const Flags = Schema.Struct({
  today: Calendar.CalendarDate,
  initialWorkHistoryEntryId: Schema.String,
  initialEducationEntryId: Schema.String,
  initialSkillsEntryId: Schema.String,
})
export type Flags = typeof Flags.Type

export const flags: Effect.Effect<Flags> = Effect.gen(function* () {
  const today = yield* Calendar.today.local
  const crypto = yield* Crypto.Crypto
  const initialWorkHistoryEntryId = yield* Effect.orDie(crypto.randomUUIDv4)
  const initialEducationEntryId = yield* Effect.orDie(crypto.randomUUIDv4)
  const initialSkillsEntryId = yield* Effect.orDie(crypto.randomUUIDv4)
  return {
    today,
    initialWorkHistoryEntryId,
    initialEducationEntryId,
    initialSkillsEntryId,
  }
}).pipe(Effect.provide(BrowserCrypto.layer))

// INIT

export const init: Runtime.ApplicationInit<Model, Message, Flags> = ({
  today,
  initialWorkHistoryEntryId,
  initialEducationEntryId,
  initialSkillsEntryId,
}) => ({
  model: {
    currentStep: 'PersonalInfo',
    personalInfo: PersonalInfo.init(today),
    workHistory: WorkHistory.init(today, initialWorkHistoryEntryId),
    education: Education.init(today, initialEducationEntryId),
    skills: Skills.init(initialSkillsEntryId),
    coverLetter: CoverLetter.init(),
    attachments: Attachments.init(),
    isPreviewVisible: false,
    submission: Submission.NotSubmitted(),
    stepMenu: Menu.init({ id: 'step-menu' }),
    stepTabs: Tabs.init({ id: 'step-tabs' }),
    isSubmitAttempted: false,
  },
})

export { Message, Model, update, view }
```
