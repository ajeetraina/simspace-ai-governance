# MCP Governance

```text no-run-button
   Agent  ── /mcp ──▶  ONE mcp-gateway  ──▶  backends (local-wiki, DHI, ...)
                       (SBX_MCP_URL points here)
                       every tool call: authenticated · authorized (Cedar) · audited
```

*The agent talks to one aggregated `mcp-gateway`; every backend sits behind it, and
every tool call is checked against the org's Cedar policy. That single governed
endpoint is the whole point of Pillar 2.*

The `sbx mcp` subcommand registers MCP servers, fronted by the **Docker MCP
Gateway**, that your sandboxed agents can call - and the org governs *which* tools
they may invoke.

## The one concept: `SBX_MCP_URL` must point at a gateway

`sbx mcp` is hidden until an env var enables it. Point it at a **real gateway** -
Docker's hosted control plane (`https://gateway.docker.com`), **not** the public
registry (a catalog, not a gateway).

```bash
export SBX_MCP_URL=https://gateway.docker.com
```

```bash
sbx daemon start
```

Confirm the subtree unlocked:

```bash
sbx mcp --help
```

## Step 1 - Register a server

Register the Wikipedia MCP server as a **local stdio** container:

```bash
sbx mcp add local-wiki --command docker --args "run,-i,--rm,mcp/wikipedia-mcp"
```

```bash
sbx mcp ls
```

```bash
sbx mcp inspect local-wiki
```

> [!WARNING]
> **Local stdio servers run on the HOST**, with your full permissions. Use them
> for development, not untrusted code - exactly the risk the gateway exists to govern.

## Step 2 - Attach it to a sandbox and verify inside the agent

```bash
sbx run claude --static-mcp local-wiki
```

Inside the agent, list servers:

```bash
/mcp
```

You see **one** server - the gateway - aggregating every backend. Now make it call
a tool:

```bash
claude -p "Use the wiki tools to search Wikipedia for 'Eiffel Tower', then give me the summary and 3 key facts. Tell me which tool(s) you called."
```

The tool-call line proves the full chain: `sbx → mcp-gateway → local-wiki →
Wikipedia`, every call through the governed gateway.

## Govern it - an MCP access policy (admin)

Steps 1-2 were **developer-side**. The governance side is where an **org admin**
decides which tools agents may call at all - authored once in Docker Hub
(**AI governance → MCP policy**) as a **[Cedar](https://www.cedarpolicy.com/)**
document, enforced at the gateway for everyone in `$$org$$`.

MCP governance is **default-deny**, so a `permit` is the whole allow-list. This
one permits exactly the three tools the Wikipedia prompt uses:

```cedar no-run-button
permit(
  principal,
  action == MCP::Action::"invokeTool",
  resource is MCP::Tool
)
when {
  resource.server == "local-wiki" &&
  ["search_wikipedia", "get_summary", "extract_key_facts"].contains(resource.name)
};
```

Any other `local-wiki` tool is denied and audited. The Cedar schema gates MCP at
**three** points, each default-deny:

| Action | Fires on | Governs |
| --- | --- | --- |
| `register` | `sbx mcp add` | whether a server may be registered at all |
| `invokeTool` | a server tool call | whether a specific `(server, tool)` pair may run |
| `invokePrimordial` | a gateway built-in | the gateway's own escape hatches (`mcp-exec`, `code-mode`) |

## A read/write split - the DHI server

Docker's Hardened Images server splits cleanly into read-only and mutating tools -
the ideal shape for a real rule:

```bash
sbx mcp add remotedhi --url https://dhi.io/mcp
```

```bash
sbx mcp tools remotedhi
```

Attach it and ask the agent to do one of each:

```bash
sbx run claude --static-mcp remotedhi
```

```bash
claude -p "Get the CVEs for a Docker Hardened Image, then create a mirror of it."
```

`dhi_get_image_cves` matches a `permit` and **returns data**; `dhi_create_mirror`
matches none and is **denied and audited**. Same server, same principal - the
allow-list decides **tool by tool**.

## Clean up

```bash
sbx mcp rm remotedhi
```

## Recap

- `sbx mcp` is gated behind `SBX_MCP_URL`, which must point at a **real gateway**.
- Inside the agent, the gateway appears as one aggregated `mcp-gateway` - the
  governed endpoint every tool call flows through.
- The org's Cedar policy controls what's *invocable*, tool by tool, and audits it.

You've now governed everything the agent touches - network, filesystem,
credentials, the image it builds, and its tools. Next, watch one rogue agent try
to break **all** of it at once: **Putting It All Together**.
