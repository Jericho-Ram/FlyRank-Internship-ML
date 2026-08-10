# FlyRank Refresh Opportunity Model Report

> **Run provenance.** Regenerated 2026-08-09 from a clean clone.
> > Scored population excludes the `3,388` of `30,000` rows with
> `impressions_prev_30d == 0`, leaving `26,612` rows scored, since
> pages with no prior impressions cannot decline.
> Earlier versions published a Precision@50 of 0.680 over all 30,000 rows;
> that figure is withdrawn. See `outputs/known_failure_modes.md`, entries 1
> and 4. Models are selected by ROC AUC, not Precision@50 (entry 6).
> Quote ROC AUC alongside Precision@50, never Precision@50 alone.


This report is generated from the bundled anonymized starter dataset (`data/raw/content_refresh_anonymized.csv`).
The model ranks existing content for refresh review. It does not use titles, URLs, client names, domains, or keywords.

## Data

- Rows scored: 26,612
- Declining-label rows: 16,262
- Declining-label rate: 0.611
- Split strategy used for validation: client_holdout
- Target: `is_declining_label`

### Which dataset these numbers describe

Every metric in this report scores the bundled starter CSV, using its supplied
label `is_declining_label`. The leakage audit in
`work/notebooks/w03_feature_leakage_check.ipynb` uses a different source: the
FlyRank warehouse, 176,737 rows, with the label computed directly as
`clicks_apr < clicks_mar`. Base rate there is 0.255, or 0.655 once the 61.1%
of rows with `clicks_mar == 0` are removed — those rows cannot decline, because
April clicks cannot fall below zero.

Different dataset, different label, different population. Metrics from the two
are not comparable, and the numbers below should not be read as the accuracy I
would quote for a live system.

## Model Comparison

Best model: `logistic_regression` selected by `roc_auc`.

| Model | ROC AUC | Avg precision | Precision@50 | Recall | F1 |
|---|---:|---:|---:|---:|---:|
| decision_tree | 0.505 | 0.623 | 0.500 | 0.924 | 0.748 |
| logistic_regression | 0.583 | 0.689 | 0.740 | 0.854 | 0.734 |
| random_forest | 0.566 | 0.662 | 0.600 | 0.992 | 0.768 |
| baseline_rules | 0.380 | 0.535 | 0.460 | - | - |

## Final Queue

- High-confidence items: 2,653
- Medium-confidence items: 10,653
- Low-confidence items: 13,306
- `monitor` items: 9,403
- `refresh` items: 8,692
- `refresh_and_review_ctr` items: 6,538
- `refresh_and_review_engagement` items: 1,899
- `expand_and_refresh` items: 80

## Top Features

- `log_impressions_90d`: 0.8476
- `log_clicks_90d`: 0.5881
- `days_with_impressions`: 0.4298
- `word_count`: 0.4116
- `avg_position`: 0.3927
- `days_since_last_update`: 0.2402
- `word_count_tier_1000-2000`: 0.2140
- `char_count`: 0.1495
- `days_with_sessions`: 0.1440
- `word_count_tier_2000-3500`: 0.1418

## Top 10 Queue Preview

| Rank | Score | Model probability | Action | Reasons | Impressions | Sessions | Trend |
|---:|---:|---:|---|---|---:|---:|---|
| 1 | 89.7 | 0.871 | refresh_and_review_ctr | declining_with_demand, page_one_decay_risk, low_ctr_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate | 208678 | 6 | down |
| 2 | 89.0 | 0.861 | refresh_and_review_ctr | low_ctr_visible_page, low_engagement_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate, engagement_review_candidate | 128068 | 87 | up |
| 3 | 86.9 | 0.929 | refresh_and_review_ctr | declining_with_demand, low_ctr_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate | 22456 | 4 | down |
| 4 | 86.8 | 0.854 | refresh_and_review_ctr | low_ctr_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate | 14830 | 4 | up |
| 5 | 86.1 | 0.887 | refresh_and_review_ctr | page_one_decay_risk, low_ctr_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate | 5123 | 22 | up |
| 6 | 85.8 | 0.884 | refresh_and_review_ctr | declining_with_demand, page_one_decay_risk, low_ctr_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate | 6822 | 3 | down |
| 7 | 85.7 | 0.829 | refresh_and_review_ctr | low_ctr_visible_page, low_engagement_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate, engagement_review_candidate | 33286 | 194 | up |
| 8 | 85.4 | 0.881 | refresh_and_review_ctr | low_ctr_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate | 4792 | 17 | stable |
| 9 | 85.0 | 0.916 | refresh_and_review_ctr | declining_with_demand, low_ctr_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate | 1986 | 2 | down |
| 10 | 84.8 | 0.848 | refresh_and_review_ctr | declining_with_demand, low_ctr_visible_page, model_decline_risk, visible_model_opportunity, ctr_review_candidate | 6635 | 10 | down |

## Generated Files

- `outputs/refresh_queue.csv`
- `outputs/model_results.json`
- `outputs/summary.json`
- `outputs/charts/action_mix.svg`
- `outputs/charts/confidence_mix.svg`
- `outputs/charts/top_reason_codes.svg`
- `outputs/charts/top_feature_importance.svg`
- `outputs/charts/trend_distribution.svg`

## Practical Use

Use the ranked queue as a reviewer aid, not as an automatic publishing decision.
The safest first production use is to inspect high-confidence rows, verify the page manually, and compare the recommendation against editorial context.

Known failure modes, with fixes and the conditions under which this score should not be trusted, are maintained by hand in `outputs/known_failure_modes.md`.
