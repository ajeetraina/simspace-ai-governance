# Observability, Audit & API

```text no-run-button
   sbx daemon (every decision) ──▶ daemon.log (JSONL)  ──▶ live dashboard :8090
                               └──▶ auditkit/*.jsonl   ──▶ Splunk / Datadog / Sentinel
                                    + user · org · session
```

*Every decision the daemon makes is written as structured JSONL. This section reads
it, watches it live, and ships it to a SIEM — the visibility half of the story —
then closes the loop with governance-as-code via the API.*

## Part 1 — The traffic log

`sbx policy ls` tells you the *rules*; `sbx policy log` tells you what they actually
*did* — every host your sandboxes reached, split into blocked vs allowed, with counts:

```bash
sbx policy log
```

`api.anthropic.com` appears under **Allowed** via `forward`; `paste.ee` (matched
`deny exfiltration`) and `example.com` (`default-deny`) under **Blocked** — each
attributed to the rule that decided it. For scripting, emit JSON:

```bash
sbx policy log --json
```

## Part 2 — The live dashboard

Every policy decision is also written to a structured `daemon.log` (JSONL). Start
the observability kit, which tails it in real time:

```bash
docker compose --profile with-gateway up -d --build
```

```bash
open http://localhost:8090
```

Generate a few decisions and watch three rows appear live — an allow for
`api.anthropic.com`, an explicit deny for a denylisted host, and an implicit
(default-deny) block. Read the raw log directly with `jq` (reference):

```bash no-run-button
LOG="$HOME/Library/Application Support/com.docker.sandboxes/sandboxes/sandboxd/daemon.log"
jq -c 'select(.msg == "governance policy evaluation" and .allowed == false)' "$LOG" | tail -20
```

> [!NOTE]
> The raw `daemon.log` answers *what* was decided and *why*, but has **no user
> identity** — user attribution is the job of `auditkit` (next).

## Part 3 — SIEM-grade audit (`auditkit`)

Docker AI Governance writes a separate, purpose-built audit log — one sealed JSONL
event per decision, **with the signed-in user, org, and session on every record**:

```json no-run-button
{
  "timestamp": "2026-05-28T19:15:00Z",
  "category": "AUDIT_CATEGORY_EVALUATION",
  "decision": "AUDIT_DECISION_DENY",
  "username": "jordandoe",
  "user_email": "jordandoe@example.com",
  "org_name": "Acme Inc",
  "audit_session_id": "8a3bc076-79d0-4502-baf3-cc6ad35fb578",
  "resource_id": "example.com:443",
  "deny_reason": ["no applicable policies for op(action=net:connect:tcp, ...)"],
  "action_type": "network_egress"
}
```

Two operational rules for the shipper (Splunk UF, Filebeat, LogScale): **only
collect `*.jsonl`** (skip the half-written `.tmp`), and **retention is yours**.

> [!IMPORTANT]
> Audit logging is a **paid** part of Docker AI Governance and only activates when
> `$$org$$` enforces a centralized policy. No governance → no audit records.

## Part 4 — Governance-as-code (the API)

Everything you did in the Admin Console has an HTTP equivalent. The same helper
from the Network demo drives the Governance API to provision policies — ideal for
version control and CI. Scope it to one domain, or run it bare for both:

```bash
bash setup-policies.sh network
```

Minting an admin token is a one-time step (a JWT from a Personal Access Token):

```bash no-run-button
curl -fsS -X POST https://hub.docker.com/v2/users/login \
  -H "Content-Type: application/json" \
  -d '{"username":"<you>","password":"<PAT>"}'   # → { "token": "..." }
```

Both front doors — Console and API — write to the **same** source of truth for
`$$org$$`. Pick whichever fits your workflow.

## What you've built

Across this lab you took the exact blast radius the horror-story agent measured and
closed **all four boundaries** — network, filesystem, credential, MCP — with one
policy engine that fails closed and leaves an audit trail:

- **Define once** — in the Admin Console or via the Governance API
- **Enforce everywhere** — synced to every developer, un-overridable locally
- **See everything** — live dashboard, SIEM-ready `auditkit` with user attribution

That's the defensible, end-to-end enforcement story — one you can now walk a
security team through. 🎉

Learn more at [docker.com/products/ai-governance](https://www.docker.com/products/ai-governance/).
