# quick-add — Trigger Eval Set

**Purpose:** Confirm `skills/quick-add/SKILL.md`'s description reliably self-triggers on phrasing
that names one or more concrete items to buy right now, in Czech, English, or mixed — and does
**NOT** fire on basket-inspection phrasing, which is `basket-review`'s job (D-03,
`.planning/phases/02-quick-add-basket-review/02-CONTEXT.md`). This is a manual test-case list —
there is no automated runner. Run each case by opening a fresh chat in the claude.ai Project (both
skills uploaded) and typing the exact phrasing, then record which skill actually fired.

Per research Pattern 1 (CLAUDE.md): overlapping trigger vocabulary across grocery skills is the
single biggest cause of wrong-skill firing once more than 2-3 skills share a domain. This file
specifically guards the `quick-add` ↔ `basket-review` boundary — confirm "add milk and bananas"
fires `quick-add` and never `basket-review`.

## Must-trigger cases (expect: quick-add fires)

| # | Input phrasing | Language | Expected skill | Expected NOT-triggered | Result (pass/fail) |
|---|-----------------|----------|-----------------|-------------------------|---------------------|
| 1 | "add milk and bananas" | English | quick-add | basket-review | |
| 2 | "we need eggs" | English | quick-add | basket-review | |
| 3 | "přidej máslo" | Czech | quick-add | basket-review | |
| 4 | "přidej mléko a banány" | Czech | quick-add | basket-review | |
| 5 | "add rohlíky and some milk" | Mixed CZ/EN | quick-add | basket-review | |

(≥3 required by acceptance criteria; 5 provided, one Czech, one mixed-language, to exercise D-08
bilingual acceptance.)

## Cross-skill near-miss cases (expect: quick-add does NOT fire)

These phrasings must trigger a **different** skill, never `quick-add` — proving the disjoint
trigger vocabulary from D-03 holds in practice.

| # | Input phrasing | Language | Expected skill | Must NOT trigger | Result (pass/fail) |
|---|-----------------|----------|-----------------|-------------------|---------------------|
| 1 | "what's in the basket" | English | basket-review | quick-add | |
| 2 | "co je v košíku" | Czech | basket-review | quick-add | |
| 3 | "what's our running total" | English | basket-review | quick-add | |
| 4 | "restock the usual" | English | (future: staples-restock — not built yet in Phase 2; at minimum must NOT be treated as a quick-add "add these named items" request) | quick-add | |

## How to read a result

For each row, "pass" means the skill that actually fired in a fresh chat matches the "Expected
skill" column, and the "Must NOT trigger" / "Expected NOT-triggered" skill did not fire instead or
alongside it. A "fail" on any near-miss row is a cross-firing bug in one of the two SKILL.md
descriptions and must be fixed (tighten the disjoint trigger vocabulary, per D-03) before this
plan's UAT (Task 3) is considered meaningful.

## Cross-references

- `skills/quick-add/SKILL.md` — the description under test.
- `skills/basket-review/SKILL.md` — the sibling skill this file guards against cross-firing with;
  see `skills/evals/basket-review-evals.md` for its mirrored eval set.
- `.planning/phases/02-quick-add-basket-review/02-CONTEXT.md` — D-03 (disjoint trigger vocabulary).
