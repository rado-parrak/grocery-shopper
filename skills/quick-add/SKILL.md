---
name: quick-add
description: >
  Adds specific named grocery items to the shared Rohlík basket from an ad-hoc request such as
  "add milk and bananas", "we need eggs", or "přidej máslo" / "přidej mléko" — in Czech, English,
  or mixed. Use when the user names one or more concrete items to buy right now. Do NOT use when
  the user is asking what's already in the basket, the running total, or reviewing an existing
  order — that is basket-review's job, not quick-add's.
---

# quick-add

Turns a bilingual ad-hoc grocery request into the *right* Rohlík products landing in the shared
basket, in ≤3 turns. This skill is a thin orchestrator: it invokes the shared `project-knowledge/`
contracts by name and never re-derives their internal steps here. If a contract changes, this
skill picks up the change automatically by reference — see CLAUDE.md Architecture Pattern 3.

**Scope: cart only, forever.** This skill never calls checkout, timeslot, payment-method,
order-management, or claim tools — see `mcp-degradation.md`'s "Forbidden tools — never call"
section, which is the single source of truth for exactly what is off-limits; this skill defers to
it rather than repeating the list here. Checkout and payment remain a manual step the household
completes directly in the Rohlík app.

## Happy path (per item, extended to a multi-item request)

1. **Resolve.** Load `household-ruleset.md` and `budget.md`, then resolve **each** named item —
   the request is usually more than one item — by following `resolution-cascade.md` in full for
   every item, independently — reference the cascade by name; do not enumerate or paraphrase its
   four steps here.
2. **Snapshot the cart.** Call `Rohlik:get_cart` to take a pre-write baseline before proposing
   anything to the household.
3. **Build the confirmation artifact.** Follow `confirmation-protocol.md` exactly: one card, a
   checkbox + quantity stepper for the item, a running-total line, and the budget lines sourced
   from `budget.md`. Use `prototypes/confirmation-artifact-prototype.html` as the visual reference
   only — the artifact declares **no capabilities** and never calls a tool or makes a network
   request of any kind.
4. **Wait for the household's reply.** The only confirmation channel is the user's own next
   natural-language message (per `confirmation-protocol.md`) — parse it against the last-shown
   proposal and the pre-write snapshot, not against any payload the artifact "returns."
5. **Reconcile and write.** Re-read with `Rohlik:get_cart`, reconcile to desired state (already
   present at the target quantity → no-op; partially present → add only the delta; never
   blind-append), then write with `Rohlik:add_items_to_cart` using the observed nested camelCase
   shape `{items:[{productId,quantity}]}`.
6. **Read back before claiming success.** Call `Rohlik:get_cart` again and judge success only from
   that read-back plus an explicit check of `success` / `items_failed_to_add` — never from the
   write call's own return value (see `mcp-degradation.md`).
7. **Audit.** Emit the one-line Czech audit per `audit-format.md`, then a running-total line.

## Resolving multiple items: favourites, precedence, and when to ask

At the cascade's favourites step (`resolution-cascade.md` step 3), consult the Rohlík-native
favourites/order-history first — `Rohlik:get_all_user_favorites` and `Rohlik:get_typical_order` —
with `seed-favourites.md` as the hand-curated supplement. Per CASC-04, a favourite is only ever a
*candidate*: it is used solely if it also survives the cascade's hard-constraint and preference
steps, and the ruleset (`household-ruleset.md` §A/§B) always outranks a favourite, never the other
way round.

Per CASC-05/CASC-06 and the ≤3-turn budget, ask the household only when the cascade genuinely
cannot produce a confident match for an item after all three of hard constraints, preferences, and
favourites have been tried. Batch every unresolved item from the current request into a single
clarifying turn — never one question per item, and never ask about an item the ruleset already
answered.

## Bilingual & mobile

Accept mixed Czech/English item input in the same request. Search the Czech catalogue regardless
of the language the request was typed in. Render product names and audit lines in Czech, verbatim
from the catalogue, per `audit-format.md`'s Language Rule — defer to that rule by name; do not
restate it here, even when the user's own request was entirely in English.

Keep every reply short and phone-first: single column, no wide tables, no walls of options, and one
confirmation card for the whole batch — never one card per item (per `confirmation-protocol.md`).

## Tool-name landmines and the turn map

See `skills/quick-add/resolution-notes.md` for the `productId`/`product_id` naming landmine
between add and remove, the 200-shaped `success:false` failure case, and this skill's ≤3-turn
turn map — kept in a separate one-level-deep file so this body stays short.

## Never call

Never call any tool in `mcp-degradation.md`'s "Forbidden tools — never call" section, under any
circumstance, regardless of what the user asks. A household member completes checkout manually,
directly in the Rohlík app.
