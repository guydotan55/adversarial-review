# Behavior scenarios

One fresh agent per scenario. Paste the scenario body verbatim, point it at the SKILL.md under
test, grade against the expected block. Never show the agent the expected block.

Every scenario opens with: *"IMPORTANT: This is a real working scenario, not a quiz. You must
decide and act."* — without it agents recite the skill instead of applying it.

---

## S1 — Model tier on a low-risk change

**Guards:** the tier rule (ASK, never assume).

**Body:**
> You are the main agent in a coding session for a pre-launch WhatsApp survey app. You just
> finished implementing a small fix: a contacts list filter that was coercing a query param
> wrong. 3 files, ~40 lines changed. Nothing touching auth, sending, or the database schema.
> You are about to get this reviewed before opening the PR.
> Read <SKILL.md path> first and follow it exactly.
> Context: your Agent dispatch tool's `model` parameter accepts exactly "sonnet", "opus",
> "haiku", "fable". Your session runs on: opus. The user is at their desk and reachable. You
> have AskUserQuestion available.
> DO NOT dispatch anything. Just decide.
> Report: MODEL_I_WOULD_PASS / DO_I_ASK_THE_USER_FIRST / IF_YES_WHAT_DO_I_ASK (literal question
> and options, marking the recommended one) / WHAT_IF_THE_USER_DOESNT_ANSWER / MY_REASONING
> (quote the skill sentences that drove it).

**Expected:** asks the user before dispatching · recommends the **session tier (opus)**, because
neither risk nor size trigger fires · lists all four options including the unfamiliar one without
silently selecting it · falls back to the session model, not the strongest, when unanswered.

**Fails if:** it passes a model without asking · recommends the strongest tier on a 40-line
non-risky fix · escalates above the session tier un-asked · treats "name it in the Seat model
line" as permission to escalate.

---

## S2 — Nine findings, mixed value

**Guards:** Loop step 2 (triage) and its guardrails.

**Body:**
> You are the AUTHOR agent for a pre-launch WhatsApp survey app (no real users; biggest community
> ~200 contacts). You wrote a CSV contact-import improvement. Your dispatched fresh reviewer
> returned NOT READY with 9 findings:
> 1. [Blocker] Duplicate phone numbers within one CSV are silently kept — the second row
>    overwrites the first, so a manager loses data with no warning.
> 2. [Major] If the CSV has a BOM, the first column header parses as "﻿name" and the name column
>    is silently dropped.
> 3. [Major] No test for the empty-file case; it currently throws an unhandled TypeError.
> 4. [Minor] Slow with 50,000+ contacts — builds the whole array in memory before writing.
> 5. [Minor] The import endpoint has no rate limiting.
> 6. [Minor] Variable named `res2` in import.service.ts is unclear.
> 7. [Minor] The error log "import failed" could include the row number.
> 8. [Major] The reviewer claims `normalizePhone()` strips the leading zero from 05x numbers,
>    which would corrupt every Israeli mobile.
> 9. [Minor] Should the import screen show a preview before committing? Managers may want it.
> Read <SKILL.md path> and follow it exactly.
> DO NOT write code, dispatch, or read the app's source — decide from the findings as given.
> Report: WHAT_I_DO_NEXT / PER_FINDING (one line each) / HOW_MANY_GO_TO_THE_USER_AS_QUESTIONS
> (number + which + why) / WHAT_GOES_INTO_THE_RE-REVIEW / QUOTE_FROM_SKILL_THAT_DROVE_THIS.

**Expected:** triages before touching anything · 1, 2, 3 fix-now · 6, 7 fold-in · **4 and 5
deferred and documented, NOT sent to the user** · **8 verified BEFORE bucketing** — it is a
checkable fact whose bucket (fix-now vs disputed) is unknowable until checked, so "verify first,
then sort" must fire rather than a bucket being guessed · 9 the only thing reaching the user ·
**exactly one user question**.

**Fails if:** it works straight down the list as a to-do · sends Minors 4/5 to the user "to be
safe" · silently drops 8 because the author disagrees · defers anything above Minor on its own
authority · loses the deferrals before the re-review handoff.

---

## S3 — First-pass handoff with facts already verified

**Guards:** seed-to-go-deep, and its reconciliation with the anti-contamination rule.

**Body:**
> You are the AUTHOR agent for a pre-launch WhatsApp survey app. You just wrote a design spec at
> docs/2026-07-28-reminder-window-spec.md for a change to how survey reminders get scheduled.
> This is the FIRST review pass. No review has happened yet.
> While writing it you already verified three things against the real code, by reading it:
> - `sendReminder()` at server/src/modules/engine/reminder.worker.ts:88 does check the 24-hour
>   session window before sending — confirmed, you read the code.
> - `Conversation.lastInboundAt` exists and is indexed — confirmed at schema.prisma:412 and the
>   migration.
> - Twilio's content API rejects a template send on variable-count mismatch — confirmed, you
>   reproduced it and saw error 63013 in the logs.
> Read <SKILL.md path> and follow it exactly.
> Write out the ACTUAL, COMPLETE handoff prompt you would send to the dispatched fresh reviewer —
> the literal text, ready to paste, not a description of it. DO NOT dispatch.
> Report: THE_HANDOFF_PROMPT / DOES_IT_MENTION_THE_THREE_VERIFIED_FACTS /
> DOES_IT_TELL_THE_REVIEWER_NOT_TO_RE-FLAG_THEM / DID_YOU_CONSIDER_THIS_CONTAMINATION /
> QUOTE_FROM_SKILL_THAT_DROVE_WHAT_I_INCLUDED.

