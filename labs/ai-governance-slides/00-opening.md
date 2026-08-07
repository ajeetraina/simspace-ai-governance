<!--
layout: title
theme: dark
byline: A 12-minute tour, before you get your hands dirty
-->

# Docker AI Governance

Define once, enforce everywhere. Contain a rogue agent's network, filesystem, credential, and MCP attacks with one policy engine.

Note: This is the slide people photograph. Set the room up: we're going to hand a
real app to an autonomous agent, watch it do something useful *and* something
dangerous, then close every boundary. Everything is simulated - the same
allow/deny decisions for everyone.

---

## An agent runs with *your* blast radius

Claude, Copilot, Cursor, a custom MCP server - they run with the **same access as the developer running them**. That's fine until it isn't:

:::fragment
- A prompt-injected agent uploads SSH keys to `paste.ee`
:::

:::fragment
- A misconfigured MCP server exfiltrates source to an unknown host
:::

:::fragment
- An agent reads your `.env` and posts it to the model API alongside your code
:::

Note: The point of this slide is that the danger isn't hypothetical malware -
it's the agent doing exactly what it was asked, with far more reach than it
needs.

---

<!-- layout: quote -->

> "Don't let agents do that" doesn't scale - developers want agents, and they'll find a way.

The answer is **guardrails around the agent's execution environment**, so it physically *cannot* exceed its scope. That's AI governance.

---

<!-- layout: split -->

# Three pillars, one source of truth

<!-- region -->

### 1 · Sandbox policies

Network allowlists, filesystem mount rules, credential isolation - enforced at the **proxy and mount layer**.

<!-- region -->

### 2 · MCP tool governance

Which MCP servers and tools an agent may call - enforced at the **gateway**.

<!-- region -->

### 3 · Audit + visibility

A structured event per decision - user, time, rule - written **SIEM-ready**.

Note: This deck focuses on Pillar 1, with a real taste of 2 and 3. Define the
policy once in your Docker Hub org; the three pillars keep the agent inside the
guardrails.
