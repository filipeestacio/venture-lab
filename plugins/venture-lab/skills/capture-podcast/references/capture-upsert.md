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

1. `notion-create-file-upload { filename: "<id>.md" }` → returns `upload_url`,
   `upload_form_field` (`"file"`), and `upload_headers` (an `Authorization:
   Bearer …` — a short-lived secret).
2. POST the file as one multipart request. The bearer token is a secret, so keep
   it **out of argv** (a `-H "Authorization: Bearer …"` on the command line is
   visible in the process list): write the header to a `curl` config file
   (mode 600) and pass it with `--config`:
   ```
   # upload.curl  (chmod 600)
   url    = "<upload_url>"
   header = "Authorization: Bearer <token>"
   form   = "file=@/tmp/<id>.md;type=text/markdown"
   ```
   ```bash
   curl -sS --config upload.curl        # method is POST because of the form field
   ```
   The response JSON carries `file_upload_id` and `status: "uploaded"`.
3. Reference that id in the `Transcript` property — but note **only
   `notion-update-page` accepts a `file_upload` object in a property**;
   `notion-create-pages` does not (see §5). So the attach always happens on an
   `update-page` call:
   ```json
   "Transcript": [{ "type": "file_upload", "file_upload": { "id": "<file-upload-id>" } }]
   ```

A file upload is single-use — mint a fresh one per run (each run re-attaches).
The single-part limit is 20 MiB; no podcast transcript approaches it (~100 KiB
for a 2-hour episode), so there is no multi-part path to worry about.

## 5. Create / update payloads

**`create-pages` cannot set a Files property** — its `properties` accept only
strings/numbers/string-arrays, not a `file_upload` object. So a **new** capture is
always two calls: create the row with the text fields, then `update-page` to
attach the transcript. An **update** is one `update-page` that both refreshes the
fields and re-attaches. Either way the `Transcript` write lands on `update-page`.

**Create the row** (`notion-create-pages`, parent `data_source_id: "<ds>"`) — no
`Transcript` here:
```json
{
  "Name": "<Show> — <title>",
  "Source URL": "https://www.youtube.com/watch?v=<id>&t=<sec>s",
  "Video ID": "<id>",
  "Timestamp": "<raw ts or empty>",
  "Hook": "<hook or empty>",
  "Status": "Captured"
}
```
`Source URL` is a url property (plain string). `Video ID` / `Timestamp` / `Hook`
are text (strings). Do **not** put transcript text in the page `content`.

**Attach / update** (`notion-update-page`, `command: "update_properties"`,
`page_id` = the new page id, or the matched row's `url` on a re-run): set the
`Transcript` upload, and on a re-run refresh `Source URL` / `Timestamp` / `Hook` /
`Status` too:
```json
{ "Transcript": [{ "type": "file_upload", "file_upload": { "id": "<upload-id>" } }] }
```
Leave `Related Research` untouched — it is never written here.

## 6. Report

State whether the row was **created** or **updated**, its `Name`, and its Notion
URL. On any non-zero `podcast-transcript` exit, report the reason and that no row
was written.
