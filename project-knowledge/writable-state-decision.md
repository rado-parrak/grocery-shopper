# Writable-State Decision

**Implements:** FOUND-09 · D-08 (artifact-storage spike acceptance)

**Status:** ✅ RESOLVED — 2026-07-31. The household RAN the two-device
`spikes/artifact-storage-spike.md`. The `window.storage` capability **is present** and a same-device
write+read-back succeeds, but the **cross-device read fails with a server-side error**
(`Internal server error while processing action`), reproducibly — so **artifact cross-device storage
does NOT work as tested**. This is a **tested NO-GO on artifact storage** (specifically the
cross-device path), not an assumption. Writable state rides on the confirmed-working Rohlík-native
path. FOUND-09's go/no-go is determined.

---

## Observed spike result — household, 2026-07-31 (verbatim)

> **FOUND-09 · D-08 — Artifact storage cross-device spike**
> **Result: NO-GO (observed live, 2026-07-31)**
>
> - **Step 0 (capability exists):** Pass. `window.storage` confirmed present (not `window.claude.storage`); `get`/`set`/`delete`/`list`, shared/personal pools via a `shared` boolean.
> - **Step 1 (write, Device A, shared pool):** Pass. `window.storage.set("found09_spike_test", ..., true)` succeeded, confirmed by a successful same-device read-back afterward — the write genuinely persisted server-side, not just a locally-optimistic success.
> - **Step 2 (read, Device B, same account, fresh session):** Fail. `window.storage.get("found09_spike_test", true)` returned `Storage get failed: Internal server error while processing action` — a server-side error, not an empty/absent result. Reproducible (failed on first attempt and again on retry) while the same key read successfully on Device A throughout.
> - **Interpretation:** the failure mode is a hard error rather than delayed propagation, so this isn't a "wait longer and it'll show up" case as tested. Whether the underlying cause is that shared-pool storage isn't actually cross-device-synced, or a bug in the cross-device read path specifically, isn't determinable from the client side — recorded as observed, not diagnosed.
> - **Consequence for FOUND-09/Phase 4:** learned-state write-back via artifact storage's shared pool is not viable as tested. Any future revisit needs to reproduce this NO-GO going away before being treated as GO — per the spike doc, no re-litigating this via a same-device test alone.
>
> Worth noting: this also quietly settles which direction Phase 4 should lean, if it revisits write-back at all — `seed-favourites.md` already flagged Rohlík-native favourites (confirmed present via `Rohlik:get_all_user_favorites` in the last spike) as durable and shared across the account with no new infrastructure. This result is a concrete reason to prefer that path over artifact storage, rather than just a convenience argument.

**⚠ OPEN — REVISIT BEFORE PHASE 4:** with artifact storage confirmed non-functional, household state
that Rohlík cannot model (restock cadence, rejected-substitution history, explicit "we switched to
brand X") has **no working writable store**. The interim is hand-edited Project-file diffs (below);
a durable store for this is an unsolved question the household explicitly flagged to revisit before
Phase 4 (staples-restock / household-prefs).

**Outstanding, non-blocking to-do (unaffected by this decision):** `household-ruleset.md` §A/§B and
`budget.md`'s `⚠ FILL BEFORE FIRST REAL SHOP` placeholders (allergies, dislikes/never-buy, brand
preferences, budget soft/hard amounts) remain unfilled — they were left untouched, not fabricated.
They must be filled with the household's real values before the first real shop, but do not block
this decision or FOUND-09's completion.

**This is a one-way gate for Phase 4** in the sense that Phase 4's learned-state skills (staples
cadence, curated recipe repertoire, rejected-substitution history, favourites write-back — LEARN-01
and neighbours) bind to whatever is recorded below. It remains **provisional and human-revisable**:
the household explicitly delegated rather than definitively closed the underlying artifact-storage
question, so this decision can be revisited before Phase 4 without waiting for a one-way-door
justification — see "Reversibility" below.

---

## Decision

**Outcome:** **NO-GO on artifact storage (household-tested — does not work cross-device) / GO on
Rohlík-native writable state.** Writable state for v1 = Rohlík-native (`Rohlik:get_all_user_favorites`,
`Rohlik:get_typical_order`) as the primary signal, plus hand-edited Project-knowledge diffs for
anything Rohlík can't model. Artifact storage is **NOT USED in v1**.
**Date:** 2026-07-31
**Confirmed by:** The household ran the two-device spike; artifact cross-device storage did not work
(no confirmed cross-device read-back), 2026-07-31.
**Status of this outcome:** Tested NO-GO on artifact storage; revisitable only if the runtime later
gains a working storage capability — see "Reversibility" below.

