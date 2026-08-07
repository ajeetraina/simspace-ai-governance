<!--
layout: section
-->

# Hardened Images

Govern what the agent is allowed to *build on*.

---

<!-- layout: split -->

# Swap the base, re-scan, watch them collapse

<!-- region -->

:::card{label="before" accent=red variant=outline}
```dockerfile filename=Dockerfile no-run-button
FROM node:20
```
**47 CVEs** — 2C 12H 20M 13L. Fat base, unused packages.
:::

<!-- region -->

:::card{label="after" accent=green variant=fill}
```dockerfile filename=Dockerfile no-run-button
FROM dhi.io/node:20-hardened
```
**2 CVEs** — 0C 0H 1M 1L. Minimal, signed, SBOM + SLSA provenance.
:::

Note: Same app, a fraction of the attack surface. The app code was always fine -
the base was the liability. One `FROM` line, one rebuild, one re-scan.

---

<!-- layout: stats -->

## Same app. A fraction of the surface.

:::stat{value="47 → 2" accent=green label="total CVEs"}
after switching to a hardened base
:::

:::stat{value="0" accent=green label="critical + high"}
essentially eliminated
:::

:::stat{value="1" accent=blue label="line changed"}
the `FROM` in the Dockerfile
:::

---

## Why it's a governance story, not just a smaller image

Each Docker Hardened Image is:

- **Minimal** — no shells or package managers for an agent (or attacker) to abuse
- **Signed + attested** — a signature, an SBOM, and SLSA provenance you can *verify* before it runs
- **Continuously patched** — CVEs remediated at the source; you inherit the fix

:::fragment
That makes "use a hardened base" something an org can **require**, not just recommend - the supply-chain complement to the sandbox policies.
:::

Note: Sandbox policies govern what the agent can *do*. Hardened Images govern what
the agent is allowed to *build on*. Docker also ships a DHI MCP server - which is
the perfect segue into governing the agent's *tools*.
