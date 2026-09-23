# AWS deployment rules for Cursor

Cursor project rules for DevOps engineers who deploy application and infrastructure code to AWS.

## Stack

Cloud Agent and local work use this host stack only:

- Ubuntu 24.04
- Node.js v22
- npm 10
- Python 3.12
- Go 1.22
- **No Docker** (no daemon, Dockerfiles, or Compose)

The rule file is always applied. Use it as a pre-flight checklist so releases:

- **Succeed** — immutable artifacts, reviewed plans, health-gated rollouts, known rollback
- **Stay compliant** — least-privilege IAM, secrets out of git, encryption, no public data planes
- **Avoid downtime** — rolling/blue-green/canary deploys, backward-compatible changes, and no surprise outages of shared dev/staging/CI

## Rule file

| File | When it applies |
| --- | --- |
| [`.cursor/rules/aws-deployment.mdc`](.cursor/rules/aws-deployment.mdc) | Every Agent chat (`alwaysApply: true`) |
| [`.cursor/rules/don-devops.mdc`](.cursor/rules/don-devops.mdc) | Every Agent chat (`alwaysApply: true`) |

## How to use

1. Keep `.cursor/rules/aws-deployment.mdc` in the repo you deploy from (or copy it into each service repo).
2. In Cursor Agent, ask for a deploy plan, Terraform/CDK change, pipeline edit, or incident hotfix. The rule is injected automatically.
3. Treat the **Hard stops** section as a merge gate. If a change hits one, split the work or schedule a window instead of forcing the deploy.

To apply the rule only when you `@`-mention it, set `alwaysApply: false` in the frontmatter and drop the description.

## What this is not

These rules do not replace IAM permission boundaries, SCPs, AWS Config, or your change-management process. They tell the agent (and the operator) what not to skip.
