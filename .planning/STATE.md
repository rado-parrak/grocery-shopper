---
gsd_state_version: 1.0
milestone: v1.0
milestone_name: milestone
current_phase: 2
current_phase_name: Quick-Add & Basket-Review
status: executing
stopped_at: Completed 02-01-PLAN.md (quick-add SKILL.md + resolution-notes.md authored, all verify gates pass)
last_updated: "2026-07-31T20:04:02.079Z"
progress:
  total_phases: 2
  completed_phases: 1
  total_plans: 6
  completed_plans: 4
---

# Project State: Household Grocery Assistant (Rohlík)

## Project Reference

- **Core value:** From "add milk and bananas" to the *right kind* of items in the shared Rohlík basket in ≤3 turns, on a phone, without opening a computer.
- **Current focus:** Phase 2 — Quick-Add & Basket-Review
- **Mode:** mvp (vertical slices)
- **Granularity:** coarse (4 phases)

## Current Position

- **Phase:** 2 (Quick-Add & Basket-Review) — EXECUTING
- **Plan:** 2 of 3
- **Status:** Ready to execute
- **Progress:** [███████░░░] 67%

## Phase Map

| Phase | Name | Depends on | Status |
|-------|------|------------|--------|
| 1 | Foundation & Shared Spine | — | Complete |
| 2 | Quick-Add & Basket-Review | Phase 1 | Not started |
| 3 | Recipe-to-Basket & Meal Planning | Phase 2 | Not started |
| 4 | Staples-Restock & Household Preferences | Phase 2, Phase 1 (FOUND-09), Phase 3 | Not started |

## Performance Metrics

- Phases complete: 1/4
- Requirements mapped: 44/44
- Requirements validated (shipped): 0/44

**Per-Plan Metrics:**

| Plan | Duration | Tasks | Files |
|------|----------|-------|-------|
| Phase 01 P01 | 15min | 3 tasks | 9 files |
| Phase 01 P02 | 15min | 3 tasks | 5 files |
| Phase 01 P03 (partial — Task 1 only) | 4min | 1 task | 2 files |
| Phase 01 P03 | 12min | 2 tasks | 4 files |
| Phase 02 P01 | 15min | 3 tasks | 2 files |

## Accumulated Context

### Safety Invariants (carry into every phase)

- Confirmation before any basket write; the confirmation artifact never calls the MCP.
- Idempotent adds: read the cart before writing, never duplicate.
- Graceful degradation: never fake a successful add; fall back to a plain manual list.
- Budget hard cap blocks additions that would exceed it.
- The assistant never checks out and never pays — **CORRECTED 2026-07-31 (01-02 live round-trip),
  RATIFIED 2026-07-31 (01-03): this is POLICY-enforced, not platform-enforced.** The connected
  Rohlík MCP connector DOES expose checkout/timeslot/payment-method/order-management/claim tools
  (`submit_checkout`, `get_checkout`, `reserve_timeslot`, `cancel_order`, `submit_claim`, etc. —
  full list in `project-knowledge/mcp-degradation.md`'s "Forbidden tools — never call" section).
  Every skill's own instructions must never call any of these tools, under any circumstance.
  PROJECT.md and .claude/CLAUDE.md now consistently state this guarantee is policy-enforced, not
  platform-structural.

### Key Decisions

- One shared Rohlík account (single shared basket = household state).
- One personal Claude account shared across two devices.
- Resolution cascade is one shared internal, not per-skill.
- Confirmation is a response-collector artifact that never calls the MCP.
- First milestone = quick-add end-to-end (proves cascade + confirmation + real basket write).
- **[Phase 1, 01-02, 2026-07-31] CRITICAL — checkout exposure finding:** the live Rohlík MCP
  round-trip found the connector exposes checkout/order/payment/claim tools, contradicting
  PROJECT.md/CLAUDE.md's "order submission not exposed via MCP — structurally guaranteed"
  constraint. Mitigated immediately with an explicit "Forbidden tools — never call" prohibition in
  `mcp-degradation.md` (policy-enforced, not platform-enforced). **Awaiting household ratification**
  of corrected constraint language in PROJECT.md/CLAUDE.md.

- **[Phase 1, FOUND-09, 2026-07-31] ⚠ REVISIT BEFORE PHASE 4 — writable state:** the household RAN
  the two-device artifact-storage spike and it does **NOT work** (no confirmed cross-device
  read-back) — a **tested NO-GO on artifact storage**. v1 writable state = Rohlík-native
  (`get_all_user_favorites`/`get_typical_order`), which is confirmed working, so Phase 2 is
  unaffected. OPEN: household state Rohlík can't model (restock cadence, rejected-substitution
  history, explicit brand switches) has no working store; interim = hand-edited Project-file diffs.
  This must be revisited before Phase 4 (staples-restock / household-prefs). See
  `project-knowledge/writable-state-decision.md`.

- **[Phase 1, 01-02, 2026-07-31] MCP tool surface grounded:** `mcp-degradation.md`'s Observed tool
  surface table is now populated from a real, dated live round-trip (search, cart_read, add,
  remove, forced error). Key findings: `Rohlik:` fully-qualified prefix; add uses `productId`
  (camelCase, nested) while remove uses `product_id` (snake_case, top-level) — a naming
  inconsistency every cart-mutating skill must call out; a forced invalid-productId add returns a
  normal 200-shaped body with `success:false`, not a protocol error — the definitive case for
  "never trust a write's own return value." Favourites-equivalent tools confirmed present
  (`get_all_user_favorites`, `get_typical_order`) — `seed-favourites.md`'s bootstrap role is now
  confirmed temporary.

