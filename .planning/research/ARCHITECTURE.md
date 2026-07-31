# Architecture Research

**Domain:** Phone-first household grocery assistant — Claude Skills over Rohlík MCP, single shared Claude Project, single shared retailer basket, two-user household
**Researched:** 2026-07-31
**Confidence:** MEDIUM — the Claude Skills/Project platform constraints (read-only Project knowledge, no cross-skill code sharing, artifact storage scope) are well-established from the platform brief; the *specific* mechanics of artifact persistent storage (does a write from device A show up on device B under the same account?) are unverified and explicitly flagged as a spike, not a researched fact.

## Standard Architecture

### System Overview

```
┌───────────────────────────────────────────────────────────────────────┐
│                    PROJECT KNOWLEDGE (read-only, hand-edited)          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌───────────────┐ │
│  │  household-  │ │   recipe-    │ │  budget.md   │ │ seed-         │ │
│  │  ruleset.md  │ │  sources.md  │ │ (soft/hard)  │ │ favourites.md │ │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └───────┬───────┘ │
└─────────┼────────────────┼────────────────┼─────────────────┼─────────┘
          │  (read by every skill, via the shared internals — never    │
          │   copied into a skill folder)                              │
┌─────────▼────────────────▼────────────────▼─────────────────▼─────────┐
│                SHARED INTERNALS (single source of truth doc set,      │
│                referenced, not reimplemented, by every skill)         │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐             │
│  │  resolution-   │ │ confirmation-  │ │ substitution-  │             │
│  │  cascade.md    │ │ protocol.md    │ │ policy.md      │             │
│  └───────┬────────┘ └───────┬────────┘ └───────┬────────┘             │
│  ┌────────────────┐ ┌────────────────┐                                │
│  │  mcp-          │ │  audit-        │                                │
│  │  degradation.md│ │  format.md     │                                │
│  └───────┬────────┘ └───────┬────────┘                                │
└──────────┼──────────────────┼──────────────────┼──────────────────────┘
           │  (invoked by, not duplicated in)     │
┌──────────▼──────────────────▼──────────────────▼──────────────────────┐
│            SKILLS (thin orchestrators — sequence + copy only)          │
│ ┌──────────┐┌──────────────┐┌───────────────┐┌───────────┐┌──────────┐│
│ │quick-add ││staples-      ││recipe-to-     ││meal-plan  ││household-││
│ │          ││restock       ││basket         ││           ││prefs     ││
│ └────┬─────┘└──────┬───────┘└───────┬───────┘└─────┬─────┘└────┬─────┘│
│ ┌──────────┐                                                          │
│ │basket-   │                                                          │
│ │review    │                                                          │
│ └────┬─────┘                                                          │
└──────┼───────────────────────────────────────────────────────────────┘
       │  (calls, via MCP tool use)          (reads/writes)
┌──────▼───────────────────────────┐  ┌───────────────────────────────┐
│   EXTERNAL SOURCE OF TRUTH        │  │   WRITABLE STATE               │
│   Rohlík basket (live cart)       │  │   1. Rohlík favourites/        │
│   via Rohlík MCP                  │  │      order-history (preferred) │
│   — read before write, idempotent │  │   2. Artifact persistent        │
│   — cart add/remove/update        │  │      storage, single-account    │
│   — search, deals, recs           │  │      scope (fallback — spike     │
│   — NOT: checkout/payment         │  │      needed for cross-device     │
└───────────────────────────────────┘  │      write + read)              │
                                        └───────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| Project knowledge | Static, human-curated facts: what "good" looks like, what's off-limits, what's trusted, what money limits apply | Markdown files in Claude Project knowledge (`household-ruleset.md`, `recipe-sources.md`, `budget.md`, `seed-favourites.md`); edited by the humans outside chat, never by the agent |
| Shared internals — resolution cascade | The one algorithm every skill uses to turn a vague item into a concrete SKU: check ruleset → check favourites/history → ask user | One reference doc (`resolution-cascade.md`) in Project knowledge; every skill's SKILL.md says "follow resolution-cascade.md," never re-describes the steps |
| Shared internals — confirmation protocol | The one interaction pattern for presenting a proposed cart change and collecting a yes/no/edit before any MCP write | One reference doc (`confirmation-protocol.md`); defines the artifact shape, the mobile-safe format, the verbal read-back requirement |
| Shared internals — substitution policy | Rule for when a substitution may be proposed vs silently applied (never silent) and how it's surfaced in the confirmation | One reference doc (`substitution-policy.md`) |
| Shared internals — MCP-degradation policy | What every skill does when Rohlík MCP is slow, erroring, or gone: fall back to a plain list, state degraded mode explicitly, never claim a fake success | One reference doc (`mcp-degradation.md`) |
| Shared internals — audit format | Common shape for "what was added, quantity, cost, which rule/favourite matched" so every skill's output is consistent and scannable on a phone | One reference doc (`audit-format.md`) |
| Skills | Orchestrate: parse the trigger phrase, gather the item list, invoke the resolution cascade, build the confirmation artifact, call MCP once approved, emit the audit line | Six `SKILL.md` files, each is a short script that *references* the shared internals docs by name and adds only what's unique to that skill (e.g., staples-restock's cadence check, recipe-to-basket's source-trust check) |
| Rohlík basket (external SoT) | The one place "what's actually going to be delivered" lives; every skill must read it before deciding what to add, and every add must be idempotent against its current contents | Rohlík MCP: `cart_read`, `cart_add`, `cart_update`, `cart_remove`, `product_search`, `order_history` |
| Writable state | Anything that must persist and *change over time* but isn't itself the basket: approved favourites, staples cadence, recipe repertoire | Primary: Rohlík favourites/order-history (already durable, already shared, zero extra infra). Secondary/fallback: artifact persistent storage scoped to the single shared Claude account — holds only what Rohlík's data model can't (e.g. staples cadence, curated repertoire notes) |

## Recommended Project Structure

This is not a codebase in the software-repo sense — it's a Claude Project's knowledge base plus a set of Skill folders. The "project structure" is the knowledge/skills layout:

```
Claude Project Knowledge (read-only at chat time)
├── household-ruleset.md      # quality rules, dislikes, allergies, brand prefs
├── recipe-sources.md         # trusted sites + uploaded cookbook references
├── budget.md                 # soft threshold, hard cap
├── seed-favourites.md        # starting favourites (bootstrap only — Rohlík favourites take over after)
├── resolution-cascade.md     # SHARED INTERNAL — the one resolution algorithm
├── confirmation-protocol.md  # SHARED INTERNAL — the one confirmation artifact contract
├── substitution-policy.md    # SHARED INTERNAL — substitution rules
├── mcp-degradation.md        # SHARED INTERNAL — fallback behavior contract
└── audit-format.md           # SHARED INTERNAL — output/audit line shape

