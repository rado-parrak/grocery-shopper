# Feature Research

**Domain:** Phone-first, bilingual (Czech/English) household grocery-shopping assistant — Claude Skills over the Rohlík MCP, two people sharing one basket, manual checkout
**Researched:** 2026-07-31
**Confidence:** MEDIUM — grounded in the project brief (PROJECT.md), the household grocery assistant category broadly (shopping-list apps like AnyList/Bring!/Out of Milk, recipe-to-cart tools like Mealime/Paprika, and voice assistants' shopping-list skills), and first-principles reasoning about LLM-agent-over-e-commerce-MCP constraints. No live competitor testing was performed (text-only research); treat competitor specifics as directional, not verified.

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete or unsafe to use unattended-ish on a shared account.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Reliable item resolution (vague → concrete product) | Every grocery list tool — from a paper list to Alexa's — turns "milk" into a purchasable thing; if it resolves to the wrong milk repeatedly, the product is unusable | MEDIUM | This is the resolution cascade itself (ruleset → favourites → ask). Core spine every skill depends on. |
| Confirmation before any basket write | Any tool that mutates a shared, money-bearing cart without a review step will get untrusted fast — one bad silent add and the household stops using it | LOW–MEDIUM | Response-collector artifact (tickboxes + qty stepper); agent reads back and confirms verbally. Must be one-handed, mobile-safe. |
| Running total / basket visibility | Users comparison-shop mentally against a rough budget; not seeing a total before adding is how surprise overspend happens | LOW | Basket-review skill's core function; also surfaces inside quick-add/meal-plan confirmations. |
| Idempotent adds (no duplicate items) | A shared basket edited by two people from two phones will get "add milk" issued twice; if that doubles the order, trust is destroyed immediately | LOW–MEDIUM | Read-cart-before-add pattern. Must be the default behavior of every skill that writes, not just quick-add. |
| Graceful MCP degradation | Rohlík MCP is explicitly experimental/personal-use and can vanish; a tool that just errors or hallucinates a successful add when the backend is down is worse than no tool | MEDIUM | Falls back to a plain manual list (markdown/text) the user can act on themselves in the Rohlík app. Applies to every skill uniformly. |
| Search/add a single named item | The absolute minimum of "grocery assistant" — add one thing to the list/cart | LOW | quick-add's base case (no ambiguity, no substitution). |
| Handling ambiguous/out-of-stock items without silently guessing | Even basic list apps (AnyList, Bring!) let a user pick from suggestions rather than auto-picking; auto-picking on a shared paid basket is worse because it spends real money | LOW–MEDIUM | Covered by substitution policy (propose + approve, never silent) and the cascade's "ask user" tier. |
| Recurring/staple items list | Every household grocery workflow has a "same 15 things every week" pattern; competitors (AnyList favorites, Bring! recipes, Instacart reorder) all support this | MEDIUM | staples-restock skill. Table stakes as a *concept*; the "propose never auto-add" policy is what makes it safe, not differentiating on its own — most competitors also require a tap to add. |
| Recipe → shopping list | Converting a recipe's ingredients into a shopping list is standard in the category (Paprika, Mealime, Rohlík's own site probably has something similar) | MEDIUM–HIGH | recipe-to-basket. Parsing/ingredient-extraction and quantity normalization is the hard part; the "add to basket" tail end is the easy part. |
| Basic multi-meal/week list generation | Meal-planning-to-list is a well-known pattern (Mealime, Whisk, Paw Print, etc.) | HIGH | meal-plan skill — table stakes as a category feature, but the pack-size-aware de-dup is where this product tries to be better than "just concatenate ingredient lists" (see Differentiators). |
| A single durable ruleset for preferences (allergies, dislikes, brand prefs) | Any two-person household using a shared tool needs one place preferences live, or every session re-litigates "not that milk" | MEDIUM | household-prefs skill. Table stakes that this exists at all; where it lives (Rohlík favourites vs. artifact storage, since Project files are read-only) is an open architecture question, not a feature question. |
| Auditability of what got added and why | Users won't trust an agent spending shared household money without being able to see what/qty/cost/why after the fact, especially since two different people may check the basket | LOW–MEDIUM | Every write reports what/qty/cost/matched-rule. Comes largely free if resolution+confirmation are implemented well — it's a reporting layer on top of decisions already made. |
| Bilingual input tolerance (Czech/English mixed) | Both household members code-switch; a tool that only understands one language half-fails on day one | LOW–MEDIUM | This is a prompt/skill-design property (Claude's native multilingual strength) more than new engineering — but worth calling out because Czech product names/units (ks, g, l) must map correctly regardless of input language. |

### Differentiators (Competitive Advantage)

Features that set this product apart from generic shopping-list apps or a bare Rohlík MCP session. Not required for baseline usability, but this is where the product earns its keep over "just use the Rohlík app."

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Ruleset-driven cascade resolving to the *right* product in ≤3 turns | This is the stated Core Value in PROJECT.md. Generic tools return a search-results grid and make the human choose every time; this product encodes the household's standing preferences so "milk" *just means* the right milk, forever, without re-asking | HIGH | Requires the ruleset to be expressive enough (fat %, brand, organic, allergy exclusions) and the cascade logic to prefer ruleset > favourites > ask, never skipping to "ask" when the ruleset already answers. Biggest engineering and prompt-design investment in the whole project. |
| Learned staples from order history (vs. hand-maintained lists) | Competitors' recurring lists are manually curated (Bring!, AnyList favorites); inferring "you buy this every ~9 days" from Rohlík order history removes upkeep labor entirely | HIGH | Depends on Rohlík MCP exposing order history with enough granularity (dates, SKUs, quantities) to infer cadence. Also depends on where the "learned" state persists, since Project files are read-only (per PROJECT.md's Claude constraint) — likely needs artifact storage or re-derivation from Rohlík history each session. Propose-never-auto-add keeps this safe even if the inference is sometimes wrong. |
| Meal-plan pack-size-aware de-duplication | Naively merging N recipes' ingredient lists produces "0.3 onion, 0.5 onion, 0.2 onion" nonsense or triple-buys of a 1kg flour bag; rounding to real Rohlík pack sizes while merging across meals is materially harder than a basic meal-planner and directly reduces both waste and cost — a genuine budget/sustainability edge | HIGH | Requires mapping ingredient quantities to actual catalogue pack sizes (via product search) *before* finalizing the merged list, and depends on the resolution cascade to know which product a merged ingredient should resolve to. |
| Bilingual input, Czech-normalized output | Beyond "tolerates Czech and English," actually always confirming/reporting in the language of the Rohlík catalogue avoids confirmation-list items not matching what's on the receipt/app screen later | MEDIUM | Cheap once the cascade resolves to a real Rohlík product (the product object arrives Czech-labelled); the differentiator is disciplined consistency, not new capability. |
| Interactive one-handed confirmation (stepper + tickboxes, response-collector only) | Most chat-based shopping bots make you type quantities or re-describe changes; a structured tappable artifact that never itself calls the MCP is faster on a phone and keeps the "who actually approved this write" boundary crisp | MEDIUM | This is UX differentiation with a safety side-benefit: because the artifact can't call MCP directly, the mutating call is always a separate, auditable, agent-initiated step after a human looks at the final read-back. |
| Two-person shared-basket awareness (idempotency + attribution) | Most personal shopping assistants assume one user; explicitly designing for "two people, one basket, from two different phones/sessions" — read-before-write, no duplicate adds, audit trail showing what happened — is a real edge over single-user tools repurposed for a household | MEDIUM–HIGH | Depends on idempotent adds (table stakes) but the household-specific framing (both people's inputs treated as equally authoritative, ruleset as the single source of truth both defer to) is the differentiating packaging. |
| Budget guardrail with soft-warn + hard-block (not just visibility) | Showing a total is table stakes; a hard cap that actually *blocks* further adds until acknowledged is a meaningfully stronger commitment than competitors, most of which only show running totals passively | MEDIUM | basket-review + cross-cutting budget logic. Requires deciding cap ownership (per-order? weekly?) — a product decision this research doesn't resolve, but the mechanism (soft warn, hard block) is the differentiator vs. passive-total-only tools. |
| Uploaded cookbook / photographed recipe → basket | Recipe-to-basket from a name or URL is table stakes-adjacent (competitors do this); ingesting a photographed page from a physical cookbook is a step beyond typical recipe importers, which usually require a URL or manual paste | HIGH | Multimodal ingestion + ingredient extraction from an image, then the same resolve/confirm pipeline as any other recipe source. Higher complexity than URL-based import; worth sequencing after the URL/name path proves out. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems for this specific product — deliberately excluded from v1 per PROJECT.md's Out of Scope section, with rationale for *why* each is a trap here, not just a "not now."

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|------------------|-------------|
| Checkout / payment / delivery-slot automation | "Just finish the job" — if the agent can add items, why not let it also submit the order and pick a slot? | Rohlík deliberately doesn't expose order submission over MCP (platform-enforced), and even if it did: irreversible spend + delivery-slot commitments made by an LLM agent without a human's final look is the single highest-blast-radius mistake this product could make. It converts a shopping assistant into an unsupervised payment agent | Basket-review skill hands off a fully-prepared, reviewed basket; the human opens the Rohlík app/site and taps "place order" themselves. This boundary is structurally guaranteed by the platform, not just a policy choice — lean into that rather than working around it. |
| Multi-retailer support (Tesco, Kaufland, etc.) | "What if Rohlík doesn't have X" / "compare prices across stores" | Doubles the resolution cascade, ruleset, and MCP-integration surface for a household that has explicitly standardized on one shared Rohlík basket as its central data model; cross-retailer basket merging and price comparison is a different, much larger product | Graceful degradation already covers "Rohlík doesn't have it" (falls back to a manual note); if a genuine multi-retailer need emerges post-v1, it's a new milestone, not a v1 feature |
| Nutrition / calorie / macro tracking | Natural extension once you know exactly what's in the basket | Scope creep into a health/diet product with its own accuracy expectations (mislabeling calories has different stakes than mislabeling a milk brand) and UI needs (charts, trends) that fight the "short reply, no walls of options" mobile constraint | If wanted later, it's a separate reporting layer read *from* Rohlík order history, not a resolver/writer skill — keep it decoupled from the basket-mutation spine entirely |
| Pantry / inventory tracking (what's already in the fridge) | Feels like the obvious next step from a shopping list — "don't buy milk if we still have some" | Requires an entirely separate, high-maintenance data source (manual stock entry or computer-vision fridge scanning) that has nothing to do with Rohlík's catalogue/cart APIs; staleness of that data (nobody logs what they used) silently produces wrong resolutions and erodes trust in the *whole* system, not just this feature | staples-restock's learned cadence from order history is a much cheaper, self-maintaining proxy for "we're probably low on this" without needing real-time inventory truth |
| Any human-out-of-loop automation (scheduled auto-restock, auto-approve substitutions, "just handle it") | Reduces friction — "why do I have to confirm every time, just add my staples every Monday" | Directly violates the project's safety boundary: every mutation needs a person present and approving, on a shared account where money moves. Auto-anything on a shared basket is exactly the failure mode (duplicate/wrong/unwanted adds) that erodes household trust the fastest and is hardest to undo once it's a habit users rely on | staples-restock proposes on request or next chat open, never on a timer with auto-add; the interactive confirmation step is non-negotiable infrastructure, not a v1-vs-v2 toggle |
| Silent substitution when an item is out of stock | Feels efficient — "just get something close enough" | A silent swap on a shared account is indistinguishable from a wrong/duplicate add until someone notices the receipt; violates the explicit "propose + approve, never silent" policy and undermines auditability | Substitution is always proposed in the same confirmation flow as a normal add — no separate "trusted auto-swap" tier, even for close substitutes |
| Rich/long chat replies, wide comparison tables, multi-screen option grids | Feels thorough — "show me all 8 milk options with prices so I can choose the best" | Directly fights the stated mobile-only UX constraint (short replies, no wide tables, no walls of options, one-handed usable); also slows down the ≤3-turn core-value target this product is graded on | Cascade should surface *one* recommended product per ambiguous item (from ruleset/favourites), only falling back to a short pick-list (2-3 options max) when nothing matches; never a full catalogue dump |

## Feature Dependencies

```
Resolution Cascade (ruleset → favourites → ask)
    └──requires──> Household-Prefs (ruleset/favourites/staples/sources as data)
    └──requires──> Rohlík MCP (product search/read)
                       └──degrades-to──> Manual List Fallback (graceful degradation)

Interactive Confirmation Artifact (tickboxes + stepper, response-collector only)
    └──requires──> Resolution Cascade (needs resolved candidates to present)
    └──gates──> Any Basket Write (Rohlík MCP add/update)

Idempotent Adds
    └──requires──> Basket Read (Rohlík MCP cart read, before every write)
    └──requires──> Rohlík MCP

Substitution Policy (propose + approve, never silent)
    └──requires──> Interactive Confirmation Artifact
    └──requires──> Resolution Cascade (to generate the proposed substitute)

Budget Guardrail (running total, soft warn, hard cap)
    └──requires──> Basket Read (Rohlík MCP cart read, for live total)
    └──enhances──> Interactive Confirmation Artifact (total shown at confirm time)
    └──enhances──> Basket-Review

Auditability (what/qty/cost/why)
    └──requires──> Resolution Cascade (need to know which rule/favourite matched)
    └──requires──> Interactive Confirmation Artifact (need to know what was actually approved)

quick-add
    └──requires──> Resolution Cascade
    └──requires──> Interactive Confirmation Artifact
    └──requires──> Idempotent Adds

staples-restock
    └──requires──> Resolution Cascade
    └──requires──> Household-Prefs (staples list, learned or hand-edited)
    └──requires──> Interactive Confirmation Artifact
    └──enhanced-by──> Learned Staples from Order History (Rohlík MCP order history read)

recipe-to-basket
    └──requires──> Resolution Cascade (per-ingredient resolution)
    └──requires──> Interactive Confirmation Artifact
    └──requires──> Ingredient Extraction (from name/URL/uploaded book — increasing complexity in that order)

meal-plan
    └──requires──> recipe-to-basket's ingredient extraction (reused, not duplicated)
    └──requires──> Meal-Plan Pack-Size Dedup (merge + round to real Rohlík pack sizes)
                       └──requires──> Resolution Cascade (must resolve to a real product to know its pack size)
    └──requires──> Interactive Confirmation Artifact

basket-review
    └──requires──> Basket Read (Rohlík MCP cart read)
    └──requires──> Budget Guardrail
    └──produces──> Handoff to Manual Checkout (a boundary, not a feature — deliberately stops here)

Bilingual Input/Output ──enhances──> Resolution Cascade, all skills (cross-cutting language handling, not a dependency chain of its own)

Checkout/Payment Automation ──conflicts──> Platform constraint (order submission not exposed via MCP) and the "human always approves mutations" safety boundary
Pantry/Inventory Tracking ──conflicts──> staples-restock's cheaper "learned cadence" approach (redundant, higher-maintenance alternative to the same goal)
```

### Dependency Notes

- **Everything downstream requires the Resolution Cascade + Interactive Confirmation Artifact + Rohlík MCP layer.** These three are the actual spine of the product, exactly as PROJECT.md frames the "if everything else fails, this must work" core value. Every one of the six skills is a variation on: gather intent → resolve via cascade → confirm via artifact → write idempotently → audit. Get the spine wrong and no skill can be right.
- **Idempotent adds requires basket read before every write**, which in turn means basket-review's "read cart" capability is not just its own skill — it's a shared primitive every write-capable skill needs first. Consider it cross-cutting infrastructure rather than basket-review-exclusive.
- **Substitution policy sits downstream of both the cascade (to propose the substitute) and the confirmation artifact (to get approval)** — it cannot be implemented as a shortcut that bypasses either, or it becomes exactly the "silent substitution" anti-feature.
- **meal-plan depends on recipe-to-basket's extraction logic being reusable**, not reimplemented — building meal-plan first without recipe-to-basket existing would duplicate the hardest part (ingredient parsing) twice. Sequencing recipe-to-basket before meal-plan is the natural order, not just alphabetical.
- **Learned staples from order history depends on Rohlík MCP exposing usable order-history granularity** — this is an external dependency risk (the MCP is declared experimental) and also depends on solving where "learned" state persists, since Project files are read-only. If artifact-storage or Rohlík-favourites can't hold this, learned staples degrades to "hand-edited only," which is still table stakes-safe but loses the differentiator.
- **Budget guardrail and auditability both enhance rather than gate the confirmation artifact** — they make each confirmation more informative but the confirmation flow works (in degraded form) without them. This matters for sequencing: ship the bare confirm-then-write loop first, then layer total/cost-visibility on top.
- **Checkout/payment automation conflicts with the platform constraint directly** (order submission isn't exposed over MCP) and with the human-in-loop safety requirement — this isn't a "someday" feature so much as a structurally foreclosed one; no amount of product polish changes that boundary, so it shouldn't appear on any roadmap.
- **Pantry/inventory tracking conflicts with (is a redundant, costlier alternative to) learned-staples-from-order-history** — both attempt to answer "what do we probably need," but inventory tracking requires new, staleness-prone data entry while order-history inference reuses data Rohlík already has. Building both would be wasted effort chasing the same signal.

## MVP Definition

### Launch With (v1)

Minimum viable product — what's needed to validate the concept per PROJECT.md's stated first milestone ("quick-add end-to-end proves the cascade + confirmation artifact + a real basket write").

- [ ] Resolution cascade (ruleset → favourites → ask) — the mechanism the whole product's value proposition depends on
- [ ] household-prefs, hand-edited only (ruleset, favourites, allergies, brand prefs) — the cascade has nothing to resolve against without it; learned/auto-population can come later
- [ ] Interactive confirmation artifact (tickboxes + quantity stepper, response-collector only) — non-negotiable safety gate before any write
- [ ] quick-add skill, full path (ad-hoc item → resolve → confirm → basket) — proves the entire spine end-to-end on the simplest possible skill
- [ ] Idempotent adds (read cart before write) — required from day one on a shared basket, not an later hardening pass
- [ ] Graceful MCP degradation (manual list fallback) — must exist before real usage, since the MCP is explicitly unstable
- [ ] Basic auditability (what/qty/cost/why reported after each write) — cheap once resolution+confirmation exist; skipping it undermines trust immediately

### Add After Validation (v1.x)

Features to add once the quick-add spine is proven reliable in real household use.

- [ ] staples-restock (hand-edited staples list, propose-never-auto-add) — trigger: quick-add's cascade+confirm loop is trusted and stable enough to reuse for a recurring list
- [ ] basket-review with running total + budget soft-warn/hard-cap — trigger: enough real baskets have been built via quick-add/staples that "what's my total and am I over budget" becomes a real pain point
- [ ] recipe-to-basket via name/URL — trigger: staples-restock validates that the cascade generalizes beyond single ad-hoc items to a list of resolved ingredients
- [ ] Substitution policy (propose + approve) as an explicit, tested flow — trigger: real out-of-stock events start occurring in practice, not just theoretically

### Future Consideration (v2+)

Features to defer until the core loop has product-market fit within the household.

- [ ] Learned staples from order history (auto-inferred cadence) — defer until hand-edited staples-restock is in daily use and the "what do we actually reorder, how often" pattern is empirically visible, and until the writable-state persistence question (artifact storage vs. Rohlík-derived) is resolved
- [ ] meal-plan with pack-size-aware de-duplication — defer until recipe-to-basket's ingredient-resolution path is solid for single recipes; meal-plan multiplies that complexity across N recipes and adds a genuinely hard merge/rounding problem
- [ ] Uploaded/photographed cookbook ingestion for recipe-to-basket — defer behind the URL/name-based recipe import; multimodal extraction is materially harder and validates nothing the simpler path doesn't already validate
- [ ] Hard budget cap that blocks adds — defer behind soft-warn-only budget visibility until the household has agreed on what a sensible cap actually is (a policy question, not an engineering one)

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|----------------------|----------|
| Resolution cascade | HIGH | HIGH | P1 |
| Interactive confirmation artifact | HIGH | MEDIUM | P1 |
| quick-add skill | HIGH | MEDIUM | P1 |
| Idempotent adds | HIGH | LOW | P1 |
| Graceful MCP degradation | HIGH | MEDIUM | P1 |
| Auditability (what/qty/cost/why) | MEDIUM | LOW | P1 |
| household-prefs (hand-edited) | HIGH | MEDIUM | P1 |
| staples-restock (hand-edited) | MEDIUM | MEDIUM | P2 |
| basket-review + running total | MEDIUM | LOW–MEDIUM | P2 |
| Budget soft-warn | MEDIUM | LOW | P2 |
| Substitution policy (propose+approve) | MEDIUM | MEDIUM | P2 |
| recipe-to-basket (name/URL) | MEDIUM | MEDIUM–HIGH | P2 |
| Budget hard-cap | LOW–MEDIUM | LOW–MEDIUM | P3 |
| Learned staples from order history | HIGH (differentiator) | HIGH | P3 |
| meal-plan + pack-size dedup | HIGH (differentiator) | HIGH | P3 |
| Uploaded cookbook ingestion | LOW–MEDIUM | HIGH | P3 |
| Checkout/payment automation | — | — | Excluded (platform-foreclosed) |
| Pantry/inventory tracking | — | — | Excluded (redundant, high-maintenance) |
| Multi-retailer support | — | — | Excluded (breaks core basket assumption) |

**Priority key:**
- P1: Must have for launch (the quick-add spine)
- P2: Should have, add when possible (rounds out the everyday skill set)
- P3: Nice to have, future consideration (the stated differentiators — high value but sequenced behind a proven spine, per HIGH complexity and dependency on P1/P2 groundwork)

## Competitor Feature Analysis

Directional comparison against the closest analogues in the category (shared shopping lists, recipe-to-list tools, and voice-assistant shopping skills). Not independently verified against live products in this research pass — treat as informed priors to sanity-check design choices, not as ground truth about current competitor behavior.

| Feature | Generic shared shopping-list apps (e.g., AnyList, Bring!, Out of Milk) | Voice-assistant shopping lists (e.g., Alexa/Google Assistant shopping list) | Our Approach |
|---------|---------------------------------------------------------------------|-------------------------------------------------------------------------|--------------|
| Item resolution | User picks from their own manually-curated item/favorites list; little to no automatic disambiguation | Fuzzy match to a retailer catalogue, often with no household-specific preference layer; frequently over-broad or generic matches | Ruleset-driven cascade that encodes household-specific quality/brand/allergy rules ahead of a bare catalogue search — resolves to *our* right answer, not just *a* plausible match |
| Confirmation before commit | Adding to a shared list is low-stakes (no basket, no spend) so most apps add instantly, no confirmation step | Voice add is typically instant/no confirmation; basket-add (where supported) sometimes has a lightweight voice confirm | Structured, always-on confirmation artifact before any basket write — because this product writes to a spendable shared basket, not a passive list |
| Recurring items | Manually maintained favorites/recipes; no inference from purchase history in most consumer list apps | Some reorder-from-history features exist on retailer apps, but not paired with a household ruleset | staples-restock starts hand-edited (parity with competitors) with learned-from-order-history as a deferred differentiator |
| Recipe → list | Common (Paprika, Mealime, Whisk-style tools); usually one recipe at a time, list output only, no cart integration with resolution/substitution logic | Not typically supported | recipe-to-basket adds the resolve/confirm/substitute pipeline on top, plus (eventually) meal-plan-level pack-size-aware merging across multiple recipes, which most consumer tools don't attempt |
| Budget visibility | Rare — most shopping-list apps don't track live pricing or a running total at all | Not typically supported | Running total, soft-warn, hard-cap tied directly to a real, live Rohlík basket total — a genuine gap most competitors in this category don't fill |
| Graceful degradation on backend failure | N/A — most of these apps *are* the source of truth (no fragile third-party API dependency) | Backed by stable, first-party retailer integrations, so degradation is rarely a designed-for scenario | Explicit fallback to a manual list is a differentiator forced by necessity (Rohlík MCP is experimental/personal-use), not one competitors needed to solve |

## Sources

- `/home/user/grocery-shopper/.planning/PROJECT.md` — project brief: scope, skill inventory, cross-cutting internals, constraints, out-of-scope decisions (primary source for this research)
- Category knowledge of shared shopping-list apps (AnyList, Bring!, Out of Milk) and recipe-to-list tools (Paprika, Mealime, Whisk) as general reference points for table-stakes expectations
- Voice-assistant shopping-list skills (Alexa, Google Assistant shopping lists) as reference points for "instant add, no confirmation" contrast
- First-principles reasoning about LLM-agent-over-e-commerce-MCP risk surface (silent writes, duplicate writes, unstable backend) — no external tool/document searched for this; treated as domain expertise applied to the brief's stated constraints
- No live product testing or web search was performed for this research pass; competitor-specific claims above are directional priors, flagged as MEDIUM confidence

---
*Feature research for: Phone-first bilingual household grocery assistant (Claude Skills + Rohlík MCP)*
*Researched: 2026-07-31*
