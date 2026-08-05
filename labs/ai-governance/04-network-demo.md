# Network Enforcement Demo

```text no-run-button
   sandbox curl ×3 ──▶ sbx proxy (org network policy) ──▶
        allow AI services  ──▶ api.anthropic.com   404  (reached origin)
        deny exfiltration  ──▶ paste.ee            403  (proxy refused)
        default-deny       ──▶ example.com         403  (proxy refused)
```

*Three `curl`s, three outcomes: allowed traffic reaches the origin (404), the deny
rule blocks paste.ee (403), default-deny blocks anything unlisted (403) — all
decided at the sbx proxy.*

## Step 1 — Define the network policy

Two equivalent paths write the **identical** three rules to `$$org$$`:

**Admin Console (manual)** — open
[app.docker.com/accounts/$$org$$](https://app.docker.com/accounts/$$org$$) →
**AI governance → Network access**, and add:

```text no-run-button
allow AI services      (allow)  api.anthropic.com:443, api.openai.com:443, *.googleapis.com:443
allow Docker services  (allow)  *.docker.com:443, *.docker.io:443, dhi.io:443
deny exfiltration      (deny)   paste.ee, pastebin.com, hooks.slack.com
```

...and delete any `0.0.0.0/0` catch-all allow so default-deny is active.

**API / CLI (scripted)** — a helper wraps the Governance API calls and provisions
both the network *and* filesystem policies in one shot:

```bash
bash setup-policies.sh
```

## Step 2 — Verify policies reached your machine

```bash
sbx policy reset
```

```bash
sbx policy ls
```

You should see the three network rules with `ORIGIN: remote` and several
`default-*` rules marked inactive — the org policy overriding sbx's defaults.

## Step 3 — Spin up a sandbox

Launch a shell inside an isolated microVM whose workspace is under the allowed
`~/workdemo` tree:

```bash
sbx run shell ~/workdemo/scratch
```

Outbound traffic from the sandbox goes through the proxy that enforces org policy.

## Step 4 — Run the three enforcement tests

From inside the sandbox:

```bash
curl https://api.anthropic.com -sS -o /dev/null -w "anthropic: %{http_code}\n"
```

```bash
curl https://paste.ee -sS -o /dev/null -w "paste.ee: %{http_code}\n"
```

```bash
curl https://example.com -sS -o /dev/null -w "example.com: %{http_code}\n"
```

## Step 5 — Read the results

| Destination | Code | What it means |
| --- | --- | --- |
| `api.anthropic.com` | **404** | The proxy **let it through** (`allow AI services`); the bare root has no handler so Anthropic replies 404. Any non-403 reply proves it reached the origin. |
| `paste.ee` | **403** | The **sbx proxy refused** it — `deny exfiltration`. paste.ee never got the connection. |
| `example.com` | **403** | No allow rule covers it → **default-deny**. |

> [!NOTE]
> **`404` is the success signal, not an error.** What matters is **404 vs 403**:
> any non-`403` (`404`/`401`/`405`) means you *reached* the origin → allowed;
> `403` means the sbx proxy refused it.

> [!TIP]
> **Prove the policy is what's blocking it:** open **Settings** (the ⚙ gear,
> top-right) and turn **deny exfiltration** off, then re-run the `paste.ee` curl —
> it now returns `200`. Toggle it back on and it's `403` again. Same command, same
> destination; the *only* variable is the org policy.

## Step 6 — See the proxy refusal up close

```bash
curl -v https://paste.ee
```

The `O=GoProxy untrusted MITM proxy Inc` certificate is the proxy openly
identifying itself: it terminated TLS, inspected the request inside the tunnel,
matched `deny exfiltration`, and returned `403` — paste.ee never received anything.

Leave the sandbox:

```bash
exit
```

## What you demonstrated

1. **One source of truth** — policies defined once for `$$org$$` (Console or API)
2. **Automatic propagation** — every logged-in developer inherits them
3. **Real enforcement** — the proxy blocked the deny and unscoped destinations,
   allowed the permitted one
4. **No developer override** — local rules went inactive in favour of org rules

Next: the filesystem half of the same model.
