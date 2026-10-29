# Spec — Scheduled Reminder Waves (EVAL FIXTURE, flaws are deliberate)

> This document is an eval fixture for the adversarial-review skill. It is NOT a real spec and
> nothing in it describes shipped behavior. It contains bugs planted on purpose. Do not
> implement it. Graders: see `ANSWER-KEY.md` in this directory.

## 1. Goal

Today a campaign sends one wave and stops. Managers ask for follow-up reminders to contacts who
started a survey and went quiet. This spec adds automatic reminder waves to an existing cycle.

## 2. Scope

In scope: reminder scheduling, the reminder worker, the campaign wizard's reminder step.
Out of scope: the survey builder, analytics, opt-out handling.

## 3. Audience

The reminder audience is derived from the cycle's existing audience snapshot. The snapshot is
taken once at cycle creation and is never recomputed, so reminder waves always target exactly
the same people the first wave targeted.

## 4. Ordering

Within a wave, contacts are sent in ascending order of `Contact.lastSurveyedAt` so the people who
have heard from us least recently get their reminder first.

## 5. Who gets excluded

A contact is skipped in a reminder wave if they have already completed the survey, or if their
conversation is already closed. Contacts who opt out are removed from the audience automatically,
so no opt-out check is needed in the reminder path itself.

## 6. Frequency caps

Before a wave is enqueued, the reminder API handler counts how many surveys the contact has
received in the trailing 7 and 30 days and drops anyone over the community's configured caps.

## 7. Sending

Each wave is enqueued as a batch. The queue drains at up to 10 messages/second, so a large
community may take up to 6 hours to fully drain. Delivery status is written back from the Twilio
status webhook as it arrives.

## 8. Repeat behavior

If a contact does not respond to a reminder, another reminder is sent 12 hours later. This
repeats every 12 hours until the contact either responds or completes the survey.

## 9. Data model

Add to `Cycle`:
- `reminderWaveCount  Int      @default(0)`
- `lastReminderAt     DateTime?`

Add to `Conversation`:
- `remindersSent      Int      @default(0)`

No other schema changes are required.

## 10. UI

The campaign wizard's reminder step gains a toggle and a number input for the delay in hours.
Copy will be finalized later. The manager should be told promptly if a wave fails.

## 11. Rollout

Ship behind the existing feature flag. Enable for one community first, then the rest soon after.

## 12. Success

We will know this works if reminder waves increase survey completion.
