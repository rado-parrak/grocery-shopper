---
phase: 02-quick-add-basket-review
plan: 03
subsystem: testing
tags: [claude-skills, trigger-evals, uat, packaging, mobile]

# Dependency graph
requires:
  - phase: 02-quick-add-basket-review (02-01)
    provides: skills/quick-add/SKILL.md — the description under eval in Task 1/3
  - phase: 02-quick-add-basket-review (02-02)
    provides: skills/basket-review/SKILL.md — the description under eval in Task 1/3
provides:
  - "skills/evals/quick-add-evals.md — 5 must-trigger + 4 near-miss phrasings guarding quick-add ↔ basket-review cross-firing"
  - "skills/evals/basket-review-evals.md — 5 must-trigger + 3 near-miss phrasings, mirrored"
  - "skills/README.md — zip-per-skill upload flow to claude.ai Settings → Features, Project Knowledge upload of the 8 project-knowledge/ contracts, connector scope, OAuth-only rule, explicit NOT-for-.claude/skills/ statement"
  - "skills/evals/uat-quick-add-basket-review.md — authored (not yet run) human UAT checklist measuring the ≤3-turn quick-add path + basket-review read-back"
affects: [phase-3-recipe-to-basket-meal-planning]

actuals:
  tokens: 4850
  tasks: 2
  commits: 3

tech-stack:
  added: []
  patterns:
    - "Manual (no-runner) eval-set markdown files as the trigger-collision guard, per research Pattern 1"
    - "Human UAT authored as a checklist doc with an explicit stated precondition, never fabricated as run"

key-files:
  created:
    - skills/evals/quick-add-evals.md
    - skills/evals/basket-review-evals.md
    - skills/README.md
    - skills/evals/uat-quick-add-basket-review.md
  modified: []

key-decisions:
  - "Eval sets are plain markdown test-case tables (no runner) — pass/fail is recorded by a human running each phrasing in a fresh claude.ai chat, per the plan's own framing (there is no automated skill-trigger harness available)."
  - "skills/README.md documents zip-per-skill (not one combined zip) uploads, since claude.ai's Settings → Features accepts one skill per upload."
  - "The UAT doc is authored only, not executed — this plan's Task 3 human-check verify step requires a household member on a real phone with the Rohlík OAuth connector and filled ⚠ FILL data, neither of which this executor session can provide or fabricate."

patterns-established:
  - "Trigger eval sets live under skills/evals/, one file per skill, always naming the sibling skill(s) that must NOT fire for each near-miss case."

requirements-completed: []  # UX-04 intentionally NOT marked complete — see status/pause note below; only marked complete once the household reports a UAT pass.

coverage:
  - id: D1
    description: "Trigger eval sets (quick-add + basket-review) proving disjoint cross-skill triggering, ≥3 must-trigger + ≥2 near-miss phrasings each, incl. Czech"
    requirement: "UX-04"
    verification:
      - kind: other
        ref: "grep gate: skills/evals/quick-add-evals.md + skills/evals/basket-review-evals.md contain required phrasings (Task 1 <verify><automated>, ran and passed — EVALS_OK)"
        status: pass
    human_judgment: false
  - id: D2
    description: "skills/README.md packaging/upload guide (zip → claude.ai Settings → Features, Project Knowledge upload, NOT for .claude/skills/, OAuth-only)"
    verification:
      - kind: other
        ref: "grep gate: skills/README.md contains Settings→Features/zip, .claude/skills/, OAuth/credential/token (Task 2 <verify><automated>, ran and passed — README_OK)"
        status: pass
    human_judgment: false
  - id: D3
    description: "Human UAT procedure authored (uat-quick-add-basket-review.md) measuring the real ≤3-turn quick-add path and basket-review read-back on a phone"
    requirement: "UX-04"
    verification:
      - kind: other
        ref: "grep gate: skills/evals/uat-quick-add-basket-review.md contains required checklist markers (Task 3 <verify><automated>, ran and passed — UAT_DOC_OK)"
        status: pass
    human_judgment: false
  - id: D4
    description: "The household actually runs the UAT on a real phone against the real shared Rohlík basket and reports pass/fail per checklist item — the only place UX-04 (≤3-turn measurement) is truly provable"
    requirement: "UX-04"
    verification: []
    human_judgment: true
    rationale: "Requires a real phone, the live Rohlík OAuth connector, and household-ruleset.md/budget.md ⚠ FILL data completed by the household — none of which this executor session can perform or fabricate. This plan is PAUSED at exactly this task; see status below."

