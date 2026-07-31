# Seed Favourites

**Implements:** FOUND-08 · D-01 (favourites step of the cascade), specifics §Placeholders

**Status:** Read-only household config — hand-edited by the household, never written by a skill at
chat time. This is the **bootstrap** favourites source `resolution-cascade.md` step 3 reads before
any Rohlík-native favourites/order-history tool is confirmed available (see the "Bootstrap vs.
steady-state" note below).

---

## What this file is

A hand-editable list of pre-approved Rohlík product IDs — items the household already knows it
wants, keyed by a short label so the cascade (and a human skimming this file) can match a request
to a specific product without needing a live search.

**Every entry below is a placeholder.** Replace `⚠ FILL — product ID` with the real Rohlík product
ID once known. Product IDs can be:
- Entered by hand as the household discovers them through normal shopping, or
- Bootstrapped from Rohlík order history during the live MCP round-trip (plan 01-02), if the
  official server exposes a favourites-equivalent tool (see `mcp-degradation.md`'s "Observed tool
  surface" section, step 8 of the discovery protocol).

## Format

```
- {label} — {Rohlík product ID} — {optional note}
```

## Starter list (placeholders)

- Milk (plnotučné, default brand) — ⚠ FILL — product ID
- Eggs (free-range / z podestýlky) — ⚠ FILL — product ID
- Rohlíky (plain bread rolls) — ⚠ FILL — product ID
- Bananas — ⚠ FILL — product ID

_Add more rows as the household's regulars become clear._

## Important: favourites are still filtered

A favourite listed here is a **candidate**, not a bypass. Per `resolution-cascade.md` step 1,
**every** favourite is still subject to §A hard-constraint filtering (allergies, never-buy) before
it can ever be proposed or added — being on this list does not exempt an item from that check. If
household-ruleset.md §A ever changes such that a previously-favourited item now violates a hard
constraint, that favourite must be treated as any other eliminated candidate.

## Bootstrap vs. steady-state (CONFIRMED temporary — resolved 2026-07-31)

This file's role is explicitly **temporary/bootstrap**. Plan 01-02's live MCP round-trip
(2026-07-31) **confirmed** the official Rohlík MCP exposes two favourites/order-history-equivalent
tools: `Rohlik:get_all_user_favorites` (28 items observed live) and `Rohlik:get_typical_order`
(order-history-derived `frequent_items[]`) — see `mcp-degradation.md`'s "Observed tool surface"
section. This is no longer a theoretical possibility pending a check; the Rohlík-native source
exists and is populated today, and becomes the steady-state favourites signal (durable, shared
across the one household account, no new infrastructure — see STACK.md's writable-state analysis).
This file remains useful as a hand-curated override/supplement even so — Rohlík-native favourites
is populated by the retailer's own purchase-history logic, not directly controllable by a skill the
way a household might want to *deliberately* pin a specific product.

**Candidate matches from the live favourites/order-history data** (recorded in
`spikes/mcp-round-trip-results.md`, NOT auto-written into the placeholders below — the household
must review and confirm before replacing any `⚠ FILL`):
- Milk → `1443104` (Miil BIO Čerstvé mléko plnotučné 3,6 %) — in both favourites and frequent items
- Eggs → `1316385` (Schubert BIO Natur vejce M–L) or `1466294` (Pohodová vejce z volného výběhu) — both in favourites
- Rohlíky → not in favourites; `1346771` (Antonínovo pekařství Karlínský rohlík) is a frequent-order candidate
- Bananas → not in favourites; `1349785` (Banán Chiquita 1 ks) is a frequent-order candidate

## Cross-references

- `resolution-cascade.md` — step 3 reads this file (or Rohlík-native favourites, once confirmed)
  only after §A/§B have already filtered and ranked candidates.
- `household-ruleset.md` §A — every favourite here is still subject to hard-constraint filtering.
- `mcp-degradation.md` — records whether a favourites-equivalent MCP tool exists, resolving this
  file's bootstrap-vs-steady-state timeline.

## What NOT to do

- Do not treat a favourite as exempt from hard-constraint filtering.
- Do not have any skill write to this file — it is human-edited only (learned-state write-back is
  deferred to Phase 4, gated on the FOUND-09 spike outcome).
- Do not assume the placeholder product IDs are real — replace them before relying on this file for
  an actual shop.
