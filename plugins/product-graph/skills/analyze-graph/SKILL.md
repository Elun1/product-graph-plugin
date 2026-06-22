---
name: analyze-graph
description: 'Produce a dated interpretive portrait of the product model''s shape — what the structure reveals that the prose doesn''t. Reads the deterministic metric verbs, applies the judgment the core refuses (naming, ranking, theme-calling), leads with the structural STORIES the wiring tells (feedback loops, single points of failure, latent contradictions, exposure asymmetries), composes a narrative, and persists it as a stored snapshot via the loopback http_api. Use when: analyze graph, graph analysis, shape of the model, what does the graph reveal, interpretive analysis, load-bearing nodes, path asymmetry, analyze strategy, portrait of the model, stories in the graph, structural tensions.'
---

# Analyze the Graph (reflective portrait)

You are the **smart orchestrator** over the `product-graph` MCP server (the "dumb tool").
This skill produces a **dated interpretive portrait** of the model's shape — "what the
structure surfaces that the prose doesn't." It is **not** a triage briefing (that is
`brief-graph`). It is occasional and reflective; its output is a stored snapshot, not an
action list.

The portrait has two registers, and **stories lead**:

- **Analysis** says: *"this node is load-bearing; here are the gaps."* (size)
- **Story** says: *"the loop is built to raise the one number its own growth pushes
  down."* (tension and direction)

Same graph — the difference is whether you read the topology for **tension and
direction** or just for size. The analysis sections are still produced (the Briefing's
visuals bind to them), but the snapshot opens with 2–5 **named stories** found by the
four reading moves in Step 2.

## Hard rules (identical to brief-graph — carried from the build)

1. **Read-only. No write path.** You only call read verbs here. Recommendations route
   through the normal `propose_* → ratify_*` loop — you never propose or ratify inside
   this analysis. Persisting the snapshot to `/api/analysis` (see §4) is NOT a graph
   write — it creates no node, no edge, and seals nothing.
2. **Respect edge `category`.** The causal spine is `causal` + `epistemic` only. Never
   treat a `structural` (`realizes`/`supersedes`/`decomposes_into`) or `measurement`
   (`measures`) edge as causal support. The metric verbs already enforce this — don't undo
   it by mixing categories.
3. **The core is dumb on purpose.** It returns raw mechanical counts. Ranking, naming
   ("crown-jewel", "lynchpin", "assumption-heavy"), and "does this matter" are *your* job.

## How this differs from `brief-graph`

| | `brief-graph` | `analyze-graph` |
|---|---|---|
| Purpose | action-triage: what needs judgment **now** | reflective portrait: what the **shape** reveals |
| Cadence | frequent | occasional |
| Output | ranked attention list + propose→ratify next steps | named stories + a dated narrative essay (stored snapshot) |
| Judgment | which findings to act on | reading the wiring for tension and direction — stories, path asymmetries, load-bearing nodes, evidence vs assumption balance |

## Prerequisites

The `product-graph` MCP server must be connected — via the plugin or, on a local
checkout, `claude mcp add` / `.mcp.json`. The metric verbs run over MCP. **Persisting a
snapshot** additionally needs the `/api/*` loopback (same process; default port 8799),
which is reachable on a local checkout but not over the hosted plugin connection.

## Step 1 — Gather mechanical metrics (read verbs only)

Run these in parallel where possible. All are MCP tools on the `product-graph` server.

### 1a. Node load-bearingness
```
node_metrics()   # defaults to all active nodes
```
Returns per-node in/out degree split by all 6 categories. From this you derive:
- **Load-bearing nodes**: nodes with high `out.causal + out.epistemic + out.measurement`
  (things many others depend on downstream)
- **Convergence nodes**: nodes with high `in.causal + in.epistemic` (where many paths meet)
- **Threat-exposed nodes**: nodes with high `in.threat` (risks threaten them)
- **Assumption-bearing nodes**: nodes with high `in.grounding` (many assumptions underwrite them)

