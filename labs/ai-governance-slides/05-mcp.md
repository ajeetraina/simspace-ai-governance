<!--
layout: section
-->

# MCP Governance

One governed gateway. Every tool call authenticated, authorized, audited.

---

## One gateway, every backend behind it

```text no-run-button
Agent ── /mcp ──▶  ONE mcp-gateway  ──▶  backends (local-wiki, DHI, ...)
                   every tool call: authenticated · authorized (Cedar) · audited
```

The agent talks to **one** aggregated gateway. Inside the agent, `/mcp` shows a single server - not a dozen. That single governed endpoint is the whole point of Pillar 2.

Note: `sbx mcp` registers servers fronted by the Docker MCP Gateway. It's gated
behind `SBX_MCP_URL`, which must point at a real gateway
(`https://gateway.docker.com`), not the public registry - a catalog isn't a
gateway.

---

## The allow-list decides, tool by tool

MCP governance is **default-deny** - a Cedar `permit` is the whole allow-list:

```cedar filename=policy.cedar highlight=6-7 no-run-button
permit(
  principal,
  action == MCP::Action::"invokeTool",
  resource is MCP::Tool
)
when {
  resource.server == "local-wiki" &&
  ["search_wikipedia", "get_summary"].contains(resource.name)
};
```

Any other tool on that server is **denied and audited**.

Note: Cedar gates MCP at three points, each default-deny: `register` (may a
server be added at all), `invokeTool` (may this specific server+tool run), and
`invokePrimordial` (the gateway's own escape hatches like mcp-exec).

---

<!-- layout: split -->

# A read/write split - the DHI server

<!-- region -->

:::card{label="dhi_get_image_cves" accent=green}
Matches a `permit` → **returns data**. A read-only tool the org allows.
:::

<!-- region -->

:::card{label="dhi_create_mirror" accent=red}
Matches nothing → **denied and audited**. A mutating tool the org withholds.
:::

Same server, same principal - the allow-list decides **tool by tool**.

Note: This is the shape of a real rule. Read-only metadata is safe to expose;
mutating actions stay behind an explicit permit. The gateway enforces it for
everyone in the org, and every denied call is logged.
