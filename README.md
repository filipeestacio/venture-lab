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

Once installed and reloaded, the skill is model-invoked — ask Claude to "set up a Venture Lab in my Notion" (or invoke `/venture-lab:venture-lab-setup`) in a session where the **Notion connector is connected to your own account**.

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
│           └── venture-lab-setup/
│               ├── SKILL.md
│               └── references/
│                   ├── databases.md
│                   └── operating-model.md
├── README.md
└── LICENSE
```

## License

[MIT](./LICENSE).
