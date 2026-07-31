# Resolution Cascade

**Implements:** FOUND-01 · D-01, D-02, D-03

**Status:** Shared internal — the single source of truth for how any vague grocery request
becomes a specific Rohlík product. Every skill (quick-add, staples-restock, recipe-to-basket,
meal-plan, basket-review) that resolves an item MUST follow this doc exactly and MUST NOT
re-derive, summarize, or paraphrase these steps inside its own SKILL.md. If this doc changes,
every skill picks up the change automatically by reference — that is the entire point of keeping
it here and nowhere else.

## Why this exists

Five-plus skills all resolve grocery items. Without one shared cascade, each skill would grow its
own slightly-different resolution logic, and a fix or a new rule in one copy would silently fail
to propagate to the others — the exact drift failure this project's architecture is designed to
prevent (see CLAUDE.md Architecture Pattern 3, PITFALLS.md P4/P5/P6).

## The cascade

For **every** requested item, run these four steps **in this order, every time, with no
shortcuts**:

### 1. Hard constraints (household-ruleset.md §A) — unconditional, first, absolute

Eliminate any candidate that violates an entry in `household-ruleset.md` §A (allergies,
dislikes/never-buy). This step:

- Runs **first**, before preferences, before favourites, before anything else.
- Runs **unconditionally** — it is never skipped, never made optional, never deferred "to save a
  turn."
- Is **absolute** — a violating candidate is never proposed, never substituted in, never
  surfaced to the user as an option, not even as a "did you mean" fallback.
- **A favourite (seed-favourites.md, or Rohlík-native favourites/order-history once confirmed
  available) can never override a hard constraint.** If a favourite product violates §A, it is
  discarded exactly as any other candidate would be, with no exception because it was previously
  bought or marked as preferred.

This is the safety invariant this whole document exists to protect (STATE.md §Safety Invariants;
CONTEXT.md D-03). Allergies default to the literal string `none recorded — unconfirmed` in
household-ruleset.md — never treat an unfilled allergy field as "no allergies" or "safe to ignore."
An unconfirmed allergy field means: treat as if a real, unspecified constraint may exist, and
prefer asking (step 4) over guessing, for anything allergy-adjacent.

### 2. Preferences (household-ruleset.md §B) — rank what remains

Of the candidates that survive step 1, rank them using household-ruleset.md §B's ranked,
per-category preference rules (e.g. BIO > farm-sourced > cheapest acceptable for produce; Czech
origin where sensible; smallest adequate pack size). Preferences never re-admit a candidate
step 1 already eliminated — ranking only ever narrows or orders what's left, never expands it.

### 3. Favourites — fill confident matches

If step 2 doesn't produce one clearly-best candidate, consult favourites: today,
`seed-favourites.md`'s hand-curated pre-approved product-ID list; later, Rohlík-native
favourites/order-history once the live MCP round-trip (plan 01-02) confirms whether such a tool
exists. A favourite is still subject to step 1's hard-constraint filter and step 2's preference
ranking — it is a *source of candidates*, never a bypass of the two steps above it.

### 4. Ask — only if 1–3 all fail

Only when hard constraints, preferences, and favourites together fail to produce a confident
match, ask the user. Per the turn-economy constraint (PROJECT.md, CLAUDE.md), **batch every
unresolved item from the current request into one clarifying turn** — never ask one question per
item, and never ask about an item the ruleset already answered.

## Cross-references

- `household-ruleset.md` §A / §B — the hard-constraint and preference data this cascade reads.
- `seed-favourites.md` — the bootstrap favourites source consulted at step 3.
- `substitution-policy.md` — re-runs this exact cascade (not a separate procedure) when an
  originally-resolved item turns out to be out of stock.
- `audit-format.md` — records, for every item actually added, which step resolved it (a rule ID
  from §B, or `oblíbené` for a favourite match).

## What NOT to do

- Do not skip step 1 "because the user asked for a specific brand" — a hard constraint still
  filters that specific brand out if it violates §A.
- Do not let a favourite short-circuit past step 1 or step 2.
- Do not ask the user about an item that hard constraints, preferences, or favourites already
  resolved confidently — this wastes the ≤3-turn budget the whole product is built around.
- Do not re-describe this cascade's steps inside any skill's own SKILL.md; reference this file by
  name instead.
