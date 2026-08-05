# Product Catalog

```text no-run-button
   Host ~/workdemo/catalog-service-node (source) ══bind mount══▶ MicroVM (sandbox)
                                                                  Claude (autonomous)
   edits land on host (git diff)  ◀── edits ──                   sandbox's OWN Docker
   allow: anthropic · npm         ◀── proxy ──▶  approved hosts   (Testcontainers here)
   deny:  paste.ee · ~/.ssh · host Docker  ── blocked
```

*A real autonomous agent on a real service: it edits bind-mounted source (the diff
lands on your laptop), builds and tests in the sandbox's own Docker daemon, and
reaches only approved hosts — host Docker, SSH keys, and off-policy destinations
all stay out of reach.*

The earlier demos proved governance with synthetic tests. This proves it on a
**real application** with a **real autonomous coding agent**: the
[Product Catalog service](https://github.com/dockersamples/catalog-service-node) —
a Node.js + Express API backed by Postgres, S3, and Kafka. The network and
filesystem policies from earlier are the only thing between "autonomous" and
"uncontained."

## Step 1 — Clone into the allowed path

The `allow workdemo` rule covers `~/workdemo/**`, so the sandbox can mount it:

```bash
git clone https://github.com/dockersamples/catalog-service-node
```

The repo ships a **known bug** — a Kafka message drops the `upc` field on product
creation — plus a Testcontainers integration suite that proves the fix. A clear
goal with real tests: the ideal autonomous task. Peek at the culprit:
:filelink[PublisherService.js]{path="catalog-service-node/src/services/PublisherService.js"}

## Step 2 — Give the agent its credential

The proxy injects it, so the key never enters the sandbox:

```bash
sbx secret set -g anthropic
```

## Step 3 — Launch the autonomous agent

```bash
sbx run --name catalog claude
```

This creates the sandbox (the filesystem allow rule lets the catalog directory
mount) and drops you into Claude **inside the microVM**. Confirm it has its own
Docker daemon — where all builds and tests run:

```bash
sbx exec catalog -- docker version --format "{{.Server.Version}}"
```

That version is from a daemon that is **not** your host's.

## Step 4 — Hand it the goal

```prompt
There's a bug where the Kafka message published on product creation drops the upc
field. Find it, fix it, and prove the fix by running the integration test suite.
Iterate until the tests pass.
```

The agent reads the source, edits it (changes appear in your host tree via the
bind mount), and runs the Testcontainers suite — spinning up throwaway Postgres,
Kafka, and LocalStack **inside the sandbox's Docker daemon** — until it's green.

## Step 5 — Review the change on your laptop

Leave the agent and drop back to your host shell:

```bash
exit
```

Because the source is bind-mounted, the agent's edit is already in your local tree:

```bash
git diff
```

You review a small, contained diff — the agent never had access to your host's
Docker, your SSH keys, or any off-policy network destination while producing it.

## What you demonstrated

This is the **golden path**: `sbx run claude`, the agent works fully autonomously
on a real service, and produces a reviewable diff — with no approval prompts and no
ability to step outside the policy boundary.

- **Compute** is contained — builds/tests run in the sandbox's Docker, not yours
- **Network** is contained — the agent reaches only approved hosts
- **Filesystem** is contained — it mounts only approved paths; writes land only where allowed
- **Review** stays human — the diff surfaces on your laptop before anything merges

The payoff: **define policy once, and a real autonomous agent on a real codebase
stays inside the lines — automatically.** Next, the capstone puts all four
boundaries against one rogue agent at once.
