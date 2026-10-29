---
name: adversarial-review
description: Runs a ruthless, severity-ranked review that assumes the artifact is broken until proven otherwise, then returns findings + a binary ready/not-ready verdict. Use proactively before approving, locking, shipping, merging, or finalizing ANY artifact (spec, plan, design, architecture, code, PR, doc, config, contract), after finishing each task/phase of a plan, and whenever tempted to say "looks good" / "this is ready" / "good enough" - even if the user did not explicitly ask for a review. If something is about to be signed off, use this first.
---

# Adversarial Review

## Overview

A ruthless review pass that assumes the artifact is flawed until proven otherwise. The job is to find what is wrong BEFORE it costs real time or money downstream. Flattery is failure. Works on anything reviewable: specs, plans, designs, architecture, code, PRs, docs, configs, contracts.

**Core principle: you did not write it and you have no ego in it. Default assumption is that it is broken.**

## When to Use

- Before approving, locking, shipping, merging, or finalizing ANYTHING.
- The moment you catch yourself thinking "looks good", "this is ready", or "good enough".
- After finishing each task/phase of a plan, before committing or moving on.
- When someone asks "is this done?" / "any problems?" / "review this".
- Whenever a first draft is about to be accepted as final.

Do NOT skip because the artifact "seems simple" - simple artifacts hide unexamined assumptions, which is where the expensive failures live.

## Who Reviews - Independence Is Structural

"No ego in it" is enforced by WHO reviews, not by trying harder. Run the role check first:

1. **Dispatched to perform this review?** Execute the contract in this file directly. Do NOT dispatch a further reviewer.
2. **Asked to review something you did NOT build?** You ARE the fresh reviewer - review it directly; do not dispatch.
3. **Did you build (any part of) the artifact?** You are the AUTHOR. Dispatch a fresh agent to review; never review - or synthesize the review of - your own work.

Author rules (non-negotiable):
- Your own pass over your own work is prep, not review. It never counts toward the verdict, and doing it does NOT make dispatch optional. "I'll review it myself first, then also dispatch for a second opinion" - no: the dispatched review IS the review; yours was never one.
- Dispatch is unconditional, not "given the stakes". Stakes decide panel-vs-single (below), never whether independence applies.

