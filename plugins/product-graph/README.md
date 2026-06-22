# product-graph plugin

Connects Claude to a [Product Graph](https://product-graph.app) — a typed causal
model of your product where **AI proposes and a human ratifies**. Installing this plugin
brings two things at once:

- **The skills** — `extract-graph` (doc → typed graph), `brief-graph` and `analyze-graph`
  (sense-making), and the `narrate-*` family (why / exec / strategy / roadmap / premortem /
  opportunities) that render stakeholder artifacts from the graph.
- **The `product-graph` MCP server** — already pointed at the hosted instance, auth'd as
  you with a token you mint in the app.

## Install

```
/plugin marketplace add Elun1/product-graph-plugin
/plugin install product-graph@productos
```

Before (or right after) installing, mint a token in the app under **Admin → Tokens** and
expose it to Claude as an environment variable:

```bash
export PRODUCT_GRAPH_TOKEN="<your token>"
```

The server defaults to `https://product-graph.app`. Self-hosting? Override it:

```bash
export PRODUCT_GRAPH_URL="https://graph.internal.example.com"
```

The token can read and **propose**, but never **ratify** — sealing always happens by a
human signed into the app. That contract is structural, not a convention.

> Skills are namespaced by the plugin, e.g. `/product-graph:extract-graph`. Claude also
> invokes them automatically when a task matches their description.
