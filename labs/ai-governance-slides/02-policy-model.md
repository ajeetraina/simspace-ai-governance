<!--
layout: section
-->

# The policy model

Authored once, synced to every developer, applied to every sandbox.

---

## Define once, enforce everywhere

Policies for `$$org$$` live in one control plane, authored **two ways** that write to the same source of truth:

:tag[Admin Console]{accent=blue} point-and-click at **app.docker.com → AI governance**. Best for a human making a one-off change.

:tag[Governance API]{accent=green} the same control plane over HTTP. Best for governance-as-code: version control, CI, admin tooling.

:::fragment
Only org admins can modify policies. Developers **cannot override them locally**. That's the point.
:::

---

## How rules evaluate: deny → allow → default-deny

Every request is checked at the sandbox proxy, in order:

:::fragment
1. Does any **deny** rule match? → **block (403)**
:::

:::fragment
2. Does any **allow** rule match? → **forward**
:::

:::fragment
3. Otherwise → **default-deny (block)**
:::

:::fragment
**Explicit deny always wins**, and **default-deny** means you never have to enumerate every bad destination. Not allowed = blocked.
:::

Note: A `0.0.0.0/0` catch-all allow defeats this entire model - it permits
everything. Deleting it is what turns default-deny on.

---

## Watch policy sync to the machine

```bash terminal-id=demo
sbx login
```

::terminal{id=demo height=280}

Note: `sbx login` with org credentials fetches the org's AI governance policies;
the local daemon caches them and applies them to every sandbox. `local` rules of
a governed type go inactive - "corporate policy takes precedence." The CISO has
the wheel.
