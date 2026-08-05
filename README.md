# Docker AI Governance — Simspace lab

An interactive, fully in-browser lab that proves how **Docker AI Governance**
policies flow from an org's Admin Console to every developer's sandbox — stopping a
rogue agent's **network, filesystem, credential, and MCP** attacks with one policy
engine. Everything in the terminal is simulated — no real Docker, `sbx` daemon, or
network — so it runs the same for everyone, with nothing to install.

It's a Simspace conversion of the
[labspace-docker-ai-governance](https://github.com/ajeetraina/labspace-docker-ai-governance)
workshop, condensed into a focused, self-contained narrative where every
`sbx`/`curl`/`docker` command actually runs in the simulator.

The lab lives in [`labs/ai-governance/`](labs/ai-governance/) with 10 sections that
follow one arc — *an agent does real work, and you govern what it touches and ships*:

| # | Section | What you do |
| --- | --- | --- |
| 0 | Why AI Governance | The three pillars; set your org |
| 1 | **Product Catalog** | **Start here** — an agent containerises a real app: the image is full of CVEs and it ran with your full access |
| 2 | The Policy Model | How org policy flows: deny → allow → default-deny |
| 3 | Network Enforcement | Three `curl`s, three outcomes (404 / 403 / 403) |
| 4 | Filesystem Isolation | The agent can't read or mount your secrets (403 at creation) |
| 5 | Credential Isolation | The real key never enters the sandbox |
| 6 | Hardened Images (DHI) | Fix the opener's CVEs — swap the base, re-scan, they collapse |
| 7 | MCP Governance | One governed gateway; gate tools with Cedar |
| 8 | Putting It All Together | One rogue agent, four attacks, one policy engine |
| 9 | Observability & Audit | The visibility half + governance-as-code (the API) |

**Interactive extras:** a **Settings** dialog (⚙, next to Reset) exposes the org
policies as live toggles — *AI Governance enforced*, *deny exfiltration*, *deny
credentials* — so learners can flip a rule off and watch enforcement disappear
(`paste.ee` → 200, the `~/.ssh` mount succeeds, the governance header vanishes). The
final section also drives the real [AI Governance
API](https://docs.docker.com/reference/api/ai-governance/) shapes via runnable
`curl` calls (list/create policies).

## About this repo

You edit labs under [`labs/`](labs/) — each lab in its own `labs/<id>/`
directory. The app that runs them is a prebuilt image, and labs are loaded at
runtime, so there's no build step for content. With one lab the app opens it
directly; with several it shows a landing page to choose from.

## Author locally

You only need Docker.

```bash
docker compose up dev              # live preview at http://localhost:5173
docker compose run --rm validate   # validate every lab (fails on errors)
```

Edit the files under `labs/<id>/` and refresh the browser to see changes:

- `labspace.yaml` — title, catalog card, terminals, seed files, sections, variables
- `simulator.yaml` — what each command does (scenarios)
- `*.md` — one file per section of instructions

The `labs.json` catalog is **generated** from each lab's `labspace.yaml` (by the
preview server and by `validate`), so you never write or edit it. Add a lab by
adding a `labs/<new-id>/` directory and running `validate`.

Pin the toolchain to a released version for reproducibility:

```bash
export SIMSPACE_AUTHORING_IMAGE=dockersamples/simspace-authoring:1
```

## Deploy

**GitHub Pages (default):** enable Pages (Settings → Pages → Source: "GitHub
Actions"), then push to `main`. The workflow in
[`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) validates the labs,
generates the catalog, and publishes. Pin `runtime-tag` there to a released
version for a stable site. Pull requests are validated first by
[`.github/workflows/validate.yml`](.github/workflows/validate.yml).

**As a container:** the [`Dockerfile`](Dockerfile) bases on the runtime image,
generates the catalog, and swaps in your labs.

```bash
docker build -t my-lab .
docker run --rm -p 8080:80 my-lab    # http://localhost:8080
```

## Authoring with an AI agent

This repo is set up for agent authoring. In Claude Code, an `authoring-lab` skill
(under `.claude/`) knows the workflow, `docker compose` / `validate-lab` are
pre-allowed, and a hook auto-validates the labs after every edit under `labs/`.
[`CLAUDE.md`](CLAUDE.md) loads the guide automatically.

## Learn more

See [`AGENTS.md`](AGENTS.md) for an authoring cheat-sheet, and the
[Simspace specs](https://github.com/dockersamples/simspace/tree/main/spec) for the
full `simulator.yaml` / `labspace.yaml` / `catalog.md` reference.
