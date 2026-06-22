---
name: narrate-why
description: 'Render a "why does this exist" narrative for any node — the rationale chain from a feature/initiative up the causal spine to the diagnoses, levers and evidence that demand it, plus what has to be true and what threatens it. The first renderer skill: a stakeholder artifact as a regenerable projection of the graph. Use when: why does this exist, why-trace, why narrative, rationale chain, context for engineers, explain this initiative, explain this bet, why are we building this, narrate why.'
---

# Narrate Why (renderer: the rationale chain)

You are a **renderer** over the `product-graph` MCP server. The graph holds the
typed facts and the written rationales; **you** compose them into a narrative a
human can read top-to-bottom. Every stakeholder artifact is a *projection* of
the graph: a **scope** (which subgraph), an **altitude** (which families
survive), and a **frame** (the question the audience is implicitly asking).
This skill's frame is the engineer's / team's question: **"why am I building
this?"** — answered by walking the causal spine upward and quoting the edge
rationales the graph already requires.

## Hard rules (non-negotiable)

1. **Read-only. No write path.** Only read verbs. If the trace exposes a gap
   worth fixing, *recommend* a `propose_* → ratify_*` action — never perform it.
2. **Quote, don't invent.** The narrative's connective tissue is the edge
   `rationale` fields and node bodies. You may compress and order them; you may
   not fabricate causal claims that have no edge. If a link is missing, say so —
   a visible gap is a feature of this artifact, not a failure.
3. **Render trust state honestly.** `ratified` content is stated plainly.
   Anything `proposed` or `contested` in the scope is marked inline —
   *(proposed — not yet ratified)* / *(contested)* — and counted in the footer.
   Never present an unratified claim with the same confidence as a sealed one.
4. **Respect edge `category`.** The spine is `causal` + `epistemic` only;
   `causal_ancestors` already enforces this. `grounding`/`threat` edges are
   rendered in their own sections (assumptions, risks), never as causal support.

## Prerequisites

The `product-graph` MCP server connected (via the plugin or, on a local checkout,
`claude mcp add` / `.mcp.json`) — read-only suffices.

## Step 1 — Resolve the target

Input is a node id or a title fragment. If a fragment, `query_nodes(text=...)`
and pick the match (ask if ambiguous). Then anchor by family:

- **execution** (initiative/epic/feature): `query_edges(source_id=...)` →
  follow `realizes` to the causal node it enacts (usually a bet; this is the
  anchor) and note `expected_to_drive` targets (outcomes — the "so that").
  If an epic/feature, also walk `decomposes_into` *upward* one hop so the
  narrative names its parent. **No `realizes` path at all → stop and report
  it**: "this work isn't connected to any bet" is the whole finding.
- **causal** (bet/outcome/...): it is its own anchor.
- **artifact**: follow `documents`/`evidences` to the causal node, anchor there.

## Step 2 — Gather the chain (read verbs)

1. `causal_ancestors(anchor_id, depth=6)` — the spine: diagnoses (`calls_for`),
   levers (`concentrates_into`), opportunities (`converts_to`), and the
   signals/insights evidencing them (`is_evidence_for`, `refines`,
   `contributes_to`).
2. `neighbors(anchor_id, categories=["grounding","threat"], direction="both")`
   — assumptions (`assumes`), the hypothesis under test (`tests`, with its
   `falsification` attr), and risks (`threatens`, with probability × impact).
3. Optionally `causal_descendants(anchor_id, depth=3)` when the audience needs
   the "so that" chain (outcomes → business results) spelled out.

## Step 3 — Compose (fixed structure, engineer altitude)

Render markdown with this shape — order matters, it is the argument:

1. **Title + one-line why.** One sentence synthesizing the strongest
   diagnosis→bet rationale. This is the only sentence you compose freely.
2. **The wager.** The anchor bet: title, body essence, `lifecycle`.
3. **What demands it.** Each diagnosis with a `calls_for` edge into the bet:
   one line of diagnosis + its edge rationale, then its evidence **nested
   underneath** (signal → `is_evidence_for` rationale, with the edge's
   `attrs.evidence_strength` when stated). Concrete numbers in signal bodies
   are gold — keep them. If the chain rests on `weak` evidence, let that show
   rather than overselling the case.
4. **What powers it.** Levers (`concentrates_into`): the `leverage_description`
   attr is the point — where the disproportionate force is.
5. **What it opens.** Opportunities (`converts_to`) and supporting insights —
   the upside frame.
6. **What has to be true.** Assumptions (with `attrs.confidence` where stated —
   a `low` one is the soft ground), then the hypothesis under test with its
   falsification criterion verbatim. This is the intellectual honesty section;
   never omit it.
7. **What threatens it.** Risks sorted by probability × impact (high/high
   first), each with its `threatens` rationale and mitigation if the body
   names one.
8. **Provenance footer.** Graph name, generation date, scope counts:
   "N nodes / M edges in scope — X ratified, Y proposed, Z contested. Sources:
   <distinct provenance.source values>." This footer is what makes the artifact
   trustworthy and regenerable; never skip it.

Altitude discipline: this frame keeps diagnoses, levers, opportunities,
signals, deliberation. It **drops** sibling initiatives, artifact nodes, and
anything reachable only through `structural` edges — those belong to other
frames (roadmap, portfolio), not "why".

## Step 4 — Deliver

Write the narrative to `narratives/<date>-why-<slug>.md` (create the directory
if needed) and present it. If the user names an output target (Notion, slides),
hand the markdown to that flow — the graph stays the source of truth either
way; the file is a disposable rendering.

## Notes

- A broken chain (bet with no diagnosis, initiative with no `realizes`) is
  reported plainly in place — "no recorded reason" — and flagged at the end
  with a suggested `propose_edge` to close it.
- Re-run after the graph changes; the artifact is cheap to regenerate and
  should never be hand-edited (edit the graph instead).
