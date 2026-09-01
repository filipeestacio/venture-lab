---
name: venture-lab-setup
description: Scaffold a "Venture Lab" Product workspace in the current Notion account — a 🧪 home page, a portable (Notion-only) Operating Model SOP, and the 🤔 Plans + 💭 Research databases (with the Research↔Plans relation and a ⭐ TEMPLATE row in each). Load when someone asks to set up / create / scaffold a Venture Lab in their Notion, or to mirror a partner's Venture Lab structure.
---

Build a **Venture Lab Product workspace** in the Notion account the current session is connected to. This is the shareable, portable version: it scaffolds the *Product* side only (research, PRDs, phased plans in Notion). **Engineering — GitHub, repos, issues, code, deploys — is NOT part of this workspace; it stays entirely with the engineering partner** who receives Approved Plans and executes on their own GitHub.

Use the Notion MCP (`notion-create-database`, `notion-create-pages`, `notion-update-page`, `notion-fetch`). Author page bodies in Notion-flavoured Markdown (NfM); full spec at `notion://docs/enhanced-markdown-spec`.

## What gets created

```
🧪 Venture Lab                         (home page)
├── 🧭 Operating Model — Product        (SOP page; body in references/operating-model.md)
├── 🤔 Plans        (database)  ── Research relation ──┐
└── 💭 Research     (database)  ◀── Related Plan ───────┘
        each DB carries one ⭐ TEMPLATE row (skeleton body, Status = Draft)
```

Exact schemas, select options + colours, property order, and the two ⭐ TEMPLATE page bodies are in **`references/databases.md`**. The Operating Model page body is in **`references/operating-model.md`**. Read both before creating anything.

## Before you build — gather 3 things

1. **Whose account is this?** Confirm the Notion MCP is connected to the *runner's own* account (`notion-fetch` with `id: "self"`). Never build into someone else's workspace by mistake.
2. **Two names.** Ask the runner for:
   - **their own name** (`<YOU>`) — becomes the default `Owner`/`Author`;
   - **the engineering partner's name** (`<PARTNER>`) — the person who owns all GitHub/engineering and receives Approved Plans. Default `<PARTNER>` to **Filipe** if they don't say.
   Substitute both wherever the reference files show `<YOU>` / `<PARTNER>`.
3. **Where to put it.** Ask for a parent page. If none is given, create the 🧪 Venture Lab page as a **private top-level page**, then tell the runner it's private and offer to move it.

**Idempotency:** first `notion-search` for an existing "Venture Lab" page in this account. If one exists, stop and ask before creating a second — offer to add only what's missing instead.

## Procedure

1. **Create the 🧪 Venture Lab home page** at the chosen location, with the intro + "How this space works" body from `references/operating-model.md` (the `HOME PAGE BODY` block). Icon `🧪`.
2. **Create the 🤔 Plans database** as a child of the home page, using the Plans schema in `references/databases.md`. Icon `🤔`. Create it *first* — Research relates to it.
3. **Create the 💭 Research database** as a child of the home page, using the Research schema. Icon `💭`. Its `Related Plan` property is a **two-way relation to Plans**; enable the reciprocal side so Plans shows a `Research` property. (If the MCP can't create a two-way relation in one step, create the relation on Research → Plans, then add/confirm the back-relation on Plans.)
4. **Add one ⭐ TEMPLATE row to each database** — title exactly `⭐ TEMPLATE`, `Status = Draft`, no other data — with the matching skeleton page body from `references/databases.md`. If the runner's Notion supports native database templates, set this row as the database's default template; otherwise it stays a normal row to duplicate-and-fill.
5. **Create the 🧭 Operating Model — Product page** as a child of the home page, body from `references/operating-model.md` (the `OPERATING MODEL BODY` block), with `<PARTNER>`/`<YOU>` substituted. Icon `🧭`.
6. **Cross-link:** on the home page body, ensure the two databases and the Operating Model page are referenced/embedded so the space reads as one unit.
7. **Report back:** list every created page/DB with its URL, note anything created private, and state the two things the runner still does by hand (below).

## Hand to a human afterwards (the MCP can't do these)

- **Sharing/permissions** — invite `<PARTNER>` (comment or edit) so they can pick up the next stage. Both partners keep *parallel* workspaces; either can own a Research doc or Plan and share it for comment.
- **The Approved gate is human-only.** Agents never flip a Plan to `Approved` and never create GitHub work — that is `<PARTNER>`'s engineering step, off in their own GitHub.

## House style (apply to every page you author)

- **BLUF** — every page opens with a one-callout TL;DR (💡 `blue_bg`): the decision/finding/goal + current state in 1–3 sentences.
- **Real headings only** (`#`/`##`/`###`, never bold-as-heading; never skip a level) so the Table of Contents works.
- **Callout vocabulary** (icon + colour always travel together): 💡`blue_bg` info/TL;DR · ⚠️`yellow_bg` caution · ✅`green_bg` decided/approved · 🛑`red_bg` blocker · 📎`gray_bg` aside. Max ~3 colours per page; colour supports hierarchy, never decorates.
- **Status vocabulary is fixed** — reuse the options defined in `references/databases.md`; do not invent per-page statuses. **Refined-and-ready ⇒ `In Review`, not Draft.** Only a human sets `Approved`.
- Add `<table_of_contents/>` to any page longer than one screen; separate major sections with `---`.

Do not reproduce any fleet-specific machinery (nixos, SOPS, deploy bots, a specific GitHub account) — those live only with `<PARTNER>` and are deliberately out of this Notion workspace.
