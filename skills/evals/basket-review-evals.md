# basket-review — Trigger Eval Set

**Purpose:** Confirm `skills/basket-review/SKILL.md`'s description reliably self-triggers on
phrasing that asks to inspect the existing shared basket — items, running total, budget state —
in Czech, English, or mixed. This is a manual test-case list — there is no automated runner. Run
each case by opening a fresh chat in the claude.ai Project (both skills uploaded) and typing the
exact phrasing, then record which skill actually fired.

Mirrors `skills/evals/quick-add-evals.md` from the opposite direction: this file specifically
confirms "what's in the basket" fires `basket-review` and never `quick-add`, per D-03
(`.planning/phases/02-quick-add-basket-review/02-CONTEXT.md`) and research Pattern 1 (CLAUDE.md) on
avoiding cross-skill collisions.

## Must-trigger cases (expect: basket-review fires)

| # | Input phrasing | Language | Expected skill | Expected NOT-triggered | Result (pass/fail) |
|---|-----------------|----------|-----------------|-------------------------|---------------------|
| 1 | "what's in the basket" | English | basket-review | quick-add | |
| 2 | "co je v košíku" | Czech | basket-review | quick-add | |
| 3 | "kolik toho máme v košíku" | Czech | basket-review | quick-add | |
| 4 | "check the cart / running total" | English | basket-review | quick-add | |
| 5 | "what's our running total" | English | basket-review | quick-add | |

(≥3 required by acceptance criteria; 5 provided, two Czech, to exercise D-08 bilingual acceptance.)

## Cross-skill near-miss cases (expect: basket-review does NOT fire)

These phrasings name specific items to buy right now and must trigger `quick-add` instead — never
`basket-review`, which only ever reads the cart and never resolves or adds new items.

| # | Input phrasing | Language | Expected skill | Must NOT trigger | Result (pass/fail) |
|---|-----------------|----------|-----------------|-------------------|---------------------|
| 1 | "add milk and bananas" | English | quick-add | basket-review | |
| 2 | "we need eggs" | English | quick-add | basket-review | |
| 3 | "přidej máslo" | Czech | quick-add | basket-review | |

## How to read a result

For each row, "pass" means the skill that actually fired in a fresh chat matches the "Expected
skill" column, and the sibling "Must NOT trigger" / "Expected NOT-triggered" skill did not fire
instead or alongside it. A "fail" on any near-miss row is a cross-firing bug and must be fixed
(tighten the disjoint trigger vocabulary, per D-03) before this plan's UAT (Task 3) is considered
meaningful.

## Cross-references

- `skills/basket-review/SKILL.md` — the description under test.
- `skills/quick-add/SKILL.md` — the sibling skill this file guards against cross-firing with; see
  `skills/evals/quick-add-evals.md` for its mirrored eval set.
- `.planning/phases/02-quick-add-basket-review/02-CONTEXT.md` — D-03 (disjoint trigger vocabulary).
