# Capstone Report — <Ranking Signal Analysis>

- **Author:** Joshua Kotze
- **Lane:** Ranking Signal Analysis
- **Repo:** https://github.com/josh777-ops/Fly-Rank-AI/edit/main/work/capstone_report_template.md
- **Date:** 7/21/2026

> Copy this file to `work/capstone_report.md` and fill it in as you build. The eight
> sections mirror the Pass / Needs-Work rubric axes, so nothing here is optional.

## 1. Problem framing

What decision does this support? Name the unit of analysis (page, client, day…), the output
(score, rank, cluster, report), the action a human takes from it, and the cost of a wrong
call. Why does data/ML help here at all?
Decision supported: Which content/structural signals should a FlyRank editor prioritize when trying to improve a page's click performance, given where it already ranks?

Unit of analysis: page (optionally page-day, if the warehouse supports a time dimension for signals like freshness).
Output: a ranked signal report — which signals are associated with CTR, how strongly, and in which direction — translated into an editorial action list.
Action a human takes: an editor decides what to fix first (e.g. rewrite titles, add schema, update stale content) based on which signals show the strongest association.
Cost of a wrong call: wasted editorial time on signals that don't actually move CTR, or missed opportunity on signals that do but were overlooked. Not catastrophic, but has real opportunity cost — this is why "directional, evidence-backed" framing matters more than a single point estimate.
Why data/ML helps: the number of candidate signals and pages is too large to reason about by eye; a model with interpretable outputs (coefficients / SHAP) surfaces which signals matter without requiring a controlled experiment for every one.

## 2. Data safety

Which data you used and which columns you deliberately excluded (and why). Leakage risks you
considered — especially label-derived fields (`trend_direction`, `trend_pct`) and pseudonymous
IDs (grouping only, never features). Confirm nothing client-identifying appears anywhere in
`work/`.
TODO — finalize once the warehouse is queried, but drafted from what's known now:

Tables/columns planned: page/content metadata (title, meta description, heading structure, word count, publish/update dates, template/category) joined with performance metrics (impressions, clicks, average position) from the FlyRank warehouse via hf://.
Deliberately excluded: any client, domain, or URL-identifying fields; raw exports; credentials; anything that would let a reader reconstruct which real site/client this is.
Leakage risks considered:
CTR is the target, so no feature may be a transformation of clicks or CTR itself (e.g. "click rate last week" as a feature, unless explicitly designed as a controlled lagged/momentum feature with strict time-ordering).
Pseudonymous IDs (client/page IDs) used only for grouping in the split, never as model features.
Average position is a control/context variable, not a leak, since it's the condition CTR is being explained under — but this will be justified explicitly in Section 4.
Confirmation: no client-identifying details will appear anywhere in work/ — to be verified again before submission.

## 3. Baseline

The transparent rule or score you built first. Why it's a fair comparison, and its numbers on
the same data and metric as your model.

TODO — to be computed once data is pulled.

Planned baseline: predict CTR using average position alone (a simple regression or lookup-table mapping position → expected CTR, e.g. industry-standard position-CTR curves or an empirical curve fit on this dataset). This is a fair comparison because it represents "no signal analysis needed — just rank" — if the signal model can't beat this, the signals aren't adding explanatory value.

Numbers to fill in: baseline MAE / R² (or chosen metric) on the same split as the model.

## 4. Model / analysis

Your method and why it fits the lane. The exact feature list (and what you left out on
purpose). The target or proxy definition, in one sentence.

Target definition (one sentence): CTR (clicks ÷ impressions) at the page level, chosen over raw clicks to avoid the circularity of "high-ranked pages get more clicks because they're high-ranked," and over movement/position-change to avoid the stricter time-series leakage control that a momentum target would require.

Method: an interpretable model — likely gradient-boosted trees with SHAP values, or a regularized linear/logistic model if the feature set ends up small — chosen because the goal is explaining which signals matter, not squeezing out maximum predictive accuracy.

Candidate feature list (to be confirmed against actual warehouse schema):

Content structure: title length, meta description length/presence, heading count, word count, presence of schema markup
Freshness: days since last update, publish age
Page type/template or content category
SERP features (if available): featured snippet presence, rich result presence
Internal signals: internal link count, page depth in site structure
Control variable: average position (included as context, not treated as "the answer")

Deliberately left out: any click/CTR-derived fields as features (leakage), client/domain identifiers (privacy), and any field requiring interpretation as causal (e.g. no "Google algorithm change" indicators).

## 5. Evaluation

Your split (grouped by client? time-aware?) and why. Metrics, model vs baseline **on the same
split**. What the errors look like — a short error analysis beats a big metric table.

Planned split: time-aware split (train on an earlier date window, evaluate on a later one) rather than a random split, since content signals should generalize forward in time, not just within a snapshot. If the warehouse doesn't support a clean time split, fall back to a grouped split by page/client to prevent leakage across near-duplicate pages.

Metrics: primary metric TBD based on target distribution (e.g. MAE/R² for continuous CTR, or precision@K / lift-over-baseline if CTR is bucketed into an opportunity score). Report model vs. baseline on the same split, and report the task's base rate alongside any precision@K or accuracy figure so a high score can't be mistaken for a high base rate.

TODO — fill in once evaluation is run: metric values, error analysis (which pages/segments the model gets most wrong and why).

## 6. Interpretation

What the model/clusters actually found. Feature importances or cluster profiles in plain
words. Surprises and negative results — a well-understood "no effect" is a valid result.

TODO — fill in after modeling. Plan: report feature importances / SHAP summary in plain language (which signals push CTR up or down, and by roughly how much), explicitly call out any signal that turned out to have no measurable effect (a well-understood null result is a valid finding), and flag anything surprising relative to common SEO assumptions.

## 7. Recommendation

The ranked actions or decisions your output supports, and how a FlyRank editor would use them
tomorrow. State your confidence and the limits explicitly.

TODO — fill in after interpretation. Plan: convert the top signals into a ranked action list (e.g. "pages missing meta descriptions show the strongest CTR gap — prioritize these first") with an explicit confidence/limits statement per recommendation (observed / directional, not causal).

## 8. Reproducibility

The exact commands to re-run everything from a fresh clone, your random seeds, and your
environment (`pip freeze` highlights or `requirements.txt` deltas).

TODO:

Exact commands to re-run from a fresh clone
Random seeds used
requirements.txt / pip freeze highlights

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
