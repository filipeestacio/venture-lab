# Page bodies to create verbatim

Substitute `<YOU>` (the runner) and `<PARTNER>` (the engineering owner; default **Filipe**) throughout. Author in NfM.

---

## HOME PAGE BODY — 🧪 Venture Lab

```
<callout icon="💡" color="blue_bg">
	**Venture Lab — a Product workspace.** We do the *thinking* here: research → PRD → phased **Plan**. A Plan's `Status` is the one gate. **Engineering lives entirely with <PARTNER>** (his GitHub, his repos, his execution) — this Notion space never holds code, repos, or issues. When a Plan is **Approved**, it is handed to <PARTNER> to build.
</callout>

## How this space works
- **Plans** (below) is the single home for every venture idea → PRD → phased plan. A Plan's `Status` is the one gate.
- **Research** (below) holds the findings, market/user/tech research and options that feed a Plan. Link a Research doc to its Plan via the relation.
- **Only a human flips a Plan to `Approved`.** Agents draft, research, and refine — they never self-approve.
- When a Plan is **Approved**, `<PARTNER>` picks it up and scaffolds the engineering work on **his own GitHub**. He writes the repo/issue links back into the Plan's `Links`; the Plan `Status` mirrors his execution (`In Progress` → `Done`). Rationale lives here; live execution state lives in <PARTNER>'s GitHub. Cross-link, never hand-mirror.

See the **Operating Model — Product** page below for the full lifecycle, the Definition of Ready, and the handoff.

<!-- The agent embeds/references the two databases and the Operating Model page here. -->
```

---

## OPERATING MODEL BODY — 🧭 Operating Model — Product (Venture Lab)

```
<callout icon="💡" color="blue_bg">
	**TL;DR** — This is a **Product** workspace. We do research → PRD → phased **Plan** in Notion; `Status` is the one gate. **All engineering is <PARTNER>'s** and lives on his GitHub — never in this space. Only a human sets `Approved`; that is the moment a Plan is handed to <PARTNER> to build.
</callout>

<table_of_contents/>

---

## 1. The one rule
Notion is the **Product** tool — thinking, research, decisions, PRDs → *Plans*. **Engineering is out of scope for this workspace**: repos, issues, code, and deploys are owned end-to-end by **<PARTNER>** on his GitHub. This space is authoritative for *why* and *what/why-now* (pre-Approval). Once a Plan is **Approved**, `<PARTNER>` owns *how* and current execution state. The two sides **cross-link** (a URL into his GitHub), never hand-mirror each other's live state.

**Anti-drift:** a given fact lives in exactly one place. Rationale, research, product decisions → here. Implementation detail + execution status → <PARTNER>'s GitHub. The link between them is a URL, not a copy.

## 2. The two databases
- **🤔 Plans** — every venture idea → PRD → phased plan. `Status` is the one gate (§3). The PRD is a *stage of a Plan*, not a separate page — it lives in the Plan's `Product definition (PRD)` toggle.
- **💭 Research** — findings, options, market/user/tech research that inform a Plan. Link each Research doc to its Plan via `Related Plan`.

## 3. Lifecycle & gates
`Draft → In Review → Approved` (human) → *handed to <PARTNER>* → `In Progress → Done`.

<table header-row="true">
<tr><td>Status</td><td>Lives / worked in</td><td>Entry criteria</td><td>Gate to leave</td></tr>
<tr><td>**Draft**</td><td>Notion (here)</td><td>an idea + a Plan page exists</td><td>author judges it worth review</td></tr>
<tr><td>**In Review**</td><td>Notion (here)</td><td>Plan meets the Definition of Ready (§4)</td><td>a human decides to approve</td></tr>
<tr><td>**Approved**</td><td>handoff</td><td>DoR met; a human flips it</td><td>shared with <PARTNER>; he scaffolds on GitHub</td></tr>
<tr><td>**In Progress**</td><td><PARTNER>'s GitHub</td><td>he's building it</td><td>work merged / shipped</td></tr>
<tr><td>**Done**</td><td>back here</td><td>work shipped</td><td>outcome + decisions recorded back in the Plan</td></tr>
<tr><td>**Superseded**</td><td>Notion</td><td>dropped / replaced</td><td>—</td></tr>
</table>

Operational statuses off the happy path — **Backlog**, **Awaiting Action**, **Blocked** — are also on the `Status` field. They do not change the gate: **Approved** is still the only handoff trigger.

## 4. Definition of Ready — a Plan may be **Approved**
Only flip `Status → Approved` when the Plan page contains **all** of:
- [ ] **Problem / goal** — what and why now, in a sentence or two.
- [ ] **Context & findings** — the research that informs it (or a link to a Research doc).
- [ ] **Decision log** — the choices made and the options rejected, with reasons.
- [ ] **Definition of Done (plan-level)** — the outcome that must hold when the whole plan is done.
- [ ] **Phased task breakdown** — phases, each with concrete tasks; every task carries its own **Definition of Done** and named **dependencies**.
- [ ] **Out of scope** — what this explicitly does not do.
- [ ] **Risks / guardrails** — what could go wrong; any safety constraints.

If a task is too vague to write a Definition of Done for, the Plan is not Ready.

## 5. The handoff: Approved → <PARTNER>
**Trigger (human-only).** A human flips `Status → Approved`. An agent never approves a Plan and never creates engineering work.

Then: **share the Plan with <PARTNER>.** He owns everything downstream — he scaffolds the work on his GitHub (his mapping: Phase → Milestone, Task → Issue), and writes the repo/issue links back into the Plan's `Links`. Nobody creates repos, issues, or code in this Notion space. As <PARTNER> executes, the Plan `Status` moves to `In Progress` and then `Done`.

## 6. Closing the loop
When <PARTNER> reports the work shipped, set the Plan `Status → Done` and record the **outcome + any decisions made during execution** back in the Plan. If execution discovers something that changes the plan, it flows **back to this Plan as a decision first** — the product intent must not silently diverge from what got built.

## 7. Collaboration between partners
Both partners keep **parallel Venture Lab workspaces** with this same structure. Either can own a Research doc or a Plan (set `Owner` / `Author`), share the page, and comment on the other's. The workflow is: one prepares research / a PRD / a Plan and owns it; the other comments and picks up the next stage. Ownership and hand-offs are explicit via the person properties and sharing — never by copying pages between accounts.

## 8. Guardrails
- Only a **human** flips a Plan to **Approved**; agents never self-approve.
- **No secrets** (tokens, keys, credentials) in any Notion page.
- **One source of truth per fact**; cross-link, don't duplicate or hand-mirror.
- Engineering — repos, issues, code, deploys — is **<PARTNER>'s**, never scaffolded from here.
```
