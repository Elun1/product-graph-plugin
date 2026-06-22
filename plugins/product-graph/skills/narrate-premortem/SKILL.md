---
name: narrate-premortem
description: 'Render a one-page pre-mortem for a bet from the product-graph: what has to be true (assumptions, tested vs naked), the hypothesis with its falsification criterion, the risks ranked by probability × impact with mitigation state, and the naked list. Use when: premortem, pre-mortem, what could kill this, risk story for a bet, what has to be true, naked assumptions, stress the bet, narrate premortem.'
---

# Narrate Pre-mortem (renderer: the risk story of a bet)

You are a **renderer** over the `product-graph` MCP server. This frame answers
the steering question: **"what has to be true for this bet, what are we
testing, and what are we just hoping?"** The deliberation family makes it
mechanical: `assumes` edges are the load-bearing beliefs, `tests` edges are
the checks, `threatens` edges are the dangers — and the gaps between them
(assumptions nothing tests, risks nothing mitigates) are the punchline.

## Hard rules (non-negotiable)

1. **Read-only. No write path.** The output *recommends* tests and mitigations
   as `propose_*` actions; it never creates them.
2. **Quote, don't invent.** Falsification criteria verbatim. Probability ×
   impact from the risk's `attrs`, never your gut. **Evidence strength** from
   the edge's `attrs.evidence_strength` (`weak`/`moderate`/`strong`) and
   assumption **confidence** from the node's `attrs.confidence`
   (`low`/`medium`/`high`) — read them, never guess them; absent = *unstated*
   (which is itself a finding), never silently "strong." Mitigations only where
   the risk body or a `tests` edge names one.
3. **Render trust state honestly.** A `proposed` risk is marked; a `contested`
   assumption is a headline, not a footnote.
4. **Naked and thin are both findings.** Three rungs, loudest first:
   **naked** (an assumption with *no* incoming `tests`/`is_evidence_for` edge,
   or a high×high risk with no named mitigation) → **thinly evidenced** (a
   belief that *is* tested but only by `evidence_strength: weak` edges, or
   carries `confidence: low`) → **solid**. The first two rungs are the most
   valuable lines in the artifact. Never pad them away, and never let "tested"
   hide "tested weakly."

## Prerequisites

The `product-graph` MCP server connected (via the plugin, or `claude mcp add` /
`.mcp.json` on a local checkout) — read-only suffices.

## Step 1 — Resolve the target

Input is a bet id or title fragment (`query_nodes(type="bet", text=...)`;
ask if ambiguous). With **no argument**: run the portfolio variant — every
`lifecycle=active` bet as one ranked risk table, naked counts per bet, no
per-bet deep dive.

## Step 2 — Gather (read verbs)

1. **The belief layer:** `neighbors(bet_id, categories=["grounding","threat"],
   direction="both")` — assumptions (`assumes`), hypotheses (`tests` from the
   bet), risks (`threatens` into the bet).
2. **Test coverage per assumption/hypothesis:** for each deliberation node,
   `query_edges(target_id=<id>, type="tests")` — what (artifact, execution
   item, or causal node) actually checks it, **and each edge's
   `attrs.evidence_strength`**. No incoming `tests` edge = **naked**; only
   `weak` edges (or the node's own `attrs.confidence: low`) = **thinly
   evidenced**; a `moderate`/`strong` edge = **solid**. Strength absent on an
   edge = *unstated* — report as thin-by-default-of-evidence, not as solid.
3. **Evidence pressure:** `query_edges(target_id=<id>)` filtered to
   `is_evidence_for` / `contradicts`, reading each edge's
   `attrs.evidence_strength`. Beliefs the graph is actively confirming or
   undermining, and *how hard*. A ratified `contradicts` is the loudest signal
   — a `strong` one is decisive; lead with it if present. A `weak` confirming
   edge is not a reason to relax.
4. **Mitigation state:** risk bodies that name a mitigation, and any
   execution node whose `tests` edge targets the risk.

## Step 3 — Compose (one page, fixed structure)

1. **The bet in one line** + lifecycle.
2. **What has to be true.** Table: assumption (+ `confidence` if stated) ·
   tested by (the `tests` source + its `evidence_strength`, or **— naked**) ·
   evidence pressure (confirming / contradicted / none, with strength). Mark
   the rung — naked / thin / solid — so the eye lands on the soft ground.
3. **The hypothesis under test.** Statement + **falsification criterion
   verbatim** + what runs the test. A hypothesis whose falsification can't
   currently be observed by anything in the graph is itself a finding.
4. **What could kill it.** Risks sorted high×high first: threat rationale,
   probability × impact, mitigation state (named and underway / named only /
   **none**).
5. **The naked & thin list.** Untested assumptions (naked) + assumptions held
   up only by `weak`/`low-confidence` evidence (thin) + unmitigated high risks +
   contradicted beliefs, in one blunt block, naked above thin. Then 2–3
   suggested next checks, each phrased as a `propose_*` action (e.g. "propose an
   experiment_canvas node with a `tests` edge into assumption X, then ratify";
   or "strengthen the weak test on Y"). Strengthening a `weak` edge to `strong`
   is a `propose_change` on its `attrs`, not a new edge.
6. **Provenance footer.** Graph name, date, scope counts
   (ratified/proposed/contested), regenerate hint.

## Step 4 — Deliver

Write to `narratives/<date>-premortem-<slug>.md` and present it. Re-run after
new `tests` edges land — watching the naked list shrink across runs is the
de-risking story itself.

## Notes

- An empty threat neighborhood on an active bet is not "no risk" — it means
  *no one has written the risks down*. Say exactly that.
- Keep the judgment honest: `tests` coverage is structural, not proof the
  test is good. Where the graph states `evidence_strength`, that weakness is a
  **graph fact** — report it as such. Where it's silent, a weak-looking test is
  *your* judgment — flag it as that, distinct from the graph's facts, and
  consider recommending someone record the strength.
