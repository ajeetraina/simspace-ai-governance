<!--
layout: section
theme: dark
-->

# One rogue agent. Four attacks. One engine.

Everything, at once, in a single sandbox.

---

## Four attacks, four boundaries

```text no-run-button
One rogue / prompt-injected agent, one sandbox, one policy engine:
  1 · mount ~/.ssh             ─▶ 403 at CREATION        (filesystem)
  2 · curl paste.ee            ─▶ 403 at the PROXY       (network)
  3 · read $ANTHROPIC_API_KEY  ─▶ proxy-managed sentinel (credential)
  4 · call a rogue MCP tool    ─▶ denied at the GATEWAY  (MCP)
```

*One agent tries all four in a single sandbox, and one policy engine stops each - fail-closed, at a different point in the lifecycle, with no per-attack setup.*

---

<!-- layout: stats -->

## The scorecard

:::stat{value="creation" accent=blue label="filesystem"}
denied mount fails before the sandbox starts
:::

:::stat{value="request" accent=blue label="network"}
proxy refuses per outbound request
:::

:::stat{value="injection" accent=blue label="credential"}
never held in the sandbox
:::

:::stat{value="invocation" accent=blue label="MCP"}
gateway authorizes each tool call
:::

Note: The row that lands with a security team is **fail-closed timing**:
filesystem at creation, network at request, credential at injection, MCP at
invocation. Four different moments, one policy engine, every one default-deny.

---

<!-- layout: quote -->

> "Show me an agent doing something dangerous, and I'll show you where the policy stops it - and where the CISO sees it happen."

Default-deny everywhere. One source of truth. Un-overridable locally.
