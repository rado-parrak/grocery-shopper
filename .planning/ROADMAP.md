# Roadmap: Household Grocery Assistant (Rohlík)

**Created:** 2026-07-31
**Granularity:** coarse
**Mode:** mvp (vertical slices)
**Core Value:** From "add milk and bananas" to the *right kind* of items in the shared Rohlík basket in ≤3 turns, on a phone, without opening a computer.

## Build-Order Constraint

The shared spine (resolution cascade, confirmation protocol, substitution policy, MCP-degradation
policy, audit format) plus the read-only ruleset / budget / favourites **must exist before any skill
is useful**. Phase 1 authors that spine. Phase 2 (quick-add + basket-review) is the first vertical
slice that proves the whole spine end-to-end: cascade → cart_read → confirmation artifact → read-back
→ idempotent cart_add → audit. Every later skill reuses the spine rather than reimplementing it.

## Phases

- [x] **Phase 1: Foundation & Shared Spine** - Author the five shared-internal contracts + read-only config, prove the MCP round-trip, and settle the writable-state go/no-go
- [ ] **Phase 2: Quick-Add & Basket-Review** - First vertical slice: "add milk and bananas" reaches the shared basket in ≤3 turns; "what's in the basket" reads it back with budget state
- [ ] **Phase 3: Recipe-to-Basket & Meal Planning** - Recipes and weekly meal plans from trusted sources become confirmed, ruleset-resolved baskets
- [ ] **Phase 4: Staples-Restock & Household Preferences** - Learned staples and evolving preferences, always proposed for approval and routed to the right writable state

## Phase Details

### Phase 1: Foundation & Shared Spine

**Goal**: The shared spine every skill depends on exists — the resolution cascade, confirmation protocol, substitution policy, MCP-degradation policy, and audit format as single-source-of-truth documents — alongside the read-only household config (ruleset, budget, favourites), a proven live Rohlík MCP round-trip, and a settled answer to whether agent-writable state can persist across devices.
**Mode:** mvp
**Depends on**: Nothing (first phase)
**Requirements**: FOUND-01, FOUND-02, FOUND-03, FOUND-04, FOUND-05, FOUND-06, FOUND-07, FOUND-08, FOUND-09
**Success Criteria** (what must be TRUE):

  1. The five shared-internal contracts — resolution cascade, confirmation protocol, substitution policy, MCP-degradation policy, audit format — each exist as one source-of-truth Project document, validated against a live MCP round-trip, so no skill reimplements them. [FOUND-01…05]
  2. The household ruleset loads at the start of every resolution and the budget config (soft threshold + hard cap) loads on every confirmation, both hand-editable. [FOUND-06, FOUND-07]
  3. A hand-editable seed-favourites file of pre-approved Rohlík product IDs is in place. [FOUND-08]
  4. The artifact-storage spike returns a documented go/no-go on cross-device writable state, gating the learned-state skills in Phase 4 (staples, preferences). [FOUND-09]

**Plans**: 3/3 plans executed

- [x] 01-01-PLAN.md — Author the shared spine: five contracts + three read-only config files + Walking-Skeleton + confirmation-artifact prototype [FOUND-01,02,03,05,06,07,08]
- [x] 01-02-PLAN.md — Prove the live Rohlík MCP surface: setup + probe protocol + human round-trip, grounded into mcp-degradation.md [FOUND-04]
- [x] 01-03-PLAN.md — Settle writable state: artifact-storage go/no-go spike + household personalisation [FOUND-09] — resolved by household-delegated decision (provisional/human-revisable; spike skipped, superseded by confirmed Rohlík-native path); checkout-constraint correction ratified in PROJECT.md/CLAUDE.md

### Phase 2: Quick-Add & Basket-Review

