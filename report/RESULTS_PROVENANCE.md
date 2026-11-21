# Results Provenance

## Latest macroeconomic experiment

The README and project report use the aggregate results from the latest executed `04_macroeconomic_features_and_incremental_value.ipynb` uploaded on 22 November 2026.

| Metric | Baseline | Baseline + macro |
|---|---:|---:|
| Average precision | 0.2487 | 0.2519 |
| ROC-AUC | 0.6702 | 0.6744 |
| Brier score | 0.1210 | 0.1205 |
| Default precision at validation-selected threshold | 22.58% | 22.14% |
| Default recall at validation-selected threshold | 62.99% | 66.67% |
| Predicted default rate at validation-selected threshold | 41.42% | 44.72% |

The comparison uses four chronological test vintages (2012–2015), representing 633,767 out-of-time predictions per condition. Thresholds are selected using validation data; test labels are not used for threshold selection.

## Interpretation guardrails

- The macroeconomic improvements are modest and are not uniform across all test vintages.
- No statistical significance test is reported. Do not describe these differences as statistically significant.
- The 70% recall target is a validation objective; achieved test recall is lower.
- The target is based on resolved outcomes in cohorts passing a 99.5% resolution-share heuristic. It is not a fixed-horizon default outcome for every issued loan.
- Current FRED historical values may contain revisions. The two-month lag addresses release timing conservatively but does not fully eliminate revision bias.

## Rerun requirement

If Notebook 4 is edited or rerun and produces different metrics, regenerate the README table and project report from the new executed outputs. Do not copy values from earlier notebook runs or previous report drafts.
