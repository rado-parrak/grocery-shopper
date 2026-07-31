# Skills — Packaging & Upload Guide

This directory (`skills/`) holds the **version-controlled sources** for this project's Claude
Skills: `quick-add` and `basket-review`, each a `SKILL.md` plus its own one-level-deep reference
files. This README explains how those sources get from this git repo into a running skill on
claude.ai — the authoring→runtime flow decided in `.planning/phases/02-quick-add-basket-review/02-CONTEXT.md`
(D-01, D-02).

## Where these skills actually run

**These skills run on claude.ai, not in this repo.** The runtime is a shared Claude Project on the
household's one shared Claude account (`.claude/CLAUDE.md` §Constraints — Accounts), not Claude
Code, not any CI, and not this git checkout.

## ⚠ NOT for `.claude/skills/`

**Do not copy or symlink anything from `skills/<name>/` into `.claude/skills/` in this repo.**
`.claude/skills/` is Claude Code's own execution surface, and it is already occupied by the GSD
workflow tooling that plans and executes this project's phases. Placing a product skill there
would:

- wrongly activate `quick-add`/`basket-review` inside this repo's own Claude Code tooling sessions
  (where there is no Rohlík MCP connector, no household ruleset context loaded the way claude.ai
  loads Project Knowledge, and no shared-basket runtime at all), and
- pollute the zip you upload to claude.ai with GSD's own files.

The claude.ai Settings → Features upload (below) is the **only** intended runtime for these two
skills. See `.claude/CLAUDE.md`'s "Version / Surface Compatibility Notes" table: claude.ai custom
Skills and Claude Code's filesystem-based skills are two separate formats/surfaces that do not
sync with each other.

## Upload flow: repo → zip → claude.ai Project

1. **Pick one skill directory** to package at a time — `skills/quick-add/` or
   `skills/basket-review/`. Each is uploaded as its own separate zip; they are not bundled together.
2. **Zip the directory's contents** (the `SKILL.md` and any reference files inside it, e.g.
   `skills/quick-add/resolution-notes.md`) — zip the *contents* of the skill folder, not a
   zip-of-a-zip or a zip containing an unrelated top-level wrapper folder.
   ```bash
   cd skills/quick-add && zip -r ../quick-add.zip . && cd -
   cd skills/basket-review && zip -r ../basket-review.zip . && cd -
   ```
3. **Upload in claude.ai:** open the shared Project → **Settings → Features** → add a custom Skill
   → upload the zip. Repeat for the second skill.
4. **Re-upload to update:** claude.ai custom Skills are edited by re-zipping and re-uploading the
   whole skill after any change to its `SKILL.md` or reference files in this repo — there is no
   live sync from git to claude.ai. Treat a merged change in this repo as "pending" until someone
   re-uploads the corresponding zip.

## Project Knowledge: the eight shared contracts (uploaded separately)

The `project-knowledge/` docs (`resolution-cascade.md`, `confirmation-protocol.md`,
`substitution-policy.md`, `mcp-degradation.md`, `audit-format.md`, `household-ruleset.md`,
`budget.md`, `seed-favourites.md`, plus `writable-state-decision.md`) are **not** part of either
skill's zip. Upload each of these files individually as **Project Knowledge** in the same claude.ai
Project (Settings → Project Knowledge, or the Project's own file-upload area). Both skills
reference these files by name at chat time and expect them to be present in Project Knowledge —
without them, the skills have nothing to resolve against.

`household-ruleset.md` and `budget.md` are hand-edited, human-owned files: fill their `⚠ FILL`
placeholders (allergies, dislikes, brand/milk preferences, soft/hard CZK budget amounts) directly
in this repo before uploading, and re-upload the file to Project Knowledge whenever a household
member edits it here. No skill ever writes back to these two files.

## Rohlík MCP connector

Grant the Rohlík MCP connector at **This-project** scope (not Global, not per-conversation) inside
the same claude.ai Project, so its blast radius stays limited to this one household tool and it
doesn't need re-approving every chat (`.claude/CLAUDE.md` Architecture Pattern 2).

## OAuth-only — never a token in a skill file

The Rohlík MCP connector is authenticated through claude.ai's own OAuth connector flow, set up
once. **Never paste an OAuth token, client secret, or raw auth header into any `SKILL.md`, any
`skills/evals/*.md` file, this README, or any Project Knowledge file — not even "temporarily, to
get it working."** Skill files and Project Knowledge are durable, widely-visible context with no
secret-scoping of their own; a leaked credential there has a long-lived, wide blast radius until
manually found and rotated. If the connection ever needs re-establishing (token expiry, a
server-side change — expected, given the connector's declared experimental status), re-run the
OAuth flow in claude.ai's connector settings; do not work around it by hardcoding a credential
anywhere in this repo or in claude.ai.

## Summary checklist

- [ ] Zip `skills/quick-add/` and `skills/basket-review/` separately, upload each via
      **Settings → Features** in the shared claude.ai Project.
- [ ] Upload all `project-knowledge/*.md` files as Project Knowledge in the same Project.
- [ ] Grant the Rohlík MCP connector at This-project scope.
- [ ] Fill `household-ruleset.md` / `budget.md` `⚠ FILL` placeholders before the first real shop,
      and re-upload after any edit.
- [ ] Never place these skills in `.claude/skills/`, and never paste a credential into any skill or
      Project Knowledge file.
