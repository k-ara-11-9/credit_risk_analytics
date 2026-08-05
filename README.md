# Credit Risk Analytics

Predicting loan default using machine learning, built on LendingClub's accepted-loans dataset (2007–2018 Q4). The workflow covers data exploration, feature engineering, and model preparation.

## Project Structure

```
credit_risk_analytics/
├── data/
│   ├── raw/            # accepted_2007_to_2018Q4.csv (not tracked — see Data)
│   └── processed/      # cleaned_data.csv, X.csv, y.csv (not tracked — see Data)
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_feature_engineering.ipynb
│   └── 03_modeling.ipynb
├── requirements.txt
├── LICENSE
└── README.md
```

## Data

Source: [LendingClub dataset on Kaggle](https://www.kaggle.com/datasets/wordsforthewise/lending-club) — only the **accepted** loans file (`accepted_2007_to_2018Q4.csv`) is used; rejected applications are not part of this project.

The raw file (~2.26M rows × 151 columns) is sampled down to 100,000 rows (`random_state=42`) for tractability. `data/` is git-ignored, so the raw CSV and all processed outputs need to be downloaded/regenerated locally rather than pulled from the repo.

## Workflow

### 1. Exploratory Data Analysis
- Sample 100,000 rows from the raw dataset
- Drop identifier/free-text columns (`id`, `member_id`, `url`, `title`, `zip_code`, `emp_title`, `desc`)
- Drop post-origination leakage columns (`total_pymnt`, `recoveries`, `last_pymnt_amnt`, etc.)
- Create target variable `default` from `loan_status` (Charged Off / Default / Late 16–120 days → 1, else 0)
- Drop hardship/settlement columns and any column with >70% missing values
- Narrow down to a curated set of ~24 features + target
- Export `cleaned_data.csv`

### 2. Feature Engineering
- Engineer `fico_avg`, `income_to_loan`, and `installment_to_income`
- Clean `revol_util` (strip `%`), `term` (strip "months"), `emp_length` (extract years)
- Drop `fico_range_low` / `fico_range_high` after averaging
- Fill remaining missing numeric values with the median
- Separate features and target:
  - `X` → one-hot encoded feature set
  - `y` → target (`default`)
- Export `X.csv` and `y.csv`

### 3. Modeling (in progress)
- `X.csv` / `y.csv` loaded (100,000 rows × 80 features)
- Model training and evaluation to be implemented

## Notes
- The original `loan_status` column is transformed into `default`
- `X.csv` contains only features (target removed); `y.csv` holds the target separately

## Setup

```
pip install -r requirements.txt
```

## License
MIT — see [LICENSE](LICENSE).

## Roadmap
- [x] EDA & cleaning
- [x] Feature engineering
- [ ] Baseline model
- [ ] Model evaluation (SHAP, metrics)
- [ ] Hyperparameter tuning
