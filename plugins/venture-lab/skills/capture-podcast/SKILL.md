---
name: capture-podcast
description: Capture a podcast/YouTube episode into the Venture Lab Sources inbox — from an episode URL (+ optional timestamp and a one-line hook), fetch a clean transcript with podcast-transcript, and upsert one Source row in the user's Notion (idempotent on YouTube video id) with the transcript attached as a .md. Load when someone shares a YouTube/podcast episode link and wants to save, capture, clip, or remember it ("capture this episode", "save this to my sources / Venture Lab", "add this podcast to Notion", "remember this bit at 12:34"). Writes at Captured status only; never touches Research.
---

Turn one podcast/YouTube episode URL into a durable **Source** capture in the
runner's own Notion: a pointer row (URL + timestamp + hook) with the full
timestamped transcript attached as an archival `.md`, plus a short **descriptive
summary** in the row body so the inbox is scannable without opening the
transcript. **Idempotent** — re-running on the same episode refreshes its one row
(fields, attachment, and summary), never duplicates.

This skill only ever writes a Source at **`Captured`** status. It never writes
Research, never sets `Related Research`, and never sets a human-gated status
(`Promoted`/`Discarded`) — promotion to Research is a separate, downstream act.

The deterministic details (field derivation, exit-code handling, the exact Notion
query/create/update/attach calls) are in **`references/capture-upsert.md`** —
follow it; this page is the shape of the task.

## Inputs

- **Episode URL** (required) — a YouTube watch/share link (`watch?v=`, `youtu.be`,
  `/live/`, `/embed/`, `/shorts/`). Shows publish full episodes on YouTube; that
  is the transcript source.
- **Timestamp** (optional) — the moment that caught the user (e.g. `12:34`). Goes
  on the row and as `&t=` on the Source URL.
- **Hook** (optional) — one line on *why* it's worth keeping. Not a summary. If
  the user didn't give one, leave it empty — don't invent it.

## Preconditions

- **`podcast-transcript` on PATH** (Phase 1; shipped fleet-wide via the flake). If
  it's missing, stop and say so — this skill drives it, it does not reimplement it.
- **Per-user config present.** Resolve `notion.sourcesDbId` from
  `${XDG_CONFIG_HOME:-$HOME/.config}/venture-lab/config.json`. If it's missing,
  stop and tell the user to run **`configure-venture-lab`** first. Never guess or
  hardcode a workspace/DB id.

## Procedure

1. **Resolve the target.** Read the config → `sourcesDbId`; `notion-fetch` it for
   the data source URL + schema; confirm the connected Notion account is the
   intended one. (`references/capture-upsert.md` §0.)
2. **Extract the Video ID** from the URL — the 11-char id, the dedupe key. If you
   can't get a clean id, stop and ask. (§2.)
3. **Fetch the transcript.** Run `podcast-transcript "$URL"` and capture stdout to
   `"$id".md`. On any **non-zero exit, stop and report — write no row.** In
   particular exit 4 = this show has no captions/video; say so plainly (the
   coverage gap must fail loudly, never as a silent empty row). (§1.)
4. **Derive the fields** — `Name` = `"<Show> — <title>"` from the transcript
   header, `Source URL` (with `&t=<sec>s` if a timestamp was given), `Video ID`,
   `Timestamp`, `Hook`, `Status = Captured`. (§2.)
5. **Upsert, keyed on Video ID.** Query the data source for the id first; **update**
   the existing row if present (refresh fields + re-attach the transcript),
   **create** one if absent. Never create a second row for the same id. (§3.)
6. **Attach the transcript** to the `Transcript` files property via
   `create-file-upload` → POST → reference the upload id (pointers only — no
   transcript text in the page body). (§4–5.)
7. **Write the descriptive summary** into the page body — a 3–4 sentence abstract
   + ~6–8 key-point bullets, from the transcript. Descriptive only (what it is /
   its substance), **not** idea-extraction. On a re-run, replace the existing
   summary; never append a second. (§6.)
8. **Report** created-vs-updated, the `Name`, that the summary was written, and
   the Notion row URL. (§7.)

## Guardrails

- **Capture status only.** Only ever set `Status = Captured`. Never set
  `Promoted`/`Discarded`, never write `Related Research`, never create a Research
  entry — that judgement is a separate, human/downstream act.
- **Idempotent.** The Video ID is the key; one row per episode. A second run is an
  update, never a duplicate — including the body summary (replace, don't append).
- **Summary is descriptive, not extraction.** The body summary says what the
  episode is and its substance, to aid scanning. It does **not** decide whether
  there's a claim worth testing or write anything toward a 💭 Research entry —
  that idea-extraction is the separate, deferred downstream skill.
- **Fail loudly on the coverage gap.** A show with no YouTube captions must produce
  a clear message and **no** row — never an empty or half-filled capture.
- **Per-user, no leaked ids.** The Sources DB comes from the runner's own config;
  carry no workspace/DB id in the skill.
- **No secrets.** The Notion token stays in the runner's own connector/secret
  store — never in the transcript, the row, the config file, or this repo. The
  transcript is a private research artefact (auto-captions are lossy — the stored
  timestamp keeps the claim checkable; the `.md` is provenance, not a quotable
  source of record).
