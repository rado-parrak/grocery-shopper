# Writable-State Decision

**Implements:** FOUND-09 · D-08 (artifact-storage spike acceptance)

**Status:** ⏳ SCAFFOLD — UNRESOLVED. Awaiting the human-executed `spikes/artifact-storage-spike.md`
run and the household's confirmed binding (checkpoint Task 3 of plan 01-03). Nothing below is a
decision yet — it is the shape the decision will take once recorded.

**This is a one-way gate for Phase 4.** Whichever branch below gets dated and confirmed is what
Phase 4's learned-state skills (staples cadence, curated recipe repertoire, rejected-substitution
history, favourites write-back — LEARN-01 and neighbours) bind to. Switching stores later means
migrating whatever persisted learned state already exists under the first choice — this is not a
casually-revisitable setting.

---

## Decision

**Outcome:** *UNRESOLVED — GO / NO-GO / INCOMPLETE — HUMAN FILLS THIS*
**Date:** *HUMAN FILLS THIS — YYYY-MM-DD*
**Confirmed by:** *HUMAN FILLS THIS*

Fill in exactly ONE of the three branches below (delete or ignore the other two once resolved),
based on the outcome actually recorded in `spikes/artifact-storage-spike.md`.

---

### Branch: GO — artifact storage becomes the Phase 4 learned-state home

*Only valid if `artifact-storage-spike.md` recorded a real cross-device read-back — Step 0 found a
storage API, Step 1's write succeeded, and Step 2 actually read the value back on the second
device.*

- **Pool used:** *HUMAN FILLS THIS — "shared" / "personal" / undifferentiated*
- **Why this pool:** *HUMAN FILLS THIS*
- **What Phase 4 binds to it:** staples-restock cadence, curated recipe repertoire notes,
  rejected-substitution history, and any favourites write-back that needs to persist and be visible
  from both phones.
- **Known limits to carry forward:** unpublishing the artifact deletes all associated storage with
  no republish-to-same-URL recovery — Phase 4 skills must not casually unpublish the artifact this
  storage lives on.

### Branch: NO-GO — Rohlík-native + hand-edited Project files (fallback)

*Use this if Step 0 found no storage API at all, or the write never persisted, or it persisted but
never read back correctly on the second device (or only after an unacceptable delay).*

- **Which step failed:** *HUMAN FILLS THIS*
- **Fallback (fixed, not optional):** Rohlík-native favourites/order-history as the primary signal
  for anything the retailer's own account data already models; hand-edited diffs to the read-only
  Project-knowledge files (proposed by a future `household-prefs` skill in Phase 4, applied by a
  human) for anything Rohlík's data model can't hold (restock cadence, curated repertoire notes,
  rejected-substitution history — none of these have a Rohlík-native field).
- **What Phase 4 designs around instead:** no skill-writable, mid-conversation persistent store;
  Phase 4 learned-state features degrade to this two-part fallback.

### Branch: INCOMPLETE — defer, default to the NO-GO fallback until re-run

*Use this if only one physical device was available, or the spike was interrupted before Step 2.*

- **What was missing:** *HUMAN FILLS THIS*
- **Default posture until re-run:** treat as NO-GO — Phase 4 must NOT be designed against artifact
  storage on the strength of an incomplete (same-device-only) result. Use the NO-GO fallback above
  in the meantime.
- **Re-run trigger:** the moment a second physical device becomes available, re-run
  `spikes/artifact-storage-spike.md` in full and update this decision.

---

## Cross-references

- `spikes/artifact-storage-spike.md` — the protocol that produces the raw observations this
  decision transcribes.
- `.planning/phases/01-foundation-shared-spine/01-RESEARCH.md` §"Artifact-Storage Spike Protocol"
  and §"Open Questions" 1 — the grounding for why Step 0 exists and why GO requires a real
  cross-device read-back, not an assumption.
- `.planning/phases/01-foundation-shared-spine/01-CONTEXT.md` D-08 — the locked acceptance
  criteria this decision resolves.
- Phase 4 (`staples-restock`, `household-prefs`, and any other learned-state skill) — the
  consumers this decision binds; do not plan or build those skills against artifact storage before
  this decision is resolved to GO with a dated, cross-device-confirmed result.

## What NOT to do

- Do not mark GO without a recorded cross-device read-back actually observed on a second physical
  device — a same-device write/read is INCOMPLETE, never GO, regardless of how well it worked.
- Do not invent or guess at the outcome to unblock Phase 4 planning faster — an honest INCOMPLETE
  with the safe NO-GO fallback is a correct, valid state to leave this file in until the spike is
  actually re-run.
- Do not let any skill write to this file at chat time — it is human/agent-authored in this repo,
  outside live chat sessions, same as every other Project-knowledge doc.
