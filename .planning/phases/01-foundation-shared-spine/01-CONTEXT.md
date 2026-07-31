# Phase 1: Foundation & Shared Spine - Context

**Gathered:** 2026-07-31
**Status:** Ready for planning

<domain>
## Phase Boundary

Author the shared spine every skill depends on, as single-source-of-truth Project
Knowledge documents, plus the read-only household config, a proven live Rohlík MCP
round-trip, and a settled answer on cross-device writable state. Specifically:

- Five shared-internal contracts: `resolution-cascade.md`, `confirmation-protocol.md`,
  `substitution-policy.md`, `mcp-degradation.md`, `audit-format.md` (FOUND-01…05).
- Read-only household config: `household-ruleset.md` (FOUND-06), `budget.md` (FOUND-07),
  `seed-favourites.md` (FOUND-08).
- A proven live MCP round-trip that empirically documents the experimental server's real
  tool names, parameters, and error shapes.
- The artifact-storage go/no-go spike (FOUND-09), gating Phase 4 learned-state skills.

**Not this phase:** any shopping skill (quick-add et al. are Phase 2+); the cascade is
*authored and validated* here but first *exercised end-to-end* by quick-add in Phase 2.
No basket writes beyond a single reversible probe add/remove for MCP discovery.
</domain>

<decisions>
## Implementation Decisions

### Household ruleset (FOUND-06)
- **D-01:** `household-ruleset.md` is structured in three ordered sections the cascade reads top-down: **(A) Hard constraints** (allergies, "never buy X") that *filter candidates out*; **(B) Preferences** (ranked, per category) that *rank what remains*; **(C) Notes**. Declarative, ordered, first-match-wins. — **Reversibility:** costly — the cascade doc and every skill's resolution step assume this shape; changing it means re-reading how hard-vs-soft is expressed everywhere.
- **D-02:** Ship a **starter ruleset with illustrative preference rules** (see Specifics), so the cascade is testable in Phase 2, but the **safety-critical and personal fields are placeholders the household must fill**: allergies, dislikes/never-buy, and brand preferences. These are marked in-file with `⚠ FILL BEFORE FIRST REAL SHOP`.
- **D-03:** Hard constraints are **absolute** — a candidate that violates one is never proposed, never substituted, never surfaced, and a favourite can never override one (locks CASC-02, CASC-04). Allergies default to **"none recorded — unconfirmed"**, never asserted as "no allergies".

### Budget (FOUND-07)
- **D-04:** Currency **CZK**. `budget.md` holds a **soft threshold** (warn) and a **hard cap** (block), both hand-editable. Starter placeholders: **soft 2000 Kč, hard 3000 Kč** — explicitly marked "adjust to your household".
- **D-05:** The **hard cap applies to the projected whole-basket total** (current cart total + the items about to be added), not to a single add — because the shared basket is the unit of spend and both people add to it. Before any write: read cart → compute projected total → if > hard cap, refuse and ask to cut items or raise the cap (locks BUDG-03). — **Reversibility:** reversible.

### Confirmation hand-back (FOUND-02)
- **D-06:** The confirmation artifact stays a **pure response-collector** (tickboxes + quantity steppers, one card for the batch, no MCP). The user's decision returns to the agent as their **next natural-language chat turn** — typed or dictated ("yes add all", "skip the milk", "2 rohlíky not 4") — which the agent parses against the pre-artifact `cart_read` snapshot. The artifact also renders a compact plain-text "final list" the user *can* copy, but the expected ≤3-turn path is a short confirming reply. — **Reversibility:** costly — this is the artifact→agent contract every mutating skill inherits; changing it (e.g. to a structured hand-back) touches confirmation-protocol.md and all skills.
- **D-07:** Rationale on record for *why not* an mcp-capable artifact: preserves the one auditable "agreed → written" boundary, keeps read-before-write/idempotency/audit logic in editable markdown not compiled artifact JS, and keeps the fragile experimental-MCP dependency behind the cheaper-to-fix layer.

### Spike scope + MCP discovery (FOUND-09)
- **D-08:** **Artifact-storage spike acceptance:** on ONE shared Claude account, prove (a) a *published* artifact declaring the `storage` capability can WRITE a key from the household's normal chat-driven flow, and (b) that value READS BACK on the SECOND device signed into the same account. **Go** → artifact storage becomes the home for learned state (favourites write-back, staples cadence, recipe repertoire). **No-go** → fall back to Rohlík-native favourites/order-history + hand-edited Project files; Phase 4 learned-state features degrade accordingly. Result is documented as an explicit go/no-go note. — **Reversibility:** one-way for dependents — Phase 4 skills bind to whichever store wins; switching later means migrating persisted learned state.
- **D-09:** **MCP discovery is empirical, not doc-driven** (official docs are inaccessible/403). Phase 1 connects the OAuth connector and does a live round-trip — product search → `cart_read` → a single **reversible** probe `cart_add` then `cart_remove` on a throwaway item → `cart_read` again — and records the **actual tool names, parameters, and error shapes** into `mcp-degradation.md` (plus a short capability note). The degradation policy is grounded in observed behaviour, and every mutating flow is specified as read-before-write + read-back-after-write regardless of the server's own idempotency claims.

