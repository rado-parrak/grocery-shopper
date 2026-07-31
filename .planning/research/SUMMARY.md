# Project Research Summary

**Project:** Household Grocery Assistant (Rohlík)
**Domain:** Phone-first, bilingual household shopping assistant using Claude Skills + official Rohlík MCP
**Researched:** 2026-07-31
**Confidence:** MEDIUM-HIGH

## Executive Summary

This is a carefully constrained, platform-native implementation built entirely within claude.ai: Claude Skills for self-triggering behaviors, Claude Project for shared preloaded context, and official Rohlík MCP for cart integration. The core value—converting vague requests into the *right* concrete Rohlík product in ≤3 turns—depends entirely on a single shared resolution cascade and confirmation step, both authored as Project Knowledge documents (shared internals) rather than reimplemented per-skill.

The recommended approach builds in strict order: (1) design shared internals (resolution cascade, confirmation protocol, degradation policy, audit format) + minimal ruleset, (2) implement quick-add as validation milestone exercising the entire spine, (3) add basket-review for read-only reconciliation, then (4) layer specialized skills (staples-restock, recipe-to-basket, meal-plan) reusing the same backbone. One critical unresolved dependency—artifact persistent storage's cross-device write capability—must be spiked before staples/favorites skills; until verified, lean on Rohlík's native account data as the writable-state layer.

Largest risks are architecture erosion (skills reimplementing cascade instead of referencing it, confirmation artifact drifting toward calling MCP directly) and MCP brittleness (Rohlík is explicitly experimental/personal-use and may change without notice), both mitigated by treating shared internals as single source of truth and designing every skill with graceful-degradation fallback paths.

## Recommended Stack

**Core technologies:**
- **Claude Skills** (SKILL.md + YAML frontmatter) — Only preload mechanism on claude.ai; each shopping behavior is independently-triggered
- **Claude Projects** — Preload read-only Knowledge (ruleset, budget, favorites, recipe sources); no agent-side write path
- **Rohlík MCP** (OAuth custom connector, declared experimental/personal-use) — Cart operations (search, add, remove, update, read), order history, deals; checkout intentionally not exposed
- **Claude Artifacts** (Oct 2025 storage capability) — Interactive confirmation UI (response-collector only, never calls MCP); fallback writable-state layer
- **Skill-Creator tool** — Validates SKILL.md structure and description-triggering behavior

**Critical conventions:**
- SKILL.md descriptions must be third-person, concrete "what + when" (≤1024 chars) — sole signal Claude uses for skill triggering
- Fully-qualified MCP tool names (e.g., `Rohlík:cart_read`) once multiple connectors exist
- Shared logic (cascade, confirmation, degradation, audit) lives in Project Knowledge docs, never reimplemented per-skill
- OAuth connector setup once at platform level; credentials never in Project files or skill definitions

## Expected Features

**Table Stakes (users expect these):**
- Resolution cascade (ruleset → favourites → ask)
- Interactive confirmation artifact (tickboxes + stepper, response-collector only)
- Idempotent adds (read cart before write, never duplicate)
- Graceful MCP degradation (fallback to plain list if Rohlík unavailable)
- quick-add skill proving end-to-end spine
- Auditability (what/qty/cost/why)
- Bilingual input (Czech/English mixed, normalized to catalogue Czech)

**Differentiators (competitive advantage):**
- Ruleset-driven resolution with household-specific quality rules, allergies, brand preferences
- Learned staples from order history (infer restock cadence from Rohlík patterns)
- Pack-size-aware meal-plan merging (reduces waste, optimizes cost)
- Budget guardrails (soft-warn + hard-block, not just visibility)
- Two-person shared-basket awareness (read-before-write, no-duplicate logic for concurrent use)

**Defer to v2+:**
- Checkout/payment automation (platform-blocked, violates safety boundary)
- Multi-retailer support (breaks core one-basket assumption)
- Pantry/inventory tracking (redundant to learned staples)
- Auto-anything without human present (architecturally foreclosed)

## Architecture Approach

