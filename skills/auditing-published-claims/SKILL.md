---
name: auditing-published-claims
description: Traces numbers already published on a page back to the code and outputs that produced them — source pinning, precision matching, provenance to file and line. Use when checking a live page, report, or portfolio claim against the repo behind it.
---

# Auditing published claims

A published number is only checkable if a reader can get from the page to the line that
produced it. This skill does that trace. It does not decide whether a claim is wrong — the
operator does that, and the split matters enough that it is stated before anything else.

## What this skill does not do

| Not this | Because |
|---|---|
| Notice discrepancies on its own | It has no trigger hook. It runs when asked, on the page it is handed. |
| Issue a verdict | It returns what the page says and what the source says, side by side. The judgement is the operator's. |
| Scan a whole site unprompted | Scope is one page, one claim at a time. |

The operator supplies the suspicion. The agent supplies the provenance. A run where the agent
finds nothing is not a clean bill of health — it is a run where nobody asked the right question.

## The loop

1. Locate the data source the page draws on.
2. Read `README.md`; load the skills the task needs from `skills/README.md`.
3. Operator asks whether a baseline figure exists for the claim under check.
4. Operator compares the published figure against it.
5. On a difference, operator asks where the number came from.
6. Agent traces it to a file and line — the generating script, not the committed output.
7. Repeat until the figure reconciles or the mismatch is named.

Step 6 is the one that has to be exact. `outputs/model_report.md` can carry a number its
generator no longer produces; only the script settles it.

## Pin the population before comparing any two figures

The agent's own worst failure was here. Handed one source filtered to months through April and
one not filtered at all, it compared across the two and reported the gap as a defect in the
page. The gap was the filter.

Before any two numbers are set beside each other, state for both:

- which dataset — the bundled slice or the full release
- which date window, and whether one was applied at all
- which row population after filtering, as a count

If those three do not match on both sides, there is no comparison to make yet.

## Match precision to source

A claim can be correct and still unverifiable. The cases page reported a grouped split buying
`0.008`; the two published figures, `0.908` and `0.901`, subtract to `0.007`. Re-run at four
decimals they were `0.9085` and `0.9008`, a gap of `0.0077`. Every figure was right. The page
had rounded away the precision a reader needed to reproduce the delta.

Rounded display plus a derived delta produces a number a careful reader will disagree with, and
being right is no defence if they cannot get there from what was published.

Where bars or rows are rounded for display, say so in the caption.

## What a real defect looks like

A chart caption claimed all `30,000` rows before the zero-impression filter, but the bars summed
to about `26,650` — which rounds onto the filtered `26,612`. The plotting call at
`scripts/04_evaluate_and_export.py:193` passes `final_frame`, not the raw CSV. Wrong by
construction, not by rounding. The trace to the line is what separated this from the case above.

## Two cases worth keeping as evals

| Case | Controls against |
|---|---|
| The `0.008` delta — correct, unverifiable | An auditor that flags everything |
| The `30,000` caption — wrong by construction | An auditor that flags nothing. It has to raise something that looks wrong, then let the evidence clear it |

## Output

List, table, or prose, matching whatever the deliverable needs. Always: the published figure,
the source figure, and the file and line the source figure came from.
