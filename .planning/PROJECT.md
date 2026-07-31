# Household Grocery Assistant (Rohlík)

## What This Is

A set of Claude Skills, living in one shared Claude Project, that lets two people
(husband + wife) run their household grocery shopping entirely from their phones.
The assistant resolves vague human requests ("add milk", "we need stuff for
Thursday's dinner") into specific Rohlík.cz products via the official Rohlík MCP,
respecting a household quality ruleset, and pushes them to the shared Rohlík
basket. Checkout and payment stay manual — this is a policy-enforced hard
prohibition (the assistant is instructed to never call any checkout,
order-submission, or payment tool; see `project-knowledge/mcp-degradation.md`
"Forbidden tools"), not a platform-structural guarantee. The connected Rohlík
MCP connector does, in fact, expose checkout/order/payment tools (confirmed by
a live round-trip, 2026-07-31) — the never-checks-out/never-pays requirement
holds entirely on instruction-level discipline.

## Core Value

Opening a fresh chat on a phone and going from "add milk and bananas" to the
**right kind** of items in the shared basket in **under three turns**, without
opening a computer. If everything else fails, the resolution cascade turning a
vague item into a correct, ruleset-compliant Rohlík product must work.

## Requirements

### Validated

(None yet — ship to validate)

### Active

<!-- Full, testable list lives in REQUIREMENTS.md. High-level hypotheses here. -->

- [ ] The resolution cascade (ruleset → favourites → ask) resolves abstract items to concrete Rohlík products
- [ ] `quick-add` takes ad-hoc items to the basket end-to-end with an interactive confirmation artifact
- [ ] A single household ruleset (quality rules, dislikes, allergies, brand prefs) governs every resolution
- [ ] Budget guardrails: running total on every confirmation, soft-threshold warning, hard cap that blocks
- [ ] Substitutions are proposed and approved, never silent
- [ ] Idempotent basket writes: read cart before adding, never duplicate
- [ ] Graceful MCP degradation: fall back to a plain shopping list, never fake a successful add
- [ ] Every add is audited: what, how many, cost, and which rule/favourite matched
- [ ] Staples restock, recipe-to-basket, meal-plan, and basket-review skills build on the same spine
- [ ] Bilingual input (Czech/English), Czech product output

### Out of Scope

- Checkout, payment, delivery-slot selection — policy-enforced manual step (the assistant is instructed to never call any checkout/order-submission/payment tool; not a platform-structural guarantee); a deliberate safety boundary
- Multi-retailer support — v1 is Rohlík-only; the whole design assumes one shared Rohlík basket
- Nutrition / calorie / macro tracking — not the product's job
- Inventory / pantry tracking (what's in the fridge) — out of scope for v1
- Any automation running without a human in the chat — every mutation needs a person present and approving

## Context

- **Two users, one household, one shared basket.** Mobile-only — every interaction
  is thumb-typed or dictated on a phone. Long outputs, wide tables, and
  multi-screen confirmations are failures.
- **Bilingual:** users write Czech and English interchangeably; catalogue data is Czech.
- **Non-technical usage mode:** no setup steps at chat time; skills must be preloaded
  via the Project and self-trigger from natural phrasing.
- **Rohlík MCP** is the official first-party server (`https://mcp.rohlik.cz/mcp`,
  custom connector, OAuth). Capabilities: product search, cart add/remove/update,
  cart read, order status/history, deals/promotions, personalised recommendations.
  Declared experimental, personal-use-only, and subject to change or termination
  without notice — **design for graceful degradation.**
- **Claude constraint:** Project knowledge files are read-only to the agent. It can
  read them every chat but cannot write back. Anything that must *learn* cannot
  live there.

## Constraints

- **Policy**: Never checks out, never pays — a policy-enforced hard prohibition, not a platform-structural guarantee. The Rohlík MCP connector DOES expose checkout/order/payment tools (confirmed by the live round-trip, 2026-07-31); the assistant is instructed to never call any of them, under any circumstance (see `project-knowledge/mcp-degradation.md` "Forbidden tools — never call").
- **Platform**: Rohlík MCP is experimental and may change or disappear without notice — every skill must degrade to a plain manual list rather than fail silently or hallucinate a successful add.
- **Data**: Claude Project files are read-only to the agent — writable state (learned favourites, staples, recipe repertoire) cannot live there.
- **Accounts**: One shared Rohlík account (one shared basket — the design's central assumption). One personal Claude account shared across two devices (no Team/Enterprise Project needed; single account = single natural scope for shared writable state).
- **UX**: Mobile-only. Short replies, no wide tables, no walls of options. Confirmation must be usable one-handed.
- **Privacy**: OAuth only — no credentials in project files or skill definitions.
- **Turn economy**: Common path ≤ 3 turns; never ask what the ruleset already answers.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| One shared Rohlík account | The whole design assumes a single shared basket as household state; two accounts would break it | ✓ Good |
| One personal Claude account shared on two devices (not Team/Enterprise, not two accounts) | Sidesteps the Team/Enterprise requirement for a shared Project; a single account gives one natural scope for shared writable state and one preloaded skill set | ✓ Good |
| Writable state: Rohlík-native primary; artifact storage NOT used | Household tested the two-device artifact-storage spike 2026-07-31 → **NO-GO (does not work)**. Rohlík-native favourites/order-history (`get_all_user_favorites`/`get_typical_order`) confirmed working; hand-edited Project-file diffs for what Rohlík can't model | ⚠️ Revisit before Phase 4 (no store for Rohlík-unmodellable state) |
| Resolution cascade is one shared internal, not per-skill | Brief mandates a single source of truth used by every shopping skill | ✓ Good — authored in Phase 1 (`project-knowledge/resolution-cascade.md`), consumed by Phase 2 skills |
| Confirmation is a response-collector artifact that never calls the MCP | Keeps the mutate step explicit: agent reads back the final list and confirms verbally before any basket write | ✓ Good — contract + prototype in Phase 1; wired into quick-add in Phase 2 |
| Never-checks-out is policy-enforced, not platform-structural | Live round-trip (2026-07-31) found the connector DOES expose checkout/payment tools; mitigated by the forbidden-tools prohibition in `mcp-degradation.md` | ✓ Good (corrected + ratified) |
| First milestone = quick-add end-to-end | Proves the cascade + confirmation artifact + a real basket write; every other skill is a variation on that spine | — Authored (Phase 2); awaiting live-phone UAT (UX-04) |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-07-31 after initialization*
