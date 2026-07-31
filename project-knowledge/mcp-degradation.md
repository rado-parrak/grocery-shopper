# MCP Degradation Policy

**Implements:** FOUND-04 · D-09 (policy section; empirical tool surface recorded by plan 01-02)

**Status:** Shared internal — the single source of truth for how every skill talks to the Rohlík
MCP connector, and what it does when that connector misbehaves. Every mutating or reading skill
MUST follow this policy exactly and MUST NOT re-derive it inside its own SKILL.md.

## Why this exists

The official Rohlík MCP (`https://mcp.rohlik.cz/mcp`) is explicitly declared experimental and may
change without notice (CLAUDE.md, PROJECT.md). First-party does not mean hardened. No tool
annotation (`destructiveHint`, `idempotentHint`, `readOnlyHint`) should be trusted at face value,
and no write call's own return value should be trusted as proof of success — this policy exists to
make that distrust concrete and mechanical rather than a vague aspiration.

## The shape every mutating flow follows

Every Rohlík MCP call site — not just cart_add, every mutating call — follows this shape:

```
try:
  result = call Rohlík:<tool_name>(...)
except (timeout, error, unexpected shape):
  degrade: render a plain manual shopping list from whatever was already
  resolved; state explicitly "Rohlík connection isn't working right now —
  here's your list to add by hand"; never retry silently; never claim success.
```

Concretely, for any write:

1. **Read before write.** Call `Rohlík:cart_read` (fully-qualified tool name) immediately before
   computing what to add — not just once at the start of the conversation, but re-read right
   before the actual write, to minimize the staleness window given two household members may act
   on the shared basket concurrently.
2. **Reconcile-to-desired-state, never blind-append.** Treat "add" as: if the item is already
   present at the target quantity, no-op; if partially present, add only the delta; never append a
   duplicate line for an item already in the cart.
3. **Write.** Call the mutating tool (e.g. `Rohlík:cart_add`).
4. **Read back.** Immediately call `Rohlík:cart_read` again after the write.
5. **Report success only from the read-back.** Never phrase a success message directly off the
   write call's own return value. An experimental server's write acknowledgement may not reflect
   true state — the read-back is the only trustworthy signal.

## Degradation branch (mandatory at every call site, not bolted on later)

If search, `cart_read`, or any write fails, times out, or returns an unexpected shape:

- Fall back to producing a **plain, well-formatted manual shopping list** from whatever was
  already resolved.
- State plainly and explicitly that the Rohlík connection isn't working right now and here is the
  list to add by hand — never silently retry, never fabricate a success message, never leave the
  user unsure whether something was actually added.
- This degrade branch is designed in from the outset for every call site — it is not something to
  add later once a failure is observed in practice.

## Order submission is out of scope, by design

Order submission (checkout/payment) is **not exposed** by the Rohlík MCP — this is the platform's
own design choice, not a gap in this policy. No defensive code is needed here to prevent an
accidental checkout call, because no such call is ever possible through this connector. Every
*other* mutating call still needs the full discipline above, precisely because nothing else about
the server's reliability is guaranteed.

## Tool naming caution

A community, reverse-engineered, username/password-authenticated project
(`github.com/tomaspavlin/rohlik-mcp`) publishes a tool-name list (`search_products`,
`add_to_cart`, `get_cart_content`, `get_frequent_items`, etc.) that is **not** the official,
OAuth-based `mcp.rohlik.cz` server this project uses. Treat that list only as a **naming
hypothesis** to sanity-check observed tool names against — never as a substitute for the live
round-trip below. Official docs are inaccessible (403 to automated fetch), so this policy's actual
tool names, parameters, and error shapes can only come from empirical observation.

## Observed tool surface (filled during the live round-trip)

**This section is intentionally empty in this plan.** Plan 01-02 performs the live, human-executed
Rohlík MCP round-trip (product search → `cart_read` → reversible probe `cart_add` → `cart_remove`
→ `cart_read`, plus one deliberately-forced error case) and fills the table below with what was
actually observed. Until then, every tool name/parameter/error shape in this document is a
**hypothesis, not ground truth** — do not treat the shape above as verified.

| Observed tool name | Matches community-list hypothesis? | Parameters (as observed) | Return shape (as observed) | Error shape (as observed, if forced) | Degrade behavior if this call fails |
|---|---|---|---|---|---|
| *(to be filled by plan 01-02 — search)* | | | | | fall back to plain manual list, state degraded mode explicitly |
| *(to be filled by plan 01-02 — cart_read)* | | | | | fall back to plain manual list, state degraded mode explicitly |
| *(to be filled by plan 01-02 — cart_add)* | | | | | fall back to plain manual list, state degraded mode explicitly |
| *(to be filled by plan 01-02 — cart_remove)* | | | | | fall back to plain manual list, state degraded mode explicitly |
| *(to be filled by plan 01-02 — forced error case)* | | | | | fall back to plain manual list, state degraded mode explicitly |

**Provenance requirement:** every row, once filled, must be dated and marked "observed live,
[date]" — never populated from the community tool-name hypothesis without the live check. Where
the observed tool name differs from the hypothesis, note both.

**Favourites-equivalent tool check:** while enumerating available tools during the round-trip,
plan 01-02 also checks specifically for a `get_frequent_items`-equivalent tool. If one exists,
`seed-favourites.md`'s bootstrap role becomes explicitly temporary and this section should note
that finding.

## Cross-references

- `confirmation-protocol.md` — supplies the pre-write `cart_read` snapshot this policy's step 1
  reuses (the same read, not a second one, when the confirmation flow already took one recently).
- `substitution-policy.md` — how an unavailability discovered via this policy's write/read-back
  flow gets re-resolved.
- `audit-format.md` — what a successful, read-back-confirmed `cart_add` reports to the user.

## What NOT to do

- Do not report success from a write call's own return value.
- Do not retry a failed call silently — degrade to a manual list and say so.
- Do not treat the community `tomaspavlin/rohlik-mcp` tool list as ground truth.
- Do not populate the Observed tool surface table without a live, dated observation.
- Do not re-describe this policy inside any skill's own SKILL.md; reference this file by name.