duration: 22min
completed: 2026-07-31
status: paused
---

# Phase 2 Plan 3: Trigger Evals + Packaging README Summary

**Authored the quick-add/basket-review trigger eval sets, the skills/README.md packaging guide, and the human UAT checklist — PAUSED at the human-check task awaiting a real-phone run.**

## Performance

- **Duration:** 22 min
- **Started:** 2026-07-31T20:15:00Z (approx.)
- **Completed:** 2026-07-31T20:37:00Z (agent-doable tasks; plan itself is paused, not complete)
- **Tasks:** 3 of 3 authored; Task 3's human-check verify step NOT yet run
- **Files modified:** 4 (all new)

## Accomplishments

- **Task 1 — Trigger eval sets.** `skills/evals/quick-add-evals.md` (5 must-trigger phrasings incl.
  Czech + mixed CZ/EN, 4 cross-skill near-misses naming basket-review) and
  `skills/evals/basket-review-evals.md` (5 must-trigger incl. two Czech, 3 near-misses naming
  quick-add) — each proving the D-03 disjoint trigger vocabulary in a manual, no-runner test-case
  format. Both pass their automated grep gates (`EVALS_OK`).
- **Task 2 — Packaging README.** `skills/README.md` documents the zip-per-skill upload flow to the
  claude.ai Project's Settings → Features, the separate Project Knowledge upload of the eight
  `project-knowledge/` contracts, the This-project Rohlík connector scope, and states explicitly
  and prominently that these skills are **not** for `.claude/skills/` (that's Claude Code's own
  execution surface, occupied by GSD). Restates the OAuth-only / no-credential rule. Passes its
  automated grep gate (`README_OK`).
- **Task 3 — UAT procedure authored, execution PAUSED.** `skills/evals/uat-quick-add-basket-review.md`
  is a full step-by-step checklist for a household member to run on a real phone: English + Czech
  "add milk and bananas" quick-add turns, confirmation artifact checks, ≤3-turn measurement,
  correct-Czech-product + Czech-audit verification, then a `basket-review` read-back with budget
  state and manual-checkout handoff. Its stated precondition (Rohlík OAuth connector set up +
  `household-ruleset.md` §A/§B + `budget.md` ⚠ FILL data completed) is unmet in this environment —
  **the plan stops here.** This executor did **not** run the UAT and did **not** fabricate a
  pass/fail result.

## Task Commits

Each task was committed atomically:

