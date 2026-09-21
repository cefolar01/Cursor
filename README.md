# Implementation Team — Agents

Cursor agent rules for a small software implementation team that delivers **child-welfare / foster-care / juvenile-justice** software from requirements through a verified AWS release.

**Purpose:** Clear handoffs, least-privilege handling of sensitive child and family data, and no skipped quality gates.

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
```

| Handoff | From → To | What moves |
| --- | --- | --- |
| Work packages | Barb → Dave | Buildable slices: problem, scope, actors, requirements, acceptance criteria, data-sensitivity notes |
| Slack user-story tasks | Barb → Dave | Pick-up tasks in the **project-solution** workspace: As a… / I want… / So that…, acceptance criteria, links/refs to package detail (synthetic/redacted only) |
| Test scenarios | Barb → Tony | Business-level scenarios mapped to acceptance criteria (not executable scripts) |
| Deployable artifacts | Dave → Don | CI-green git SHA + immutable artifact (image digest, AMI, Lambda version—never `:latest`) |
| Deploy evidence | Don → Tony | Verified serving version, health/soak proof, change record, rollback path |
| Status / progress | All → Paul | Completions, blockers, decisions needed; Paul keeps the pipeline moving |

Paul does not replace pipeline roles. He tracks progress, prioritizes, and escalates conflicts. Agents escalate to Paul when scope conflicts, policy is missing, or a handoff is blocked.

## Agents and rule files

| Order | Agent | Role | Rule file | Status |
| ---: | --- | --- | --- | --- |
| 1 | Barb BA | Business Analyst | [`.cursor/rules/barb-ba.mdc`](.cursor/rules/barb-ba.mdc) | **Populated** (`alwaysApply: true`) |
| 2 | Dave Dev | Developer | [`.cursor/rules/dave-dev.mdc`](.cursor/rules/dave-dev.mdc) | Scaffold (`alwaysApply: false`) |
| 3 | Don DevOp | DevOps (AWS) | [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc) | **Populated** (`alwaysApply: true`) |
| 4 | Tony Tester | Tester | [`.cursor/rules/tony-tester.mdc`](.cursor/rules/tony-tester.mdc) | Scaffold (`alwaysApply: false`) |
| — | Paul PM | Project Manager | [`.cursor/rules/paul-pm.mdc`](.cursor/rules/paul-pm.mdc) | Scaffold (`alwaysApply: false`) |

**Rules status:** Barb and Don have full operating rules. Dave, Tony, and Paul are scaffolds (purpose + collaboration stubs) until Rich fills them in. Do not invent their detailed rules.

### Per-agent summaries

**Barb BA** turns intake docs, forms, and process maps into work packages for Dave, Slack user-story pick-up tasks in the **project-solution** workspace, and test scenarios for Tony. Domain is government child-related functions. Optimizes for child outcomes and case-worker efficiency; requires least privilege, need-to-know, minimization, and auditability on sensitive data (no real identifiers in Slack). Does not write code or executable tests.

**Dave Dev** picks up Barb’s Slack user-story tasks from **project-solution** (backed by her work packages) and turns them into working code; hands CI-green, immutable artifacts to Don. Detailed scope, rules, and done criteria still to be defined.

**Don DevOp** deploys app and infra to AWS under the project’s AWS Deployment Rules: confirm account/region/env/role before mutate; plan-then-apply; immutable CI-green artifacts; security guardrails on; rolling/blue-green/canary with rollback ready; hands deploy evidence to Tony.

**Tony Tester** validates the release against Barb’s requirements: turns Barb’s test scenarios into scripts and runs them against Don’s deployed environment using deploy evidence. Detailed scope, rules, and done criteria still to be defined.

**Paul PM** oversees the pipeline: tracks progress across agents, moves work forward, surfaces blockers and status. Does not author packages, code, deploys, or tests. Detailed scope, rules, and done criteria still to be defined.

## AWS deployment rules (Don)

Don’s source checklist is [`.cursor/rules/aws-deployment.mdc`](.cursor/rules/aws-deployment.mdc) (`alwaysApply: true`). Use it so releases:

Picks up Barb’s **Slack user-story tasks** from the **project-solution** workspace (backed by her work packages) and turns them into working code; also takes **Paul’s assignments of approved tasks**. Hands **CI-green, immutable artifacts** to Don. Detailed scope, rules, and done criteria still to be defined (scaffold).

Treat the **Hard stops** section as a merge gate. If a change hits one, split the work or schedule a window instead of forcing the deploy.

**PMP-certified** PM: tracks **risks**, **issues**, and **ETC**; posts **daily status** to **project-solution**; polls the task list every **15 minutes** for **approved** work and **assigns approved tasks to Dave Dev**. Logs risks/issues in **Project Risks and Issues**. Oversees Barb → Dave → Don → Tony without authoring packages, code, deploys, or tests. Full rules in his rule file.
