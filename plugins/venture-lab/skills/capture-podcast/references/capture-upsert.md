# Capture mechanics — transcript, upsert, attach

The deterministic details behind the `capture-podcast` skill: how to derive each
field, how to keep the write idempotent, and the exact Notion calls. The schema
and per-user config contract live in the sibling `configure-venture-lab` skill's
`references/sources-inbox.md` — read that for `sourcesDbId` discovery.

## 0. Resolve the target (per-user, no hardcoded ids)

1. Read `${XDG_CONFIG_HOME:-$HOME/.config}/venture-lab/config.json`. If missing or
   without `notion.sourcesDbId`, **stop** and tell the user to run
   `configure-venture-lab`. Never guess a DB id.
2. `notion-fetch` the `sourcesDbId` to get its **data source URL**
   (`collection://<ds>`) and confirm the schema. Confirm the connected account is
   the intended one (`notion-fetch id:"self"`; sanity-check against
   `workspaceHint` if set).

## 1. The transcript (Phase-1 executable)

`podcast-transcript <youtube-url>` prints the transcript markdown to **stdout**.
Capture it to a file named for the video id: `"$id".md`.

```bash
podcast-transcript "$URL" > "/tmp/<id>.md"; rc=$?
```

**Exit codes — never write a Source row on a non-zero exit** (the coverage gap
must fail loudly, not land an empty row):

| rc | meaning | do |
|----|---------|----|
| 0 | success | proceed |
| 2 | usage error | fix the argument; do not write |
| 3 | yt-dlp missing / metadata fetch failed | stop; report (transient or PATH) |
| 4 | **no captions** (e.g. audio-only show, no YouTube video) | stop; tell the user this show isn't covered — do **not** create a row |
| 5 | couldn't read metadata / fetch caption track | stop; report |

The executable takes **no** timestamp flag — `--at` and the hook are handled here
(they shape the Source URL and fields), not by the executable.

## 2. Derive the fields

- **Video ID** — extract the 11-char id (`[A-Za-z0-9_-]{11}`) from the URL:
  `watch?v=<id>`, `youtu.be/<id>`, `/live/<id>`, `/embed/<id>`, `/shorts/<id>`.
  This is the **dedupe key**. If you cannot extract a clean id, stop and ask —
  never invent one.
- **Source URL** — canonical `https://www.youtube.com/watch?v=<id>`; if a
  timestamp was given, append `&t=<seconds>s` (convert `mm:ss` / `h:mm:ss` /
  `90s` / `12m` to whole seconds).
- **Name** — `"<Show> — <title>"`, parsed from the transcript header: the first
  line is `# <title>`; the `- Show: <show>` line gives the show. If either is
  `unknown`, fall back to the other alone; never leave `Name` blank.
- **Timestamp** — the raw value the user gave (`12:34`), or empty.
- **Hook** — the one line the user gave, or empty. Do not invent one.
- **Status** — always `Captured`. Never `Promoted`/`Discarded`.
- **Related Research** — never set here.

## 3. Idempotent upsert, keyed on Video ID

Query the data source for an existing row **before** writing:

```
notion-query-data-sources  (mode sql)
  data_source_urls: ["collection://<ds>"]
  query:  SELECT url, "Name" FROM "collection://<ds>" WHERE "Video ID" = ?
  params: ["<id>"]
```

- **≥1 row** → **update** the first (`url` is the page id): refresh `Source URL`,
  `Timestamp`, `Hook`, `Status = Captured`, and replace `Transcript` with the new
  upload. Do not create a second row.
- **0 rows** → **create** one page under the data source.

## 4. Attach the transcript to the `Transcript` files property

Pointers only — the transcript is a **file attachment**, never page-body text.
Use the file-upload → property path (a Files property takes a `file_upload` id):

1. `notion-create-file-upload { filename: "<id>.md" }` → returns `upload_url` and
   `upload_headers`.
2. POST the file (one multipart request; include every returned header):
   ```bash
   curl -s -X POST "<upload_url>" \
     -H "<each upload_headers entry>" \
     -F "file=@/tmp/<id>.md;type=text/markdown"
   ```
   The response JSON carries the file-upload **`id`** (and `markdown_source`).
3. Set the property with that id, in the same create/update call:
   ```json
   "Transcript": [{ "type": "file_upload", "file_upload": { "id": "<file-upload-id>" } }]
   ```

If a transcript ever exceeds the single-part upload limit (20 MiB — no podcast
transcript will), it still fits; there is no multi-part path to worry about here.

## 5. Create / update payloads

**Create** (`notion-create-pages`, parent `data_source_id: "<ds>"`):
```json
{
  "Name": "<Show> — <title>",
  "Source URL": "https://www.youtube.com/watch?v=<id>&t=<sec>s",
  "Video ID": "<id>",
  "Timestamp": "<raw ts or empty>",
  "Hook": "<hook or empty>",
  "Status": "Captured",
  "Transcript": [{ "type": "file_upload", "file_upload": { "id": "<upload-id>" } }]
}
```
`Source URL` is a url property (plain string). `Video ID` / `Timestamp` / `Hook`
are text (strings). Do **not** put transcript text in the page `content`.

**Update** (`notion-update-page`, `command: "update_properties"`, `page_id` = the
matched row's `url`): the same `properties` map, refreshing the `Transcript`
upload. Leave `Related Research` untouched.

## 6. Write the descriptive summary to the page body

Pointers stay in the properties; the **summary** is the one piece of synthesised
text that goes in the page **body**, so the row is scannable without opening the
transcript. It is a *descriptive* summary — say what the episode is and its
substance — **not** idea-extraction: do not judge whether there's a testable
claim, and write nothing toward Research (that is the deferred downstream skill).

From the transcript you already fetched, write this into the page body:

```markdown
## Summary

<3–4 sentence abstract: what this is, who's on, the shape of the substance>

### Key points

- <~6–8 bullets covering the actual content — the arguments, cases, takeaways>

> _Auto-summary of the attached transcript (auto-captions, lossy). Provenance,
> not a quotable source of record._
```

**Idempotent:** on first capture, `insert_content` this at the end of the
freshly-created row. On a **re-run, replace** the existing summary — do not
append a second `## Summary`. The body holds only this section (no child
pages/databases), so `notion-update-page` `replace_content` with the new summary
is the safe refresh. Never put the transcript text itself in the body — the
transcript is the `Transcript` file attachment.

## 7. Report

State whether the row was **created** or **updated**, its `Name`, that the
summary was written, and the Notion URL. On any non-zero `podcast-transcript`
exit, report the reason and that no row was written.
