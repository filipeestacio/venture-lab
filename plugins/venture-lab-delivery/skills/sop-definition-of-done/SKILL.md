---
name: sop-definition-of-done
description: The Definition of Done standard for plans/tasks/issues -- item-specific criteria plus the standard gate. Load when defining or checking whether something is "done".
---

Every plan, task, and issue carries an explicit Definition of Done with two parts:

1. **Item-specific criteria** — what is concretely true when THIS item is done.
2. **The standard gate** (always also applies): CI green (tests / build / lint / typecheck); code + security review passed; no secrets in the diff / PR / logs; deployed where Done requires (production promotion stays human-gated).

"Done" = the acceptance criteria are met — **not necessarily deployed**. The common terminal is a merged PR that has been activated, but some issues are done at merge, at a delivered artifact, or a recorded decision. When Done does reach production, **a human is the required gate** — agents never self-approve, self-merge, or self-promote. No implicit done: if you cannot write a Definition of Done, it is not ready to start.

Distinguish this from the **Definition of Ready** (the criteria a Plan meets before a human can flip it to `Approved`) — Ready is about *starting*, Done is about *finishing*.
