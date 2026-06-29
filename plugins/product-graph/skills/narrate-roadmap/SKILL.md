---
name: narrate-roadmap
description: 'Render a now/next/later roadmap from the product-graph. Two variants: an INTERNAL why-led story (the through-line + why-this-horizon from each initiative''s sequencing_rationale + the leading→lagging indicator ladder) so an observer understands why we''re doing things and why now vs later; and an EXTERNAL outcome-led publication for customers/GTM. Buckets come from delivery_status; never invents dates. Use when: roadmap, roadmap story, now next later, why now why later, what is coming, gtm roadmap, customer roadmap, public roadmap, narrate roadmap.'
---

# Narrate Roadmap (renderer: now / next / later)

You are a **renderer** over the `product-graph` MCP server. The roadmap is a
projection of the **execution layer** by horizon. Two disciplines define it:
**honest time** (the graph has no dates — timing comes only from
`delivery_status` buckets, never invented quarters) and **quote-don't-invent**
(the connective tissue is each initiative's recorded `sequencing_rationale` and
edge rationales — you weave and order, you do not fabricate reasons).

## The two variants — pick one (ask, or infer from the request)

- **`internal`** (default) — **the why-led story.** For a stakeholder/peer/
  leadership observer who needs to understand *why we're doing things, why now,
  and why something is later.* Leads with the through-line; each item carries
  its **why-this-horizon** (quoted `sequencing_rationale`), **what it drives**
  (outcomes), and the **leading signal** we'll watch. Plain language, no graph
  ids/vocabulary — questions route to the PM, not the model.
- **`external`** — **the customer publication.** Outcome-led, problem-language,
  **ratified nodes only**, only `now`/`next`/`shipped` work, no rationales, no
  sequencing reasons, no internal sources. When in doubt, leave it out.

## Hard rules (non-negotiable)

1. **Read-only. No write path.** A gap is reported, with a suggested
   `propose_*` fix (internal) or simply omitted (external).
2. **Never invent dates.** Buckets come from `delivery_status`
   (under consideration / now / next / later / shipped). Resist quarter labels
   even under pressure — "Now/Next/Later" survives slip; "Q3" becomes a broken
   promise. User-supplied dates are *their* input, marked as such (internal only).
3. **Quote-don't-invent.** Why-this-horizon is the verbatim/compressed
   `sequencing_rationale`; what-it-drives and the leading signal come from
   `expected_to_drive` / `measures` edges. No reason that isn't in the graph.
4. **Render honestly.** A missing `sequencing_rationale` is shown as
   "— not recorded" (internal), not papered over. Diagnoses, levers, risks,
   and deliberation nodes never appear here.

## Step 1 — Gather (read verbs)

