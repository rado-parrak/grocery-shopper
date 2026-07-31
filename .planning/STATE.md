---
gsd_state_version: 1.0
milestone: v1.0
milestone_name: milestone
current_phase: 1
current_phase_name: Foundation & Shared Spine
status: executing
stopped_at: Completed 01-02-PLAN.md (MCP round-trip transcribed, mcp-degradation.md grounded, checkout-exposure prohibition added). Still PAUSED at 01-03 Task 2 (blocking-human).
last_updated: "2026-07-31T19:26:03.260Z"
progress:
  total_phases: 1
  completed_phases: 0
  total_plans: 3
  completed_plans: 2
---

# Project State: Household Grocery Assistant (Rohlík)

## Project Reference

- **Core value:** From "add milk and bananas" to the *right kind* of items in the shared Rohlík basket in ≤3 turns, on a phone, without opening a computer.
- **Current focus:** Phase 1 — Foundation & Shared Spine
- **Mode:** mvp (vertical slices)
- **Granularity:** coarse (4 phases)

## Current Position

- **Phase:** 1 (Foundation & Shared Spine) — EXECUTING
- **Plan:** 2 of 3 (01-02) COMPLETE; 3 of 3 (01-03) still paused
- **Status:** 01-02 complete. PAUSED at 01-03 Task 2 (blocking-human checkpoint) — two-device artifact-storage spike + household ⚠ FILL values
- **Progress:** [██████░░░░] 67%

## Phase Map

| Phase | Name | Depends on | Status |
|-------|------|------------|--------|
| 1 | Foundation & Shared Spine | — | Not started |
| 2 | Quick-Add & Basket-Review | Phase 1 | Not started |
| 3 | Recipe-to-Basket & Meal Planning | Phase 2 | Not started |
| 4 | Staples-Restock & Household Preferences | Phase 2, Phase 1 (FOUND-09), Phase 3 | Not started |

## Performance Metrics

- Phases complete: 0/4
- Requirements mapped: 44/44
- Requirements validated (shipped): 0/44

**Per-Plan Metrics:**

| Plan | Duration | Tasks | Files |
|------|----------|-------|-------|
| Phase 01 P01 | 15min | 3 tasks | 9 files |
| Phase 01 P02 | 15min | 3 tasks | 5 files |
| Phase 01 P03 (partial — Task 1 only) | 4min | 1 task | 2 files |

## Accumulated Context

### Safety Invariants (carry into every phase)

- Confirmation before any basket write; the confirmation artifact never calls the MCP.
- Idempotent adds: read the cart before writing, never duplicate.
- Graceful degradation: never fake a successful add; fall back to a plain manual list.
- Budget hard cap blocks additions that would exceed it.
- The assistant never checks out and never pays — **CORRECTED 2026-07-31 (01-02 live round-trip):
  this is POLICY-enforced, not platform-enforced.** The connected Rohlík MCP connector DOES expose
  checkout/timeslot/payment-method/order-management/claim tools (`submit_checkout`,
  `get_checkout`, `reserve_timeslot`, `cancel_order`, `submit_claim`, etc. — full list in
  `project-knowledge/mcp-degradation.md`'s "Forbidden tools — never call" section). Every skill's
  own instructions must never call any of these tools, under any circumstance. **Needs human
  ratification in PROJECT.md/CLAUDE.md**, which currently states this guarantee is
  platform-structural — that claim is now known to be inaccurate.

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

- **FOUND-09 artifact-storage spike (Phase 1):** go/no-go on cross-device writable state — gates Phase 4 (staples, preferences).
- ~~**MCP tool inventory (Phase 1):** verify actual tool names, parameters, error contract (official docs inaccessible).~~ DONE (01-02, 2026-07-31) — see mcp-degradation.md's Observed tool surface.
- **PROJECT.md/CLAUDE.md ratification (Phase 1, new — 01-02):** the "order submission not exposed via MCP, structurally guaranteed" constraint is factually inaccurate per the live round-trip; household must ratify corrected, policy-enforced language.
- **Bilingual/diacritic edge cases (Phase 1/2):** test code-switched Czech/English input on a real device.
- **Turn-economy UAT (Phase 2 onward):** measure ≤3-turn quick-add path.

### Blockers

- Plan 01-03 paused at blocking-human checkpoint: (A) the artifact-storage cross-device spike (Step 0 existence check -> Device-A write -> Device-B read-back, or fast NO-GO if no storage capability exists) must be run by a human on two physical phones signed into the shared Claude account; (B) the household's private ⚠ FILL values (allergies, dislikes/never-buy, brand preferences, budget soft/hard amounts) must come from the household directly. Neither is producible from this repo/CI. Resume by pasting back (A) the spike outcome and (B) the ⚠ FILL values or "fill later", then confirming the Task 3 writable-state decision.

## Session Continuity

**Last session:** 2026-07-31T19:26:03.246Z
**Stopped at:** Completed 01-02-PLAN.md (MCP round-trip transcribed, mcp-degradation.md grounded, checkout-exposure prohibition added). Still PAUSED at 01-03 Task 2 (blocking-human).
**Resume file:** .planning/phases/01-foundation-shared-spine/01-03-PLAN.md

- **Last action:** Transcribed the human's live Rohlík MCP round-trip into `spikes/mcp-round-trip-results.md`, populated `mcp-degradation.md`'s Observed tool surface table, added the "Forbidden tools — never call" section for the newly-discovered checkout/order/payment tool exposure, updated `seed-favourites.md`'s bootstrap note, completed FOUND-04, and marked plan 01-02 COMPLETE (2026-07-31).
- **Next action:** Human runs `spikes/artifact-storage-spike.md` (01-03) inside claude.ai on two physical phones, and pastes back the spike outcome plus the household's ⚠ FILL values (or "fill later") to resume plan 01-03. Separately, the household should review and ratify the checkout-exposure finding in PROJECT.md/CLAUDE.md (currently states the never-checks-out guarantee is platform-structural; live evidence shows it must be policy-enforced instead).

---
*State initialized: 2026-07-31*

## Decisions

- [Phase ?]: Shared-internal docs and read-only config authored under project-knowledge/, prototype under prototypes/ (Claude's discretion, D-10)
- [Phase ?]: D-07 mcp-capable-artifact rationale recorded permanently inside confirmation-protocol.md, not just referenced
- [Phase ?]: household-ruleset.md and budget.md placeholders left genuinely unfilled (no invented allergies/brand-prefs/budget amounts) pending plan 01-03
- [Phase ?]: [Phase 1, 01-02] Live MCP round-trip found Rohlík connector exposes checkout/order/payment/claim tools -- contradicts PROJECT.md's 'not exposed, structurally guaranteed' claim; mitigated with a Forbidden-tools policy prohibition in mcp-degradation.md, flagged for household ratification in PROJECT.md/CLAUDE.md.
- [Phase ?]: [Phase 1, 01-02] mcp-degradation.md's Observed tool surface grounded in real dated observations; favourites-equivalent tools (get_all_user_favorites, get_typical_order) confirmed present, seed-favourites.md bootstrap role now confirmed temporary.