### Claude's Discretion
- **Audit format (FOUND-05, not separately selected):** one short mobile line per added item — `✓ {qty}× {Czech product name} — {line total} Kč · {why}` where `{why}` is the matched rule id or `oblíbené` (favourite) — followed by a running basket total. One line per item, no wide tables (locks AUDT-01, UX-02, UX-03). Adjustable if you dislike the shape.
- File naming, exact section ordering within each contract doc, and whether the read-only docs live under a `project-knowledge/` folder vs flat — planner/executor's discretion, as long as skills reference them by stable name.
</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project spine
- `.planning/PROJECT.md` — constraints, key decisions, core value
- `.planning/REQUIREMENTS.md` §Foundation & Shared Internals — FOUND-01…09 (this phase's requirements)
- `.planning/ROADMAP.md` §"Phase 1: Foundation & Shared Spine" + §"Build-Order Constraint" — goal and success criteria
- `.planning/STATE.md` §"Safety Invariants" — the five invariants that constrain every phase

### Research (grounds the contracts)
- `.planning/research/STACK.md` — Skill authoring, response-collector artifact pattern, MCP discipline, writable-state options with confidence levels
- `.planning/research/ARCHITECTURE.md` — three-layer structure (read-only knowledge / shared internals / skills), data flow, build order
- `.planning/research/PITFALLS.md` — the domain pitfalls each contract must design against (P1 hallucinated success, P2 non-idempotent writes, P3 MCP disappearance, P4 wrong-kind resolution, P5 silent substitution, P6 constraint override, P9 artifact-calls-MCP)
- `.planning/research/SUMMARY.md` — synthesized approach + suggested build order + open gates

No external ADRs yet — the contracts authored in this phase become the canonical refs for Phase 2+.
</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- None yet — this is the first build phase on a fresh repo. Phase 1's outputs (the five contracts + config) ARE the reusable assets every later phase consumes.

### Established Patterns
- GSD planning conventions in place (`.planning/`), and the GSD framework itself is vendored under `.claude/` (agents, commands) on `main`.

### Integration Points
- The Rohlík MCP connector (OAuth, `https://mcp.rohlik.cz/mcp`) — connected once at the claude.ai Project level; Phase 1 is where its real tool surface is first exercised and documented.
- Claude Project Knowledge — where the read-only docs (ruleset, budget, favourites, and the five contracts) are uploaded so every chat preloads them.
</code_context>

<specifics>
## Specific Ideas

**Starter preference rules to seed in `household-ruleset.md` (illustrative — household edits):**
- Vegetables & fruit → prefer **BIO**; else **farm-sourced** (farmářské); else cheapest acceptable.
- Eggs → free-range / z podestýlky or better.
- Milk → `[preferred brand — FILL]`; default to plnotučné unless noted.
- Prefer **Czech origin** where sensible.
- Pack size → prefer the **smallest pack** that meets the requested quantity (anti-waste; supports meal-plan pack-size dedup later).

**Placeholders the household MUST personalise before first real shop:**
- `⚠ Allergies` (hard constraint — safety-critical)
- `⚠ Dislikes / never-buy` (hard constraint)
- `⚠ Brand preferences` (soft)
- `⚠ Budget soft/hard amounts` in `budget.md`
- `seed-favourites.md` product IDs (can also be bootstrapped from Rohlík order history during the MCP round-trip)
</specifics>

<deferred>
## Deferred Ideas

- **Learned-state write-back** (auto-updating favourites/staples cadence) — depends on the FOUND-09 spike outcome; belongs to Phase 4 (LEARN-01). Noted, not built here.
- **Bilingual/diacritic edge-case hardening** — real-device testing of code-switched Czech/English input belongs to Phase 2 UAT; Phase 1 only sets the "Czech output" convention.
- **Deals/promotions + personalised recommendations** feeding the cascade — v2 (LEARN-03/04).

### None reviewed-but-deferred todos — no matching todos existed.
</deferred>

---

*Phase: 1-Foundation & Shared Spine*
*Context gathered: 2026-07-31*
