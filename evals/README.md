# Evals — adversarial-review

A graded test for the skill, not part of any review. Nothing here runs during normal work.

**Run it when you EDIT the skill**, before opening the PR. Editing one rule's wording can
quietly degrade an unrelated one; per-edit RED/GREEN proves the new rule fires, and this
proves everything else still does.

## What's here

| File | What it tests |
|------|---------------|
| `scenarios.md` | **Behavior** — does an agent holding the skill do the right thing under pressure? Six scenarios, each with the exact expected outcome. |
| `artifacts/flawed-reminder-spec.md` | **Depth** — a spec with bugs planted on purpose. |
| `artifacts/ANSWER-KEY.md` | What is wrong with that spec. Do NOT show this to the runner. |
| `RESULTS.md` | Scorecard from each run. Append, never overwrite. |

## How to run

1. Pick a runner model and keep it constant across runs, or the scores aren't comparable.
   The recorded baseline used **Sonnet**. A cheap runner is the right choice here: if a rule
   only holds on the strongest tier, it isn't in the text yet.
2. **Behavior scenarios:** dispatch one fresh agent per scenario in `scenarios.md`. Paste the
   scenario verbatim, point it at the edited `SKILL.md`, collect its answer. Grade against the
   scenario's "expected" block. One agent per scenario — never batch them into one agent, the
   scenarios leak into each other.
3. **Depth artifact:** dispatch one agent, tell it to review `artifacts/flawed-reminder-spec.md`
   using the skill. Grade its findings against `ANSWER-KEY.md`: how many planted flaws it caught,
   at what severity, and what real extra flaws it found beyond the key.
4. Append a dated block to `RESULTS.md`, and **record the SKILL.md commit SHA you graded** — a
   scorecard without it cannot be compared to anything.

**Scoring.** Record raw marks — full catch / partial / miss — never a derived fraction. Partial
credit invites rounding in the author's favour; the first version of `RESULTS.md` did exactly
that and the change's own reviewer caught it. Compare runs on full-catch count first, partials second.

**What counts as a regression.** A behavior scenario flipping PASS → FAIL is a red test: fix it
before merging. A depth-run score moving by one partial is inside the noise — agents are
non-deterministic and one run per edit cannot separate noise from signal. If a depth score drops
by a full catch or a Blocker goes missing, re-run that arm once before believing it; if it
reproduces, treat it as red.

## Writing a new scenario — the trap that voided two of ours

**Do not leak the answer into the scenario.** The first draft of the triage scenario mentioned
that the user worries about never-converging review loops, and its report format asked "did you
sort these into categories?" Both point straight at the behavior under test, so the agent passed
for the wrong reason and the result had to be thrown away. Same for the panel scenario: it told
the runner which three findings were the same defect, which was the whole thing being measured.

Rules that follow from that:
- The scenario states **facts and pressure**, never the desired behavior.
- The report format asks **what did you do**, never **did you do X**.
- If a scenario passes on the FIRST try before you've written the rule, suspect the scenario
  before believing the skill.

## Honest limits

- Six behavior scenarios and one artifact is a small set. A pass here is evidence, not proof.
- The planted flaws are more obvious than real ones. Expect high catch rates in both arms;
  the signal is in output discipline and depth, not raw recall.
- Agents are non-deterministic. A single differing answer is noise; a consistent drop is not.
- This measures the skill's TEXT. It cannot measure whether a real review found a real bug.
