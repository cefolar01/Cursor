# Agent email monitoring (reusable)

Reusable pattern for **true unattended** inbox monitoring by Implementation Team agents. Barb BA is the first onboarded instance; other agents reuse this template later.

**Credentials:** never invent app passwords, API keys, or tokens. Use the **named mailbox address** plus **Gmail MCP auth** (OAuth via `mcp_auth`) only. Do not put secrets in this doc, agent rules, Slack, or logs.

## Goals

- Poll or wake without requiring someone to open chat.
- Classify each inbound message as **question** vs **directive**.
- Reply professionally with the most relevant known project info (synthetic/redacted only).
- Route directives through that agent’s normal intake rules; keep questions in ask mode (no pipeline side effects until confirmed).
- Log enough for Paul / Rich to see that monitoring is alive—without secrets or real case identifiers.

## Mechanisms discovered (this environment)

| Mechanism | What it does | Durable / unattended? | Create from agent? | Notes |
|-----------|--------------|----------------------|--------------------|-------|
| **Cursor Automations** (scheduled / cron) | Spawns a Cloud Agent run on a schedule or event | **Yes — preferred** | No create API in tools; create at [cursor.com/automations](https://cursor.com/automations) (or Agents Window / `/automate` locally). Lookup only via `cursor-cloud` `get-automation` (UUID) | Docs: [Automations](https://cursor.com/docs/cloud-agent/automations). Enable **Gmail** MCP on the automation. Prefer **no repository** unless the poll must edit code. |
| **Subscriptions `subscribe_timer`** | Recurring `cron` or `delaySeconds` follow-up on **this** Cloud Agent conversation | Partial — lives with the agent conversation; **expires** (server-assigned); re-subscribe if still needed | Yes (`cursor-subscriptions`) | Good bridge / same-run loop. List/remove via `list_subscriptions` / `unsubscribe`. Dedupes by `name`. |
| **Gmail MCP** | Read/send mail after OAuth | Required for mailbox I/O | Auth via `Gmail` `mcp_auth` when `needsAuth` | Must authenticate as the agent’s named mailbox (e.g. Barb = `barbba.cursor@gmail.com`). Rich completes the Google OAuth UI if the agent’s `mcp_auth` times out. |
| **Other subscriptions** (Slack, GitHub, Linear) | Event wake-ups | Yes for those sources | Yes | **No Gmail push subscription** in catalog — email uses scheduled poll + Gmail MCP. |

**Preference order:** (1) Cursor Automation with scheduled trigger + Gmail MCP → (2) long-lived agent + `subscribe_timer` poll → (3) never “only when someone opens chat.”

## Pipeline (every wake)

```
Trigger (Automation cron OR subscribe_timer)
        │
        ▼
 Authenticate / confirm Gmail MCP for named mailbox
        │
        ▼
 Fetch new / unread since last cursor (store state file)
        │
        ▼
 For each message: classify Question vs Directive
        │
        ├─ Question  → ask-mode professional reply (no packages / Slack pick-ups / scenarios)
        └─ Directive → agent intake per that agent’s rules file
        │
        ▼
 Log poll result (counts, message ids synthetic-safe, auth errors) → Project store
        │
        ▼
 End turn (timer/automation schedules the next wake)
```

### Classify

- **Question** — seeks information. Draft answer in **ask mode** only. Do **not** start side-effecting pipeline work until confirmed as a directive.
- **Directive** — asks the agent to do work. Process under that agent’s current rules (Barb: work packages → user-story pick-up **list records** on canonical list **F0C35F6198W** — **https://cardinalityproject.slack.com/lists/T0C37AFKSJE/F0C35F6198W**, workspace **T0C37AFKSJE** — never as channel messages → test scenarios). Escalate unclear scope/priority to **Paul PM** (or Rich).
- Mixed: answer the question first; confirm the actionable part before intake.

### Respond / intake

- Professional tone; most relevant **known** info from project context and that agent’s rules.
- Synthetic/redacted examples only—never real child/family/case identifiers.
- No invented credentials, statute numbers, or fake tool IDs.

### Logging (Project store)

Maintain a small state/log under `internal/` (not `docs/`):

- Suggested path: `internal/email-monitor/<agent-slug>-state.md` (e.g. `barb-ba-state.md`).
- Record: last successful poll time (UTC), last message id/thread id processed (opaque Gmail ids OK), counts handled, auth/MCP blockers, automation/subscription id + URL when known.
- Never log message bodies that contain secrets or real sensitive identifiers; summarize (“question about process X”) if needed.

## Onboard another agent

1. **Mailbox** — Rich provisions a named address (e.g. `dave….cursor@gmail.com`). Document the address in that agent’s rules; do not invent passwords.
2. **Rules file** — Add an Email section to `docs/agents/<agent>.md` mirroring Barb’s classify/reply/privacy rules, and point to **this** doc for the unattended trigger.
3. **Gmail MCP auth** — In a Cloud Agent / Automation that will poll, call `Gmail` `mcp_auth` and complete Google OAuth **as that mailbox**. If `needsAuth` / timeout, Rich finishes auth in the Cursor UI.
4. **Automation (preferred)** — At [cursor.com/automations](https://cursor.com/automations):
   - Name: `Implementation Team — <Agent> email monitor`
   - Trigger: scheduled (e.g. every **15 minutes**, or cron `*/15 * * * *`)
   - Tools: enable **Gmail** MCP (and Slack only if that agent’s intake needs it)
   - Repository: **none** unless the poll must change git
   - Instructions: paste the **Automation prompt template** below (fill mailbox + rules path)
   - After save, copy **automation id** + dashboard **URL** into this doc’s registry table and `docs/project-context.md`
5. **Bridge (optional)** — On a dedicated long-lived Cloud Agent, `subscribe_timer` with the same poll prompt (`delaySeconds: 900` or matching cron). Record `subscriptionId` in the state file. Re-subscribe when expired.
6. **State file** — Create `internal/email-monitor/<agent-slug>-state.md` with empty cursor and “auth pending” until the first green poll.
7. **Verify** — Send a synthetic test question to the mailbox; confirm ask-mode reply and a log line. Then a synthetic directive; confirm intake without inventing policy.

## Automation prompt template

Copy into a Cursor Automation (or into `subscribe_timer` `prompt`). Replace placeholders:

```text
You are the unattended email monitor for <AGENT_NAME> (<ROLE>).

Mailbox: <MAILBOX_ADDRESS>
Canonical rules: /cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/docs/agents/<agent-file>.md
Pattern: /cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/docs/agent-email-monitoring.md
State: /cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/internal/email-monitor/<agent-slug>-state.md
Project store: /cursor/stores/bc-e89b3f6b-9d7d-4934-9870-f6430596f626/

Do this wake only:
1. If Gmail MCP namespaceStatus is needsAuth, call mcp_auth once. If auth fails/times out, update the state file with the blocker and end (do not invent credentials).
2. Confirm authenticated identity is <MAILBOX_ADDRESS> (or that this MCP session is the team binding for that mailbox). If wrong account, stop and log for Rich.
3. Read the state file for last-processed cursor; fetch new/unread mail only.
4. For each new message, follow the Email section in the agent rules: classify question vs directive; reply or intake; synthetic/redacted only.
   For Barb directives that create Slack pick-up stories: create list records on https://cardinalityproject.slack.com/lists/T0C37AFKSJE/F0C35F6198W (Workspace T0C37AFKSJE, List ID F0C35F6198W)—never as #project-solution (or other) channel messages. Optional brief channel notice pointing at the list record is OK. Do not invent other list/channel IDs or tokens.
5. Update the state file (UTC time, ids processed, counts, errors). No secrets in the file.
6. Do not commit to git unless Rich asked. Do not expand into unrelated work.
7. End the turn when the poll is done (or blocked on auth).
```

## Barb BA — first instance

| Field | Value |
|-------|--------|
| Mailbox | `barbba.cursor@gmail.com` |
| Rules | Project store `docs/agents/barb-ba.md` (canonical) · in-repo [`.cursor/rules/barb-ba.mdc`](../.cursor/rules/barb-ba.mdc) |
| State file | [internal/email-monitor/barb-ba-state.md](../internal/email-monitor/barb-ba-state.md) |
| Preferred trigger | Cursor Automation @ [cursor.com/automations](https://cursor.com/automations) — **create checklist:** [barb-email-automation-setup.md](./barb-email-automation-setup.md) |
| Automation id / URL | *Pending Rich save — paste UUID/URL after create; checklist [barb-email-automation-setup.md](./barb-email-automation-setup.md)* |
| Bridge trigger | `subscribe_timer` `sub_3364a027-2428-423a-85d4-6da2e88a6933` on [Wire Barb unattended email](https://cursor.com/agents/bc-d176e015-5b83-5d59-b269-5c08f6c0a88c) (name `barb-ba-email-monitor`, 900s) |
| Cadence | Target **every 15 minutes** (`delaySeconds: 900` or cron `*/15 * * * *`) |
| Gmail MCP | **Authenticated** — verified as `barbba.cursor@gmail.com` (read poll OK) |
| Slack pick-up (directives) | **List records** on **https://cardinalityproject.slack.com/lists/T0C37AFKSJE/F0C35F6198W** (Workspace **T0C37AFKSJE**, List ID **F0C35F6198W**)—never channel messages as the claimable task; see [`.cursor/rules/barb-ba.mdc`](../.cursor/rules/barb-ba.mdc) |

### Live monitoring status

- **Rules + pattern:** shipped.
- **Live mailbox I/O:** **active** — Gmail MCP authenticated; verified read as `barbba.cursor@gmail.com` (2026-09-23T01:22:13Z).
- **Unattended wake:** bridge timer **armed** (`sub_3364a027-2428-423a-85d4-6da2e88a6933`). **Durable** Automation still pending Rich create at [cursor.com/automations/new](https://cursor.com/automations/new) — checklist + create packet emailed 2026-09-23T13:08:33Z (`docs/barb-email-automation-setup.md`).
- **Auth note:** Earlier `mcp_auth` calls timed out with no visible prompt in Project/chat; a later `mcp_auth` completed (Rich signed in as Barb). Prefer completing OAuth from [cursor.com/agents](https://cursor.com/agents) MCP controls if a chat prompt does not appear.


## Rich — Gmail auth (Barb)

- **Status:** done for this agent run — mailbox verified as `barbba.cursor@gmail.com`.
- If a future run shows `needsAuth` again (OAuth is per-user / per surface): open [cursor.com/agents](https://cursor.com/agents) → MCP / Gmail, or approve when `mcp_auth` is called; sign in as **`barbba.cursor@gmail.com`**. If no prompt appears in Project chat, use the Agents page MCP controls rather than waiting in-thread.
- After re-auth, confirm `internal/email-monitor/barb-ba-state.md` has a successful poll timestamp.


## Registry (fill as agents onboard)

| Agent | Mailbox | Rules | Automation id / URL | State file | Status |
|-------|---------|-------|---------------------|------------|--------|
| Barb BA | barbba.cursor@gmail.com | Project store `docs/agents/barb-ba.md` · [`.cursor/rules/barb-ba.mdc`](../.cursor/rules/barb-ba.mdc) | *pending Automation* | [barb-ba-state.md](../internal/email-monitor/barb-ba-state.md) | **Gmail auth OK**; bridge timer armed; Automation still pending |
