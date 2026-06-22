---
name: narrate-strategy
description: 'Render the full strategy narrative from the product-graph as a Rumelt kernel: the challenge (diagnoses + evidence), the guiding policy (levers), coherent actions (bets + the initiatives realizing them), what we are explicitly not doing, and what would change our mind. Board/steering altitude, prose-led. Use when: strategy narrative, strategy memo, board narrative, the strategy in prose, rumelt kernel, write up the strategy, narrate strategy, strategy document from the graph.'
---

# Narrate Strategy (renderer: the Rumelt kernel)

You are a **renderer** over the `product-graph` MCP server. This frame answers
the board/steering question: **"is the strategy sound — does the reasoning
hold?"** The catalog already encodes Rumelt's kernel structurally —
`diagnosis —calls_for→ lever/bet`, `lever —concentrates_into→ bet`,
`bet ← converts_to ← opportunity` — so the memo is a projection, not an
invention. Unlike the other renderers, this one is **prose-led**: the output
is an essay that weaves the nodes together, not a sectioned list.

## Hard rules (non-negotiable)

1. **Read-only. No write path.**
2. **Quote, don't invent.** The argument's joints are edge rationales. You
   choose the weave and the emphasis; the graph supplies every claim. A causal
   link with no edge behind it may not appear in the essay.
3. **Render trust state honestly — in plain language.** A kernel resting on
   not-yet-committed diagnoses says so as prose ("our view here is still
   forming"), never as trust-state labels or graph vocabulary. The audience
   is a board: the graph is the silent source, never the subject.
4. **Spine discipline.** The kernel is built from `causal` + `epistemic`
   edges. `grounding`/`threat` material appears only in the "what would change
   our mind" section, never as causal argument.

## Prerequisites

The `product-graph` MCP server connected (via the plugin, or `claude mcp add` /
`.mcp.json` on a local checkout) — read-only suffices.

## Step 1 — Gather the kernel (read verbs)

1. **Diagnoses:** `query_nodes(type="diagnosis")`. For each, its evidence:
   `neighbors(id, categories=["epistemic"], direction="in")` (signals/insights),
   and what it calls for: `query_edges(source_id=id, type="calls_for")`.
2. **Levers:** `query_nodes(type="lever")` — each carries
   `attrs.leverage_description`, the where-the-power-is sentence. Follow
   `concentrates_into` edges to the bets.
3. **Bets:** `query_nodes(type="bet")` — *all* lifecycles. `active`/`shipped`
   are the coherent actions; `rejected` and `considered` feed the
   not-doing section. For active bets, find realizing work:
   `query_edges(target_id=bet_id, type="realizes")`.
4. **The downstream stakes:** `causal_descendants(bet_id, depth=3)` for the
   outcome → business_result chain — what winning looks like.
5. **The epistemic edge cases:** `query_nodes(type="hypothesis")` (with
   `falsification` attrs) and the top assumptions
   (`query_edges(type="assumes")` → resolve targets).
6. **The turning points:** `query_nodes(type="decision")` — the ratified
   choices. For each, what it chose (`query_edges(source_id=id,
   type="selects")`), what it ruled out (`query_edges(source_id=id,
   type="rules_out")`), and the evidence behind it
   (`neighbors(id, categories=["epistemic"], direction="in")`). The
   `rules_out` rationale is the *written* refusal — stronger than a bet's
   bare `lifecycle=rejected`.

## Step 2 — Compose (Rumelt's order, as prose)

1. **The challenge** (diagnosis). Weave the diagnoses into one coherent
   account of what is really going on and where the leverage is. Anchor each
   claim in its strongest evidence — concrete numbers from signal bodies are
   the spine of credibility. Where diagnoses refine or contribute to each
   other (`refines`, `contributes_to`), let the prose follow those edges.
2. **The guiding policy** (levers). Each lever's `leverage_description`,
   woven: where we have disproportionate power and why concentrating force
   there beats spreading it.
3. **Coherent actions** (bets). Each active bet, the `concentrates_into` and
   `calls_for` rationales that justify it, and — one clause each — the
   initiatives that realize it. Show the actions *cohere*: they reinforce
   each other rather than compete (the shared levers make this argument).
4. **What we are explicitly not doing.** Rejected bets (`lifecycle=rejected`)
   with their rationale, and considered-but-not-chosen options. Where a
   `decision` ruled an option out, quote *its* `rules_out` rationale — the
   recorded "we chose X over Y, at this time, because" is the strongest form
   of a named refusal. A strategy that names its refusals reads as a strategy,
   not a wish list. Where a decision supersedes an earlier one
   (`supersedes`), the prose can note the turn ("we previously held … ; the
   evidence moved us to …") without dwelling on it.
5. **What would change our mind.** Hypotheses with their falsification
   criteria verbatim, and the load-bearing assumptions. Where an assumption
   carries `attrs.confidence: low` — or its support is only `evidence_strength:
   weak` — say so plainly: a low-confidence assumption underwriting the kernel
   is exactly where the strategy is most exposed. This section is what
   separates a kernel from a pitch — never omit it.
6. **Footer.** Two lines max: named sources and the generation date. No
   counts, no graph mechanics — boards don't care how many nodes back the
   argument; the argument has to carry itself.

## Step 3 — Deliver

Write to `narratives/<date>-strategy-kernel.md` and present it. For a deck,
hand the markdown to the slides flow — the file is a disposable rendering;
the graph stays the source of truth.

## Notes

- A kernel gap is a finding, not an embarrassment to smooth over: a bet no
  diagnosis calls for ("action without diagnosis"), a diagnosis nothing acts
  on, a lever concentrating into nothing. Note each inline and list them at
  the end with suggested `propose_edge` fixes.
- Target length: 1.5–2 pages. The graph may hold 30 diagnoses-and-signals;
  the essay holds the five that carry the argument. Selection is your
  judgment — but selection means omitting, never inventing.