Skills (each its own folder, each SKILL.md thin)
├── quick-add/
│   └── SKILL.md              # references cascade + confirmation + degradation + audit docs
├── staples-restock/
│   └── SKILL.md              # + cadence logic unique to this skill
├── recipe-to-basket/
│   └── SKILL.md              # + recipe-source lookup, ingredient extraction
├── meal-plan/
│   └── SKILL.md              # + multi-day/multi-recipe aggregation before cascade
├── basket-review/
│   └── SKILL.md              # read-only skill: cart_read + budget.md, no cascade needed
└── household-prefs/
    └── SKILL.md              # the ONE skill allowed to propose ruleset edits (human applies them)
```

### Structure Rationale

- **Knowledge docs are flat, not nested:** Claude Project knowledge has no real folder hierarchy that skills can traverse reliably — flat, well-named `.md` files that a SKILL.md can reference by name (or that get pulled into context automatically as project knowledge) are the reliable pattern.
- **Shared internals live in Project knowledge, not in any skill folder:** this is the only mechanism Claude's Skills model offers for "shared library code" across independently-loaded skills (see Pattern 1 below). Skills are isolated folders with no import/require mechanism between them.
- **household-prefs is structurally special:** it's the only skill that touches the read-only boundary. It cannot write to Project knowledge directly (agent can't write project files), so its job is to *propose* a diff and hand it to a human to paste into the file manually, or to write the "learned" portion to writable state instead. That boundary should be explicit in its SKILL.md.

## Architectural Patterns

### Pattern 1: Shared reference doc as a pseudo-library

**What:** Because Skills are independent folders with no cross-skill code sharing, the "single source of truth" requirement is met by putting the cascade/confirmation/substitution/degradation logic in Project-level knowledge files, and having every SKILL.md open with a line like: "Before resolving any item, follow `resolution-cascade.md` in Project knowledge. Do not re-derive these steps here." This makes the knowledge doc the de facto shared module; the skill is the caller.

**When to use:** Any behavior that must be identical across ≥2 skills (cascade, confirmation, degradation, audit format, substitution policy — all five, per this brief).

**Trade-offs:** Pro — single edit point, guaranteed consistency, matches the brief's mandate exactly. Con — nothing enforces that a skill actually *follows* the referenced doc; correctness depends on the SKILL.md prompt wording and the agent's instruction-following, not on a compiler/import. Mitigate by keeping the shared docs short, imperative, and by testing each skill against the same cascade scenarios during build.

**Example (SKILL.md excerpt, conceptual):**
```markdown
## Resolution
For every item to resolve, follow `resolution-cascade.md` exactly:
ruleset → favourites/history → ask. Do not invent alternate resolution logic here.

