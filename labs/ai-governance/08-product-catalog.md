# Product Catalog

```text no-run-button
   Host ~/workdemo/catalog-service-node (source) ══bind mount══▶ MicroVM (sandbox)
                                                                  Claude (autonomous)
   Dockerfile lands on host  ◀── writes ──                       sandbox's OWN Docker
   allow: anthropic · npm    ◀── proxy ──▶  approved hosts        (build + scout run here)
   deny:  paste.ee · ~/.ssh · host Docker  ── blocked
```

*A real autonomous agent on a real service: it writes a Dockerfile (which lands on
your laptop), builds and scans the image in the sandbox's own Docker daemon, and
reaches only approved hosts — host Docker, SSH keys, and off-policy destinations
all stay out of reach.*

The earlier demos proved governance with synthetic tests. This proves it on a
**real application** with a **real autonomous coding agent**: the
[Product Catalog service](https://github.com/dockersamples/catalog-service-node) —
a Node.js + Express API backed by Postgres, S3, and Kafka, with **no Dockerfile
yet**. You'll hand it to the agent to **containerise** and **scan for
vulnerabilities** — the whole time contained by the network and filesystem policies
you configured earlier.

## Step 1 — Clone into the allowed path

The `allow workdemo` rule covers `~/workdemo/**`, so the sandbox can mount it:

```bash
git clone https://github.com/dockersamples/catalog-service-node
```

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
Docker daemon — where the build and scan will run:

```bash
sbx exec catalog -- docker version --format "{{.Server.Version}}"
```

That version is from a daemon that is **not** your host's.

## Step 4 — Hand it the goal: containerise the app

```bash
claude -p "Containerise this Node.js app for production: read package.json and src/, write a Dockerfile, and build the image as catalog-service:latest."
```

The agent reads the project, writes a `Dockerfile` (the change appears in your host
tree via the bind mount), and builds the image **inside the sandbox's Docker
daemon** — never touching your host Docker. Review what it wrote:

```bash
cat catalog-service-node/Dockerfile
```

## Step 5 — Fetch the vulnerabilities

Now scan the freshly built image with Docker Scout — also running in the sandbox:

```bash
docker scout quickview catalog-service:latest
```

```bash
docker scout cves catalog-service:latest
```

The agent picked the convenient `node:20` base — which drags in dozens of CVEs from
packages the app never uses. Scout surfaces them (critical/high/medium/low), and
most trace back to that fat base image. A **hardened base image** (Docker Hardened
Images) would cut them sharply — the natural next step, and exactly the kind of
change you'd let the agent make and then review.

Leave the agent and drop back to your host shell:

```bash
exit
```

## Step 6 — Read the results

| Concern | Where it happened | Why it was safe |
| --- | --- | --- |
| Agent reasoning + writing the Dockerfile | Sandbox microVM | Isolated VM; only the mounted workspace is visible |
| `docker build` + `docker scout` | Sandbox's own Docker daemon | Never touches host Docker — no blast radius |
| Reaching `api.anthropic.com` + npm registry | Through the proxy | Allowed by network policy; everything else denied |
| Exfil to `paste.ee` / mounting `~/.ssh` | Blocked | Network deny + filesystem deny (earlier sections) |
| The Dockerfile itself | Bind-mounted source | Lands in your local tree for human review |

## What you demonstrated

This is the **golden path**: `sbx run claude`, the agent works fully autonomously
on a real service — containerising it, building, and scanning for vulnerabilities —
and produces a reviewable Dockerfile, with no approval prompts and no ability to
step outside the policy boundary.

- **Compute** is contained — build and scan run in the sandbox's Docker, not yours
- **Network** is contained — the agent reaches only approved hosts
- **Filesystem** is contained — it mounts only approved paths; writes land only where allowed
- **Review** stays human — the Dockerfile and the CVE report surface on your laptop

The payoff: **define policy once, and a real autonomous agent on a real codebase
stays inside the lines — automatically.** Next, the capstone puts all four
boundaries against one rogue agent at once.
