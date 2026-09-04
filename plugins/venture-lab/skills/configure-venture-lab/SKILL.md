---
name: configure-venture-lab
description: Point the Venture Lab plugin at the current user's own Notion workspace — confirm the connected Notion account, create (or record) their 🎧 Sources inbox database, and write the per-user config the capture skill reads. Load when someone says "configure venture lab", "set up the podcast capture", "connect venture lab to my Notion", "create my Sources inbox", or the capture skill reports missing config. Run once per machine/workspace; idempotent.
---

One-time, per-user setup so the `venture-lab` capture skills write into **the
runner's own** Notion — never a hardcoded workspace. This skill confirms the
connected account, ensures a **🎧 Sources inbox** database exists, and records
its id in a small local config file. The capture skill (separate) reads that
file; **no workspace or database id ever lives in the repo.**

Read **`references/sources-inbox.md`** first — it is the canonical Sources
schema and the config-file contract this skill writes.

## What this produces

- A **🎧 Sources** database in the runner's Notion (created here if absent),
  matching the schema in `references/sources-inbox.md`.
- A per-user config file at
  `${XDG_CONFIG_HOME:-$HOME/.config}/venture-lab/config.json` holding
  `notion.sourcesDbId` (+ optionally `researchDbId`, `workspaceHint`). Ids only,
  **never secrets.**

## Before you build — confirm the account

1. **Whose Notion is this?** `notion-fetch` with `id: "self"` and confirm the
   connected account/workspace is the runner's own. Never build into someone
   else's workspace by mistake. Note the workspace label for `workspaceHint`.
2. **Is there already a config?** Read
   `${XDG_CONFIG_HOME:-$HOME/.config}/venture-lab/config.json`. If it already has
   a `notion.sourcesDbId`, `notion-fetch` that id to confirm the DB still exists
   and has the right schema — if so, report "already configured" and stop (offer
   to repair only what's missing). This skill is idempotent; do not create a
   second Sources DB.
3. **Where should the Sources DB live?** Search for the runner's `🧪 Venture Lab`
   home page (`notion-search`) and create the Sources DB as a child of it. If no
   Venture Lab home exists, create the Sources DB as a private top-level page,
   say so, and offer to move it later. (Setting up the wider workspace is the
   `venture-lab-setup` skill's job, not this one.)

## Procedure

1. **Find or create the 🎧 Sources database.**
   - Search the runner's workspace for an existing `Sources` database under
     Venture Lab. If a suitable one exists, use it — do not create a duplicate.
   - Otherwise create it with `notion-create-database` under the Venture Lab
     home page, exactly the schema in `references/sources-inbox.md`: `Name`
     (title), `Source URL` (url), `Video ID` (text), `Timestamp` (text), `Hook`
     (text), `Status` (select: `Captured` gray / `Promoted` green / `Discarded`
     brown), `Transcript` (files), and — only if a 💭 Research database exists —
     `Related Research` (two-way relation → that Research DB). Icon `🎧`.
   - If Research does not exist yet, create the DB **without** `Related Research`
     and note that it can be added later; the capture skill does not need it.
2. **Resolve the Research DB id (optional).** If a 💭 Research database exists in
   the workspace, capture its id for `researchDbId` (used by the downstream
   extract skill, not by capture).
3. **Write the config file.** Create
   `${XDG_CONFIG_HOME:-$HOME/.config}/venture-lab/config.json` (make the
   directory if needed) with:
   ```json
   {
     "notion": {
       "sourcesDbId": "<the Sources DB id>",
       "researchDbId": "<the Research DB id, or omit>",
       "workspaceHint": "<human label of the connected workspace>"
     }
   }
   ```
   Overwrite only the keys you set; preserve any unrelated keys already present.
   **Never write a Notion token or any secret into this file** — it is ids only.
4. **Verify.** Re-read the config file and `notion-fetch` the `sourcesDbId` to
   confirm the DB resolves and its schema matches. Report the DB URL, the config
   path, and that the capture skill is now pointed at the runner's workspace.

## Guardrails

- **No hardcoded ids in the repo.** Every workspace/DB id lives only in the
  runner's local config file. A repo-committed id is both a bug and a privacy
  leak — this skill exists precisely to keep the plugin workspace-agnostic.
- **No secrets anywhere.** The config file carries ids, not auth. The Notion
  token stays in the runner's own connector / secret store — never in the config
  file, a skill, the repo, or a Notion page.
- **Idempotent.** Re-running must not create a second Sources DB or a second
  config file — find-or-create, then update in place.
- **Capture status only, downstream.** This skill sets up structure; it writes no
  Source rows. The capture skill it enables only ever sets `Status = Captured`
  and never writes `Related Research` — the human-gated promotion is a separate,
  downstream act.
