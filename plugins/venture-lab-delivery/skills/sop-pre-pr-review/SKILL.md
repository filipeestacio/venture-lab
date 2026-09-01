---
name: sop-pre-pr-review
description: Self-review gate to run BEFORE opening a PR -- generic checks plus dispatch into the change-shaped standards. Load before opening a PR.
---

Run this against the diff you are about to push, **before** you open the PR. The human is the **approver**, not the first reviewer — everything this catches is a review round trip you did not spend. Landing flow: `sop-engineering-flow`.

**1. Read your own diff** in full. If you cannot say in one sentence why a hunk is there, it does not belong in this PR.

**2. Dispatch on what the diff touches** — load the matching change-shaped standard and work its checklist. Common shapes:
- a **dependency / lockfile pin** moved → the standard for bumping a pinned input (verify the shipped artifact, not just that it resolves).
- a **secret / credential** added or rewired → the standard for wiring a secret end to end (it encrypts fine and then nothing reads it — that is the failure mode).
- an **issue for a scaffolded Plan** → `sop-issue-contract`.

A project that has these standards as their own skills (e.g. a `*-flow` skill) should name them; load that project skill so the dispatch targets are concrete.

**3. Always:**
- **No secrets anywhere** in the diff, the commit message, or the PR body — tokens, keys, decrypted values. Check the body file too, not just the code.
- **Build the actual artifact, not an eval.** A clean eval / type-check can still ship a broken build — a bundler can inline an import and mask a file the package never shipped.
- Formatter / linter clean; the build passes.
- **One concern per PR**; `main` stays deployable at every step. For a pure refactor, verify the build output is identical.
- **State the restart semantics**: does landing this cycle the running service (code / unit change) or does it need a manual restart (config / env-only change)? Put the answer in the PR body — whoever deploys needs it.
- The Definition of Done is written and met — `sop-definition-of-done`.
- For non-trivial code changes, run a code review and a security review.

**4. Write the body for the approver,** not for yourself: what changed, why it is needed, what you verified and how, what activation will do, and the explicit ask. Pass it with `--body-file` — never inline, so no secret or guard-tripping word lands in argv.
