# Database schemas + ⭐ TEMPLATE bodies

Two databases: **🤔 Plans** and **💭 Research**. Create Plans first, then Research (which relates to it). Property order below is the intended left-to-right order in the default table view.

---

## 🤔 Plans — schema

Icon: `🤔`. Title property: `Name`.

| Property | Type | Details |
|---|---|---|
| `Name` | title | — |
| `Status` | select | **The one gate.** Options + colours (in order): `Backlog` gray · `Draft` default · `In Review` yellow · `Approved` green · `In Progress` blue · `Done` purple · `Awaiting Action` orange · `Blocked` red · `Superseded` brown. Description: "The one gate. Only a human flips to Approved; agents never self-approve." |
| `Owner` | person | default to `<YOU>` |
| `Risk` | select | `Low` green · `Med` yellow · `High` red |
| `Slug` | text | Short kebab id the engineering partner uses to label work (`plan:<slug>`). Description: "Short kebab id; <PARTNER> labels GitHub issues plan:<slug>." |
| `Target date` | date | — |
| `Links` | url | Repo / issues / PRD links (filled in by `<PARTNER>` after handoff) |
| `Research` | relation → 💭 Research | reciprocal side of Research's `Related Plan`; two-way |
| `Created` | created_time | system |
| `Last edited` | last_edited_time | system |

> Note vs. the original: the fleet Plans DB also had a `Systems Design` relation. This portable version omits it — Systems Design is an engineering-architecture stage that lives with `<PARTNER>`, not in this Product workspace.

### 🤔 Plans — ⭐ TEMPLATE page body

Title: `⭐ TEMPLATE` · Status: `Draft`.

```
<callout icon="💡" color="blue_bg">
	**TL;DR** — <the goal / bet in 1–3 sentences> · Current state: <where this stands>.
</callout>

<table_of_contents/>

---

## Problem / goal
<what and why now, in a sentence or two>

## Product definition (PRD) {toggle="true"}
	### Users & the job to be done
	### Requirements — functional
	### Requirements — non-functional
	<performance, security, compliance, cost — the *what must hold*, not the how>
	### UX / key flows

## Context & findings
<the research that informs this — link the 💭 Research docs via the Research relation>

## Decisions
- <decision> — because <reason>; rejected <alternative> because <reason>

## Plan — phases → tasks
### Phase 1 — <name>
- [ ] Task — goal; **Definition of Done** (item-specific criteria); depends on <…>
### Phase 2 — <name>
- [ ] …

## Definition of Done (plan-level)
- <the outcome that must hold when the whole plan is done>

## Out of scope
<what this explicitly does not do>

## Risks / guardrails
<what could go wrong; any safety constraints>

## Handoff to engineering {toggle="true"}
	Once **Approved** (human-only), share this Plan with **<PARTNER>**, who owns all engineering. He scaffolds the work on his GitHub (Phase → Milestone, Task → Issue) and writes the repo/issue links back into `Links`. You do not create repos or issues here. `Status` moves to `In Progress` / `Done` to mirror his execution.
```

---

## 💭 Research — schema

Icon: `💭`. Title property: `Name`.

| Property | Type | Details |
|---|---|---|
| `Name` | title | — |
| `Status` | select | Options + colours: `Draft` default · `In Review` yellow · `Done` green · `Superseded` brown |
| `Author` | person | default to `<YOU>` |
| `Date` | date | — |
| `Topic` | multi_select | `Agents` purple · `Infra` blue · `Product` green · `Market` orange · `Ops` gray |
| `Source links` | url | — |
| `Related Plan` | relation → 🤔 Plans | two-way; reciprocal side is Plans' `Research` property |

### 💭 Research — ⭐ TEMPLATE page body

Title: `⭐ TEMPLATE` · Status: `Draft`.

```
<callout icon="💡" color="blue_bg">
	**TL;DR** — the key finding / recommendation in 2–3 sentences.
</callout>

<table_of_contents/>

---

## Question / motivation
<what are we trying to learn, and why now>

## Key findings
<most important first — the answers, not the journey>

## Evidence & analysis
### <theme 1>
### <theme 2>

## Recommendation / implications
<what we should do about it — feeds a Plan via Related Plan>

## Limitations & open questions
<what we don't yet know; what would change the conclusion>

## Sources
<linked list>

## Raw notes {toggle="true"}
	<working detail, interview notes, dumps>
```
