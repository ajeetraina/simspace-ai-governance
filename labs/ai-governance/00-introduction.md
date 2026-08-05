# Why AI Governance

```text no-run-button
   Docker Hub Org  ──define policy once──▶  Three pillars  ──▶  AI agent
   (your org)                               1 Sandbox policies    contained
                                            2 MCP tool governance  by policy
                                            3 Audit + visibility
```

*Define policy once in your Docker Hub org, and three pillars keep the agent
inside the guardrails. This lab focuses on Pillar 1 (and gives you a taste of 2 & 3).*

AI agents — Claude, Copilot, Cursor, custom MCP servers — run with the **same
blast radius as the developer running them**: your filesystem, your secrets, your
network, your everything. That's fine when the agent does what you expect. It's a
disaster when:

- A prompt-injected agent uploads SSH keys to `paste.ee`
- A misconfigured MCP server exfiltrates source code to an unknown destination
- An agent acting on hallucinated instructions pushes a malicious commit to `main`
- A coding agent reads your `.env` and posts it to the model API alongside your code

"Don't let agents do that" doesn't scale — developers want agents, and they'll
find a way. The right answer is **guardrails around the agent's execution
environment** so it physically *cannot* exceed its scope. That's AI governance.

> [!NOTE]
> Everything in this lab is **simulated** — no real Docker, `sbx` daemon, or
> network. Every learner sees the exact same allow/deny decisions, with nothing
> to install. The commands, outputs, and policy behaviour mirror the real
> [Docker AI Governance](https://www.docker.com/products/ai-governance/) product.

## Set your organization

Most commands and links below substitute `$$org$$` for your Docker Hub org. Set it
once here:

:variableDefinition[org]{prompt="Which Docker Hub organization will you use?"}

## The three pillars

| Pillar | What it controls | Enforced |
| --- | --- | --- |
| **1 · Sandbox policies** | Network allowlists, filesystem mount rules, credential isolation | At the proxy and mount layer |
| **2 · MCP tool governance** | Which MCP servers and tools agents may use | At the MCP gateway |
| **3 · Audit + visibility** | Structured event per policy decision (user, time, rule) | Written to a SIEM-ready log |

## What this lab covers

| Section | What you'll do |
| --- | --- |
| **Product Catalog** | **Start here** — an agent containerises a real app; the image is full of CVEs and it ran with your full access |
| The Policy Model | See how org policies flow to every developer's sandbox |
| Network Enforcement | Contain the agent's egress — three `curl`s, three outcomes |
| Filesystem Isolation | The agent can't read or mount your secrets (403 at sandbox creation) |
| Credential Isolation | The real API key never enters the sandbox |
| Hardened Images (DHI) | Fix the opener's CVEs — swap the base, re-scan, they collapse |
| MCP Governance | Register servers behind one governed gateway; gate tools with Cedar |
| Putting It All Together | **The capstone** — stop one rogue agent's four attacks in one sandbox |
| Observability & Audit | The visibility half, plus governance-as-code (the API) |

By the end you'll have a defensible, end-to-end enforcement story you can walk a
security team through. Head to **Product Catalog** — where an agent does something
real, and shows you exactly what needs governing.
