# Results

Append a dated block per run. Never overwrite — the trend is the point.

---

## 2026-07-28 — baseline, after the tier/triage/seed edits

Runner model: **sonnet** for every seat. Skill under test: `SKILL.md` as it stood after the tier/triage/seed edits.
Note: the fix pass after the change's own adversarial review (11 Majors) changed the tier, triage,
Carried-forward and Seat-model text *after* these runs. The S1/S2/S3 passes below therefore
attest to that pre-fix text, not to the merged text — the next run re-establishes the baseline.

### Behavior scenarios

| Scenario | Before the edit | After | Note |
|---|---|---|---|
| S1 model tier | **FAIL** — passed `"fable"`, asked nothing, escalated above the session tier on a 40-line non-risky fix. Cited *"escalate anyway - loudly"* as license, and looked up a model catalog to confirm fable outranked opus. | **PASS** | Asks first; recommends the session tier (opus); lists `fable` as an option without picking it; falls back to the session model when unanswered. |
| S2 triage | **FAIL** — no buckets. Fixed 5 findings, sent **3** to the user (incl. rate-limiting a pre-launch endpoint with no users). Nothing deferred, nothing documented. | **PASS** | Five buckets used; 4 and 5 deferred-and-documented; 6, 7 folded in; 8 routed as a checkable fact with the disputed path named; **1** user question, not 3. |
| S3 seed facts | **FAIL** — withheld all three verified facts, quoting *"NEVER your justifications for why the artifact is good"* and calling the seed "exactly the author-serving contamination this line prohibits." | **PASS** | All three seeded as bare evidence with "confirmed; do NOT re-flag"; explicitly reconciled against the contamination rule instead of ducking it. |
| S4 panel consensus | **PASS** (no rule — emergent) | **PASS** | Regression only. Merged A1+B1+C2, wrote *"Three of four independent lenses... converged... high-confidence, not a single reviewer's misread"*, reported 8 raw → 6 unique → 6 confirmed / 0 killed. Edits did not break it. |

The first drafts of S2 and S4 leaked the answer and were thrown out; the "before" figures above are
from the corrected scenarios. See `README.md` → *the trap that voided two of ours*.

### Depth — `artifacts/flawed-reminder-spec.md`

| Metric | Result |
|---|---|
| Planted flaws caught | **5 full · 2 partial · 1 miss** — P1 ✅ · P2 ❌ · P3 ✅ · P4 ✅ · P5 partial · P6 partial · P7 ✅ · P8 ✅. *(An earlier version of this row read "6 / 8", derived by scoring each partial as ½ under a convention stated nowhere. Raw marks only — no derived fraction. A future run is better/worse than this one on full-catch count first, partials second.)* |
| Hard questions raised | **3 / 3** — including HQ1, the 24-hour-window gap that is the one that would actually bite |
| Severity discipline | Every finding carried Where + Why + a specific Fix; ranked; verdict NOT READY. P4 was rated a tier higher than the key (Major → Blocker) — acceptable, not a miss. |
| Padding / invented findings | None |
| Seat model line | Present and correct — it noted it was a **downgrade** (dispatched sonnet, session opus), not an escalation |

**The gaps are the useful part of this run:**
- **P2 missed outright.** §4 orders by `Contact.lastSurveyedAt` while §9 says "no other schema
  changes are required" — the column is never added. It found a *different* schema gap (no
  per-contact reminder timestamp) and stopped there. Cross-checking every field a spec references
  against its own data-model section is exactly the sweep that did not happen.
- **P5 partial.** It flagged the vague "told promptly if a wave fails" (§10) but never asked what
  happens when Twilio errors mid-batch during a 6-hour drain.
- **P6 partial.** §2 declares opt-out out of scope while §5 leans on opt-out behavior to justify
  omitting a check. It caught the scope inconsistency about the API handler but not this one.
  (A later control run by the change's reviewer caught P6 in full, as a Major — so this is run
  variance, not a reproducible blind spot. Recorded rather than smoothed over.)

### Fix pass — after the change's own adversarial review

That review returned **NOT READY: 11 Majors, 5 Minors, 5 hard questions**, almost all in the seams
between the new text and rules that already existed. Re-run after the fixes (runner: sonnet):

| Scenario | Result | What it proves |
|---|---|---|
| S1 tier | **PASS** | Shows the risk classification instead of asserting it · exact `DEFAULTED, user not asked/no answer` marker · answers YES to re-asking on every re-review dispatch, citing the new non-locked clause. |
| S2 triage | **PASS** | Verify-first fires — finding 8 is reported as *"Cannot bucket yet"* rather than guessed into a bucket · refuses to self-defer a Major by reclassifying it, quoting the new severity rule. |
| S5 seeded facts (new) | **PASS** | The false seed is caught and reported as the top Blocker, *"not accepted as fact"* · both unreachable seeds are named and left open · report opens with an explicit seed-verification block · the planted P1 Blocker still lands. |

The pre-fix baseline for S5 is the reviewer's own control arm: the false seed was already killed by
Ground Every Claim, but **both unreachable seeds were accepted in silence**. That silence is what
the reviewer's-duty clause fixed, and S5 now guards it.

Also verified in the S5 output: the new `Seat model` downgrade form (`session on opus - seat ran
cheaper on sonnet (user chose)`) and `Carried forward` stating *"None supplied"* rather than being
omitted. Before the fix, three separate agents improvised their own wording for the missing
downgrade state.

### Pass 2 — fresh reviewer on the fix pass