### 1b. Causal-spine path lengths (for bet→business_result or chosen anchors)
To find path asymmetry ("retention is one hop; acquisition is five"), call
`causal_path(source_id, target_id)` for each business_result or chosen anchor pair. You
may first call `query_nodes(type="business_result")` to enumerate the anchors.

### 1c. Subgraph census — grounding vs evidence balance
For the whole graph, and optionally per strategy subgraph (pass `node_ids` from
`causal_descendants`):
```
subgraph_census()
```
Returns `edges_by_category` (grounding = assumes/tests, epistemic = is_evidence_for /
contradicts) and `nodes_by_type` (risk : assumption : hypothesis tally + deliberation
census). Covers: "assumption-heavy vs evidence-grounded strategies", "test-light
deliberation" (few `tests` edges relative to deliberation nodes).

### 1d. Measurement gaps (reuse existing verb)
```
find_gaps(node_type="outcome", direction="in", edge_types=["measures"], neighbor_type="indicator")
```
Returns unmeasured outcomes — the "ambitious but unmeasured" finding.

### 1e. Threats on the board (reuse existing verb)
```
query_edges(category="threat")
```
Cross-reference with `node_metrics` to rank threats by target load-bearingness.

### 1f. Recent activity for provenance (reuse existing verb)
```
query_events(since=<ISO 30 days ago>, limit=200)
```
Summarises what's been ratified / proposed lately to give the analysis a timestamp context.

