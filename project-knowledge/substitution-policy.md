# Substitution Policy

**Implements:** FOUND-03 · D-01, D-03 (precedence deferred, not restated)

**Status:** Shared internal — the single source of truth for what happens when an item resolved
earlier in a session turns out to be out of stock or otherwise unavailable at write time. Every
mutating skill MUST follow this policy exactly and MUST NOT re-derive it inside its own SKILL.md.

## The policy, in one sentence

An out-of-stock or unavailable item is **re-resolved through `resolution-cascade.md`** — never
silently swapped — and the substitute is proposed for the user's approval before it is added.

## The procedure

1. **Detect unavailability.** Discovered either at `Rohlík:search` time (no matching product) or
   at write time (`Rohlík:cart_add` fails or the read-back in `mcp-degradation.md` shows the item
   didn't actually land — e.g. delisted between search and write).
2. **Re-resolve through the cascade, from step 1.** Do not invent a special "substitution mode"
   with its own rules. Run the unavailable item back through `resolution-cascade.md` in full:
   hard constraints (household-ruleset.md §A) still eliminate violating candidates first and
   unconditionally — an out-of-stock situation is never an excuse to relax a hard constraint.
   Preferences (§B) rank what remains among the next candidates. Favourites may surface an
   alternative if one fits. This document defers all precedence questions to
   `resolution-cascade.md` — it does not restate or duplicate the ordering here.
3. **Never silently swap.** Whatever the cascade produces as the best remaining candidate is a
   **proposed substitute**, not a done deal. It is surfaced to the user through the same
   confirmation-artifact flow (`confirmation-protocol.md`) as any other item — the user sees what
   changed and why, and must confirm it like any other addition, before any write occurs.
4. **If the cascade still fails to produce a confident match** (e.g. every candidate in that
   category is unavailable or all remaining candidates are marginal), fall through to
   `resolution-cascade.md` step 4 — ask the user, batched with any other unresolved items in the
   same request, rather than guessing.
5. **Record the substitution in the audit line** (`audit-format.md`) so the household can see, at
   a glance, that what was added differs from what was originally requested — the audit format's
   `{why}` field carries the matched rule ID or `oblíbené` exactly as it would for any other
   resolved item; a substitution is not a separate audit category, it is simply a later resolution
   of the same item.

## Why re-resolve rather than special-case

A dedicated "find something similar" heuristic would duplicate the cascade's hard-constraint/
preference/favourite logic in a second place, reintroducing the exact drift risk
`resolution-cascade.md`'s own "Why this exists" section warns against. Out-of-stock handling is
not a different problem from first-time resolution — it is the same problem (pick the best
available candidate for this request) run again with a smaller candidate pool.

## Cross-references

- `resolution-cascade.md` — the full procedure this policy re-invokes; this document defers to it
  entirely rather than restating precedence.
- `confirmation-protocol.md` — how a proposed substitute is surfaced for approval before any
  write.
- `mcp-degradation.md` — how unavailability is actually detected at write time (failed add, or a
  read-back that doesn't show the expected state).
- `audit-format.md` — how a substituted item is reported once added.

## What NOT to do

- Do not silently swap an unavailable item for "something similar" without running it back through
  the cascade and getting user approval.
- Do not relax a hard constraint "just this once" because the preferred option is out of stock.
- Do not invent substitution-specific ranking rules — the cascade's existing preference ranking
  already covers "what's the next-best candidate."
- Do not re-describe hard-constraint-vs-preference precedence here; that lives only in
  `resolution-cascade.md`.
