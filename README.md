# Implementation Team

Cursor project rules for a small software implementation team that delivers child-welfare, foster-care, and juvenile-justice software from requirements through a verified release.

Purpose: clear handoffs, least-privilege handling of sensitive child and family data, and no skipped quality gates. Best value for the child is the goal; case-worker daily efficiency is how the team gets there.

## How they execute

```
Rich / intake
    │
    ▼
Barb BA ── work packages + Slack user stories ──► Dave Dev ── CI-green artifacts ──► Don DevOp ── deploy evidence ──► Tony Tester
    │                                                                                         ▲
    └── test scenarios ───────────────────────────────────────────────────────────────────────┘

Paul PM oversees progress, priorities, blockers, and status.
```

| Handoff | From → To | What moves |
| --- | --- | --- |
| Work packages | Barb → Dave | Buildable slices (problem, scope, actors, ACs, data-sensitivity notes) |
| Slack user stories | Barb → Dave via **project-solution** | Claimable pick-up tasks; the work package stays the source of truth |
| Test scenarios | Barb → Tony | Business/workflow scenarios, not executable scripts |
| Deployable artifacts | Dave → Don | CI-green git SHA + immutable artifact (never `:latest`) |
| Deploy evidence | Don → Tony | Serving version, health/soak, change record, rollback path |
| Status | All → Paul | Completions, blockers, decisions needed |

Rich defines each agent's rules after that agent is established. Agent-rule commits go on `main` unless Rich asks otherwise.

## Agents

| Agent | Role | Status | Rule file |
| --- | --- | --- | --- |
| **Barb BA** | Business Analyst | Populated | [`.cursor/rules/barb-ba.mdc`](.cursor/rules/barb-ba.mdc) |
| **Dave Dev** | Developer | Scaffold | [`.cursor/rules/dave-dev.mdc`](.cursor/rules/dave-dev.mdc) |
| **Don DevOp** | DevOps (AWS) | Populated | [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc) |
| **Tony Tester** | Tester | Scaffold | [`.cursor/rules/tony-tester.mdc`](.cursor/rules/tony-tester.mdc) |
| **Paul PM** | Project Manager | Populated | [`.cursor/rules/paul-pm.mdc`](.cursor/rules/paul-pm.mdc) |

### Barb BA (populated)

Gathers requirements from business documents, forms, process maps, and other intake. Outputs:

- **Work packages** Dave can implement without guessing policy
- **User-story pick-up tasks** in the **project-solution** Slack workspace (`As a` / `I want` / `So that` + acceptance criteria + link to the package)
- **Test scenarios** Tony can turn into executable scripts

Domain constraints: child-centered outcomes, case-worker efficiency, and data protection (least privilege, need-to-know, minimization, auditability; synthetic or redacted samples only; no invented statutes).

### Dave Dev (scaffold)

Turns Barb's work packages into working code and picks up Slack user-story tasks from **project-solution**. Hands CI-green immutable artifacts to Don. Scaffold rule file is in place; detailed scope, rules, and done criteria still to be defined by Rich.

### Don DevOp (populated)

Deploys application and infrastructure code to AWS under the [AWS deployment rules](.cursor/rules/aws-deployment.mdc). Receives CI-green artifacts from Dave; hands deploy evidence to Tony. Does not invent procedures beyond those rules.

### Tony Tester (scaffold)

Turns Barb's test scenarios into executable scripts and runs them against Don's deployed environment using deploy evidence. Scaffold rule file is in place; detailed scope, rules, and done criteria still to be defined by Rich.

### Paul PM (populated)

Oversees the Implementation Team pipeline: tracks risks, issues, and ETC; reports daily status to **project-solution**; and assigns **approved** work to Dave Dev. Does not author packages, code, deploys, or tests.

## Slack pick-up (project-solution)

Barb posts one user-story task per coherent Dave slice in the **project-solution** workspace. Slack is the claimable surface; the work package holds the detail. Do not invent channel IDs or bot credentials. Never post real child, family, or case identifiers.

## AWS deployment rules

Don's always-applied checklist so releases succeed, stay compliant, and avoid unplanned downtime.

The rule file is always applied. Use it as a pre-flight checklist so releases:

- **Succeed** — immutable artifacts, reviewed plans, health-gated rollouts, known rollback
- **Stay compliant** — least-privilege IAM, secrets out of git, encryption, no public data planes
- **Avoid downtime** — rolling/blue-green/canary deploys, backward-compatible changes, and no surprise outages of shared dev/staging/CI

| File | When it applies |
| --- | --- |
| [`.cursor/rules/aws-deployment.mdc`](.cursor/rules/aws-deployment.mdc) | Every Agent chat (`alwaysApply: true`) |
| [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc) | Every Agent chat (`alwaysApply: true`) |

### How to use

1. Keep `.cursor/rules/aws-deployment.mdc` in the repo you deploy from (or copy it into each service repo).
2. In Cursor Agent, ask for a deploy plan, Terraform/CDK change, pipeline edit, or incident hotfix. The rule is injected automatically.
3. Treat the **Hard stops** section as a merge gate. If a change hits one, split the work or schedule a window instead of forcing the deploy.

To apply the AWS rule only when you `@`-mention it, set `alwaysApply: false` in the frontmatter and drop the description.

### What this is not

These rules do not replace IAM permission boundaries, SCPs, AWS Config, or your change-management process. They tell the agent (and the operator) what not to skip.
