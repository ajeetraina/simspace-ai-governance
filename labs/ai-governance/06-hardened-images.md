# Hardened Images (DHI)

```text no-run-button
   catalog-service:latest              catalog-service:latest (rebuilt)
   FROM node:20                        FROM dhi.io/node:20-hardened
     47 CVEs  (2C 12H 20M 13L)   ──▶     2 CVEs  (0C 0H 1M 1L)
     fat base, unused packages          minimal · signed · SBOM + SLSA provenance
```

*The agent's image was full of CVEs — almost all from the fat base. Swap in a
**Docker Hardened Image**, rebuild, and re-scan: the vulnerabilities collapse. Same
app, a fraction of the attack surface.*

Now we fix the **other** problem from the opener: the vulnerable image. The agent's
`node:20` base carried dozens of CVEs from packages the app never uses. The fix is a
**Docker Hardened Image (DHI)** — a minimal, continuously-patched, signed base with
an SBOM and SLSA provenance built in.

## Step 1 — Recall the damage

```bash
docker scout cves catalog-service:latest
```

Dozens of CVEs, most tagged `(base image)`. The app code is fine; the **base** is
the liability.

## Step 2 — Have the agent switch to a hardened base

```bash
claude -p "Switch the Dockerfile to a Docker Hardened Image base (a hardened, minimal equivalent of node:20) and rebuild catalog-service:latest."
```

The agent rewrites the `FROM` line to a DHI base and rebuilds. See the change:

```bash
cat catalog-service-node/Dockerfile
```

## Step 3 — Re-scan

```bash
docker scout cves catalog-service:latest
```

The CVE count collapses — the critical and high findings are essentially gone,
because the hardened base ships only what the app needs and is patched upstream.
Same application, a fraction of the attack surface.

## Why DHI is a governance story, not just a smaller image

A Docker Hardened Image isn't only "fewer CVEs." Each one is:

- **Minimal** — no shells, package managers, or unused libraries for an agent (or
  an attacker) to abuse.
- **Signed + attested** — a cryptographic signature, an SBOM, and SLSA provenance
  you can *verify* before it runs.
- **Continuously patched** — CVEs are remediated at the source and the image is
  rebuilt, so you inherit the fix.

That makes "use a hardened base" something an org can **require**, not just
recommend — the supply-chain complement to the sandbox policies you just set:

- **Sandbox policies** (network / filesystem / credential) govern what the agent
  can *do*.
- **Hardened Images** govern what the agent is allowed to *build on*.

> [!NOTE]
> Docker also ships a **DHI MCP server** that exposes hardened-image metadata —
> CVEs, SBOMs, attestations, mirrors — as governed tools. You'll register it and
> apply a read-only-vs-mutating policy to it in the next section, **MCP Governance**.

## What you fixed

The opener's two problems are now both addressed: the agent's **blast radius** is
contained (network, filesystem, credential isolation), and the **image it builds**
is hardened (DHI). Next, govern the **tools** the agent can call: **MCP Governance**.
