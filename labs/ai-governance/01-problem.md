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

Section 0 made the argument. This section makes it **visceral**: you'll run a
coding agent with no sandbox — straight on your host, as *you* — and ask it to go
find your secrets. Nothing stops it, because nothing is watching.

## Step 1 — Ask the agent to find your secrets

This is how most people run coding agents today: on the metal, with your full
permissions. Give Claude a one-shot task:

```bash
claude -p "Search my home directory for API keys, cloud credentials, and SSH private keys — check ~/.aws, ~/.ssh, ~/.docker, ~/.config/gcloud, and any .env files. Show me the exact file paths."
```

> [!TIP]
> Prefer another agent? `codex -p "<same prompt>"` behaves identically — the
> point isn't the agent, it's that *nothing constrains it*.

## Step 2 — Read what came back

The agent happily reports every credential it could read — AWS keys, an SSH
private key, your registry token, a GCP OAuth cred, an `ANTHROPIC_API_KEY` from a
`.env`. It read every one of them. There was:

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

That boundary is a sandbox. Next: **Sandboxing the Agent** — you'll run the *same*
agent, ask it the *same* question, and watch the secrets vanish.
