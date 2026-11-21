# Credit Risk Analytics: Time-Aware Loan Default Prediction

## Project Overview

This project investigates whether borrower characteristics, loan attributes, and historical macroeconomic conditions can help predict loan default.

It examines how default patterns differ across loan origination vintages and whether models trained on earlier periods generalize to later loan cohorts.

The project emphasizes chronological validation and evaluates whether macroeconomic indicators provide incremental predictive information beyond borrower and loan characteristics.

## Research Questions

1. Can borrower financial and credit characteristics predict observed loan default?
2. Do default patterns differ across origination years, loan terms, and borrower segments?
3. How well do models trained on earlier loan vintages generalize to later vintages?
4. Do historical macroeconomic indicators improve predictive performance beyond borrower and loan characteristics?

## Dataset

**Source:** LendingClub accepted-loan dataset, obtained through Kaggle.

Dataset: [All Lending Club Loan Data](https://www.kaggle.com/datasets/wordsforthewise/lending-club)

Raw file: `accepted_2007_to_2018Q4.csv`

Place the downloaded file at:

`data/raw/accepted_2007_to_2018Q4.csv`

The raw dataset is not included in this repository because of its size and source-data redistribution considerations.

### Target Definition

The binary target is constructed from resolved loan statuses:

- `default = 1`: Charged Off, Default, and Does not meet the credit policy. Status:Charged Off.
- `default = 0`: Fully Paid and Does not meet the credit policy. Status:Fully Paid.

Unresolved statuses, including Current, Late, In Grace Period and Issued, are excluded.

### Outcome Maturity Filtering

Loans are grouped by origination year and loan term. Cohorts with at least 99.5% resolved loans are retained.

This 99.5% threshold is an operational maturity-screening heuristic. It is not a verified dataset snapshot date or a fixed-horizon outcome definition.

The resulting eligible population contains approximately 676,302 records. A reproducible 100,000-record benchmark sample is also retained.

**Important limitation:** The target represents observed outcomes among resolved loans in selected cohorts. It does not represent a fully observed 36-month or 60-month default event for every loan. The results should therefore not be interpreted as definitive lifetime default probabilities.

## Project Workflow

### Notebook 1 — Data Quality and Vintage Analysis

`notebooks/01_data_quality_and_vintage_analysis.ipynb`

- Loads required columns from the raw LendingClub dataset.
- Examines loan statuses and missingness.
- Analyzes resolution rates by origination year and loan term.
- Applies the 99.5% cohort-resolution rule.
- Creates the full eligible population and a reproducible benchmark sample.

Main outputs:

- `data/processed/modeling_full.csv`
- `data/processed/modeling_sample.csv`
- Vintage and maturity diagnostic files under `outputs/`

### Notebook 2 — Feature Engineering

`notebooks/02_feature_engineering.ipynb`

- Cleans numerical and categorical loan fields.
- Converts interest rate, revolving utilization, loan term and employment length.
- Creates average FICO score.
- Creates income-to-loan and installment-to-income ratios.
- Examines missing values and extreme observations.
- Preserves metadata for temporal evaluation.

Main outputs:

- `data/processed/featured_data_full.csv`
- `data/processed/featured_data.csv`
- Feature-quality and term/year diagnostic files under `outputs/`

Imputation, scaling and categorical encoding are performed inside modeling pipelines to avoid preprocessing leakage.

### Notebook 3 — Modeling and Temporal Validation

`notebooks/03_modeling_and_temporal_validation.ipynb`

Three experiments are conducted:

1. **Pooled experiment:** combines 36-month and 60-month loans, with loan term included as a predictor.
2. **36-month experiment:** evaluates 36-month loans separately.
3. **60-month experiment:** evaluates 60-month loans separately.

The notebook compares:

- Logistic Regression
- Logistic Regression with origination-time features
- Random Forest
- Histogram Gradient Boosting

Expanding-window chronological validation is used. Each test vintage is later than its training and validation periods.

Model selection and threshold tuning use validation data. Final test data is reserved for evaluation.

Evaluation includes average precision, ROC-AUC, precision, recall, F1-score, Brier score, calibration, and performance by year and borrower/loan segment.

### Notebook 4 — Macroeconomic Features and Incremental Predictive Value

`notebooks/04_macroeconomic_features_and_incremental_value.ipynb`

This notebook investigates whether economic conditions around loan origination improve predictive performance.

Historical indicators are retrieved from the Federal Reserve Economic Data (FRED) service:

- US unemployment rate
- CPI year-over-year inflation
- Effective federal funds rate
- Three-month unemployment-rate change
- Three-month federal-funds-rate change

A conservative two-month lag is used when joining monthly indicators to loan origination months.

Two otherwise comparable conditions are evaluated:

- Baseline borrower and loan features
- Baseline features plus macroeconomic indicators

Both conditions use identical chronological test folds and validation-based model selection.

The notebook also includes probability calibration and validation-selected operating thresholds targeting 70% recall.

## Macroeconomic Experiment Results

The following results are from the latest executed Notebook 4. They are aggregate results across four chronological test vintages (2012–2015), representing 633,767 out-of-time predictions per condition.

| Metric | Baseline | Baseline + Macro |
|---|---:|---:|
| Average Precision | 0.2487 | 0.2519 |
| ROC-AUC | 0.6702 | 0.6744 |
| Brier Score | 0.1210 | 0.1205 |
| Default Precision at selected threshold | 22.58% | 22.14% |
| Default Recall at selected threshold | 62.99% | 66.67% |
| Predicted Default Rate at selected threshold | 41.42% | 44.72% |

The macro-enhanced condition produces modest improvements in average precision, ROC-AUC and Brier score.

These results suggest that macroeconomic indicators provide some incremental predictive information, but do not substantially transform overall model discrimination.

The 70% recall target is selected using validation data. Achieved test recall is lower, demonstrating that validation performance does not necessarily transfer perfectly to later vintages.

A probability threshold of 0.50 produces very few positive classifications in this experiment and should not be presented as a useful default-screening policy.

## Methodological Considerations and Limitations

1. **Resolved outcomes:** Unresolved loans are excluded. The outcome-maturity filter is a heuristic and does not create a fixed-horizon default target.
2. **Temporal generalization:** Chronological evaluation is more realistic than random splitting for later-vintage testing, but does not guarantee performance on present-day lending.
3. **Prediction timing:** Grade, sub-grade and interest rate are included as predictors. The intended prediction point is after these underwriting attributes have been assigned.
4. **Macroeconomic data revisions:** Current FRED historical values may differ from values originally published at the time. The two-month lag addresses publication timing conservatively but does not eliminate historical revision bias.
5. **Association, not causation:** Differences across years, grades, purposes and economic conditions are predictive associations, not evidence of causal effects.
6. **Decision thresholds:** Threshold selection depends on the intended business costs of false positives and false negatives. A recall-oriented threshold can produce many false-positive classifications.

## Installation and Reproducibility

Use Python 3.10 or newer.

From the project root, run:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
jupyter notebook
```

Place the raw LendingClub CSV at:

`data/raw/accepted_2007_to_2018Q4.csv`

Run the notebooks in this order:

1. `01_data_quality_and_vintage_analysis.ipynb`
2. `02_feature_engineering.ipynb`
3. `03_modeling_and_temporal_validation.ipynb`
4. `04_macroeconomic_features_and_incremental_value.ipynb`

Notebook 4 requires an internet connection to retrieve historical FRED data.

Generated processed datasets and evaluation outputs are stored under `data/processed/` and `outputs/`. These generated files are excluded from Git.

## Repository Structure

```text
credit_risk_analytics/
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
│   ├── 01_data_quality_and_vintage_analysis.ipynb
│   ├── 02_feature_engineering.ipynb
│   ├── 03_modeling_and_temporal_validation.ipynb
│   └── 04_macroeconomic_features_and_incremental_value.ipynb
├── outputs/
├── report/
├── README.md
├── requirements.txt
├── LICENSE
└── .gitignore
```

## License and Data Attribution

The project code is distributed under the MIT License.

The LendingClub dataset remains subject to its source provider's terms and conditions. The MIT License does not grant rights to redistribute the original dataset.

Historical macroeconomic indicators are sourced from the Federal Reserve Economic Data (FRED) service.