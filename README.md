# Implementation Team — Agents

How the Implementation Team works: who does what, how work moves, and where each agent’s rules live.

**Purpose:** Deliver child-welfare / foster-care / juvenile-justice software from requirements through verified release—with clear handoffs, least-privilege data handling, and no skipped quality gates. **Team-Docs** (cross-cutting) packages design docs, test execution reports, and monthly status for proof of work and invoicing.

## How they execute

```
Rich / intake
     │
     ▼
 Barb BA ──work packages + Slack user stories (project-solution)──► Dave Dev ──CI-green artifacts──► Don DevOp ──deploy evidence──► Tony Tester
     │                                                              ▲                                                      ▲
     └──test scenarios──────────────────────────────────────────────┼──────────────────────────────────────────────────────┘
                                                                    │
                         Paul PM (oversees status, priorities, blockers)
                         Team-Docs (cross-cutting: design docs, test reports, monthly status for proof/invoicing)
```

| Handoff | From → To | What moves |
|---------|-----------|------------|
| Work packages | Barb → Dave | Buildable slices: problem, scope, actors, requirements, acceptance criteria, data-sensitivity notes |
| Slack user-story tasks | Barb → Dave | Pick-up tasks in the **project-solution** workspace: As a… / I want… / So that…, acceptance criteria, links/refs to package detail (synthetic/redacted only) |
| Test scenarios | Barb → Tony | Business-level scenarios mapped to acceptance criteria (not executable scripts) |
| Deployable artifacts | Dave → Don | CI-green git SHA + immutable artifact (image digest, AMI, Lambda version—never `:latest`) |
| Deploy evidence | Don → Tony | Verified serving version, health/soak proof, change record, rollback path |
| Status / progress | All → Paul | Completions, blockers, decisions needed; Paul keeps the pipeline moving |
| Approved → assign | Paul → Dave | After a task is **approved**, Paul assigns it to Dave (aligned with Barb’s **project-solution** pick-up stories) |
| Design source | Barb → Team-Docs | Work packages / requirements for **design documents** (proof of work) |
| Execution results | Tony → Team-Docs | Pass/fail/block cases and evidence for **test execution reports** |
| Monthly rollup input | Paul → Team-Docs | Completions, ETC, blockers, risk/issue highlights for **monthly status reports** |

Paul does not replace pipeline roles; he tracks risks/issues/ETC, reports daily to **project-solution**, assigns only **approved** work to Dave, and escalates conflicts. Agents escalate to Paul when scope conflicts, policy is missing, or a handoff is blocked.

**Team-Docs** is cross-cutting (like Paul)—not a sequential step between Barb → Dave → Don → Tony. It packages Barb, Tony, and Paul outputs into client/billing evidence artifacts.

## Agents at a glance

| Order | Agent | Role | Rule file | Status |
|------:|-------|------|-----------|--------|
| 1 | Barb BA | Business Analyst | [`.cursor/rules/barb-ba.mdc`](.cursor/rules/barb-ba.mdc) | **Populated** |
| 2 | Dave Dev | Developer | [`.cursor/rules/dave-dev.mdc`](.cursor/rules/dave-dev.mdc) | Scaffold |
| 3 | Don DevOp | DevOps (AWS) | [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc) | **Populated** |
| 4 | Tony Tester | Tester | [`.cursor/rules/tony-tester.mdc`](.cursor/rules/tony-tester.mdc) | **Populated** |
| — | Paul PM | Project Manager (PMP) | [`.cursor/rules/paul-pm.mdc`](.cursor/rules/paul-pm.mdc) | **Populated** |
| — | Team-Docs | Deliverable docs (proof / invoicing) | [`.cursor/rules/team-docs.mdc`](.cursor/rules/team-docs.mdc) | **Populated** |

**Rules status:** Barb, Don, Tony, Paul, and Team-Docs have full operating rules. Dave is a scaffold (purpose + collaboration stubs) until Rich fills it in.

## Per-agent summaries

### Barb BA — [`.cursor/rules/barb-ba.mdc`](.cursor/rules/barb-ba.mdc)

Turns user docs/forms/process maps into **work packages** for Dave, **Slack user-story pick-up tasks** in the **project-solution** workspace, and **test scenarios** for Tony. Domain: government child-related functions. Optimizes for child outcomes and case-worker efficiency; requires least privilege, need-to-know, minimization, and auditability on sensitive data (no real identifiers in Slack). Does not write code or executable tests.

### Dave Dev — [`.cursor/rules/dave-dev.mdc`](.cursor/rules/dave-dev.mdc)

Picks up Barb’s **Slack user-story tasks** from the **project-solution** workspace (backed by her work packages) and turns them into working code; also takes **Paul’s assignments of approved tasks**. Hands **CI-green, immutable artifacts** to Don. Detailed scope, rules, and done criteria still to be defined (scaffold).

### Don DevOp — [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc)

Deploys app and infra to **AWS** under the [AWS Deployment Rules](.cursor/rules/aws-deployment.mdc): confirm account/region/env/role before mutate; plan-then-apply; immutable CI-green artifacts; security guardrails on; rolling/blue-green/canary with rollback ready; hands **deploy evidence** to Tony. Full rules live in those rule files—not duplicated here.

### Tony Tester — [`.cursor/rules/tony-tester.mdc`](.cursor/rules/tony-tester.mdc)

Turns Barb’s **test scenarios** into granular **test cases** (tasks associated to the main user story in **project-solution**), executes them against Don’s **deployed** environment using deploy evidence, and records discrete **pass / fail / block**—covering happy path and anomaly paths—**before** sign-off or closing the Slack story as complete. Synthetic/redacted data only. Full rules in his rule file.

### Paul PM — [`.cursor/rules/paul-pm.mdc`](.cursor/rules/paul-pm.mdc)

**PMP-certified** PM: tracks **risks**, **issues**, and **ETC**; posts **daily status** to **project-solution**; polls the task list every **15 minutes** for **approved** work and **assigns approved tasks to Dave Dev**. Logs risks/issues in **Project Risks and Issues**. Oversees Barb → Dave → Don → Tony without authoring packages, code, deploys, or tests. Supplies monthly rollup input to **Team-Docs**. Full rules in his rule file.

### Team-Docs — [`.cursor/rules/team-docs.mdc`](.cursor/rules/team-docs.mdc)

Cross-cutting (like Paul): produces **design documents** (from Barb), **test execution reports** (from Tony), and **monthly status reports** (with Paul’s input) for **proof of work and invoicing**. Not a sequential pipeline step. Synthetic/redacted only; references **project-solution** / **Project Risks and Issues** by name without inventing credentials. Full rules in the rule file.
