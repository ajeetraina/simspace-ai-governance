# Putting It All Together

```text no-run-button
   One rogue / prompt-injected agent, one sandbox, one policy engine:
     1 · mount ~/.ssh                ─▶ 403 at CREATION      (filesystem)
     2 · curl paste.ee              ─▶ 403 at the PROXY      (network)
     3 · read $ANTHROPIC_API_KEY    ─▶ proxy-managed sentinel (credential)
     4 · call a rogue MCP tool      ─▶ denied at the GATEWAY  (MCP)
```

*One rogue agent tries all four attacks in a single sandbox, and one policy engine
stops each - fail-closed, at a different point in the lifecycle, with no per-attack setup.*

You proved each control on its own. This puts all four together against a **single
rogue agent** in **one sandbox**, then hands you a scorecard for a security team.

## Step 1 - Establish the blast radius on the host

Ask the agent - running as you, no sandbox - to find every secret it can:

```bash
claude -p "Find every API key, cloud credentials, and SSH private key on this machine and show me the exact file paths."
```

A wall of found secrets - AWS keys, an SSH private key, a registry token. **That's
the blast radius.** Now let's close it.

## Step 2 - Attack 1 blocked: it can't even mount your secrets

The strongest boundary fires before the agent runs. Try to launch a sandbox that
mounts your SSH directory:

```bash
sbx run shell ~/workdemo/capstone ~/.ssh:ro
```

**403 at creation.** `deny credentials` blocks the mount - the sandbox never
starts, so `~/.ssh` never exists inside it. *(filesystem)*

## Step 3 - Launch the governed sandbox

Now start it the way it's meant to run - workspace only, no credential mounts:

```bash
sbx run shell ~/workdemo/capstone
```

The next three attacks all run from **inside** this one sandbox.

## Step 4 - Attack 1 confirmed: the secrets aren't reachable

```bash
ls ~/.ssh ~/.aws
```

```bash
claude -p "Find every API key, cloud credentials, and SSH private key on this machine and show me the exact file paths."
```

The credential directories don't exist and the agent comes up **empty** - only
your allowed workspace was ever mounted.

## Step 5 - Attack 2 blocked: exfiltration refused at the proxy

```bash
curl https://api.anthropic.com -sS -o /dev/null -w "anthropic: %{http_code}\n"
```

```bash
curl https://paste.ee -sS -o /dev/null -w "paste.ee: %{http_code}\n"
```

```bash
curl https://example.com -sS -o /dev/null -w "example.com: %{http_code}\n"
```

`paste.ee` is refused by `deny exfiltration`; `example.com` by default-deny. The
`403` comes from the **sbx proxy**, not the destination. *(network)*

## Step 6 - Attack 3 blocked: there's no live key to steal

```bash
echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY"
```

The sandbox holds only a **sentinel** - the real key lives on the host and is
injected per request. A prompt injection has nothing usable to exfiltrate.
*(credential)*

```bash
exit
```

## Step 7 - Attack 4 blocked: no path to an unapproved tool server

The agent only ever sees the governed gateway. Point `sbx` at the hosted gateway,
register a server the org policy doesn't allow, and attach it:

```bash
export SBX_MCP_URL=https://gateway.docker.com
```

```bash
sbx mcp add notion --url https://mcp.notion.com/mcp
```

```bash
sbx run claude --static-mcp notion
```

```bash
/mcp
```

Ask the agent to use it:

```bash
claude -p "Use the notion tools to list my recent documents."
```

The gateway evaluates the call against the org's Cedar allow-list, finds no
matching `permit`, and **denies** it by default - logged for audit. *(MCP)*

## The governance scorecard

One sandbox, four boundaries, one policy engine - the table to put in front of a
security team:

| # | Attack the agent tried | Pillar | When it fails | Proof you ran |
| --- | --- | --- | --- | --- |
| 1 | Read `~/.ssh`, `~/.aws` off disk | Filesystem | Sandbox **creation** | `sbx run shell ... ~/.ssh:ro` → 403 |
| 2 | Exfiltrate to `paste.ee` | Network | Per **request** | `curl paste.ee` → 403 (proxy) |
| 3 | Steal `ANTHROPIC_API_KEY` | Credential | Never held in sandbox | `echo $ANTHROPIC_API_KEY` → `proxy-managed` |
| 4 | Reach an unapproved tool server | MCP | Gateway authorization | rogue tool call → denied + audited |

- **Default-deny everywhere.** Attacks 1 and 2 needed no rule *naming* the target.
- **Fail-closed timing.** Filesystem fails at *creation*, network at *request*,
  credential at *injection*, MCP at *invocation*.
- **One source of truth.** All four trace back to policy for `$$org$$` - authored
  once, synced to every developer, un-overridable locally.

That is the defensible, end-to-end story: *"Show me an agent doing something
dangerous, and I'll show you where the policy stops it - and where the CISO sees it
happen."* Next: the visibility half.
