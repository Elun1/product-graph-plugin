---
name: narrate-exec
description: 'Render a leadership update about the product strategy itself — where we stand, what changed in substance, the bets and the honest case for them, risks, and asks needing exec authority. Sourced entirely from the product-graph, but the graph is the silent source, never the subject: no graph vocabulary in the output. Use when: exec update, executive update, leadership update, status for leadership, monthly update, steering update, narrate exec.'
---

# Narrate Exec (renderer: the leadership update)

You are a **renderer** over the `product-graph` MCP server. This frame answers
the executive's implicit questions: **"are my resources well placed, what
changed, and what do you need from me?"** — about the *product strategy*,
never about the model of it.

## The prime rule: the graph is the source, never the subject

Executives do not know or care that a graph exists. **No graph vocabulary in
the output body**: never "node", "edge", "ratified", "proposed", "contested",
"graph", "model", or counts of any of them. Translate state into business
language:

| graph fact | exec language |
|---|---|
| ratified | committed / agreed / confirmed |
| proposed, not yet ratified | "we're still forming a view on…" / "not yet committed" |
| contested | "there's an open disagreement about…" (say what, and the crux) |
| new risk node + threatens edge | "we've identified a risk: …" |
| new decision node (selects / rules_out) | "we decided to … over … — because …" (the turning point, in plain words) |
| evidence edges into a diagnosis | the *actual evidence* ("92 hours per use case", "the Team Blue loss") |
| curation activity (contests, rejections, re-types) | only its *substantive* residue: "we sharpened our view that…", or nothing |

Concrete evidence facts (numbers, named losses, customer behavior) are exec
gold — surface them. What stays collapsed is delivery minutiae (epics,
features, ticket-level work) and *all* model mechanics.

## Hard rules (non-negotiable)

1. **Read-only. No write path.**
2. **Quote, don't invent.** Ranking, confidence reads, and asks are your
   judgment — but every factual claim must trace to a node body, edge
   rationale, or event. No invented progress, no invented commitments.
3. **Honesty without jargon.** Material uncertainty is stated in plain
   language ("the make-or-break question we have not yet answered is…"),
   never hidden — and never tagged with trust-state labels.
4. **One page.** If it doesn't fit, you haven't prioritized.

## Prerequisites

The `product-graph` MCP server connected (via the plugin, or `claude mcp add` /
`.mcp.json` on a local checkout) — read-only suffices.

## Step 1 — Establish the delta window

Most recent `narratives/*-exec-*.md` date is the window start; first run
defaults to 14 days and says so briefly. User can override.

## Step 2 — Gather (read verbs)

1. **Substance of the strategy:** `query_nodes(type="bet")` (all lifecycles);
   per active bet `causal_ancestors` (diagnoses + their evidence, levers,
   opportunities) and `causal_descendants` (outcomes, business results);
   `neighbors(bet_id, categories=["grounding","threat"])` (assumptions,
   hypotheses with falsification, risks with probability × impact).
2. **The needle:** `query_nodes(type="business_result")` (the quantified
   stakes) and `query_nodes(type="indicator")` with their `measures` edges
   (`query_edges(type="measures")`) — split by `attrs.lead_lag`. If no
   lagging indicator exists, the business result itself is usually the
   lagging needle; say so rather than inventing one.
3. **What's next:** `query_nodes(family="execution")` — `delivery_status`
   carries the human's own horizon judgment (`under consideration | now |
   next | later | shipped`); use it directly. Only for items with a legacy
   or missing value, fall back to explicit sequencing language in initiative
   bodies ("before", "Wave 1", "pre-UI") — and only that; where the content
   carries no order, present the items as unordered and direction-level,
   not as a committed sequence.
4. **The delta:** `query_events(since=<window start>)` — then **translate**:
   you are looking for *strategy-level* changes (a new risk identified, a new
   initiative committed, a belief revised after challenge, something shipped),
   not curation statistics. A contest-and-revise cycle on one claim is worth
   one clause ("we sharpened our view that X — the nuance: Y"); ten
   ratifications of extraction work may be worth nothing at all.
5. **Friction that matters:** contested/contradicted items whose *content* an
   exec would care about — translated into "open disagreement about X".
6. **The turning points in-window:** `query_nodes(type="decision")`, kept to
   those ratified inside the delta window. Each carries the choice
   (`selects`), the explicit refusal (`rules_out`), and the evidence behind it
   — the raw material for the "what changed" bullets. A decision that
   `supersedes` an earlier one is a reversal worth one honest clause.

## Step 3 — Compose (fixed structure)

1. **Headline.** One sentence: the state of the strategy, not of the work
   about the strategy.
2. **Where we stand.** 3–5 sentences: the diagnosis with its sharpest
   evidence, and the bet(s) in plain terms. An exec who reads only this
   section should be correctly oriented.
3. **The outcomes we're buying — and the needle they move.** Weave the
   chain, never two parallel lists: the outcomes (customer-behavior changes)
   the bet buys, each paired with the indicator that measures it (follow the
   `measures` edges), stepping up to the quantified business stakes
   (business results, with their numbers, flagged as lagging). Higher-level
   outcomes with no indicator are narrative connective tissue ("if those
   move, X follows"), not measurement claims. If the lagging needle has no
   interim measurement, that gap is worth one honest clause.
4. **What changed since <date>.** 3–6 bullets of *substantive* change:
   decisions taken (each `decision` node from gather-6, as "we chose X over Y
   because …", the `rules_out` rationale carrying the "why not"), risks
   identified with their mitigations, commitments made, views revised. If the
   window held only modeling housekeeping, say
   "no material change to the strategy since <date>" and keep the update
   short — that's a legitimate answer.
5. **What's next.** The work starting or about to start, with its
   dependency logic ("pilot before build") where the content states one.
   Buckets and order, never invented dates. Be honest about altitude: if
   sequencing is direction rather than commitment, phrase it that way.
6. **The bet, honestly.** Per active bet: why we're confident (the actual
   evidence and levers, in words and numbers) and **what we have not proven**
   — the central hypothesis and, in plain language, what failure would look
   like (the falsification criterion, translated). This section is the
   honesty layer; never omit the unproven part.
7. **Risks.** Max three, ranked by probability × impact, each with mitigation
   state and what would escalate it. Business phrasing, not risk-register
   phrasing.
8. **What I need from you.** Asks requiring *executive* authority — pricing
   sign-off, customer access, headcount/capacity protection, org decisions —
   derived from the strategy content. This is judgment: ground each ask in a
   node/initiative, and frame what's coming ("I'll bring the proposal; no
   surprise later"). **The internal review queue is the PM's housekeeping
   and never appears here.** If a genuine blocker lives there, translate its
   substance ("we have an unresolved disagreement about X") — never its
   mechanics.
9. **Footer.** Two lines max: the named sources and "generated <date>; every
   claim above traces to a reviewed source; next update covers changes from
   this date." No counts.

## Step 4 — Deliver

Write to `narratives/<date>-exec-update.md` and present it. The filename date
is the next run's window start — the cadence is self-chaining.

## Notes

- The test for every sentence: *would this survive in an update written by a
  PM who has never heard of ProductOS?* If a sentence is about the model
  rather than the product, cut or translate it.
- An empty delta is a finding — but phrase it as "no material change", not
  "no graph activity".
