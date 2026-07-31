# Rohlík MCP Round-Trip — Results

**Implements:** FOUND-04 · D-09

**Status:** ⏳ EMPTY TEMPLATE — awaiting the human-executed live round-trip.

This file is intentionally empty. It will be filled in **after** a human runs
`mcp-round-trip-protocol.md` end-to-end inside a real claude.ai chat, with the Rohlík MCP OAuth
connector connected, per `project-setup-checklist.md`. Nothing below should be treated as observed
fact until that round-trip actually runs and this file is transcribed with real, dated
observations.

**⚠ HUMAN FILLS THIS.** Do not populate any row below from assumption, training knowledge, the
community `tomaspavlin/rohlik-mcp` tool-name hypothesis, or any other non-live source. Every row
must come from an actual live tool call observed in a real chat session, dated, and marked
"observed live, [date]".

---

## Pre-flight confirmation

- [ ] `project-setup-checklist.md` fully completed (tier confirmed, 8 docs uploaded, connector
      shows Connected, credential scan clean) — *HUMAN FILLS THIS*
- **Date round-trip run:** *HUMAN FILLS THIS — YYYY-MM-DD*
- **Run by / device:** *HUMAN FILLS THIS*

## Step-by-step raw observations

### Step 1 — Product search
- Tool name invoked: *HUMAN FILLS THIS*
- Parameters passed: *HUMAN FILLS THIS*
- Return shape (product-ID field, price field, unit/pack-size field, stock field): *HUMAN FILLS THIS*

### Step 2 — `cart_read` (baseline)
- Tool name invoked: *HUMAN FILLS THIS*
- Parameter shape: *HUMAN FILLS THIS*
- Return shape (fields per line item, total field, currency format): *HUMAN FILLS THIS*

### Step 3 — Reversible add (1 unit)
- Tool name invoked: *HUMAN FILLS THIS*
- Parameters passed: *HUMAN FILLS THIS*
- Tool's own return value (note: not to be trusted at face value): *HUMAN FILLS THIS*

### Step 4 — `cart_read` (verify add)
- Confirmed item present at correct qty/price? *HUMAN FILLS THIS — yes/no*
- Did Step 3's return value alone match what this read-back showed? *HUMAN FILLS THIS*

### Step 5 — Reversible remove
- Tool name invoked: *HUMAN FILLS THIS*
- Parameters it required (cart-line ID / product ID / quantity delta — which?): *HUMAN FILLS THIS*

### Step 6 — `cart_read` (verify revert)
- Confirmed cart matches Step 2 baseline exactly? *HUMAN FILLS THIS — yes/no*

### Step 7 — Forced error case
- Action taken to force the error: *HUMAN FILLS THIS*
- Exact error shape returned (HTTP-like status / MCP protocol error / plain string / message text):
  *HUMAN FILLS THIS*

### Step 8 — Favourites-equivalent tool check
- Present / absent / couldn't determine: *HUMAN FILLS THIS*
- If present, tool name and what it returns: *HUMAN FILLS THIS*

## Recording table (transcribe the above into this shape once filled)

| Observed tool name | Matches community-list hypothesis? | Parameters (as observed) | Return shape (as observed) | Error shape (as observed, if forced) | Notes |
|---|---|---|---|---|---|
| *HUMAN FILLS THIS* | | | | | |

## If the round-trip could not complete

- **Blocked at step:** *HUMAN FILLS THIS (or N/A if completed)*
- **Exact blocking message/symptom:** *HUMAN FILLS THIS*
- If blocked, `mcp-degradation.md`'s Observed tool surface section stays UNVERIFIED with this
  blocking reason recorded, and the policy remains in force with tool names flagged as unconfirmed
  hypotheses — no observed values are invented to fill the gap.

---

*This results doc is transcribed into `project-knowledge/mcp-degradation.md`'s "Observed tool
surface" table once filled — see that file's provenance requirement.*