1. **Task 1: Trigger eval sets — 3+ phrasings per skill, cross-skill near-misses** - `bf344df` (test)
2. **Task 2: Packaging README — zip-and-upload flow, not for .claude/skills/** - `61034a2` (docs)
3. **Task 3: Author the human UAT procedure** - `1903c74` (docs) — authoring only; the human-check
   run itself has not happened (see below)

**Plan metadata:** committed separately after this SUMMARY (docs: complete/pause 02-03 plan)

## Files Created/Modified

- `skills/evals/quick-add-evals.md` - 5 must-trigger + 4 near-miss phrasings for quick-add
- `skills/evals/basket-review-evals.md` - 5 must-trigger + 3 near-miss phrasings for basket-review
- `skills/README.md` - zip/upload packaging guide, not-for-.claude/skills/, OAuth-only rule
- `skills/evals/uat-quick-add-basket-review.md` - the authored (not-yet-run) human UAT checklist

## Decisions Made

- Eval sets are manual markdown test-case tables — there is no skill-trigger test runner available
  to this project, so "pass" is recorded by a human running each phrasing in a fresh claude.ai chat
  against both uploaded skills.
- README documents per-skill zips (two separate uploads), matching claude.ai's one-skill-per-upload
  Settings → Features flow, rather than one combined zip.
- The UAT doc's precondition (connector + ⚠ FILL data) is stated explicitly and checked first in the
  document itself, so the household doesn't attempt a real-basket run against placeholder data.

## Deviations from Plan

None — plan executed exactly as written through the point of the mandatory human checkpoint. No
Rule 1-4 auto-fixes were needed; all three tasks' automated `<verify>` gates passed on first
attempt.

## Issues Encountered

None. The only "issue" is expected and by design: Task 3's `<human-check>` verify step requires a
real phone, the live Rohlík OAuth connector, and completed `⚠ FILL` household data — none available
in this execution environment. This is not a deviation; it is the plan's designed stopping point
(see `<critical_human_checkpoint_rule>` in this plan's PLAN.md and the executor's own instructions).

## Threat Flags

None found — no new network endpoints, auth paths, or schema changes were introduced; all four
files are pure markdown authoring (per this plan's own `<threat_model>`, T-02-SC "accept: no
package installs").

A token-shaped-string scan (`sk-...`, `bearer ...`, `token: ...` patterns) was run over all four
authored files before each commit — no matches found (mitigates T-02-11).

## User Setup Required

**Yes — this is the entire remaining blocker for this plan and this phase.** Before the UAT can be
run:

1. Set up the Rohlík MCP OAuth connector at **This-project** scope on the shared Claude account
   (see `skills/README.md`'s "Rohlík MCP connector" section).
2. Fill `project-knowledge/household-ruleset.md` §A (allergies, dislikes) and §B (milk/brand
   preferences), and `project-knowledge/budget.md`'s soft/hard CZK amounts — replace the `⚠ FILL`
   placeholders with real household values.
3. Zip and upload `skills/quick-add/` and `skills/basket-review/` to the shared claude.ai Project
   (Settings → Features), and upload all `project-knowledge/*.md` files as Project Knowledge, per
   `skills/README.md`.
4. Run `skills/evals/uat-quick-add-basket-review.md` end to end on a phone, in the shared Project.
5. Report the pass/fail result per checklist item back into a chat/session that can resume this
   plan (the exact resume signal — see below).

## Next Phase Readiness

**This plan (02-03) and Phase 2 are PAUSED, not complete.** `UX-04` remains **pending** — it is
NOT added to `requirements-completed` above, and `.planning/STATE.md` / `.planning/ROADMAP.md` are
updated to reflect "paused at human UAT," not "complete." Phase 3 (Recipe-to-Basket & Meal
Planning) depends on Phase 2 per `.planning/ROADMAP.md` and should not be started until this UAT is
run and reported.

**Resume signal:** re-invoke plan 02-03 (or its continuation) once the household has run
`skills/evals/uat-quick-add-basket-review.md` on a real phone and can paste back, for each
checklist section (Part 1 English, Part 1 Czech, Part 2 basket-review, overall UX-04), a pass/fail
result and any notes. On resume: if UX-04 passed, mark it complete in `REQUIREMENTS.md`, update this
SUMMARY's `coverage` D4 entry and `requirements-completed`, and mark Phase 2 complete. If it failed,
treat the specific failing checklist item as a Rule 1-4 deviation against `skills/quick-add/SKILL.md`
or `skills/basket-review/SKILL.md` (most likely fixes: tightening trigger vocabulary per D-03, or a
bug in the resolve/confirm/write/read-back/audit chain) before re-running.

---
*Phase: 02-quick-add-basket-review*
*Completed: 2026-07-31 (agent-doable tasks only — plan paused at human UAT)*

## Self-Check: PASSED

All 4 created files confirmed present on disk; all 3 task commit hashes (`bf344df`, `61034a2`,
`1903c74`) confirmed present in git log.
