---
name: sop-pre-dor-review
description: Self-review gate to run on a scaffolded issue's body BEFORE Karnak's Definition-of-Ready review sees it -- right after sop-scaffold-procedure opens it, and again before relabeling status:triage/needs-info back to status:refinement after addressing a finding. Checks the same 5-item DoR rubric Karnak judges, plus the mechanical patterns that keep failing it, so an issue clears DoR in one round instead of two. Load whenever you are about to open a scaffolded issue or relabel one back into review.
---

Karnak (the automated DoR reviewer) is the last check on an issue's readiness, not the first — every finding it posts is a review round you did not need to spend. This is `sop-pre-pr-review`'s counterpart for issue text instead of a diff: same idea, applied before the *ticket* is picked up rather than before the *PR* is opened.

**Where this fires:** after `sop-scaffold-procedure` writes an issue body and before it is opened (it will land on `status: triage` either way — the point is to make that first Karnak pass a `ready: true`); and again before you relabel an issue `status: triage`/`status: needs-info` → `status: refinement` to address a finding. That relabel is the *only* thing that re-triggers Karnak (a poller diffs the label signature, deliberately not `updated_at`, so editing the body alone does nothing) — an issue has no author trailer to route a bounced verdict back through, and there is no round cap, so relabeling before you're actually done just burns another cycle for nothing.

Karnak judges exactly 5 items (`dor-issue.yml`, PLAN-47) and one `fail` sinks the whole verdict. Work each one adversarially — as Karnak will, by re-reading the text cold and grepping the repo, not as the author who already knows what they meant:

1. **Problem.** Impact, not solution. If you can swap the Problem and Goal sections and the issue still reads fine, Problem is not written yet — it's currently just Goal repeated. State what's broken/missing/costly *today* and who or what it affects. A Risk line about the fix is not a substitute.

2. **Scope.** Explicit in/out is usually the easy pass. What actually fails review here: a mid-plan conditional fork — "recover X; if that doesn't work, do Y instead" — which makes the issue two different pieces of work wearing one number. Either resolve the fork now (commit to Y and say so) or split the discovery step into its own issue.

3. **Acceptance criteria.** Every DoD bullet must be checkable by someone who wasn't in the room. Grep the issue for any term a bullet depends on (an acronym, a named artifact, a decision) and confirm it's either defined in the body or already a settled fact — never a term the same issue lists as *out of scope to select*. An undefined-term gate is an unfalsifiable gate.

4. **Affected surface.** Real paths, not mechanism names. "Wire it into the existing X mechanism" is not a surface. For every file/module/nix attribute you name: `grep`/`find` the repo and confirm it exists today, or — if it's new — that its name and location follow an existing sibling's convention (a `pkgs/*.nix` pin, a `checks.${system}` entry alongside the others, a `modules/services/*.nix` file next to its neighbors). Karnak checks this by searching the repo, not by trusting the prose; do that search yourself first.

5. **Dependencies/conflicts.** Don't stop at "does this issue name its blockers." Grep sibling open issues carrying the same Plan label for an Affected-surface path that overlaps yours. Two issues editing the same file with no stated ordering is exactly the kind of conflict Karnak will flag — and it's cheap to catch by reading the sibling issues yourself before Karnak has to.

**Also, cross-check drift:** the issue was scaffolded from a Notion Plan at a point in time. Before resubmitting, confirm the issue's scope, phase, and any decision it cites (e.g. an adopt/reject call) still match the Plan's *current* state — not the state when the issue was written. A stale cross-reference is a dependencies/conflicts fail even when the issue text alone reads fine.

**If you find a genuine gap,** fix the body and leave one line (in the body or a short comment) saying what changed, so a human skimming the thread can see the delta — don't pad it into prose that repeats the issue back to itself. If nothing is missing, don't edit for the sake of touching the ticket; go straight to relabeling.

This does not replace Karnak — it exists so Karnak's opinion is the one that catches genuine judgment calls (is this fixture design actually sound, does this threshold reflect real risk), not the same mechanical gaps every time.