**Goal**: The first vertical slice proves the entire spine end-to-end — "add milk and bananas" (Czech or English) becomes the *right* Rohlík products in the shared basket in ≤3 turns via an interactive confirmation, with idempotent writes, graceful degradation, budget guardrails, and a Czech audit — and "what's in the basket" reads the live cart back with its running total and budget state.
**Mode:** mvp
**Depends on**: Phase 1
**Requirements**: CASC-01, CASC-02, CASC-03, CASC-04, CASC-05, CASC-06, CONF-01, CONF-02, CONF-03, CONF-04, CONF-05, BUDG-01, BUDG-02, BUDG-03, SUBS-01, DEGR-01, DEGR-02, AUDT-01, UX-01, UX-02, UX-03, UX-04, QADD-01, QADD-02, REVW-01, REVW-02
**Success Criteria** (what must be TRUE):

  1. "add milk and bananas" (typed in Czech or English) triggers quick-add and resolves each item through the cascade — hard constraints filter candidates out first, ruleset preferences rank what remains, favourites fill confident matches, and the user is asked only when both fail — reaching the shared basket in ≤3 turns without asking what the ruleset already answers. [CASC-01…06, QADD-01, UX-01, UX-04]
  2. Before any write, an interactive confirmation artifact with per-item tickboxes and a quantity stepper — usable one-handed on a phone, no wide tables — collects the user's choices without ever calling the MCP; the agent reads the final list back verbally and adds only explicitly approved items. [CONF-01, CONF-02, CONF-03, CONF-04, UX-03, QADD-02]
  3. The cart is read before writing so items already present are never duplicated, and an add is reported successful only after a read-back confirms it; if the Rohlík MCP is unavailable or changed, the skill degrades to a plain shopping list the user can enter manually instead of faking success. [CONF-05, DEGR-01, DEGR-02]
  4. Every add returns a Czech-product audit line — what item, how many, what it cost, and which rule or favourite matched — and out-of-stock items are proposed as ruleset-driven substitutions for approval, never swapped silently. [AUDT-01, UX-02, SUBS-01]
  5. Every confirmation shows a running basket total, warns on crossing the soft threshold, and refuses any add that would exceed the hard cap (asking the user to cut items or raise the cap); "what's in the basket" reads the live cart with its total and budget-threshold state, then hands off to manual checkout. [BUDG-01, BUDG-02, BUDG-03, REVW-01, REVW-02]

**Plans**: 1/3 plans executed

- [x] 02-01-PLAN.md — quick-add SKILL.md: bilingual cascade → confirmation artifact → idempotent write → Czech audit, with budget/substitution/degradation safety rails [CASC-01..06, CONF-01..05, BUDG-01..03, SUBS-01, DEGR-01,02, AUDT-01, UX-01,02,03, QADD-01,02]
- [ ] 02-02-PLAN.md — basket-review SKILL.md: live cart read + running total + budget-threshold state + manual-checkout handoff [REVW-01, REVW-02]
- [ ] 02-03-PLAN.md — trigger eval sets (cross-skill near-misses) + packaging README + human UAT (≤3-turn measurement) [UX-04]

**UI hint**: yes

### Phase 3: Recipe-to-Basket & Meal Planning

**Goal**: Recipes and weekly meal plans from trusted sources become confirmed, ruleset-resolved Rohlík baskets, reusing the same cascade and confirmation spine — a single recipe turns into a resolved ingredient list, and "plan next week" merges N recipes into one pack-size-aware shopping list.
**Mode:** mvp
**Depends on**: Phase 2
**Requirements**: RCPE-01, RCPE-02, RCPE-03, MEAL-01, MEAL-02
**Success Criteria** (what must be TRUE):

  1. A recipe given by name, URL, or uploaded PDF/ebook from a trusted source is turned into an ingredient list. [RCPE-01]
  2. Recipe ingredients resolve to concrete Rohlík products through the same cascade as quick-add and are confirmed in the standard artifact before anything is added. [RCPE-02]
  3. The trusted recipe-source list is curated and user-editable, so untrusted sources are not silently ingested. [RCPE-03]
  4. "plan next week" builds N meals from trusted sources into one merged shopping list. [MEAL-01]
  5. The merged list deduplicates ingredients shared across recipes and rounds to real pack sizes — no four bunches of parsley for four recipes. [MEAL-02]

**Plans**: TBD

### Phase 4: Staples-Restock & Household Preferences

**Goal**: The household's recurring staples and evolving preferences are captured and applied — staples learned from Rohlík order history and always proposed (never auto-added), and "we don't buy X anymore" propagated to the ruleset, favourites, staples, and recipe sources, routed to the right writable state per the Phase 1 spike outcome.
**Mode:** mvp
**Depends on**: Phase 2 (spine); Phase 1 FOUND-09 gate (writable-state go/no-go); Phase 3 (so preference edits can reach recipe sources)
**Requirements**: STPL-01, STPL-02, PREF-01, PREF-02
**Success Criteria** (what must be TRUE):

  1. Staples are learned from Rohlík order history (frequency + recency) and remain hand-editable. [STPL-01]
  2. Restock proposes the staple list for approval through the standard confirmation and never auto-adds. [STPL-02]
  3. "we don't buy X anymore" updates the household ruleset, favourites, staples, and recipe sources accordingly. [PREF-01]
  4. Preference changes that must persist are routed to writable state (Rohlík-native or artifact storage per the FOUND-09 outcome), or surfaced as a hand-editable diff to the read-only Project files. [PREF-02]

**Plans**: TBD

## Progress

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation & Shared Spine | 3/3 | Complete | 2026-07-31 |
| 2. Quick-Add & Basket-Review | 1/3 | In Progress|  |
| 3. Recipe-to-Basket & Meal Planning | 0/? | Not started | - |
| 4. Staples-Restock & Household Preferences | 0/? | Not started | - |

## Coverage

- v1 requirements: 44 total
- Mapped to phases: 44 ✓
- Unmapped: 0

Every v1 requirement maps to exactly one phase. No orphans, no duplicates.

---
*Roadmap created: 2026-07-31*
