# Requirements: Household Grocery Assistant (Rohlík)

**Defined:** 2026-07-31
**Core Value:** From "add milk and bananas" to the *right kind* of items in the shared Rohlík basket in ≤3 turns, on a phone, without opening a computer.

## v1 Requirements

Requirements for initial release. Each maps to a roadmap phase (see Traceability).

### Foundation & Shared Internals

- [ ] **FOUND-01**: A single resolution-cascade document is the one source of truth every shopping skill references (never reimplemented per skill)
- [ ] **FOUND-02**: A single confirmation-protocol document defines the interactive confirmation-artifact contract used by every mutating skill
- [ ] **FOUND-03**: A single substitution-policy document governs all out-of-stock / unavailable handling
- [ ] **FOUND-04**: A single MCP-degradation policy defines fallback behaviour for every Rohlík MCP call
- [ ] **FOUND-05**: A single audit-format spec defines what every basket add reports back
- [ ] **FOUND-06**: The household ruleset (quality rules, dislikes, allergies, brand prefs) is read at the start of every resolution
- [ ] **FOUND-07**: The budget config (soft threshold, hard cap) is loaded and applied on every confirmation
- [ ] **FOUND-08**: A seed-favourites file provides hand-editable pre-approved product IDs
- [ ] **FOUND-09**: An artifact-storage spike determines whether agent-writable state persists across devices (go/no-go gate for learned-state features)

### Resolution Cascade

- [ ] **CASC-01**: An abstract item resolves to a concrete Rohlík product via ruleset → favourites → ask, in that order
- [ ] **CASC-02**: Hard constraints (allergies, dislikes, "we don't buy X") remove candidates and are never proposed or substituted
- [ ] **CASC-03**: Ruleset preferences (BIO, farm-sourced, brand, pack size) rank candidates by declared order
- [ ] **CASC-04**: A favourite is used only if it satisfies every hard constraint — the ruleset always outranks a favourite
- [ ] **CASC-05**: The user is asked only when ruleset and favourites both fail to produce a confident match
- [ ] **CASC-06**: The system never asks a question the ruleset already answers (turn economy)

### Confirmation & Mutation Safety

- [ ] **CONF-01**: Every proposed shop is presented as an interactive artifact with per-item tickboxes and a quantity stepper
- [ ] **CONF-02**: The confirmation artifact is a response collector only and never calls the MCP
- [ ] **CONF-03**: The agent reads back the final list and verbally confirms before any basket write
- [ ] **CONF-04**: No basket mutation occurs without explicit user approval
- [ ] **CONF-05**: The cart is read before adding; items already present are not duplicated (idempotency)

### Budget

- [ ] **BUDG-01**: A running basket total is shown on every confirmation
- [ ] **BUDG-02**: Crossing the soft threshold produces a warning
- [ ] **BUDG-03**: Additions that would exceed the hard cap are refused; the agent asks the user to cut items or raise the cap

### Substitution

- [ ] **SUBS-01**: Out-of-stock / unavailable items are substituted per the ruleset, then approval is requested — never silently swapped

### Reliability & Degradation

- [ ] **DEGR-01**: If the Rohlík MCP is unavailable or changed, the skill degrades to a plain shopping list the user can enter manually
- [ ] **DEGR-02**: A basket add is never reported successful unless a cart read confirms it (no hallucinated success)

### Auditability

- [ ] **AUDT-01**: Every add reports what item, how many, what it cost, and which rule or favourite matched

### Bilingual & Mobile UX

- [ ] **UX-01**: Users can write requests in Czech and English interchangeably
- [ ] **UX-02**: Product output is in Czech (matches the catalogue)
- [ ] **UX-03**: Replies are short and phone-friendly — no wide tables, no walls of options; the confirmation is usable one-handed
- [ ] **UX-04**: The common quick-add path completes in ≤3 turns

### Quick-Add

- [ ] **QADD-01**: "add milk and bananas" triggers quick-add and resolves each item through the cascade
- [ ] **QADD-02**: Quick-add pushes approved items to the shared Rohlík basket and returns the audit report

### Basket-Review

- [ ] **REVW-01**: "what's in the basket" reads the live cart and shows items with a running total
- [ ] **REVW-02**: Basket-review reports budget-threshold state and hands off to manual checkout

### Staples-Restock

- [ ] **STPL-01**: Staples are learned from Rohlík order history (frequency + recency) and are hand-editable
- [ ] **STPL-02**: Restock proposes staples for approval and never auto-adds

### Recipe-to-Basket

- [ ] **RCPE-01**: A recipe (name, URL, or uploaded PDF/ebook) from a trusted source is turned into an ingredient list
- [ ] **RCPE-02**: Recipe ingredients are resolved to Rohlík products via the same cascade and confirmed before adding
- [ ] **RCPE-03**: Trusted recipe sources are a curated, user-editable list

### Meal-Plan

- [ ] **MEAL-01**: "plan next week" builds N meals from trusted sources into a merged shopping list
- [ ] **MEAL-02**: Meal-plan deduplicates ingredients across recipes and accounts for pack sizes (no 4 bunches of parsley for 4 recipes)

### Household-Prefs

- [ ] **PREF-01**: "we don't buy X anymore" updates the household ruleset / favourites / staples / sources
- [ ] **PREF-02**: Preference changes that must persist are routed to writable state (Rohlík-native or artifact storage) or surfaced as a hand-editable diff to the read-only Project files

## v2 Requirements

Deferred to a future release. Tracked but not in the current roadmap.

### Learning & Personalisation

- **LEARN-01**: Agent auto-writes learned favourites back to persistent storage (gated on the FOUND-09 spike outcome)
- **LEARN-02**: Saved/extracted recipe repertoire persists across sessions
- **LEARN-03**: Rohlík deals/promotions are surfaced during restock and review
- **LEARN-04**: Rohlík personalised recommendations feed the cascade beyond hand-seeded favourites

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Checkout, payment, delivery-slot selection | Platform-enforced manual step; not exposed via MCP; a deliberate safety boundary |
| Multi-retailer support | v1 is Rohlík-only; the whole design assumes one shared Rohlík basket |
| Nutrition / calorie / macro tracking | Not the product's job |
| Pantry / inventory tracking (what's in the fridge) | Out of scope for v1; partly covered later by learned staples |
| Any automation without a human in the chat | Every mutation requires a person present and approving — architecturally foreclosed |

## Traceability

Populated during roadmap creation (granularity: coarse). Each requirement maps to exactly one phase.

| Requirement | Phase | Status |
|-------------|-------|--------|
| FOUND-01…09 | TBD | Pending |
| CASC-01…06 | TBD | Pending |
| CONF-01…05 | TBD | Pending |
| BUDG-01…03 | TBD | Pending |
| SUBS-01 | TBD | Pending |
| DEGR-01…02 | TBD | Pending |
| AUDT-01 | TBD | Pending |
| UX-01…04 | TBD | Pending |
| QADD-01…02 | TBD | Pending |
| REVW-01…02 | TBD | Pending |
| STPL-01…02 | TBD | Pending |
| RCPE-01…03 | TBD | Pending |
| MEAL-01…02 | TBD | Pending |
| PREF-01…02 | TBD | Pending |

**Coverage:**
- v1 requirements: 44 total
- Mapped to phases: 0 (roadmapper to populate)
- Unmapped: 44 ⚠️

---
*Requirements defined: 2026-07-31*
*Last updated: 2026-07-31 after initial definition*
