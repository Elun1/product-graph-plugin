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

That's it — **authentication is OAuth by default**. The first time Claude uses the server
it'll prompt you to sign in: run `/mcp`, pick `product-graph`, and **Authenticate**. A
browser opens to the app's consent page; sign in and **Approve**, and Claude is connected
as you. Nothing to copy or export.

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
