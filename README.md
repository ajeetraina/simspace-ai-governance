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

The lab lives in [`labs/ai-governance/`](labs/ai-governance/) with 11 sections:

| # | Section | What you do |
| --- | --- | --- |
| 0 | Why AI Governance | The three pillars; set your org |
| 1 | The Problem Statement | Ask an unsandboxed agent (`claude -p`) to find your secrets |
| 2 | Sandboxing the Agent | Run it in a sandbox — the secrets vanish |
| 3 | The Policy Model | How org policy flows: deny → allow → default-deny |
| 4 | Network Enforcement Demo | Three `curl`s, three outcomes (404 / 403 / 403) |
| 5 | Filesystem Enforcement Demo | Denied mounts fail at sandbox creation |
| 6 | Credential Isolation | The real key never enters the sandbox |
| 7 | MCP Governance | One governed gateway; gate tools with Cedar |
| 8 | Product Catalog | A real autonomous agent containerises a real app and scans it for CVEs |
| 9 | Putting It All Together | One rogue agent, four attacks, one policy engine |
| 10 | Observability, Audit & API | The visibility half + governance-as-code |

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