**Handoff - what the dispatched reviewer receives:**
- The shipped artifact itself: the actual diff/code/doc under sign-off. Never a spec or description standing in for it - reviewing the spec while the shipped code goes unread is the most common real-world review failure.
- The user's locked decisions VERBATIM - AskUserQuestion-derived decisions accompanied by their one-line technical mapping (see Answering Hard Questions) - and the out-of-scope list.
- The review contract: a dispatched agent does not inherit this skill - tell it to read this skill file (or paste the contract into its prompt).
- The model the seat was dispatched on AND the session's own tier (or "same") - the seat echoes both in the Seat model line. The seat never guesses its model; but if its own system record names a different model than the handoff states (an ignored or overridden model param), it flags the mismatch in that line rather than suppressing it.
- **Already-verified facts - on EVERY pass, including the first** (seed the reviewer to go deep, not wide-and-shallow): the claims you checked against reality while building, as bare evidence - claim + file:line or command + observed output, no advocacy - under "these are confirmed; do NOT re-flag them, go deeper or find NEW ones." A pass that re-reports what you already knew was spent, not used. Seed only what you actually verified: listing an assumption here to get it waved through is the author-serving framing the reviewer is told to flag. **Prefer seeds the reviewer can check inside the artifact under review** - a seed pointing somewhere it cannot reach is unfalsifiable by construction.
- **Reviewer's duty on seeds:** "do NOT re-flag" is not "accept". Re-verify what you can reach; Ground Every Claim outranks the seed, and a seed your check contradicts is a finding, not a protected fact. **Account for EVERY seed in the required `## Seeds checked` section** (see Output Format) - one line each, RE-VERIFIED / CONTRADICTED / UNREACHABLE. Measured: when this was only stated here and not a required output section, agents complied about half the time - they reliably killed a false seed but silently skipped reporting the ones they could not reach. An unreachable seed is **load-bearing** when the artifact's correctness depends on it - if it were false, a finding would exist that otherwise does not. A load-bearing unreachable seed becomes an open hard question **only if nothing in the artifact corroborates it**; otherwise report it as unverified in the seed block and move on. Reporting an unverified seed is not the same as blocking on it: an author who verified something by actually running the system (which this skill demands) must not be punished with an automatic NOT READY. Silently accepting an unchecked seed is the one way this rule can launder an assumption past you.
- On re-review passes only: prior findings + factual fix descriptions (what changed, where), plus author-verified answers to prior checkable-fact hard questions in the same bare-evidence form - the reviewer re-verifies them. **Include every deferred and disputed finding from ALL prior passes** (Loop step 2), each with its stated reason and latest ruling - not just the immediately preceding pass's, or a pass-1 deferral ruled on at pass 2 silently vanishes at pass 3. A deferral the next reviewer never sees is a deletion; a disputed finding is exactly what the fresh reviewer is there to adjudicate. Overturned findings stop travelling (see Carried forward).
- NEVER your justifications for why the artifact is good - that is the contamination independence removes. **Bare evidence is not a justification**, and neither the *Already-verified facts* bullet nor the *re-review passes* bullet above is an exception to this one: evidence is a checkable claim the reviewer can re-verify and overturn; a justification is an argument for a conclusion, which it cannot check. If you catch yourself writing *why* something is right rather than *what* you observed, it is contamination - cut it.
- Reviewer duty: if the handoff framing looks author-serving (e.g. the out-of-scope list conveniently excludes the riskiest area), flag it as a finding.

**Cannot dispatch?** (no Agent tool, nested subagent, permission denied): run the contract inline and label the verdict **"non-independent (author-reviewed)"**, recommending a fresh-agent pass before final sign-off.

## When to Escalate to a Panel

Two triggers - **either** one means recommend a panel; **neither** means one fresh reviewer:

- **Risk:** hard-to-reverse or high blast-radius - multi-tenant/authz, data-loss, external send paths, schema/migration, security, dependency/infra swap.
- **Size:** big/multi-surface - a large feature spec, many moving parts. (A genuinely huge spec: review in sections, not one skimmed pass.)

**Consent gate - consent is PER PANEL LAUNCH:**
- Never launch a panel un-asked. Recommend it (stance count + model tier + one-line cost estimate) and WAIT.
- "Review this" / "make sure it's solid" is a review request, NOT panel consent - even if a trigger fires.
- "It's read-only", "it's reversible", "it's what they wanted", "the user is away" do NOT create consent.
- Consent = the user names a panel / multi-agent / multiple-reviewers pass in their own words, or answers yes to the recommendation.
- Declined or no answer? Proceed with one fresh reviewer now and note the standing panel recommendation in the verdict.
- Re-panels in the same fix loop each need a fresh yes.

