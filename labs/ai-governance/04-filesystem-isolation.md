# Filesystem Isolation

```text no-run-button
   Host (no sandbox)                    Sandbox (microVM)
   agent runs as YOU                    agent sees only ~/workdemo
     ├─ reads ~/.ssh   ✗                  ~/.ssh   → not mounted, doesn't exist
     ├─ reads ~/.aws   ✗                  ~/.aws   → not mounted, doesn't exist
     └─ reads .env     ✗                  a denied mount → 403 at CREATION
```

*The agent that containerised your app ran as you - it could read every secret on
disk. Put it in a sandbox and those files simply aren't there; add an org
filesystem policy and a denied mount fails before the sandbox even starts.*

The opener flagged the agent's blast radius. Let's make the **filesystem** half of
it visceral, then close it.

## Step 1 - Prove the agent can read your secrets

Running as you, on the host, the agent has your whole filesystem. Ask it:

```bash
claude -p "Search my home directory for API keys, cloud credentials, and SSH private keys - check ~/.aws, ~/.ssh, ~/.docker, and any .env files. Show me the exact file paths."
```

It reports every credential it found - AWS keys, an SSH private key, a registry
token, an `ANTHROPIC_API_KEY`. Nothing stopped it, because nothing was watching.

## Step 2 - Put it in a sandbox

`sbx` runs the agent inside an isolated **microVM** that only mounts the directory
you hand it. Confirm it's ready and launch the agent in a workspace:

```bash
sbx run claude ~/workdemo
```

Ask it the **same** question:

```bash
claude -p "Search the host for API keys, cloud credentials, and SSH private keys - check ~/.aws, ~/.ssh, ~/.docker, and any .env files. Show me what you found."
```

This time it comes up **empty**. `~/.ssh`, `~/.aws`, and `~/.docker` were never
mounted into the sandbox, so inside it they don't exist. Leave the sandbox:

```bash
exit
```

## Step 3 - Make it enforceable for everyone: the filesystem policy

A sandbox that "just didn't mount" your secrets is good; an **org policy** that
*forbids* mounting them is better - it's enforced for every developer, and checked
**at sandbox-creation time**. The `Labspace AI Governance - filesystem` policy has:

```text no-run-button
allow workdemo     (allow, read+write)  ~/workdemo/**
deny credentials   (deny,  read+write)  ~/.ssh/**, ~/.aws/**, ~/.docker/config.json
```

Confirm it reached your machine (look for `ORIGIN: remote`):

```bash
sbx policy ls --include-inactive
```

## Step 4 - Three mount tests, three outcomes

**Allowed workdir** - the sandbox starts:

```bash
sbx run shell ~/workdemo/test-1
```

```bash
exit
```

**Allowed workdir + a denied extra mount** - try to also mount your SSH keys:

```bash
sbx run shell ~/workdemo/test-2 ~/.ssh:ro
```

**403 at creation.** `deny credentials` blocks `~/.ssh:ro` - the sandbox never
starts, so the keys never exist inside it.

> [!TIP]
> Open **Settings** (⚙) and turn **deny credentials** off, then re-run the command
> above - now the sandbox **starts** and mounts `~/.ssh`. That's exactly the
> exposure the rule prevents; toggle it back on to restore the guardrail.

**Unallowed workdir (default-deny)** - a path no rule covers:

```bash
sbx run shell /tmp/outside-workdemo
```

**403 at creation** again - no allow rule covers `/tmp/outside-workdemo`.

## What you proved

| Test | Workdir | Extra mount | Outcome |
| --- | --- | --- | --- |
| 1 | `~/workdemo/test-1` | none | Sandbox starts (`allow workdemo`) |
| 2 | `~/workdemo/test-2` | `~/.ssh:ro` | 403 - blocked by `deny credentials` |
| 3 | `/tmp/outside-workdemo` | none | 403 - default-deny |

Filesystem rules are checked **at creation**, not at read time - the denied mount
never exists in the sandbox. No race, no partial read. The agent that freely read
your secrets in Step 1 now **can't even mount them**.

Next: the agent still needs *some* credentials to do its job - how does it use them
without ever holding them? **Credential Isolation**.
