# Venture Lab — Claude Code plugin marketplace

A public [Claude Code](https://code.claude.com) plugin marketplace for sharing skills, starting with **Venture Lab**: a Notion-based *Product* workspace for taking venture ideas from discovery to an engineering handoff.

This repo is both a **marketplace** (`.claude-plugin/marketplace.json`) and the home of the plugins it lists (under `plugins/`). It grows over time — new skills land in the `venture-lab` plugin, or as new plugins alongside it.

## Install

In Claude Code:

```
/plugin marketplace add filipeestacio/venture-lab
/plugin install venture-lab@venture-lab
/reload-plugins
```

- `marketplace add` registers this repo as a source.
- `install venture-lab@venture-lab` installs the **venture-lab** plugin from the **venture-lab** marketplace.
- `/reload-plugins` activates the newly installed skills in your session.

To get later updates:

```
/plugin marketplace update venture-lab
```

## What's inside

### Plugin: `venture-lab`

| Skill | What it does |
|---|---|
| `venture-lab-setup` | Scaffolds a Venture Lab Product workspace in the Notion account the session is connected to — a 🧪 home page, a portable (Notion-only) Operating Model SOP, and the 🤔 Plans + 💭 Research databases (with the Research↔Plans relation and a ⭐ TEMPLATE row in each). |
| `configure-venture-lab` | Points the plugin at *your own* Notion: confirms the connected account, creates (or records) your 🎧 Sources inbox database, and writes a per-user config file the podcast-capture skill reads. Run once per machine/workspace; no workspace/DB id ever lives in the repo. |
| `capture-podcast` | From a podcast/YouTube episode URL (+ optional timestamp and a one-line hook), fetches a clean transcript with `podcast-transcript` and upserts one 🎧 Source row in your Notion — idempotent on the YouTube video id, transcript attached as a `.md`. Captures at `Captured` status only; never touches Research. Needs `configure-venture-lab` run first. |

Once installed and reloaded, the skills are model-invoked — ask Claude to "set up a Venture Lab in my Notion", "configure venture lab for my Notion", or "capture this episode" with a YouTube link (or invoke `/venture-lab:venture-lab-setup` / `/venture-lab:configure-venture-lab` / `/venture-lab:capture-podcast`) in a session where the **Notion connector is connected to your own account**.

> `capture-podcast` also needs the `podcast-transcript` executable on PATH (a YouTube-captions → clean-markdown tool shipped separately, e.g. via your Nix flake). It downloads no audio and runs no model.

**What it builds:**

```
🧪 Venture Lab
├── 🧭 Operating Model — Product   (portable, Notion-only SOP)
├── 🤔 Plans      ──Research relation──┐
└── 💭 Research   ◀──Related Plan──────┘
      each database carries one ⭐ TEMPLATE row (house-style skeleton)
```

Engineering — GitHub, repos, issues, code, deploys — is deliberately **not** part of this workspace. It stays with your engineering partner, who receives Approved Plans and executes on their own GitHub.

## Prerequisites

- Claude Code with the **Notion** connector enabled and connected to the account you want the workspace built in.

## Repo layout

```
venture-lab/
├── .claude-plugin/
│   └── marketplace.json          # marketplace manifest (added via /plugin marketplace add)
├── plugins/
│   └── venture-lab/
│       ├── .claude-plugin/
│       │   └── plugin.json        # plugin manifest
│       └── skills/
│           ├── venture-lab-setup/
│           │   ├── SKILL.md
│           │   └── references/
│           │       ├── databases.md
│           │       └── operating-model.md
│           ├── configure-venture-lab/
│           │   ├── SKILL.md
│           │   └── references/
│           │       └── sources-inbox.md
│           └── capture-podcast/
│               ├── SKILL.md
│               └── references/
│                   └── capture-upsert.md
├── README.md
└── LICENSE
```

## License

[MIT](./LICENSE).
