---
name: brief-graph
description: 'Produce a prioritized "what''s worth my attention" briefing over the product-graph by reading its deterministic diagnostics. Use when: brief the graph, graph briefing, sense-making, what needs my attention, what''s stale/contested/unsupported, review the graph, what changed in the graph.'
---

# Brief the Graph (Layer 2 sense-making)

You are the **smart orchestrator** over the `product-graph` MCP server (the "dumb
tool"). The server answers deterministic structural questions; **you** supply the
judgment it deliberately refuses to hold: which findings matter today, why, and
what to look at first. This skill turns the graph's mechanical facts into a
ranked briefing.

## Hard rules (non-negotiable — carried from the build)

1. **Read-only. No write path.** You only call read verbs here. When a finding
   warrants action, your output *recommends* a fix routed through the normal
   `propose_* → ratify_*` loop — you never propose or ratify inside this briefing.
2. **Respect edge `category`.** The causal spine is `causal` + `epistemic` only.
   Never treat a `structural` (`realizes`/`supersedes`/`decomposes_into`) or
   `measurement` (`measures`) edge as causal support. The `find_gaps` and
   `causal_ancestors/descendants` verbs already enforce this — don't undo it by
   passing the wrong categories.
3. **The core is dumb on purpose.** Don't ask it to rank or score. Ranking,
   staleness thresholds, and "does this matter" are *your* job, applied to its
   raw findings.

## Prerequisites

The `product-graph` MCP server must be connected — via the plugin or, on a local
checkout, `claude mcp add` / `.mcp.json`. A read+propose token is all this skill needs.
All verbs below are MCP tools on the `product-graph` server.

## Step 1 — Gather deterministic findings (read verbs only)

Run these and collect the raw results. Each is a single MCP call.

**Structural gaps** (`find_gaps` — nodes lacking a qualifying incident edge;
defaults are strict: only `ratified` nodes are candidates, only `ratified` edges
count as support):

| Finding | Call |
|---|---|
| Bets with no causal/epistemic support | `find_gaps(node_type="bet", direction="in", categories=["causal","epistemic"])` |
| Outcomes not measured by any indicator | `find_gaps(node_type="outcome", direction="in", edge_types=["measures"], neighbor_type="indicator")` |
| Dangling business results (no incoming spine) | `find_gaps(node_type="business_result", direction="in", categories=["causal","epistemic"])` |
| Opportunities that convert to no bet | `find_gaps(node_type="opportunity", direction="out", edge_types=["converts_to"])` |

**Status anomalies** (reuse existing verbs):
- Contested: `query_nodes(status="contested")` and `query_edges(status="contested")`.
- **Contradicted** (high signal): `query_edges(type="contradicts", status="ratified")`
  — then resolve each edge's `target_id` (the node being contradicted) with `get_node`.
  Read each edge's `attrs.evidence_strength`: a `strong` contradiction outranks a `weak`
  one — rank by it.
- **Thinly evidenced** (a finding `find_gaps` can't see — it only knows edge *absence*):
  a load-bearing belief that *is* supported but only by `evidence_strength: weak`
  `tests`/`is_evidence_for` edges, or an assumption carrying `attrs.confidence: low`.
  This is distinct from naked/unsupported (no edge at all) and from contested — it's
  "ratified, but standing on thin ice." Pull the grounding/epistemic edges, read their
  strength, and surface the ones whose target is high-load (`node_metrics`).
- **Stale relationships** (`stale_edges()`): ratified relationships whose endpoint was
  edited *after* the relationship was ratified — i.e. sealed against content that has
  since moved. The core only reports the mechanical fact ("endpoint changed since this
  was sealed"); **you** judge whether the relationship still holds. Resolve each edge's
  `source_id`/`target_id` with `get_node` to see what moved.
- **Pending changes to ratified items** (`list_proposed()` → `changes`): proposed
  changes the orchestrator has staged against *ratified* nodes/edges (e.g. a
  re-type). The target stays sealed until you approve. Each change carries
  `fields` (the diff), `rationale`, and `would_invalidate` — a live preview of the
  relationships that approving it would drop to `proposed`. Read `would_invalidate`
  so you approve with eyes open. Resolve `target_id` with `get_node`/`get_edge` for
  context.
- Stale review queue: `list_proposed()` → keep the `nodes`/`edges` whose
  `created_at` is older than your staleness threshold (**default: 14 days** before
  today; this threshold is *yours*, not the core's).

**What changed** (the audit log):
- `query_events(since=<ISO timestamp 7 days ago>)` — the recent activity stream.
  Group by `verb` for the narrative ("3 ratified, 1 contested, 2 new proposals").

## Step 2 — Apply the judgment the core won't

- **Leading-indicator refinement.** For each outcome that *is* measured (i.e. not
  flagged by the `find_gaps` indicator check), confirm a *leading* one exists:
  call `neighbors(outcome_id, edge_types=["measures"], direction="in")` and check
  each indicator's `attrs.lead_lag == "leading"`. An outcome measured only by
  lagging indicators is a real (softer) gap — surface it as such.
- **Traces-back-to-a-bet (multi-hop).** For each business result you care about,
  call `causal_ancestors(br_id, depth=5)` and inspect the returned `nodes` for one
  with `type == "bet"`. No bet in the ancestry = "this result isn't tied to any
  committed wager" — distinct from the single-hop "dangling" check.
- **Spine completeness.** For a chosen anchor (e.g. a business result or
  opportunity), walk `causal_ancestors` / `causal_descendants` and note where the
  chain breaks (e.g. an outcome with no upstream bet, a bet with no upstream
  opportunity). The core gives you the reachable set; *you* read the break.

## Step 3 — Prioritize and narrate

Produce a briefing. Suggested order (highest-attention first):

1. **Contradicted / contested** — the graph is actively disputing itself. Cite the
   node/edge ids and the contesting reason or the contradicting edge.
2. **Unsupported bets & dangling business results** — committed things with no
   evidentiary/causal grounding. These erode trust in the graph fastest.
3. **Pending changes to ratified items** (`list_proposed().changes`) — staged
   re-types/edits awaiting your approval. Lead with the change's `rationale` and its
   `would_invalidate` preview (what approving would drop), so the seal decision and
   its relationship fallout are visible together. Only an admin approves; flag, don't
   pre-judge.
4. **Stale relationships** (`stale_edges`) — ratified links whose endpoint changed
   after sealing. Surface them as "still valid?" review prompts; cite the edge id,
   its reading, and which endpoint moved. Never assume they're broken — that's the
   human's call.
5. **Measurement gaps** — outcomes with no indicator, or only lagging ones.
6. **Stale review queue** — proposals sitting unratified past the threshold.
7. **What changed** — a short recent-activity narrative for context.

For each item give: **what** (one line), **why it matters** (your judgment),
**the ids** (so it's actionable), and a **suggested next step that goes through
`propose_* → ratify_*`** (e.g. "propose an `is_evidence_for` edge from signal X to
bet Y, then ratify" — never an auto-fix). Keep it scannable; lead with the 3–5
things actually worth attention, not an exhaustive dump.

## Notes

- If a verb returns nothing, say so plainly ("no unsupported bets") — an empty
  result is a healthy signal, not a failure.
- All findings are point-in-time reads of the current ratified graph; re-run the
  skill after ratifying fixes to confirm gaps closed.
