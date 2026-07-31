# Phase 2: Quick-Add & Basket-Review - Context

**Gathered:** 2026-07-31
**Status:** Ready for planning

> Decisions defaulted by Claude on the household's delegated authority (they chose to keep moving rather than answer per-question). Everything here is revisable; items only the household can finalize are flagged `⚠ HOUSEHOLD`.

<domain>
## Phase Boundary

Build the first end-to-end vertical slice as two Claude Skills that consume — never re-derive — the Phase-1 shared spine:

- **`quick-add`** — "add milk and bananas" (Czech or English) → resolve each item via the cascade → read cart → interactive confirmation artifact → verbal read-back → idempotent basket write → Czech audit, in ≤3 turns.
- **`basket-review`** — "what's in the basket" → read the live cart → running total + budget-threshold state → hand off to manual checkout.

**In scope:** the two SKILL.md files + any one-level-deep reference files, wired to the eight `project-knowledge/` contracts and the observed Rohlík tool surface.
**Out of scope (later phases):** staples-restock, recipe-to-basket, meal-plan, household-prefs (Phases 3–4). No new shared contracts — those are frozen from Phase 1. No checkout (forbidden-tools prohibition stands).
</domain>

<decisions>
## Implementation Decisions

### Skill packaging & location
- **D-01:** Authored skill sources live in a top-level repo dir `skills/<name>/SKILL.md` (+ optional one-level-deep reference files). These are the version-controlled sources the household **zips and uploads to the claude.ai Project** (Settings → Features). — **Reversibility:** reversible.
- **D-02:** Do NOT place them in `.claude/skills/` — that directory is Claude Code's own execution path and is already occupied by the GSD framework; putting product skills there would wrongly activate them in this repo's tooling and pollute the zip. The claude.ai upload is the only intended runtime for these skills.

### Trigger vocabulary (avoid cross-firing — the top multi-skill risk)
- **D-03:** `quick-add` description triggers on *naming specific items to buy now* (e.g. "add milk and bananas", "we need eggs", "přidej máslo"). `basket-review` triggers on *inspecting the cart* ("what's in the basket", "co je v košíku", "check the cart", "running total"). Descriptions are third-person "what + when", ≤1024 chars, with concrete CZ+EN trigger phrases, explicitly disjoint so "add X" never fires basket-review and vice-versa.

### Confirmation artifact
- **D-04:** The confirmation artifact is generated **fresh each shopping turn** (ephemeral, per confirmation-protocol.md), built from the pre-write `Rohlik:get_cart` snapshot + the cascade's proposed items. It declares **no capabilities**, never calls MCP, single-column, large tap targets, no horizontal scroll. `prototypes/confirmation-artifact-prototype.html` is the visual/behavioral reference, not the runtime artifact. — **Reversibility:** costly — this is the confirmation contract every future mutating skill reuses.
- **D-05:** Hand-back is the household's next natural-language turn ("yes add all" / "skip the milk" / "2 not 4"), parsed against the pre-artifact cart snapshot (per Phase-1 D-06).

### The ≤3-turn quick-add flow
- **D-06:** Turn 1 (user): names items. Agent (same turn): loads ruleset+budget, resolves each item via the cascade (hard-constraint filter → preference rank → favourites via `get_all_user_favorites`/`get_typical_order`+seed-favourites → ask only if unresolved), reads cart, renders the confirmation artifact showing proposed items + running total + budget state. Turn 2 (user): ticks/edits + confirms. Agent (same turn): reconciles to desired state, idempotent `add_items_to_cart` (only the delta), read-back via `get_cart`, emits the Czech audit. Ask-the-user only fires a 3rd turn when the cascade genuinely can't resolve an item.