**Panel rules** (full playbook: read `references/panel-workflow.md` in this skill's directory BEFORE assembling a panel):
- The synthesis/verify seat is a DISPATCHED agent - never the author. "Only I can judge the findings, since I act on them" is self-review through the back door: acting on findings is yours; judging them is not.
- Re-panels use NEW fresh agents (never reuse the prior panel), reduced to 2-3 lenses on the fixed areas.

**Model tier - ASK, never assume.** The strongest tier costs real money on every review, so which model a review runs on is the USER's call, not yours. The **verdict-rendering seat** - the solo reviewer, or the panel's synthesis - is the one this decides; finder seats may run cheaper, and dedup may drop a tier, with "when in doubt, don't merge". Before dispatching, ask - one AskUserQuestion, recommendation first with its reason in the description:

- **Show the classification, don't just assert it - for BOTH triggers.** Name the Risk trigger that fires (authz, data-loss, external send paths, schema/migration, security), **or** name the Size measure that fires with its concrete number (surfaces touched, files, spec sections), **or** state that neither does and list what you checked. Recommend the strongest available tier when either fires; recommend the session's own model otherwise. Size is the softer trigger and therefore the easier one to wave at - "it's large" with no number is not a shown classification, and it is a clean route back to always-max spending - most reviews are routine and the session tier is enough. The user is being asked to fund YOUR risk assessment, and you are the author: show the work so a wrong call is visible to them, since no reviewer exists yet to catch it.
- **Enumerate the dispatch tool's actual model options** rather than assuming you know them; an unfamiliar option name is likely NEWER than your training data, so list it as a real choice. Unfamiliar-therefore-strongest is a reason to OFFER it, never to pick it silently. **When you must name which is strongest** (a trigger fired, so the recommendation depends on the ordering) and the ordering is genuinely unknown, verify it - provider docs, a model-reference skill - rather than resolving by heuristic alone. A "(Recommended)" label on a guessed ordering is worse than no recommendation.
- **Plain words, one exception.** Answering Hard Questions requires jargon-free options; model names are unavoidable jargon here, so name each option by what it COSTS and BUYS ("the strongest tier - slowest and most expensive, worth it because this touches the send path") rather than by model name alone. That is the only carve-out from the plain-words rule in this file.
- **No answer, or the user is unreachable? Run on the session's model**, and mark it in the Seat model line as **defaulted, not chosen**. Never escalate un-asked - an un-asked escalation spends the user's money on a decision they never made. A silent default that reads like a choice defeats the whole point of recording the tier.
- **Asked before EVERY dispatch, including every re-review pass.** This is a deliberate exception to the locked-decisions rule: a tier answer does NOT carry forward, because a fix loop's risk profile changes between passes and each dispatch is its own spend. Keep the question cheap - one question, and the no-answer fallback means it never blocks.
- **One question per dispatch, not per seat.** A panel asks once for the whole launch. Ask it **alongside** the panel consent question, never fused into it: consent and tier are two judgment calls, and a declined panel must not swallow the tier answer. If the panel is declined and the tier was answered, the solo reviewer runs on the answered tier.
- Whatever is chosen is passed **explicitly** at dispatch - never left to inheritance - and handed to the seat for its Seat model echo (see the handoff list).
- Applies to DISPATCHED seats. A reviewer the user addressed directly runs at whatever tier the user chose. A dispatched reviewer never asks - it cannot reach the user; only the controller asks (same reach rule as Answering Hard Questions).

## What to Attack - Stage Lenses

Match the lenses to the artifact's stage. Artifacts spanning stages take the union of rows (a UI change arriving as a PR = Code/PR + UI).

| Stage | Attack |
|-------|--------|
| **Discovery / problem-framing** *(thin evidence)* | Is the problem real or assumed? Is the evidence real? Attack the problem, not a solution. |
| **Design / spec** | Mechanism feasibility vs the REAL code · invariants & safety · data-model/migration · scope/feasibility · UX-vs-audience · a completeness critic ("what did every other lens miss?"). |
| **Plan (implementation)** | Does its code compile against real types/schema/imports? Honest tests (fail-then-pass, not tautology)? Coverage, sequencing, task independence. |
| **Code / PR** | Line-by-line correctness · removed behavior · cross-file tracing · framework pitfalls · security/authz/IDOR · invariants · fail-loud + tests · i18n/RTL · migration/schema. |
| **UI** | Behavior preservation · RTL/i18n · perf · a11y · render and drive the real thing. |

**Wildcard lens (mandatory on every review):** also ask *"what angle does THIS artifact demand that no list names?"* The table is a floor, not a ceiling.

**Four blind-spot lenses** - each one's absence let a real bug ship; apply where relevant:

- **A. Drive the real rendered artifact** - render and interact with the real thing, worst-case/real content, smallest supported viewport; assert primary CTAs stay visible and hit-testable.
- **B. External boundary / dependency swap** - does the outside system actually accept/render what we send? After a component/library/action swap, does carried-over config still connect end-to-end?
- **C. Empty/default/null path** - trace the exact payload when fields are UNFILLED: null vs undefined, client defaults clobbering server fallbacks - not just the fully-populated happy path.
- **D. Worst-case localized input** *(provisional - one confirmed case)* - run the localization lens over parsers/importers with the messiest real file, not just rendered strings.

**Run/drive it, don't just read it:** any artifact with a runtime surface gets exercised for real. Reading, types, and unit tests structurally cannot see runtime crashes, layout overlap, or lost hit-targets.

## The Iron Rule

**Never return SHIP IT on the first pass unless the artifact is genuinely airtight.** Most first passes end in NOT READY. If you found nothing, you did not look hard enough - look again before concluding.

**Violating the letter of this rule is violating its spirit.** "It's basically fine" is not a verdict.

## Ground Every Claim

Review the artifact as the single source of truth a builder would implement from. But verify its claims against reality where you can: read the referenced code, check the cited file/line, confirm the API exists, run the numbers. A finding grounded in "I checked, the function is actually named X" beats a vague "this might be wrong."

## Attack Across These Axes

1. **Holes** - What is missing? Unstated assumptions, undefined terms, unhandled cases, absent error/edge/failure behavior, missing acceptance criteria.
2. **Logic bugs** - Contradictions, circular dependencies, steps that cannot work as described, ordering problems, race conditions, wrong data flow.
3. **Ambiguity** - Anything a builder could interpret two ways. Name each one.
4. **Scope & feasibility** - Over-engineering, under-specification, hidden complexity, unrealistic constraints, costs that will balloon.
5. **Hard questions** - The uncomfortable questions whose answers might invalidate the whole approach. Ask them. Do not soften them.

## Answering Hard Questions - who answers what

Every hard question must be answered before SHIP IT. Split by who owns the answer:

- **Checkable facts** ("does the code/API/schema really do X?"): the author answers by verifying against reality (read the code, run it) - never by guessing. The answer travels to the re-review as bare evidence (claim + file:line or command + observed output, no advocacy); the fresh reviewer re-verifies it per Ground Every Claim before treating the question as closed.
- **Judgment calls** (scope, risk acceptance, product trade-offs, "do we even want this?"): the USER answers. Never decide these silently on the user's behalf.
- **Mixed questions** (part fact, part judgment): verify the factual part first, then bring only the remaining judgment to the user.

Bringing judgment calls to the user is the CONTROLLER's job - the main agent in the conversation with the user (usually the author, who dispatched the review). The rule keys on reach, not role: any reviewing agent that cannot reach the user (a dispatched reviewer, a nested subagent) only lists the judgment questions in its report, and the controller relays them via AskUserQuestion. A reviewer that IS in the conversation with the user (addressed directly, or the main agent in the cannot-dispatch inline case) is its own controller and asks the user itself. When the author is not an agent in the conversation (user-authored or external artifact), the controller verifies the checkable facts itself.

- Ask via the AskUserQuestion tool, one question per judgment call, 2-4 concrete options each.
- Always include a recommendation: put it FIRST, label it "(Recommended)", and give the one-line reason in its description.
- Phrase the question and every option in simple, everyday words - as if explaining to a smart 14-year-old. No jargon, no code terms; describe user-visible consequences ("some people could get the same survey twice"), not mechanisms.
- The user's answers become LOCKED DECISIONS: hand the question-and-answer pair verbatim to the next review pass with "do not re-litigate", plus a one-line technical mapping of what the chosen option means in the artifact ("this option = X at Y").
- Answers alone never flip the verdict: even when the artifact is unchanged and the answers closed every open question, the move to SHIP IT comes from a NEW fresh re-review pass (handed the locked decisions), per the Loop.

## Output Format

```
## Findings
(ordered by severity, blockers first. Each finding is one block:)
- **Severity:** Blocker | Major | Minor
- **What:** the problem in one sentence
- **Where:** the section / file / line it lives in
- **Why it matters:** the concrete downstream consequence
- **Fix:** a specific, actionable correction (NOT "consider improving")

## Hard Questions
(numbered; the questions that must be answered before this is buildable/shippable)

## Seeds checked
(REQUIRED whenever the handoff supplied already-verified facts - one line per seed, and
 never silence about one you skipped. Exactly one of:
   RE-VERIFIED   you checked it yourself and it holds
   CONTRADICTED  your check disagrees with it - it is now a FINDING above, not a seed
   UNREACHABLE   you could not check it; say WHY (no access to that file/system/run),
                 and whether anything in the artifact corroborates it
 "None supplied" when the handoff carried no seeds. An unlisted seed reads as verified
 when nobody verified it - that is how an assumption gets laundered past a review.)

## Carried forward
(every deferred/disputed finding from ALL prior passes, one line each: the author's
 stated reason, then your ruling - exactly one of
   UPHELD          the finding stands; it is live again at its original severity
   AGREED-DEFERRED real, but acceptable to leave for now; keeps travelling
   RESOLVED        a carried deferral the author has since FIXED; confirm it landed in
                   the artifact yourself, then it stops travelling
   OVERTURNED      the author's dispute was right, the finding was wrong; give the
                   reason and it stops travelling
 Empty only when the handoff genuinely supplied none. A deferral nobody restates is a
 deletion, so say "none supplied" rather than omitting the section.)

## Verdict
(exactly one:)
- NOT READY - at least one Blocker or Major, OR any open hard question. (Default.)
- SHIP IT AS IS - genuinely nothing left worth changing: no Blockers, no Majors other
  than those covered by the exceptions below, no unanswered hard questions, remaining
  Minors not worth another revision.
(What does NOT block SHIP IT: documented Minor deferrals; an above-Minor deferral the
 USER accepted, handed to you as a locked decision - list it in the verdict, do not
 re-litigate it; and anything OVERTURNED.
 What DOES block: a dispute you UPHELD - it is a live Blocker/Major again, and the
 author's disagreement does not lower it.)

## Seat model
(always present, one line. Echo BOTH handoff values, and say how the tier was set -
 a defaulted tier must never read like a chosen one:)
- same tier:   "dispatched on X - session same (user chose)"
- escalation:  "session on X - verdict seat escalated to Y (user chose)"
- downgrade:   "session on X - seat ran cheaper on Y (user chose)" - expected, not an
               error: the user may pick a tier cheaper than the session's. (Finder and
               dedup seats also run cheap, but they do not render this line.) "(user
               chose)" is never stamped on a tier nobody chose - see defaulted.
- defaulted:   "dispatched on X - session same (DEFAULTED, user not asked/no answer)"
               - the tier rule's fallback fired. Say so; never dress it as a choice.
- non-dispatched seat (inline author-review fallback, or a reviewer the user
  addressed directly): the session model from the session's own system record,
  labeled "(not dispatched - session model)".
- If the seat's own system record contradicts the handoff, name the mismatch loudly.
```

## Severity Definitions

| Severity | Meaning |
|----------|---------|
| **Blocker** | Wrong/contradictory/unbuildable as written. Cannot proceed. |
| **Major** | Will cause real downstream damage (cost, rework, incorrect behavior) if unfixed. |
| **Minor** | Real but localized; worth fixing, does not reopen the design. |

## The Loop (review -> revise -> re-review)

One pass is rarely enough. Run the cycle until it converges:

1. Review -> findings + verdict.
2. **TRIAGE before fixing.** A findings list is NOT a to-do list. Treating every finding as a mandate is how a review *causes* the over-engineering it exists to catch - and how a fix loop runs for rounds without shipping.
   **Verify first, then sort.** A finding whose severity turns on a checkable fact ("does `normalizePhone()` really strip the leading zero?") has no bucket until you check - fix-now and disputed are the same finding under two different answers. Verify those against reality FIRST (Answering Hard Questions), then bucket on what you found. Never bucket a fact you have not checked. **Verify-first chooses only between fix-now and disputed** - it is never a route to lowering a severity so you can defer it yourself (see the severity guardrail below).
   Then sort EVERY finding into exactly one bucket, and write the bucket next to it:
   - **Fix now** - real Blocker/Major. It blocks the verdict.
   - **Fold in** - cheap and you are already touching that code. Do it silently: *silently* means it never earns its own review round, NOT that it goes unreported. Fold-ins still appear in the next handoff's fix descriptions - an undocumented edit in the diff is a seam the reviewer cannot trace.
   - **Defer and document** - real, but the fix costs more than the risk it currently carries. Name why it is acceptable *now* (scale, stage, blast radius) and write it somewhere it will be found again. This is the default home for a Minor you are not fixing.
   - **Disputed** - you believe the finding is WRONG. Never silently drop it: record the claim plus your evidence (file:line, or command + observed output - bare, no advocacy) and hand BOTH to the next fresh reviewer, who decides. You do not get to overrule a reviewer about your own work.
   - **User's call** - genuine product/scope/risk-acceptance judgment. Goes to the user per Answering Hard Questions.
3. If NOT READY: the author (you or another agent) fixes the fix-now and fold-in findings.
4. Re-review the revised artifact - and specifically hunt for issues the FIX introduced (fixes create new seams).
5. Repeat until SHIP IT.

**Triage guardrails** (each one closes an observed failure):
- **Deferring a Blocker or Major is the USER's call, never yours.** You may defer Minors on your own judgment, documented. Anything above Minor that you want to defer becomes a "user's call", not a deferral you grant yourself - otherwise triage becomes a way to wave the verdict through.
- **"Defer and document" is not "ask the user".** A Minor whose fix costs more than its risk is a deferral you make and record - not a question you spend the user's attention on. "Should we rate-limit a pre-launch endpoint with no users" is a documented deferral; "do we want a preview screen before import" is a product decision. Route on *what kind of thing it is*, not on how unsure you feel.
- **Deferred and disputed findings survive the loop.** Both travel to EVERY later pass in the handoff - carried cumulatively from all prior passes, not just the last one - and both are listed in the final verdict. Triage decides what you act on now; it never deletes a finding. **Two exits, both ruled by a fresh reviewer, never by you:** a dispute ruled OVERTURNED (you were right, the finding was wrong), or a deferral the reviewer confirms you have since FIXED (RESOLVED). Nothing else leaves the list - and a deferral you later fix must be ruled off it, not left to accumulate as a dead entry the reviewer keeps re-reporting as live.
- **Severity is the reviewer's, not yours.** Reclassifying a Major as a Minor so you can defer it yourself is the loophole the rule above is written against. If you think the severity is wrong, that is a **dispute** - evidence to the next fresh reviewer, who rules.

Re-review must first confirm each prior fix actually LANDED in the artifact (read it - never take fix notes on faith) before hunting new seams; severity oscillates when fixes are assumed instead of verified.

Re-review mechanics under the role rules: **EVERY re-review pass is a NEW fresh reviewer** — never continue the prior reviewer, at either rung (owner's rule: fresh context every pass). The fresh reviewer is handed the prior findings, the factual fix descriptions, the fix diff, and any fact-answer evidence (permitted context per the handoff rule); continuity lives in that handoff, not in reusing the agent. After a panel: a new panel of fresh agents, per the panel rules above. "The previous reviewer already has the context, it's cheaper to continue it" - no: it is also anchored on its own findings; fresh eyes plus the written handoff is the rule.

Track convergence honestly: "pass 1: 4 blockers -> pass 2: 2 majors -> pass 3: 1 seam -> pass 4: clean" is a healthy trajectory. If severity is not dropping, the fixes are not real.

## Rules (non-negotiable)

- Do NOT fix or rewrite the artifact in a review pass. Review it. (Fixing is a separate step.)
- Do NOT invent praise to balance criticism. No "great work, but...".
- Do NOT manufacture filler findings to look thorough. If a pass is genuinely clean, say so and give SHIP IT.
- Tempted to say "looks good"? Look harder first - find the holes, then decide.

## Red Flags - you are rationalizing, STOP

| Thought | Reality |
|---------|---------|
| "This is too simple to need review" | Simple = unexamined assumptions. Review it. |
| "Looks good to me" | You have not attacked it yet. Run the axes. |
| "I'll just approve to keep momentum" | A downstream blocker costs more than a review pass. |
| "I don't want to be harsh" | Flattery is failure. The artifact is the target, not the author. |
| "SHIP IT on the first pass" | Almost never true. Look again. |
| "I found a few things, good enough" | Order by severity and finish the sweep before the verdict. |
| "Re-review is overkill after a fix" | Fixes introduce new seams. Re-review hunts exactly those. |
| "I'll review my own work first, then dispatch too" | Your pass is prep, not review. Dispatch IS the review. |
| "Dispatching is only for high-stakes changes" | Independence is unconditional. Stakes pick panel-vs-single. |
| "The panel is read-only/reversible, so no need to ask" | Consent is about spend and control, not reversibility. Ask. |
| "The user said 'make sure it's solid' - that's consent" | That is a review request. Panel consent names a panel. |
| "Only I can synthesize - I have to act on the findings" | Acting is yours. Judging is not. Dispatch the synthesis seat. |
| "I know the code, reviewing the spec is enough" | The shipped artifact is the target. Hand over the diff. |
| "The prior reviewer has the context - just continue it" | It is anchored on its own findings. New fresh reviewer + written handoff, every pass. |
| "The strongest model is safer, so just dispatch on it" | It costs the user real money on every single review. The tier is their call. Ask. |
| "Escalating the tier is fine as long as I say so in the report" | Announcing a spend is not being granted it. Un-asked escalation is the thing to avoid. |
| "The findings list is what I have to fix" | Triage first. A findings list is not a to-do list; mandating all of it is how reviews cause over-engineering. |
| "I'm sure this finding is wrong, I'll drop it" | Disputed, not deleted. Your evidence plus the finding go to the next fresh reviewer - they rule, not you. |
| "I'll defer this Major, it's not that risky right now" | Deferring above Minor is the user's call. Yours is Minors only. |
| "Better ask the user about this Minor to be safe" | Trivia is a documented deferral. Their attention is for real judgment calls. |
| "Handing over facts I verified would contaminate the reviewer" | Bare evidence is checkable and overturnable. Justifications contaminate; observations don't. Seed them. |
| "The handoff says this is confirmed, so I'll skip checking it" | Ground Every Claim outranks the seed. Check what you can reach; report what you couldn't. |
| "It's really only a Minor, so I can defer it myself" | Severity is the reviewer's call. Disagree? That's a dispute, not a reclassification. |
| "I already asked about the tier last pass" | Tier is asked per dispatch. It is the one answer that deliberately does not carry forward. |

## Common Mistakes

- **Stopping at the first issue.** Sweep all five axes before concluding.
- **Vague fixes.** "Add error handling" is not a fix. Name the case and the correction.
- **Reviewing only what is written, never reality.** Verify cited code/APIs/numbers.
- **Softening hard questions.** The ones that might invalidate the approach are the point.
- **Declaring victory early.** Severity must actually reach zero, not "close enough".
- **Reviewing the description instead of the artifact.** The diff/code/doc under sign-off is the target.
- **Forgetting the reviewer doesn't have this skill.** Dispatched agents must be told to read it.
