# Product Catalog: An Agent Containerises It

```text no-run-button
   You ── "containerise this app" ──▶  Claude (autonomous)  ──▶  product-catalog:latest
                                                                   │
                    ┌──────────────────────────────────────────────┤
                    ▼                                                ▼
         image is FULL of CVEs                        the agent ran as YOU
         (fat node:20 base)                           network · filesystem · credentials
         → fix later with Hardened Images             → contain it first
```

*Hand a real app to an autonomous agent and it happily containerises it - but the
image it ships is riddled with CVEs, and it did the work with your full blast
radius. Those are the two problems this lab closes.*

Let's start where real work starts: a real service and an agent doing a real task.
The [Product Catalog service](https://github.com/ajeetraina/product-catalog-demo-showcase)
is a Node.js + Express API backed by Postgres, S3, and Kafka - with **no Dockerfile
yet**. You'll hand it to Claude and ask it to containerise it.

## Step 1 - Meet the app

```bash
git clone https://github.com/ajeetraina/product-catalog-demo-showcase
```

Take a look at what the agent will work on:
:filelink[package.json]{path="product-catalog-demo-showcase/package.json"}

## Step 2 - Ask the agent to containerise it

```bash
claude -p "Containerise this Node.js app for production: read package.json and src/, write a Dockerfile, and build the image as product-catalog:latest."
```

The agent reads the project, writes a `Dockerfile`, and builds the image. It works -
nothing failed. Review what it wrote:

```bash
cat product-catalog-demo-showcase/Dockerfile
```

## Step 3 - Scan what it built

Before you ship it, scan the image with Docker Scout:

```bash
docker scout quickview product-catalog:latest
```

```bash
docker scout cves product-catalog:latest
```

## Step 4 - Two problems just surfaced

The agent did the job - and introduced **two** distinct risks:

1. **The image is vulnerable.** It picked the convenient, fat `node:20` base, which
   drags in dozens of CVEs from packages the app never uses. That's a **supply-chain**
   problem - we'll fix it later with **Hardened Images (DHI)**.
2. **The agent ran as *you*.** It had full access to your **network**, your
   **filesystem**, and your **credentials** while doing this. On an ungoverned
   laptop it could just as easily have exfiltrated data to a paste site, read your
   `~/.ssh` keys, or leaked your API key. That's the **blast-radius** problem - and
   we close it *first*.

> [!NOTE]
> "Don't run autonomous agents" doesn't scale - developers want them, and they
> deliver. The answer is **governance**: contain what the agent can touch, and
> harden what it produces. That's the rest of this lab.

## Where we go from here

| Next | Closes |
| --- | --- |
| Policy Model → Network → Filesystem Isolation → Credential Isolation | The agent's **blast radius** - it can't reach, read, or leak what it shouldn't |
| Hardened Images (DHI) | The **vulnerable image** - swap the base, re-scan, CVEs drop |
| MCP Governance | The agent's **tools** - one governed gateway, tool-by-tool policy |
| Putting It All Together · Audit Logging | The **proof** - every decision enforced and logged |

First, the model that makes it all work: **The Policy Model**.
