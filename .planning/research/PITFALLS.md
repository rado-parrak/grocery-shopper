# Pitfalls Research

**Domain:** Phone-first household grocery assistant — Claude Skills over the experimental Rohlík MCP, two users, one shared basket, bilingual CZ/EN input with Czech catalogue output
**Researched:** 2026-07-31
**Confidence:** MEDIUM — grounded in the project's own stated constraints (PROJECT.md), general MCP/agentic-tool-use failure modes, and conversational-commerce/shared-cart UX precedent. No access to Rohlík MCP's actual behavior/docs in this pass; treat MCP-specific warning signs as hypotheses to confirm against the real server in Phase 0.

## Critical Pitfalls

### Pitfall 1: Hallucinated success — reporting an add that never happened

**What goes wrong:**
The agent calls the Rohlík MCP add-to-cart tool, the call errors, times out, or returns an ambiguous/partial response, and the agent — following its normal conversational instinct to be helpful and fluent — tells the user "Added 2% milk to your basket" anyway. The user believes the basket is correct and never checks. The mistake surfaces days later at delivery, or never, because nobody re-verifies a chat transcript against a live cart.

**Why it happens:**
LLM agents are trained to produce a plausible confirming sentence once a tool call has been dispatched, and tool-call error payloads (JSON error objects, empty 200s, MCP protocol-level failures) are often less salient in context than "the user asked to add milk, so the next sentence should confirm milk was added." Experimental/first-party MCP servers are exactly the kind that emit inconsistent error shapes, since they're not hardened against edge cases yet.

**How to avoid:**
Make "confirm success" structurally dependent on a verified post-condition, not on the add call merely returning: after every write, immediately do a read-back (get-cart) and confirm success only if the added item/quantity is actually present in the returned cart. Never phrase a success message directly from the add-call's return value alone. If the read-back is unavailable or ambiguous, report uncertainty explicitly ("I called the add, but couldn't confirm it landed — check your basket") rather than defaulting to a confident success message.

**Warning signs:**
- Success messages that are generated before or without a subsequent cart-read tool call in the transcript.
- Any UAT case where killing/mocking the MCP mid-call still produces a cheerful confirmation.
- User reports of "it said it added X but X isn't in my basket."

**Phase to address:**
Foundation/spine phase (the quick-add end-to-end milestone) — the read-after-write verification must be baked into the shared mutation primitive from the first working slice, not bolted on later.

---

### Pitfall 2: Non-idempotent writes — duplicate adds from not reading before writing

