# Phase 1: Foundation & Shared Spine - Research

**Researched:** 2026-07-31
**Domain:** claude.ai platform mechanics (Skills, Projects, Artifacts) + official Rohlík MCP connector, for authoring five shared-internal markdown contracts, read-only household config, a live MCP capability-discovery round-trip, and an artifact-storage go/no-go spike
**Confidence:** MEDIUM-HIGH — the Skills/Project-Knowledge platform mechanics are confirmed directly against current official docs this session. The artifact-storage capability and the exact official Rohlík MCP tool surface are the two genuinely open items, and this research surfaces a **new, load-bearing finding** on the former (see Summary) that changes how the FOUND-09 spike must be scoped.

## Summary

Phase 1 has no application code — its deliverables are markdown documents (five shared-internal contracts, three read-only config files) plus two pieces of empirical proof that can only be produced by a human operating the real claude.ai product: a live Rohlík MCP round-trip, and an artifact-storage cross-device spike. This research confirms the current (2026) SKILL.md authoring spec directly against Anthropic's own docs, confirms Project Knowledge's read-only/size characteristics, and confirms — by directly reading this session's own live artifact-capabilities skill — that the in-conversation confirmation artifact has no documented channel to write structured data back into the conversation other than the user's own next chat message, exactly as CONTEXT.md's D-06 already assumes.

The most important new finding from this research pass: **this session's own live capability roster for published artifacts lists only two declarable runtime capabilities — `downloads` and `mcp` — no `storage` capability appears in it.** Every external secondary source describing a `window.claude.storage`/`window.storage` persistent-storage API (20MB/artifact, personal + shared pools) is a community blog, not an Anthropic doc reached this session (official support.claude.com pages 403'd to automated fetch, exactly as prior research found). This does not prove storage doesn't exist — it may be an always-on API for published artifacts that isn't part of the viewer-consent "capabilities" declaration mechanism `mcp`/`downloads` use — but it means the FOUND-09 spike cannot assume storage is reachable the way `mcp`/`downloads` are; **the first concrete step of the spike must be "does `window.claude.storage` (or `window.storage`) exist at all in a published artifact opened right now," not "write and read back a key."** This reframing is captured in the spike protocol below.

