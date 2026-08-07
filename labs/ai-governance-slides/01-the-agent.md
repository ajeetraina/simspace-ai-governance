<!--
layout: section
-->

# The agent does something real

An autonomous agent containerises a live app - and shows you exactly what needs governing.

---

## "Containerise this app"

Hand the [Product Catalog service](https://github.com/ajeetraina/product-catalog-demo-showcase) - Node.js + Express, backed by Postgres, S3, and Kafka, **no Dockerfile yet** - to Claude:

```bash no-run-button
claude -p "Containerise this Node.js app: read package.json and src/,
write a Dockerfile and a compose.yaml, build product-catalog:latest."
```

:::fragment
It works. The agent reads the project, writes a `Dockerfile` and a `compose.yaml`, and builds the image. **Nothing failed.**
:::

Note: The demo lands harder when it "just works" first. No error, no drama - the
agent is helpful and competent. The problems are invisible until you look.

---

## So scan what it shipped

```bash terminal-id=demo
docker scout quickview product-catalog:latest
```

::terminal{id=demo height=300}

Note: Click Run. The agent picked the convenient, fat `node:20` base - and it
dragged in dozens of CVEs from packages the app never uses.

---

<!-- layout: split -->

# Two problems just surfaced

<!-- region -->

:::card{label="problem 1 · supply chain" accent=amber}
The **image is vulnerable**. The fat `node:20` base drags in dozens of CVEs from packages the app never uses.

→ fixed later with **Hardened Images**.
:::

<!-- region -->

:::card{label="problem 2 · blast radius" accent=red}
The **agent ran as *you***. Full access to your network, filesystem, and credentials while it worked.

→ we close this **first**.
:::

Note: Two distinct risks from one helpful action. Order matters: contain what the
agent can *do* before you worry about what it *builds*. A contained agent that
builds a bad image is a scan away from fixed; an ungoverned agent with a perfect
image can still exfiltrate your keys.