**What goes wrong:**
User says "add milk," agent resolves and adds it. Five minutes later, in a fresh chat (Claude Skills don't share memory across sessions by default), the same or other user says "did we add milk? add it if not," and the agent — with no read of current cart state — adds it again, doubling the quantity. Or: a retry after a perceived failure (see Pitfall 1) re-adds an item that actually succeeded the first time.

**Why it happens:**
Treating "add to cart" as a fire-and-forget append operation rather than a reconcile-to-desired-state operation. This is the single most common shared-cart bug pattern: mutation without a preceding read of current state, because reads feel like unnecessary overhead when the "obvious" thing to do is just add.

**How to avoid:**
Every basket-mutating skill must read the current cart before writing, and treat the add as a delta against what's already there (if item already present at quantity N and user wants N, no-op; if user wants more, add the difference; surface "you already have 2, want me to add more?" rather than blindly stacking). This read-before-write must be a property of the shared internal spine (the resolution/mutation core), not something each skill re-implements inconsistently.

**Warning signs:**
- Any add-flow in the plan that goes straight from "resolve item" to "call add" without an intervening cart-read step.
- Duplicate line items or doubled quantities appearing in manual UAT after two independent add requests for the same item.
- Basket-review skill surfacing near-duplicate entries (same product, split across two cart lines).

**Phase to address:**
Foundation/spine phase — read-before-write is a hard requirement already named in PROJECT.md ("Idempotent basket writes: read cart before adding, never duplicate"); it belongs in the shared mutation primitive built in the first milestone, verified with a "run the same add-request twice" UAT case.

---

### Pitfall 3: Silent MCP disappearance / schema drift with no fallback

**What goes wrong:**
The Rohlík MCP is explicitly declared experimental and personal-use-only, "subject to change or termination without notice." A tool gets renamed, a parameter's shape changes, a rate limit gets added, or the whole server goes down for a period. Without a designed fallback, every skill either throws an unhandled error the user can't act on, or worse, silently does nothing while claiming otherwise (compounding Pitfall 1).

**Why it happens:**
Teams build the happy path against the MCP as if it were a stable dependency, because that's the only version they've tested against, and graceful-degradation code paths are easy to skip when there's no visible failure yet to motivate them.

**How to avoid:**
Design every skill with an explicit degrade path from day one: if any Rohlík MCP call (search, read-cart, or add) fails or is unavailable, the skill falls back to producing a plain, well-formatted manual shopping list (with the resolved product names/quantities) that the user can act on by hand in the Rohlík app — and says so plainly ("Rohlík connection isn't working right now, here's your list to add manually"). Treat MCP tool-schema mismatches as a first-class error case to test, not just network failures. Since PROJECT.md flags this as a structural risk, the resolution cascade and the confirmation artifact should be MCP-agnostic in their internal representation (resolve to a structured item list first, touch the MCP only at the edges) so a degrade path is a natural fallback rather than a rewrite.

**Warning signs:**
- No test/UAT case exists that simulates the MCP being down, slow, or returning an unexpected shape.
- Any skill's logic assumes cart-read or add always succeeds (no try/fallback branch).
- No plain-list rendering path exists independent of MCP calls.

**Phase to address:**
Foundation/spine phase — must be validated as part of the first quick-add milestone (PROJECT.md explicitly lists this as a required behavior), with an explicit "MCP unavailable" UAT scenario before the milestone is considered done.

---

### Pitfall 4: Resolving "milk" (or any generic item) to the wrong concrete product

**What goes wrong:**
"Milk" resolves to whole-fat when the household drinks semi-skimmed; "bread" resolves to sliced white when they always buy rye; a vague recipe ingredient resolves to the first Rohlík search hit rather than the household's usual pick. The item lands in the basket looking "done" but is wrong, and because it's a low-attention purchase, nobody notices until it's delivered.

**Why it happens:**
Generic-name-to-SKU resolution is inherently ambiguous, and an LLM under turn-economy pressure (≤3-turn common path) is incentivized to just pick the top/most-generic search result rather than pause and disambiguate — especially when disambiguation looks like it would cost a turn.

**How to avoid:**
The resolution cascade must have a defined precedence: ruleset (explicit household preference, e.g. "milk = semi-skimmed") first, then learned favourites/purchase-history second, then — only if genuinely unresolvable — ask. Crucially, resolution should never silently pick "a" product when the ruleset or favourites already encode which specific product/brand/variant is correct; the whole point of the ruleset is to make this a zero-turn decision for known items. New/ambiguous items with no ruleset or favourite match should trigger disambiguation (a short pick-one prompt), not silent best-guessing. The confirmation step (see Pitfall 6) is the safety net: it must show *which specific product* was resolved (not just the generic name "milk") so the human catches a wrong resolution before the write happens.

**Warning signs:**
- Confirmation UI shows generic names ("milk", "bread") rather than the resolved product name/brand/size — this means the human literally cannot catch a bad resolution.
- No documented precedence order between ruleset, favourites, and free-text search fallback.
- UAT reveals the same input ("add milk") resolving differently across sessions (indicates it's falling through to raw search rather than a stable rule/favourite).

**Phase to address:**
Resolution cascade phase (the shared internal spine) — precedence order and "show the resolved product, not the request text" must be a spec requirement verified before any skill built on the cascade ships.

---

### Pitfall 5: Silent substitution of out-of-stock items

**What goes wrong:**
Requested item is out of stock; the agent — trying to be maximally helpful and complete the task in the fewest turns — substitutes a "close enough" alternative and adds *that* without flagging it as a substitution, or flags it so briefly/vaguely that the human doesn't register a decision was made on their behalf.

**Why it happens:**
Same turn-economy pressure as Pitfall 4, plus a helpfulness bias: an agent optimizing for "task completed" treats an unflagged substitution as a win rather than an unauthorized decision. This is explicitly called out in PROJECT.md as a required behavior ("Substitutions are proposed and approved, never silent"), meaning it's a known, anticipated failure mode for this exact class of system.

**How to avoid:**
Out-of-stock must be a distinct, structurally different branch from normal resolution: never auto-add a substitute. Always surface the substitution as an explicit proposal requiring the same confirm step as any other add ("Whole milk is out of stock — swap for semi-skimmed [same brand]?"), and if no substitute is confident/appropriate, fall back to reporting the gap rather than guessing. This should be enforced at the same layer as read-before-write and hard-cap enforcement — a property of the shared mutation spine, not a per-skill judgment call.

**Warning signs:**
- Any code/prompt path where "item out of stock" leads directly to "add alternative" without an intervening confirmation turn.
- Confirmation artifacts that don't visually distinguish "you asked for this" vs. "we're proposing this instead."

**Phase to address:**
Resolution cascade / mutation-safety phase — must be tested explicitly with a forced out-of-stock scenario during the first milestone's UAT.

---

### Pitfall 6: Favourites/learned preference silently overriding a hard constraint (allergy/dislike)

**What goes wrong:**
A learned favourite ("usually buys brand X peanut butter") or a purchase-history-based shortcut resolves an item to a product that conflicts with a hard rule in the ruleset (an allergy, an explicit dislike, a "never buy" entry) because the favourites layer was consulted without being filtered through the hard-constraint layer first, or because favourites were learned before the constraint existed and never reconciled.

**Why it happens:**
Favourites and hard constraints are conceptually two different data sources (one behavioral/statistical, one declarative/safety-critical) and it's easy to implement them as peers that get merged or as "favourites checked first because they're faster/more specific," inverting the safety-critical precedence.

**How to avoid:**
Hard constraints (allergy, explicit dislike) must be an filter that gates *every* candidate resolution — ruleset-derived, favourite-derived, or search-derived — before it's ever proposed, never something checked "if there's time" or only for fresh resolutions. Precedence must be: hard constraints filter first (eliminate disallowed candidates entirely), *then* ruleset/favourites pick among what's left. This should be encoded once in the shared resolution cascade so no individual skill can bypass it, and it should be part of the design of the ruleset itself (allergies/dislikes tagged as blocking, not just "preferences").

**Warning signs:**
- Ruleset schema doesn't distinguish "hard constraint" from "preference/favourite" as different types — if allergy and "prefers organic" live in the same undifferentiated list, precedence bugs are likely.
- No UAT case that plants a favourite conflicting with an allergy and checks the allergy wins.

**Phase to address:**
Ruleset design + resolution cascade phase — this is a data-modeling decision (constraint type as first-class field) that must be made before the cascade is built, since retrofitting precedence into an undifferentiated rule list is error-prone.

---

### Pitfall 7: Asking questions the ruleset already answers (turn-economy failure)

**What goes wrong:**
The agent asks "what kind of milk do you want?" when the household ruleset already states the default milk preference, burning a turn on a phone where the whole point is ≤3-turn completion. Repeated across a session, this makes the assistant feel dumber than a plain shopping list and erodes trust/adoption — worse than a wrong resolution, because it's a a visible, every-time failure rather than an occasional one.

**Why it happens:**
Conversational agents default to asking when uncertain, and if the ruleset lookup isn't wired as a hard first step before generating any clarifying question, the model will "play it safe" by asking — which feels correct locally but violates the product's core value prop (PROJECT.md: "under three turns... If everything else fails, the resolution cascade... must work").

**How to avoid:**
Structurally require a ruleset/favourites lookup to complete (and come back empty) before the agent is allowed to formulate a clarifying question. Treat "ask the user" as the last stage of the cascade, gated behind both ruleset and favourites lookups failing to resolve, not a parallel option the model can reach for whenever convenient. Test this explicitly: any item with a ruleset entry must never trigger a clarifying question in UAT, regardless of phrasing variation.

**Warning signs:**
- Clarifying questions appearing for items that do have a ruleset/favourite entry, when tested across paraphrases ("add milk" vs "we need milk" vs "milk please").
- Turn counts in UAT trending above 3 for the common path even when ruleset coverage is supposedly complete.

**Phase to address:**
Resolution cascade phase — turn economy is a named, hard requirement (PROJECT.md constraint) and should be a measured UAT criterion (turn count) for the quick-add milestone, not just a qualitative check.

---

### Pitfall 8: Mutating the shared basket without an explicit, unambiguous confirmation step

**What goes wrong:**
The agent adds items directly off its own resolution without a distinct human confirmation turn, or the "confirmation" is so implicit (e.g., buried in a longer message, or the agent proceeds unless the user objects within the same turn) that a mutation happens the human didn't knowingly approve. On a shared account, this is worse than a single-user mistake — the *other* household member may be the one who eventually notices an unwanted item, with no way to tell who or what caused it.

**Why it happens:**
Confirmation-before-mutation adds a turn, which conflicts with the turn-economy goal, creating pressure to skip or compress it. It's also simply easy to conflate "I've resolved what to add" with "the user has approved adding it" when both happen in the agent's own generation.

**How to avoid:**
Every basket write must be preceded by a distinct, explicit confirmation artifact presenting the final resolved list (specific products, quantities, running total) and requiring an affirmative response before any MCP write call is made. This can still be turn-economical (one confirmation covering the whole batch, not one per item) but it must never be skipped or merged into the same turn as the original request for anything beyond the most trivial cases the ruleset already fully specifies with no ambiguity. PROJECT.md already treats this as first-class ("Substitutions are proposed and approved, never silent"; the response-collector artifact decision) — the discipline is to also apply it to *ordinary*, non-substitution adds, not just the risky cases.

**Warning signs:**
- Any flow where the agent's confirmation message and the MCP add call happen in the same turn/generation without waiting on a distinct user response.
- UAT where a user says "add milk and eggs" and the assistant both resolves *and* writes to the basket before the user has seen the resolved list.

**Phase to address:**
Foundation/spine phase (quick-add milestone) — the confirmation-artifact-then-write sequence is the core mechanic of the milestone per PROJECT.md and must be verified end-to-end, including a "user declines/edits the confirmation" UAT path.

---

### Pitfall 9: The confirmation artifact itself calling the MCP (collapsing the safety boundary)

**What goes wrong:**
The confirmation artifact — meant to be a pure response-collector that shows the proposed list and waits for a yes/no/edit — is implemented (or drifts, over iteration) into something that itself has the ability to call the Rohlík MCP directly, e.g. because it's "more efficient" to let the artifact submit straight to the cart on button-press rather than round-tripping through the agent. This silently removes the human-in-the-loop boundary the whole design depends on: the artifact becomes an unaudited, unreviewed mutation path.

**Why it happens:**
Artifacts (interactive HTML/JS surfaces) are technically capable of making calls if given the hooks to do so, and "let the UI just do the write, it's faster" is a natural optimization under both turn-economy and engineering-convenience pressure. This is exactly the kind of architectural erosion that happens gradually across iterations rather than being introduced deliberately.

**How to avoid:**
Enforce as an explicit architectural invariant: the confirmation artifact's only job is to collect the human's response (confirm/edit/cancel) and hand it back to the agent conversation; the *agent*, not the artifact, is the only thing that ever calls the Rohlík MCP write endpoints, and it does so only after receiving the artifact's collected response in the conversation. This should be a stated constraint checked in code review for every skill that uses the confirmation artifact, not just a one-time design decision (PROJECT.md already names this: "Confirmation is a response-collector artifact that never calls the MCP" — the risk is drift away from this over time/iterations, not the initial build).

**Warning signs:**
- Any artifact code that imports/calls MCP client logic, holds Rohlík credentials/tokens, or has a "submit" button wired to anything other than returning a value to the conversation.
- Code review finding artifact-side network calls to Rohlík domains.

**Phase to address:**
Foundation/spine phase at initial build, plus a standing check in every subsequent phase that touches the confirmation artifact (regression risk, not a one-time gate) — worth calling out explicitly in a code-review checklist item used every phase the artifact is touched.

---

### Pitfall 10: Racing writes when both household members act on the shared basket concurrently

**What goes wrong:**
Both users have Claude open on their own phones (two devices, one shared account per PROJECT.md). One is mid-flow adding dinner ingredients while the other, in a separate chat, adds "milk and bananas." If both read the cart, compute a resolution, and write back based on a stale read, one write can clobber or fail to account for the other's concurrent change — e.g., a lost-update race where the second write's "current cart" snapshot didn't include the first's already-completed add, causing a wrong running-total display or (worse) an idempotency check that wrongly concludes an item isn't present yet and duplicates it (compounding Pitfall 2).

**Why it happens:**
The single-shared-basket design is a deliberate simplification (PROJECT.md's central assumption), but it inherits a distributed-systems problem — concurrent read-modify-write — while the design as described has no obvious session-to-session coordination (Claude Skills sessions per PROJECT.md don't share memory, and "no automation without a human present" argues against a lock/queue service). Because it's a two-person household, this reads as a low-probability edge case rather than a must-fix, so it's easy to deprioritize.

**How to avoid:**
Given the "no unattended automation" constraint, a full distributed lock is likely overkill; instead, make the pattern robust to staleness: always re-read the cart *immediately before* the actual write (not just at the start of the conversation) so the window for a stale-based decision is as small as possible, and treat the read-before-write step (Pitfall 2) as happening right before the write call rather than early in the flow. Since perfect prevention isn't achievable without infrastructure this project deliberately avoids, treat this as a residual, monitored risk: the basket-review skill (explicitly named in PROJECT.md as one of the skills built on the spine) should be positioned as the recovery mechanism — a quick way for either user to catch and fix a collision after the fact — rather than assuming collisions can't happen. Document this as a known accepted risk given the two-person low-frequency-concurrency scale, rather than silently ignoring it.

**Warning signs:**
- Design docs that treat "read cart" as a one-time step at conversation start rather than immediately pre-write.
- No basket-review/reconciliation skill in the roadmap, or it treated as low priority — it's the main recovery tool for this specific residual risk.

**Phase to address:**
Mutation-safety hardening within the foundation/spine phase (re-read-immediately-before-write as an implementation detail of the shared primitive); basket-review skill scheduled early enough to serve as the recovery net, not deferred indefinitely.

---

### Pitfall 11: Budget adds pushed past the hard cap, or soft/hard threshold confusion

**What goes wrong:**
Either the hard cap isn't actually enforced (it's advisory text the agent can talk its way past under user pressure — "just this once"), or the soft-warning and hard-block thresholds are conflated so a soft warning gets treated as blocking (annoying, erodes trust) or a hard cap gets treated as a mere warning (defeats the point of having a "hard" cap at all).

**Why it happens:**
LLM agents are conversationally compliant by default; if budget enforcement is implemented as "mention the limit in the prompt" rather than a structural gate the write path checks before calling the MCP, a sufficiently insistent user ("add it anyway") can talk the agent past it, because the agent has no non-conversational mechanism forcing refusal. Soft/hard conflation happens when both thresholds are implemented as the same code path with only a copy/message difference, so a bug or prompt drift can silently swap their behavior.

**How to avoid:**
Implement the hard cap as a structural precondition on the write step — compute the running total from the read-back cart before the actual add call, and refuse to make the write call (not just refuse to *recommend* it) if it would exceed the hard cap, regardless of what the user says in-conversation; the only way past a hard cap is the user changing the cap itself (a distinct, deliberate action), not asking nicely in the same flow. Soft threshold, by contrast, should visibly warn but still permit the write. These two behaviors should be implemented as clearly distinct branches (not the same function with a flag) precisely so they can't drift into each other. Every confirmation artifact should show the running total against both thresholds so the user always sees where they stand before approving.

**Warning signs:**
- Budget cap enforcement exists only as instruction text in a prompt/skill description rather than as a check the code path executes before the write call.
- UAT where a user insists ("add it anyway, I don't care about budget") and the agent complies past the hard cap.
- Confirmation artifact doesn't show a running total at all, or shows it without reference to the cap(s).

**Phase to address:**
Foundation/spine phase — the running-total-and-cap check belongs in the same shared mutation primitive as read-before-write and confirmation-gating, verified with an explicit "try to push past the hard cap" UAT case.

---

### Pitfall 12: Mobile-hostile output — long outputs, wide tables, walls of options, multi-screen confirmations

**What goes wrong:**
A confirmation or resolution response that's fine on a desktop chat window — a markdown table with columns for product/brand/size/price/quantity, or a bulleted list of 8 substitution options — becomes an unreadable wall of tiny wrapped text or a horizontally-scrolling table on a phone screen, directly violating the core mobile-only constraint and turning a 3-turn interaction into a frustrating scroll-and-squint exercise.

**Why it happens:**
It's natural for an LLM to default to "complete and thorough" formatting (tables, exhaustive option lists) because that's the generically helpful pattern from non-mobile contexts, and there's no automatic signal in-context reminding it that the render target is a narrow phone screen unless the skill/prompt design actively constrains output shape.

**How to avoid:**
Bake phone-first formatting constraints directly into the skill definitions and the confirmation artifact's design: short line-item lists (not tables), at most a small number of substitution/disambiguation options presented as a short pick-list rather than an exhaustive menu, and confirmation content sized to fit one scroll-free (or minimal-scroll) screen. Where an artifact is used for confirmation, design it mobile-first (single column, large tap targets, no horizontal scroll) rather than a shrunk-down desktop layout. Treat "does this fit on a phone screen without horizontal scroll or excessive vertical scroll" as a literal UAT check, done on an actual phone-sized viewport, not just "reads fine in the terminal/desktop chat."

**Warning signs:**
- Any skill's example output uses a markdown table with more than 2-3 columns.
- Confirmation artifacts tested only in a desktop browser/preview, never at a narrow mobile viewport.
- Disambiguation prompts listing more than ~4-5 options at once.

**Phase to address:**
UI/confirmation-artifact design phase and every skill-authoring phase thereafter — should be a stated design constraint checked at each skill's UAT (mobile viewport check), not just the initial artifact build.

---

### Pitfall 13: Trying to write learned state (favourites, staples, "learned" preferences) into read-only Project files

**What goes wrong:**
A skill tries to "remember" that the household now prefers a different milk brand, or that bananas should auto-restock weekly, by attempting to update the Project's knowledge files — which are read-only to the agent (explicitly stated in PROJECT.md). This either silently fails (the "learned" preference evaporates at the end of the session) or, worse, the agent *reports* success at "remembering" something it structurally cannot persist, misleading the user into thinking a preference is saved when it isn't (a variant of Pitfall 1's hallucinated-success pattern, applied to state rather than cart writes).

**Why it happens:**
"Remember this for next time" is a completely natural user request and a completely natural thing for an agent to attempt via whatever write-like tool looks available, without necessarily distinguishing "Project file" (read-only) from other persistence surfaces (artifact storage, Rohlík's own favourites/history) that actually support writes.

**How to avoid:**
Establish, at the architecture level, exactly one writable target for each category of learned state — per PROJECT.md's own tentative decision, lean on Rohlík's native favourites/order-history as source of truth where Rohlík supports it, and use artifact persistent storage only for what Rohlík genuinely can't hold — and make every skill that needs to "remember" something write only through that established path, never attempt a Project-file write. Any "remember X" user request should map deterministically to one of these two mechanisms, and the agent should never claim to have "saved" a preference without it having actually landed in one of them (apply the same read-back verification discipline as Pitfall 1, adapted to state writes).

**Warning signs:**
- Any skill design or prompt that references "update the project file" or "save this to the household ruleset" as an in-session, agent-driven action (Project files are meant to evolve at phase transitions by the human/GSD workflow, not by the shopping-skill agent at chat time).
- User reports that a stated preference didn't stick in a later session.

**Phase to address:**
Writable-state architecture phase (the "artifact-storage spike" already flagged as pending in PROJECT.md's Key Decisions) — must resolve *before* any "learning" skill (staples restock, favourites) is built, since it defines where such skills are structurally allowed to write.

---

### Pitfall 14: Artifact-storage write/shared-access assumptions that turn out false

**What goes wrong:**
The design leans on Claude artifact persistent storage as the writable-state layer for whatever Rohlík can't hold, assuming it (a) supports writes from a skill at chat time, and (b) is shared/consistent across the two devices logged into the one shared Claude account. If either assumption is wrong — e.g., storage turns out to be per-session, per-device, read-mostly, or has consistency lag — the "shared learned state" design silently doesn't work: one user's phone doesn't see the other's saved staple, or a save from Monday isn't visible by Wednesday.

**Why it happens:**
This is flagged as "Pending (artifact-storage spike)" in PROJECT.md itself — it's an explicitly unverified assumption baked into a Key Decision, which is exactly the kind of thing that's easy to build several skills on top of before discovering it doesn't hold, at which point the fix is expensive (redesign of the whole learned-state layer) rather than cheap.

**How to avoid:**
Run the artifact-storage spike explicitly and early — before any skill depends on it — and specifically test the two properties the design needs: (1) can a skill write to it from an ordinary chat interaction (not just read), and (2) is the same storage visible/consistent across the two devices/sessions sharing the one Claude account, including reasonable propagation delay. Treat this as a go/no-go gate: if either assumption fails, the writable-state architecture decision (Pitfall 13) needs to be revisited (e.g., leaning harder on Rohlík-native persistence, or another mechanism) before building staples/favourites-learning skills on top of it.

**Warning signs:**
- Staples/favourites-learning skills being planned or built before the artifact-storage spike has concluded.
- No test in the spike that specifically checks cross-device/cross-session visibility (as opposed to just "can I write and read back in the same session").

**Phase to address:**
Early — the artifact-storage spike should be resolved before or during the foundation/spine phase, definitely before any later milestone (staples restock, recipe-to-basket) that assumes it works.

---

### Pitfall 15: Favourites/staples drifting stale relative to actual current preference

**What goes wrong:**
A "favourite" or "staple" learned early on (e.g., a specific yogurt brand, a weekly milk order) keeps getting auto-applied long after the household's actual preference changed, because there's no mechanism to update or expire a learned favourite short of manually editing the ruleset. The assistant becomes confidently wrong in a way that's harder to notice than an outright error, since it's "usually right" and the drift is gradual.

**Why it happens:**
Learning systems that only ever add/reinforce (favourite chosen → reinforce) without any signal for "this is now wrong" or "this hasn't been chosen in N months" will monotonically ossify around whatever was learned first, especially in a low-frequency, two-person household context where there isn't a lot of data to naturally wash out an outdated favourite.

**How to avoid:**
Treat favourites as provisional defaults surfaced *visibly* at confirmation time (so staleness is always one glance away from being caught and corrected by a human), not as silent auto-resolutions indistinguishable from ruleset-mandated ones (see also Pitfall 4's "show the resolved product" requirement — this doubles as the anti-staleness mechanism). Consider a lightweight recency/frequency signal (e.g., deprioritize or flag a favourite that hasn't been reordered in a long time) if the basket-review or favourites-management skill has visibility into Rohlík's own order history, since Rohlík is the source of truth for that data per the Key Decisions table.

**Warning signs:**
- No UI path for a user to see "here's what I think your favourite X is" and correct it outside of a live add-flow.
- Favourites treated identically to hard ruleset entries in the confirmation display (undermining the "visible and correctable" property).

**Phase to address:**
Favourites/staples skill phase (later milestone, built on the spine) — the display-and-correct mechanism should be part of that skill's initial design, not an afterthought.

---

### Pitfall 16: Bilingual input mishandling — Czech/English mixed input breaking resolution

**What goes wrong:**
Users mix Czech and English freely ("add mléko and bananas," "potřebujeme milk na víkend"). A resolution/search pipeline that's only been tested against clean single-language input mishandles code-switched or Czech-inflected phrasing — e.g., failing to recognize "mlíko" (colloquial/misspelled) or a Czech grammatical case ending as "mléko" (milk, nominative), or treating an English item name as a literal search term against a Czech-language catalogue with no translation step, returning zero or wrong results.

**Why it happens:**
Bilingual, code-switched, colloquial input is genuinely harder than clean single-language input, and it's easy to build and test primarily against whichever language the developer defaults to in their own testing, only discovering gaps in the other language (or in mixed input) once real usage happens.

**How to avoid:**
Treat language handling as a normalization step *before* search/resolution: parse the user's request (in whatever language/mix) into a canonical item concept (e.g., "milk" as a concept, independent of language), then translate/map that concept to the Czech search term(s) actually likely to hit in the Rohlík catalogue — rather than passing the user's raw phrase straight to product search. Explicitly test mixed-language and Czech-inflected input (not just clean English and clean Czech separately) during the resolution-cascade build, including common colloquialisms and misspellings for staple items.

**Warning signs:**
- Resolution logic that passes user text directly to Rohlík product search without a language-normalization/translation step.
- UAT only covers clean single-language sentences, never a code-switched one.
- Search misses specifically on Czech grammatical case variants of common items (a strong sign the pipeline is doing literal string search, not concept resolution).

**Phase to address:**
Resolution cascade phase — canonical-concept-then-translate should be part of the cascade's core design, tested with explicit bilingual/mixed-input UAT cases.

---

### Pitfall 17: Product output drifting into English, or the wrong Czech register

**What goes wrong:**
Even when input handling works, the *output* — confirming what will be added — starts describing products in English ("Semi-skimmed milk, 1L") when the actual catalogue entry and the household's mental model is Czech ("Polotučné mléko, 1l"), creating a confirmation the user has to mentally re-translate to match against what they'll actually see in the Rohlík app/receipt — undermining trust that "what's confirmed" matches "what's real."

**Why it happens:**
The agent's default conversational language may follow the user's most recent input language (if they wrote in English) rather than being pinned to the catalogue's actual language, especially since Claude will naturally try to be helpful by mirroring the user's language choice — which is usually right for conversation, but wrong for product names that must match what's literally in Rohlík.

**How to avoid:**
Explicitly decouple conversational language (mirror the user, EN or CZ, whichever they used) from product-name rendering (always render product names, brands, and units exactly as they appear in the Rohlík catalogue, i.e., Czech) — this should be a hard formatting rule in the confirmation artifact and any resolution-echoing text, not left to the model's language-mirroring instinct. State this explicitly in the skill/prompt design ("conversational text follows the user's language; product names always render in Czech, verbatim from the catalogue") since it cuts against the model's default mirroring behavior.

**Warning signs:**
- Confirmation text showing translated/anglicized product names rather than the literal Czech catalogue string.
- Any user report of "the app shows something different from what Claude confirmed" that isn't attributable to a substitution or resolution error (i.e., it's a display/translation mismatch, not a resolution mismatch).

**Phase to address:**
Resolution cascade / confirmation-artifact phase — the "product names always in Czech, verbatim" rule should be specified alongside the cascade's output format, verified in UAT conducted partly in English.

---

### Pitfall 18: Diacritics and search-term mismatches against the Czech catalogue

**What goes wrong:**
Users on mobile keyboards frequently drop or mistype Czech diacritics (typing "mleko" instead of "mléko," "rohliky" instead of "rohlíky") — common on phone keyboards where diacritics require an extra tap/hold. If the search/resolution step does an exact or near-exact string match against the Rohlík catalogue (which presumably uses correct diacritics), a dropped-diacritic query may return no results, wrong results, or a lower-relevance match than intended, especially for words where the diacritic changes meaning (Czech has several near-homographs differing only by diacritics).

**Why it happens:**
Diacritic-insensitive matching isn't a given in every search backend, and it's tempting to assume "the search is provided by Rohlík's own MCP, so it must already handle this" without actually verifying it (the MCP being experimental means this can't be assumed reliable or tested).

**How to avoid:**
Explicitly test the actual Rohlík MCP search behavior against diacritic-dropped input early (part of understanding the MCP's real capabilities before building on it) rather than assuming it's handled. If the MCP's search is diacritic-sensitive or otherwise fragile to typos, add a normalization step (diacritic-insensitive matching, common typo/colloquialism mapping for frequent grocery terms) in front of it as part of the resolution cascade, rather than passing raw mistyped user text straight through.

**Warning signs:**
- No explicit test of the MCP's search tool with deliberately diacritic-stripped or misspelled Czech input during initial MCP exploration.
- Resolution failures correlating with diacritic-dropped queries specifically (as opposed to genuine catalogue gaps).

**Phase to address:**
MCP-capability-discovery step within the foundation phase (before/alongside the resolution cascade build) — this is exactly the kind of "assumption about an experimental dependency" that should be verified, not assumed, before the cascade is finalized.

---

### Pitfall 19: OAuth/credential handling leaking into Project files or skill definitions

**What goes wrong:**
Setting up the Rohlík MCP connector, or debugging it, results in a token, API key, or other credential-shaped string ending up pasted into a Project knowledge file, a skill definition, a troubleshooting note, or committed into whatever repo/config backs the project — even though the design explicitly calls for OAuth-only with no credentials in project files or skill definitions. Because Project files are meant to be relatively durable/shared context, a leaked credential there has a wide, long-lived blast radius (visible to anyone with Project access, indefinitely, until manually found and rotated).

**Why it happens:**
During setup/debugging, it's common to paste raw request/response examples (which may include tokens or auth headers) into notes for later reference, or to hardcode a token "just to get the connector working" with the intention of removing it later — a step that's easy to forget once things work.

**How to avoid:**
Treat "no credentials in Project files or skill definitions" as a literal, checked constraint: the OAuth connector setup happens once at the platform/connector level (Claude's custom-connector OAuth flow), never re-entered or referenced by value anywhere in Project markdown, skill instructions, or example transcripts kept for documentation. Any debugging notes or example MCP call/response logs saved into the Project should be scrubbed of tokens/headers before being written down, and a credential-scan pass (search for token-shaped strings) should be run over Project files before considering the connector setup phase complete.

**Warning signs:**
- Any Project file, skill definition, or committed doc containing a bearer-token-shaped string, an OAuth `access_token`/`refresh_token` field, or a raw HTTP header dump from MCP debugging.
- Setup instructions that say "add your token to X" rather than "connect via OAuth" as the only credential-entry path.

**Phase to address:**
Setup/connector-configuration phase (very first phase, before any skill work) — should be a one-time explicit check ("grep Project files for anything token-shaped") at the end of that phase, and worth a quick recheck at any later phase where MCP debugging notes get added to the Project.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|-----------------|------------------|
| Skip the cart read-back after add, trust the add call's return value | Faster to build the first working add-flow | Hallucinated-success bugs (Pitfall 1) surface exactly when the MCP is flaky — which is often, given experimental status | Never — this is a named hard requirement, not a nice-to-have |
| Hardcode a starter ruleset with only a couple of items (milk, bread) rather than modeling constraint-vs-preference types | Faster to demo quick-add end-to-end | Retrofitting hard-constraint precedence (Pitfall 6) into an undifferentiated rule list later is error-prone and risks an allergy slipping through during the transition | Acceptable only for the very first internal spike, before any real allergy/dislike data is entered — must be fixed before the household's real ruleset (with real allergies) is loaded |
| Let the confirmation artifact call the MCP directly "for now" to move faster on the first prototype | Simpler wiring, fewer round-trips to build initially | Normalizes the exact architecture violation (Pitfall 9) that removes the human-in-the-loop safety boundary; hard to notice once it "works" | Never acceptable beyond a disposable local prototype never exposed to the real shared account |
| Skip bilingual/diacritic testing, ship English-only tested resolution first | Faster path to a working demo in the developer's preferred language | Silent resolution failures for the actual bilingual household (Pitfall 16, 18) once real usage starts — the whole point of the project is bilingual support | Acceptable for a throwaway spike only; must be closed out before the quick-add milestone is considered done, since bilingual support is a named Active requirement |
| Treat artifact-storage as definitely writable/shared without running the spike first | Lets favourites/staples-skill work start immediately | Expensive rework if the assumption (Pitfall 14) turns out false after multiple skills are built on it | Never — PROJECT.md itself marks this "Pending," so building on it before resolving is building on a known-unverified foundation |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|-----------------|-------------------|
| Rohlík MCP (experimental, personal-use) | Assuming tool names/parameter shapes are stable across sessions/updates since it "worked yesterday" | Treat every MCP call site as needing a defensive/degrade branch; re-verify tool behavior periodically rather than assuming permanence, given the explicit "may change without notice" status |
| Rohlík MCP cart operations | Calling add without a preceding or immediately-following cart read, assuming add is idempotent by item ID | Always read-before-write and read-back-after-write; never assume the MCP itself deduplicates or reports true state reliably |
| Rohlík MCP search | Passing raw user text (possibly English, possibly diacritic-dropped) directly as the search query | Normalize to a canonical concept, translate to Czech, and only then query search; verify diacritic-handling behavior empirically rather than assuming |
| Claude custom connector OAuth | Treating one-time OAuth setup as "done forever" without a plan for re-auth if the connector's OAuth flow changes (again, experimental server) | Document (outside of Project files, to avoid Pitfall 19) that OAuth may need periodic re-establishment, and design skills to surface a clear "reconnect needed" message rather than a raw auth error |
| Claude artifact persistent storage | Assuming write support and cross-device/cross-session shared visibility without testing | Run the spike explicitly (Pitfall 14) before depending on it for any shared learned state |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|-----------------|
| Re-reading the full cart and re-running full resolution on every micro-turn of a multi-item conversation | Slower, more turn-heavy interactions than the ≤3-turn target; more MCP calls than necessary | Batch resolution for all items mentioned in a single user turn before doing a single cart read/write cycle, rather than one read/write per item | Noticeable as soon as a single "add milk, eggs, and bread" request takes visibly longer or more turns than a single-item add — should be caught in early UAT, not at scale |
| Long confirmation lists that grow unbounded as more items are requested in one turn (e.g., a full week's meal-plan grocery list) | Confirmation artifact becomes a long scroll on mobile, violating the mobile-UX constraint | Cap/paginate or group confirmation display (e.g., by category) once item count exceeds a small threshold, rather than rendering an ever-growing flat list | Breaks as soon as a meal-plan-to-basket or weekly-restock skill (later milestones) generates double-digit item counts in one go — worth designing for before those skills are built, not just for quick-add's typical 1-3 items |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Credentials/tokens pasted into Project files, skill definitions, or saved debugging transcripts | Long-lived, widely-visible credential exposure since Project files are durable shared context | OAuth-only, never store token values in any Project artifact; scrub debugging notes before saving them (Pitfall 19) |
| Confirmation artifact given any capability beyond collecting a response (e.g., holding a token to call the MCP itself) | Removes the human-approval boundary the whole mutation-safety design depends on; an artifact bug or drift becomes an unaudited basket-write path | Keep the artifact strictly a response-collector; all MCP calls happen only from the agent side after reading the artifact's returned response (Pitfall 9) |
| Treating the shared single Rohlík account/single Claude account as needing no per-user distinction anywhere | Not itself a classic "security" bug, but any future audit/attribution feature ("who added this?") is impossible to build after the fact if no per-turn user signal is ever captured, and a compromised device has full basket-write and order-history-read access with no differentiation | If low-cost, capture which conversation/device a mutation came from at write time (e.g., in the audit trail already required by PROJECT.md: "what, how many, cost, and which rule/favourite matched") even though full user-level auth isn't in scope, so at least post-hoc attribution is possible |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-------------------|
| Confirmation shows generic request text ("milk") instead of the specific resolved product | User can't actually catch a wrong resolution before it's added — the confirmation step becomes rubber-stamping rather than real review | Always show the specific product name/brand/size/price as it will appear in the real cart (Pitfall 4) |
| Wide/table-heavy confirmation or option layouts | Horizontal scrolling, tiny wrapped text, frustration on a phone — directly against the mobile-only constraint | Short vertical line-item lists, no tables with more than 2-3 columns, capped option counts (Pitfall 12) |
| Asking a clarifying question the ruleset already answers | Feels dumber than a manual list, burns turns, erodes trust in the ≤3-turn value prop | Gate clarifying questions behind a completed ruleset/favourites lookup that came back empty (Pitfall 7) |
| Silent substitution or silent favourite-override of a stated dislike/allergy | User loses trust entirely once they discover an unwanted or unsafe item arrived without their knowledge — this is the single most trust-destroying failure mode for a shared household tool | Every substitution and every constraint-adjacent resolution is visible and explicitly confirmed, never silent (Pitfall 5, 6) |
| No visible running total against budget thresholds | User has no chance to self-correct before a cap is hit, and can't tell soft warnings from hard blocks | Always show running total plus both thresholds at confirmation time (Pitfall 11) |

## "Looks Done But Isn't" Checklist

- [ ] **Add-to-cart flow:** Often missing the mandatory read-back-after-write — verify by killing/mocking the MCP mid-call and confirming the agent reports uncertainty, not success.
- [ ] **Resolution cascade:** Often missing hard-constraint-first precedence — verify by planting a favourite that conflicts with a declared allergy and confirming the allergy wins every time.
- [ ] **Confirmation artifact:** Often drifts toward calling the MCP directly for "efficiency" — verify by code-reviewing the artifact's code for any MCP client capability or credential access, on every phase that touches it.
- [ ] **Budget cap:** Often implemented as prompt text rather than a structural gate — verify by explicitly instructing the agent, mid-conversation, to "add it anyway" past the hard cap and confirming the write is actually refused.
- [ ] **Bilingual support:** Often tested only in clean single-language sentences — verify with genuinely mixed Czech/English input and Czech grammatical-case/diacritic variants, not just parallel clean-EN/clean-CZ test pairs.
- [ ] **Mobile formatting:** Often only checked in a desktop preview — verify every confirmation/disambiguation screen at an actual narrow mobile viewport, checking for horizontal scroll and vertical length.
- [ ] **Writable learned state:** Often assumes artifact storage "just works" for writes and cross-device sharing — verify the spike actually tested both properties (write-from-chat, and visible-from-the-other-device) before any favourites/staples skill ships.
- [ ] **Idempotency:** Often only tested for a single add in a single session — verify by issuing the same add request twice, in two separate fresh chats, and confirming no duplicate.

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|-----------------|
| Duplicate items from a missed read-before-write | LOW | Basket-review skill surfaces duplicate lines; user removes the extra via a corrective skill/manual app edit |
| Silent substitution or wrong resolution already added | LOW–MEDIUM | Basket-review skill flags recently-added items against the ruleset/request; user removes/corrects via the app or a follow-up "remove X" turn |
| Hallucinated success discovered after the fact (item never actually added) | LOW | Basket-review or a "verify last add" skill re-reads the cart and reports the true state; user re-issues the add if genuinely missing |
| Confirmation artifact found to have MCP-calling capability (architecture violation caught late) | HIGH | Requires a code audit of every skill using the artifact, a redesign of the artifact to strip the capability, and re-verification that all writes now route through the agent only — treat as a stop-the-line issue, not a minor fix |
| Artifact-storage assumption found false after favourites/staples skills already built on it | HIGH | Requires redesigning the writable-state layer (likely leaning harder on Rohlík-native persistence) and migrating/rebuilding the dependent skills — this is why the spike (Pitfall 14) should run before, not after, that work |
| Allergy/dislike violation actually reaches the shared basket | MEDIUM | Immediate manual removal via the app; retroactively audit the ruleset-precedence logic (Pitfall 6) to find and close the gap that let it through, since this is the single most safety-critical failure in the whole system |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|-------------------|---------------|
| Hallucinated success (P1) | Foundation/spine (quick-add milestone) | UAT: mock/kill MCP mid-add, confirm agent reports uncertainty not success |
| Non-idempotent duplicate writes (P2) | Foundation/spine (quick-add milestone) | UAT: issue the same add twice in separate fresh chats, confirm no duplicate |
| Silent MCP disappearance / no fallback (P3) | Foundation/spine (quick-add milestone) | UAT: MCP-unavailable scenario produces a usable plain list, not silence or a hard error |
| Wrong-kind resolution (P4) | Resolution cascade phase | UAT: confirmation shows specific resolved product, not generic request text |
| Silent substitution (P5) | Resolution cascade / mutation-safety phase | UAT: force an out-of-stock item, confirm substitution is proposed and requires confirm, never auto-added |
| Favourite overriding hard constraint (P6) | Ruleset design + resolution cascade phase | UAT: plant a favourite conflicting with a declared allergy, confirm allergy always wins |
| Asking what ruleset already answers (P7) | Resolution cascade phase | UAT: measure turn count across paraphrases for ruleset-covered items; must stay ≤3 |
| Mutation without explicit confirmation (P8) | Foundation/spine (quick-add milestone) | UAT: confirm agent never calls MCP write in the same turn as initial resolution, always waits for a distinct confirm response |
| Confirmation artifact calling MCP directly (P9) | Foundation/spine build + every phase touching the artifact | Code review checklist item: artifact has no MCP-client capability or credentials, every phase it's touched |
| Concurrent/racing writes (P10) | Mutation-safety hardening within foundation/spine; basket-review skill scheduled early | UAT: simulate near-simultaneous adds from two sessions, confirm no silent loss/duplication; basket-review present as recovery net |
| Budget cap bypass / soft-hard confusion (P11) | Foundation/spine (quick-add milestone) | UAT: instruct agent to "add anyway" past hard cap mid-conversation, confirm write is refused; confirm soft threshold warns but permits |
| Mobile-hostile output (P12) | Confirmation-artifact/UI design phase + every skill-authoring phase | UAT: every confirmation/disambiguation screen checked at a narrow mobile viewport, no horizontal scroll |
| Writing learned state to read-only Project files (P13) | Writable-state architecture phase (artifact-storage spike) | Verify no skill attempts a Project-file write; "remember X" requests map deterministically to Rohlík-native or artifact storage |
| Artifact-storage write/shared-access assumptions (P14) | Early — before/during foundation/spine, before any learning skill | Spike explicitly tests write-from-chat and cross-device visibility before being relied upon |
| Favourites/staples drifting stale (P15) | Favourites/staples skill phase (later milestone) | UAT: favourites are visibly shown and correctable at confirmation time, not silently auto-applied indefinitely |
| Bilingual input mishandling (P16) | Resolution cascade phase | UAT: mixed Czech/English and grammatical-case-variant input resolves correctly, not just clean single-language input |
| Product output drifting from Czech (P17) | Resolution cascade / confirmation-artifact phase | UAT conducted partly in English confirms product names still render in Czech, verbatim from catalogue |
| Diacritic/search-term mismatches (P18) | MCP-capability-discovery step, foundation phase | Explicit test of MCP search with diacritic-dropped/misspelled Czech input before finalizing the cascade |
| Credential leakage (P19) | Setup/connector-configuration phase (first phase) | Credential-scan pass over Project files and skill definitions at end of setup phase, and after any MCP-debugging notes are added later |

## Sources

- Project's own stated constraints and Key Decisions — /home/user/grocery-shopper/.planning/PROJECT.md (the MCP's declared experimental/personal-use/no-notice-change status, the read-only-Project-files constraint, the pending artifact-storage spike, and several requirements — idempotent writes, non-silent substitutions, response-collector-only confirmation artifact — are already named there as anticipated risks, corroborating this research rather than being novel discoveries)
- General agentic tool-use failure patterns: LLM agents defaulting to confident success narration after ambiguous/failed tool calls; conversational compliance overriding stated hard limits under user pressure
- Shared-cart / multi-user commerce UX precedent: lost-update races on concurrent cart modification; unflagged substitution as a common trust-destroying failure in grocery-delivery apps generally
- Conversational-commerce mobile UX precedent: table-heavy and multi-option outputs as a known-poor fit for narrow mobile chat surfaces
- Bilingual NLP/search precedent: literal string search against diacritic-bearing catalogues failing on diacritic-dropped or colloquial input; code-switched input being under-tested relative to single-language input in typical dev/test cycles

---
*Pitfalls research for: phone-first household grocery assistant (Claude Skills + Rohlík MCP)*
*Researched: 2026-07-31*