## Confirmation
Before any MCP write, build the confirmation artifact per `confirmation-protocol.md`.
```

### Pattern 2: Read-modify-confirm-write, never write-on-first-pass

**What:** Every skill that mutates the basket follows the same four-step shape: (1) MCP `cart_read` to get current state, (2) resolve target items via the cascade, (3) render a confirmation artifact and collect approval, (4) MCP write, followed by an audit line. No skill is allowed to call a cart-mutating MCP tool before step 3 completes.

**When to use:** Any skill that changes the shared basket (quick-add, staples-restock, recipe-to-basket, meal-plan). Not needed for basket-review (read-only) or household-prefs (writes to preference state, not the cart).

**Trade-offs:** Pro — enforces idempotency (read-before-write catches "already in cart"), enforces the "confirmation before mutation" safety requirement, keeps behavior uniform across skills so a user's mental model transfers between them. Con — adds a guaranteed round trip (turn 1: propose, turn 2: confirm) to every mutating flow, which competes with the ≤3-turn budget — this is why the confirmation artifact must be single-shot and bundle *all* items in one card rather than one confirmation per item.

### Pattern 3: Tiered writable state (retailer-native first, artifact storage as gap-filler)

**What:** Don't build a separate data store for anything Rohlík's own object model already covers. Favourites and order history are native Rohlík account data — durable, already shared across the one shared Rohlík account, already exposed via MCP. Only fall to artifact persistent storage for state Rohlík has no field for: staples cadence (how often, not just what), curated repertoire annotations, and any "approved over time" curation metadata that isn't itself a product.

**When to use:** Whenever a skill needs to remember something between sessions.

**Trade-offs:** Pro — minimizes new infrastructure, maximizes reuse of a store that's already proven shared and durable. Con — artifact persistent storage's exact semantics (per-account? per-conversation? synced across the two devices sharing the one Claude account?) are unverified; this is called out in PROJECT.md as a pending spike, and the architecture must not assume success — degrade to "ask the human to note it down" if the spike fails.

## Data Flow

### Request Flow (quick-add, the reference spine every other skill reuses)

```
"add milk and bananas" (vague, spoken/typed on phone)
    ↓
Skill trigger match → quick-add/SKILL.md loads
    ↓
Resolution cascade (per resolution-cascade.md):
  read household-ruleset.md (quality rules, dislikes, allergies, brand prefs)
  read seed-favourites.md / Rohlík favourites+order-history (via MCP)
  → if both silent on an item: ask user (bounded, batched question — not per-item)
    ↓
MCP cart_read (current basket state) — required before any write, for idempotency
    ↓
Substitution check (per substitution-policy.md) — flag, never silently swap
    ↓
Budget check against budget.md (soft threshold / hard cap) — running total
    ↓
Confirmation artifact built (per confirmation-protocol.md) — response collector,
  does NOT call MCP itself
    ↓
Verbal read-back + user confirms/edits (turn 2 of the ≤3-turn budget)
    ↓
MCP cart_add/cart_update (idempotent against the cart_read snapshot)
    ↓
Audit report emitted (per audit-format.md): what, qty, cost, which rule/favourite matched
    ↓
