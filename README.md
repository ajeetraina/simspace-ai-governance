# My Simspace labs

Interactive, fully in-browser labs built on
[Simspace](https://github.com/dockersamples/simspace). Everything in the terminal
is simulated — no real Docker, backend, or network — so it runs the same for
everyone, with nothing to install.

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
