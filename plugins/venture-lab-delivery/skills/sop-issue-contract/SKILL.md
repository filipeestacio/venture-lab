---
name: sop-issue-contract
description: The self-contained GitHub issue template every scaffolded Plan issue must follow. Load when writing or reviewing issues for an Approved Plan.
---

An autonomous agent picking up an issue has only the issue, not the Plan — so **every issue must stand alone**. Required sections:

- **Context** — one line + a link to the Plan and its phase.
- **Goal** — one sentence: what "done" means.
- **Implementation plan** — concrete steps; the files / functions / anchors to touch.
- **Definition of Done (merge gate)** — item-specific criteria + the standard gate (see `sop-definition-of-done`).
- **Dependencies** — `#`-linked blockers; what must land first.
- **Guardrails / notes** — the machine policy that applies (no secrets in the diff; worktree + bot-authored PR; deploy/promotion only via the human-gated flow; identical-derivation check for pure refactors).
- **Out of scope** — what not to touch in this PR.
- **Risk** — Low / Med / High.

If you cannot write a Definition of Done for the issue, it is not ready to start.
