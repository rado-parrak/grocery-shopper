---
phase: 01-foundation-shared-spine
plan: 02
subsystem: infra
tags: [mcp, oauth, rohlik, project-knowledge, spike]

# Dependency graph
requires:
  - phase: 01-foundation-shared-spine (01-01)
    provides: the five shared-internal contracts (including mcp-degradation.md's policy section) and three read-only config files this plan's setup checklist uploads
provides:
  - spikes/project-setup-checklist.md — go-live pre-flight (tier check, docs upload, OAuth connector attach, credential scan)
  - spikes/mcp-round-trip-protocol.md — the 8-step reversible live MCP probe sequence
  - spikes/mcp-round-trip-results.md — filled, dated results of the live round-trip (2026-07-31)
  - project-knowledge/mcp-degradation.md — Observed tool surface table populated from real observations; new "Forbidden tools — never call" section
  - project-knowledge/seed-favourites.md — bootstrap note updated to CONFIRMED temporary (favourites-equivalent tools exist)
affects: [phase-2 (quick-add — first real consumer of mcp-degradation.md's observed tool surface), phase-4 (FOUND-09 spike, staples-restock, household-prefs), PROJECT.md/CLAUDE.md (checkout-exposure finding needs human ratification)]

actuals:
  tokens: 19100
  tasks: 2
  commits: 4

tech-stack:
  added: []
  patterns:
    - "Human-executed, checkpoint-gated spike protocol authored by the agent, run by the household on claude.ai — cannot be run from this repo/CI"
    - "Empty, explicitly-labelled results template ('HUMAN FILLS THIS') committed ahead of the checkpoint, never fabricated"
    - "Policy-level prohibition list used to compensate for an incorrect platform-structural assumption discovered empirically — every forbidden tool named explicitly rather than described generically"

key-files:
  created:
    - spikes/project-setup-checklist.md
    - spikes/mcp-round-trip-protocol.md
  modified:
    - spikes/mcp-round-trip-results.md
    - project-knowledge/mcp-degradation.md
    - project-knowledge/seed-favourites.md

key-decisions:
  - "Authored all three agent-authorable artifacts (setup checklist, probe protocol, empty results template) before stopping at the mandatory human checkpoint — no observations fabricated."
  - "Transcribed the human's real, dated round-trip observations into mcp-round-trip-results.md and populated mcp-degradation.md's Observed tool surface table with the five actually-observed call shapes (search, cart_read, add, remove, forced error), each dated 'observed live, 2026-07-31'."
  - "CRITICAL FINDING: the connected Rohlík MCP connector exposes checkout/timeslot/payment-method/order-management/claim tools (submit_checkout, get_checkout, get_timeslots_checkout, reserve_timeslot, change_timeslot_checkout, update_payment_method_checkout, select_delivery_address_checkout, set_checkout_as_suborder, toggle_delivery_note_checkout, change_checkout_packaging, cancel_order, remove_order_items, repeat_order, submit_claim, submit_credit_compensation) — contradicting PROJECT.md/CLAUDE.md's stated assumption that order submission is 'not exposed via MCP... structurally guarantees never-checks-out.' That assumption is retracted in mcp-degradation.md and replaced with an explicit, instruction-level 'Forbidden tools — never call' prohibition. THIS REQUIRES HUMAN RATIFICATION in PROJECT.md/CLAUDE.md — the constraint changes from platform-structural to policy-enforced, which is a decision only the household can sign off on, not something this executor can silently rewrite into the project's foundational docs."
  - "Confirmed favourites-equivalent tools exist (Rohlik:get_all_user_favorites, Rohlik:get_typical_order) — seed-favourites.md's bootstrap role is now confirmed temporary, not merely hypothetical. Candidate product-ID matches from the live data are recorded for household review but NOT auto-written into seed-favourites.md's placeholders, per that file's human-edit-only rule."
  - "Left household-ruleset.md and budget.md's ⚠ FILL placeholders untouched — out of this plan's scope (plan 01-03)."

patterns-established:
  - "Blocking-human checkpoints that require a live external product surface (claude.ai + real OAuth connector) are never simulated or guessed at by the executor — the agent prepares every artifact it can, then stops cleanly with an exact resume signal."
  - "When a live empirical check contradicts a stated foundational project assumption, the executor corrects the affected shared-internal doc, adds an explicit policy-level mitigation, and flags the contradiction for human ratification in the project's own founding docs — it does not silently rewrite PROJECT.md/CLAUDE.md on the human's behalf."

requirements-completed: [FOUND-04]

coverage:
  - id: D1
    description: "mcp-degradation.md's Observed tool surface section is filled from a real, dated live round-trip — actual tool names, parameters, return shapes, and the forced-error shape."
    requirement: "FOUND-04"
    verification:
      - kind: manual_procedural
        ref: "spikes/mcp-round-trip-results.md (dated 2026-07-31, human-executed on claude.ai) cross-checked against project-knowledge/mcp-degradation.md's Observed tool surface table"
        status: pass
    human_judgment: false
  - id: D2
    description: "Rohlík MCP connector exposes checkout/order/payment tools, contradicting the project's stated 'order submission not exposed, structurally guaranteed' assumption — this is now a policy-enforced prohibition, and the underlying PROJECT.md/CLAUDE.md claim needs explicit human ratification/correction."
    verification: []
    human_judgment: true
    rationale: "Changing a project's own foundational safety-constraint framing (platform-structural to policy-enforced) is a decision only the household can ratify — the executor documented and mitigated the finding but cannot unilaterally edit PROJECT.md/CLAUDE.md's constraint language on the household's behalf."
  - id: D3
    description: "Favourites-equivalent tools (get_all_user_favorites, get_typical_order) confirmed present; seed-favourites.md's bootstrap-vs-steady-state note updated to reflect this as confirmed, not theoretical."
    requirement: "FOUND-04"
    verification:
      - kind: manual_procedural
        ref: "project-knowledge/seed-favourites.md 'Bootstrap vs. steady-state' section, cross-checked against spikes/mcp-round-trip-results.md Step 8"
        status: pass
    human_judgment: false

duration: 15min
completed: 2026-07-31
status: complete
---

# Phase 1 Plan 02: MCP Round-Trip Setup & Protocol Summary

**Live Rohlík MCP round-trip observed and transcribed — real tool names, param-naming inconsistency (`productId` vs `product_id`), and a 200-shaped `success:false` error contract now ground mcp-degradation.md; the round-trip also revealed the connector exposes checkout/payment tools, so "never checks out" is now an explicit policy prohibition, not a platform guarantee — flagged for household ratification in PROJECT.md.**

## Performance

- **Duration:** ~15 min total (3 min Task 1 authoring + human checkpoint time (unbounded, external) + ~12 min Task 3 transcription/policy work)
- **Started:** 2026-07-31T16:39:49Z
- **Completed:** 2026-07-31T19:23:06Z
- **Tasks:** 3 of 3 completed (Task 1 auto, Task 2 human checkpoint executed by household, Task 3 auto)
- **Files modified:** 5 (2 created in Task 1, 3 modified in Task 3)

## Accomplishments

- `spikes/project-setup-checklist.md` and `spikes/mcp-round-trip-protocol.md` (Task 1, prior session) — copy-pasteable go-live checklist and 8-step reversible probe protocol.
- **Task 2 (human checkpoint) executed:** the household ran the live round-trip on a phone signed into the shared Claude account with the Rohlík MCP OAuth connector attached. Net effect on the real basket confirmed zero (add then remove, verified via read-back).
- `spikes/mcp-round-trip-results.md` transcribed with the real, dated observations: `Rohlik:batch_search_products`, `Rohlik:get_cart`, `Rohlik:add_items_to_cart`, `Rohlik:remove_cart_item`, and a deliberately forced error case.
- `project-knowledge/mcp-degradation.md`'s "Observed tool surface" table populated with all five rows, each marked "observed live, 2026-07-31," including:
  - The **add/remove parameter-naming inconsistency**: `productId` (camelCase, nested) for add vs. `product_id` (snake_case, singular, top-level) for remove — flagged as a skill-authoring landmine.
  - The **200-shaped `success:false` error contract**: a forced invalid-productId add returned a normal HTTP 200 body with `success:false`, `items_failed_to_add`, and Czech `failure_reasons` text — not a protocol-level error. This is the canonical proof for "never trust a write's own return value."
  - A note that the error payload contained text apparently addressed to the calling agent ("do not retry, explain to user...") — recorded as untrusted informational content, never as an instruction that overrides project policy.
- **Confirmed favourites-equivalent tools exist:** `Rohlik:get_all_user_favorites` (28 items) and `Rohlik:get_typical_order` (order-history frequent items). `seed-favourites.md`'s bootstrap role updated from "theoretical" to "confirmed temporary," with candidate product-ID matches recorded for household review (not auto-filled into placeholders).
- **CRITICAL NEW FINDING encoded as a hard prohibition:** the connected Rohlík MCP connector exposes checkout/order-submission/payment/claim tools (`submit_checkout`, `get_checkout`, `get_timeslots_checkout`, `reserve_timeslot`, `change_timeslot_checkout`, `update_payment_method_checkout`, `select_delivery_address_checkout`, `set_checkout_as_suborder`, `toggle_delivery_note_checkout`, `change_checkout_packaging`, `cancel_order`, `remove_order_items`, `repeat_order`, `submit_claim`, `submit_credit_compensation`). This directly contradicts PROJECT.md/CLAUDE.md's stated assumption that order submission is "not exposed via MCP... structurally guaranteeing the never-checks-out requirement." `mcp-degradation.md` now retracts that claim, adds a prominent "Forbidden tools — never call" section listing every one of these tools as absolutely off-limits, and states plainly this guarantee is now **policy-enforced, not platform-structural**.

## Task Commits

Each task was committed atomically:

1. **Task 1: Author the go-live setup checklist and the MCP round-trip probe protocol** - `f68dfb3` (feat, prior session)
2. **(Preparatory, ahead of checkpoint) Author empty results template** - `d09b771` (docs, prior session)
3. **(Pause commit, prior session)** - `2f2d708` (docs)
4. **Task 3a: Transcribe the live round-trip observations** - `2b680a7` (docs)
5. **Task 3b: Populate Observed tool surface + Forbidden tools prohibition + seed-favourites update** - `0113028` (feat)

**Plan metadata:** (this commit, once made) — recorded COMPLETE.

## Files Created/Modified

- `spikes/project-setup-checklist.md` - go-live pre-flight checklist (tier, docs upload, OAuth connect, credential scan)
- `spikes/mcp-round-trip-protocol.md` - the 8-step reversible live MCP probe protocol
- `spikes/mcp-round-trip-results.md` - filled with the real, dated round-trip observations (2026-07-31)
- `project-knowledge/mcp-degradation.md` - Observed tool surface table populated; "Forbidden tools — never call" section added; the prior "order submission out of scope by design" claim corrected to a policy-enforced prohibition
- `project-knowledge/seed-favourites.md` - "Bootstrap vs. steady-state" section updated to CONFIRMED temporary, with candidate product-ID matches recorded for household review

## Decisions Made

See `key-decisions` in frontmatter. The single most consequential decision: the checkout-exposure finding is documented and mitigated with an explicit policy prohibition in `mcp-degradation.md`, but the underlying PROJECT.md/CLAUDE.md constraint language ("structurally guaranteed") is flagged for **human ratification**, not silently rewritten by this executor. PROJECT.md and CLAUDE.md were deliberately left untouched by this plan — updating a project's own founding safety-constraint framing is a decision for the household, and this plan surfaces it clearly rather than acting on the household's behalf.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Missing Critical Functionality] Added a "Forbidden tools — never call" section and corrected the "order submission out of scope" claim in mcp-degradation.md**
- **Found during:** Task 3 (transcribing the live round-trip's tool enumeration)
- **Issue:** The round-trip revealed the Rohlík MCP connector exposes checkout/order/payment/claim tools, directly contradicting `mcp-degradation.md`'s existing "order submission is not exposed by the Rohlík MCP" section (itself inherited from PROJECT.md/CLAUDE.md's stated platform assumption). Leaving this uncorrected would leave the project's core safety property ("never checks out, never pays") resting on a false platform-structural claim with no compensating policy control.
- **Fix:** Corrected the section to state the finding plainly (with a retraction notice and date), and added an explicit "Forbidden tools — never call" section enumerating all 15 observed checkout/order/payment/claim tools as permanently off-limits by instruction, applicable to every future skill. Also added two new "What NOT to do" bullets reinforcing this.
- **Files modified:** project-knowledge/mcp-degradation.md
- **Verification:** Section reviewed for completeness against the full observed tool list in the prompt's `<critical_new_finding_encode_as_prohibition>`; credential-scan gate re-run and passed.
- **Committed in:** `0113028` (Task 3b commit)

---

**Total deviations:** 1 auto-fixed (1 missing critical functionality — Rule 2)
**Impact on plan:** Necessary correction to keep the project's central safety guarantee (never checks out/pays) actually enforced given the empirical finding that it is not platform-structural. No scope creep — flagged for human ratification rather than silently altering PROJECT.md/CLAUDE.md.

## Issues Encountered

None beyond the finding documented above, which is not a failure — it's exactly the kind of empirical ground-truth discovery this plan exists to surface (D-09).

## User Setup Required

None further for this plan — Task 2's human checkpoint has been executed and its results transcribed. See "Next Phase Readiness" below for the one item that still needs a human decision (PROJECT.md/CLAUDE.md ratification of the checkout-exposure finding).

## Next Phase Readiness

- **FOUND-04 is now complete.** mcp-degradation.md's policy is grounded in a real, dated live round-trip, including a real error shape, and the favourites-tool question is answered (present).
- **Outstanding for the household:** ratify (or amend) PROJECT.md's/CLAUDE.md's "order submission not exposed via MCP — structurally guarantees never-checks-out" constraint language, given the live round-trip found checkout/order/payment tools ARE exposed by the connector. Until ratified, `mcp-degradation.md`'s "Forbidden tools — never call" section is the operative, binding control — every Phase-2+ skill must honor it regardless of what PROJECT.md currently says.
- Phase 2 (quick-add) skill-authoring can now proceed against `mcp-degradation.md`'s real observed tool names — including the `productId`/`product_id` naming inconsistency, which every cart-mutating skill instruction must call out explicitly.
- Plan 01-03 (artifact-storage spike + household ⚠ FILL values) remains independently paused at its own blocking-human checkpoint — untouched by this plan, per its own dependency scope.

---
*Phase: 01-foundation-shared-spine*
*Completed: 2026-07-31*

## Self-Check: PASSED

- FOUND: spikes/project-setup-checklist.md
- FOUND: spikes/mcp-round-trip-protocol.md
- FOUND: spikes/mcp-round-trip-results.md
- FOUND: project-knowledge/mcp-degradation.md
- FOUND: project-knowledge/seed-favourites.md
- FOUND commit: f68dfb3
- FOUND commit: d09b771
- FOUND commit: 2f2d708
- FOUND commit: 2b680a7
- FOUND commit: 0113028