### Open Gates / Todos

- ~~**FOUND-09 artifact-storage spike (Phase 1):** go/no-go on cross-device writable state — gates Phase 4 (staples, preferences).~~ RESOLVED (01-03, 2026-07-31, provisional/human-revisable) — see `project-knowledge/writable-state-decision.md`. Household delegated the decision; writable state = Rohlík-native favourites/order-history + hand-edited Project-file diffs; artifact storage NOT used. Revisit before Phase 4 if reopened.
- ~~**MCP tool inventory (Phase 1):** verify actual tool names, parameters, error contract (official docs inaccessible).~~ DONE (01-02, 2026-07-31) — see mcp-degradation.md's Observed tool surface.
- ~~**PROJECT.md/CLAUDE.md ratification (Phase 1, new — 01-02):** the "order submission not exposed via MCP, structurally guaranteed" constraint is factually inaccurate per the live round-trip; household must ratify corrected, policy-enforced language.~~ DONE (01-03, 2026-07-31) — all instances in PROJECT.md and .claude/CLAUDE.md rewritten to policy-enforced language.
- **Household ⚠ FILL personalisation (non-blocking, Phase 1 carryover):** `household-ruleset.md` §A/§B and `budget.md`'s allergy/dislike/brand-pref/budget placeholders remain unfilled — must be completed before the first real shop, but do not block Phase 2+ planning.
- **Bilingual/diacritic edge cases (Phase 1/2):** test code-switched Czech/English input on a real device.
- **Turn-economy UAT (Phase 2 onward):** measure ≤3-turn quick-add path.

### Blockers

None. Phase 1 is complete (3/3 plans). The household ⚠ FILL personalisation to-do above is
tracked but non-blocking for Phase 2+ planning.

## Session Continuity

**Last session:** 2026-07-31T20:04:02.064Z
**Stopped at:** Completed 02-01-PLAN.md (quick-add SKILL.md + resolution-notes.md authored, all verify gates pass)
**Resume file:** None

- **Last action:** Recorded the provisional/human-revisable writable-state decision in `project-knowledge/writable-state-decision.md` (resolving FOUND-09), marked the two-device spike SKIPPED/superseded in `spikes/artifact-storage-spike.md`, and corrected the checkout-constraint claim from platform-structural to policy-enforced everywhere it appeared in `.planning/PROJECT.md` and `.claude/CLAUDE.md`. Plan 01-03 and Phase 1 (3/3 plans) marked COMPLETE (2026-07-31).
- **Next action:** Advance to Phase 2 (Quick-Add & Basket-Review) via `/gsd-plan-phase` or the standard GSD phase workflow. Separately, the household should fill `household-ruleset.md`/`budget.md`'s ⚠ FILL placeholders before the first real shop (non-blocking to-do).

---
*State initialized: 2026-07-31*

## Decisions

- [Phase ?]: Shared-internal docs and read-only config authored under project-knowledge/, prototype under prototypes/ (Claude's discretion, D-10)
- [Phase ?]: D-07 mcp-capable-artifact rationale recorded permanently inside confirmation-protocol.md, not just referenced
- [Phase ?]: household-ruleset.md and budget.md placeholders left genuinely unfilled (no invented allergies/brand-prefs/budget amounts) pending plan 01-03
- [Phase ?]: [Phase 1, 01-02] Live MCP round-trip found Rohlík connector exposes checkout/order/payment/claim tools -- contradicts PROJECT.md's 'not exposed, structurally guaranteed' claim; mitigated with a Forbidden-tools policy prohibition in mcp-degradation.md, flagged for household ratification in PROJECT.md/CLAUDE.md.
- [Phase ?]: [Phase 1, 01-02] mcp-degradation.md's Observed tool surface grounded in real dated observations; favourites-equivalent tools (get_all_user_favorites, get_typical_order) confirmed present, seed-favourites.md bootstrap role now confirmed temporary.
- [Phase ?]: [Phase 1, 01-03, 2026-07-31] Household delegated the writable-state decision (chose to proceed on Rohlik-native path rather than run the two-device artifact-storage spike). Recorded as provisional/human-revisable, dated 2026-07-31, resolving FOUND-09: writable state for v1 = Rohlik-native favourites/order-history + hand-edited Project-file diffs; artifact storage NOT used.
- [Phase ?]: [Phase 1, 01-03, 2026-07-31] Ratified the checkout-exposure correction flagged in 01-02: PROJECT.md and .claude/CLAUDE.md's 'order submission not exposed via MCP -- structurally guaranteed' claim rewritten to 'policy-enforced hard prohibition' everywhere it appeared; the never-checks-out/never-pays requirement itself preserved and strengthened, not weakened.
- [Phase ?]: [Phase 2, 02-01] Authored skills/quick-add/SKILL.md as a thin orchestrator + skills/quick-add/resolution-notes.md; tracer task verified end-to-end before expanding to multi-item/bilingual resolution and budget/substitution/degradation safety rails. No forbidden checkout/order/payment tool literal present.
