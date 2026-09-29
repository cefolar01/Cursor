# Implementation Team — Agents

How the Implementation Team works: who does what, how work moves, and where each agent’s rules live.

## Stack

Cloud Agent and local work use this host stack only:

- Ubuntu 24.04
- Node.js v22
- npm 10
- Python 3.12
- Go 1.22
- **No Docker** (no daemon, Dockerfiles, or Compose)

The rule file is always applied. Use it as a pre-flight checklist so releases:

**Unattended email monitoring** (reusable pattern for Barb now, other agents later): [docs/agent-email-monitoring.md](docs/agent-email-monitoring.md). Barb durable Automation checklist: [docs/barb-email-automation-setup.md](docs/barb-email-automation-setup.md).

## How they execute

| File | When it applies |
| --- | --- |
| [`.cursor/rules/aws-deployment.mdc`](.cursor/rules/aws-deployment.mdc) | Every Agent chat (`alwaysApply: true`) |
| [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc) | Every Agent chat (`alwaysApply: true`) |

| Handoff | From → To | What moves |
|---------|-----------|------------|
| Work packages | Barb → Dave | Buildable slices: problem, scope, actors, requirements, acceptance criteria, data-sensitivity notes |
| Slack user-story tasks | Barb → Dave | Pick-up **list records** on **https://cardinalityproject.slack.com/lists/T0C37AFKSJE/F0C35F6198W** (Workspace **T0C37AFKSJE**, List ID **F0C35F6198W**)—never channel messages as the claimable task: As a… / I want… / So that…, acceptance criteria, links/refs to package detail (synthetic/redacted only). Optional brief channel notice pointing at the list record is OK. |
| Test scenarios | Barb → Tony | Business-level **test scenarios on the user story** (list **F0C35F6198W**)—Tony develops validation **test cases** from those scenarios (not executable scripts from Barb) |
| Deployable artifacts | Dave → Don | CI-green git SHA on **`main`** (after feature-branch merge + unit tests in **Childcare_Solution**) + immutable artifact (never `:latest`) |
| Deploy evidence | Don → Tony | Verified serving version, health/soak proof, change record, rollback path |
| Status / progress | All → Paul | Completions, blockers, decisions needed; Paul keeps the pipeline moving |
| Approved → release for execution | Paul → Dave | Paul monitors Slack list **F0C35F6198W** every **15 min**; when a task is **approved**, he **releases it for execution** (assigns Dave on that same list record—no second queue) and includes newly approved/released items in daily **project-solution** status |
| Design source | Barb → Team-Docs | Work packages / requirements for **design documents** (proof of work) |
| Execution results | Tony → Team-Docs | Pass/fail/block cases and evidence for **test execution reports** |
| Monthly rollup input | Paul → Team-Docs | Completions, ETC, blockers, risk/issue highlights for **monthly status reports** |

Paul does not replace pipeline roles; he tracks risks/issues/ETC, polls list **F0C35F6198W** every **15 minutes** for approval, **releases approved work for execution** (assigns Dave on that list—no second queue), reports daily to **project-solution** (including newly approved/released items), and escalates conflicts. Agents escalate to Paul when scope conflicts, policy is missing, or a handoff is blocked.

**Team-Docs** is cross-cutting (like Paul)—not a sequential step between Barb → Dave → Don → Tony. It packages Barb, Tony, and Paul outputs into client/billing evidence artifacts.

## Agents at a glance

| Order | Agent | Role | Rule file | Status |
|------:|-------|------|-----------|--------|
| 1 | Barb BA | Business Analyst | [`.cursor/rules/barb-ba.mdc`](.cursor/rules/barb-ba.mdc) | **Populated** |
| 2 | Dave Dev | Developer | [`.cursor/rules/dave-dev.mdc`](.cursor/rules/dave-dev.mdc) | **Populated** |
| 3 | Don DevOp | DevOps (AWS) | [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc) | **Populated** |
| 4 | Tony Tester | Tester | [`.cursor/rules/tony-tester.mdc`](.cursor/rules/tony-tester.mdc) | **Populated** |
| — | Paul PM | Project Manager (PMP) | [`.cursor/rules/paul-pm.mdc`](.cursor/rules/paul-pm.mdc) | **Populated** |
| — | Team-Docs | Deliverable docs (proof / invoicing) | [`.cursor/rules/team-docs.mdc`](.cursor/rules/team-docs.mdc) | **Populated** |

**Rules status:** Barb, Dave, Don, Tony, Paul, and Team-Docs have full operating rules in their `.cursor/rules/*.mdc` files.

## Per-agent summaries