1. **The items + their buckets + their why:** `query_nodes(family="execution")`
   — each carries `delivery_status` (the bucket, the human's recorded judgment)
   and `attrs.sequencing_rationale` (the why-this-horizon).
2. **The through-line:** the bet each item `realizes`
   (`query_edges(type="realizes")`) — usually one shared bet; that shared anchor
   is the spine of the story.
3. **What each drives:** `query_edges(type="expected_to_drive")` from each item
   → `outcome` targets (the customer value) and any `indicator` targets.
4. **Leading signals (BOTH paths — or you under-report):**
   - *direct:* an `expected_to_drive` edge whose target is an `indicator` with
     `attrs.lead_lag == "leading"` (e.g. pricing → Time to First Lap).
   - *outcome-mediated:* for each driven `outcome`, the indicators that
     `measures` it (`query_edges(type="measures")`) with `lead_lag == "leading"`.
   Dedup the union per item.
5. **The lagging stake (bottom of the ladder):** the `business_result`(s) the
   outcomes ladder to (`causal_descendants` of a bet/outcome), plus any
   long-horizon outcome that is itself a stake (e.g. a partner-GTM target).

## Step 2 — Bucket

From `delivery_status`, in horizon order: **Now · Next · Later** (plus
**Under consideration** and **Recently shipped** when populated). An item with a
value outside the current enum is **unbucketed** — internal lists it under
"needs a horizon"; external omits it. Don't second-guess the human's bucket.

## Step 3 — Compose

### internal (the why-led story)

1. **The through-line** (one paragraph). The shared bet every item realizes;
   the logic of the sequence (what *now* is for vs *next* vs *later*); and how
   the **leading signal shifts across the horizon**, laddering to the lagging
   needle. This is the spine the buckets hang on.
2. **Per bucket, per item:** the **initiative** (the thing that carries the
   horizon) · **why this horizon** (quoted `sequencing_rationale`, or
   "— not recorded") · **what it drives** (outcomes, plainly) · **leading
   signal(s)**. Order within a bucket by your judgment of leverage. Where a
   `sequencing_rationale` resolves a tension (e.g. "crucial soon, but needs
   deliberate design" → why it's *next* not *now*), name it — that's the
   richest signal.
3. **How we'll know — the indicator ladder.** Leading signals by horizon →
   the lagging stake. Every leading signal is an early read on the lagging one.
4. **Honest gaps.** Items missing a `sequencing_rationale`; enablers that carry
   no leading indicator *by design* (de-risking / capacity work — state it,
   don't force a metric). Close with a one-line health note when clean.

### external (the publication)

Now/Next/(Recently shipped) of **customer-phrased outcomes** ("X becomes Y"),
one line of why-it-matters per item drawn from a translated edge rationale.
No sequencing reasons, no indicators, no ids. Open with a one-paragraph
through-line in customer language.

## Step 4 — Deliver

**internal — persist as a snapshot (the board's prose companion).** Exactly like
`analyze-graph`, save it by calling the **`save_analysis` MCP verb** (NOT a graph
write — this seals nothing, creates no node/edge, and works over the hosted
connection). The roadmap view reads the latest `kind="roadmap"` snapshot and
renders it beside the live board. Call `save_analysis` with:

- `kind: "roadmap"` — keeps it separate from `analyze-graph` portraits in the same store
- `generated_by: "narrate-roadmap skill"`
- `metrics: { "now": <n>, "next": <n>, "later": <n>, "under_consideration": <n>, "shipped": <n> }`
- `narrative: "<the full markdown story>"`

A successful save returns `{ "id": ..., "generated_at": ..., "kind": "roadmap" }`;
re-running creates a new row and the board shows the newest. **Always use the
`save_analysis` MCP verb** — it works over the hosted connection
(product-graph.app) and is the only path for remote users. Do **not** call a
loopback/`localhost` HTTP route: there is no loopback when you're connected to a
hosted server, and the attempt just fails. (The same persistence is *also*
reachable via `POST /api/analysis` **only** when you're self-hosting a local
checkout — never on a hosted connection.)
Also write the markdown to `narratives/<date>-roadmap-internal.md` for portability.

**external — publication file.** Write to `narratives/<date>-roadmap-external.md`
and hand to its destination — the external variant is *not* shown in the board UI.

Re-run on `delivery_status` / `sequencing_rationale` / edge changes — the buckets
and the why shift with the graph, which is the point.

**Footer (inside the narrative):** internal — graph name, date, item counts by
bucket, and "connective tissue quoted from each initiative's sequencing_rationale
and expected_to_drive rationale." External — "Plans evolve as we learn; this
reflects our direction as of <date>." and nothing else.

## Notes

- An initiative with no `expected_to_drive` outcome/indicator (e.g. a pure
  de-risking pilot or a capacity-clearing programme) has no leading signal *by
  design* — say so plainly; don't invent one.
- The board UI renders this same projection live; this skill is its prose
  companion. Keep them consistent — both lead with the initiative and surface
  the same leading signals (direct + outcome-mediated).
