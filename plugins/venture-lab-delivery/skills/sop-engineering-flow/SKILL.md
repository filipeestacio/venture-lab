---
name: sop-engineering-flow
description: How to land a change on a governed repo -- worktree -> bot PR -> human approval -> deploy, plus secrets handling. Load when implementing or landing a change.
---

The execution flow for landing a change on a governed repo. This is the portable **method**; a project may ship its own flow skill with the concrete commands (branch names, PR wrapper, deploy verb) — load that too when you have one.

- **Worktree only** on governed (fetch/ff-only) repos: never edit or commit the canonical checkout directly; branch in a worktree so parallel agents cannot diverge `main`.
- **Bot-authored PR**: open PRs under a distinct bot identity, not your own — **author must not be approver**. Put the body in a file (`--body-file`), not inline, so no secret or guard-tripping word lands in argv. The bot requests the human as reviewer, or the PR never surfaces in their review list.
- **Human approval + merge**: agents never self-merge.
- **Deploy** (only when Done requires activation): human-gated. Agents never activate production.
- **Secrets**: the machine guard matches command *text* — keep secrets, and `rm`/`sudo`-like words, out of argv and commit messages; pass bodies via `--body-file`; inject secret values via a pipe or editor shim, never as an argument. Never put a secret in the repo, logs, or a product doc.
- **One concern per PR**; keep `main` deployable at every step. For a pure refactor, verify the build output is identical.

Self-review with `sop-pre-pr-review` **before** you open the PR — every issue it catches is a review round trip you did not spend.