Similarly, this research found that the Rohlík MCP tool-name list circulating in search results (`search_products`, `add_to_cart`, `get_cart_content`, `get_frequent_items`, etc.) comes from a **different, unofficial, reverse-engineered, username/password-authenticated community project** (`github.com/tomaspavlin/rohlik-mcp`), not from the official OAuth-based `mcp.rohlik.cz` server this project actually uses. It is a reasonable naming *hypothesis* (same underlying retailer API is likely being wrapped) but must not be treated as a verified tool list — CONTEXT.md's D-09 empirical-discovery mandate is the only way to actually know the official server's tool names, and this research gives a concrete recording template for that discovery plus a specific thing to check for (`get_frequent_items`-equivalent, which would resolve STACK.md's open "does a favourites tool exist" question).

**Primary recommendation:** Author all five shared-internal docs and three config files exactly per CONTEXT.md's locked shape (this is agent-executable, no platform dependency); treat the MCP round-trip and artifact-storage spike as two separate **human-executed, checkpoint-gated** tasks with the concrete step sequences below, and record their results back into `mcp-degradation.md` and a dedicated go/no-go note respectively before Phase 2 planning begins.

## Architectural Responsibility Map

This project has no browser/server/API tiers in the conventional sense — everything runs inside claude.ai chat. The tiers below are this platform's actual analogs.

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Resolution cascade, substitution policy, audit format (docs) | Shared Internals (Project Knowledge, referenced-not-duplicated) | — | Single source of truth every skill must reference; no cross-skill import mechanism exists, so Project Knowledge is the only shared-library tier available |
| Confirmation protocol (doc) | Shared Internals | Confirmation Artifact (in-conversation, ephemeral, no declared capabilities) | The doc specifies the artifact's *contract*; the artifact itself is a separate, capability-free UI tier that must never gain MCP/storage capabilities |
| MCP-degradation policy (doc) | Shared Internals | Rohlík MCP (external service) | Governs how every skill talks to the external tier; must be grounded in the live round-trip's observed behavior, not assumption |
| household-ruleset.md, budget.md, seed-favourites.md | Project Knowledge (read-only config/data tier) | — | Human-owned, durable, rarely-changing facts; zero agent write path — a hard platform constraint |
| Live MCP round-trip (search → cart_read → reversible add/remove → cart_read) | Rohlík MCP (external service tier) | — (no Skill exists yet to own this in Phase 1) | First empirical contact with the real tool surface; must be human-executed inside a real claude.ai chat with the connector attached |
| Artifact-storage go/no-go spike | Writable State — Artifact Storage (published-artifact-scoped client tier) | Claude Project/account (single shared identity across two devices) | Determines whether this tier is usable at all before any Phase 4 skill is designed to depend on it |

## User Constraints

<user_constraints>

### Locked Decisions (from CONTEXT.md — do not reopen)

- **D-01 (household-ruleset.md shape):** Three ordered sections read top-down by the cascade: **(A) Hard constraints** (allergies, "never buy X") that filter candidates out; **(B) Preferences** (ranked, per category) that rank what remains; **(C) Notes.** Declarative, ordered, first-match-wins. Reversibility: costly.
- **D-02 (starter ruleset):** Ship illustrative preference rules (BIO/farm-sourced produce, free-range eggs, milk default, Czech-origin preference, smallest-adequate pack size) but mark allergies/dislikes/brand-preferences as `⚠ FILL BEFORE FIRST REAL SHOP` placeholders.
- **D-03 (hard constraints are absolute):** A candidate violating a hard constraint is never proposed, substituted, or surfaced; a favourite can never override one. Allergies default to "none recorded — unconfirmed", never "no allergies".
- **D-04 (budget currency/shape):** CZK. `budget.md` holds a soft threshold (warn) and hard cap (block), hand-editable. Starter placeholders: soft 2000 Kč, hard 3000 Kč, marked "adjust to your household".
- **D-05 (hard cap semantics):** Applies to the projected whole-basket total (current cart total + items about to be added), not a single add. Read cart → compute projected total → refuse if over hard cap, ask to cut items or raise the cap. Reversibility: reversible.
- **D-06 (confirmation hand-back):** Confirmation artifact stays a pure response-collector (tickboxes + quantity steppers, one card for the batch, no MCP). User's decision returns as their **next natural-language chat turn** ("yes add all", "skip the milk", "2 rohlíky not 4"), parsed against the pre-artifact `cart_read` snapshot. Artifact also renders a compact plain-text "final list" the user can copy. Reversibility: costly.
- **D-07 (rationale against mcp-capable artifact):** Preserves the auditable "agreed → written" boundary; keeps read-before-write/idempotency/audit logic in editable markdown, not compiled artifact JS; keeps the fragile experimental-MCP dependency behind the cheaper-to-fix layer.
- **D-08 (artifact-storage spike acceptance):** On ONE shared Claude account, prove (a) a *published* artifact declaring the `storage` capability can WRITE a key from the household's normal chat-driven flow, and (b) that value READS BACK on the SECOND device signed into the same account. Go → artifact storage becomes home for learned state (Phase 4). No-go → fall back to Rohlík-native favourites/order-history + hand-edited Project files. Document as an explicit go/no-go note. Reversibility: one-way for dependents.
- **D-09 (MCP discovery is empirical, not doc-driven):** Official docs are inaccessible/403. Phase 1 connects the OAuth connector and does a live round-trip — product search → `cart_read` → single reversible probe `cart_add` then `cart_remove` on a throwaway item → `cart_read` again — recording actual tool names, parameters, and error shapes into `mcp-degradation.md` plus a short capability note. Every mutating flow specified as read-before-write + read-back-after-write regardless of server's own idempotency claims.
- **Audit format (FOUND-05, not separately selected):** One short mobile line per added item — `✓ {qty}× {Czech product name} — {line total} Kč · {why}` where `{why}` is the matched rule id or `oblíbené` (favourite) — followed by a running basket total. One line per item, no wide tables.

### Claude's Discretion

- File naming, exact section ordering within each contract doc, and whether the read-only docs live under a `project-knowledge/` folder vs flat — planner/executor's discretion, as long as skills reference them by stable name.
- Audit format is adjustable if the household dislikes the shape (not separately re-litigated, but not frozen either).

### Deferred Ideas (OUT OF SCOPE for this phase)

- **Learned-state write-back** (auto-updating favourites/staples cadence) — depends on the FOUND-09 spike outcome; belongs to Phase 4 (LEARN-01). Noted, not built here.
- **Bilingual/diacritic edge-case hardening** — real-device testing of code-switched Czech/English input belongs to Phase 2 UAT; Phase 1 only sets the "Czech output" convention.
- **Deals/promotions + personalised recommendations** feeding the cascade — v2 (LEARN-03/04).

</user_constraints>

## Phase Requirements

<phase_requirements>

| ID | Description | Research Support |
|----|-------------|------------------|
| FOUND-01 | Resolution-cascade doc is the one source of truth every skill references | Pattern 1 (shared-internals-as-pseudo-library) confirmed no cross-skill import exists; see Architecture Patterns §1 and Code Examples §1 |
| FOUND-02 | Confirmation-protocol doc defines the interactive artifact contract | Confirmed this session: no send-back API exists for a non-published, non-`mcp`-capability in-conversation artifact — the next chat message is the only channel. See Common Pitfalls §3 and Code Examples §2 |
| FOUND-03 | Substitution-policy doc governs out-of-stock handling | See Architecture Patterns §2 (read-modify-confirm-write) and household-ruleset D-01 hard/soft split, which substitution must respect |
| FOUND-04 | MCP-degradation doc defines fallback for every Rohlík MCP call | See MCP Empirical Discovery Protocol; degrade-branch template in Code Examples §3 |
| FOUND-05 | Audit-format doc defines what every basket add reports | Locked format from CONTEXT.md D-05/audit-format; Czech-output convention research in Common Pitfalls §5 |
| FOUND-06 | household-ruleset.md read at start of every resolution | Concrete template in Code Examples §4, grounded in D-01/D-02/D-03 |
| FOUND-07 | budget.md loaded/applied on every confirmation | Concrete template in Code Examples §5, grounded in D-04/D-05 |
| FOUND-08 | seed-favourites.md hand-editable pre-approved product IDs | Template in Code Examples §6; bootstrap-vs-steady-state distinction from ARCHITECTURE.md Pattern 3 |
| FOUND-09 | Artifact-storage spike go/no-go, gating Phase 4 | **Primary new finding this session** — see Summary + dedicated protocol in "Artifact-Storage Spike Protocol" section |

</phase_requirements>

## Project Constraints (from CLAUDE.md)

- Order submission is never exposed via MCP — no defensive code needed to block checkout, but every *other* mutating call needs read-before-write/read-back-after-write discipline.
- Rohlík MCP is experimental/personal-use; every skill (and, in this phase, the MCP-degradation doc) must specify a degrade branch, never silent failure or hallucinated success.
- Project Knowledge files are read-only to the agent — the five shared-internal docs and three config files are authored by the human/agent pair *outside* live chat sessions (i.e., in this repo, then uploaded), never written by a skill at chat time.
- OAuth only — no credentials in Project files, SKILL.md, or debugging transcripts. This phase's MCP round-trip must be scrubbed of any token-shaped string before being recorded into `mcp-degradation.md`.
- Mobile UX: no wide tables, short replies, one-handed confirmation — applies to the *content* of confirmation-protocol.md and audit-format.md even though this phase doesn't build the artifact itself.
- Every SKILL.md (future phases) must reference these shared docs by stable name and never re-derive their logic — this phase's naming choices for the five docs are load-bearing for every later phase.

## Standard Stack

Not applicable in the conventional sense — no libraries are installed by this phase. The "stack" is the claude.ai product surface itself:

| Component | Version/Status (2026) | Purpose | Confidence |
|-----------|----------------------|---------|------------|
| Claude Skills (SKILL.md + YAML frontmatter) | Current spec: `name` + `description` required; `name` ≤64 chars, lowercase/numbers/hyphens only, no XML tags, no reserved words "anthropic"/"claude"; `description` ≤1024 chars, non-empty, no XML tags | Not directly authored this phase (Phase 2+), but shared docs must be shaped for skills to reference | HIGH — `[CITED: platform.claude.com/docs/en/agents-and-tools/agent-skills/overview]`, `[CITED: .../best-practices]`, fetched directly this session |
| Claude Projects — Knowledge files | 30MB per file, unlimited files; formats PDF/DOCX/CSV/TXT/HTML/ODT/RTF/EPUB; 200K-token context window is a soft cap beyond which Claude switches to RAG-style retrieval | Hosts the five shared-internal docs + three config files | MEDIUM — `[CITED: web search synthesis of fast.io, datastudios.org, GitHub issue #46655]`; not independently confirmed against an official Anthropic doc this session (support.claude.com 403'd) |
| Claude Artifacts — capabilities (`downloads`, `mcp`) | Runtime contract 0.1.15 (this session) | Confirmation artifact deliberately declares **neither** — stays capability-free per D-06/D-07 | HIGH — `[VERIFIED: artifact-capabilities skill, read directly this session]` |
| Claude Artifacts — `storage` (unconfirmed) | Described in community sources as added ~Oct 2025, 20MB/artifact, personal+shared pools | Candidate mechanism for FOUND-09; **not listed in this session's live capability roster** | LOW — `[ASSUMED]`, contradicted by this session's own authoritative capability list; must be empirically re-checked at spike time, see below |
| Rohlík MCP (official) | `https://mcp.rohlik.cz/mcp`, OAuth custom connector, confirmed genuinely OAuth-based (not username/password) via Rohlík's own connector-setup description found in search | Sole integration surface for cart operations | MEDIUM — `[CITED: rohlik.cz product pages via web search]` for existence/OAuth; tool names/parameters remain LOW/`[ASSUMED]` until the live round-trip (official docs 403'd) |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Project Knowledge as shared-internals library | A dedicated internal-only Skill other skills "combine with" | Worth testing only if Project Knowledge context inclusion proves unreliable in practice (e.g. truncated against 6 skills' description metadata) — not indicated by anything found this session, so not recommended now |
| Rohlík-native favourites/order-history as primary writable state | A second, purpose-built MCP server / hosted key-value store | Only if BOTH the artifact-storage spike fails AND Rohlík's exposed tools don't cover needed metadata (restock cadence) — meaningfully more infrastructure than this project wants; last resort |

**Installation:** None — this phase produces markdown files only. No package manager, no ecosystem.

## Package Legitimacy Audit

**Not applicable.** This phase installs no external packages (npm, PyPI, or otherwise) — its entire deliverable is markdown documentation authored directly in this repo and later uploaded to claude.ai Project Knowledge / Settings. The Package Legitimacy Gate is skipped for this phase; re-run it in any future phase that introduces actual code dependencies (none are currently planned — this project has no application codebase, per PROJECT.md).

## Architecture Patterns

### System Architecture Diagram

```
 Human (outside chat) authors/edits, in this git repo:
 ┌────────────────────────────────────────────────────────────┐
 │  household-ruleset.md   budget.md   seed-favourites.md      │  ← Project Knowledge
 │  resolution-cascade.md  confirmation-protocol.md            │  ← (read-only to agent
 │  substitution-policy.md mcp-degradation.md audit-format.md  │  ←  once uploaded)
 └───────────────────────────┬──────────────────────────────────┘
                              │  human uploads flat files via
                              │  claude.ai Settings/Project UI
                              ▼
 ┌────────────────────────────────────────────────────────────┐
 │   Claude Project (shared, one household)                    │
 │   preloads Knowledge into every chat, regardless of which   │
 │   Skill (if any) triggers — this is the ambient-context     │
 │   mechanism that makes the docs "shared" without a skill    │
 │   import mechanism (none exists)                             │
 └───────────────────────────┬──────────────────────────────────┘
                              │  human opens a chat in the Project,
                              │  Rohlík MCP connector already attached (OAuth, once)
                              ▼
 ┌────────────────────────────────────────────────────────────┐
 │  MCP ROUND-TRIP (Phase 1's only live exercise of the spine) │
 │  search product → Rohlík:cart_read → Rohlík:cart_add        │
 │  (throwaway item, qty 1) → Rohlík:cart_remove (same item)   │
 │  → Rohlík:cart_read (confirm reverted)                       │
 │  every step's real tool name / params / error shape          │
 │  recorded → mcp-degradation.md                               │
 └───────────────────────────┬──────────────────────────────────┘
                              │  separately, in parallel:
                              ▼
 ┌────────────────────────────────────────────────────────────┐
 │  ARTIFACT-STORAGE SPIKE (human-executed, two physical phones)│
 │  Device A: publish artifact → check window.claude.storage    │
 │  exists → write test key → Device B: open same published URL │
 │  under the SAME shared Claude account → read key back        │
 │  → go/no-go recorded                                         │
 └────────────────────────────────────────────────────────────┘
```

### Recommended Project Structure

```
.planning-independent deliverable set (uploaded to claude.ai, not code):
project-knowledge/                    # or flat — Claude's discretion (D-10)
├── household-ruleset.md              # FOUND-06 — hard constraints / preferences / notes
├── budget.md                         # FOUND-07 — soft/hard CZK thresholds
├── seed-favourites.md                # FOUND-08 — bootstrap product IDs
├── resolution-cascade.md             # FOUND-01 — SHARED INTERNAL
├── confirmation-protocol.md          # FOUND-02 — SHARED INTERNAL
├── substitution-policy.md            # FOUND-03 — SHARED INTERNAL
├── mcp-degradation.md                # FOUND-04 — SHARED INTERNAL (+ round-trip findings)
└── audit-format.md                   # FOUND-05 — SHARED INTERNAL

artifact-storage-spike.md             # go/no-go record (FOUND-09), or a section in mcp-degradation.md's sibling
```

### Pattern 1: Shared reference doc as pseudo-library (no cross-skill import exists)

**What:** Because Claude Skills are isolated directories with no documented cross-skill filesystem or import mechanism `[CITED: platform.claude.com/docs/.../overview — Skills "exist as directories," no cross-skill reference documented]`, and because Project Knowledge is read by Claude as ambient context in every chat within a Project regardless of which Skill triggered, Project Knowledge is the only mechanism this platform offers for genuinely shared logic across skills.

**When to use:** Any behavior that must be byte-identical across ≥2 skills — exactly the five FOUND-01…05 docs.

**Example (conceptual SKILL.md excerpt a Phase-2+ skill will use, per the fully-qualified-tool-name convention confirmed this session):**
```markdown
## Resolution
Before resolving any item, follow resolution-cascade.md exactly:
hard constraints filter → preferences rank → favourites → ask.
Do not re-derive these steps here.

## MCP calls
Use the Rohlík:cart_read tool before computing any add.
Use the Rohlík:cart_add tool only after user confirmation.
```
Source: fully-qualified tool-name convention `[CITED: platform.claude.com/docs/.../best-practices — "MCP tool references" section]`.

### Pattern 2: Read-modify-confirm-write (the shape every mutating flow must follow)

**What:** cart_read → resolve via cascade → build confirmation artifact (no MCP call) → collect user's next-turn response → cart_add/update → cart_read (read-back) → audit line. Phase 1 does not build this as a skill, but the MCP-degradation doc and the confirmation-protocol doc must specify this shape precisely because Phase 2's quick-add is the first thing that will actually run it.

**When to use:** Any future basket-mutating skill. Not needed for the Phase 1 round-trip itself, which is a manual, human-supervised, single-pass version of the same shape (search → read → reversible add/remove → read).

### Anti-Patterns to Avoid

- **Re-describing cascade/confirmation logic inside a skill body "for speed":** guarantees drift the moment one copy gets a fix and the other four don't (already named in ARCHITECTURE.md Anti-Pattern 1 — this phase's job is to make the *one* copy right).
- **Declaring `mcp` or `storage` capability on the confirmation artifact:** the confirmation artifact built in Phase 2 must declare **no capabilities at all** — confirmed this session that `downloads` and `mcp` are the only declarable capabilities, and a page must explicitly opt in; omission is safe, not accidental exposure.
- **Treating the tomaspavlin/rohlik-mcp community tool list as the official tool list:** it's a different, unofficial, reverse-engineered project with different auth. Useful as a naming hypothesis to sanity-check discovered tool names against, never as a substitute for the live round-trip.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Cross-skill shared logic | A custom "include" convention inside skill zips | Project Knowledge docs, referenced by stable filename | No cross-skill import mechanism exists on this platform — Project Knowledge is the only documented shared-context surface |
| Confirmation artifact → chat data return | A custom postMessage/webhook bridge, or declaring `mcp` on the artifact "just to send one message" | The user's own next chat turn, parsed by the skill against the last-rendered proposal | No send-back API exists for a plain in-conversation artifact; every capability that *is* declarable (`mcp`, `downloads`) is scoped to published, viewer-consented pages, not a data-return channel |
| Cross-device shared "memory" | Assuming artifact storage or any bespoke mechanism "just works" before testing | Rohlík-native favourites/order-history as default; artifact storage only if the spike (this phase) proves it | The spike's own premise (a `storage` capability) is not currently visible in this session's authoritative capability roster — must be checked, not assumed |

**Key insight:** This phase's biggest risk isn't writing the docs (that's straightforward authoring against locked decisions) — it's over-trusting secondary-source claims about platform capabilities that could not be confirmed against an official source this session. Every capability claim not confirmed via direct doc fetch or this session's own live skill roster is tagged `[ASSUMED]` below and must be re-verified by the human at spike time.

## Common Pitfalls

### Pitfall 1: Assuming the artifact-storage capability exists as described in secondary sources
**What goes wrong:** A Phase 4 skill gets designed around `window.claude.storage` before anyone checks it's actually reachable in the current runtime, discovered only when staples-restock silently doesn't persist.
**Why it happens:** Multiple community blogs (caipi.ai, eigent.ai) describe the Oct-2025 storage feature confidently and in detail (20MB, personal/shared pools), which reads as authoritative even though none of them are Anthropic's own docs, and this session's own authoritative capability roster (read directly, not searched) lists only `downloads` and `mcp`.
**How to avoid:** Treat "does `window.claude.storage` (or `window.storage`) exist at all right now" as the literal first checkpoint of the FOUND-09 spike, before attempting any write/read test. See the dedicated protocol below.
**Warning signs:** Any Phase 4 planning that references a storage API without a completed, dated go/no-go note from this phase.

### Pitfall 2: Treating the community Rohlík MCP tool list as ground truth
**What goes wrong:** `mcp-degradation.md` gets written with tool names copied from `tomaspavlin/rohlik-mcp` (search_products, add_to_cart, ...) that turn out not to match the real, official, OAuth-based server's actual tool names.
**Why it happens:** It's the only detailed tool list search results surface, and it's tempting to treat "well, it's the same retailer's API" as good enough.
**How to avoid:** Use the community list only as a **naming hypothesis to check against**, not a source of truth; the recording template below requires writing down the tool name **as observed live**, with a column noting whether it matched the hypothesis.
**Warning signs:** `mcp-degradation.md` shipped without an "observed on [date], live round-trip" provenance line next to each tool name.

### Pitfall 3: Assuming the confirmation artifact has a structured return channel
**What goes wrong:** `confirmation-protocol.md` gets written assuming some `window.claude.complete()`-style or postMessage mechanism can hand structured JSON back into the conversation, when what actually exists (`window.claude.complete()`) is for calling a Claude *completion* from inside a published artifact — a different mechanism entirely, and irrelevant to a non-published, ephemeral confirmation card.
**Why it happens:** Search results surface `window.claude.complete()` in the same breath as "returning data to the conversation," which is easy to misread as the answer to "how does the artifact tell Claude what was confirmed."
**How to avoid:** `confirmation-protocol.md` must state explicitly: the artifact renders the proposal and a copyable plain-text summary; the **only** channel back into the conversation is the human's own next message, which the invoking skill parses against its own last-rendered proposal (already confirmed as the correct model in STACK.md and re-confirmed by this session's fresh search).
**Warning signs:** Any draft of confirmation-protocol.md that references an API call the artifact makes to "submit" the confirmation.

### Pitfall 4: Confusing "no capability declared" with "capability unavailable to check"
**What goes wrong:** Someone concludes storage is definitely unavailable forever, based on this session's roster, and skips checking again at spike time — when the roster could differ by account, by artifact type (published vs. draft), or by a subsequent platform update.
**Why it happens:** It's tempting to treat a single observation as final.
**How to avoid:** The spike must still be run empirically on the household's actual account/devices — this research flags the *risk* (don't assume storage works), not a final verdict (it may still be present and just gated differently than `mcp`/`downloads`).
**Warning signs:** Skipping the spike entirely based on this document alone.

