# Sources inbox — schema + per-user config contract

The **Sources inbox** is the landing zone for podcast (and other) captures: one
lightweight **pointer** row per episode — URL + timestamp + why it caught you —
with the full transcript riding along as an archival `.md` attachment. It is
**not** a transcripts database: the transcript is regenerable from the video id,
so the row stores the pointer and caches the transcript as a file, nothing more.

Two documents define the boundary the capture skill writes to:

1. **The database schema** (below) — what a Sources inbox looks like.
2. **The per-user config contract** (below) — how any plugin user points the
   skill at *their own* workspace, so no workspace/DB id ever lives in the repo.

---

## 1. Database schema — 🎧 Sources

Icon: `🎧`. Title property: `Name`. Property order below is the intended
left-to-right table order.

| Property | Type | Details |
|---|---|---|
| `Name` | title | `show — episode` (e.g. `My First Million — How to find a business idea`). |
| `Source URL` | url | The YouTube watch URL, with `&t=<seconds>` appended when a timestamp is given. |
| `Video ID` | rich_text | The YouTube video id — **the dedupe key** the upsert is keyed on. One row per id. |
| `Timestamp` | rich_text | The moment that caught you, as given (e.g. `12:34` or `1:02:00`). Free text; empty is fine. |
| `Hook` | rich_text | One line: why this is worth keeping. Not a summary — the reason. |
| `Status` | select | `Captured` (gray) · `Promoted` (green) · `Discarded` (brown). New captures are **`Captured`**. |
| `Transcript` | files | The archival `.md` transcript, attached. Provenance, not a quotable record. |
| `Related Research` | relation → 💭 Research | Two-way. **Written by the downstream extract skill, never by capture.** |

Notes:

- **Pointers only.** No transcript text in the page body — the transcript is the
  `Transcript` file attachment. A row is a pointer plus a cached artefact.
- **`Status` is capture-only for the agent.** The capture skill only ever sets
  `Captured`. `Promoted` / `Discarded` are human/downstream decisions — the
  capture skill must never set them, and never writes `Related Research`.
- **`Related Research`** is a two-way relation to the Venture Lab 💭 Research
  database; creating it adds a reciprocal property on Research. If the workspace
  has no Research database yet, create the Sources inbox without this property
  and add the relation once Research exists — the capture skill does not need it.

---

## 2. Per-user config contract

The `venture-lab` plugin is multi-user and public. **No workspace or database id
is hardcoded in any skill.** Each user points the plugin at their own Notion by
writing a small local config file; the skills read it.

### Location

```
${XDG_CONFIG_HOME:-$HOME/.config}/venture-lab/config.json
```

Local to the user's machine, **outside the repo**. It carries **ids only, never
secrets** — the Notion auth lives in the user's own connector/secret store, never
here.

### Shape

```json
{
  "notion": {
    "sourcesDbId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "researchDbId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "workspaceHint": "Filipe — personal Notion"
  }
}
```

| Key | Required | Meaning |
|---|---|---|
| `notion.sourcesDbId` | **yes** | The Sources inbox database id the capture skill upserts into. |
| `notion.researchDbId` | no | The 💭 Research database id, for the downstream extract skill. |
| `notion.workspaceHint` | no | Human-readable label so a mis-connected session is caught early. Never used as an id. |

### Discovery contract (what any skill reading this must do)

1. Read `${XDG_CONFIG_HOME:-$HOME/.config}/venture-lab/config.json`.
2. If the file is missing, unparseable, or has no `notion.sourcesDbId`:
   **stop and tell the user to run `configure-venture-lab`.** Never guess a DB
   id, never fall back to a hardcoded one.
3. Before writing, confirm the connected Notion account is the intended one
   (`notion-fetch` `id: "self"`); if `workspaceHint` is set and clearly does not
   match, stop and ask rather than write into the wrong workspace.
4. Use `notion.sourcesDbId` as the upsert target.

A hardcoded id would be both a bug and a privacy leak — the whole point of this
file is that the repo stays workspace-agnostic.
