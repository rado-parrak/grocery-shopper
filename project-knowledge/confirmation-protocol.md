# Confirmation Protocol

**Implements:** FOUND-02 · D-06, D-07

**Status:** Shared internal — the single source of truth for the contract between any
basket-mutating skill and the confirmation artifact it shows the user before writing anything.
Every mutating skill MUST follow this contract exactly and MUST NOT re-derive it inside its own
SKILL.md.

## The contract, in one sentence

The confirmation artifact is a **pure response-collector**: it shows the proposed batch and
collects a decision, but it never itself writes to the basket, never calls a connector, and never
has any channel back into the conversation other than the human's own next chat message.

## What the artifact does

- Renders **one card for the whole batch** (never one confirmation per item — this protects the
  ≤3-turn budget for multi-item requests like a recipe or a weekly plan).
- For each item: a checkbox (include/exclude) and a quantity stepper (minus / number / plus).
- A running total line, and the soft/hard budget lines from `budget.md` (warn if projected total
  crosses the soft threshold; state plainly if it is at or over the hard cap).
- A compact, copyable plain-text "final list" summary reflecting the artifact's current
  tick/quantity state, so the user can paste it elsewhere if they want to, independent of the
  in-chat confirmation path.
- Single-column, mobile-first layout: large tap targets, no horizontal scroll, no wide tables
  (CLAUDE.md Architecture Pattern 4).

## What the artifact declares — and does not

**The artifact declares NO runtime capabilities.** Not `mcp`, not `downloads`, not `storage`, not
any future capability. This is a **permanent** rule, not a placeholder to revisit once the
underlying MCP server matures — see Rationale below. Any future revision of the confirmation
artifact that proposes adding a capability must treat that as a Rule-4-style architectural change
requiring explicit human sign-off, not a routine enhancement.

Concretely, the artifact must never:
- Call `Rohlík:cart_add`, `Rohlík:cart_read`, or any other MCP tool, directly or indirectly.
- Make any network request (fetch, XHR, WebSocket, or otherwise).
- Attempt to declare or use a `storage` capability to persist or transmit the user's decision.

## How the decision actually gets back into the conversation

There is **no send-back mechanism**. The artifact has no API to submit structured data into the
chat. The **only** channel is the user's own next natural-language message — typed or dictated —
for example: "yes add all," "skip the milk," "2 rohlíky not 4."

The invoking skill:
1. Took a `cart_read` snapshot **before** rendering the artifact (this is the baseline every later
   MCP write must be computed against — see `mcp-degradation.md`).
2. Rendered the artifact showing its own last proposal (already in the skill's own context — no
   external round-trip needed to know what was shown).
3. Parses the user's reply **against that last-rendered proposal and the pre-artifact cart_read
   snapshot** — not against any payload the artifact "returns," because no such payload exists for
   a non-published, capability-free, in-conversation artifact.

The expected common-case path is a short confirming reply (protects the ≤3-turn budget); the
copyable plain-text summary exists as a secondary, user-initiated escape hatch, not the primary
return channel.

## Rationale: why not an mcp-capable artifact (D-07)

Recorded here permanently so no future revision "helpfully" re-adds a capability without
re-litigating this reasoning:

1. **Single auditable boundary.** Keeping the artifact capability-free preserves one clean,
   auditable line between "the household agreed to X" (the confirming chat message) and "X was
   actually written" (the skill's own MCP call, logged via `audit-format.md`). If the artifact
   could write directly, that boundary disappears and there is no clean reconciliation point if a
   write partially fails.
2. **No duplicated logic in a harder-to-edit layer.** The read-before-write / idempotency / audit
   discipline in `mcp-degradation.md` and `audit-format.md` lives in editable markdown a human can
   revise directly. An mcp-capable artifact would need that same discipline duplicated into
   compiled/published artifact JavaScript — a second copy that drifts from the first exactly as
   this project's architecture is built to prevent (see `resolution-cascade.md` "Why this
   exists").
3. **Fragile dependency behind the cheaper-to-fix layer.** The Rohlík MCP is explicitly
   experimental and may change without notice. Keeping every actual tool call inside a skill's own
   instructions (fixed by re-uploading a SKILL.md) rather than inside published-artifact code
   (fixed only by republishing the artifact) keeps this project's one truly fragile external
   dependency behind the cheaper layer to repair.
4. **Scope mismatch.** An `mcp`-capability-declaring artifact page "cannot be shared publicly" per
   the current runtime contract — a sign this capability is scoped for viewer-driven
   dashboards/tools, not a safety-critical, audited mutation step like a shared household basket
   write.

## Cross-references

- `resolution-cascade.md` — produces the proposal this artifact renders.
- `mcp-degradation.md` — the pre-artifact `cart_read` snapshot and the post-confirmation
  read-before-write/read-back-after-write discipline the invoking skill follows once the user
  confirms.
- `budget.md` — the soft/hard threshold lines rendered on the card.
- `audit-format.md` — what gets reported back to the user once the skill actually writes the
  confirmed items.

## What NOT to do

- Do not declare `mcp`, `downloads`, or `storage` on the confirmation artifact, ever.
- Do not design any "submit" button that calls out to a connector or network endpoint.
- Do not assume a structured payload comes back from the artifact — only the user's plain next
  message does.
- Do not re-describe this contract inside any skill's own SKILL.md; reference this file by name.
