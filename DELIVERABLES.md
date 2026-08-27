# FlyRank Internship — Deliverables Index

Edo Sanjaya Perkasa · ML and General AI Fluency tracks

A scoring model for refresh and content opportunity, built for reviewers who can't read every row. Every deliverable from both tracks below, with what each one is evidence of.

---

## Capstone

| Deliverable | What it proves |
|---|---|
| [Which Pages Should You Refresh First?](https://jericho-ram.github.io/FlyRank-Internship-ML/) | The deployed capstone paper. A client-grouped, leakage-audited model for ranking declining content. Source: `docs/index.html`. |
| [Capstone report](work/capstone_report.md) | End-to-end scoring model: leak-free split, baseline, held-out metrics, named failure modes with fixes. |
| [Capstone page](https://jericho-ram.github.io/capstone.html) | Portfolio-side write-up of the same work. |

## Model and evaluation

| Deliverable | What it proves |
|---|---|
| [Model report](outputs/model_report.md) | Recorded metrics and the retraction of a figure produced under leakage. |
| [Evaluation and export script](scripts/04_evaluate_and_export.py) | The retraction is embedded in the generating script, not only in the report it writes. |
| [Notebooks](notebooks/) | Step-by-step run from raw slice to scored output. Reads `data/raw/content_refresh_anonymized.csv`, committed here. |
| [Scripts](scripts/) | The same pipeline as command-line steps, with the input path passed as an argument. Same committed slice. |

**Finding worth reading before the numbers:** model selection flips with population. Unfiltered data favours random forest (`0.750`); excluding zero-impression rows favours logistic regression (`0.583`). The score is only trustworthy on the population it was selected against.

## Practice and process

| Deliverable | What it proves |
|---|---|
| [Claim auditor skill](skills/auditing-published-claims/SKILL.md) | A retrieval tool that quotes source lines for a human to judge. It does not issue verdicts. |
| [Skills router](skills/README.md) | Index of documented skills. |
| [Data dictionary](docs/data-dictionary.md) | Terms, column definitions, and how the model works — what a reader needs to check a figure against the source. |
| [Fix log, Week 06](reports/FL-06_Fix_Log_ESP.pdf) | Seven findings across eight pages at 360, 390, 430 and 820px. Six fixed, one left open with a stated reason. Three were not mobile defects; opening the site on a phone is what prompted the audit that found them. |

## Documentation and demo (FL-09)

| Deliverable | What it proves |
|---|---|
| [README](README.md) | Setup a stranger can follow, usage examples, architecture sketch, v2 eval results, limitations. |
| TODO — demo video | Live end-to-end run with narration, including one limitation stated out loud. |

## Final package (FL-10)

| Deliverable | What it proves |
|---|---|
| TODO — retrospective | What changed in how I work across the track. |
| TODO — build-in-public post | One real decision and one real limitation. |
| Hours log | Completed in the internship portal. |

## Site

| Deliverable | What it proves |
|---|---|
| [Portfolio](https://jericho-ram.github.io) | Positioning, working links, published work. |
| [CV](https://jericho-ram.github.io/cv.html) | Working history and links. |
| [Contact](https://jericho-ram.github.io/contact.html) | Working form, live backend. |

---

## Where AI did the work

Notebook code was drafted with Claude and I checked every published number, figure caption, and column name against the data myself, which is how the leakage got caught: the filter order changes the population, and the model that wins on one loses on the other. Claude also got things wrong that I corrected: misread paths, and interpretations of the results that didn't match the source once I checked them.
