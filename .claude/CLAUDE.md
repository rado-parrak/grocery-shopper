<!-- GSD:project-start source:PROJECT.md -->

## Project

**Household Grocery Assistant (Rohlík)**

A set of Claude Skills, living in one shared Claude Project, that lets two people
(husband + wife) run their household grocery shopping entirely from their phones.
The assistant resolves vague human requests ("add milk", "we need stuff for
Thursday's dinner") into specific Rohlík.cz products via the official Rohlík MCP,
respecting a household quality ruleset, and pushes them to the shared Rohlík
basket. Checkout and payment stay manual (and are platform-enforced — order
submission is not exposed over MCP).

**Core Value:** Opening a fresh chat on a phone and going from "add milk and bananas" to the
**right kind** of items in the shared basket in **under three turns**, without
opening a computer. If everything else fails, the resolution cascade turning a
vague item into a correct, ruleset-compliant Rohlík product must work.

### Constraints

- **Platform**: Order submission not exposed via MCP — checkout/payment always manual. Structurally guarantees the "never checks out, never pays" requirement.
- **Platform**: Rohlík MCP is experimental and may change or disappear without notice — every skill must degrade to a plain manual list rather than fail silently or hallucinate a successful add.
- **Data**: Claude Project files are read-only to the agent — writable state (learned favourites, staples, recipe repertoire) cannot live there.
- **Accounts**: One shared Rohlík account (one shared basket — the design's central assumption). One personal Claude account shared across two devices (no Team/Enterprise Project needed; single account = single natural scope for shared writable state).
- **UX**: Mobile-only. Short replies, no wide tables, no walls of options. Confirmation must be usable one-handed.
- **Privacy**: OAuth only — no credentials in project files or skill definitions.
- **Turn economy**: Common path ≤ 3 turns; never ask what the ruleset already answers.

<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->

## Technology Stack

## Recommended Stack

### Core Technologies

| Technology | Version/Status (2026) | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Claude Skills (custom, claude.ai) | Current format: `SKILL.md` + YAML frontmatter (`name`, `description` only required fields), uploaded as a zip via **Settings → Features** | Packages each shopping behavior (quick-add, staples-restock, recipe-to-basket, meal-plan, basket-review, household-prefs) as an independently-triggered capability | This is the only supported way to get "preloaded, self-triggering, no-setup-at-chat-time" behavior on claude.ai. Skills are the platform's answer to exactly this shape of problem: reusable, natural-language-triggered, no per-conversation setup. |
| Claude Projects (claude.ai) | Current product surface; Project Knowledge files are **read-only to the agent** at chat time | Container that gives both household members, on both phones, the same preloaded context (Project Knowledge) inside every chat started in that Project | Verified independently (beyond PROJECT.md's own framing) via multiple 2026 sources: Project files can be uploaded/replaced by a human but the agent has no in-chat write path back to them — this is a hard platform constraint, not a design choice, and it must shape where "learned" state lives. |
| Rohlík MCP (official, `https://mcp.rohlik.cz/mcp`) | Custom connector, OAuth-authenticated, declared **experimental / personal-use-only**, "any other use only with prior written consent" per Rohlík's own MCP terms | The only integration surface into the shared Rohlík basket: product search, cart read/add/update/remove, order status/history, deals, personalised recommendations. Order submission deliberately not exposed. | It's the first-party server for the one retailer this project is scoped to (v1 is Rohlík-only, one shared basket). Its experimental status is a Rohlík-side fact, independently corroborated in web search results, not just a PROJECT.md claim — treat every integration decision as defensive-by-default. |
| Claude Artifacts (interactive, in-conversation) | Current product surface; **published** artifacts additionally support two capabilities as of an Oct 2025 platform update: `mcp` (call the viewer's own connectors client-side) and persistent `storage` (per-viewer "personal" store + a "shared" store, up to 20MB text per artifact) | Renders the response-collector confirmation UI (tickboxes + quantity steppers) that the household reviews before any cart mutation | It's the native, no-install way to build a structured, checkbox/stepper confirmation UI inside a chat on a phone. Its *newer* capabilities (artifact-callable MCP, artifact-scoped persistent storage) are real and relevant to this project's writable-state problem, but — see Architecture Patterns below — deliberately NOT the mechanism to use for the confirm→mutate step itself. |

### Supporting Conventions

| Convention | Where it applies | When to Use |
|---------|---------|-------------|
| Third-person, "what + when" `description` field (≤1024 chars, no XML tags) | Every SKILL.md | Always — this is the single field Claude matches incoming phrasing against to decide whether to trigger a Skill among potentially 100+ installed ones. Write "Adds items to the shared Rohlík basket from an ad-hoc request like 'add milk and bananas'. Use when the user names specific grocery items to buy right now." not "I can help you add groceries." |
| Gerund or noun-phrase `name` (kebab-case, ≤64 chars, no reserved words) | Every SKILL.md | Always — e.g. `quick-add`, `staples-restock`, `recipe-to-basket`, not `helper` or `groceries`. |
| Progressive disclosure: SKILL.md body <500 lines, one-level-deep file references | Any skill whose behavior needs more than a short procedure | Once a skill's body approaches ~500 lines or needs domain-specific reference material (e.g. recipe-to-basket's ingredient-extraction rules), split into linked `.md` files referenced directly from SKILL.md — never nest references two files deep, since Claude may only partially read a doubly-indirect file. |
| Fully-qualified MCP tool names (`ServerName:tool_name`) in skill instructions | Any SKILL.md instruction that names an MCP tool | Always, once more than one connector could plausibly be present in the Project — write "Use the Rohlík:cart_read tool" not "read the cart," to avoid tool-not-found ambiguity. |
| MCP tool annotations (`destructiveHint`, `idempotentHint`, `readOnlyHint`) | Read at the connector level, honored at the skill-instruction level | Rohlík MCP is first-party but still labeled experimental; do not assume its tool annotations are conservative/correct. Treat every cart-mutating tool as if `idempotentHint` were false regardless of what the server claims, and always read-before-write yourself (see Architecture Patterns, MCP discipline). |
| One shared Claude account across two devices (not Team/Enterprise) | Project + connector + artifact-storage scoping | This is a deliberate PROJECT.md decision, not a limitation to work around — it sidesteps org-wide skill/connector distribution limits entirely (claude.ai custom Skills are "individual to each user, not shared organization-wide" — irrelevant here because there is only one user/account) and gives artifact "personal" storage a single, stable identity across both phones. |

### Development / Authoring Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| `skill-creator` (Anthropic-distributed authoring skill, also usable inside Claude Code during development) | Scaffolds SKILL.md + frontmatter correctly, and can run description-triggering evals | Use it to draft and iterate every SKILL.md before zipping/uploading to claude.ai — it encodes the same best-practices this document cites (concise body, third-person description, progressive disclosure) and can benchmark whether a description reliably self-triggers against paraphrased test phrasing. |
| Manual eval set (3+ scenarios per skill, per skill's description) | Verifies a skill actually triggers on the natural phrasing it's meant to catch, and does NOT trigger on a sibling skill's phrasing | Anthropic's own authoring guidance: "build evaluations BEFORE writing extensive documentation." For this project, the eval set should specifically include cross-skill collision cases — e.g. confirm "add milk and bananas" triggers `quick-add` and not `staples-restock` or `meal-plan`, given five skills will coexist with overlapping vocabulary (all about groceries). |
| A version-controlled source (git repo, even a personal one) for the SKILL.md sources and shared Project Knowledge docs | Packaging/upload workflow | claude.ai custom Skills are uploaded as zips and Project Knowledge files are uploaded/replaced as flat files — neither is git-backed on the platform side. Keep the authored sources in a repo so edits to the shared internals (resolution cascade, confirmation protocol, degradation policy) are diffable and so re-uploading a corrected zip/file is a deliberate, reviewable act. |

## Architecture Patterns (the six things this stack must get right)

### 1. Skill authoring & reliable self-triggering

- Write descriptions as "what it does" + "when to use it," in third person, with concrete trigger vocabulary the household will actually type/say: not just "quick-add — adds items to the basket" but "Adds specific named grocery items to the shared Rohlík basket from an ad-hoc request. Use when the user names one or more concrete items to buy now (e.g. 'add milk and bananas', 'we need eggs')."
- Because five-to-six skills will coexist and all concern "groceries," deliberately differentiate trigger vocabulary at the description level: `quick-add` triggers on named items; `staples-restock` on recurring/routine language ("we're out of," "restock the usual"); `recipe-to-basket` on a named dish/recipe; `meal-plan` on a date/week-range planning request; `basket-review` on "what's in the cart" / "check the basket" phrasing. Overlapping descriptions across skills is the single biggest cause of wrong-skill or dual-skill triggering once more than 2-3 skills share a domain.
- Keep SKILL.md bodies under ~500 lines; push anything skill-specific-but-long (e.g. recipe-to-basket's ingredient-extraction heuristics) into a same-skill reference file, referenced one level deep from SKILL.md.
- Build 3+ eval phrasings per skill before finalizing its description, explicitly including near-miss phrasings that should trigger a *different* skill, per the multi-skill collision risk above.

### 2. The shared Project as the preload + read-only-knowledge container

- Put static, human-curated, safety-relevant facts in Project Knowledge: the household ruleset (quality rules, dislikes, allergies, brand prefs), budget thresholds, recipe-source trust list, and a bootstrap seed-favourites list. These are exactly the kind of "durable but rarely-changing, human-owned" facts Project Knowledge is designed for.
- Because connector approval in claude.ai has explicit scopes (Once / This conversation / This project / Global), grant the Rohlík MCP connector at **This project** scope rather than Global — it keeps the connector's blast radius to the one Project this household actually uses it in, without needing to re-approve every chat.
- Don't treat Project Knowledge as a place any skill can "learn into." Any skill design that assumes it can persist a new fact there at chat time is designing against a hard platform constraint, not a soft one (see Pitfall 13 in PITFALLS.md, which this independently corroborates).

### 3. One shared source of truth ("resolution cascade"), not per-skill duplication

- Author the resolution cascade, the confirmation-artifact contract, the substitution policy, the MCP-degradation policy, and the audit-line format each as its own Project Knowledge file (not inside any skill's zip).
- Every skill's SKILL.md should *reference* these by name and explicitly instruct "do not re-derive these steps here" — the SKILL.md becomes a thin orchestrator ("for resolution, follow resolution-cascade.md; for confirmation, follow confirmation-protocol.md") rather than a place where the cascade is re-described, however briefly, per skill.
- **What NOT to do and why:** Do not paste even a "just the key steps" summary of the cascade into each skill's own SKILL.md "for speed" or "so the skill is self-contained." The moment two copies of the same logic exist, they will drift — one skill's copy gets a fix or a new field (e.g. an added substitution rule) and the other four don't, silently reintroducing exactly the inconsistent-behavior-across-skills failure this single-source-of-truth requirement exists to prevent. A skill referencing an external doc is only as reliable as the model's instruction-following, since nothing compiles or enforces the reference — mitigate this with the eval set from Pattern 1, testing that each skill's actual resolution behavior matches the doc's cascade at least once per skill.

### 4. Interactive confirmation artifact as a pure response-collector

- Do **not** declare the `mcp` capability on the confirmation artifact. Keep it a pure response-collector: it renders the proposed item list (checkboxes + quantity steppers, one card for the whole batch, not one confirmation per item, to protect the ≤3-turn budget), the human taps/adjusts it, and the *result* — confirmed items, quantities, any unchecked/edited items — is communicated back into the conversation as an ordinary next chat turn (typed, dictated, or a short "yes, add these" / "skip the milk"), which Claude then acts on by calling the Rohlík MCP tools itself, from the skill's own instructions, using the cart-read snapshot taken before the artifact was shown.
- **Why not the mcp-capable route, explicitly:** (1) It would silently reintroduce Pattern 3's single-source-of-truth problem at the confirmation layer — the read-before-write/idempotency/audit-line logic lives in the shared Project Knowledge docs and the skill's own MCP calls; if the artifact also independently calls MCP, that logic must be duplicated into the artifact's JavaScript, which is a second, harder-to-edit copy of the exact same discipline (JS bundled in a zip vs. a markdown doc a human can quickly revise). (2) It removes the one clean auditable boundary between "the household agreed to X" and "X was actually written," which is the core safety property this whole design is built around — if the write partially fails, there is no clean reconciliation point. (3) An `mcp`-capability-declaring artifact "cannot be shared publicly" per the current runtime contract — not a blocker for a private household tool, but a sign this capability is scoped for a different use case (viewer-driven dashboards/tools) than a safety-critical, audited mutation step. (4) The Rohlík MCP is explicitly experimental and may change without notice; keeping every actual tool call inside the skill's own instructions (editable by re-uploading a SKILL.md) rather than inside compiled/published artifact JavaScript (editable only by republishing the artifact) keeps the system's one truly fragile external dependency behind the cheaper-to-fix layer.
- Design the artifact mobile-first: single column, large tap targets for the steppers, no horizontal scroll, and cap the number of items shown per screen/batch (relevant once meal-plan or a weekly restock produces double-digit item counts) — this is a UX requirement independent of the MCP question but belongs to the same artifact.

### 5. MCP tool-usage discipline: idempotence and graceful degradation

- Never trust Rohlík MCP's own idempotency/annotation claims at face value, given its explicit experimental status. Every mutating flow must independently: (a) call `cart_read` immediately before computing what to add (not just once at conversation start — re-read right before the actual write, to minimize the staleness window given two people may act concurrently on the same shared basket); (b) treat "add" as reconcile-to-desired-state (if the item is already present at the target quantity, no-op; if partially present, add only the delta) rather than blind-append; (c) after writing, immediately `cart_read` again and confirm success only from that read-back — never phrase a success message directly off the write call's own return value, since an experimental server's write acknowledgement may not reflect true state.
- Design every MCP call site to have an explicit degrade branch from the outset, not bolted on later: if search, cart-read, or cart-write fails, times out, or returns an unexpected shape, fall back to producing a plain, well-formatted manual shopping list and say so plainly ("Rohlík connection isn't working right now — here's your list to add by hand"). Never silently retry-and-hope, and never fabricate a success message when the underlying call didn't verifiably succeed.
- Use fully-qualified tool names (`Rohlík:cart_read`, `Rohlík:cart_add`, etc.) in every skill's instructions once the Project has more than one connector, to avoid tool-resolution ambiguity.
- Because order submission is structurally not exposed via this MCP (Rohlík's own design choice, not this project's), no defensive code is needed to prevent an accidental checkout call — but every *other* mutating call needs the discipline above precisely because nothing else about the server's reliability is guaranteed.

### 6. OAuth-only credential handling

- Set up the Rohlík MCP connector once via claude.ai's custom-connector OAuth flow; never paste a token, client secret, or raw auth header into any Project Knowledge file, SKILL.md, or saved debugging transcript — including "just temporarily, to get it working."
- Treat this as a literally checked constraint at the end of the connector-setup phase: scan every Project file and skill definition for token-shaped strings before considering setup complete, and repeat the scan after any session where MCP debugging notes get added to the Project.
- Given the server's experimental status, expect the OAuth connection may occasionally need to be re-established (server-side changes, token expiry, or the server itself changing shape); design skills to surface a clear "Rohlík connection needs reconnecting" message on an auth-shaped failure rather than a raw error, but do not attempt to work around this by hardcoding any credential anywhere.

### 7. Writable state, given Project Knowledge is read-only

- *What it is:* Data the retailer already persists natively, per-account, and already exposes read access to via the MCP (`order_history`, presumably a favourites-equivalent surface, if exposed by the official server's tool list).
- *Tradeoffs:* Zero new infrastructure; already durable; already shared across the one shared Rohlík account (and thus both phones); no spike required to prove it's shared, since it's the retailer's own account data. Limitation: it can only hold what Rohlík's own data model supports (which specific products were bought, when) — it cannot hold household-specific metadata Rohlík has no field for, like "restock cadence" or "why this substitution was rejected."
- *Confidence:* HIGH that this data exists and is durable/shared (it's literally the retailer's own account state). MEDIUM on exactly which of it (favourites vs. only order history) the *official* MCP server exposes as callable tools — PROJECT.md's own tool list names "order status/history" and "personalised recommendations" but is agnostic on whether a distinct "favourites" write/read tool exists; verify against the real tool list at connector-setup time.
- *What it is:* A published artifact can declare a `storage` capability giving it up to 20MB of text-only key-value state, split into "personal" (per-viewer, isolated) and "shared" (one common pool every visitor of that published artifact sees/writes) pools. Since this household uses one shared Claude account across two phones, the "personal" pool is, in effect, single-household-scoped already — there's only one authenticated viewer identity to begin with.
- *Tradeoffs:* No separate infrastructure to run; a real, documented 2025-era platform feature, not a hack. Real limitations: (1) storage silently no-ops during artifact *development/preview* — it only activates once the artifact is *published* to a stable URL, which is a different lifecycle than the ephemeral, regenerated-each-turn confirmation artifact described in Pattern 4; a "learned state" artifact would need to be a separate, one-time-published artifact from the per-turn confirmation UI. (2) Unpublishing permanently deletes all associated storage with no republish-to-same-URL recovery — an operational risk if anyone experiments with unpublishing. (3) The precise mechanics of a *skill* (running in the agent's own tool-use loop, not inside a browser iframe) reading or writing that storage are not documented as a direct API — storage read/write only happens from JavaScript running inside the published artifact's own iframe, meaning a skill can't simply query it like a database mid-conversation without somehow routing through a rendered/opened artifact. This is exactly the gap PROJECT.md flags as an unresolved spike, and this research does not resolve it — it only confirms the storage mechanism itself is real, current, and scoped as described.
- *Confidence:* HIGH that the storage mechanism exists, is current (2026), and is scoped per-viewer/per-shared-pool as described (independently corroborated across multiple 2026 sources plus the platform's own runtime-capability documentation fetched in this session). LOW on whether it is *practically usable* as a skill-writable, skill-readable store within this project's actual conversational flow — this is the single biggest open question in the whole stack and should be the first thing spiked, before any skill (staples-restock, household-prefs) is designed to depend on it.

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Custom Skills uploaded per-account on claude.ai | Claude API + Skills API (`skill_id` in the `container` param) with a self-hosted client | If this ever needed to run outside claude.ai's own chat UI (e.g. a bespoke phone app) — but that reintroduces exactly the "no setup steps at chat time" and "preloaded in one shared Project" requirements this brief explicitly wants solved by the consumer product, so it's not a fit here. |
| One shared Claude account across two devices | Claude Team/Enterprise with org-wide Skill/connector provisioning and two named seats | If per-user attribution of who-added-what ever became a hard requirement, or if centrally-managed (admin-pushed) Skills became necessary — neither is true here (PROJECT.md explicitly treats attribution as a nice-to-have audit field, not an auth requirement), and Team/Enterprise adds cost and setup complexity this two-person household doesn't need. |
| Project Knowledge docs as the shared-internals "library" | A dedicated, narrowly-triggered internal Skill that other skills reference by name (Anthropic's docs explicitly encourage "combining Skills for complex tasks") | If Project Knowledge context inclusion ever proves unreliable in practice (e.g. a long file gets truncated or deprioritized against a large system prompt with 6 skills' worth of description metadata already loaded) — a fallback worth testing if Pattern 3 misbehaves in practice, since it uses a documented composition mechanism instead of an inferred one. |
| Rohlík's own favourites/order-history as primary writable state | A dedicated external database/service (e.g. a small hosted key-value store the skills call via a second MCP server) | If the artifact-storage spike fails AND Rohlík's exposed tools don't cover the needed metadata (e.g. restock cadence) — at that point a purpose-built second MCP server (self-hosted, OAuth-protected) is the fallback, but it's meaningfully more infrastructure than this project's "no automation without a human present, keep it simple" ethos wants for a two-person household tool; treat as a last resort, not a first design. |

## What NOT to Use / Do

| Avoid | Why | Use Instead |
|-------|-----|--------------|
| Re-describing the resolution cascade (or confirmation protocol, degradation policy, audit format) inside more than one SKILL.md | Guarantees drift between skills over time — a fix or added rule in one copy won't propagate to the others, silently reintroducing inconsistent per-skill behavior (Pitfall/Anti-Pattern already named in this project's own ARCHITECTURE.md and PITFALLS.md) | One Project Knowledge doc per shared concern; every skill references it by name and is instructed not to re-derive it |
| Declaring an `mcp` capability on the confirmation artifact so its "Confirm" button writes the cart directly | Removes the human-in-the-loop audit boundary between "agreed" and "written"; duplicates read-before-write/idempotency/audit logic into artifact JavaScript instead of the shared Project Knowledge docs; the Rohlík MCP's experimental status argues for keeping the one truly fragile dependency behind the cheaper-to-edit layer (a markdown skill doc, not published artifact code) | Confirmation artifact stays capability-free (pure response-collector); the actual MCP write happens from the skill's own instructions, in the same conversation turn, immediately after the human's confirming message |
| Pasting an OAuth token, client secret, or raw auth header into any Project Knowledge file, SKILL.md, or saved debugging note "temporarily" | Project files and skill bodies are durable, relatively widely-visible context with no secret-scoping of their own; a leaked credential there has a long-lived, wide blast radius until manually found and rotated | OAuth-only, entered once through the connector's own auth flow; scrub any saved MCP debugging transcripts of token-shaped strings before keeping them |
| Assuming Rohlík MCP's write calls are idempotent or that a write's own return value proves success, because it's a first-party official server | It's explicitly declared experimental and may change without notice; first-party does not mean hardened, and no annotation/behavior should be trusted without verification | Independent read-before-write and read-back-after-write on every mutating call, regardless of what the server claims about itself |
| Building any skill that assumes artifact persistent storage is trivially skill-readable/writable mid-conversation, before testing it | This is the one genuinely unverified mechanism in this whole stack — the storage API is real, but it lives inside a published artifact's iframe, not obviously reachable from a skill's own tool-use loop; assuming otherwise risks building several skills on a foundation that silently doesn't do what's needed | Spike this explicitly and early (write-from-chat? cross-device-visible?); default to Rohlík-native favourites/order-history until proven otherwise |
| Uploading the same custom Skill separately to claude.ai and expecting it to also work via the API, or vice versa | Custom Skills do not sync across claude.ai / API / Claude Code — each surface requires its own separate upload/registration | If this project ever needs the same skill logic on another surface, treat it as a separate packaging step, not a given |

## Stack Patterns by Variant

- Use it directly as the resolution cascade's steady-state preference source (ahead of the read-only seed-favourites.md bootstrap file), since it's both native and mutable via ordinary Rohlík shopping, requiring zero new writable-state infrastructure.
- Because Rohlík favourites is Because it's not directly write-controllable by a skill in the way a purpose-built store would be (it's populated by the retailer's own logic, likely from actual purchase behavior), don't rely on it for state a skill needs to *deliberately* set (like an explicit "we've decided X is the new brand" instruction) — that still needs Option B or a human-edited Project Knowledge update.
- Promote it to the primary mechanism for anything Rohlík can't model (staples cadence, curated repertoire notes, rejected-substitution history for staleness detection) and treat Rohlík favourites/history as a secondary signal.
- Fall back to a "state viewer/editor" pattern: a dedicated, explicitly-triggered skill (e.g. `household-prefs`) that, when a preference needs updating, renders a small published artifact whose sole job is to read/write that one piece of state and report a human-readable summary back into the conversation for the human to note — accept that this is a heavier-weight interaction than transparent skill-side storage, and scope it to genuinely low-frequency updates (preference changes, not per-turn cascade lookups).

## Version / Surface Compatibility Notes

| Surface | Skill format | Sharing scope | Notes |
|---------|--------------|---------------|-------|
| claude.ai (this project's target) | SKILL.md zip, uploaded via Settings → Features | Individual to the uploading account (irrelevant here — one shared account) | Requires code execution enabled on the plan; Pro/Max/Team/Enterprise |
| Claude API | Skill uploaded via `/v1/skills`, referenced by `skill_id` in the `container` param alongside the code-execution tool | Workspace-wide | Not the target surface for this project; mentioned only because Skills don't sync from claude.ai to here if ever needed |
| Claude Code | Filesystem-based, `.claude/skills/` or `~/.claude/skills/` | Personal or project(git)-shared | Not the target surface; mentioned only to avoid confusing this format with claude.ai's zip-upload format when authoring |

## Sources

- `/home/user/grocery-shopper/.planning/PROJECT.md` — project brief; requirements, constraints, and key decisions this research verifies against current external documentation
- `/home/user/grocery-shopper/.planning/research/ARCHITECTURE.md` — sibling research document; this STACK research grounds and cross-checks its "shared Project Knowledge as pseudo-library" and "response-collector, never-calls-MCP" patterns against current platform documentation rather than re-deriving them independently
- `/home/user/grocery-shopper/.planning/research/PITFALLS.md` — sibling research document; corroborates several of this document's "what NOT to do" items as named, anticipated failure modes
- [Agent Skills — Claude Platform Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) — fetched directly; SKILL.md structure, frontmatter field rules, progressive disclosure (3-level loading), per-surface sharing scope, cross-surface non-sync
- [Skill authoring best practices — Claude Platform Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) — fetched directly; description-writing conventions, naming conventions, one-level-deep references, MCP fully-qualified tool names, eval-first development
- `artifact-capabilities` skill (bundled, this session) + its `mcp.d.ts` / `downloads.d.ts` type definitions — fetched/loaded directly; confirms the `mcp` and `storage`-adjacent runtime capabilities for published artifacts, the viewer-consent/non-public-sharing constraint on `mcp`-declaring pages, and the call-envelope shape
- WebSearch: "Claude Skills SKILL.md YAML frontmatter description best practices 2026" — corroborating secondary sources (joseparreogarcia.substack.com, KDnuggets, agentman.ai) on frontmatter rules and description requirements
- WebSearch: "Claude Projects preload skills custom connector OAuth MCP 2026" — support.claude.com / sunpeak.ai / explainx.ai secondary sources on connector OAuth setup and per-org Skill provisioning (Team/Enterprise-only; not applicable to this project's single-account design)
- WebSearch: connector approval scopes ("Once / This conversation / This project / Global") — secondary-source corroboration only; primary Anthropic docs pages returned HTTP 403 to automated fetch in this session — MEDIUM confidence, worth re-confirming against the live UI at connector-setup time
- WebSearch: "Claude Artifacts persistent storage window.claude 2025 2026" — caipi.ai, eigent.ai, and general search-result synthesis corroborating the October 2025 "MCP and persistent storage" artifact update, the 20MB text-only limit, personal-vs-shared storage pools, and unpublish-deletes-all-storage behavior
- WebSearch: "Rohlik MCP mcp.rohlik.cz official server experimental personal use" — corroborates PROJECT.md's own characterization of the official server's experimental/personal-use-only terms independently (rohlik.cz's own MCP docs/terms pages returned HTTP 403 to automated fetch in this session and could not be read directly — treat exact tool list and rate limits as needing empirical verification in an early capability-discovery spike, per PITFALLS.md's own recommendation)
- WebSearch: "MCP tool annotations destructiveHint idempotentHint" — mcpblog.dev / chatforest.com secondary sources on the MCP tool-annotation convention this document's discipline guidance is grounded in
- WebSearch: "Claude custom connector OAuth credentials security best practice" — corroborates OAuth-only, no-credentials-in-URLs/prompts guidance

<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->

## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->

## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->

## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->

## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:

- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->

## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
