---
name: notion-editing
description: Safe editing of Notion pages via the Notion MCP -- avoids the silent corruption modes in update-page (replace_content/insert_content) and content_updates. Load before any non-trivial edit to a Notion page body (plans, PRDs, docs).
---

Editing a Notion page body via the Notion MCP fails **silently**: the tool returns `{page_id}` (success) even when the render is corrupted or the document is truncated. Three rules avoid the traps; always verify after.

**1. Use REAL newline characters, not `\n` escapes, in `update-page` (`replace_content` / `insert_content`).** A literal `\n` renders as the letter `n`, collapsing the whole document into one blob and breaking code fences. (`create-pages`' `content` field DOES accept `\n`; `update-page` does not — that asymmetry is the trap.)

**2. Do not lead a body with a multi-line `<callout>`.** Combined with any newline mishandling it breaks the block parser and **truncates everything after it**, leaving only a mangled callout. Use a blockquote (`>`) or a heading as the first block; keep callout content single-line, or build callouts via `create-pages`.

**3. `content_updates` (search/replace) `old_str` must match Notion's NORMALIZED markdown, not what you authored.** Notion reshuffles bold/inline-code boundaries on save — e.g. what you wrote as `**one `x` per**` comes back as `**one ****`x`**** per**`, so hand-written anchors usually miss. Fetch the page first and copy the exact stored text, or anchor on short plain-text substrings with no bold/code.

**Always fetch-verify.** After any large `replace_content`, `notion-fetch` the page and read it — the success return proves nothing about the render.