(optional) writable-state update: e.g. new favourite candidate noted for household-prefs
```

### Store Read/Write Matrix

| Store | Read by | Written by | Notes |
|-------|---------|------------|-------|
| household-ruleset.md | Every mutating skill (via cascade) | Human only (hand-edited); household-prefs skill may *propose* edits but cannot commit them | Read-only to the agent — this is a hard platform constraint, not a design choice |
| recipe-sources.md | recipe-to-basket, meal-plan | Human only | Same constraint |
| budget.md | Every mutating skill, basket-review | Human only | Same constraint |
| seed-favourites.md | Resolution cascade (bootstrap tier, before Rohlík favourites accumulate) | Human only (initial seed); becomes decreasingly relevant as Rohlík favourites/order-history accrue real data | Same constraint; treat as bootstrap, not steady-state |
| Rohlík basket (live cart) | Every skill, at start (cart_read) | Every mutating skill, after confirmation (cart_add/update/remove) | External source of truth; never trust a local memory of "what's in the cart" — always re-read |
| Rohlík favourites/order-history | Resolution cascade (steady state) | Passively, by the platform, as orders accrue; not directly written by skills in v1 | Preferred writable-state location — durable, already shared, no spike needed |
| Artifact persistent storage (single-account scope) | Resolution cascade (for cadence, repertoire), staples-restock | household-prefs (curated favourites, staples cadence, recipe repertoire notes) | Needs the cross-device-shared-write spike before being load-bearing; design must degrade gracefully if unavailable |
| Confirmation artifact | User (visually/verbally), the invoking skill | The invoking skill, per-turn (ephemeral, not persisted state) | Response collector only — must never itself call MCP; that's what keeps "confirm" and "mutate" as two distinct, auditable steps |

### Key Data Flows

1. **Vague-request-to-basket-write (the spine):** user phrase → skill trigger → resolution cascade (reads ruleset + favourites) → cart_read → substitution/budget checks → confirmation artifact (no MCP call) → user approval → cart_add/update (idempotent) → audit report. This is the *only* flow that touches the live basket, and every mutating skill is this flow with a different front end (staples-restock adds a cadence check before the cascade; recipe-to-basket adds ingredient extraction from recipe-sources.md; meal-plan adds multi-recipe aggregation).
2. **Preference learning (household-prefs):** user states a durable preference ("we don't like brand X") → household-prefs skill drafts the change → since Project knowledge is read-only to the agent, the skill either (a) surfaces the diff for a human to paste into household-ruleset.md by hand, or (b) writes an interim note to artifact persistent storage that the cascade also consults, pending the human formalizing it in the ruleset file. This is the one place the "read-only Project knowledge" constraint directly shapes the interaction design.
3. **Degraded-MCP flow:** any skill, at cart_read or cart_add time, if the Rohlík MCP call fails/times out/is unavailable → mcp-degradation.md dictates falling back to presenting a plain shopping list (no fabricated "added!" confirmation) and telling the user explicitly the write did not happen — never silently retry-and-hope or fake success.
4. **Read-only review flow (basket-review):** cart_read + budget.md only; no cascade, no confirmation artifact, no MCP write. Structurally the simplest skill and a natural second build target after quick-add, since it exercises the MCP read path and audit-format doc without touching the mutate path at all.

## Scaling Considerations

This system has a hard, fixed scale — two users, one basket, one Claude account — so classic scaling tables don't apply. Reframe "scale" as *skill count and interaction complexity* instead of user count:

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 1 skill (quick-add only) | Shared internals can be sketched thin — even a single skill benefits from splitting cascade/confirmation into separate docs, because meal-plan and recipe-to-basket are already known to be coming. Don't inline the cascade into quick-add's SKILL.md even for the MVP. |
| 2-6 skills (full roadmap) | Shared internals docs become load-bearing — this is the point of the whole design. Any drift (a skill re-describing the cascade instead of referencing it) shows up as inconsistent behavior between skills and should be treated as a defect. |
| Beyond 6 skills / multi-retailer (explicitly out of scope) | Not a near-term concern; noted only because the "external source of truth" abstraction (MCP-backed live cart, read-before-write) is the one part of this architecture that would need to become a real interface if a second retailer were ever added. Don't build that abstraction now — YAGNI — but keep MCP calls confined to the shared internals' degradation doc rather than scattered ad hoc in each skill, so a future swap is localized. |

### Scaling Priorities

1. **First bottleneck: turn economy, not compute.** The constraint that actually bites first is the ≤3-turn budget colliding with the mandatory read-confirm-write sequence. Fix: the confirmation artifact must batch all items into one card (one confirmation turn, not one per item), and the cascade must resolve silently whenever the ruleset/favourites already answer the question — only escalate to a question when genuinely ambiguous, and batch multiple ambiguities into a single question turn.
2. **Second bottleneck: shared-internals drift.** As skills are added, the risk isn't load, it's five skills each acquiring their own slightly-different copy of "how confirmation works." Fix: treat the shared internals docs as the thing code review / build QA actually checks skills against — before shipping any new skill, diff its behavior against the reference doc's described contract.

## Anti-Patterns

### Anti-Pattern 1: Reimplementing the cascade/confirmation per skill

**What people do:** Copy-paste (or re-derive from the requirement) the resolution logic and confirmation format into each new skill's SKILL.md because it's faster than referencing a shared doc, especially once a skill needs "just one more field" in the confirmation.
**Why it's wrong:** Directly violates the brief's single-source-of-truth mandate; guarantees the five skills drift out of sync over time (one skill checks budget before substitution, another after; one degrades gracefully, another doesn't) — exactly the failure mode this architecture exists to prevent.
**Do this instead:** Any time a skill needs "one more field," add it to the shared doc (resolution-cascade.md / confirmation-protocol.md) so every skill gets it, and reference the doc by name in the skill instead of inlining logic.

### Anti-Pattern 2: Treating the confirmation artifact as a proxy that can call MCP

**What people do:** For convenience, have the confirmation artifact itself trigger the cart write once the user taps "confirm," collapsing the confirm-then-write boundary into one step to save a turn.
**Why it's wrong:** Breaks the explicit safety design ("confirmation is a response-collector artifact that never calls the MCP") and removes the one clean audit point between "user agreed to X" and "X was actually written" — if the write partially fails, there's no clean state to reconcil against.
**Do this instead:** Confirmation artifact only collects a decision; the invoking skill (not the artifact) performs the MCP write immediately after, using the cart_read snapshot taken before confirmation to keep the write idempotent. This still costs only one extra logical step, not an extra user-facing turn, if implemented as "artifact returns → skill immediately writes → skill immediately reports," all within the same assistant turn.

### Anti-Pattern 3: Assuming artifact persistent storage "just works" across devices before the spike

**What people do:** Design staples-restock's cadence tracking, or household-prefs' curated-favourites list, assuming writable artifact state is trivially readable from both phones under the one shared Claude account, without verifying it.
**Why it's wrong:** PROJECT.md explicitly flags this as unverified ("Pending — artifact-storage spike"); if it turns out to be per-conversation or per-device rather than per-account, any skill that depends on it silently breaks for the second device/user, which is worse than never building the feature.
**Do this instead:** Build the spike before any skill depends on artifact storage for anything load-bearing; until verified, prefer Rohlík favourites/order-history (already proven shared) and treat artifact storage as strictly best-effort/supplementary, with every skill that reads it tolerating "not found" as a normal case, not an error.

### Anti-Pattern 4: Asking one question per ambiguous item

**What people do:** When the cascade can't resolve an item, ask about it immediately, one item at a time, as soon as it's hit — natural if you're processing a list sequentially.
**Why it's wrong:** Blows the ≤3-turn budget almost immediately on any multi-item request ("milk, bananas, that cheese we like, snacks for the kids" could trigger 2-3 separate questions if handled naively).
**Do this instead:** Resolve everything the cascade *can* resolve silently first, collect every remaining ambiguity, and ask a single batched question (or fold the ambiguity into the confirmation artifact itself as an inline choice) so the whole request costs one clarifying turn at most, not one per item.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Rohlík MCP (`https://mcp.rohlik.cz/mcp`) | OAuth custom connector; called only from within the shared internals' MCP-facing steps (cart_read, cart_add/update/remove, product_search, order_history, deals) | Experimental, personal-use-only, may change/disappear without notice per PROJECT.md — every call site must be wrapped by mcp-degradation.md behavior; checkout/payment intentionally not exposed, which is a platform-enforced safety boundary, not something the architecture needs to guard against separately |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Project knowledge ↔ Skills | One-directional read; skills reference docs by name, no write path back | Read-only is a hard platform constraint (agent cannot write Project files); household-prefs must route "learned" changes to writable state or a human-applied diff instead |
| Shared internals ↔ Skills | Skills invoke/reference, never fork/copy | The only mechanism available in Claude's Skills model for cross-skill shared logic, since skill folders don't import from each other |
| Skills ↔ Rohlík MCP | Skills call MCP tools directly, but only through the sequence mandated by the shared internals (read → resolve → confirm → write → audit) | Keeping this sequence identical across skills is what makes idempotency and degradation guarantees hold system-wide rather than skill-by-skill |
| Skills ↔ Writable state (favourites/history, artifact storage) | Skills read at cascade time, write after confirmed mutations or explicit household-prefs updates | Favourites/order-history preferred (proven shared, durable); artifact storage is the fallback for cadence/repertoire data Rohlík can't model, pending the cross-device spike |
| Confirmation artifact ↔ Skill | Artifact returns a decision to the invoking skill; skill (not artifact) performs the MCP write | See Anti-Pattern 2 — this boundary is the core safety mechanism of the whole design |

