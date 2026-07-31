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
  - spikes/mcp-round-trip-results.md — EMPTY results template awaiting human-executed live round-trip
affects: [phase-2 (quick-add — first real consumer of mcp-degradation.md's observed tool surface), phase-4 (FOUND-09 spike, staples-restock, household-prefs)]

actuals:
  tokens: 12300
  tasks: 1
  commits: 2

tech-stack:
  added: []
  patterns:
    - "Human-executed, checkpoint-gated spike protocol authored by the agent, run by the household on claude.ai — cannot be run from this repo/CI"
    - "Empty, explicitly-labelled results template ('HUMAN FILLS THIS') committed ahead of the checkpoint, never fabricated"

key-files:
  created:
    - spikes/project-setup-checklist.md
    - spikes/mcp-round-trip-protocol.md
    - spikes/mcp-round-trip-results.md
  modified: []

key-decisions:
  - "Authored all three agent-authorable artifacts (setup checklist, probe protocol, empty results template) before stopping at the mandatory human checkpoint — no observations fabricated."
  - "mcp-degradation.md's Observed tool surface table is left exactly as authored in 01-01 (unpopulated) — Task 3 (transcription) cannot run until a human pastes back real round-trip observations."

patterns-established:
  - "Blocking-human checkpoints that require a live external product surface (claude.ai + real OAuth connector) are never simulated or guessed at by the executor — the agent prepares every artifact it can, then stops cleanly with an exact resume signal."

requirements-completed: []
# FOUND-04 is NOT complete yet — the plan is paused before Task 3 (recording observed
# tool surface). Do not mark FOUND-04 done until the human round-trip completes and
# Task 3 transcribes it into mcp-degradation.md.

coverage: []

duration: 3min
completed: 2026-07-31
status: paused
---

# Phase 1 Plan 02: MCP Round-Trip Setup & Protocol (PAUSED at human checkpoint) Summary

**Authored the go-live setup checklist and the 8-step reversible Rohlík MCP probe protocol, plus an empty results template — plan is PAUSED awaiting a human-executed live round-trip inside claude.ai; no tool names, parameters, or observations were fabricated.**

## Performance

- **Duration:** ~3 min (agent-authorable portion only; the human checkpoint duration is unknown/unbounded)
- **Started:** 2026-07-31T16:39:49Z
- **Completed (this session):** 2026-07-31T16:42:36Z
- **Tasks:** 1 of 3 completed (Task 1); Task 2 is the checkpoint (blocking); Task 3 not yet runnable
- **Files created:** 3

## Accomplishments

- `spikes/project-setup-checklist.md` — copy-pasteable pre-flight: plan-tier check, upload the eight `project-knowledge/` docs, attach the Rohlík MCP OAuth connector at "This project" scope and confirm Connected, and a credential-scan reminder — each item with an explicit tickable done-state.
- `spikes/mcp-round-trip-protocol.md` — the full 8-step reversible probe (search → `cart_read` baseline → reversible 1-unit add → `cart_read` verify → reversible remove → `cart_read` verify revert → forced error case → favourites-tool check), with the recording-table headers verbatim and a dated observed-live provenance rule. States plainly this cannot be run from this repo/CI.
- `spikes/mcp-round-trip-results.md` — an intentionally **empty** results template, every field marked `*HUMAN FILLS THIS*`, prepared ahead of the checkpoint so the human has a ready structure to fill in-session, without the executor guessing at or inventing any tool name, parameter, or return shape.

## Task Commits

Each task was committed atomically:

1. **Task 1: Author the go-live setup checklist and the MCP round-trip probe protocol** - `f68dfb3` (feat)
2. **(Preparatory, ahead of checkpoint) Author empty results template** - `d09b771` (docs)

**Plan metadata:** (this commit, once made) — recorded PAUSED, not complete.

Task 2 (`checkpoint:human-verify`, `gate="blocking-human"`) has NOT been executed — it requires a human, on a phone signed into the shared Claude account, operating the real claude.ai product with the Rohlík MCP OAuth connector attached. It cannot be run from this repository or CI. Task 3 (transcribing observations into `mcp-round-trip-results.md` and `project-knowledge/mcp-degradation.md`'s Observed tool surface table) cannot start until Task 2 produces real, pasted-back observations.

## Files Created/Modified

- `spikes/project-setup-checklist.md` - go-live pre-flight checklist (tier, docs upload, OAuth connect, credential scan)
- `spikes/mcp-round-trip-protocol.md` - the 8-step reversible live MCP probe protocol
- `spikes/mcp-round-trip-results.md` - empty results template, all fields marked HUMAN FILLS THIS

## Decisions Made

- Authored the empty results template (`mcp-round-trip-results.md`) as a preparatory artifact ahead of the checkpoint, rather than waiting for Task 3, so the human has the exact recording structure ready to fill during the live session. This does not fulfill Task 3 — Task 3 still requires transcribing the human's actual pasted-back observations and populating `mcp-degradation.md`'s Observed tool surface table, which remains untouched (still marked "to be filled by plan 01-02" per 01-01's authoring).
- No architectural deviations. Plan executed exactly as written up to the mandatory checkpoint.

## Deviations from Plan

None - plan executed exactly as written for every agent-authorable task. No auto-fixes, no Rule 1-4 triggers.

## Issues Encountered

None. This is not a failure state — the checkpoint is the plan's intended, designed stopping point (D-09: MCP discovery is empirical and human-executed, not doc-driven; it cannot be run from this repo/CI environment, which has no Rohlík MCP connector available).

## User Setup Required

**Yes — this IS the checkpoint.** The household must, on a phone signed into the shared Claude account:

1. Work through `spikes/project-setup-checklist.md`: confirm plan tier, upload the eight `project-knowledge/` docs to the shared Project's Knowledge, attach the Rohlík MCP OAuth connector at "This project" scope, confirm it reads **Connected**, and do the credential-scan pass.
2. Work through `spikes/mcp-round-trip-protocol.md` end to end: search → `cart_read` baseline → reversible 1-unit add → `cart_read` (verify present) → reversible remove → `cart_read` (verify baseline restored) → force one error case → check for a favourites-equivalent tool.
3. Paste back, for each of the 8 steps: the exact tool name invoked, the parameters passed, the return shape, and the exact error shape forced — scrubbing anything token-shaped first. If the round-trip cannot complete, paste back exactly which step failed and the exact symptom instead (a valid, recordable outcome).

**Resume signal:** the raw per-step observations (or "connector won't connect" / "MCP unavailable" plus the exact error), pasted back into the next turn with this plan.

## Next Phase Readiness

- Not ready to advance past 01-02 — Task 3 (populate `mcp-degradation.md`'s Observed tool surface, resolve the favourites-tool question) is still blocked on the human round-trip.
- 01-03 (if it exists / the FOUND-09 artifact-storage spike) is independent of this blocker and could proceed in parallel if separately scheduled, but per plan dependencies this plan's own Task 3 must wait.
- Phase 2 (quick-add) should NOT begin skill-authoring against `mcp-degradation.md`'s tool names until this plan's Task 3 completes — the observed tool surface is what Phase 2 will actually call.

---
*Phase: 01-foundation-shared-spine*
*Completed (this session): 2026-07-31 — PAUSED at human checkpoint, not yet complete*

## Self-Check: PASSED

- FOUND: spikes/project-setup-checklist.md
- FOUND: spikes/mcp-round-trip-protocol.md
- FOUND: spikes/mcp-round-trip-results.md
- FOUND: .planning/phases/01-foundation-shared-spine/01-02-SUMMARY.md
- FOUND commit: f68dfb3
- FOUND commit: d09b771
