# Project Go-Live Setup Checklist

**Implements:** FOUND-04 (pre-flight for the MCP round-trip) · CLAUDE.md §6 (OAuth-only, credential-scan)

**Who runs this:** A human, inside the real claude.ai product, on a phone signed into the shared
household Claude account. **This checklist cannot be completed from this repo or any CI process** —
it requires the actual claude.ai UI (Settings, Project pages, connector OAuth flow).

**When:** Once, before the live MCP round-trip in `mcp-round-trip-protocol.md`, and again any time
the connector or Project Knowledge set needs to be re-established (e.g. after a disconnect).

Work through the items in order. Each has an explicit done-state — tick it only when that exact
state is observed, not "probably fine."

---

## 1. Confirm the plan tier supports what this project needs

- [ ] **Done-state:** the claude.ai account (the one shared household account) is on a plan tier
      that supports **custom Skills** (uploading a SKILL.md zip via Settings → Features) **and**
      **publishable Artifacts** (Pro / Max / Team / Enterprise — code execution / Artifacts feature
      enabled).
- **How to check:** Settings → Features (or Settings → Capabilities) in claude.ai. Look for an
  explicit toggle or mention of custom Skills and of Artifacts/code execution being available on
  the current plan.
- **If not supported:** this is a hard blocker — no workaround exists at a lower tier. Upgrade the
  plan before continuing, or stop here and record the blocker.

## 2. Upload the eight `project-knowledge/` docs as Project Knowledge files

- [ ] **Done-state:** all eight files below appear in the shared Project's **Knowledge** section,
      each uploaded successfully (no upload error, correct file shown in the list):
  - `household-ruleset.md`
  - `budget.md`
  - `seed-favourites.md`
  - `resolution-cascade.md`
  - `confirmation-protocol.md`
  - `substitution-policy.md`
  - `mcp-degradation.md`
  - `audit-format.md`
- **How to do it:** claude.ai → open the shared Project → **Knowledge** tab → Add/Upload files →
  select each of the eight files from this repo's `project-knowledge/` folder → confirm each shows
  up in the Knowledge file list afterward.
- **Note:** these files are **read-only to the agent** once uploaded — any future edit must happen
  in this repo and be **re-uploaded** (replacing the old copy), never edited live in chat.

## 3. Attach the Rohlík MCP custom connector via OAuth, at "This project" scope

- [ ] **Done-state:** the Rohlík MCP connector (`https://mcp.rohlik.cz/mcp`) is attached to the
      shared Project and its status reads **"Connected"** (not "Needs approval", not "Disconnected",
      not "Error").
- **How to do it:** claude.ai → the shared Project → **Connectors** (or Settings → Connectors) →
  Add connector → enter/select the Rohlík MCP custom connector URL → complete the OAuth flow in the
  browser (log in with the shared Rohlík account credentials, grant consent) → back in claude.ai,
  confirm the connector card shows **Connected**.
- **Scope:** attach at **"This project"** scope specifically (not "Global", not "Once" /
  "This conversation" only) — this keeps the connector's blast radius to the one Project this
  household actually uses it in, per CLAUDE.md's connector-scoping guidance, while still meaning it
  doesn't need re-approval every chat within this Project.
- **If OAuth fails or the connector won't reach "Connected":** do not retry credential entry
  repeatedly hoping it resolves. Record the exact failure message and treat the round-trip as
  blocked (see `mcp-round-trip-protocol.md`'s note on recording a blocked outcome) rather than
  guessing at a workaround.

## 4. Credential-scan reminder

- [ ] **Done-state:** a manual re-read of everything just typed/pasted into claude.ai (connector
      setup fields, any chat message during setup) and everything about to be committed to this
      repo (results template, `mcp-degradation.md`) confirms **no OAuth token, client secret, bearer
      header, or other credential-shaped string** was pasted anywhere — not into a Project Knowledge
      file, not into a chat message, not into this repo, "not even temporarily to get it working."
- **How to check:** the OAuth flow itself happens entirely inside the browser's redirect to
  Rohlík's own login page — claude.ai's connector UI should never ask you to paste a token directly.
  If at any point a field asks for a raw token/secret/API key to be typed in, stop and treat that as
  a red flag rather than proceeding (OAuth-only means the standard flow never needs this).
- **Repeat this scan** after any session where MCP debugging notes get added anywhere (chat,
  Project Knowledge, or this repo).

---

## Summary — done-state for this whole checklist

All four items ticked means: the account tier is confirmed adequate, all eight shared docs are live
in Project Knowledge, the Rohlík MCP connector reads Connected at "This project" scope, and a
credential scan found nothing token-shaped anywhere. Only once all four are true should
`mcp-round-trip-protocol.md` be run.

**If any item cannot be completed:** stop and record which item blocked, and why, instead of
proceeding past it. A blocked setup step is a valid, recordable outcome — do not skip ahead to the
round-trip protocol with an unconnected connector or a missing doc.
