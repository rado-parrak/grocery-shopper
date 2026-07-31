---
gsd_state_version: 1.0
milestone: v1.0
milestone_name: milestone
current_phase: 2
current_phase_name: Quick-Add & Basket-Review
status: paused
paused_at: "2026-07-31T20:17:28.000Z"
stopped_at: PAUSED at 02-03 Task 3 human-check UAT (evals + README + UAT doc authored and committed; UX-04 unverified pending household run)
last_updated: "2026-07-31T20:17:28.000Z"
progress:
  total_phases: 2
  completed_phases: 1
  total_plans: 6
  completed_plans: 5
---

# Project State: Household Grocery Assistant (Rohlík)

## Project Reference

- **Core value:** From "add milk and bananas" to the *right kind* of items in the shared Rohlík basket in ≤3 turns, on a phone, without opening a computer.
- **Current focus:** Phase 2 — Quick-Add & Basket-Review
- **Mode:** mvp (vertical slices)
- **Granularity:** coarse (4 phases)

## Current Position

- **Phase:** 2 (Quick-Add & Basket-Review) — PAUSED
- **Plan:** 3 of 3
- **Status:** PAUSED at Task 3's human-check UAT — Tasks 1 & 2 complete (evals + README), Task 3's
  eval doc authored and committed, but the household has not yet run
  `skills/evals/uat-quick-add-basket-review.md` on a real phone. **UX-04 remains pending.**
- **Progress:** [████████░░] 83% (unchanged — Plan 03 not counted complete until UAT reported)

## Phase Map

| Phase | Name | Depends on | Status |
|-------|------|------------|--------|
| 1 | Foundation & Shared Spine | — | Complete |
| 2 | Quick-Add & Basket-Review | Phase 1 | Paused — 02-01/02-02 complete, 02-03 paused at human UAT (UX-04 pending) |
| 3 | Recipe-to-Basket & Meal Planning | Phase 2 | Not started (blocked on Phase 2's UAT) |
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
| Phase 02 P02 | 12min | 2 tasks | 1 files |
| Phase 02 P03 (agent-doable tasks only — paused before human UAT) | 22min | 3 tasks | 4 files |

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

- **FOUND-09 artifact-storage spike (Phase 1):** go/no-go on cross-device writable state — DETERMINED (2026-07-31): the household RAN the two-device spike and artifact cross-device storage **does NOT work** — a tested NO-GO. v1 writable state = Rohlík-native favourites/order-history (confirmed working) + hand-edited Project-file diffs; artifact storage NOT used. **⚠ OPEN — REVISIT BEFORE PHASE 4:** state Rohlík can't model (restock cadence, rejected-substitution history, brand switches) has no working store. See `project-knowledge/writable-state-decision.md`.
- ~~**MCP tool inventory (Phase 1):** verify actual tool names, parameters, error contract (official docs inaccessible).~~ DONE (01-02, 2026-07-31) — see mcp-degradation.md's Observed tool surface.
- ~~**PROJECT.md/CLAUDE.md ratification (Phase 1, new — 01-02):** the "order submission not exposed via MCP, structurally guaranteed" constraint is factually inaccurate per the live round-trip; household must ratify corrected, policy-enforced language.~~ DONE (01-03, 2026-07-31) — all instances in PROJECT.md and .claude/CLAUDE.md rewritten to policy-enforced language.
- **Household ⚠ FILL personalisation (Phase 1 carryover — now BLOCKING for the Phase 2 UAT):** `household-ruleset.md` §A/§B and `budget.md`'s allergy/dislike/brand-pref/budget placeholders remain unfilled — required before `skills/evals/uat-quick-add-basket-review.md` can be run for real (its stated precondition), and before the first real shop.
- **Bilingual/diacritic edge cases (Phase 1/2):** test code-switched Czech/English input on a real device — covered by the UAT's Czech-variant turn.
- ~~**Turn-economy UAT (Phase 2 onward):** measure ≤3-turn quick-add path.~~ UAT procedure AUTHORED (02-03, 2026-07-31) as `skills/evals/uat-quick-add-basket-review.md` — **⚠ OPEN, BLOCKING Phase 2 completion:** not yet run by the household. This is the active gate; see Blockers below.

### Blockers

**Phase 2 is blocked on one item: the household has not yet run the live UAT.**
`skills/evals/uat-quick-add-basket-review.md` (authored 02-03, 2026-07-31) must be run on a real
phone against the real shared Rohlík basket, which requires: (1) the Rohlík MCP OAuth connector set
up at This-project scope, and (2) `household-ruleset.md` §A/§B + `budget.md`'s `⚠ FILL`
placeholders completed with real household data. Neither can be done or fabricated by an executor
session. Phase 3 (depends on Phase 2) must not start until this UAT is reported and `UX-04` is
marked complete in `REQUIREMENTS.md`. See `.planning/phases/02-quick-add-basket-review/02-03-SUMMARY.md`
"Next Phase Readiness" for the exact resume signal.

## Session Continuity

**Last session:** 2026-07-31T20:17:28.000Z
**Stopped at:** PAUSED at 02-03 Task 3's human-check UAT — evals (Task 1) and README (Task 2)
authored and committed; the UAT procedure (Task 3) is authored and committed but not yet run.
**Resume file:** skills/evals/uat-quick-add-basket-review.md (the household runs this, then reports
pass/fail per checklist item to resume plan 02-03)

- **Last action:** Authored `skills/evals/quick-add-evals.md`, `skills/evals/basket-review-evals.md`
  (trigger eval sets, cross-skill near-misses per D-03), `skills/README.md` (zip-and-upload
  packaging guide, explicit not-for-`.claude/skills/` rule, OAuth-only rule), and
  `skills/evals/uat-quick-add-basket-review.md` (the human UAT checklist). Committed each task
  atomically. Wrote `02-03-SUMMARY.md` documenting the pause. **Did not run the UAT and did not
  fabricate a pass/fail result** — that requires a household member on a real phone.
- **Next action:** The household must (1) set up the Rohlík MCP connector (This-project scope),
  (2) fill `household-ruleset.md` §A/§B and `budget.md`'s `⚠ FILL` placeholders, (3) upload both
  skills + the eight `project-knowledge/` contracts to the claude.ai Project per `skills/README.md`,
  then (4) run `skills/evals/uat-quick-add-basket-review.md` end to end and report pass/fail. Once
  reported, resume plan 02-03: mark `UX-04` complete (or fix and re-run if failed), then close out
  Phase 2 and advance to Phase 3.

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
- [Phase ?]: Disjoint trigger vocabulary from quick-add: basket-review's description excludes add-item phrasing without literal 'add milk and bananas' collision, keeping the phase's negative-grep gate clean
- [Phase ?]: Budget-threshold reporting in basket-review is read-only/non-blocking — this skill has no write path, so blocking at the hard cap remains quick-add's responsibility
- [Phase 2, 02-03, 2026-07-31] Authored quick-add/basket-review trigger eval sets, skills/README.md packaging guide, and the human UAT checklist; PAUSED plan execution at the UAT's human-check task rather than fabricate a pass/fail — UX-04 stays pending until the household runs it on a real phone against the live Rohlík connector.
