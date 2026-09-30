# Barb BA — durable email Automation (create checklist)

Create this at [cursor.com/automations](https://cursor.com/automations) (or Agents Window → Automations). There is **no create API** from agents—only lookup via `get-automation` after you save.

Prepared: 2026-09-23T01:24:37Z

## Settings

| Field | Value |
|-------|--------|
| **Name** | Implementation Team — Barb BA email monitor |
| **Trigger** | Scheduled · every **15 minutes** (or cron `*/15 * * * *`) |
| **Repository** | **None** (mailbox / MCP only; no code edits) |
| **Tools** | **Gmail** MCP enabled (required). Sign in / auth as `barbba.cursor@gmail.com` if prompted. |
| **Memories** | Leave **on** (fallback if Project store path is missing on a run) |
| **Project** | Prefer linking to **Implementation Team** Project if the UI offers it (so `/cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/` is available) |

## Instructions (paste as-is)

```text
You are the unattended email monitor for Barb BA (Business Analyst).

Mailbox: barbba.cursor@gmail.com
Canonical rules: /cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/docs/agents/barb-ba.md
Pattern: /cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/docs/agent-email-monitoring.md
State: /cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/internal/email-monitor/barb-ba-state.md
Project store: /cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/

Do this wake only:
1. If Gmail MCP namespaceStatus is needsAuth, call mcp_auth once. If auth fails/times out, update the state file with the blocker and end (do not invent credentials).
2. Confirm authenticated identity is barbba.cursor@gmail.com (or that this MCP session is the team binding for that mailbox). If wrong account, stop and log for Rich.
3. Read the state file for last-processed cursor; fetch new/unread mail only via Gmail MCP (search_threads for unread/new since last cursor).
4. For each new message, follow the Email section in barb-ba.md: classify question vs directive; reply or intake; synthetic/redacted only. Questions = ask-mode only (no pipeline side effects). Directives = normal Barb intake; blocking acceptance/scope questions → **original requester** (hold WP/stories); Paul only for pipeline priority / conflicting stakeholders.
5. Update the state file (UTC time, ids processed, counts, errors). No secrets in the file. If the Project store path is unavailable, write the same cursor fields into automation Memories instead and note that in the reply.
6. Do not commit to git unless Rich asked. Do not expand into unrelated work.
7. End the turn when the poll is done (or blocked on auth).
```

## After you save

1. Copy the automation **URL** (includes UUID) from the browser.
2. Paste it back into chat (or into `docs/agent-email-monitoring.md` registry + `internal/email-monitor/barb-ba-state.md`).
3. Confirm **Gmail** MCP is connected for runs as `barbba.cursor@gmail.com`.
4. Optionally disable or leave the bridge `subscribe_timer` (`sub_3364a027-2428-423a-85d4-6da2e88a6933`) — Automation is the durable path; bridge is optional redundancy until the first green Automation run.

## Why this step is manual

Agents can only **look up** Automations (`cursor-cloud` `get-automation` by UUID). Creating/editing is dashboard / Agents Window / local `/automate` skill only.