Three tightly-coupled layers: **Project Knowledge** (read-only, human-edited) holds domain facts AND shared internals (cascade, confirmation, degradation, audit format); **Skills** are thin orchestrators referencing shared internals, adding only domain-specific logic; **External sources of truth** are live Rohlík basket (read before every write) and writable state (primary: Rohlík's native favorites/order-history; fallback: artifact storage).

**Major components:**
1. **Shared internals (Project Knowledge docs)** — Cascade precedence, confirmation artifact contract, substitution rules, MCP degradation fallback, audit format
2. **Resolution cascade** — Single algorithm; ruleset → favorites → ask; shows specific resolved product; referenced (not reimplemented) by every skill
3. **Confirmation artifact** — Pure response-collector; batches all items in one card; never calls MCP itself
4. **Read-before-write primitive** — Every basket-mutating skill must cart_read immediately before writing
5. **Writable state layer** — Primary: Rohlík's account data; secondary: artifact persistent storage (needs cross-device spike first)

## Top Critical Pitfalls

1. **Hallucinated success (P1)** — Never confirm add succeeded without cart_read verifying it. Phase: foundation.
2. **Non-idempotent duplicate writes (P2)** — No read-before-write causes duplicates. Mitigate: shared primitive. Phase: foundation.
3. **Silent MCP disappearance (P3)** — Experimental server, no fallback means errors or fake-success. Mitigate: explicit degrade path at every call site. Phase: foundation.
4. **Wrong-kind resolution (P4)** — Generic item resolves incorrectly. Mitigate: cascade precedence explicit; confirmation shows specific product. Phase: cascade.
5. **Silent substitution (P5)** — Out-of-stock item swapped without approval. Mitigate: always propose; never auto-add. Phase: cascade.
6. **Constraint override (P6)** — Favorite silently overrides hard constraint (allergy). Mitigate: hard constraints filter first. Phase: ruleset + cascade.
7. **Mobile-hostile output (P12)** — Wide tables unreadable on phone. Mitigate: design phone-first; test on actual narrow viewport. Phase: confirmation + every skill.
8. **Asking what ruleset answers (P7)** — Clarifying questions waste turns. Mitigate: gate questions behind completed ruleset/favorites lookup. Phase: cascade.
9. **Confirmation artifact calling MCP (P9)** — Removes human-approval audit boundary. Mitigate: enforce invariant—artifact collects response only; agent alone calls MCP. Phase: foundation build + standing code-review check.

## Roadmap Implications: Suggested Phase Structure

### Phase 1: Foundation & Shared Internals
**Rationale:** Every skill downstream depends on shared internals existing. Nothing can be built until cascade, confirmation, degradation, and audit-format contracts are designed.

**Delivers:**
- Five Project Knowledge documents (resolution-cascade.md, confirmation-protocol.md, substitution-policy.md, mcp-degradation.md, audit-format.md)
- household-ruleset.md (hand-edited, minimal viable)
- budget.md, seed-favourites.md
- Proven MCP integration (cart_read, cart_add idempotent reconcile, read-back verification)
- Artifact-storage spike (go/no-go gate for later phases)

**Addresses:** Resolution cascade, confirmation, idempotent adds, graceful degradation, auditability, budget guardrails.

**Avoids:** Pitfalls P1–P9, P12–P13 via upfront design.

**Research flags:** MCP capability discovery (official docs inaccessible); artifact-storage cross-device spike (critical go/no-go); bilingual edge-case testing.

### Phase 2: Quick-Add (End-to-End Spine Validation)
**Rationale:** Minimal skill that exercises entire spine: trigger → cascade → cart_read → substitution/budget checks → confirmation artifact → user confirms → idempotent cart_add → audit. Validation that shared internals are real and sufficient.

**Delivers:** quick-add SKILL.md; end-to-end resolution; confirmation UX validated (mobile, one-handed); idempotency verified; graceful degradation tested; audit proven; bilingual input tested; turncount ≤3.

### Phase 3: Basket-Review (Read-Only Reconciliation)
**Rationale:** Cheapest second build; reads cart without mutation risk. Serves as recovery mechanism for concurrent-write races. Gives household visibility into basket state.

**Delivers:** basket-review SKILL.md; running total with soft/hard thresholds; reconciliation UI for catching duplicates/errors.

### Phase 4: Staples-Restock (Recurring Items)
**Rationale:** Reuses proven spine; adds cadence logic. Depends on artifact-storage spike outcome. First skill testing writable-state strategy.

**Delivers:** staples-restock SKILL.md; hand-edited or learned list; writable-state integration.

### Phase 5: Recipe-to-Basket (Name/URL Import)
**Rationale:** Adds ingredient-extraction and recipe-source-trust logic on top of cascade. Ingredients resolved via same cascade as quick-add.

**Delivers:** recipe-to-basket SKILL.md; recipe-sources.md; ingredient-extraction pipeline.

### Phase 6: Meal-Plan (Multi-Recipe, Pack-Size De-Duplication)
**Rationale:** Extends recipe-to-basket to N recipes with pack-size-aware merging. Depends on recipe-to-basket extraction logic.

**Delivers:** meal-plan SKILL.md; multi-recipe aggregation; pack-size rounding logic.

### Phase 7: Household-Prefs (Preference Learning & Editing)
**Rationale:** Only skill touching read-only boundary. Surfaces learned signals as proposals. Routes changes to writable state or human-applied diffs.

**Delivers:** household-prefs SKILL.md; preference-capture and correction UI.

## Phase Ordering Rationale

1. **Shared internals first:** Every skill depends on cascade, confirmation, degradation. It's the skeleton, not a feature.
2. **Quick-add next:** Validates skeleton works end-to-end. If it doesn't, every later skill inherits the bug.
3. **Basket-review third:** Lowest-risk second build; read-only; serves as recovery net for concurrency; visibility from day one.
4. **Staples-restock fourth:** Tests writable-state strategy (Rohlík-native vs. artifact); first skill touching learned state.
5. **Recipe-to-basket fifth:** Ingredient extraction hard part; independent of cascade. Stage for meal-plan.
6. **Meal-plan sixth:** Depends on recipe-to-basket extraction; adds pack-size merging.
7. **Household-prefs last:** Depends on other skills generating learned signals.

## Research Flags

**Need deeper research:**
- **Phase 1:** MCP capability discovery (empirical test of tool names, error shapes, diacritic handling); artifact-storage cross-device spike (go/no-go gate)
- **Phase 2:** Bilingual/diacritic edge-case testing on real device; Rohlík search typo-resilience
- **Phase 4:** Depends on artifact-storage spike outcome
- **Phase 5:** Spike extraction quality on 3-5 real recipes

**Standard patterns (skip research):**
- **Phase 3:** Straightforward read + display
- **Phase 6:** Extends Phase 5; pattern established
- **Phase 7:** Writable-state proven by Phase 4

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | MEDIUM-HIGH | Official docs verified; artifact storage real (Oct 2025), but cross-device visibility UNVERIFIED |
| Features | MEDIUM | Project brief solid; no live competitor testing; dependencies sound but should be re-verified during quick-add build |
| Architecture | MEDIUM | Platform constraints established; "shared reference docs as pseudo-library" is inferred best pattern, not documented intent |
| Pitfalls | MEDIUM-HIGH | 19 pitfalls identified from first principles; clear prevention strategies; some (concurrent writes, bilingual edge cases) are residual risks |

**Overall confidence:** MEDIUM-HIGH

### Gaps to Address During Planning

- **Artifact persistent storage cross-device:** Spike before Phase 4; test write-from-chat + cross-device visibility
- **MCP tool inventory:** Verify actual tool list, parameters, error contract (official docs inaccessible)
- **Confirmation artifact response-passing:** Assume non-MCP artifact returns decision via user's next message; test in Phase 1 spike
- **Bilingual edge-case normalization:** Test colloquialisms, grammatical variants, code-switched input during Phase 1/2
- **Turn-economy validation:** Measure ≤3-turn target from quick-add onward as UAT metric
- **Budget hard-cap enforcement:** Validate with explicit UAT (try to push past cap, confirm write refused)

## Sources

Synthesized from `.planning/research/STACK.md`, `FEATURES.md`, `ARCHITECTURE.md`, and `PITFALLS.md` (each with its own cited sources), grounded against `.planning/PROJECT.md`.
