# Artifact-Storage Cross-Device Spike

**Implements:** FOUND-09 · D-08 (artifact-storage spike acceptance)

**Status (2026-07-31): RAN → NO-GO.** Step 0 (capability exists) and Step 1 (write + same-device read-back) passed; **Step 2 (cross-device read on Device B, same account) failed** with `Storage get failed: Internal server error while processing action`, reproducibly. Result recorded verbatim in `project-knowledge/writable-state-decision.md`. Re-runnable if the runtime's storage behaviour changes.
The household delegated the writable-state decision rather than running this spike. Plan 01-02's
live MCP round-trip confirmed `Rohlik:get_all_user_favorites` and `Rohlik:get_typical_order`
already persist server-side on the one shared Rohlík account (inherently cross-device), which
independently answers the cross-device writable-state question this spike exists to settle. See
`project-knowledge/writable-state-decision.md` for the recorded, dated, provisional decision. This
protocol remains valid and re-runnable — see that decision's "Re-run trigger" — but was not
executed, and nothing below should be read as an executed result.

**Who runs this:** A human, on the shared household Claude account, using **two physical devices**
(both phones, both signed into the SAME shared account). **This step cannot be run from this
repository or any CI process** — it requires a real claude.ai session on real hardware, with a
plan tier that supports publishing artifacts (Pro/Max/Team/Enterprise).

**Why Step 0 exists (read this before doing anything else):** This phase's research found that
this session's own live artifact-capabilities roster lists only two declarable runtime
capabilities — `downloads` and `mcp`. No `storage` capability appears in it. Every description of
a `window.claude.storage` / `window.storage` persistent-storage API (20MB/artifact, personal +
shared pools) comes from community blogs, not a directly-fetched Anthropic doc — official
support.claude.com pages 403'd to automated fetch this session. That does **not** prove storage
doesn't exist (it may be an always-on API that isn't part of the viewer-consent capability
declaration mechanism `mcp`/`downloads` use), but it means this spike must **check existence
first**, before attempting any write/read test. Skipping straight to "write a key and see if it
reads back" risks burning spike time against an API that was never there.

**Net effect on the household:** a throwaway test key on a private, single-household published
artifact — no real household data, no cart interaction, no cost.

---

## Precondition

- Confirm both devices are signed into the **same shared Claude account** (not two separate
  accounts) — the whole point of this spike is the cross-device question, and it is meaningless
  with two different accounts.
- Confirm the account's plan tier supports publishing artifacts (should already be true if custom
  Skills work on this account, per the shared feature-gating tier, but confirm at spike time rather
  than assuming).
- If only **one** physical device is available right now: you can still run Step 0 and a
  same-device write/read, but the result must be recorded as **INCOMPLETE**, not NO-GO — a
  same-device test does not answer D-08's actual cross-device question. Re-run with the second
  device before treating the result as final.

---

## Step 0 — Does a storage capability exist at all? (run this FIRST, before any write/read test)

1. In the shared Project, ask Claude to create a small test artifact and check what runtime
   capabilities are available to declare on it. Note explicitly whether the capability picker /
   available options list anything storage-related, or only `downloads` and `mcp`.
2. Publish the artifact. Open its rendered, published page.
3. If you're able to, inspect the browser's JS console on the published page for the existence of
   either `window.claude.storage` or `window.storage` — check **both** names, since different
   secondary sources report different global names.
4. **Record what you found:**
   - Capability picker offered a storage-shaped option? yes / no / unclear
   - `window.claude.storage` exists in the published page's console? yes / no / couldn't check
   - `window.storage` exists? yes / no / couldn't check

5. **If neither exists:** record **NO-GO immediately** and stop here — skip Steps 1-2 entirely.
   This is a valid, fast, cheap outcome, not a failed spike. Go straight to "Step 3 — Record the
   go/no-go" below and write the NO-GO branch.
6. **If one exists:** proceed to Step 1.

---

## Step 1 — Write from Device A

7. From Device A (whichever phone you use first), through the household's **normal chat-driven
   flow** (ask Claude in a regular chat message — not a raw devtools console edit), have the
   artifact write a test value to a key, e.g. `test_favourite_v1 = "milk-2percent-test"`. If the
   API distinguishes "personal" vs. "shared" pools, use the **shared** pool (that's the one a
   two-person household's learned state would actually need) and note which pool you used; if only
   one undifferentiated store exists, note that too.
8. Record the literal call made — the method/API name and the argument shape, as observed. There
   is no authoritative documentation for this call reachable this session, so your own observation
   **is** the documentation for the write-up.

## Step 2 — Read from Device B

9. On Device B (the second phone), open the **same published artifact URL**, signed into the
   **same shared account**, in a **fresh session** (not a synced/resumed tab — you want a genuinely
   independent load).
10. Attempt to read the same key back. Record:
    - Did it return the value at all?
    - Did it return immediately, or after a delay — and if delayed, roughly how long?
    - Anything unexpected (stale value, error, empty result, needed a manual refresh).

## Step 3 — Record the go/no-go

11. Write the dated result into `project-knowledge/writable-state-decision.md` (the scaffold
    authored alongside this protocol) — do not leave it only in this file. Pick exactly one branch:
    - **GO** — Step 0 found a storage API, Step 1's write succeeded, and Step 2's read-back on the
      second device actually returned the value. State which pool was used.
    - **NO-GO** — state precisely which step failed: capability doesn't exist (Step 0), write
      didn't persist (Step 1), or write persisted but wasn't visible on Device B, or was only
      visible after an unacceptable delay (Step 2).
    - **INCOMPLETE** — only one physical device was available, or the spike was interrupted before
      Step 2 could run. State exactly what's missing and that this defaults to the NO-GO fallback
      until re-run with both devices.

Do not assert GO without a recorded cross-device read-back actually observed on Device B. A
same-device write/read, however successful, is INCOMPLETE, not GO.

---

## What to paste back

Paste the raw observations from Step 0 (capability check result), and — if Step 0 found a storage
API — Step 1 and Step 2's raw observations (the literal call made, and the cross-device read-back
result including any delay). If the spike stopped at Step 0 with NO-GO, that alone is a complete,
valid result — paste that and stop.

If only one device was available, say so explicitly rather than letting it pass as a full result.
