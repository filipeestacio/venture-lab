---
name: sop-address-review
description: The contract for responding to a code review (Karnak or a human) BEFORE you push a fix -- enumerate, decompose, address-or-refute, test each, re-verify, reply, re-request. Load when a reviewer has left findings. The mirror of sop-pre-pr-review.
---

Load this when a reviewer — **Karnak** or a human — has left findings on your PR and you are about to push a fix. It is the mirror of `sop-pre-pr-review`: that gate runs *before* the review; this contract runs *after* it. The reviewer re-reviews the **code**, not your claim, so "I think I addressed it" is worth nothing until the change proves it. Landing flow: `sop-engineering-flow`.

**1. Enumerate every finding.** List each point the reviewer raised. A consolidated review comment usually bundles several — a risk verdict, then N findings. Miss none.

**2. Decompose each finding into its sub-actions.** One finding can name more than one change ("deny it *here* AND widen the pattern *there* AND add a test"). Write out the parts. Half-doing a multi-part finding is the exact failure this contract exists to prevent.

**3. Address or refute each — never silently drop.** For every finding and every sub-action: fix it, or state plainly why it is not a real problem. A finding you disagree with is answered with a reason, not ignored. "Unsure" is not a licence to skip.

**4. Add a regression test per finding, where testable.** The test is what turns *addressed* into *proven*. If a finding is genuinely not testable (docs, naming), say so explicitly.

**5. Re-run the reviewer's own reproduction.** If the finding named a repro — a command, an input, a scenario — run exactly that and confirm it now passes.

**6. Build / verify the actual artifact, not an eval.** A clean eval / type-check can still ship a broken build (a bundler can inline an import and mask a file the package never shipped). Formatter / linter clean.

**7. Reply per-finding.** On the PR, answer each finding: what you changed (or why you refuted it) and the test that covers it. This reply is for the **human** and the audit trail — the reviewer does not grade it; it re-reviews the new code.

**8. Re-request review — it is an EVENT, not a message.** Push the fix; the re-review fires from a `PR updated:` wake, not from you @-messaging the reviewer. Do not argue a verdict with the reviewer: a disagreement over a finding is adjudicated by the **human**, who is the only approver. Author never merges own work.

Never treat your own belief that you are done as the terminal condition — the loop ends on a clean (🟢 Green) re-review or the human's merge, not on your say-so. That misplaced confidence is what a silent half-fix looks like from the inside.
