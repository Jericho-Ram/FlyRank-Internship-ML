# Capstone Report — Refresh/Content Opportunity Scoring

- **Author:** Edo Sanjaya P
- **Lane:** Refresh/Content Opportunity Scoring
- **Repo:** github.com/Jericho-Ram/FlyRank-Internship-ML
- **Date:** 18-08-2026

> The eight sections mirror the Pass / Needs-Work rubric axes, so nothing here is optional.

## 1. Problem framing

The unit is one page — one `content_id`. The output is a ranked queue, and the action is an
editor working down the top `50` and refreshing what they find there.

Both error types are present, but they are not equally visible. A false positive reaches the
queue, an editor opens the page, sees it does not need work, and moves on — the mistake gets
inspected and discarded, and the cost is the time spent looking. A false negative never enters
the top `50`. Nothing prompts anyone to open it, so the page keeps declining and no one finds
out it was missed.

The asymmetry flatters the metric, where Precision@50 measures only the good half on
inspection. The biggest cost is the unseen error that couldn't be taken care of.

## 2. Data safety

The bundled anonymized slice, `data/raw/content_refresh_anonymized.csv`, `30,000` rows at one
row per `content_id`. Nothing client-identifying is in it: `client_id` is a pseudonymous hash
and appears nowhere in the feature matrix.

Excluded on purpose:

| Field | Why |
|---|---|
| `trend_direction` | The label itself. |
| `trend_pct` | The label's magnitude — the same quantity, unrounded. |
| `impressions_last_30d`, `clicks_last_30d`, `sessions_last_30d` | The window the label is computed from. |
| `impressions_prev_30d`, `clicks_prev_30d`, `sessions_prev_30d` | The comparison window on the other side of the same subtraction. |
| `client_id` | Grouping only, for the split. Never a feature — a model that learns the client is not predicting decline. |

The label is `is_declining_label`, set where `trend_direction` reads `down`.

One exclusion is a population filter rather than a column drop, and it is the one that matters
most. Rows with `impressions_prev_30d == 0` cannot decline — impressions do not fall below zero
— so their label is fixed before any feature is read. `3,388` rows (`11.3%`) are in that state.
They are removed in `scripts/01_prepare_features.py`, leaving `26,612`. Failure mode 1 in
`outputs/known_failure_modes.md` records what including them did to the result.

## 3. Baseline

A transparent weighted rule in `scripts/02_baseline_score.py`, built before any model. Four
percentile-ranked components: visibility at `0.40`, freshness risk at `0.30`, position
opportunity at `0.25`, depth gap at `0.05`. It reads top to bottom and can be argued with,
which is the point of a baseline.

On the filtered population, scored on the same split and the same metric as the models:

| | ROC AUC | Precision@50 |
|---|---:|---:|
| baseline_rules | `0.380` | `0.460` |
| base rate | — | `0.611` |

It is a fair comparison in construction and a useless one in result. `0.380` is below random,
so beating it demonstrates nothing — a model can clear this bar and still be worthless. I am
not presenting the gap as evidence. Reporting a baseline that fails is more useful than
quietly replacing it with one that flatters the model.

## 4. Model / analysis

Three models trained on the same features and split: logistic regression, decision tree,
random forest. Ranking is the right shape for the lane — an editor works down a queue, so what
matters is the order, not a calibrated probability per page.

`18` numeric features and `8` categorical, expanding to `52` columns after one-hot encoding.
Numeric: search volume, competition, CPC, word and character count, log-transformed
`impressions_90d`, `clicks_90d`, `sessions_90d` and `ai_sessions_90d`, days with impressions,
days with sessions, content age, days since last update, CTR, average position, engagement
rate, scroll rate, AI traffic share. Categorical: competition level, content type, main intent,
and the age, freshness, word-count, impression and position tiers.

Counts are log-transformed because impressions and clicks are heavy-tailed; untransformed, a
handful of large pages dominate the scale.

Target, in one sentence: `is_declining_label` is `1` where the slice's own `trend_direction`
reads `down`. It is a proxy, not a prediction — the label describes a movement already
measured in the same window the features come from, so this ranks pages that look like ones
that declined, not pages that will.

## 5. Evaluation

Split by client holdout: whole `client_id` groups on one side or the other, so no client
appears in both train and test. A model that memorises one client cannot then score itself on
that client's other pages. `25,401` train rows, `1,211` test rows.

On the filtered population, all four scored on the same split:

| Model | ROC AUC | Precision@50 |
|---|---:|---:|
| logistic_regression | `0.583` | `0.740` |
| random_forest | `0.565` | `0.600` |
| decision_tree | `0.505` | `0.540` |
| baseline_rules | `0.380` | `0.460` |

Base rate `0.611`. The script selects on ROC AUC, which picks logistic regression here.

The errors say more than the table. Feature importance sits on `log_impressions_90d` and
`log_clicks_90d` by a clear margin, so the ranking is driven by traffic volume: a large stable
page and a shrinking page look alike if their ninety-day totals match. Size, not movement.

One result worth recording because it is not obvious: model selection flips with the
population. Unfiltered, ROC AUC picks random forest at `0.750`; filtered, it picks logistic
regression at `0.583`. The choice of best model was partly an artifact of the rows that could
not decline.

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

The calculation is good as a filter and not as a ranking. Excluding pages that cannot decline
is a real improvement to how the queue is built, it reproduces from a clean clone, and it needs
no model. The ordering inside that filtered set is not yet worth acting on — on the same split
it shows no lift over the base rate, so a page's position in the queue does not tell an editor
which one to open first.

What the ranking needs is better input, not a better algorithm. The features on hand describe
traffic volume over ninety days, and the model leans hardest on exactly those. Volume is not
movement. Until the inputs carry something about how a page is changing rather than how large
it is, the ordering will keep reproducing size.

## 8. Reproducibility

From a fresh clone, on the bundled slice — no warehouse access or token needed:

```
pip install -r requirements.txt
python scripts/01_prepare_features.py
python scripts/02_baseline_score.py
python scripts/03_train_model.py
python scripts/04_evaluate_and_export.py
```

Or `python scripts/run_all.py` for all five steps including the PDF.

Determinism, and its limit. Two filtered runs in separate directories return identical figures,
so the pipeline is deterministic within a stack. Across stacks it is not: `GUIDE.md` records
random forest Precision@50 moving between roughly `0.68` and `0.74` depending on library
versions, and that movement is what failure mode 4 is about. A re-run reproducing entry 1's
unfiltered table exactly — base rate `0.542`, RF ROC AUC `0.750`, RF Precision@50 `0.740`,
hand rule `0.627` and `0.240` — confirms the filtered and unfiltered figures both hold.

Any figure quoted from this repo should be regenerated from a clean clone first, and the
runtime recorded beside it. `outputs/model_report.md` is generated; the retraction notice it
carries is not, which is a gap noted in failure mode 4.

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
