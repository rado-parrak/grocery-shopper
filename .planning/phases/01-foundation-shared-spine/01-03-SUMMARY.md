---
phase: 01-foundation-shared-spine
plan: 03
subsystem: infra
tags: [artifacts, storage-capability, spike, project-knowledge, writable-state]

# Dependency graph
requires:
  - phase: 01-foundation-shared-spine (01-01)
    provides: the five shared-internal contracts and the household-ruleset.md/budget.md config files this plan personalises
provides:
  - spikes/artifact-storage-spike.md — Step-0 existence check first (no storage capability assumed), then Device-A write / Device-B read-back protocol, single-device case flagged INCOMPLETE
  - project-knowledge/writable-state-decision.md — GO/NO-GO/INCOMPLETE scaffold, decision left UNRESOLVED pending the human spike
affects: [phase-4 (FOUND-09 gate — staples-restock, household-prefs, any learned-state skill binds to this decision)]

actuals:
  tokens: 5300
  tasks: 1
  commits: 1

tech-stack:
  added: []
  patterns:
    - "Human-executed, checkpoint-gated spike protocol authored by the agent, run by the household on two physical devices — cannot be run from this repo/CI"
    - "Step-0 existence check placed before any write/read test, per this phase's research finding that the live capability roster lists only downloads/mcp, not storage"
    - "Decision scaffold committed with the outcome genuinely UNRESOLVED — never fabricated to unblock Phase 4 planning early"

key-files:
  created:
    - spikes/artifact-storage-spike.md
    - project-knowledge/writable-state-decision.md
  modified: []

key-decisions:
  - "Authored the storage-spike protocol and the go/no-go scaffold before stopping at the mandatory human checkpoint — no spike outcome fabricated, no writable-state binding decided by the agent."
  - "household-ruleset.md and budget.md ⚠ FILL placeholders left exactly as they were (untouched) — the household's private allergy/dislike/brand/budget values were not invented."

patterns-established:
  - "Blocking-human checkpoints requiring a live external product surface (claude.ai on two physical devices) are never simulated or guessed at by the executor — the agent authors every artifact it can, then stops cleanly with an exact resume signal."

requirements-completed: []
# FOUND-09 is NOT complete yet — the plan is paused before Task 2 (human runs the spike) and
# Task 3 (decision checkpoint) and Task 4 (record outcome + personalise config). Do not mark
# FOUND-09 done until the human runs the spike, confirms the binding, and Task 4 transcribes it.

coverage: []

duration: 4min
completed: 2026-07-31
status: paused
---

# Phase 1 Plan 03: Artifact-Storage Spike & Household Personalisation (PAUSED at human checkpoint) Summary

**Authored the Step-0-first artifact-storage spike protocol and an unresolved GO/NO-GO/INCOMPLETE decision scaffold — plan is PAUSED awaiting the household to run the spike on two physical devices and provide their private ⚠ FILL values; no spike outcome or household data was fabricated.**

## Performance

- **Duration:** ~4 min (agent-authorable portion only; the human checkpoint duration is unknown/unbounded)
- **Started:** 2026-07-31T16:46:14Z
- **Completed (this session):** 2026-07-31T16:50:00Z
- **Tasks:** 1 of 4 completed (Task 1); Task 2 is the blocking-human checkpoint; Task 3 is a decision checkpoint gated on Task 2's outcome; Task 4 not yet runnable
- **Files created:** 2

## Accomplishments

- `spikes/artifact-storage-spike.md` — the human-executed protocol. Leads with **Step 0**: check
  whether a storage-shaped capability exists at all (capability picker + `window.claude.storage` /
  `window.storage` console check) before any write/read attempt, since this phase's research found
  the live capability roster lists only `downloads` and `mcp`. If neither exists, the protocol
  records NO-GO immediately and stops — a fast, valid, cheap outcome. Only if one exists: Step 1
  (write a test key from Device A through the normal chat flow) and Step 2 (read it back from
  Device B, second phone, same shared account, fresh session). Explicitly flags a single-device run
  as **INCOMPLETE**, never NO-GO, because it cannot answer the cross-device question. States plainly
  the spike is human-executed and not runnable from this repo/CI.
- `project-knowledge/writable-state-decision.md` — a scaffold with the decision line left
  **UNRESOLVED** (placeholder outcome/date/confirmed-by fields) and three fully-described outcome
  branches to fill: GO (names the pool used, what Phase 4 binds to it), NO-GO (fixed fallback:
  Rohlík-native favourites/order-history as primary + hand-edited Project-file diffs proposed by a
  future `household-prefs` skill), and INCOMPLETE (defaults to the NO-GO fallback until re-run).
  States this is a one-way gate for Phase 4 dependents.

## Task Commits

Each task was committed atomically:

