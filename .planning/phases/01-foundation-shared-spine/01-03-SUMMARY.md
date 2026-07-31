---
phase: 01-foundation-shared-spine
plan: 03
subsystem: infra
tags: [artifacts, storage-capability, spike, project-knowledge, writable-state, checkout-correction]

# Dependency graph
requires:
  - phase: 01-foundation-shared-spine (01-01)
    provides: the five shared-internal contracts and the household-ruleset.md/budget.md config files this plan personalises
  - phase: 01-foundation-shared-spine (01-02)
    provides: the live MCP round-trip finding that Rohlík favourites/order-history persist cross-device, and the checkout-exposure correction that needed ratification in PROJECT.md/CLAUDE.md
provides:
  - spikes/artifact-storage-spike.md — Step-0 existence check first, then Device-A write / Device-B read-back protocol; marked SKIPPED (superseded), not fabricated as run
  - project-knowledge/writable-state-decision.md — RESOLVED (provisional, human-revisable): writable state = Rohlík-native + hand-edited Project-file diffs; artifact storage NOT used in v1
  - .planning/PROJECT.md and .claude/CLAUDE.md — checkout-constraint language corrected from platform-structural to policy-enforced, requirement preserved
affects: [phase-4 (FOUND-09 gate — staples-restock, household-prefs, any learned-state skill binds to the Rohlík-native + hand-edited-diffs path)]

actuals:
  tokens: 3500
  tasks: 2
  commits: 3

tech-stack:
  added: []
  patterns:
    - "Human-delegated decisions are recorded as provisional/human-revisable, not fabricated as a definitively-tested outcome — the decision doc states plainly that the spike was skipped, not run, and remains re-runnable if reopened"
    - "A false structural/platform claim discovered mid-project is corrected at its source (PROJECT.md, CLAUDE.md) with the underlying requirement strengthened (policy + forbidden-tools list) rather than weakened"

key-files:
  created: []
  modified:
    - project-knowledge/writable-state-decision.md
    - spikes/artifact-storage-spike.md
    - .planning/PROJECT.md
    - .claude/CLAUDE.md

key-decisions:
  - "Household delegated the writable-state decision (chose to proceed on the Rohlík-native path rather than run the two-device artifact-storage spike). Recorded as provisional/human-revisable, dated 2026-07-31, resolving FOUND-09."
  - "Rationale for skipping the spike: the live MCP round-trip (01-02) already confirmed Rohlík-native favourites/order-history persist cross-device on the one shared account, independently answering the cross-device question; the artifact storage capability likely doesn't exist in the current runtime anyway."
  - "household-ruleset.md and budget.md ⚠ FILL placeholders left untouched (not fabricated) — noted as a non-blocking must-fill-before-first-real-shop to-do in the decision doc and STATE.md."
  - "Corrected PROJECT.md/CLAUDE.md's 'order submission not exposed via MCP — structurally guaranteed' claim to 'policy-enforced hard prohibition' across all instances (What This Is, Constraints/Platform, Out of Scope, STACK.md-derived table row, MCP discipline pattern 5), per plan 01-02's live-round-trip finding. The never-checks-out/never-pays requirement itself is preserved and strengthened (explicit policy + forbidden-tools list in mcp-degradation.md), not weakened."

patterns-established:
  - "A decision delegated by the household (rather than resolved by running the planned spike) is recorded honestly as provisional and human-revisable — this keeps future re-derivation possible without treating the skip as a permanently closed question."

requirements-completed: [FOUND-09]

coverage:
  - id: D1
    description: "writable-state-decision.md records a dated, provisional GO-on-Rohlík-native decision, resolving FOUND-09; artifact-storage-spike.md marked skipped/superseded, not fabricated as run"
    requirement: "FOUND-09"
    verification:
      - kind: manual_procedural
        ref: "grep -qi 'GO\\|NO-GO\\|INCOMPLETE' project-knowledge/writable-state-decision.md && grep -qi '2026-07-31' project-knowledge/writable-state-decision.md"
        status: pass
    human_judgment: false
  - id: D2
    description: "PROJECT.md and .claude/CLAUDE.md checkout-constraint language corrected from platform-structural to policy-enforced, with the never-checks-out/never-pays requirement preserved"
    verification:
      - kind: manual_procedural
        ref: "grep -i 'not exposed\\|structurally guarantee\\|platform-enforced' .planning/PROJECT.md .claude/CLAUDE.md returns no matches"
        status: pass
    human_judgment: false

