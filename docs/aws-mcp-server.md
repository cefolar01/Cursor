# AWS MCP Server (SigV4)

Agents reach AWS through the managed **AWS MCP Server**, fronted by the local **MCP Proxy for AWS**. The proxy signs requests with the AWS credential chain already on the machine. No access keys, session tokens, or account IDs belong in git.

This replaces the deprecated self-hosted package `awslabs.aws-api-mcp-server`.

## Config

Project file: [`.cursor/mcp.json`](../.cursor/mcp.json)

| Setting | Value |
|---|---|
| Proxy | `mcp-proxy-for-aws==1.7.0` (pinned; do not use `@latest`) |
| Endpoint | `https://aws-mcp.us-east-1.api.aws/mcp` |
| Signing region | `us-east-1` (inferred from the endpoint) |
| Default operation region | `us-east-2` (`--metadata AWS_REGION=us-east-2`) |

The endpoint region and the operation region are different. The managed MCP endpoint is in `us-east-1`. `--metadata AWS_REGION=us-east-2` is the default region for AWS API calls, matching Don’s deploy default. Do not pass `--region us-east-2`; that flag is the SigV4 signing region and would miss the endpoint.

## Credentials

The proxy uses the ambient credential chain (`AWS_PROFILE`, `AWS_CONFIG_FILE`, SSO, or `aws login`). It does not read secrets from `mcp.json`.

On Cursor Cloud Agents the injected profile is `cursor-cloud-agent`. A check on 2026-09-28 showed that profile assumes role **DonDeployRole** (session name `cursor-session`) and can call STS `GetCallerIdentity` through `aws___run_script`.

IDE sessions use whatever profile the Cursor process can see. If `uvx` is installed only under `~/.local/bin`, that directory must be on the PATH Cursor uses.

## Prerequisites

- [uv](https://docs.astral.sh/uv/) so `uvx` is on PATH:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

- AWS CLI 2.32 or later, already signed in (`aws sts get-caller-identity` succeeds) before expecting API tools to work. Documentation tools (`aws___list_regions`, `aws___search_documentation`, `aws___read_documentation`) do not need credentials.

## Where to register it

- **Cursor IDE:** the committed `.cursor/mcp.json` is the project server. Reload MCP after pulling. The first start downloads the pinned proxy and can take a few minutes.
- **Cloud Agents:** register the same stdio command in the MCP dropdown at [cursor.com/agents](https://cursor.com/agents), or for the team under Dashboard → Integrations & MCP. Cloud agents do not treat the repo file as their MCP source. `uv` is not in the base image; add the uv install to the **personal** environment’s install script in the [environment dashboard](https://cursor.com/dashboard/cloud-agents/environments/e/19210eb8-b514-11f1-bb68-864e54d14197). Do not add a repo `.cursor/environment.json` for this — a committed environment file overrides that personal environment, which is what injects the AWS role.

Leave secrets out of the dashboard MCP `env` block when the VM already has `AWS_PROFILE` and `AWS_CONFIG_FILE`. If a cloud run cannot find credentials, set those two variable names in the dashboard server config to the injected values. Do not paste access keys into git or into chat.

## Tools

`aws___run_script`, `aws___search_documentation`, `aws___read_documentation`, `aws___list_regions`, `aws___get_regional_availability`, `aws___retrieve_skill`, `aws___get_presigned_url`, `aws___get_tasks`.

`aws___run_script` is the API path. It runs in a sandbox with `call_boto3` (direct `import boto3` is rejected). `aws___call_aws` is deprecated and is not part of this setup.

## Guardrails

Don’s [AWS Deployment Rules](../.cursor/rules/aws-deployment.mdc) still apply to anything this server does. The proxy forwards calls as the caller’s IAM identity; it does not add a separate deploy approval. Add `--read-only` to the `uvx` args to hide write-capable tools.

To confirm a new session: ask which AWS regions exist, then which IAM identity the session is using. The second call must come back as `DonDeployRole` before any mutating work.
