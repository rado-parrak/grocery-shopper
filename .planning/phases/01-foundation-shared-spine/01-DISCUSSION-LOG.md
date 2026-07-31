# Phase 1: Foundation & Shared Spine - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-07-31
**Phase:** 1-Foundation & Shared Spine
**Areas discussed:** Household ruleset content, Budget numbers, Confirmation hand-back, Spike scope + MCP discovery

**Discussion style:** User signalled "keep moving" (YOLO stance). Rather than a per-question interview, gray areas were resolved with documented defaults; items only the household can finalise (allergies, dislikes, brand prefs, budget amounts) were captured as explicit `⚠ FILL` placeholders rather than assumed.

---

## Household ruleset content

| Option | Description | Selected |
|--------|-------------|----------|
| Three-section ordered doc (hard constraints → preferences → notes) | Cascade reads top-down; first-match-wins; hard constraints filter, preferences rank | ✓ |
| Free-form prose rules | Simpler to write, harder for the cascade to apply deterministically | |

**User's choice:** Selected to discuss; resolved to the structured three-section format with a seeded starter + `⚠ FILL` placeholders for allergies/dislikes/brands.
**Notes:** Hard constraints are absolute (never overridden by a favourite); allergies default to "none recorded — unconfirmed", never asserted.

## Budget numbers

| Option | Description | Selected |
|--------|-------------|----------|
| Hard cap on whole-basket projected total | Cap checked against current cart + pending adds (shared basket = unit of spend) | ✓ |
| Hard cap per individual add | Simpler but wrong for a shared basket both people add to | |

**User's choice:** Whole-basket projected total. Currency CZK. Starter soft 2000 / hard 3000 Kč, marked "adjust".
**Notes:** Amounts are placeholders for the household to set.

## Confirmation hand-back

| Option | Description | Selected |
|--------|-------------|----------|
| Natural-language reply parsed by agent | Artifact stays pure collector; user's next chat turn ("yes"/"skip milk") drives the write; fits ≤3 turns | ✓ |
| Structured summary copied back | More rigid, extra friction on a phone | |
| mcp-capable artifact writes directly | Rejected — breaks the agreed→written audit boundary; duplicates MCP discipline into artifact JS | |

**User's choice:** Natural-language reply; artifact renders a copyable final list as a fallback.
**Notes:** Rationale for rejecting the mcp-capable artifact recorded in CONTEXT.md D-07.

## Spike scope + MCP discovery

| Option | Description | Selected |
|--------|-------------|----------|
| Two-part empirical proof | (a) storage write from chat flow, (b) read-back on 2nd device same account; + live MCP round-trip to document real tool surface | ✓ |
| Assume docs / defer discovery | Rejected — official docs 403-inaccessible; server is experimental | |

**User's choice:** Empirical spike with an explicit go/no-go; reversible probe add/remove for MCP discovery.
**Notes:** Go → artifact storage for learned state; No-go → Rohlík-native + hand-edited files. Gates Phase 4.

## Claude's Discretion

- Audit line format (one mobile line per item + running total) — defaulted, adjustable.
- File naming and whether read-only docs sit under `project-knowledge/` vs flat — planner/executor's call, as long as skills reference stable names.

## Deferred Ideas

- Learned-state write-back (Phase 4, gated on FOUND-09).
- Bilingual/diacritic edge-case hardening (Phase 2 UAT).
- Deals/promotions + personalised recommendations feeding the cascade (v2, LEARN-03/04).
