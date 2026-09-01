---
name: sop-scaffold-procedure
description: Scaffold an Approved Venture Lab Plan into GitHub (Phase->Milestone, Task->Issue, tracking issue; plan-level tracking stays in the Notion Plan). Load when turning an Approved Plan into engineering work.
---

Turn an **Approved** Venture Lab Plan into GitHub work. The Plan lives in the Product workspace's Plans database; its Operating Model — Product SOP is the canonical source for the lifecycle and the Definition of Ready. **Never scaffold a Plan that is not `Approved`** — only a human flips a Plan to Approved, and that flip is the handoff trigger.

Mapping: **Phase -> Milestone . Task -> Issue.** Keep plan-level tracking **in the Notion Plan**, not in a parallel GitHub Project — cross-link each Notion task to its issue and let the Plan's `Status` mirror execution. One source of truth per fact; cross-link, never hand-mirror.

Procedure:
1. Read the Approved Plan; confirm the Definition of Ready is met. If a task is too vague to write a Definition of Done for, stop and report the gap — it is not ready to scaffold.
2. Create one **Milestone per phase** in the target repo.
3. Create one **Issue per task** following `sop-issue-contract`, assigned to its milestone, labelled `plan:<slug>`, with `#`-linked dependencies.
4. Create a **tracking issue** (the critical path + a checklist of all issues) that links back to the Plan.
5. Set the Plan `Status = In Progress`; write each task's issue URL back into the Plan (into `Links` or the task row).
6. **Idempotent:** re-running must not duplicate issues — match by title / `plan:<slug>` and update in place.

Each venture is scaffolded from its own account/org and its own Plan. Do not scaffold one venture's work from another's identity.