1. **Task 1: Author the artifact-storage spike protocol and the go/no-go decision scaffold** - `4af1a17` (feat)

**Plan metadata:** (this commit, once made) — recorded PAUSED, not complete.

Task 2 (`checkpoint:human-verify`, `gate="blocking-human"`) has NOT been executed — it requires a
human, on two physical phones signed into the shared Claude account, running the storage spike, and
separately providing the household's private ⚠ FILL values (allergies, dislikes/never-buy, brand
preferences, budget soft/hard amounts). Neither can be produced or guessed at by the executor.

Task 3 (`checkpoint:decision`, `gate="blocking"`) cannot run until Task 2 produces a real spike
outcome — the decision's options are only meaningful once the spike result is known.

Task 4 (record the dated go/no-go + personalise household-ruleset.md/budget.md) cannot start until
Tasks 2 and 3 complete.

## Files Created/Modified

- `spikes/artifact-storage-spike.md` - Step-0-first cross-device storage-capability spike protocol
- `project-knowledge/writable-state-decision.md` - unresolved GO/NO-GO/INCOMPLETE decision scaffold

`project-knowledge/household-ruleset.md` and `project-knowledge/budget.md` were **read but not
modified** — their ⚠ FILL placeholders (allergies, dislikes/never-buy, brand preferences, budget
soft/hard amounts) remain exactly as authored in 01-01, per this plan's Task 4 (not yet run).

## Decisions Made

- Placed the Step-0 existence check literally first in the protocol, ahead of any write/read
  attempt, per 01-RESEARCH.md's finding that this session's own live capability roster lists only
  `downloads`/`mcp` — treating a fast NO-GO from Step 0 as a valid, cheap, correct outcome rather
  than an incomplete spike.
- Left `writable-state-decision.md`'s outcome genuinely unresolved rather than pre-selecting a
  "likely" branch — the plan's own must-haves require no GO without a recorded cross-device
  read-back, so any agent-guessed outcome would violate that gate.
- No architectural deviations. Plan executed exactly as written up to the mandatory checkpoint.

## Deviations from Plan

None - plan executed exactly as written for the one agent-authorable task. No auto-fixes, no Rule
1-4 triggers.

## Issues Encountered

None. This is not a failure state — the checkpoint is the plan's intended, designed stopping point
(D-08: the spike must be run on two physical devices signed into the shared account, and the
household's private ⚠ FILL values can only come from the household itself; neither is producible
from this repo/CI environment).

## User Setup Required

**Yes — this IS the checkpoint.** The household must:

1. **(A) Storage spike** — work through `spikes/artifact-storage-spike.md`: Step 0 first (does a
   storage capability exist at all — check both the capability picker and `window.claude.storage` /
   `window.storage`); if it doesn't, note "storage API missing" and stop (valid NO-GO). If it does,
   write a test key from Device A, then read it back from Device B (second phone, same shared
   account, fresh session), noting whether it returned and any delay. If only one device is
   available, run same-device write/read and mark it INCOMPLETE.
2. **(B) Personalise** — paste real values for the ⚠ FILL fields: allergies, dislikes/never-buy,
   brand preferences (`household-ruleset.md` §A/§B), and soft/hard budget amounts in CZK
   (`budget.md`); or reply "fill later" to defer to before the first real shop.
3. Then, at Task 3, confirm which writable-state branch (GO / NO-GO / INCOMPLETE) Phase 4 binds
   to — the option must be consistent with the Task 2 spike result (GO requires a recorded
   cross-device read-back).

**Resume signal:** paste (A) the storage-spike outcome — "storage API missing" / the cross-device
read result / "one device only" — and (B) the ⚠ FILL values or "fill later"; then select the Task 3
decision option once the spike outcome is known.

## Next Phase Readiness

- Not ready to advance past 01-03 — Tasks 2, 3, and 4 are blocked on the human spike, the
  household's private values, and the resulting decision confirmation.
- FOUND-09 stays open until Task 4 records a dated, spike-consistent outcome in
  `writable-state-decision.md`.
- Phase 4 (staples-restock, household-prefs, and any other learned-state skill) must NOT be planned
  or built against artifact storage until this plan's decision resolves to GO with a documented
  cross-device read-back.
- Plan 01-02 (MCP round-trip) remains separately paused at its own blocking-human checkpoint; this
  plan's pause is independent (01-03 depends only on 01-01, not 01-02) and does not block or get
  blocked by 01-02's resolution.

---
*Phase: 01-foundation-shared-spine*
*Completed (this session): 2026-07-31 — PAUSED at human checkpoint, not yet complete*

## Self-Check: PASSED

- FOUND: spikes/artifact-storage-spike.md
- FOUND: project-knowledge/writable-state-decision.md
- FOUND: .planning/phases/01-foundation-shared-spine/01-03-SUMMARY.md
- FOUND commit: 4af1a17
