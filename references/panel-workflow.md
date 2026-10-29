# Panel Workflow — assembling a multi-agent adversarial review

Read this BEFORE assembling a panel. The consent gate, role check, and tier rules in SKILL.md
apply to everything here and are not repeated in full.

## Shape — light by default

**Default pipeline: N diverse finders → one dispatched synthesis.** The synthesis agent merges
duplicates, sanity-checks findings against the artifact, and renders the verdict in the skill's
standard output format. No separate dedup or voting stage by default — evidence from ~69 real runs:
a majority of successful panels used exactly this shape, and orthogonal finder lenses produce
near-zero duplicate collisions (standalone dedup was a no-op: 47→47, 31→31).

Add stages only when earned:
- **Standalone dedup** only when finder lenses are expected to overlap (generic stances merged
  25–47% in practice). Run it with "do NOT drop distinct findings; when in doubt, don't merge."
- **Refutation voting (heavy mode — UNTESTED):** per-finding skeptics with refute-by-default and
  majority-kill. Label honestly: in 69 historical runs this structure never once completed (the
  only attempt aborted mid-run). A single refute-by-default skeptic, or synthesis-as-verifier,
  pruned a median ~11% of findings and once caught a fabricated Blocker — that is the proven
  verification shape.

## Sizing

- **Default: 4 orthogonal lenses.** The modal, reliably-completing size.
- **2–3 lenses** for a narrow, well-scoped diff or follow-up.
- **5–6 lenses** for consequential/novel/security-sensitive work.
- **8 is a ceiling, not a target.** Bigger panels carry a reliability tax: silent stance loss and
  hard aborts rise with fan-out (in real runs, 7-of-8 and 6-of-8 finders were silently rate-limited
  away, and the largest panel hard-aborted).
- **Diversity beats count.** Each lens must be a distinct failure-mode angle naming concrete things
  to attack — not N copies of "find bugs".

## Stance design

Derive stances from the stage table in SKILL.md, then TAILOR to the artifact — never copy a prior
panel's stances verbatim. Archetypes that recur:
- state-machine / concurrency / timing attacker
- invariant guardian (enumerate the system's invariants; try to construct a violation)
- implementation feasibility vs the REAL schema/code
- production-ops: deploy order, blast radius, rollback, logging
- product/UX + analytics: user-visible consequences
- completeness critic: "what is missing that no other lens covers?"

Include the four blind-spot lenses from SKILL.md where relevant (drive-the-real-thing, external
boundary/dependency swap, empty/null path, worst-case localized input).

## Context block — what EVERY panel agent receives

Proven load-bearing on real runs. Each agent's prompt carries, in this order (shared prefix first —
identical prefixes across agents maximize prompt-cache hits; put the stance-specific text last):

1. Artifact path(s) + repo root, and the instruction to read the artifact FIRST, then verify its
   claims against the real code. Ground findings in file:line evidence.
2. One-paragraph product/system context.
3. **Locked decisions — verbatim from the user** — with "do NOT re-litigate; attack correctness
   GIVEN them." Include the one-line technical mapping of each chosen option (SKILL.md,
   Answering Hard Questions).
4. Out-of-scope list ("do not report as missing").
5. **Already-verified facts — on EVERY panel launch, including the first** (SKILL.md handoff
   rules): author-verified claims as bare evidence (claim + file:line or command + observed
   output, no advocacy) under "confirmed; do NOT re-flag, go deeper or find NEW ones". Finders
   re-verify what they can reach and report what they cannot — "do NOT re-flag" is not "accept".
6. On re-panels: findings, deferrals and disputes carried from **ALL** prior passes (not just the
   previous one) with their latest rulings, plus factual fix descriptions, under "hunt NEW seams,
   including seams those fixes introduced".
6. Severity definitions matching SKILL.md, and "report only real findings — an empty list is a
   valid answer; no filler."
7. The stance: this agent's specific attack angle, naming concrete things to attack.

## Re-panels (NOT READY → fix → re-review)

- **New fresh agents every launch** — never resume or reuse the prior panel's agents.
- **Reduced size:** 2–3 lenses focused on the fixed areas (precedent: re-panels ran at 3 stances).
- Context block item 5 becomes mandatory.
- **Each re-panel launch needs its own user consent** (SKILL.md consent gate).

## Synthesis seat

A dispatched agent on the tier the USER chose for this launch — asked per dispatch, **alongside**
the consent question and never fused into it (a declined panel must not swallow the tier answer),
never inherited and never escalated un-asked (SKILL.md tier rules) — and never the author
(SKILL.md role rules). Its handoff carries the model
it was dispatched on and the session's tier, echoed in its Seat model line (SKILL.md output
format). **Kill vs OVERTURNED:** *killing* applies only to findings raised in THIS pass. A finding
carried forward from a prior pass is never killed — it gets a Carried-forward ruling instead, and
OVERTURNED is the kill equivalent there (SKILL.md output format). Routing a carried finding through
the kill path would sneak it off a list SKILL.md says it cannot leave.
Its job: verify each finding against the artifact (kill fabrications), merge true
duplicates conservatively, rank by severity, render the standard Findings / Hard Questions /
Verdict / Seat model output, and report counts (raw → unique → confirmed/killed, with reasons
for kills).

## Orchestration

**With a workflow/orchestration tool:** phases Find (parallel finders) → Synthesize (one agent).
Pass each agent a structured-output schema for findings when available so results compose without
parsing.

**Without one (fallback):** the same structure via parallel Agent/Task calls — dispatch all finders
in one batch, collect their reports, then dispatch the synthesis agent with the collected findings.
The structure, not the tooling, is what matters.

## Operational hardening (all of these happened in real runs)

- **Failure-aware reporting:** report how many finders actually returned. A panel that quietly
  shrank from 8 stances to 2 looks like "clean code" when it is an infra outage.
- **Minimum surviving stances:** if fewer than ~¾ of finders returned, say so prominently and do
  not present the result as full coverage.
- **Never read a low finding count as "clean"** without checking for silently-dropped agents.
- **One hung agent must not sink the run:** prefer per-agent timeouts / partial synthesis over
  waiting indefinitely; a single verifier stuck in a polling loop once discarded a whole run.
- **Salvage after an abort/stall:** completed agents' results usually survive on disk. The durable
  artifacts are the per-agent transcript files (e.g. `agent-*.jsonl` under the run's
  subagents/workflows directory) — a single `journal.jsonl` may NOT exist. Read what completed
  before re-running anything; on resume, cached phases are typically reusable.

## Report back

The synthesis output IS the review (SKILL.md output format, Seat model line included), plus panel
metadata. The CONTROLLER — the only party holding every seat's dispatch settings — appends:
stances run vs returned, and the model each seat was dispatched on (matched against the tier the
user chose at the consent question, per SKILL.md tier rules; the verdict seat's actually-run model
is confirmed by its own Seat model echo and mismatch flag). The synthesis seat reports: raw → unique → confirmed/killed
counts, and killed findings with kill reasons — the user should see what was filtered out, not
just what survived.