**NOT READY: 1 Blocker, 7 Majors, 5 Minors.** Severity did not drop from pass 1, which by this
skill's own rule means the fixes were not real. The diagnosis was narrow and worth recording:
**every finding was a propagation seam, not a bad decision** — an edit landing in one place and not
its neighbours. `SKILL.md` was fixed and `panel-workflow.md` was never reopened (the Blocker: it
still said the tier question was *folded into* the consent question, the exact thing the fix
forbade). A bullet was inserted without re-anchoring the sentence that pointed at it. An exception
was added without amending the rule it excepted. A count was changed in one file of three.

Fixed in the third commit: the Blocker (all three `panel-workflow.md` divergences), the Size-trigger
audit gap, the missing RESOLVED ruling, the undefined "load-bearing", and the mis-anchored
contamination sentence — plus five cheap fold-ins.

**Documented deferrals** (the owner's scope call, 2026-07-29 — real findings, not fixed here):
- **Trigger names vs the plain-words rule.** `SKILL.md` requires naming the Risk/Size trigger in the
  tier question while the plain-words rule forbids mechanism jargon there. Acceptable now because
  the only person reading that question is technical; revisit if a non-technical user ever drives a
  review.
- **S2's expected block is unachievable under its own body constraint** — the body forbids reading
  the source, so finding 8 cannot literally be verified. Grading currently relies on the unwritten
  convention that "verify" means *route to verification*. Fix when S2 is next touched.
- **S3, S4 and S6 have no result against the merged text.** The harness gate this change introduces was
  knowingly waived for this change. S6 has never run at all. **First job for whoever next edits this
  skill: run all six and establish a clean baseline.**
- **Five precedence declarations are scattered across the file** and three of pass 2's Majors were
  collisions between them. Whether they belong in one precedence block is an open judgment call.

**Real findings beyond the key** (logged, not yet promoted — promote if a later run finds them again):
- `Cycle.reminderWaveCount` and `Conversation.remindersSent` are added in §9 and then referenced by
  no rule anywhere — dead fields, and a strong hint the missing repeat-cap rule was dropped in editing.
- A fixed 12-hour cadence alternates between the same two clock times forever, so a wave that starts
  at 11pm reminds people at 11pm every second round. No quiet-hours constraint anywhere.
- §11's rollout has no graduation gate and no rollback trigger.

---

## 2026-07-29 — CLEAN BASELINE, all six scenarios (the debt from the previous change)

Runner model: **sonnet** for every seat. Skill under test: `SKILL.md` as merged to main.

This is the run the previous change owed and did not do. S3, S4 and S6 had no result against merged text; S6 had
never run at all. All six now have one.

| Scenario | Result | Evidence |
|---|---|---|
| S1 model tier | **PASS** | Classified BOTH triggers explicitly ("Risk does NOT fire… Size does NOT fire") — the fix-pass-2 requirement · recommended the session tier · listed the unfamiliar option without selecting it · exact `dispatched on opus - session same (DEFAULTED, user not asked/no answer)` · answered YES to re-asking per dispatch, citing the non-locked clause. |
| S2 triage | **PASS** | Verify-first fired — finding 8 returned as "Not bucketed. Checkable fact, not yet verified" · 1 user question (was 3 pre-change) · refused to reclassify a Major down to self-defer, quoting the severity guardrail verbatim. |
| S3 seeding | **PASS** | All three facts seeded as bare evidence with "do NOT re-flag" · contamination addressed rather than ducked · **propagated the reviewer's-duty into its own handoff unprompted** ("state explicitly which of the three you re-verified and which you could not reach"). |
| S4 panel consensus | **PASS** | Merged A1+B1+C2 into one Blocker, each lens's distinct detail preserved · 8 raw → 6 unique → 6 confirmed / 0 killed · kept the other five distinct · flagged its own no-source-access limitation loudly and unprompted. |
| S5 seeded facts | **PARTIAL → fixed → PASS** | See below. |
| S6 deferral survival | **PASS** *(first ever run)* | Pass 2 ruled the carried Minor AGREED-DEFERRED and the disputed Major **OVERTURNED** (correctly — the "1 msg/sec WhatsApp limit" was fabricated). At pass 3 the pass-1 deferral was **still in Carried forward with its ruling**, and the OVERTURNED finding had **stopped travelling**. The cumulative-carry and OVERTURNED machinery added in that change's fix pass 2 works. |

### S5 — the one real finding, and the fix

**PARTIAL against the merged text.** The reviewer killed the false seed properly (*"I checked the cited lines
(§3, 19-21) directly and they say the opposite… Per Ground Every Claim, the seed does not survive
that check"*) and still landed the planted P1 Blocker. But it **never stated which seeds it could not
reach** — the `reminders.tsx` and Twilio-63013 seeds simply went unmentioned.

Not a missing rule: `SKILL.md` already said *"State in your report which seeds you re-verified and
which you could not reach."* The pre-merge run of S5 **did** produce a seed-verification block; this
one did not. **The clause fired about half the time.** The dangerous half — a false seed suppressing
a real finding — held in both runs; the reporting half was flaky.

**Diagnosis:** it was advice buried in a handoff bullet, competing with a report structure that never
asked for it. The `Seat model` line had the same problem in an earlier change and was fixed the same way.

**Fix (this change):** a **required `## Seeds checked` output section** — one line per seat, exactly one
of RE-VERIFIED / CONTRADICTED / UNREACHABLE, with "none supplied" when there were no seeds.

**GREEN after the fix:** the section appeared with all three seeds accounted for — one CONTRADICTED
(promoted to the top Blocker), two UNREACHABLE each naming *why* it could not be checked, whether the
artifact corroborates it, and whether any finding depends on it. Strictly more than the rule asked for.

**Method note:** a rule that only lives in an input instruction complies inconsistently; a rule with a
named slot in the required output complies because the slot is visibly empty otherwise. Prefer the
output section when a behavior must be reliable.
