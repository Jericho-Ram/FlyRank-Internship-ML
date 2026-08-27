# Which pages should you refresh first?

A ranking model for content refresh, built for a reviewer who has more pages than time to read them. It scores pages by how likely they are to be declining, so the top of the queue is where attention goes first.

**Read this before the numbers:** the output is a ranked queue for a human editor to work through, not an automated pipeline. It is decision-support. No intervention was run, so nothing here says refreshing a flagged page fixes its decline.

Deployed paper: https://jericho-ram.github.io/FlyRank-Internship-ML/

---

## Setup

Developed on Google Colab, Python `3.12.13`, scikit-learn `1.9.0`. No install needed there.

Locally:

```bash
git clone https://github.com/Jericho-Ram/FlyRank-Internship-ML.git
cd FlyRank-Internship-ML
pip install -r requirements.txt
```

Packages are pinned in `requirements.txt`: pandas, numpy, scikit-learn, matplotlib, reportlab, duckdb, huggingface_hub.

The dataset ships with the repo at `data/raw/content_refresh_anonymized.csv` — 30,000 pseudonymized pages, 44 columns. Nothing to download. Column definitions are in `docs/data-dictionary.md`.

## Usage

Run the full reference pipeline:

```bash
python scripts/run_all.py
```

About a minute. Results land in `outputs/`.

Or run one stage at a time:

```bash
python scripts/01_prepare_features.py
python scripts/04_evaluate_and_export.py
```

The results below come from `work/notebooks/capstone.ipynb`, not from the reference pipeline. Run it top to bottom; it writes charts and the ranked queue to `work/outputs/`.

## Architecture

### The reference pipeline (`scripts/`)

Shipped with the starter repo, run unmodified. It is the comparison point, not the source of the results below.

```text
data/raw/content_refresh_anonymized.csv       30,000 rows x 44 columns
      |
      |  01_prepare_features.py     clean, engineer columns, define the label
      v
data/processed/refresh_feature_vector.csv     52 columns
      |
      |  02_baseline_score.py       hand-rule score with reason codes
      v
data/processed/baseline_refresh_queue.csv
      |
      |  03_train_model.py          logistic regression, decision tree, random forest
      |                             split holds out ~20% of clients, not rows
      v
data/processed/model_predictions.csv + outputs/model_results.json
      |
      |  04_evaluate_and_export.py  ranked queue, charts, report
      v
outputs/refresh_queue.csv, outputs/model_report.md, outputs/charts/
      |
      |  05_build_pdf_report.py
      v
outputs/flyrank_refresh_model_results.pdf
```

### The capstone path (`work/notebooks/capstone.ipynb`)

This is what produced the results below.

```text
data/raw/content_refresh_anonymized.csv       30,000 rows x 44 columns
      |
      |  filter: avg_position > 0                 drops 1,205 unranked rows
      |  filter: impressions_prev_30d > 0         drops 2,191 ranked rows
      v
scored population                             26,604 rows, 31 of 32 clients
      |
      |  two feature sets built once:
      |    permissive  38 cols
      |    strict      34 cols, minus the four window-overlap columns
      v
      |  5-fold GroupKFold on client_id
      |  random forest, logistic regression, and the Week-4 hand rule
      |  on identical folds
      v
work/outputs/capstone_action_queue.csv, work/outputs/charts/
```

**The label** is `is_declining = (trend_direction == "down")`, itself defined from a 30-day-versus-prior-30-day impression change. Because the label derives from `trend_direction`, neither `trend_direction` nor `trend_pct` can ever be a feature. Adding `trend_pct` back sends AUC to `1.000`, which is how the harness proves it can detect a leak.

**The split** groups by `client_id`. No client appears on both sides of a fold, so the score is measured on clients the model has never seen. One client is 26.2% of the population, and fold 0 is that single client — read its per-fold number as one client's result, not an average.

## Results

Scored on `26,604` pages: the rows that are ranked (`avg_position > 0`) and where decline is arithmetically possible (`impressions_prev_30d > 0`). Excluded: `1,205` unranked rows and `2,191` ranked rows that cannot decline. Base rate `0.611`, spanning 31 of 32 clients.

Five client-grouped folds. No client appears on both sides of a split.

| | ROC AUC | P@10 | P@50 | P@100 | P@500 |
|---|---:|---:|---:|---:|---:|
| Hand rule | `0.532` | `0.760` | `0.808` | `0.782` | `0.746` |
| Logistic regression, strict | `0.586` | `0.720` | `0.724` | `0.706` | `0.704` |
| Logistic regression, permissive | `0.625` | `0.740` | `0.796` | `0.760` | `0.740` |
| Random forest, strict | `0.611` | `0.840` | `0.868` | `0.812` | `0.771` |
| **Random forest, permissive** | **`0.645`** | `0.920` | `0.844` | `0.842` | `0.787` |

Random forest permissive is what shipped: `0.645` against `0.532` for the hand rule. Real separation from chance, and modest.

Strict ranks slightly better at the top of the queue (`0.868` at 50) while scoring lower overall (`0.611`). AUC and precision-at-K disagree, so both are reported.

**What the split honesty cost.** The same model reads `0.767` under a random split and `0.645` under a client-grouped one. That `0.122` is not skill — it is the model recognising which client a row belongs to, which is worth nothing on a client it has never seen.

**These are not the reference pipeline's numbers.** `scripts/run_all.py` scores `26,612` rows under a single runtime with unnamed features and reports `0.565` / `0.600`. The counts differ by 8 because the filters overlap. The model did not get better between the two — the single-runtime measurement is the weaker one. Both are reconciled side by side at https://jericho-ram.github.io/cases.html. Every number above comes from `work/notebooks/capstone.ipynb`.

## Limitations

Seven named failure modes, each with the condition under which the score should not be trusted, are in [`outputs/known_failure_modes.md`](../outputs/known_failure_modes.md). The list is hand-maintained rather than generated, so it does not silently un-retract when a script re-runs.

The short version:

1. Zero-impression pages cannot decline, and they carried the original result.
2. The 90-day query window opens inside the label month.
3. GA4 features are 59% missing, and missing because a client had no GA4 — not because a page had no engagement.
4. The committed report did not reproduce from a clean clone.
5. The label is defined from impressions in the bundled slice and from clicks in the warehouse.
6. Precision@50 is the most fragile metric in the table; never quote it alone.
7. Under the reference pipeline's single-runtime measurement, no model discriminates well and the hand rule scores below random. That entry describes `scripts/run_all.py` on `26,612` rows, not the cross-validated table above.

## Where AI did the work

Notebook code was drafted with Claude and I checked every published number, figure caption, and column name against the data myself, which is how the leakage got caught: the filter order changes the population, and the model that wins on one loses on the other. Claude also got things wrong that I corrected: misread paths, and interpretations of the results that didn't match the source once I checked them.
