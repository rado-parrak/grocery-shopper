---
phase: 02-quick-add-basket-review
plan: 02
subsystem: skills
tags: [claude-skills, rohlik-mcp, markdown, read-only]

requires:
  - phase: 01-foundation-shared-spine
    provides: "The eight project-knowledge/ contracts (mcp-degradation.md, budget.md, audit-format.md, etc.) that basket-review orchestrates by reference"
provides:
  - "skills/basket-review/SKILL.md — the read-only cart-inspection Claude Skill"
affects: [02-03-eval-set, phase-3-recipe-meal-plan, phase-4-staples-household-prefs]

actuals:
  tokens: 1141
  tasks: 2
  commits: 2

tech-stack:
  added: []
  patterns:
    - "Thin-orchestrator SKILL.md: references project-knowledge/ contracts by name, never re-derives their internal logic"
    - "Read-only skill pattern: single Rohlik:get_cart call, no confirmation artifact, no write path at all"

key-files:
  created:
    - skills/basket-review/SKILL.md
  modified: []

key-decisions:
  - "Disjoint trigger vocabulary from quick-add (D-03): basket-review's description explicitly excludes add-item phrasing and states 'Do NOT use when the user names one or more specific grocery items to buy right now'"
  - "Budget-threshold reporting is read-only and never blocks — blocking a write at the hard cap is quick-add's responsibility, not basket-review's, since this skill has no write path"

patterns-established:
  - "Read-only Rohlík skills need only one tool call (get_cart) and no confirmation artifact — a lighter shape than quick-add's full resolve/confirm/write/audit pipeline"

requirements-completed: [REVW-01, REVW-02]

coverage:
  - id: D1
    description: "basket-review triggers on CZ+EN cart-inspection phrasing (disjoint from quick-add's add-item vocabulary), reads the live cart via Rohlik:get_cart, and lists items with a Czech-named running total"
    requirement: "REVW-01"
    verification:
      - kind: other
        ref: "grep gate: test -f skills/basket-review/SKILL.md && grep košíku/what's-in-the-basket/Rohlik:get_cart/audit-format.md && ! grep 'add milk and bananas'"
        status: pass
    human_judgment: false
  - id: D2
    description: "basket-review reports budget-threshold state (soft/hard per budget.md) and hands off to manual Rohlík-app checkout, never calling any forbidden checkout tool"
    requirement: "REVW-02"
    verification:
      - kind: other
        ref: "grep gate: budget.md/soft/hard cap/mcp-degradation.md/manual+Rohlík app+checkout present; forbidden tool literals absent"
        status: pass
    human_judgment: false

duration: 12min
completed: 2026-07-31
status: complete
---

# Phase 2 Plan 2: basket-review Skill Summary

**Read-only cart-inspection Claude Skill: single Rohlik:get_cart call, Czech item list + running total, soft/hard budget-threshold report, manual-checkout handoff — no write path at all**

## Performance

- **Duration:** 12 min
- **Started:** 2026-07-31T20:07:18Z
- **Completed:** 2026-07-31T20:09:16Z
- **Tasks:** 2
- **Files modified:** 1 (created)

## Accomplishments
- Authored `skills/basket-review/SKILL.md` with a third-person, CZ+EN description that triggers on cart-inspection phrasing ("what's in the basket", "check the cart", "co je v košíku") and explicitly excludes quick-add's add-item vocabulary, keeping the two skills' trigger surfaces disjoint per D-03.
- Read flow: single `Rohlik:get_cart` call, Czech-catalogue product names per `audit-format.md`'s Language Rule, running total via the same "Celkem v košíku: {total} Kč" line convention — no re-derivation of the audit format.
- Budget-threshold reporting deferring to `budget.md` by name: plainly notes soft-threshold and hard-cap state as a **read-only report** (this skill never writes, so it never blocks — blocking is quick-add's job).
- Manual checkout handoff and a "Never call" section deferring to `mcp-degradation.md`'s Forbidden-tools list by name (not re-listed), plus an honest degrade branch that states the connection is down rather than ever fabricating cart contents.

## Task Commits

Each task was committed atomically:

1. **Task 1: Author basket-review SKILL.md — trigger + live cart read + total** - `f229305` (feat)
2. **Task 2: Budget-threshold state, degradation, and manual-checkout handoff** - `db02736` (feat)

**Plan metadata:** (this commit, to follow)

## Files Created/Modified
- `skills/basket-review/SKILL.md` - Read-only Claude Skill: CZ+EN cart-inspection triggers, Rohlik:get_cart read, Czech item list + running total, budget-threshold report, manual-checkout handoff, honest degradation, and a "Never call" reference to mcp-degradation.md's Forbidden-tools section.

## Decisions Made
- Disjoint trigger vocabulary from quick-add (D-03): the description's exclusion clause deliberately avoids the literal phrase "add milk and bananas" (used verbatim in quick-add's own description) so the negative-grep gate in this plan's Task 1 `<verify>` — which checks that basket-review's file does NOT contain that exact phrase — passes cleanly, while still conveying the same exclusion in different words ("Do NOT use when the user names one or more specific grocery items to buy right now").
- Budget-threshold reporting is explicitly framed as read-only/non-blocking, since basket-review has no write path to the cart at all — this avoids any ambiguity with quick-add's hard-cap-refuses-to-write behavior.

## Deviations from Plan

None - plan executed exactly as written. No bugs, missing functionality, blocking issues, or architectural changes encountered; both tasks' automated `<verify>` gates passed on the first attempt after the trigger-phrase wording adjustment described above (a wording choice made during authoring, not a deviation from the plan's intent).

## Issues Encountered
None.

## User Setup Required

None - no external service configuration required. This is a markdown-only authoring plan (no package installs, no infrastructure).

## Next Phase Readiness
- `skills/basket-review/SKILL.md` is ready for the plan 02-03 eval set, which must confirm the disjoint-trigger cross-skill collision cases named in 02-CONTEXT.md (e.g. "add milk and bananas" fires `quick-add` only; "what's in the basket" fires `basket-review` only).
- Both quick-add (02-01) and basket-review (02-02) are now authored against the same frozen Phase-1 spine, with no duplicated logic between them — ready for the phase's cross-skill verification pass.
- No blockers.

---
*Phase: 02-quick-add-basket-review*
*Completed: 2026-07-31*

## Self-Check: PASSED

- FOUND: skills/basket-review/SKILL.md
- FOUND: f229305 (Task 1 commit)
- FOUND: db02736 (Task 2 commit)