### 1g. Full edge + node inventory (feeds the reading moves in Step 2)
```
query_edges(limit=500)        # every edge: direction, type, category, rationale
query_nodes(limit=500)        # every node: type, title, body
```
The four reading moves need to lay edges of *opposite sign* over shared endpoints and
read what node prose *claims* against what the wiring *shows* — that takes the full
inventory, not just the counts. Also pull `query_edges(type="contradicts")` explicitly so
you know which tensions are already drawn (move 3 is about the ones that aren't).

## Step 2 — The four reading moves (read for stories, not size)

Before any ranking, make these four passes over the inventory from 1g. None of them is
counting; each is a way of reading topology for **tension and direction**. They produce
the 2–5 named stories that lead the portrait.

1. **Cycles / feedback** — find nodes where two arrows of *opposite sign* land (e.g. one
   `leads_to` in, one `threatens` in) and trace whether the positive path's own success
   feeds the threat. This reveals self-reinforcing or **self-undermining** dynamics: a
   loop built to raise the very number its own growth pushes down. Mechanics: shared
   endpoints + edge direction + category sign (causal/epistemic = supporting,
   threat = opposing).
2. **Convergence** — find what (almost) everything hangs off, then ask how thin *its*
   support is. A near-pure hub whose own grounding is a single thinly-evidenced belief
   is a **single point of failure**: the whole strategy as one bet on one barely-observed
   behaviour. Mechanics: in/out degree from `node_metrics`, then walk the hub's own
   `assumes`/`tests`/`is_evidence_for` in-edges and judge their weight.
3. **Unstated tension** — find nodes that argue with each other with **no `contradicts`
   edge drawn between them**: an assumption claiming "X isn't the constraint" sitting
   beside ratified risks that describe exactly X-shaped ceilings. The tension is real;
   the graph just hasn't admitted it. Mechanics: read node titles/bodies against each
   other (this is judgment, not string-matching) and check the `contradicts` inventory
   for absence.
4. **Asymmetry** — find where threats cluster vs where they don't, and where indicators
   sit vs where they don't. This reveals which side of the strategy is **exposed** and
   what's instrumented vs **flying blind** — e.g. every metric watches loop-health while
   the strategic payoffs have no metric on them: you'll see the machine run but not see
   it win. Mechanics: distribution of `threat`-category edges and `measures` edges across
   the model's regions.

For each story you find, record four things:

- **title** — short, punchy, human ("The loop eats itself at scale").
- **dynamic** — 1–3 sentences telling the tension in human register: *"success creates
  the condition for failure"*, not *"node X has opposing in-edges."*
- **wiring** — the specific edges/nodes that reveal it, compactly
  (`operational loop —leads_to→ resolution quality; retrieval ceiling —threatens→ resolution quality`).
  Every story must trace to verifiable graph facts.
- **origin** — `emergent` (only visible in the wiring) vs `prose` (the docs already
  half-tell it; the wiring confirms or sharpens it). Be honest: emergent stories are the
  payload of this skill, but mislabelling a known tension as a discovery erodes trust.

Aim for 2–5 stories. If a move yields nothing, that's fine — don't pad. If the graph is
genuinely story-less (small, acyclic, evenly instrumented), say so plainly in the lede
and let the analysis carry the portrait.

## Step 3 — Apply the judgment the core refuses

The core gives you raw numbers. You supply:

- **Name the load-bearing few.** Sort `node_metrics` by a weighted load formula
  (downstream causal + inCausal*0.7 + threats*1.5 + grounding*1.0) and call out the top
  2–3 by name. "These nodes are the load-bearing centre of the model."
- **Path asymmetry.** Compare `causal_path` lengths across business results. "Acquisition
  is 5 hops with 3 conditional edges; retention is 1." The names come from you.
- **Degree centrality narrative.** Which node has the most downstream dependents? Which has
  the most converging paths? Call it the "lynchpin" or "crown-jewel" as appropriate.
- **Bet count per cluster.** From the census, how many bets commit to each opportunity
  cluster? "Three bets concentrate on VA-PMF; one is isolated."
- **Evidence vs assumption balance.** `edges_by_category.grounding` (assumes/tests) vs
  `.epistemic` (is_evidence_for/contradicts). "The model is 4:1 assumptions to evidence —
  assertion-heavy, test-light." Drill per-subgraph if interesting. Then go one rung
  deeper on **quality, not just count**: pull the `tests`/`is_evidence_for` edges
  (`query_edges`) and tally their `attrs.evidence_strength`. A graph can look
  evidence-grounded by count yet be propped up by `weak` edges — "8 assumptions tested,
  but 5 only weakly; the evidence is thin where it's load-bearing." Likewise note
  assumptions carrying `attrs.confidence: low`. This is the *thinly-evidenced* story the
  raw category ratio hides.
- **Threats ranked by target load.** For each threat edge, cross-ref the target's
  `node_metrics` load score. "VA-PMF risk threatens the two most load-bearing bets."
- **Ambitious-but-unmeasured outcomes.** From `find_gaps`, call the ones that are also
  convergence nodes (high in-degree) "the most ambitious and the most unmeasured."

## Step 4 — Compose the narrative

Write a dated, attributed interpretive portrait. **Stories first, analysis underneath.**
Structure:
1. **Opening sentence**: what graph you read, when, and how many nodes/edges (from the
   census).
2. **The stories** (2–5, from Step 2): each as title + dynamic + the wiring that reveals
   it + the emergent/prose flag. This is the lead — the thing the reader couldn't get
   from the docs.
3. **The load-bearing centre** (~2 paragraphs): name the 2–3 load-bearing nodes and what
   they are structurally.
4. **Path geometry** (~1 paragraph): shortest vs longest causal paths, asymmetries.
5. **Deliberation health** (~1 paragraph): assumption vs evidence balance, test coverage,
   risks ranked by target load.
6. **Thin spots** (~1 paragraph): unmeasured outcomes, unsupported bets (from `find_gaps`).
7. **Closing label**: "This analysis is generated from the graph. It sealed nothing. Every
   structural finding above traces to verifiable graph facts."

Keep it interpretive, not a data dump. One clear name per structural finding. Keep the
register human throughout — "the loop raises the one number its own growth pushes down,"
never "node X has opposing in-edges." Under ~800 words.

### The Briefing page binds your prose to live visuals — emit it KEYED

The Briefing's Analysis surface computes the three curated visuals itself, **live from
the verbs** (the load strip, the path-geometry bars, the composition ratio + threat
callout). It does **not** parse your narrative. The **stories** travel as a structured
`metrics.stories` array (the page renders them at the top, before the finding-units);
the analysis prose travels as a small **keyed `prose` map** inside `metrics` (in
addition to the full `narrative`, which is kept for the appendix / fallback). The prose
keys map 1:1 to the finding-units:

- `prose.lede` → the page lede (1–2 sentences framing the portrait).
- `prose.load_centre` → bound to the **load strip** (the load-bearing centre +
  any unmeasured load-bearing outcome).
- `prose.path_geometry` → bound to the **path bars** (asymmetry, or symmetry
  through a shared pivot, stated honestly as a finding).
- `prose.composition` → bound to the **ratio + threat callout** (evidence vs
  grounding vs threat, threats ranked by target load).

Within these strings, wrap the **named judgment** in `**double asterisks**` — the page
renders it as the underlined named claim ("the **load-bearing centre**", "all **inward
testimony**", "the **lynchpin**"). Keep each to 1–3 sentences; the prose is primary, the
visual makes its one comparison instant. **Honesty rule still governs:** if a comparison
is symmetric, zero, or empty, say so plainly in the matching key — the visual renders
those as first-class findings, never as a broken chart.

## Step 5 — Persist as a stored snapshot

After composing the narrative, persist it via the loopback `http_api` with curl (NOT an
MCP write — this creates no node/edge and seals nothing):

```bash
curl -s -X POST http://127.0.0.1:8799/api/analysis \
  -H "Content-Type: application/json" \
  -d '{
    "generated_by": "analyze-graph skill",
    "metrics": {
      "node_count": <N>, "edge_count": <E>, "census": <census_json>,
      "stories": [
        {
          "title": "<short punchy name>",
          "dynamic": "<1–3 sentences, human register; **bold** the named tension>",
          "wiring": "<the edges that reveal it, e.g. loop —leads_to→ quality; ceiling —threatens→ quality>",
          "origin": "emergent | prose",
          "move": "cycle | convergence | tension | asymmetry",
          "node_ids": ["<ids of the nodes involved, for click-through>"]
        }
      ],
      "prose": {
        "lede": "<1–2 sentence framing>",
        "load_centre": "<load-centre section, bound to the load strip>",
        "path_geometry": "<path-geometry section, bound to the path bars>",
        "composition": "<deliberation-health section, bound to the ratio + threat callout>"
      },
      "gate_label": "This portrait is generated from the graph. It sealed nothing. Every finding traces to a node or edge you can open and judge."
    },
    "narrative": "<the full narrative text — kept for the appendix / older-client fallback>"
  }'
```

Replace `<N>`, `<E>`, `<census_json>` with the actual values from step 1, fill `stories`
from Step 2, and fill the `prose` keys with the matching sections you composed. A
successful save returns `{ "id": "...", "generated_at": "...", ... }`. Both `stories` and
the `prose` map are optional — if omitted, the page falls back to rendering the full
`narrative` — but emit them so the stories lead the page and the finding-units bind
prose to evidence as designed. Include `node_ids` whenever you can name them: the page
turns them into chips that open the node in the reasoning environment.

The briefing page reads `GET /api/analysis/latest` to display the stored snapshot.
Re-running this skill creates a new row; history is available at `GET /api/analyses`.

## Notes

- If a verb returns nothing (no threats, no unmeasured outcomes), say so plainly — an
  empty result is a healthy structural signal.
- Do not conflate `status` (the trust state: proposed/ratified/contested) with `lifecycle`
  (Bet domain field) or `delivery_status` (execution domain field).
- This skill is read-only. For any structural change the analysis surfaces, route through
  `propose_* → ratify_*` — never auto-fix inside this skill.
