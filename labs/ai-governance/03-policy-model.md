# The Policy Model

```text no-run-button
   Source of truth              Sync                 Enforcement
   ┌──────────────┐                                  ┌──────────────────┐
   │ Admin Console│──┐                            ┌─▶│ every sandbox     │
   │ (UI)         │  ├─▶ Org policy ──login/reset─┤  │ per-request eval: │
   │ Governance   │  │   network · filesystem     │  │  deny? → block    │
   │ API          │──┘   (cached by sbx daemon)   └─▶│  allow? → forward │
   └──────────────┘                                  │  else → default-  │
                                                     │        deny block │
                                                     └──────────────────┘
```

*Authored once (UI or API), synced to the daemon at login, cached, and applied to
every sandbox. Each request is evaluated **deny → allow → default-deny**.*

## Where policies live

Policies for `$$org$$` live in one control plane, authored **two ways** — both
write to the same source of truth:

1. **Docker Hub Admin Console (UI)** — point and click at
   [app.docker.com/accounts/$$org$$](https://app.docker.com/accounts/$$org$$) →
   **AI governance**. Best for a human making a one-off change.
2. **Docker AI Governance API** — the same control plane over HTTP. Best for
   governance-as-code: version control, CI, admin tooling. (See the final section.)

Only org admins can modify policies. Developers **cannot override them locally**.
That's the point.

## How policies reach developers

When a developer runs `sbx login` (or `docker login`) with org credentials, the
client fetches the org's AI governance policies. The local `sbx` daemon caches
them and applies them to every sandbox on that machine. See what's active:

```bash
sbx policy ls
```

The `Governance: managed by $$org$$` header plus a sync timestamp is your proof
governance is live. To force an immediate refresh after an admin changes a policy:

```bash
sbx policy reset
```

## How rules evaluate

Rules are evaluated **per request** at the sandbox network proxy:

1. Does any **deny** rule match? → Block (403)
2. Does any **allow** rule match? → Forward
3. Otherwise → **Default-deny** (block)

This means:

- **Explicit deny always wins.** A deny for `paste.ee` blocks it even if a broader
  allow exists.
- **Default-deny** means you don't enumerate every bad destination. Not explicitly
  allowed = blocked.
- A `0.0.0.0/0` catch-all allow **defeats** this model — it permits everything.

## Local vs remote

```bash
sbx policy ls --include-inactive
```

| `ORIGIN` | Meaning |
| --- | --- |
| `local` | Defaults shipped with sbx, or rules you added locally |
| `remote` | Pulled from your org's Admin Console — authoritative, un-overridable |

When the org sets a policy for a rule type, local rules of that type go
**inactive**: `corporate policy takes precedence`. The CISO has the wheel.

> [!NOTE]
> Network and filesystem rules flow from the admin. **Credential isolation** —
> keeping real API keys out of the sandbox — is a sandbox runtime protection you
> configure developer-side. It complements these policies (its own section later).

Now let's prove it works end-to-end — starting with **Network Enforcement**.