### MCP tool usage (observed surface — from Phase 1 round-trip)
- **D-07:** Fully-qualified observed tools only: `Rohlik:batch_search_products` (`{queries:[{keyword}]}`), `Rohlik:get_cart`, `Rohlik:add_items_to_cart` (`{items:[{productId,quantity}]}`), `Rohlik:remove_cart_item` (`{product_id}` — note the snake_case vs camelCase landmine), `Rohlik:get_all_user_favorites`, `Rohlik:get_typical_order`, `Rohlik:get_product_details`. Success is judged only by read-back + explicit `success`/`items_failed_to_add` checks (the 200-shaped `success:false` case), never by a write's own return. The forbidden checkout/order/payment tools are NEVER called. — **Reversibility:** one-way for dependents — skills bind to these names; a server change means editing every skill (mitigated by the degradation policy).

### Bilingual & mobile
- **D-08:** Accept mixed Czech/English item input; search the Czech catalogue; output Czech product names in confirmations and audits (UX-02). Replies short, phone-first, no wide tables (UX-03).

### Claude's Discretion
- Exact SKILL.md body structure and whether ingredient-heavy logic is split into a one-level-deep reference file — planner/executor's call, staying under ~500 lines/body.
- Exact confirmation-artifact HTML/CSS, reusing the prototype.
</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### The frozen spine (do NOT re-derive — reference by name)
- `project-knowledge/resolution-cascade.md` — the ruleset→favourites→ask algorithm quick-add follows
- `project-knowledge/confirmation-protocol.md` — the response-collector artifact contract
- `project-knowledge/substitution-policy.md` — out-of-stock handling (propose + approve)
- `project-knowledge/mcp-degradation.md` — observed tool surface, forbidden-tools prohibition, read-before-write/read-back discipline, degrade-to-list
- `project-knowledge/audit-format.md` — the one-line Czech audit format
- `project-knowledge/household-ruleset.md` — ⚠ HOUSEHOLD: allergies/dislikes/brands still placeholder; skills read this at resolution start
- `project-knowledge/budget.md` — ⚠ HOUSEHOLD: soft/hard CZK amounts still placeholder; applied on every confirmation
- `project-knowledge/seed-favourites.md` — bootstrap favourites (Rohlík-native now primary)
- `project-knowledge/writable-state-decision.md` — Rohlík-native primary (favourites/typical_order)
- `prototypes/confirmation-artifact-prototype.html` — reference implementation for the artifact
- `spikes/mcp-round-trip-results.md` — the observed live tool params/returns/errors

### Planning trail
- `.planning/phases/02-quick-add-basket-review/` (this phase) · `.planning/ROADMAP.md` §Phase 2 · `.planning/REQUIREMENTS.md` (CASC/CONF/BUDG/SUBS/DEGR/AUDT/UX/QADD/REVW) · `.planning/research/` (STACK/ARCHITECTURE/PITFALLS/SUMMARY)
</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- The eight `project-knowledge/` contracts + the confirmation prototype (Phase 1) — the skills orchestrate these; they add only skill-specific glue.
- Observed Rohlík tool surface documented in `mcp-degradation.md` / `spikes/mcp-round-trip-results.md`.

### Established Patterns
- Single-source-of-truth: skills reference the spine docs by name, never re-describe them (enforced by eval-style trigger tests recommended in research).
- Response-collector artifact never calls MCP; the agent alone mutates the cart.

### Integration Points
- claude.ai Project (skills uploaded as a zip; contracts uploaded as Project Knowledge) + the Rohlík MCP OAuth connector.
</code_context>

<specifics>
## Specific Ideas
- Build 3+ trigger evals per skill including cross-skill near-misses (confirm "add milk and bananas" fires `quick-add`, not `basket-review`), per research Pattern 1.
- quick-add reconciles to desired state (add only the delta) rather than blind-append, per the idempotency rule.
</specifics>

<deferred>
## Deferred Ideas
- Learned favourites write-back, staples cadence, recipe/meal-plan skills — Phases 3–4.
- Live end-to-end testing against the real shared basket (needs the household on a phone + the ⚠ HOUSEHOLD ruleset/budget data filled) — a UAT step after authoring, not part of skill authoring itself.

None reviewed-but-deferred todos.
</deferred>

---

*Phase: 2-Quick-Add & Basket-Review*
*Context gathered: 2026-07-31*
