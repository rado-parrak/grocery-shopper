# Household Ruleset

**Implements:** FOUND-06 · D-01, D-02, D-03

**Status:** Read-only household config — hand-edited by the household, never written by a skill at
chat time. `resolution-cascade.md` reads this file at the start of every resolution. Sections are
ordered and read **top-down**: §A first, unconditionally, then §B, then §C.

---

## A. Hard Constraints (absolute — filter candidates out, never overridden)

These eliminate candidates. They are never relaxed for convenience, never overridden by a
favourite, and never skipped to save a turn. See `resolution-cascade.md` step 1.

- **Allergies:** ⚠ FILL BEFORE FIRST REAL SHOP — none recorded — unconfirmed
- **Dislikes / never-buy:** ⚠ FILL BEFORE FIRST REAL SHOP

> Per D-03: an unfilled allergy field means "none recorded — unconfirmed," **never** "no
> allergies." Treat it as an unknown, not a confirmed absence — prefer asking the household over
> guessing on anything allergy-adjacent until this field is filled in.

## B. Preferences (ranked, per category — rank what remains after §A)

These rank what's left after §A has already filtered candidates out. They never re-admit
something §A eliminated. Starter rules below are illustrative — edit freely.

- **Vegetables & fruit:** prefer **BIO**; else **farm-sourced** (farmářské); else cheapest
  acceptable.
- **Eggs:** free-range / z podestýlky or better.
- **Milk:** ⚠ FILL BEFORE FIRST REAL SHOP (preferred brand); default to **plnotučné** unless
  otherwise noted.
- **General:** prefer **Czech origin** where sensible.
- **Pack size:** prefer the **smallest pack** that meets the requested quantity (anti-waste;
  also supports pack-size dedup in later meal-plan resolution).
- **Brand preferences:** ⚠ FILL BEFORE FIRST REAL SHOP

## C. Notes

Free-form household context — seasonal notes, delivery-day preferences, anything that doesn't fit
§A or §B but is useful context for resolution. Empty for now; add as needed.

---

## Cross-references

- `resolution-cascade.md` — the procedure that reads this file, §A first and unconditionally, §B
  to rank what remains.
- `seed-favourites.md` — a separate, complementary source consulted only after §A/§B, per the
  cascade.
- `substitution-policy.md` — re-reads §A/§B via the cascade when re-resolving an out-of-stock item;
  never relaxes §A because the preferred option is unavailable.

## What NOT to do

- Do not treat an unfilled `⚠ FILL` field as permission to skip that check — treat it as unknown
  and prefer asking over guessing for anything safety-relevant.
- Do not let a §B preference or a favourite override a §A hard constraint.
- Do not have any skill write to this file — it is human-edited only.