duration: 12min
completed: 2026-07-31
status: complete
---

# Phase 1 Plan 03: Writable-State Decision (Delegated) & Checkout-Constraint Correction Summary

**Recorded a provisional, human-delegated GO on Rohlík-native writable state (resolving FOUND-09), and corrected PROJECT.md/CLAUDE.md's false "checkout structurally not exposed" claim to a policy-enforced hard prohibition, per plan 01-02's live-round-trip finding.**

## Performance

- **Duration:** ~12 min
- **Started:** 2026-07-31T19:26:03Z (continuing from the 01-03 pause)
- **Completed:** 2026-07-31
- **Tasks:** 2 (writable-state decision recording; checkout-constraint correction)
- **Files modified:** 4

## Accomplishments

- **Resolved FOUND-09.** `project-knowledge/writable-state-decision.md` now records a dated
  (2026-07-31), provisional, human-revisable decision: writable state for v1 rides on Rohlík-native
  favourites (`Rohlik:get_all_user_favorites`) and order-history (`Rohlik:get_typical_order`) as the
  primary signal, plus hand-edited Project-knowledge diffs for anything Rohlík can't model. Artifact
  storage is explicitly **NOT used in v1**. The household delegated this decision rather than
  running the two-device spike — the doc states this plainly, states the rationale (the 01-02 live
  round-trip already answered the cross-device question independently; the artifact `storage`
  capability likely doesn't exist in the current runtime), and states the decision is reversible if
  a real writable artifact-store is confirmed later.
- `spikes/artifact-storage-spike.md`'s Task 2 (the two-device human spike) is marked **SKIPPED —
  superseded by a confirmed Rohlík-native writable-state path**, not fabricated as if it ran. The
  protocol itself remains intact and re-runnable if the decision is revisited.
- `household-ruleset.md` and `budget.md`'s `⚠ FILL BEFORE FIRST REAL SHOP` placeholders were left
  **untouched** (no invented allergies/dislikes/brand-prefs/budget amounts) — noted as an
  outstanding, non-blocking to-do in the decision doc and in STATE.md.
- **Corrected the checkout-constraint claim** discovered inaccurate by plan 01-02's live MCP
  round-trip. `PROJECT.md`'s "What This Is" intro, its "Out of Scope" bullet, and its
  Constraints/Platform line, plus `.claude/CLAUDE.md`'s matching project-intro, Constraints/Platform
  line, STACK.md-derived Rohlík-MCP table row, and MCP-discipline pattern-5 bullet, all previously
  asserted checkout/order submission is "not exposed via MCP" and "structurally guarantees" the
  never-checks-out requirement. All instances were rewritten to state the never-checks-out/never-pays
  requirement is a **policy-enforced hard prohibition** — the assistant is instructed to never call
  any checkout/order-submission/payment tool, per `project-knowledge/mcp-degradation.md`'s "Forbidden
  tools" list — not a platform-structural guarantee, since the connected connector does in fact
  expose those tools. The requirement itself is preserved and emphasized, not weakened.

## Task Commits

Each task was committed atomically:

1. **Checkout-constraint correction (PROJECT.md, .claude/CLAUDE.md)** - `7698814` (fix)
2. **Writable-state decision recorded (writable-state-decision.md, artifact-storage-spike.md)** - `69d0a70` (feat)

**Plan metadata:** (this commit, once made) — records 01-03 COMPLETE.

## Files Created/Modified

- `project-knowledge/writable-state-decision.md` - resolved GO/NO-GO/INCOMPLETE scaffold into a dated, provisional, human-delegated decision
- `spikes/artifact-storage-spike.md` - added a top-of-file status note marking the spike SKIPPED/superseded
- `.planning/PROJECT.md` - corrected checkout-constraint claim (4 instances: intro, Out of Scope, Constraints/Platform)
- `.claude/CLAUDE.md` - corrected matching checkout-constraint claim (4 instances: intro, Constraints/Platform, STACK table row, MCP-discipline pattern 5)

## Decisions Made

- Recorded the writable-state outcome as **provisional and human-revisable**, per the executor's
  brief — the household delegated rather than definitively closed the question, so the decision doc
  explicitly avoids over-claiming a tested NO-GO on artifact storage; it states the spike was
  skipped, not run, and remains re-runnable.
- Chose not to fill `household-ruleset.md`/`budget.md`'s `⚠ FILL` placeholders in this plan — the
  executor's brief explicitly left them untouched as the household's private data, tracked as a
  non-blocking to-do rather than invented.
- Corrected the checkout-constraint claim everywhere it appeared in both PROJECT.md and CLAUDE.md
  (not just the single line flagged in the objective), since the same false "structurally
  guaranteed" wording recurred in the "What This Is" intro, the "Out of Scope" bullet, and (in
  CLAUDE.md) the STACK.md-derived table row and MCP-discipline pattern-5 bullet — leaving any one
  uncorrected would have left a residual false claim in the same file.

