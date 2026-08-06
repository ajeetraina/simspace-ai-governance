# Credential Isolation

```text no-run-button
   MicroVM (sandbox)                     Host - sbx proxy                Service
   ANTHROPIC_API_KEY = proxy-managed ─▶  1 match destination → service   api.anthropic.com
   (sentinel only)                       2 read real key (keychain)      (sees real key)
                                         3 inject Authorization header ─────────▶
```

*The agent authenticates without ever holding the key: it sees only the
`proxy-managed` sentinel, while the proxy reads the real secret from the host and
injects it per request.*

The filesystem demo blocked the agent from **reading** secrets off disk. But agents
legitimately need credentials - `claude` has to call `api.anthropic.com`. So:

> *If the agent can't read my keys, how does it authenticate to services it's
> allowed to use?*

The answer is **credential isolation**: the real secret never enters the sandbox.
A host-side proxy injects it per request; the sandbox sees only a sentinel.

> [!NOTE]
> This is a **different control** from network/filesystem rules. Those are
> admin-governed org policies (`ORIGIN: remote`). Credential isolation is a
> sandbox runtime protection you configure **developer-side** with `sbx secret`,
> the OS keychain, or OAuth. There's no Admin Console toggle for it.

## Step 1 - See the sentinel inside a sandbox

```bash
sbx run shell ~/workdemo/creds
```

Look at the credential the `claude` agent would use:

```bash
echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY"
```

The variable exists - tools expect it - but its value is `proxy-managed`, not a
real key. There is no live secret anywhere in the sandbox.

## Step 2 - Watch the proxy inject the real credential

Still inside, make a request to an allowed AI service:

```bash
curl https://api.anthropic.com -sS -o /dev/null -w "anthropic: %{http_code}\n"
```

It reaches Anthropic (404). Now exit and inspect the proxy log on the **host**:

```bash
exit
```

```bash
sbx policy log
```

The request to `api.anthropic.com` is logged as `forward` (allowed). The proxy
matched the destination to the `anthropic` service, read your real key from the
host, and injected the `Authorization` header - all without the key ever touching
the sandbox.

## Step 3 - Store the secret in the keychain (preferred)

An env var works but sits in your shell history in plaintext. The hardened path
stores it in the OS keychain instead - keychain secrets take precedence:

```bash
sbx secret set -g anthropic
```

```bash
sbx secret ls
```

## Step 4 - Custom secrets for your own APIs

For an internal API, declare a **custom secret** keyed to a host and env var:

```bash
sbx secret set-custom -g --host api.internal.example.com --env INTERNAL_API_KEY
```

Inside the sandbox, `INTERNAL_API_KEY` shows a placeholder; the proxy substitutes
the real value on requests to the matching host.

## Read the results

| What | Where the secret lives | What the sandbox sees |
| --- | --- | --- |
| `ANTHROPIC_API_KEY` | Host keychain / env | `proxy-managed` sentinel |
| Allowed API call | Injected by proxy on the wire | Never the raw value |
| Custom secret | Host, keyed to host + env | Placeholder; substituted per request |
| SSH key | Host SSH agent | Can sign, can't read the key |

## The three sandbox protections together

- **Network egress** - the agent can't reach unapproved destinations
- **Filesystem access** - the agent can't mount unapproved paths
- **Credential isolation** - the agent can't see the secrets it uses

Even for the calls the agent is *supposed* to make, a prompt injection can't
exfiltrate a usable key - because there is no usable key inside the box.

That's the agent's **blast radius** fully contained - network, filesystem, and
credentials. Now for the *other* problem from the opener: the vulnerable image the
agent built. Next: **Hardened Images (DHI)**.