### Pitfall 5: Product-name language drifting from Czech in the audit-format doc
**What goes wrong:** `audit-format.md`'s example lines get written in English during authoring (since this repo's authors may draft in English) and the "always Czech, verbatim from catalogue" rule never gets stated explicitly, so Phase 2 skills default to translating.
**Why it happens:** Same root cause as PITFALLS.md P17 — an agent's default is to mirror the conversation's language, which is right for prose but wrong for product names.
**How to avoid:** State the rule explicitly and give a mixed-language example in `audit-format.md` itself: "conversational text follows the user's language (CZ or EN); product names, brands, and units always render exactly as they appear in the Rohlík catalogue (Czech), never translated" — with a worked example using an English-triggered request producing a Czech audit line.
**Warning signs:** No English-input example anywhere in the doc.

## Rohlík MCP Empirical Discovery Protocol

**This step is human-executed** — it requires a real claude.ai chat, in the household's actual Project, with the Rohlík MCP OAuth connector already attached, and a live Rohlík account with real (if minimal) catalogue access. It cannot be run from this repository or any CI process.

### Probe sequence (exactly as locked in CONTEXT.md D-09)

1. **Product search** — issue a search for a common, cheap, easily-reverted item (e.g. a specific bread roll). Record: exact tool name invoked, the parameters Claude actually passed, and the shape of the returned result (product ID field name, price field name, unit/pack-size field name, stock/availability field name if present).
2. **`cart_read` (baseline)** — read the cart *before* any mutation. Record: exact tool name, parameter shape (none expected), returned shape (list of line items — what fields per item, is there a total field, currency format).
3. **Reversible `cart_add`** — add exactly 1 unit of the searched throwaway item. Record: exact tool name, parameters (product ID field name it expects — confirm it matches what `search` returned), the tool's own return value (and explicitly note whether it is trustworthy or not — per D-09, it must never be trusted at face value).
4. **`cart_read` (verify add)** — re-read the cart. Confirm the added item is actually present with the right quantity and price *before* believing the add succeeded. This is the empirical basis for `mcp-degradation.md`'s "never report success from the write call's own return value" rule.
5. **Reversible `cart_remove`** — remove the same throwaway item, restoring the cart to its pre-probe state. Record: exact tool name, parameters (does it take a cart-line ID, a product ID, or a quantity delta?).
6. **`cart_read` (verify revert)** — confirm the cart is back to baseline. This closes the loop with zero net change to the real household basket.
7. **Deliberately force at least one error case** — e.g., call `cart_add` with an invalid/expired product ID, or call a tool while intentionally providing a malformed parameter. Record the exact error shape (HTTP-like status? MCP protocol error? a plain string?). This is the only way to write a real (not hypothetical) branch into `mcp-degradation.md`.
8. **Check for a favourites-equivalent tool** — while enumerating available tools (Claude will typically list callable tools when asked, or they'll be visible via the connector's tool picker), specifically look for anything resembling `get_frequent_items` / a favourites surface. This resolves STACK.md's open MEDIUM-confidence question about whether the official server exposes such a tool, and if so, seed-favourites.md's bootstrap role can be scoped as explicitly temporary (per ARCHITECTURE.md Pattern 3).

