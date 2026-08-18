# Capstone Report — Refresh/Content Opportunity Scoring

- **Author:** Edo Sanjaya P
- **Lane:** Refresh/Content Opportunity Scoring
- **Repo:** github.com/Jericho-Ram/FlyRank-Internship-ML
- **Date:** 10-08-2026

> The eight sections mirror the Pass / Needs-Work rubric axes, so nothing here is optional.

## 1. Problem framing

What decision does this support? Name the unit of analysis (page, client, day…), the output
(score, rank, cluster, report), the action a human takes from it, and the cost of a wrong
call. Why does data/ML help here at all?

## 2. Data safety

Which data you used and which columns you deliberately excluded (and why). Leakage risks you
considered — especially label-derived fields (`trend_direction`, `trend_pct`) and pseudonymous
IDs (grouping only, never features). Confirm nothing client-identifying appears anywhere in
`work/`.

## 3. Baseline

The transparent rule or score you built first. Why it's a fair comparison, and its numbers on
the same data and metric as your model.

## 4. Model / analysis

Your method and why it fits the lane. The exact feature list (and what you left out on
purpose). The target or proxy definition, in one sentence.

## 5. Evaluation

Your split (grouped by client? time-aware?) and why. Metrics, model vs baseline **on the same
split**. What the errors look like — a short error analysis beats a big metric table.

## 6. Interpretation

The first version of this model looked strong, and the reason it looked strong was an artifact. Pages with zero 
impressions cannot decline. They have no traffic to lose, so their label is fixed before any feature is read. The model 
had learned to find them. Once those rows were excluded, most of the apparent signal went with them — the model had been 
separating pages that could not decline from pages that could, not ranking decline risk among pages actually at risk.

On the valid population, logistic regression scores ROC AUC `0.583` and Precision@50 `0.740` — the model the script
selects by ROC AUC once the zero-impression rows are gone. Unfiltered, that selection goes to random forest instead.
Quoted together, they describe a model that ranks slightly better than chance and concentrates decline into the top `50` 
only modestly above the base rate 
of `0.611`. The hand rule is not a useful comparison. On the same population it scores below random, which means clearing 
it demonstrates nothing. A model can beat it and still be worthless. I am not presenting the gap as evidence.

The feature importances say something more useful than the score does. `log_impressions_90d` and `log_clicks_90d` are the 
heaviest by a clear margin. The model is leaning on traffic volume: it ranks by size, not by movement. A big page and a 
shrinking page look the same to it if their ninety-day totals match.

Failure modes are documented in `outputs/known_failure_modes.md`.

## 7. Recommendation

The ranked actions or decisions your output supports, and how a FlyRank editor would use them
tomorrow. State your confidence and the limits explicitly.

## 8. Reproducibility

The exact commands to re-run everything from a fresh clone, your random seeds, and your
environment (`pip freeze` highlights or `requirements.txt` deltas).

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
