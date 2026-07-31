# Budget

**Implements:** FOUND-07 · D-04, D-05

**Status:** Read-only household config — hand-edited by the household, never written by a skill at
chat time. `confirmation-protocol.md` renders these thresholds on the confirmation card; every
mutating skill checks the hard cap before writing.

---

**Currency:** CZK

**Soft threshold (warn):** 2000 Kč — adjust to your household
**Hard cap (block):** 3000 Kč — adjust to your household

---

## Cap semantics (D-05)

The **hard cap applies to the projected whole-basket total** — the current cart total *plus* the
items about to be added — **not to a single add**. The shared basket is the unit of spend, and
both household members add to the same basket, so a per-add check alone would miss a basket that
crept over budget through several small adds.

Before any write, every mutating skill must:

1. **Read the cart** (`Rohlík:cart_read`, per `mcp-degradation.md`) to get the current total.
2. **Project the total**: current cart total + the price of the items about to be added.
3. **Compare against the hard cap.** If the projected total is **over** the hard cap: refuse to
   write, and ask the user to either cut items from the proposed batch or explicitly raise the cap
   in this file. Never write past the hard cap silently.
4. **Compare against the soft threshold.** If the projected total is **at or over** the soft
   threshold but still under the hard cap: proceed, but warn the user plainly (e.g. on the
   confirmation card and/or in the audit reply) that the basket is approaching the limit.

The soft threshold is advisory only — it never blocks a write. The hard cap is the only value that
blocks.

## Cross-references

- `confirmation-protocol.md` — renders the soft/hard lines on the confirmation card before any
  write.
- `mcp-degradation.md` — the `cart_read` this budget check reuses (read-before-write).
- `audit-format.md` — the running basket total line reported after a write; should be read
  alongside these thresholds when informing the user of proximity to the cap.

## What NOT to do

- Do not apply the hard cap to a single item's price — it applies to the projected whole-basket
  total.
- Do not write past the hard cap and ask forgiveness afterward — refuse first, then ask the user
  to cut items or raise the cap.
- Do not have any skill write to this file — it is human-edited only.
