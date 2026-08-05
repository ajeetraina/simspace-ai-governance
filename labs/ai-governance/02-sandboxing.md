# Sandboxing the Agent

```text no-run-button
   Host machine
   ┌───────────────────────────────────────────────┐
   │  sbx daemon (proxy + policy, injects real key) │
   │  ~/.ssh · ~/.aws · ~/.docker   NOT mounted      │
   │  ┌──────────── MicroVM (sandbox) ───────────┐  │
   │  │  Agent   ANTHROPIC_API_KEY = proxy-managed │  │
   │  │  workspace ~/workdemo  (mounted)          │  │
   │  └───────────────────────────────────────────┘  │
   └───────────────────────────────────────────────┘
        real key stays on host ─▶ injected on the wire ─▶ api.anthropic.com
```

*Same agent, now inside a MicroVM. Only `~/workdemo` is mounted, the credential
dirs never enter the box, and the API key it uses is a sentinel.*

The fix for the blast radius: stop running the agent on bare metal and run it
inside a **sandbox** — an isolated MicroVM where the agent only sees the directory
you hand it, never touches the rest of your host, and never holds your real key.

## Step 1 — Confirm sbx is ready

```bash
sbx version
```

Make sure you're logged in so org policy and secrets are available:

```bash
sbx login
```

## Step 2 — Store your provider key as a governed secret

You'll run Claude inside the sandbox, authenticated to Anthropic. Store the key:

```bash
sbx secret set -g anthropic
```

Verify it's stored (the value is masked):

```bash
sbx secret ls
```

> [!IMPORTANT]
> Secrets live in your **OS keychain** and are injected at the network proxy —
> *after* the request leaves the sandbox. The agent inside never sees the raw API
> key. That's the credential-isolation guarantee (covered in depth later).

## Step 3 — Run the agent, sandboxed

Create a workspace and launch the agent **inside** a sandbox:

```bash
mkdir -p ~/workdemo
```

```bash
sbx run claude .
```

You land in the agent — but now it's inside a MicroVM that only mounted *this*
directory. Ask it the same question as the Problem Statement:

```prompt
Search the host for API keys, cloud credentials, and SSH private keys — check ~/.aws, ~/.ssh, ~/.docker, and any .env files. Show me what you found.
```

This time it comes up **empty**. Your `~/.ssh`, `~/.aws`, and `~/.docker` were
never mounted into the sandbox, so they don't exist inside it. The blast radius
from the Problem Statement is gone.

Leave the sandbox:

```bash
exit
```

## What you've set up

- `sbx` verified and logged in
- Your provider's key stored as a governed secret (keychain, never in the sandbox)
- Your agent running inside an isolated sandbox instead of on bare metal

That's the shift: from an agent that runs *as you* to one that runs *inside a
boundary*. What decides how wide that boundary is — which paths, which hosts,
which tools — is **policy**. Next: **The Policy Model**.