### Recording template for `mcp-degradation.md`

For each of the tools actually observed, record a row like:

| Observed tool name | Matches community-list hypothesis? | Parameters (as observed) | Return shape (as observed) | Error shape (as observed, if forced) | Degrade behavior if this call fails |
|---|---|---|---|---|---|
| *(fill from step 1)* | yes/no/partial | ... | ... | ... | fall back to plain manual list, state degraded mode explicitly |

**Provenance requirement:** every row must be dated and marked "observed live, [date]" — never populated from the community tool-name hypothesis without the live check. Where the observed tool name differs from the hypothesis, note both (helps future debugging if the server's naming shifts again, given its explicit "may change without notice" status).

**Confidence:** LOW `[ASSUMED]` for the exact tool names/parameters until this protocol is run — this is expected and correct; the entire point of D-09 is that this cannot be verified from training data or 403'd docs, only from the live round-trip.

## Artifact-Storage Spike Protocol

**This step is human-executed**, requiring two physical devices signed into the same shared Claude account, and a claude.ai plan that supports publishing artifacts (Pro/Max/Team/Enterprise, per the Skills/Artifacts feature-gating already confirmed for Skills — the same account tier applies to the household's existing setup, so no new gate is expected, but confirm at spike time).

### Step 0 — Capability existence check (new step this research adds; not in the original ARCHITECTURE.md framing)

1. In the shared Project, ask Claude to create a small test artifact and attempt to declare a storage-oriented capability (or, if the artifact tool's capability picker only lists `downloads`/`mcp`, note that explicitly).
2. Publish the artifact. Open its rendered page and, if possible, inspect the browser's JS console for the existence of `window.claude.storage` or `window.storage` (either name has been reported in different secondary sources — check both).
3. **If neither exists:** record **NO-GO immediately** — skip to the fallback (Rohlík-native favourites/order-history + hand-edited Project files) without spending further spike time. This is a valid, fast outcome, not a failure of the spike.
4. **If one exists:** proceed to Step 1.

### Step 1 — Write from Device A

5. From Device A (whichever phone is used first), through the household's normal chat-driven flow (not a raw devtools console edit), have Claude's artifact write a test value to a key — e.g. `test_favourite_v1 = "milk-2percent-test"` — using whichever pool semantics are offered ("shared" pool if the API distinguishes personal/shared; if only one undifferentiated store exists, note that too).
6. Record the literal API call made (method name, argument shape) — this is currently undocumented by any authoritative source reached this session, so the household's own observation becomes the only documentation available for `mcp-degradation.md`'s sibling doc.

### Step 2 — Read from Device B

7. On Device B (the second phone), open the **same published artifact URL**, signed into the **same shared Claude account**, in a fresh session (not a synced/resumed tab).
8. Attempt to read the same key. Record: did the value read back correctly, immediately or after a delay, and exactly how long that delay was if any.

### Step 3 — Record the go/no-go

9. Write the result as a dated go/no-go note (e.g. `artifact-storage-spike.md`, or a section of `mcp-degradation.md`'s sibling doc — Claude's discretion on filename per D-10):
   - **GO:** artifact storage becomes the home for Phase 4 learned state (staples cadence, curated repertoire notes, rejected-substitution history). State which pool ("shared" vs "personal") was used and why.
   - **NO-GO:** state precisely which step failed (capability doesn't exist / write didn't persist / write persisted but wasn't visible on Device B / visible only after unacceptable delay), and confirm the fallback: Rohlík-native favourites/order-history as primary, hand-edited Project-file diffs proposed by `household-prefs` (Phase 4) as secondary.

**Confidence:** the protocol itself is well-grounded (mirrors D-08 exactly, with Step 0 added based on this session's finding); the *outcome* is LOW confidence / genuinely unknown until run — this is correct and expected, not a research gap to be filled without the human's hands-on test.

## Code Examples

### 1. Shared-internal reference pattern (resolution-cascade.md excerpt a future SKILL.md will point to)
```markdown
<!-- resolution-cascade.md -->
# Resolution Cascade

For every requested item:
1. **Hard constraints** (household-ruleset.md §A) — eliminate any candidate that
   violates an allergy or "never buy" entry. This step runs first, always,
   regardless of source (ruleset match, favourite, or free search).
2. **Preferences** (household-ruleset.md §B) — rank remaining candidates.
3. **Favourites** (seed-favourites.md, or Rohlík favourites/order-history once
   confirmed available) — if ruleset alone doesn't produce a confident match.
4. **Ask** — only if 1-3 all fail. Batch all unresolved items into one
   clarifying turn, never one question per item.
```
Source: derived directly from CONTEXT.md D-01/D-02/D-03 and ARCHITECTURE.md's Key Data Flows section — `[CITED: .planning/phases/01-foundation-shared-spine/01-CONTEXT.md]`.

### 2. Confirmation-protocol.md hand-back contract
```markdown
<!-- confirmation-protocol.md -->
# Confirmation Protocol

The confirmation artifact:
- Declares NO runtime capabilities (no `mcp`, no `downloads`, no `storage`).
- Renders one card for the whole batch: per-item checkbox + quantity stepper,
  running total, soft/hard budget line.
- Also renders a compact plain-text summary the user can copy.
- Has NO mechanism to send data back into the conversation. The user's own
  next message ("yes add all" / "skip the milk" / "2 rohlíky not 4") IS the
  confirmation signal. The invoking skill parses that reply against the
  artifact's last-rendered proposal (already in its own context) — never
  against a structured payload the artifact "returns," because no such
  channel exists for a non-published, capability-free artifact.
```
Source: `[VERIFIED: artifact-capabilities skill, read directly this session]` for the capability list; `[CITED: web search synthesis — no documented artifact→conversation write-back API found]` for the negative claim.

### 3. mcp-degradation.md degrade-branch template
```markdown
<!-- mcp-degradation.md -->
# MCP Degradation Policy

Every Rohlík MCP call site follows this shape:

try:
  result = call Rohlík:<tool_name>(...)
except (timeout, error, unexpected shape):
  degrade: render a plain manual shopping list from whatever was already
  resolved; state explicitly "Rohlík connection isn't working right now —
  here's your list to add by hand"; never retry silently; never claim success.

Never phrase a success message from a write call's own return value alone.
Every mutating flow re-reads (`cart_read`) immediately before AND immediately
after the write, and success is reported only if the read-back shows the
expected state.

## Observed tool surface (filled in during the live round-trip — see
## Rohlík MCP Empirical Discovery Protocol in 01-RESEARCH.md)
[table populated post-spike]
```
Source: derived from CONTEXT.md D-09 + PITFALLS.md P1-P3 `[CITED: .planning/research/PITFALLS.md]`.

### 4. household-ruleset.md starter template
```markdown
<!-- household-ruleset.md -->
# Household Ruleset

## A. Hard Constraints (absolute — filter candidates out, never overridden)
- Allergies: ⚠ FILL BEFORE FIRST REAL SHOP — none recorded — unconfirmed
- Dislikes / never-buy: ⚠ FILL BEFORE FIRST REAL SHOP

## B. Preferences (ranked, per category — rank what remains after A)
- Vegetables & fruit: prefer BIO; else farm-sourced (farmářské); else cheapest acceptable
- Eggs: free-range / z podestýlky or better
- Milk: ⚠ FILL BEFORE FIRST REAL SHOP (brand); default plnotučné unless noted
- General: prefer Czech origin where sensible
- Pack size: prefer smallest pack meeting requested quantity (anti-waste)

## C. Notes
(free-form household context, e.g. seasonal notes, delivery-day preferences)
```
Source: verbatim from CONTEXT.md `<specifics>` — `[CITED: .planning/phases/01-foundation-shared-spine/01-CONTEXT.md]`.

### 5. budget.md starter template
```markdown
<!-- budget.md -->
# Budget

Currency: CZK
Soft threshold (warn): 2000 Kč — adjust to your household
Hard cap (block): 3000 Kč — adjust to your household

Hard cap applies to the PROJECTED WHOLE-BASKET TOTAL (current cart + items
about to be added), not a single add. Before any write: read cart, compute
projected total, refuse if over hard cap, ask user to cut items or raise cap.
```
Source: verbatim from CONTEXT.md D-04/D-05.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|---------------|--------|
| Assuming skill descriptions can be soft/vague ("I help with X") | Third-person, "what + when," specific trigger vocabulary, front-loaded in the first sentence | Confirmed current (2026) per official best-practices doc fetched this session | Directly determines whether 5-6 grocery skills self-trigger correctly without collision |
| Assuming Rohlík MCP tool names from any single web source | Empirical, live-round-trip discovery only, given official docs 403 and the only detailed tool list found belongs to a different unofficial project | Confirmed this session (new finding: unofficial project has different auth model entirely) | Changes how much trust to place in any pre-written tool name in `mcp-degradation.md` before the spike runs |
| Assuming artifact `storage` capability is confirmed and ready to design against | Treat as unconfirmed pending Step 0 of the spike protocol | New finding this session (live capability roster shows only `downloads`+`mcp`) | Directly gates whether Phase 4 (staples-restock, household-prefs) can assume a skill-writable cross-device store exists |

**Deprecated/outdated:** None specific to this domain — the platform features involved (Skills, Projects, Artifacts, MCP) are all current, actively-evolving 2026 surfaces; treat any date-specific claim in this doc as needing re-verification if picked up much later than 2026-07-31.

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | Artifact `storage` capability exists (in some form) for published artifacts, described as `window.claude.storage`/`window.storage`, 20MB/artifact, personal+shared pools | Standard Stack, Artifact-Storage Spike Protocol | If wrong (capability genuinely doesn't exist), the FOUND-09 spike returns NO-GO fast (a good, cheap outcome) — the real risk is *not* checking Step 0 first and wasting time on write/read tests against a nonexistent API |
| A2 | Community `tomaspavlin/rohlik-mcp` tool names (`search_products`, `add_to_cart`, `get_cart_content`, `get_frequent_items`, etc.) are a reasonable naming hypothesis for the official server | Rohlík MCP Empirical Discovery Protocol, Standard Stack | If wrong, no harm — protocol treats it only as a hypothesis to check against, never as ground truth; risk is limited to wasted pattern-matching effort |
| A3 | Connector approval scope terminology is "Always allow / Needs approval / Blocked" per tool/category (found this session) rather than the "Once/This conversation/This project/Global" terminology cited in prior STACK.md research | Standard Stack (implicit) | If the actual live UI differs from both descriptions, the household simply needs to read the real dialog at connector-setup time — low risk, cosmetic only |
| A4 | 30MB/file, unlimited-files Project Knowledge limit and 200K-token soft cap (triggering RAG-style retrieval beyond it) | Standard Stack | If the actual per-Project or per-file limit differs, the five shared docs + three config files are all small markdown files (well under any plausible limit) — low practical risk for this phase specifically, worth confirming before Phase 3+ adds recipe-sources.md and larger files |

## Open Questions

1. **Does the artifact `storage` capability exist at all in the household's actual account, right now?**
   - What we know: this session's own live capability roster names only `downloads` and `mcp`; multiple community sources describe a storage feature from ~Oct 2025.
   - What's unclear: whether storage is a separate, non-declared, always-on API for published artifacts (plausible, since it doesn't need viewer consent the way `mcp`/`downloads` do) or genuinely absent/renamed/removed.
   - Recommendation: Step 0 of the Artifact-Storage Spike Protocol resolves this empirically; do not let Phase 4 planning proceed on an assumption either way.

2. **What are the official Rohlík MCP server's real tool names, parameters, and error shapes?**
   - What we know: the server is genuinely OAuth-based and distinct from at least one unofficial reverse-engineered alternative; official docs 403 to automated fetch.
   - What's unclear: everything about the actual tool surface.
   - Recommendation: the Rohlík MCP Empirical Discovery Protocol above is the only way to answer this; must be run and recorded before `mcp-degradation.md` is considered complete.

3. **Does the official server expose a favourites-equivalent tool (like the unofficial project's `get_frequent_items`)?**
   - What we know: PROJECT.md's tool list is agnostic on this; STACK.md flagged it MEDIUM confidence.
   - What's unclear: confirmed presence/absence.
   - Recommendation: explicitly check during the live round-trip (step 8 of the discovery protocol); if present, seed-favourites.md's bootstrap role can be scoped as more clearly temporary.

## Environment Availability

This phase's dependencies are claude.ai product-surface features and a live Rohlík account — none of them are probeable from this repository's shell environment (no CLI, no local process, no network endpoint this sandbox can reach corresponds to "claude.ai has code execution enabled" or "the Rohlík connector is attached"). This section documents what must be confirmed by the human, not what this session could verify directly.

| Dependency | Required By | Available (checkable from this repo) | Human-verify at execution time | Fallback |
|------------|-------------|----------------------------------------|-------------------------------|----------|
| claude.ai Project with code execution / Artifacts enabled (Pro/Max/Team/Enterprise) | Confirmation artifact (Phase 2), storage spike (this phase) | N/A — not checkable from this shell | Confirm subscription tier supports custom Skills + publishable Artifacts before uploading anything | None — this is a hard blocker if the plan tier doesn't support it |
| Rohlík MCP OAuth connector attached to the shared Project | Live round-trip (this phase); every future mutating skill | N/A — not checkable from this shell | Confirm connector shows "Connected" in Project settings before running the probe sequence | If connection fails, the whole round-trip (and thus `mcp-degradation.md`'s grounding) blocks — treat as a checkpoint, not a silent skip |
| Two physical devices signed into the SAME shared Claude account | Artifact-storage spike Step 1-2 | N/A | Confirm both devices are logged into the one shared account (not two separate accounts) before starting the spike | If only one device is available, the spike can only test write-then-read-same-device, which does NOT answer D-08's actual question — must be flagged as an incomplete spike, not a NO-GO |

**Missing dependencies with no fallback:** none identified structurally — all three above are things the household already has per PROJECT.md's stated setup (existing shared account, existing two phones, existing Rohlík account), so this table is a pre-flight checklist rather than a list of expected gaps.

## Security Domain

### Applicable ASVS Categories

This phase produces markdown documentation, not application code, so most ASVS categories don't map directly. The two that do:

| ASVS Category | Applies | Standard Control |
|---------------|---------|-------------------|
| V2 Authentication | Yes | Rohlík MCP connector uses OAuth exclusively; this phase's round-trip and its recorded notes must be scrubbed of any token-shaped string before being committed to `mcp-degradation.md` or any Project file — confirmed as a hard, explicitly-stated project constraint, not a general best practice |
| V3 Session Management | No (not applicable — no custom session code; claude.ai's own OAuth session handling is out of this project's control) | — |
| V4 Access Control | Partial | Hard-constraint precedence (allergies/dislikes always win over favourites) is an access-control-shaped invariant for the resolution cascade doc, even though it's not a classic authz check — `household-ruleset.md`'s §A/§B split exists specifically to make this enforceable |
| V5 Input Validation | Partial | The ruleset's own structure (hard constraint vs. preference as distinct, typed sections rather than an undifferentiated list) is itself a validation-shaped design decision that prevents the precedence-inversion failure named in PITFALLS.md P6 |
| V6 Cryptography | No | No cryptographic operations in this phase; OAuth token handling is entirely claude.ai's platform responsibility, never touched directly by any Project file or skill |

### Known Threat Patterns for this stack

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|----------------------|
| Credential leakage into durable, widely-visible context (Project Knowledge files, SKILL.md, saved debugging transcripts) | Information Disclosure | OAuth-only; explicit credential-scan pass over every Project file and this phase's round-trip notes before considering the connector-setup/discovery step complete — named as a literal, checked constraint in CLAUDE.md and PITFALLS.md P19 |
| Hard-constraint (allergy) override by a favourite or ruleset-preference match | Tampering (of the safety invariant, not of data in a technical sense) | `household-ruleset.md`'s A/B/C structure encodes hard-vs-soft as a first-class distinction; resolution-cascade.md must specify hard-constraint filtering as the unconditional first step, never skippable | 
| Confirmation artifact scope creep (later gaining a capability it shouldn't) | Elevation of Privilege | This phase's confirmation-protocol.md doc must state explicitly and permanently: no `mcp`, no `downloads`, no `storage` capability, ever, on the confirmation artifact — a standing constraint re-checked every phase that touches the artifact (PITFALLS.md P9) |

## Sources

### Primary (HIGH confidence)
- `[VERIFIED: artifact-capabilities skill, read directly this session]` — runtime contract 0.1.15; confirms `downloads` and `mcp` are the only declarable capabilities; no `storage` capability listed
- `[CITED: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview]` — fetched directly this session; SKILL.md structure, YAML frontmatter rules, progressive disclosure, cross-surface non-sync
- `[CITED: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices]` — fetched directly this session; description-writing rules, one-level-deep references, fully-qualified MCP tool names, eval-first development

### Secondary (MEDIUM confidence)
- WebSearch: "Claude Projects Knowledge files upload limits size format" — fast.io, datastudios.org, GitHub issue #46655 (30MB/file, unlimited files, 200K-token soft cap / RAG fallback) — not independently confirmed against an official Anthropic doc this session
- WebSearch: "mcp.rohlik.cz official OAuth connector tools list" — confirms mcp.rohlik.cz is genuinely OAuth-based (distinct from the unofficial username/password community project)
- WebSearch: "Claude custom connector approval scope" — surfaces "Always allow / Needs approval / Blocked" terminology, differing from prior research's "Once/This conversation/This project/Global" — needs live-UI confirmation at connector-setup time

### Tertiary (LOW confidence — flagged for validation)
- WebSearch/WebFetch: `github.com/tomaspavlin/rohlik-mcp` tool list (search_products, add_to_cart, get_cart_content, get_frequent_items, etc.) — unofficial, reverse-engineered, username/password-authenticated project; NOT the official server; naming hypothesis only, `[ASSUMED]`
- WebSearch: caipi.ai, eigent.ai, and general blog synthesis on `window.claude.storage`/`window.storage` — 20MB/artifact, personal+shared pools — `[ASSUMED]`, contradicted (or at least unconfirmed) by this session's own live capability roster; official support.claude.com pages 403'd to automated fetch, consistent with prior research passes
- WebSearch: "Claude artifact interactive form return data to conversation" — no documented send-back API found; corroborates STACK.md's own MEDIUM-confidence conclusion that the user's next message is the only channel

## Metadata

**Confidence breakdown:**
- Standard stack (Skills/Project Knowledge mechanics): HIGH — directly fetched official docs this session
- Artifact-storage capability existence/shape: LOW — genuinely unresolved; this is the correct, honest state pending the human-executed spike
- Rohlík MCP tool surface: LOW — genuinely unresolved; this is the correct, honest state pending the human-executed round-trip
- Confirmation-artifact hand-back mechanism: MEDIUM-HIGH — no direct API confirms the negative claim exhaustively, but corroborated across two independent research passes (this session's and STACK.md's) plus this session's own capability-roster read

**Research date:** 2026-07-31
**Valid until:** 30 days for the Skills/Project-Knowledge mechanics (stable platform docs); 7-14 days for the artifact-storage-capability and Rohlík-MCP-specific claims, since both are explicitly fast-moving/unconfirmed areas that should be re-checked if this phase's execution slips significantly past early August 2026
