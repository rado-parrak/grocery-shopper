# Rohlík MCP Round-Trip Protocol

**Implements:** FOUND-04 · D-09 (MCP discovery is empirical, not doc-driven)

**Who runs this:** A human, inside a real claude.ai chat, in the shared household Project, with
the Rohlík MCP connector already showing **Connected** (see `project-setup-checklist.md` — complete
that first). **This step cannot be run from this repository or any CI process** — it requires a
live claude.ai chat session and a real Rohlík account with catalogue access.

**Why:** Official Rohlík MCP docs are inaccessible (403 to automated fetch). The only detailed
community tool-name list belongs to a *different*, unofficial, username/password-authenticated
project (`tomaspavlin/rohlik-mcp`) — not the official OAuth-based `mcp.rohlik.cz` server this
project uses. The only way to learn the real tool names, parameters, and error shapes is to run
this probe live and record exactly what happens.

**Net effect on the real basket:** zero. Every step is reversible; the one item added is removed in
the same session, and the final `cart_read` confirms the cart is back to baseline.

---

## Before you start

- Confirm `project-setup-checklist.md` is fully ticked (connector Connected, docs uploaded).
- Pick a **cheap, common, easily-reverted item** to search for (e.g. a specific plain bread roll /
  rohlík) — something whose accidental presence or absence in the cart is a non-issue.
- Have this doc open on the same device so you can copy the recording table headers as you go.

## Provenance rule (applies to every row you record)

Every observation you record — in this protocol's working notes and later transcribed into
`mcp-round-trip-results.md` — must be **dated** and marked **"observed live, [date]"**. Never write
down a tool name, parameter, or shape you did not personally see returned in this session. Where an
observed tool name differs from the community-list naming hypothesis below, note **both** the
observed name and the hypothesis name side by side.

**Community naming hypothesis (reference only — NOT ground truth):** `search_products`,
`add_to_cart`, `get_cart_content`, `get_frequent_items`, etc. — from the unofficial
`tomaspavlin/rohlik-mcp` project. Use this only to sanity-check what you observe; if the real tool
names differ, that is expected and fine — record what you actually saw.

## Recording table headers (copy verbatim into your working notes / the results doc)

| Observed tool name | Matches community-list hypothesis? | Parameters (as observed) | Return shape (as observed) | Error shape (as observed, if forced) | Notes |
|---|---|---|---|---|---|

---

## The probe sequence

Run these steps **in order, in one sitting**, in the same chat.

### Step 1 — Product search

Ask Claude to search for the throwaway item (e.g. "search Rohlík for [plain bread roll]").

**Record:**
- The exact tool name Claude invoked (as shown in its tool-use trace/citation).
- The exact parameters Claude passed (the search query string, any filters).
- The shape of the returned result: the field name used for product ID, the field name for price,
  the field name for unit/pack size, and whether a stock/availability field is present and what
  it's called.

### Step 2 — `cart_read` (baseline)

Ask Claude to read the current cart contents, **before any mutation**.

**Record:**
- The exact tool name invoked.
- The parameter shape (expect none, or confirm if something is required).
- The returned shape: what fields exist per line item, whether a total field exists, and the
  currency format used.

This is your baseline — you'll compare back to this exact state at the end.

### Step 3 — Reversible add (exactly 1 unit)

Ask Claude to add **exactly 1 unit** of the searched throwaway item to the cart.

**Record:**
- The exact tool name invoked.
- The parameters passed — specifically, confirm the product-ID field name it expects matches what
  Step 1's search actually returned.
- The tool's own return value, **and explicitly note that this return value is not to be trusted at
  face value** — per D-09 and `mcp-degradation.md`, only Step 4's read-back can prove the add
  actually happened.

### Step 4 — `cart_read` (verify the add)

Re-read the cart immediately after Step 3.

**Record:**
- Confirm the throwaway item is now actually present, at the right quantity (1) and a sane price.
- This is the empirical basis for `mcp-degradation.md`'s "never report success from the write
  call's own return value" rule — note explicitly whether Step 3's return value would have been
  sufficient on its own, or whether this read-back caught something the write's return value didn't
  show.

### Step 5 — Reversible remove

Ask Claude to remove the same throwaway item from the cart, restoring it to baseline.

**Record:**
- The exact tool name invoked.
- The parameters it required — a cart-line ID, a product ID, or a quantity delta? Which one?

### Step 6 — `cart_read` (verify revert)

Re-read the cart once more.

**Record:**
- Confirm the cart now matches Step 2's baseline exactly (same items, same quantities, same total).
- This closes the loop with **zero net change** to the real household basket.

### Step 7 — Force one error case

Deliberately trigger a failure — e.g. call the add tool again with an invalid/expired product ID,
or a deliberately malformed parameter (wrong type, missing required field).

**Record:**
- The exact error shape returned: is it an HTTP-like status code, an MCP protocol-level error, or a
  plain error string? What does the message actually say?
- This is the only way to write a real (not hypothetical) failure branch into `mcp-degradation.md`.

### Step 8 — Check for a favourites-equivalent tool

While the available tools are visible (Claude will typically list callable tools when asked, or
they're visible via the connector's tool picker in claude.ai's UI), specifically look for anything
resembling a "frequent items" / "favourites" tool (the unofficial project calls its version
`get_frequent_items`).

**Record:**
- Present / absent / couldn't determine.
- If present: note its exact name and what it returns. This resolves whether `seed-favourites.md`'s
  bootstrap role can be scoped as explicitly temporary.

---

## What to paste back

For each of the 8 steps above, paste the raw observations (tool name, parameters, return/error
shape) — using the recording table headers above as the structure. **Scrub anything token-shaped**
(anything that looks like a bearer token, API key, client secret, or authorization header) before
pasting — replace it with `[REDACTED]` if genuinely necessary to note its presence at all.

If the round-trip could not complete (connector disconnected mid-probe, a step hung, an
unrecoverable error), **stop and paste back exactly which step failed and the exact message/symptom
you saw** — a recorded blocker is a valid, useful outcome. Do not guess at what a later step "would
have" returned.