### Rationale

- The live MCP round-trip (2026-07-31, plan 01-02) confirmed `Rohlik:get_all_user_favorites` (28
  items observed live) and `Rohlik:get_typical_order` (order-history-derived `frequent_items[]`)
  already persist server-side on the single shared Rohlík account — and are therefore inherently
  cross-device, since both phones sign into the same account. This independently answers the
  cross-device writable-state capability question FOUND-09 was created to settle, without needing
  the artifact-storage spike.
- Phase-1 research predicted the artifact `storage` capability might be absent. The household's
  two-device spike (2026-07-31) found the opposite on *existence* — `window.storage` **is** present
  (`get`/`set`/`delete`/`list`; shared/personal pools) and a same-device write+read-back succeeds —
  but the **cross-device read fails with a reproducible server-side error**
  (`Internal server error while processing action`) while the same key reads fine on the writing
  device. So the NO-GO is specifically the cross-device propagation path, and it is **tested, not
  assumed**. (See "Observed spike result" above for the step-by-step.)
- Therefore: writable state for v1 rides on Rohlík-native favourites + order history. Anything
  Rohlík cannot model (e.g. an explicit "we decided brand X" instruction, restock-cadence notes,
  rejected-substitution history) is handled by hand-edited Project Knowledge files (the ⚠ FILL
  fields and their successors), surfaced as diffs for the household to apply — **never
  auto-written** by a skill at chat time.

### Reversibility

If a real artifact-storage (or other) writable store is confirmed later (e.g. the runtime's
capability roster changes to include `storage`), Phase 4 learned-state can migrate to it. This
decision is **provisional** and the household can revisit it before Phase 4 — re-running
`spikes/artifact-storage-spike.md` in full is the way to upgrade this from "skipped, superseded"
to a directly-tested GO or NO-GO.

### Spike disposition

`spikes/artifact-storage-spike.md`'s Task 2 (the human-executed, two-device spike) was **RUN by the
household (2026-07-31) and returned NO-GO — artifact cross-device storage does not work.** The
protocol remains valid and re-runnable if the runtime later changes (see Reversibility above).

### What Phase 4 binds to

- **Primary:** Rohlík-native favourites (`Rohlik:get_all_user_favorites`) and order-history
  (`Rohlik:get_typical_order`) as the steady-state signal for anything the retailer's own account
  data already models.
- **Secondary (fixed fallback for what Rohlík can't model):** hand-edited diffs to the read-only
  Project-knowledge files (proposed by a future `household-prefs` skill in Phase 4, applied by a
  human) for restock cadence, curated repertoire notes, rejected-substitution history — none of
  these have a Rohlík-native field.
- **Not used:** no skill-writable, mid-conversation artifact-storage API. Phase 4 learned-state
  features must design around the two-part fallback above, not a live persistent store.

---

## Cross-references

- `spikes/artifact-storage-spike.md` — the storage-spike protocol; **executed by the household
  2026-07-31 → NO-GO on the cross-device read** (Step 0/1 passed, Step 2 failed). Remains valid and
  re-runnable if the runtime's storage behaviour changes.
- `spikes/mcp-round-trip-results.md` and `project-knowledge/mcp-degradation.md` ("Observed tool
  surface") — the live, dated MCP round-trip that confirmed `get_all_user_favorites` and
  `get_typical_order` as the Rohlík-native writable-state signal this decision relies on.
- `.planning/phases/01-foundation-shared-spine/01-RESEARCH.md` §"Artifact-Storage Spike Protocol"
  and §"Open Questions" 1 — the grounding for why Step 0 exists and why a GO on artifact storage
  would have required a real cross-device read-back, not an assumption.
- `.planning/phases/01-foundation-shared-spine/01-CONTEXT.md` D-08 — the locked acceptance
  criteria this decision resolves.
- Phase 4 (`staples-restock`, `household-prefs`, and any other learned-state skill) — the
  consumers this decision binds; design those skills against Rohlík-native favourites/order-history
  + hand-edited Project-file diffs, not against artifact storage.

## What NOT to do

- Artifact storage is a **tested NO-GO** (household ran the spike 2026-07-31; it does not work
  cross-device). Do not design any v1 feature against artifact storage. If revisiting, re-run the
  spike — do not assume the result flipped.
- Do not design a Phase 4 skill against artifact storage on the strength of this decision — the
  bound path is Rohlík-native favourites/order-history + hand-edited Project-file diffs, per "What
  Phase 4 binds to" above.
- Do not let any skill write to this file at chat time — it is human/agent-authored in this repo,
  outside live chat sessions, same as every other Project-knowledge doc.
