# Walking Skeleton — Household Grocery Assistant (Rohlík)

**Phase:** 1
**Generated:** 2026-07-31

> Adapted to this project's nature. There is **no web stack** here — no DB, no HTTP server, no
> deployed UI. The deliverable is a set of Claude Project-Knowledge markdown documents + SKILL.md
> conventions, exercised inside claude.ai. The Walking-Skeleton mappings are therefore:
> "one real DB read/write" → **one real Rohlík MCP cart round-trip**; "one real UI interaction" →
> **one rendered confirmation-artifact response-collector prototype**; "dev deployment" →
> **the docs uploaded to the claude.ai Project + the OAuth connector attached**.

## Capability Proven End-to-End

The five shared-internal contracts + read-only config exist as one coherent, cross-referenced
document system; a single **real** Rohlík MCP cart round-trip (search → cart_read → reversible
cart_add → cart_remove → cart_read) proves the tool surface and grounds the degradation policy in
observed behaviour; and a Step-0 capability check settles whether cross-device artifact storage
even exists — so no Phase-2+ skill has to reinvent the spine or guess at the platform.

## Architectural Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Shared-logic mechanism | Project Knowledge markdown docs, referenced by stable filename; **never** re-derived per skill | No cross-skill import mechanism exists on this platform; Project Knowledge is the only ambient-shared surface (RESEARCH Pattern 1) |
| Ruleset shape | Three ordered sections read top-down: (A) Hard constraints filter out → (B) Preferences rank → (C) Notes; first-match-wins | D-01 — the cascade and every skill's resolution step assume this shape |
| Hard-constraint precedence | Absolute: a violating candidate is never proposed/substituted/surfaced; a favourite can never override one | D-03 — encodes the allergy-safety invariant as an access-control-shaped rule |
| Confirmation hand-back | Artifact is a **pure response-collector** (no capabilities); the user's next natural-language chat turn is the only return channel; the skill parses it against the pre-artifact cart_read snapshot | D-06 / D-07 — preserves the one auditable "agreed → written" boundary |
| Budget cap semantics | Hard cap applies to the **projected whole-basket total** (current cart + pending adds), not a single add; read cart → project → refuse if over cap | D-04 / D-05 — the shared basket is the unit of spend |
| MCP discovery | **Empirical, not doc-driven** (official docs 403). One live reversible round-trip records real tool names/params/error shapes into mcp-degradation.md | D-09 — training data and community tool lists are hypotheses only |
| Writable-state home | **Undecided until the Step-0 spike returns.** Default fallback = Rohlík-native favourites/order-history + hand-edited Project files; artifact storage only if proven | D-08 — this session's live capability roster lists only `downloads`/`mcp`, no `storage` |
| Currency / output language | CZK; product names/brands/units always render in Czech (catalogue-verbatim), conversational text follows the user's language | D-04, RESEARCH Pitfall 5 |
| Directory layout | `project-knowledge/` for the 8 uploadable docs + the go/no-go decision; `spikes/` for probe protocols & results; `prototypes/` for the throwaway artifact demo | Claude's discretion (D-10) — skills reference docs by stable name |

## Stack Touched in Phase 1

Mapped to this project's actual analogs (not a web stack):

- [ ] **Shared-internals "library"** — the five contract docs authored as one cross-referenced set
- [ ] **Read-only config/data tier** — household-ruleset.md, budget.md, seed-favourites.md authored (with `⚠ FILL` placeholders)
- [ ] **External-service round-trip** ("one real DB read AND write") — one live reversible Rohlík MCP cart round-trip, recorded
- [ ] **UI interaction** ("one interactive element") — one rendered confirmation-artifact response-collector prototype (tickboxes + steppers, zero MCP)
- [ ] **"Deployment"** — the 8 docs uploaded to the claude.ai Project + Rohlík MCP OAuth connector attached & showing "Connected"
- [ ] **Writable-state gate** — Step-0 artifact-storage existence check → dated go/no-go note

## Out of Scope (Deferred to Later Slices)

Explicit — prevents later phases re-litigating Phase 1's minimalism:

- Any shopping **skill** (quick-add, basket-review, recipe-to-basket, meal-plan, staples-restock, household-prefs) — Phase 2+.
- Exercising the cascade end-to-end against a real request — first done by quick-add in Phase 2.
- Any basket write beyond the single reversible probe add/remove for MCP discovery.
- Learned-state write-back (auto-updating favourites/staples cadence) — Phase 4, gated on the FOUND-09 outcome.
- Bilingual/diacritic edge-case hardening — Phase 2 UAT.
- Deals/promotions + personalised recommendations feeding the cascade — v2.

## Subsequent Slice Plan

Each later phase adds one vertical slice on top of this skeleton without changing its architectural decisions:

- **Phase 2:** quick-add + basket-review — the first slice that runs the whole spine end-to-end (cascade → cart_read → confirmation artifact → read-back → idempotent cart_add → Czech audit; "what's in the basket" reads the live cart with budget state).
- **Phase 3:** recipe-to-basket + meal-plan — recipes/weekly plans reuse the same cascade + confirmation spine, adding pack-size-aware dedup.
- **Phase 4:** staples-restock + household-prefs — learned staples + evolving preferences, routed to whichever writable store the FOUND-09 spike selected.
