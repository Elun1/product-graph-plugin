---
name: narrate-opportunities
description: 'Render an Opportunity Solution Tree from the product-graph for discovery/design/research audiences: outcomes at the root, opportunities beneath, bets and their tests as leaves — plus the discovery backlog (evidenced opportunities no bet addresses) and solution-first flags (bets with no parent opportunity). Use when: opportunity tree, opportunity solution tree, ost from the graph, discovery view, opportunity space, unaddressed opportunities, what should research look at, narrate opportunities.'
---

# Narrate Opportunities (renderer: the Opportunity Solution Tree)

You are a **renderer** over the `product-graph` MCP server. This frame answers
the discovery/design/research question: **"which problems are worth exploring,
and which are already being attacked?"** It is the Teresa Torres Opportunity
Solution Tree, derived mechanically: `outcome ← contributes_to/leads_to ←
opportunity ← converts_to → bet ← tests → hypothesis`. The two off-tree lists
— evidenced opportunities nothing converts, and bets with no parent
opportunity — are where this artifact earns its keep.

## Hard rules (non-negotiable)

1. **Read-only. No write path.**
2. **Quote, don't invent.** Tree edges come from the graph; evidence weight is
   counted, not asserted — and where edges carry `attrs.evidence_strength`,
   weigh by it, not just the raw count (three `weak` signals are not one
   `strong` one). An opportunity's "worth exploring" rank is your judgment —
   applied to the graph's evidence counts and strengths, and labeled as
   judgment.
3. **Render trust state honestly.** `proposed` opportunities are part of the
   discovery conversation — include them, marked. (Contrast with the external
   roadmap, which would drop them.)
4. **Spine discipline.** Tree structure uses `causal` + `epistemic` edges
   only; a `measures` or `realizes` edge never creates tree placement.

## Prerequisites

The `product-graph` MCP server connected (via the plugin, or `claude mcp add` /
`.mcp.json` on a local checkout) — read-only suffices.

## Step 1 — Resolve the root

Input is an outcome / business_result id or title fragment; with no argument,
use every ratified `outcome` as a root (one tree per outcome, shared
opportunities noted, not duplicated).

## Step 2 — Gather (read verbs)

1. **Opportunities:** `query_nodes(type="opportunity")` — all statuses.
2. **Tree placement:** per opportunity, `query_edges(source_id=...)` — which
   outcome it `contributes_to`/`leads_to` (placement) and which bet it
   `converts_to` (solutions). An opportunity with neither placement edge
   floats: show it in an "unplaced" group rather than guessing a parent.
3. **Evidence weight:** per opportunity,
   `neighbors(id, categories=["epistemic"], direction="in")` — supporting
   signals/insights, and any `contradicts` (an actively-contradicted
   opportunity is a discovery finding in itself).
4. **Solution leaves:** per converted bet, its `tests` edges → hypotheses
   (with `falsification` attrs) and experiment artifacts — the testing state
   of each solution branch.
5. **The two off-tree lists:**
   `find_gaps(node_type="opportunity", direction="out", edge_types=["converts_to"])`
   → the **discovery backlog**; bets with no incoming `converts_to`
   (`query_nodes(type="bet")` minus `query_edges(type="converts_to")` targets)
   → **solution-first flags**.

## Step 3 — Compose

1. **The tree.** Indented markdown per root (Mermaid `graph TD` if the user
   wants a visual): outcome → opportunities → bets → hypotheses/tests. Per
   opportunity, inline annotation: evidence weight ("3 signals, 1 insight —
   mostly weak" when strengths are recorded), trust state, conversion state
   (→ bet / **unconverted**).
2. **The discovery backlog.** Evidenced-but-unconverted opportunities, ranked
   by your judgment of evidence weight × outcome importance — labeled as
   judgment. This is the section research plans a quarter around.
3. **Solution-first flags.** Bets no opportunity converts into — working
   backwards candidates: either propose the missing opportunity (and the
   `converts_to` edge) or admit the bet is solution-looking-for-problem.
4. **Contradicted / contested opportunities.** Where the evidence is fighting
   the tree — the "stop exploring this" candidates.
5. **Provenance footer.** Graph name, date, scope counts
   (ratified/proposed/contested), regenerate hint.

## Step 4 — Deliver

Write to `narratives/<date>-opportunity-tree<-slug>.md` and present it. All
follow-ups route through `propose_* → ratify_*` (e.g. "propose opportunity X
under outcome Y, then a converts_to once a bet forms").

## Notes

- A thin tree is a finding: one opportunity per outcome means the team is
  jumping from outcome to solution without mapping the space — say so.
- Don't flatten nuance: an opportunity contributing to two outcomes appears
  under both with a cross-reference, because that coupling is real and
  discovery should see it.
