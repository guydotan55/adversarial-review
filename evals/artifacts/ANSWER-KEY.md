# Answer key — flawed-reminder-spec.md

**Never show this to a runner.** Grade its report against this list after it returns.

All of these are findable from the spec text alone — no codebase access required. That is
deliberate: it keeps the eval runnable anywhere and keeps grading objective.

## Planted flaws

| # | Severity | Where | The flaw |
|---|----------|-------|----------|
| P1 | Blocker | §3 vs §5 | Direct contradiction. §3 says the audience snapshot "is taken once at cycle creation and is never recomputed"; §5 says opted-out contacts "are removed from the audience automatically" and concludes no opt-out check is needed. Both cannot hold. As written, someone who opts out after cycle creation still gets reminders — breaking the permanent-opt-out invariant on a live send path. |
| P2 | Blocker | §4 vs §9 | Unbuildable. §4 orders by `Contact.lastSurveyedAt`, but §9 enumerates every schema change and that column is not among them, nor stated to already exist. |
| P3 | Blocker | §8 vs §7 | Unbounded reminder loop. "Repeats every 12 hours until the contact either responds or completes" has no cap and no exit for a contact who does neither — an unresponsive contact is messaged forever. §9's `remindersSent` counter is added but never used to stop anything. |
| P4 | Major | §6 | Frequency caps are enforced in the API handler. The stated system invariant is that cap logic lives in the worker, enforced there and not at call sites — so any other path that enqueues a wave bypasses the cap entirely. |
| P5 | Major | §7 (absence) | No failure behavior. Nothing says what happens when Twilio rejects or errors on a message mid-batch: retry, skip, mark failed, abort the wave? A 6-hour drain with no defined error path fails silently. |
| P6 | Major | §2 vs §5 | Scope contradiction. §2 declares opt-out handling out of scope; §5 then makes a load-bearing claim about how opt-out removal works and uses it to justify omitting a check. A spec cannot both exclude a concern and depend on it. |
| P7 | Minor | §10, §11 | Ambiguity a builder could read two ways: "promptly", "soon after", "copy will be finalized later". None are acceptance criteria. |
| P8 | Minor | §12 | Unfalsifiable success metric — "increase survey completion" names no baseline, no target, no measurement window. |

## Hard questions the reviewer should raise

| # | The question |
|---|--------------|
| HQ1 | The whole design assumes a message can be sent whenever a wave fires, but WhatsApp only permits free-form sends inside the 24-hour window after the contact's last reply. What happens when a reminder is due and the window has closed — approved template, skip, or expire the conversation? The spec never says, and §8's 12-hour cadence guarantees this case. |
| HQ2 | Does a reminder count against the frequency cap as a survey send? §6 counts "surveys received"; if reminders don't count, §8's repeat behavior walks straight through the cap. |
| HQ3 | §7 says a wave may take 6 hours to drain while §8 schedules the next wave 12 hours after the previous *reminder*. Measured from which point — enqueue, first send, or that contact's own send? On a slow drain these produce materially different schedules. |

## Grading

- **Recall:** how many of P1–P8 appear, at roughly the right severity. Missing a Blocker is a
  regression; a severity off by one tier is not.
- **Hard questions:** how many of HQ1–HQ3 surface. HQ1 is the important one — it is the failure
  mode that would actually bite in production.
- **Discipline:** severity-ranked · every finding carries Where + Why + a specific Fix · a binary
  verdict (must be NOT READY) · no reassurance padding · no invented findings.
- **Bonus:** real flaws not on this list. Log them in `RESULTS.md` — if a run keeps finding the
  same extra one, promote it into this key.
