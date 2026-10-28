# Credit Risk Analytics

A machine learning project that predicts loan default risk using LendingClub's accepted-loan dataset (2007–2018 Q4). The project explores borrower financial and credit characteristics, engineers useful features, and compares classification models to understand how well they identify potential defaults.

## Project Objective

**Research question:** Can a machine learning model predict the likelihood of loan default using borrower financial and credit characteristics?

The project uses historical loan data to build and evaluate predictive models. Its focus is on identifying potential default risk, not automatically approving or rejecting loan applications.

**Analytics type:** Predictive Analytics

## Dataset

**Source:** [LendingClub dataset on Kaggle](https://www.kaggle.com/datasets/wordsforthewise/lending-club)

This project uses the `accepted_2007_to_2018Q4.csv` file. Rejected applications are not included.

- The raw dataset contains approximately 2.26 million rows and 151 columns.
- A reproducible random sample of 100,000 rows is used for analysis.
- The target variable, `default`, is derived from resolved loan statuses:
  - `1` — Charged Off or Default
  - `0` — Fully Paid
- Unresolved loan statuses are excluded from the modeling dataset.

The dataset contains historical loans that LendingClub accepted. Therefore, the model's findings should not be assumed to generalize to rejected applicants or every lending population.

**Data availability:** Dataset files are not stored in this repository. Download the source dataset and place the accepted-loan CSV in `data/raw/` before running the notebooks.

## Project Structure

```text
credit_risk_analytics/
├── data/
│   ├── raw/                  # Source dataset (not tracked)
│   └── processed/            # Generated datasets (not tracked)
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_feature_engineering.ipynb
│   └── 03_modeling.ipynb
├── requirements.txt
├── LICENSE
└── README.md
```

## Workflow

### 1. Exploratory Data Analysis

Notebook: `01_eda.ipynb`

- Load and sample the source dataset.
- Remove identifiers, free-text fields, and columns that could leak information unavailable at prediction time.
- Create the binary `default` target from resolved loan statuses.
- Remove hardship and settlement-related columns.
- Remove columns with more than 70% missing values.
- Explore default rates across borrower and loan characteristics.
- Export the cleaned dataset to `data/processed/cleaned_data.csv`.

### 2. Feature Engineering

Notebook: `02_feature_engineering.ipynb`

- Create `fico_avg` from the two FICO score range columns.
- Create `income_to_loan` and `installment_to_income` ratios.
- Clean fields such as interest rate, revolving utilization, loan term, and employment length.
- Separate the features (`X`) from the target (`y`).
- Save the engineered dataset to `data/processed/featured_data.csv`.

Missing-value imputation, categorical encoding, and numerical scaling are handled within the modeling pipeline so that preprocessing is fitted on training data rather than the entire dataset.

### 3. Model Training and Evaluation

Notebook: `03_modeling.ipynb`

Two classification models are compared:

- **Logistic Regression** — a baseline classification model.
- **Random Forest** — an ensemble model that combines multiple decision trees.

The dataset is divided into training and test sets using stratified sampling. Models are evaluated using accuracy, precision, recall, F1-score, ROC-AUC, classification reports, and confusion matrices.

### 4. Threshold Tuning

The Logistic Regression model is also evaluated using a lower classification threshold selected on a validation set to target at least 80% recall.

Lowering the threshold can identify more potential defaults, but it can also increase false positives. The tuned-threshold results are reported separately from the original model comparison.

## Results

The following results were obtained from the current experiment on the held-out test set.

| Metric | Logistic Regression (baseline threshold) | Random Forest (baseline threshold) | Logistic Regression (tuned threshold) |
|---|---:|---:|---:|
| Accuracy | 63.10% | 79.93% | 55.15% |
| Precision | 31.06% | 51.49% | 28.30% |
| Recall | 68.33% | 5.14% | 80.08% |
| F1-score | 42.71% | 9.35% | 41.82% |
| ROC-AUC | 0.7071 | 0.6920 | 0.7071 |

### Interpretation

- Random Forest has the highest accuracy among the original model comparisons, but its recall for defaults is very low.
- Logistic Regression identifies a substantially larger proportion of actual defaults.
- Threshold tuning increases Logistic Regression's recall to approximately 80%, at the cost of lower accuracy and precision.
- The tuned threshold does not improve ROC-AUC because it changes the classification cutoff rather than the model's probability ranking.

These results illustrate why accuracy alone is insufficient when the target classes are imbalanced. The preferred model and threshold depend on the relative costs of missed defaults and false alarms.

## Setup and Usage

### Requirements

- Python 3
- Jupyter Notebook or a compatible VS Code notebook environment
- Packages listed in `requirements.txt`

Install the dependencies:

```bash
pip install -r requirements.txt
```

### Run the notebooks

1. Download the accepted-loan dataset from Kaggle.
2. Place `accepted_2007_to_2018Q4.csv` in `data/raw/`.
3. Run `01_eda.ipynb` from top to bottom.
4. Run `02_feature_engineering.ipynb` from top to bottom.
5. Run `03_modeling.ipynb` from top to bottom.

The notebooks generate intermediate CSV files in `data/processed/`. These files are excluded from version control and must be regenerated locally.

## Limitations

- The analysis uses a 100,000-row sample rather than the entire source dataset.
- Historical LendingClub loans may not represent future lending populations.
- The target is based on resolved loan outcomes; unresolved statuses are excluded.
- Threshold selection involves a trade-off between identifying defaults and incorrectly flagging non-defaulting loans.
- The model is a predictive risk-assessment experiment, not a validated production lending system.
- The results do not establish that any borrower characteristic causes default.

## Future Improvements

- Tune model hyperparameters and compare additional classifiers.
- Evaluate precision-recall curves and probability calibration.
- Investigate feature importance and model interpretability.
- Assess performance across different borrower groups and data samples.
- Explore how predicted risk could support lending decisions, subject to appropriate validation and business rules.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

The dataset remains subject to its original source terms.
