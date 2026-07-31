---
name: basket-review
description: >
  Reads the shared Rohlík basket and reports what's already in it — items, running total, and
  budget-threshold state — for requests like "what's in the basket", "check the cart", "what's our
  running total", or "co je v košíku" / "kolik toho máme v košíku", in Czech, English, or mixed. Use
  when the user wants to inspect or review the existing basket, not add anything to it. Do NOT use
  when the user names one or more specific grocery items to buy right now (e.g. naming a product
  and asking for it to be added, or "we need eggs", "přidej máslo") — that is quick-add's job, not
  basket-review's. This skill never mutates the cart and never calls checkout, timeslot,
  payment-method, order-management, or claim tools.
---

# basket-review

Answers "what's in the basket" by reading the **live** shared cart and reporting it honestly: items
with a running total. This skill is a thin orchestrator: it invokes the shared `project-knowledge/`
contracts by name and never re-derives their internal steps here. If a contract changes, this
skill picks up the change automatically by reference — see CLAUDE.md Architecture Pattern 3.

**Scope: read-only, forever.** This skill never writes to the cart and never calls checkout,
timeslot, payment-method, order-management, or claim tools — see `mcp-degradation.md`'s "Forbidden
tools — never call" section, which is the single source of truth for exactly what is off-limits;
this skill defers to it rather than repeating the list here.

## Happy path

1. **Read the live cart.** Call the fully-qualified `Rohlik:get_cart` tool. This is the only tool
   call this skill ever makes — there is no resolution, no confirmation artifact, and no write.
2. **List the items.** Render each cart line with its quantity and per-line price. Product names
   render in Czech, verbatim from the catalogue, per `audit-format.md`'s Language Rule — defer to
   that rule by name; do not restate it here, even when the user's own request was entirely in
   English. Keep the reply phone-terse: single column, one short line per item, no wide tables, no
   multi-column layouts.
3. **Show the running total.** Report the basket total using the same running-total line
   convention as `audit-format.md` ("Celkem v košíku: {basket total} Kč") — reference that
   convention by name; do not invent a different total format for this skill.
