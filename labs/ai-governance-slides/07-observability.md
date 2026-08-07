<!--
layout: section
-->

# Observability & Audit

Enforcement you can't see isn't governance. Close the loop.

---

<!-- layout: split -->

# Every decision, written down

<!-- region -->

### The traffic log

`sbx policy ls` tells you the *rules*; `sbx policy log` tells you what they actually **did** - every host reached, blocked vs allowed, with counts.

<!-- region -->

### SIEM-grade audit

`auditkit` writes one sealed JSONL event per decision - **with the signed-in user, org, and session** on every record. Ships to Splunk, Datadog, Sentinel.

Note: The raw daemon.log answers *what* and *why*, but has no user identity - user
attribution is auditkit's job. Audit logging is a paid part of the product and
only activates when the org enforces a centralized policy.

---

<!-- layout: default -->

## Governance-as-code: the API

Everything in the Admin Console has an HTTP equivalent on the AI Governance API:

```bash filename=list-policies.sh
curl -X GET https://hub.docker.com/v2/orgs/$$org$$/governance/policies \
  -H "Authorization: Bearer $TOKEN"
```

Both front doors - Console and API - write to the **same** source of truth for `$$org$$`. Pick whichever fits your workflow: point-and-click, or version-controlled in CI.

---

<!--
layout: title
theme: dark
byline: docker.com/products/ai-governance
-->

# Define once. Enforce everywhere. See everything.

Network, filesystem, credentials, hardened images, and MCP - one policy engine that fails closed and leaves an audit trail.

Now open the **Docker AI Governance** lab and prove it yourself. 🎉

Note: Hand off to the lab. Everything they just watched, they now run - the same
scripted commands, the same allow/deny decisions. Reaching this slide marks the
deck complete.
