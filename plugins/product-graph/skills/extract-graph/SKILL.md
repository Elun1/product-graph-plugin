---
name: extract-graph
description: 'Turn a prose document (strategy paper, research doc, slides) into a typed causal graph proposed into the product-graph (never ratified). Use when: extract graph from doc, doc to graph, ingest strategy doc, propose graph from document, build the graph from this paper, extract-graph.'
---

# Extract a Graph (document → proposed causal graph)

You are the **smart orchestrator** over the `product-graph` MCP server (the "dumb
tool"). This skill is the **write counterpart** to `brief-graph`: it reads a prose
document and *proposes* a typed causal graph — typed nodes plus first-class,
directional, rationale-carrying edges. **You propose; a human ratifies.** The skill
uses the orchestrator token and physically cannot ratify; your output routes the
human to the gate.

The value is the **typed decomposition + the per-edge rationale**, never a paraphrase
of the document. If the output reads like a summary, the skill failed — faithful
typed wiring surfaces what prose buries (a passing mention that turns out to be
structurally load-bearing).

## Hard rules (non-negotiable)

1. **Propose only — never ratify.** Orchestrator token; you physically cannot ratify.
   Your output recommends the human ratify in the UI (admin token). The gate is
   untouched. Never call a `ratify_*`/`approve_*`/`contest_*`/`reject_*` verb here.
2. **Model the substance, not the noun.** *The* most important rule. Don't create a
   node for a named "thing" just because it's named — a competitor mention is really
   an `insight` (what we learned from what they do) or a `risk` (what their move
   endangers). Find the claim the name represents; draw *that*. Same for any team,
   product, or artefact named in passing.
3. **The rationale is the payload, not a paraphrase.** Every edge carries a genuine
   "because", quoting the source where you can. An edge without a real rationale is a
   failure, not a stylistic lapse.
4. **Read the catalog every run; never hardcode node/edge types.** `get_catalog`
   first. The ontology *is* the extraction vocabulary and it changes between runs.
5. **Respect edge legality and direction.** Legality is **family-keyed** and edges are
   **directional**. Pick the edge whose *reading* — "{source} {verb} {target}" —
   matches the sentence. Direction is easy to get backwards (in a prior manual run
   several `is_evidence_for` edges should have been `threatens`/`assumes`). Beliefs
   (the `deliberation` family) are **spine sinks**: the causal spine is `causal` +
   `epistemic` only; `assumes`/`tests`/`threatens` sit off-spine and never relay a
   causal path.
6. **Existing = `lever`; not-yet-real = `bet` + target `outcome`.** Show reality vs
   aspiration; never flatten them into one node. (E.g. moat layers that exist are
   levers; an aspirational one is an outcome a bet must produce.)
7. **Anchor every node to the source; `dedup_key` every node; nodes before edges.**
   No node without a source anchor (the node-level equivalent of rule 3). A stable
   `dedup_key` makes re-runs idempotent. Propose all nodes before any edge.
8. **Avoid floating nodes.** Wire the leaves: opportunities `converts_to` a bet; risks
   `threatens` something; outcomes get a `measures` indicator — or are *knowingly*
   reported as a gap in step 7. A ratified risk that threatens nothing is invisible to
   any "what's at risk" read.

## Prerequisites

The `product-graph` MCP server must be connected — via the plugin
(`/plugin install product-graph@productos`) or, on a local checkout, `claude mcp add` /
`.mcp.json`. A read + propose token is all this skill needs. All verbs below are MCP
tools on the `product-graph` server.

## Args

- `path` — path to the source document (`.md` / `.txt` / `.pdf` / `.docx`). Required.
- `scope` — `dry-run` (default) | `spine` | `full`.
  - **dry-run** (default): produce the full extraction plan as text — every node with
    type, title, body, source anchor; every edge with type, direction, rationale —
    and write **nothing**. The cheapest first look; it sidesteps the admin UI
    entirely. Review it, then re-run with a writing scope.
  - **spine**: propose the causal spine only — diagnoses → bets/levers → key
    outcomes/indicators → top risks — then offer a second pass for the periphery.
  - **full**: propose everything (expect a large ratify queue — a full run was ~67
    nodes / ~85 edges).

## Step 1 — Convert the source to text

This box has **no** doc-conversion tools (`textutil`/`pandoc`/`pdftotext` absent) and
no Python doc libraries — stay dependency-free:

| Format | Method |
|---|---|
| `.md`, `.txt` | Read the file directly. |
| `.pdf` | Read the file directly — the Read tool reads PDFs natively (use the `pages` param for long docs). No external tool. |
| `.docx` | Unzip with the Python stdlib — `python3 -c` over `zipfile` to read `word/document.xml`, then strip XML tags (treat `</w:p>` as a paragraph break). No new dependency. |
| `.doc` (legacy) / other | Ask the user to pre-convert to `.txt` / `.md` / `.pdf`. |

A stdlib `.docx` extractor (no install needed):

```bash
python3 - "$PATH" <<'PY'
import sys, zipfile, re
with zipfile.ZipFile(sys.argv[1]) as z:
    xml = z.read("word/document.xml").decode("utf-8", "ignore")
xml = re.sub(r"</w:p>", "\n", xml)        # paragraph breaks
print(re.sub(r"<[^>]+>", "", xml))         # strip remaining tags
PY
```

**Read the whole document before extracting** — emphasis in the prose is not the same
as centrality in the graph.

## Step 2 — Load the catalog (never hardcode)

Call `get_catalog`. Note the active node families/types, edge types, their `reading`,
their legal family pairs, and required attrs (`risk` needs `probability` + `impact`;
`hypothesis` needs `falsification`; `lever` needs `leverage_description`; `indicator`
carries `lead_lag`). Build your type choices from *this*, not from memory of a previous
run.

## Step 3 — Check the target graph state

`query_nodes(limit=1)` to see empty vs live. On a **live** graph, call
`find_similar(title)` before proposing a node and judge whether a hit is "the same
thing" (the core won't decide this for you). On an **empty** graph, proceed directly.

## Step 4 — Extract, present the plan, and ask the forks (before any write)

Produce the extraction plan: nodes grouped by family/type (each with title, the
1–3-sentence body, and its source anchor) and edges (each with type, direction, and
its rationale). Then ask **2–3 forks** with `AskUserQuestion`, only where genuinely
ambiguous — cite the specific sentence:

1. **Scope/depth** — `dry-run` | `spine` | `full`. Ask only when the user didn't pass
   `scope` *and* the document is large enough that it matters. Default `dry-run`.
2. **Entity / competitor handling** — for an ambiguous named entity: is this an
   `insight` (a learning), a `risk` (a threat), or a `lever` (a source of power)?
3. **Existing vs aspirational** — unclear whether a capability exists now (`lever`) or
   is to-be-built (`bet` + target `outcome`)? Ask rather than flatten.

**The power / metric / deliverable test (apply to anything that reads like "a thing we
can move" — the word "lever" in prose does NOT auto-map to type `lever`; run the test
on what it *is*):**

- **Source of power → `lever`**: a *structural advantage*, no units, durable; you
  concentrate force through it. (e.g. "our proprietary interaction dataset," "incumbent
  distribution," "the only vendor certified for X.") Authoring a `lever` **requires
  `attrs.leverage_description`** — *where is the leverage?* — a hard requirement like the
  edge rationale; treat a missing one as invalid.
- **Metric → `indicator`**: has units, is watched/steered/reported; set `lead_lag`.
  (e.g. "TTFV," "Loop Rate," "weekly active resolvers.") The old "actionable input / a
  metric we can pull" sense lands **here**, not in `lever`.
- **Deliverable → `execution`** (`feature` / `epic` / `initiative`): work that ships,
  has delivery status. It `realizes` a bet and stays **off** the causal spine — it's
  *how* you enact a bet, not a link in how value is created.

Also surface (don't necessarily block on) the **leading-vs-lagging** call for each
`indicator` — it's a real judgment, not a default.

**If `scope=dry-run`, the plan text IS the deliverable. Stop here — write nothing.**

## Step 5 — Propose nodes (phase 1, batched)

Propose all nodes before any edge. Batch independent `propose_node` calls in parallel
(one message, many calls). For each node set:

- **`type`** + **`title`** + **`body`** — the body is self-contained (1–3 sentences,
  reads in isolation in the UI). Keep it **clean prose only**: do **not** append a
  `Source: "…" — §…` line. The source document already shows as the `from:` chip
  (`provenance.source`) and the verbatim quote + locator live in `provenance.source_ref`
  (below), so a `Source:` line in the body just clutters the prose and redundantly
  repeats the chip.
- **`dedup_key`** = `"<doc-slug>:<handle>"` (stable, explicit) — makes re-runs
  idempotent; a re-propose returns `{deduped_into: …}` instead of a duplicate.
- **`provenance`** = `{ "source": "<doc-slug>", "source_ref": "§section / p.N — \"…verbatim quote…\"" }` —
  document slug (rendered as the `from:` chip) + a per-node anchor carrying the section
  locator **and** the verbatim quote (queryable via `query_nodes(source_ref=…)`). This
  anchor is the node-level equivalent of the mandatory edge rationale — it's what lets
  the human trust the extraction at ratify time, now held in `source_ref` instead of the
  body.
- **required attrs** — set `lead_lag` on indicators, `probability`+`impact` on risks,
  `falsification` on hypotheses, `leverage_description` on levers (*where the leverage
  is*), or the propose is rejected.
- **evidence-strength / confidence (optional, only when the source says so)** — if the
  doc characterizes *how well-evidenced* a link is ("strong evidence that…", "a weak
  signal", "we're fairly unsure"), set `attrs.evidence_strength`
  (`weak`/`moderate`/`strong`) on the `is_evidence_for`/`contradicts`/`tests` edge, or
  `attrs.confidence` (`low`/`medium`/`high`) on the assumption/hypothesis node. **Never
  infer a strength the prose doesn't state** — absent is *unstated*, which a premortem
  reads honestly; a fabricated "strong" is worse than silence.

Collect every returned ULID into a **handle→ID map** for phase 2.

> Source anchoring caveat: the UI detail panel renders `body`, `attrs`, and
> `provenance.source` (the `from:` chip) — but **not** `provenance.source_ref`. So the
> verbatim quote you put in `source_ref` is the **queryable backstop**, not a display
> field today. We deliberately keep it out of `body` anyway — a duplicated `Source:`
> line just clutters the prose and repeats the chip. The durable fix (render
> `source_ref` / add a first-class anchor field) is a known gap that would make the
> anchor visible again — mention it if asked, but don't change the server/catalog/UI
> here.

## Step 6 — Propose edges (phase 2, batched)

Only after nodes exist. Resolve `source_id` / `target_id` from the handle→ID map.
Batch independent edges in parallel. Every edge needs a **non-empty `rationale`** that
quotes or closely tracks the source. Check each edge's legality + direction against
the catalog `reading` before proposing. Common shapes (confirm against the live
catalog):

- `diagnosis --calls_for--> lever` → `lever --concentrates_into--> bet` (the
  guiding-policy spine segment; both on the causal spine)
- `opportunity --converts_to--> bet` (opportunity → bet)
- `feature --realizes--> bet` (execution → causal, off-spine: how a bet is enacted)
- `indicator --measures--> outcome` / `business_result`
- `bet --assumes--> assumption`, `... --tests--> hypothesis`
- `risk --threatens--> bet` / `outcome` (deliberation → causal)
- `signal --is_evidence_for--> insight` / `bet` / belief; `... --contradicts--> ...`
- `leads_to` (primary cause → outcome) vs `contributes_to` (partial/supporting cause)
- `decision --selects--> bet`/`lever`, `decision --rules_out--> bet` (a recorded
  turning point: what was chosen and what was explicitly refused; both structural,
  off-spine). Evidence flows *in* via `signal --is_evidence_for--> decision`.

## Step 7 — Quality report (`find_gaps` on the freshly-proposed graph)

The graph you just proposed is **all `status=proposed`**, but `find_gaps` defaults to
ratified-only candidates *and* ratified-only support. **Pass
`node_statuses=["proposed"]` and `present_edge_statuses=["proposed"]` on every call**
or the checks see nothing:

| Gap | Call (+ `node_statuses=["proposed"], present_edge_statuses=["proposed"]`) |
|---|---|
| Ungrounded bets | `find_gaps(node_type="bet", direction="in", categories=["causal","epistemic"])` |
| Unmeasured outcomes | `find_gaps(node_type="outcome", direction="in", edge_types=["measures"], neighbor_type="indicator")` |
| Opportunities → no bet | `find_gaps(node_type="opportunity", direction="out", edge_types=["converts_to"])` |
| Floating risks | `find_gaps(node_type="risk", direction="out", edge_types=["threatens"])` |

Report gaps **by node title** (the UI doesn't navigate by ULID). An empty result is a
healthy signal — say so plainly.

## Step 8 — Route the human to ratify

Tell the human to ratify in the UI, **top-down**: diagnoses and bets first, then
outcomes/indicators, then the periphery. Remind them that only the admin token (the
UI) can ratify — the skill cannot. Re-running the skill on the same document is safe
and idempotent (`dedup_key`), so refining and re-proposing is cheap.

## Notes

- **Prose register.** In human-facing output, avoid graph-theory jargon — say "causal
  relationship", or use the catalog's `reading` directly ("is evidence for",
  "measures", "threatens"). Use titles, not IDs.
- **Messy, non-hierarchical wiring is the point.** Wire a node to everything it's
  genuinely related to; don't force a tidy tree. An insight can support both a
  diagnosis and an opportunity.
- **A recorded choice is a `decision`, not just a bet lifecycle.** When the prose
  states a *turning point* — "we chose X over Y", "we decided against Z", "this
  supersedes our earlier call" — mint a `decision` node and wire `selects` to the
  chosen path and `rules_out` to the refused one, each rationale carrying the "at
  this time, because". Don't flatten it into the loser's `lifecycle=rejected`: the
  decision node is what makes *why we turned* queryable. A bare preference with no
  stated reasoning is not a decision — don't manufacture one.
- **Re-type cascades.** Getting a node's *family* wrong and fixing it later can make
  its edges illegal and drop them to `proposed`. Cheaper to get the type right up
  front than to re-wire after.
- **This skill is extraction only.** It ends at the `find_gaps` report and the ratify
  hand-off. Analysis and visual export are a *separate* concern that runs on
  *ratified* content (build on `brief-graph`) — never bolt them onto this write skill
  operating on unreviewed proposals.
