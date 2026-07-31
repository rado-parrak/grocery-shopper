# Human UAT — quick-add + basket-review (≤3-turn measurement)

**Requirement measured:** UX-04 — a household member can run the real end-to-end quick-add on a
phone and reach the shared basket in **≤3 turns** with correct products and a Czech audit, then
confirm `basket-review` reads that basket back honestly.

**Who runs this:** a household member, on a phone, signed into the shared Claude account, inside
the shared claude.ai Project. This is **not** runnable from this repo, from Claude Code, or from
any CI — it requires the real Rohlík OAuth connector and the real shared basket. Nothing in this
file is executed by an agent; every step below is performed by a human, and results are recorded by
a human.

## Precondition — check before starting

Do **not** start this UAT until both of the following are true:

1. **Connector set up.** The Rohlík MCP connector is already authorized (This-project scope) on the
   shared Claude account, per `skills/README.md`'s "Rohlík MCP connector" section.
2. **⚠ FILL data completed.** `project-knowledge/household-ruleset.md` §A (allergies, dislikes) and
   §B (milk preference, brand preferences) are filled in, and `project-knowledge/budget.md`'s
   soft/hard CZK amounts are set to the household's real numbers — not left at their placeholder
   defaults. Running the real shop against unfilled `⚠ FILL` fields risks an allergy-adjacent
   mismatch or a meaningless budget check.

If either precondition is unmet, stop here and complete it first (see `skills/README.md` and the
two `project-knowledge/` files directly) — do not run the real-basket steps below against
placeholder data.

## Part 1 — quick-add: the ≤3-turn path

### Setup

Open a **fresh chat** in the shared claude.ai Project (both `quick-add` and `basket-review` already
uploaded per `skills/README.md`, and the Rohlík connector authorized). Have this checklist open on
the same phone or a second device so you can check items off as you go.

### Turn 1 — name the items (English)

Type exactly:

```
add milk and bananas
```

Check, in order:

- [ ] **`quick-add` triggers** (not `basket-review`, not a generic "how can I help" reply).
- [ ] The reply loads the household ruleset and budget silently (no question asked about anything
      the ruleset already answers, per the ≤3-turn budget).
- [ ] **A confirmation artifact renders** with: a checkbox + quantity stepper per item, a running
      total line, and the budget soft/hard lines from `budget.md`.
- [ ] The reply itself (outside the artifact) is short and phone-terse — no wide table, no wall of
      options, single column.

**Turn 1 result:** PASS / FAIL — notes: ______________________

### Turn 2 — confirm

In the artifact (or by typing a plain confirming reply if you prefer to test the natural-language
path), confirm the proposed items as shown — e.g. reply:

```
yes add these
```

Check, in order:

- [ ] The skill reconciles against the pre-write cart snapshot (no duplicate items land if you'd
      already had milk or bananas in the basket before Turn 1).
- [ ] The write happens via `Rohlik:add_items_to_cart`, then the skill **reads the cart back**
      before reporting anything (per `mcp-degradation.md` — never trust the write's own return).
- [ ] **The correct Czech-named products land in the shared basket** — verify by asking
      `basket-review` afterward (Part 2) or by checking the Rohlík app/site directly: the product
      names shown are real Czech catalogue names (e.g. "Mléko polotučné 1 l"), not English
      placeholders or literal translations of "milk"/"bananas".
- [ ] **A one-line Czech audit** is emitted per item, in the exact `audit-format.md` shape
      (`✓ {qty}× {Czech product name} — {line total} Kč · {why}`), followed by a
      `Celkem v košíku: {total} Kč` line.
- [ ] **No forbidden tool was ever called** — no checkout, timeslot, payment-method,
      order-management, or claim tool fired at any point in Turns 1–2 (nothing you did should have
      triggered an actual order or payment; check the Rohlík app shows the items in the *cart*,
      not as a placed order).

**Turn 2 result:** PASS / FAIL — notes: ______________________

### Turn count check

- [ ] **The whole path (Turn 1 → Turn 2) completed in ≤3 turns.** A 3rd turn is only acceptable if
      the cascade genuinely could not resolve one of the items (e.g. it had to ask a clarifying
      question) — record whether a 3rd turn happened and, if so, why.

**≤3-turn result:** PASS / FAIL — turns actually used: ____ — reason if >2: ______________________

### Turn 1 (repeat) — Czech variant

Open another fresh chat and type exactly:

```
přidej mléko a banány
```

Repeat the same checks as the English Turn 1/Turn 2 above (trigger, artifact, confirm, read-back,
Czech audit, ≤3 turns). This is the bilingual half of UX-04/D-08 — the request is in Czech this
time, and the audit/product names should look identical in shape to the English run (product names
are always Czech regardless of the request's language, per `audit-format.md`'s Language Rule).

**Czech-variant result:** PASS / FAIL — notes: ______________________

## Part 2 — basket-review: read-back + budget + manual-checkout handoff

In a fresh chat (or continuing the same one), type:

```
what's in the basket
```

Check, in order:

- [ ] **`basket-review` triggers** (not `quick-add` — it must not try to resolve or add anything).
- [ ] It calls `Rohlik:get_cart` and lists the items actually just added in Part 1 (plus anything
      else already in the shared basket), each with quantity and per-line price, product names in
      Czech.
- [ ] It reports the running total using the `Celkem v košíku: {total} Kč` convention.
- [ ] It reports **budget-threshold state** against `budget.md`'s real (filled-in) soft/hard
      amounts — a plain warning if at/over the soft threshold, a plain statement if at/over the
      hard cap, and correctly silent about both if the basket is comfortably under.
- [ ] It states that **checkout is a manual step** the household completes directly in the Rohlík
      app, and does not itself initiate, prepare, or touch checkout in any way.
- [ ] **No forbidden tool was called** by `basket-review` at any point — it only ever calls
      `Rohlik:get_cart`.

**basket-review result:** PASS / FAIL — notes: ______________________

## Overall UAT result

- [ ] Part 1 (English) — PASS / FAIL
- [ ] Part 1 (Czech) — PASS / FAIL
- [ ] Part 2 (basket-review) — PASS / FAIL
- [ ] **UX-04 (≤3-turn quick-add path) — PASS / FAIL overall**

Report the pass/fail per checklist item (and any notes) back into the conversation that requested
this UAT, so it can be recorded in this plan's SUMMARY and in `.planning/STATE.md`.

## Cross-references

- `skills/quick-add/SKILL.md`, `skills/basket-review/SKILL.md` — the skills under test.
- `skills/evals/quick-add-evals.md`, `skills/evals/basket-review-evals.md` — the trigger-only eval
  sets this UAT builds on (this file additionally measures the full ≤3-turn write/read-back path,
  not just triggering).
- `project-knowledge/household-ruleset.md`, `project-knowledge/budget.md` — the ⚠ FILL data this
  UAT's precondition requires.
- `project-knowledge/audit-format.md`, `project-knowledge/mcp-degradation.md` — the exact audit
  shape and read-before-write/read-back discipline this UAT checks for.
- `skills/README.md` — connector setup and upload flow, required before this UAT can run at all.