## Deviations from Plan

**1. [Rule 2 - missing critical fix] Corrected checkout-constraint language beyond the single line named in the objective**
- **Found during:** the checkout-constraint correction task
- **Issue:** the objective named "the Constraints/Platform lines" but the same false claim ("order
  submission not exposed... structurally guarantees") also appeared in PROJECT.md's "What This Is"
  intro and "Out of Scope" bullet, and in CLAUDE.md's matching intro plus the STACK.md-derived
  Rohlík-MCP table row and MCP-discipline pattern-5 bullet.
- **Fix:** corrected all instances in both files to consistently state the requirement is
  policy-enforced, not platform-structural, cross-referencing `mcp-degradation.md`'s "Forbidden
  tools" section.
- **Files modified:** `.planning/PROJECT.md`, `.claude/CLAUDE.md`
- **Verification:** `grep -i "not exposed\|structurally guarantee\|platform-enforced"` over both
  files returns no matches.
- **Committed in:** `7698814`

---

**Total deviations:** 1 auto-fixed (Rule 2 — missing critical, scope-consistent correction).
**Impact on plan:** Necessary to avoid leaving a residual false claim in the same files being
corrected; no scope creep beyond the two files the objective already named.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required. The household's ⚠ FILL personalisation
(allergies, dislikes, brand preferences, budget amounts) remains an outstanding, non-blocking
to-do to complete before the first real shop — see `project-knowledge/writable-state-decision.md`
and STATE.md's Open Gates / Todos.

## Next Phase Readiness

- **Phase 1 is now 3/3 plans complete.** FOUND-09 is resolved (provisional/human-revisable);
  plan 01-02's checkout-exposure finding is now ratified in PROJECT.md/CLAUDE.md.
- Phase 4 (staples-restock, household-prefs, any learned-state skill) must design against the
  Rohlík-native + hand-edited-Project-file-diffs path recorded in `writable-state-decision.md`, not
  against artifact storage.
- Outstanding, non-blocking to-do carried forward: `household-ruleset.md`/`budget.md`'s ⚠ FILL
  placeholders (allergies, dislikes/never-buy, brand preferences, budget soft/hard amounts) must be
  filled with the household's real values before the first real shop.
- Uploaded Project-Knowledge copies of `writable-state-decision.md` (and, once filled,
  `household-ruleset.md`/`budget.md`) must be re-uploaded to claude.ai to take effect in live chats.

---
*Phase: 01-foundation-shared-spine*
*Completed: 2026-07-31*

## Self-Check: PASSED

- FOUND: project-knowledge/writable-state-decision.md
- FOUND: spikes/artifact-storage-spike.md
- FOUND: .planning/PROJECT.md
- FOUND: .claude/CLAUDE.md
- FOUND commit: 7698814
- FOUND commit: 69d0a70
</content>