### Barb BA — [`.cursor/rules/barb-ba.mdc`](.cursor/rules/barb-ba.mdc)

Turns user docs/forms/process maps into **work packages** for Dave, **Slack user-story pick-up list records** on **F0C35F6198W** (workspace **T0C37AFKSJE**; never channel messages as the claimable task), and **test scenarios** for Tony; also **monitors email unattended** at **barbba.cursor@gmail.com** (Automation / `subscribe_timer` + Gmail MCP—see [docs/agent-email-monitoring.md](docs/agent-email-monitoring.md); questions → ask-mode replies; directives → normal intake). Domain: government child-related functions. Optimizes for child outcomes and case-worker efficiency; requires least privilege, need-to-know, minimization, and auditability on sensitive data (no real identifiers in Slack or email). Includes a curated **Public reference sources** catalog (federal / states / territories) for process-design reference—see [`.cursor/rules/barb-ba.mdc`](.cursor/rules/barb-ba.mdc). Does not write code or executable tests.

### Dave Dev — [`.cursor/rules/dave-dev.mdc`](.cursor/rules/dave-dev.mdc)

Implements Barb’s work packages and **approved** Slack **list** pick-ups on **F0C35F6198W** (workspace **T0C37AFKSJE**) in **https://github.com/cefolar01/Childcare_Solution** as **Express + SQLite** / **React + Vite** on **Node.js v22** / **npm 10** (Ubuntu 24.04; Python 3.12 / Go 1.22 available; **no Docker**). Takes **Paul’s assignments only after approved**. Develops each feature on a **new branch** in the Cursor **childcare_solution** cloud environment; after **unit tests pass**, merges to **`main`**; hands Don a **CI-green SHA on `main`** + immutable artifact (never `:latest`). Feasibility feedback to Barb; status to Paul; commits only when asked. Full rules in his rule file.

### Don DevOp — [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc)

Deploys app and infra to **AWS** in default region **us-east-2** (console: **https://us-east-2.console.aws.amazon.com/console/home?region=us-east-2**; CLI/API still use `us-east-2`) under the [AWS Deployment Rules](.cursor/rules/aws-deployment.mdc) on **Ubuntu 24.04** with **Node.js v22** / **npm 10**, **Python 3.12**, and **Go 1.22** (**no Docker**): confirm account/env/role (and region) before mutate; plan-then-apply; immutable CI-green artifacts (AMI / Lambda / other CI IDs — never an image built with Docker on this machine); security guardrails on; rolling/blue-green/canary with rollback ready; hands **deploy evidence** to Tony. Agents call AWS through the SigV4 [AWS MCP Server](docs/aws-mcp-server.md) proxy in [`.cursor/mcp.json`](.cursor/mcp.json). Does not invent account IDs or credentials. Full rules live in those rule files—not duplicated here.

### Tony Tester — [`.cursor/rules/tony-tester.mdc`](.cursor/rules/tony-tester.mdc)

Works from tasks on Slack list **F0C35F6198W** (workspace **T0C37AFKSJE**—same list Barb’s pick-ups and Paul’s approval use). Uses the **test scenario on the user story** to develop granular **test cases** (tasks associated to that main story on the same list—no second queue), executes them against Don’s **deployed** environment using deploy evidence, and records discrete **pass / fail / block**—covering happy path and anomaly paths—**before** sign-off or closing the Slack story as complete. Synthetic/redacted data only. Full rules in his rule file.

### Paul PM — [`.cursor/rules/paul-pm.mdc`](.cursor/rules/paul-pm.mdc)

**PMP-certified** PM: tracks **risks**, **issues**, and **ETC**; posts **daily status** to **project-solution** (including newly **approved** / **released for execution** items); polls Slack list **F0C35F6198W** (workspace **T0C37AFKSJE**—same list Barb’s pick-ups use) every **15 minutes** for approval state and **releases approved work for execution** by assigning **Dave Dev** (no second queue). Logs risks/issues in **Project Risks and Issues**. Oversees Barb → Dave → Don → Tony without authoring packages, code, deploys, or tests. Supplies monthly rollup input to **Team-Docs**. Full rules in his rule file.

### Team-Docs — [`.cursor/rules/team-docs.mdc`](.cursor/rules/team-docs.mdc)

Cross-cutting (like Paul): produces **design documents** (from Barb), **test execution reports** (from Tony), and **monthly status reports** (with Paul’s input) for **proof of work and invoicing**. Not a sequential pipeline step. Synthetic/redacted only; references **project-solution** / **Project Risks and Issues** by name without inventing credentials. Full rules in the rule file.
