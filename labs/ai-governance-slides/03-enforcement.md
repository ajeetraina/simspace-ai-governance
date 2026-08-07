<!--
layout: section
-->

# Enforcement

Three sandbox protections close the agent's blast radius: network, filesystem, credentials.

---

## Network: three curls, three outcomes

From inside a governed sandbox:

```bash terminal-id=demo
curl https://api.anthropic.com -sS -o /dev/null -w "anthropic: %{http_code}\n"
curl https://paste.ee -sS -o /dev/null -w "paste.ee: %{http_code}\n"
curl https://example.com -sS -o /dev/null -w "example.com: %{http_code}\n"
```

::terminal{id=demo height=280}

Note: **404 is the success signal.** Any non-403 means the request reached the
origin - allowed. 403 means the sbx proxy refused it. anthropic 404 (allowed),
paste.ee 403 (deny exfiltration), example.com 403 (default-deny).

---

<!-- layout: stats -->

## The proxy decided all three

:::stat{value="404" accent=green label="allow AI services"}
`api.anthropic.com` reached the origin
:::

:::stat{value="403" accent=red label="deny exfiltration"}
`paste.ee` refused at the proxy
:::

:::stat{value="403" accent=amber label="default-deny"}
`example.com` - no rule covers it
:::

Note: Same command, three destinations, one decision point. The
`O=GoProxy untrusted MITM proxy Inc` certificate is the proxy openly identifying
itself: it terminated TLS, matched the rule, and returned 403.

---

## Filesystem: the mount that never happens

Running as you on the host, the agent can read every secret on disk. Put it in a sandbox and add an org filesystem policy:

:tag[allow]{accent=green} `~/workdemo/**`  ·  :tag[deny]{accent=red} `~/.ssh/**`, `~/.aws/**`, `~/.docker/config.json`

:::fragment
```bash no-run-button
sbx run shell ~/workdemo/test ~/.ssh:ro   # → 403 at CREATION
```
:::

:::fragment
The denied mount is checked **at sandbox-creation time**. The sandbox never starts, so `~/.ssh` never exists inside it. No race, no partial read.
:::

Note: This is the strongest boundary because it fires *before* the agent runs.
The agent that freely read your secrets a moment ago now can't even mount them.

---

<!--
layout: split
theme: dark
-->

# Credentials: the key it uses but never holds

<!-- region -->

### Inside the sandbox

```bash no-run-button
echo $ANTHROPIC_API_KEY
# proxy-managed
```

A **sentinel**, not a key. There is no live secret anywhere in the box.

<!-- region -->

### On the wire

The host-side proxy matches the destination, reads the **real** key from the keychain, and injects the `Authorization` header - per request.

Note: Even for calls the agent is *supposed* to make, a prompt injection has
nothing usable to exfiltrate, because there's no usable key inside the box.

---

<!-- layout: quote -->

> Network, filesystem, credentials - the agent can't reach, read, or leak what it shouldn't.

That's the blast radius fully contained. Now the *other* problem from the opener: the vulnerable image.