## Build Order (dependency-driven)

The dependency chain is strict: nothing above the spine is useful until the spine exists, and every skill after quick-add is a thin variation on the same spine.

1. **Shared internals must exist first, even in draft form:** `resolution-cascade.md`, `confirmation-protocol.md`, `substitution-policy.md`, `mcp-degradation.md`, `audit-format.md`. None of the six skills can be meaningfully built or tested before these exist, because every skill's SKILL.md is defined largely by *referencing* them.
2. **Project knowledge seeds must exist alongside:** `household-ruleset.md`, `budget.md`, `seed-favourites.md` (minimal viable content is fine — a handful of rules/allergies/thresholds — it can grow later; the cascade needs *something* to read on day one). `recipe-sources.md` can lag until recipe-to-basket/meal-plan are built.
3. **MCP integration layer (cart_read, cart_add/update, idempotency check) must be proven working** before any skill ships — this is really part of step 1's mcp-degradation.md and the cascade, not a separate skill.
4. **quick-add is the first vertical slice.** It is the minimum skill that exercises the entire spine end-to-end: trigger → cascade (ruleset + favourites) → cart_read → substitution/budget check → confirmation artifact → verbal read-back → cart_add → audit. Building it *is* the validation that the shared internals are real and sufficient — expect to revise the shared docs while building quick-add, not after.
5. **basket-review next (cheapest second build):** exercises the read path and audit-format doc with zero mutation risk, no confirmation flow needed — good smoke test that the shared internals hold up for a second consumer without touching the highest-risk code path (writes).
6. **staples-restock, recipe-to-basket, meal-plan follow, each adding exactly one new piece of logic on top of the proven spine:** staples-restock adds a cadence check (candidate for early artifact-storage use, once spiked); recipe-to-basket adds recipe-source ingestion (needs recipe-sources.md); meal-plan adds multi-recipe aggregation before invoking the same cascade. None of these three should need to touch confirmation-protocol.md, mcp-degradation.md, or audit-format.md at all — if they do, that's a signal the shared doc was incomplete when quick-add shipped, not that the new skill needs its own logic.
7. **household-prefs last,** since it's the one skill that operates on the read-only boundary itself (proposing ruleset changes / writing to fallback writable state) rather than on the basket-write spine — it depends on the other skills already existing to have generated real preference signal (declined substitutions, repeated ad-hoc items) worth promoting.
8. **Artifact-persistent-storage spike should run in parallel with step 4-5,** not block them — quick-add and basket-review don't need it (favourites/order-history and Project knowledge cover their needs); staples-restock (step 6) is the first consumer that actually needs the spike's answer.

## Sources

- `/home/user/grocery-shopper/.planning/PROJECT.md` — project brief: requirements, constraints, key decisions, platform facts about Claude Projects (read-only knowledge) and Rohlík MCP (experimental, no checkout/payment exposure)
- Reasoning derived from the stated platform constraints (Claude Skills = isolated folders with no cross-skill import mechanism; Project knowledge read-only to the agent; single shared Claude account across two devices) rather than external documentation — no external sources were fetched for this research pass

---
*Architecture research for: phone-first household grocery assistant (Claude Skills + Rohlík MCP)*
*Researched: 2026-07-31*
