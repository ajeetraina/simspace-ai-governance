# Filesystem Enforcement Demo

```text no-run-button
   Three sbx run attempts ─▶ mount check at CREATION (cached policy):
     ~/workdemo/test-1              allow workdemo   ─▶ sandbox starts ✅
     ~/workdemo/test-2 + ~/.ssh:ro  deny credentials ─▶ 403, never starts
     /tmp/outside-workdemo          no rule → deny   ─▶ 403, never starts
```

*Filesystem rules are checked **at sandbox creation**, not at read time. An allowed
path mounts and the sandbox starts; a denied or unlisted path fails with 403 and
the sandbox never exists.*

**A key difference from network:** the sandbox refuses to be *created* with a
denied mount, instead of letting it in and blocking reads later. The denied mount
never exists inside the sandbox — no race, no partial read.

## The filesystem policy

The `setup-policies.sh` run (or the Admin Console) already created two rules:

```text no-run-button
allow workdemo     (allow, read+write)  ~/workdemo/**
deny credentials   (deny,  read+write)  ~/.ssh/**, ~/.aws/**, ~/.config/gcloud/**,
                                         ~/.kube/config, ~/.docker/config.json
```

Confirm they reached your machine (look for `ORIGIN: remote`):

```bash
sbx policy ls --include-inactive
```

> [!TIP]
> Even with leftover allow rules from other experiments, **`deny` always wins** —
> `deny credentials` still blocks `~/.ssh` in Test 2 below.

## Test 1 — Allowed workdir

```bash
sbx run shell ~/workdemo/test-1
```

The sandbox starts and you land at the prompt — the mount is real and read-write.
The `allow workdemo` rule permits `~/workdemo/**`. Leave it:

```bash
exit
```

## Test 2 — Allowed workdir + a denied extra mount

Try to also mount your SSH directory into the sandbox:

```bash
sbx run shell ~/workdemo/test-2 ~/.ssh:ro
```

The sandbox **never starts**. `deny credentials` blocks `~/.ssh:ro` at creation.

> [!TIP]
> Open **Settings** (⚙) and turn **deny credentials** off, then re-run the command
> above — now the sandbox **starts** and mounts `~/.ssh`. That's exactly the
> exposure the rule exists to prevent; toggle it back on to restore the guardrail.

## Test 3 — Unallowed workdir (default-deny)

```bash
sbx run shell /tmp/outside-workdemo
```

Again **403 at creation**. No allow rule covers `/tmp/outside-workdemo`, so
default-deny applies. (On macOS `/tmp` resolves to `/private/tmp` — the policy
engine evaluates the canonical path.)

## Read the results

| Test | Workdir | Extra mount | Outcome | Why |
| --- | --- | --- | --- | --- |
| 1 | `~/workdemo/test-1` | none | Sandbox starts | Covered by `allow workdemo` |
| 2 | `~/workdemo/test-2` | `~/.ssh:ro` | 403, no sandbox | Blocked by `deny credentials` |
| 3 | `/tmp/outside-workdemo` | none | 403, no sandbox | No policy → default-deny |

| Layer | Pattern |
| --- | --- |
| Network (previous section) | curl gets 404 (allowed) or 403 (denied) |
| Filesystem (this section) | sandbox creation succeeds (allowed) or fails 403 (denied) |

## What you demonstrated

The policy engine **prevents the sandbox from being created** with a denied mount.
Enforcement happens *before* the agent ever runs — stronger than runtime
filtering. Combined with the network demo, Pillar 1 is proven end-to-end:

- **Network egress** — agent can't reach unapproved destinations (proxy intercept)
- **Filesystem access** — agent can't even mount unapproved paths (creation-time denial)

Next: **Credential Isolation** — how the agent uses the secrets it *is* allowed to,
without the real values ever entering the sandbox.