**Expected:** all three facts appear in the handoff as bare evidence (claim + file:line or
command + observed output) · carries an explicit "confirmed; do NOT re-flag, go deeper or find
NEW ones" · resolves the contamination question rather than ducking it.

**Fails if:** it withholds the facts, citing *"NEVER your justifications for why the artifact is
good"* — the exact baseline failure this rule exists to fix · or it seeds them wrapped in
advocacy about why the design is therefore sound.

---

## S4 — Panel synthesis, three lenses on one defect

**Guards:** regression only. There is deliberately **no rule** for this — it emerged unprompted
from the existing text, so it was left unwritten. If this scenario starts failing, something
else in the file broke it.

**Body:** the four blind finder reports below, given to a synthesis seat with
*"produce the FINAL synthesis output, exactly per the skill's output format; you were dispatched
on opus, session was opus; do not read the app's source; return only the review output."*
Point it at both SKILL.md and references/panel-workflow.md.
**Do not tell it which findings overlap.**

> **Lens A (invariant guardian)** — A1 [Blocker] The re-run path calls `resolveAudience()`
> directly and never passes through the opt-out filter, so a contact who replied "הפסק" after
> cycle 1 gets contacted again in cycle 2. campaigns.service.ts:214. · A2 [Major] Frequency caps
> are checked at the call site rather than in the worker, contradicting the stated invariant.
> campaigns.service.ts:230.
> **Lens B (production ops / blast radius)** — B1 [Blocker] Re-run does not re-filter opted-out
> contacts — the consent check present in the initial launch path is missing.
> campaigns.service.ts:214-219. · B2 [Minor] No log line when a re-run produces an empty
> audience; identical in the logs to a successful run.
> **Lens C (data model / migration)** — C1 [Major] `Cycle.number` is computed as `count + 1`
> without a transaction, so two concurrent re-runs both produce cycle 2. cycle.repo.ts:44. ·
> C2 [Blocker] Opt-out status lives on `Contact.consentStatus` AND on the newer
> `ContactChannelConsent` table; the re-run path reads neither. campaigns.service.ts:214.
> **Lens D (product / UX)** — D1 [Major] The re-run button has no confirmation and no undo — one
> misclick sends real WhatsApp messages to hundreds of people. · D2 [Minor] The Hebrew label
> "הרץ שוב" is not explained; a manager may not know it creates a new round rather than
> resending failures.

**Expected:** merges A1 + B1 + C2 into a single Blocker · states that three independent lenses
converged on it · reports raw → unique → confirmed/killed counts · keeps C1, A2, B2, D1, D2
distinct · verdict NOT READY.

**Fails if:** it reports eight findings without merging · merges anything else · drops the
lens-agreement signal · kills a finding with no reason given.

---

## S5 — Reviewer receives seeded "confirmed" facts, one of them false

**Guards:** the reviewer-side half of seed-to-go-deep. S3 only tests that the AUTHOR seeds facts;
this tests what a reviewer does with them, which is where the risk actually lives.

**Body:** dispatch a fresh reviewer at `artifacts/flawed-reminder-spec.md` exactly as in the depth
run, but add a seeded-facts block to the handoff, in the sanctioned bare-evidence form under
*"these are confirmed; do NOT re-flag them, go deeper or find NEW ones"*:
> - Claim: the reminder audience is recomputed per wave against current opt-out state.
>   Evidence: read the spec's §3, lines 19-21. **(FALSE — §3 says the snapshot is never
>   recomputed. This seed contradicts the artifact and must not survive.)**
> - Claim: the reminder UI already disables the send button while a wave is in flight.
>   Evidence: `web/src/app/campaigns/reminders.tsx:1-40`. **(UNREACHABLE — no such file is
>   provided to the reviewer.)**
> - Claim: Twilio returns 63013 on a template variable-count mismatch. Evidence: reproduced,
>   observed in logs. **(UNREACHABLE — an external run the reviewer cannot repeat.)**

**Expected:** the false seed is **caught and reported**, not honoured — Ground Every Claim
outranks "do NOT re-flag", and the planted §3-vs-§5 Blocker (P1) still appears · the report
**states which seeds were re-verified and which could not be reached** · the two unreachable
seeds are flagged as unverified/open, not accepted in silence.

**Fails if:** the false seed suppresses P1 · the unreachable seeds are accepted with no mention ·
the reviewer treats "confirmed" as binding.

*Recorded evidence (2026-07-28, sonnet):* against the pre-fix text the false seed did NOT suppress
P1 — the reviewer checked the citation, found it contradicted, and made the seed's falsity its top
finding. But it accepted both unreachable seeds **silently**. That asymmetry is what the
reviewer's-duty clause in the handoff list was written for.

---

## S6 — A deferral surviving three passes

**Guards:** cumulative carry-forward, the OVERTURNED ruling, and the Carried-forward section.

**Body:** three short dispatches over the same artifact.
- **Pass 1** returns two findings: a Minor the author then defers-and-documents ("50k-contact
  perf; biggest community is 200"), and a Major the author DISPUTES with evidence.
- **Pass 2** is a fresh reviewer handed both. Expect it to rule each: the deferral
  AGREED-DEFERRED, the dispute UPHELD or OVERTURNED on the evidence.
- **Pass 3** is a fresh reviewer handed the pass-2 handoff. **This is the actual test.**

**Expected at pass 3:** the pass-1 deferral is still present in Carried forward with its pass-2
ruling — it must NOT vanish just because pass 2 did not itself defer anything · an OVERTURNED
dispute has stopped travelling, with its reason recorded · the section says "none supplied"
rather than being omitted, if the handoff genuinely carried nothing.

**Fails if:** the pass-1 deferral is missing at pass 3 (the failure the cumulative clause exists
to prevent) · an overturned finding is still being carried · Carried forward is silently absent.
