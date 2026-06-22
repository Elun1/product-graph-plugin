# product-graph plugin

Connects Claude to a [Product Graph](https://product-graph.app) — a typed causal
model of your product where **AI proposes and a human ratifies**. Installing this plugin
brings two things at once:

- **The skills** — `extract-graph` (doc → typed graph), `brief-graph` and `analyze-graph`
  (sense-making), and the `narrate-*` family (why / exec / strategy / roadmap / premortem /
  opportunities) that render stakeholder artifacts from the graph.
- **The `product-graph` MCP server** — already pointed at the hosted instance, auth'd as
  you over OAuth (no token to copy).

## Install

```
/plugin marketplace add Elun1/product-graph-plugin
/plugin install product-graph@productos
```

That's it — **authentication is OAuth by default**. In **Claude Code**, the first time
Claude uses the server it'll prompt you to sign in: run `/mcp`, pick `product-graph`, and
**Authenticate**. A browser opens to the app's consent page; sign in and **Approve**, and
Claude is connected as you. Nothing to copy or export.

### Claude Desktop / Cowork

The plugin's bundled MCP config uses a **loopback (`localhost`) OAuth callback** — that
works in Claude Code, but **not** in Claude Desktop / Cowork, where there's no local
listener to catch it (you'll see a redirect to `localhost:…/callback` that dead-ends, and
the server won't appear under **Connectors**).

There, install the plugin for the **skills**, but add the MCP server as a **custom
Connector** instead of using the bundled config: **Settings → Connectors → Add custom
connector**, URL `https://product-graph.app/mcp`. That flow uses a hosted callback (no
localhost), so Approve completes and the connector shows up — available in Cowork.

The server defaults to `https://product-graph.app`. Self-hosting? Override it:

```bash
export PRODUCT_GRAPH_URL="https://graph.internal.example.com"
```

Whatever you authorize can read and **propose**, but never **ratify** — sealing always
happens by a human signed into the app. That contract is structural, not a convention.

### Headless / CI (no browser)

OAuth needs a browser for the consent step. In a headless environment, mint a personal
access token in the app under **Admin → Tokens** and pass it as a header in your own MCP
config instead:

```json
{ "mcpServers": { "product-graph": {
  "type": "http",
  "url": "https://product-graph.app/mcp",
  "headers": { "Authorization": "Bearer <your token>" }
} } }
```

> Skills are namespaced by the plugin, e.g. `/product-graph:extract-graph`. Claude also
> invokes them automatically when a task matches their description.
