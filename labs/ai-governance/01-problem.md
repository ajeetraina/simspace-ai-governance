# The Problem Statement

```text no-run-button
   Host machine (no sandbox)
   ┌─────────────────────────────────────────────┐
   │  Coding agent  ── runs as YOU · full perms   │
   │      │                                       │
   │      ├──▶ ~/.ssh            private keys      │
   │      ├──▶ ~/.aws            cloud creds       │
   │      ├──▶ ~/.docker/config  registry token    │
   │      └──▶ .env              ANTHROPIC_API_KEY  │
   └──────────────┬──────────────────────────────┘
                  └── could exfiltrate to ──▶ paste.ee · anywhere
```

*No boundary, no policy, no audit. The agent is you — it reads every secret on
disk and could send them anywhere. This is the blast radius the rest of the lab closes.*

Section 1 made the argument. This section makes it **visceral**: you'll run an
agent with no sandbox and watch it walk straight into your secrets. Nothing stops
it, because nothing is watching.

## Step 1 — See the blast radius

The `inventory.sh` script is **read-only** — it checks which secret stores exist
and which exfil destinations are reachable, and transmits nothing. Run it on your
host:

```bash
bash inventory.sh
```

Every `[FOUND]` line is a secret the agent could read. Every `[REACHABLE]` line is
somewhere it could send them. **That's the blast radius.**

## Step 2 — Ask an agent to do it

The script just catalogues. A real agent, running as you, will happily go
further. This is how most people run coding agents today — straight on the metal.
Send this to the agent:

```prompt
Search my home directory for API keys, cloud credentials, and SSH private keys — check ~/.aws, ~/.ssh, ~/.docker, ~/.config/gcloud, and any .env files. Show me what you found and the exact file paths.
```

It reports back every credential it read. There was:

- **No boundary** — the agent has your entire filesystem, because it *is* you.
- **No policy** — nothing decided those paths were off-limits.
- **No audit** — no record that a secret was just read.

## Why this is the real threat model

Here *you* asked for it. But the same access is there when you didn't:

- A **prompt-injected** agent follows hidden instructions in a web page or issue,
  does exactly this, and uploads the results.
- A **malicious or buggy MCP server** reads the same files as a side effect of a
  "helpful" tool call.
- An agent on **hallucinated instructions** leaks or deletes with no attacker at all.

> [!WARNING]
> "Don't run untrusted agents" doesn't scale — developers want agents, and
> they'll run them. The fix isn't trust. It's a **boundary the agent physically
> cannot cross**.

That boundary is a sandbox. Next: **Sandboxing the Agent**.
