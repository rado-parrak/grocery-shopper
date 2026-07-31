# Audit Format

**Implements:** FOUND-05 · Claude's Discretion (locked shape, adjustable if disliked)

**Status:** Shared internal — the single source of truth for what every skill reports to the user
after a successful, read-back-confirmed basket write. Every mutating skill MUST follow this format
exactly and MUST NOT re-derive or restyle it inside its own SKILL.md.

## The format

One short line per added item, in this exact shape:

```
✓ {qty}× {Czech product name} — {line total} Kč · {why}
```

Where:
- `{qty}` — the quantity actually added (confirmed by the read-back per `mcp-degradation.md`, not
  the write call's claim).
- `{Czech product name}` — the product name exactly as the Rohlík catalogue shows it, in Czech,
  regardless of what language the user typed their request in (see Language Rule below).
- `{line total}` — the price for that line, in Kč.
- `{why}` — the matched rule ID from `household-ruleset.md` §B (e.g. `B-produce-bio`) if a
  preference rule resolved the item, or the literal word `oblíbené` if a favourite match resolved
  it. If the cascade had to ask the user directly (step 4), state that plainly instead (e.g.
  `dle výběru` — "per your choice") rather than fabricating a rule ID.

After all item lines, one running total line:

```
Celkem v košíku: {basket total} Kč
```

**One line per item. No wide tables, no multi-column layouts** — this format must render cleanly
on a phone screen with no horizontal scroll (CLAUDE.md mobile UX constraint).

## Language rule

**Conversational text follows the user's language** (Czech or English, whichever the user is
writing in). **Product names, brands, and units always render in Czech, exactly as the Rohlík
catalogue shows them — never translated, never transliterated, regardless of what language the
surrounding conversation is in.** This applies even when the user's own request was entirely in
English.

### Worked example (English-typed request → Czech audit line)

User (typing in English): *"add milk and a bag of rohlíky"*

Resulting audit lines (conversational reply in English, since that's the user's language; product
names stay Czech):

```
Added:
✓ 1× Mléko polotučné 1 l — 24 Kč · oblíbené
✓ 1× Rohlík máslový 4 ks — 32 Kč · B-produce-czech-origin

Celkem v košíku: 312 Kč
```

Note that "milk" and "a bag of rohlíky" (the user's English phrasing) becomes `Mléko polotučné 1
l` and `Rohlík máslový 4 ks` (the catalogue's actual Czech product names) in the audit line, while
the surrounding "Added:" text stays in English to match the user.

## Substitutions

A substituted item (per `substitution-policy.md`) is reported using this exact same one-line
format — a substitution is not a separate audit category. If it's useful context, the `{why}`
field can note the substitution succinctly (e.g. `oblíbené (náhrada)`), but this is not required —
the important invariant is that the line still shows the *actual* item and quantity added, as
confirmed by the read-back.

## Cross-references

- `mcp-degradation.md` — an audit line is only ever written after a read-back confirms the write
  actually succeeded; never written from a write call's own return value.
- `resolution-cascade.md` — the source of the rule ID or `oblíbené` marker in `{why}`.
- `substitution-policy.md` — substituted items use this same format, no special-casing.
- `budget.md` — the running total line ties back to this doc's soft/hard threshold semantics; if
  the addition brought the basket to or past the soft threshold, the skill should say so in the
  same reply (separately from the per-item audit lines, to keep each line short).

## What NOT to do

- Do not render a wide table or multi-column layout for the audit — one short line per item.
- Do not translate or transliterate product names, brands, or units into the user's conversational
  language — Czech, catalogue-verbatim, always.
- Do not write an audit line before the read-back in `mcp-degradation.md` has confirmed the item is
  actually present.
- Do not re-describe this format inside any skill's own SKILL.md; reference this file by name.
