# Credit Risk Analytics: Time-Aware Loan Default Prediction

## Abstract

This project investigates the prediction of observed loan default using borrower characteristics, loan attributes, origination vintage information, and historical macroeconomic indicators. LendingClub accepted-loan records covering 2007–2018 are prepared using an outcome-resolution screening rule. The modeling workflow emphasizes chronological evaluation so that models are assessed on later loan vintages rather than relying only on random train-test splits. Separate pooled, 36-month, and 60-month experiments are conducted, followed by an incremental-value experiment comparing borrower/loan features with and without macroeconomic indicators.

The latest macroeconomic experiment shows modest improvements in aggregate ranking and probability-error metrics after adding lagged unemployment, inflation, and federal funds rate information. The improvements are small and are not uniform across all test vintages. The results therefore support the conclusion that macroeconomic context adds some predictive information, but does not transform the model's overall discriminative performance.

## 1. Introduction

Loan default prediction is a credit-risk problem in which the available information includes borrower financial characteristics, credit attributes, and contractual loan properties. These relationships can change across origination periods as borrower composition, underwriting practices, and broader economic conditions change.

A random split can mix observations from different periods between training and testing. Such a split is useful as a benchmark but does not directly represent the task of training on historical loans and predicting outcomes for later-issued loans. This project therefore places chronological validation at the centre of the modeling design.

### 1.1 Research questions

1. Can borrower financial and credit characteristics predict observed loan default?
2. Do observed default patterns differ across origination years, loan terms, and borrower segments?
3. How well do models trained on earlier loan vintages generalize to later vintages?
4. Do historical macroeconomic indicators add predictive information beyond borrower and loan characteristics?

## 2. Dataset and target construction

### 2.1 Data source

The project uses LendingClub's accepted-loan dataset, obtained through Kaggle:

[All Lending Club Loan Data](https://www.kaggle.com/datasets/wordsforthewise/lending-club)

The source file used is `accepted_2007_to_2018Q4.csv`. The raw dataset is not included in the code repository. Users must obtain it from the source and place it at `data/raw/accepted_2007_to_2018Q4.csv`.

### 2.2 Binary target

The binary target is constructed from resolved loan statuses:

- `default = 1`: `Charged Off`, `Default`, and `Does not meet the credit policy. Status:Charged Off`.
- `default = 0`: `Fully Paid` and `Does not meet the credit policy. Status:Fully Paid`.

Unresolved statuses, including `Current`, `Late`, `In Grace Period`, and `Issued`, are excluded from this binary target.

### 2.3 Outcome-resolution screening

Loans are grouped by origination year and contractual term. Cohorts with at least 99.5% resolved loans are retained. Based on the executed data-preparation analysis, eligible cohorts include 36-month loans through 2015 and 60-month loans through 2013. The resulting eligible population contains approximately 676,302 records. A reproducible 100,000-record sample is retained as a benchmark.

The 99.5% rule is an operational screening heuristic. It is not a confirmed dataset-wide observation cutoff and does not create an exact fixed-horizon default label. The target consequently represents outcomes observed among resolved loans in selected cohorts, not the eventual lifetime default probability of every loan originated in those years.

## 3. Data preparation and feature engineering

The first notebook examines loan status counts, missingness, origination dates, last-payment and credit-pull date ranges, and resolution rates by year and term. It produces the full eligible population and a benchmark sample.

The second notebook cleans numerical and categorical fields and creates the following engineered features:

- Average FICO score from the lower and upper FICO range values.
- Income-to-loan ratio.
- Installment-to-income ratio.

Interest rate, revolving utilization, loan term, and employment length are normalized into usable forms. Non-positive income is treated as missing when constructing income ratios. Extreme values are inspected rather than automatically removed. Missing-value imputation, scaling, and categorical encoding are performed within the modeling pipelines to reduce preprocessing leakage.

The full engineered population is approximately 676,302 rows. The 100,000-row benchmark is retained separately. Source-row linkage metadata is not used as a predictor.

## 4. Modeling design

### 4.1 Experiments

The modeling notebook conducts three experiments:

1. **Pooled experiment:** combines 36-month and 60-month loans and represents loan term as a predictor.
2. **36-month experiment:** trains and evaluates using 36-month loans only.
3. **60-month experiment:** trains and evaluates using 60-month loans only.

The candidate algorithms are Logistic Regression, Logistic Regression with origination-time features, Random Forest, and Histogram Gradient Boosting.

### 4.2 Chronological validation

Expanding-window chronological folds are used. Training data comes from earlier origination periods, validation data from a subsequent period, and testing data from a later period. Model selection and threshold selection are based on validation data; test labels are reserved for final evaluation.

The full-population execution completed four pooled test folds, three 36-month test folds, and two 60-month test folds. The 60-month experiment has fewer historical observations than the 36-month experiment, so its results should be interpreted with additional caution.

### 4.3 Evaluation metrics

The project reports average precision, ROC-AUC, precision, recall, F1-score, Brier score, calibration diagnostics, and subgroup performance. Average precision is particularly relevant because defaults are a minority class. Accuracy alone is not an adequate measure of default detection.

Thresholds are decision-policy choices rather than intrinsic model properties. The macroeconomic notebook uses validation-selected operating thresholds targeting 70% recall. Achieved recall on the later test data is lower than the validation target, illustrating that validation performance does not transfer perfectly to unseen vintages.

## 5. Macroeconomic feature experiment

### 5.1 Indicators

The fourth notebook retrieves monthly historical series from the Federal Reserve Economic Data (FRED) service and constructs five indicators:

- US unemployment rate.
- Three-month change in unemployment rate.
- CPI year-over-year inflation.
- Effective federal funds rate.
- Three-month change in the federal funds rate.

A conservative two-month lag is applied when joining the monthly indicators to loan origination months because the LendingClub data records origination month rather than an exact issue day. This is intended to reduce use of information that would not yet have been available at the prediction time.

Historical FRED values can be revised. The experiment uses currently retrieved historical values rather than point-in-time ALFRED vintages, so it does not completely eliminate revision bias.

### 5.2 Comparison design

Two conditions are compared on identical chronological folds:

- **Baseline:** borrower and loan predictors, including the established origination-time model design.
- **Baseline + macro:** the same predictors plus the five macroeconomic indicators.

The refined experiment excludes `mort_acc` from both conditions because it has no observed training values in some early chronological folds. Preprocessing and model selection are performed within the training/validation workflow.

### 5.3 Aggregate results

The following figures are from the latest executed Notebook 4 and represent four chronological test vintages (2012–2015), with 633,767 out-of-time predictions per condition.

| Metric | Baseline | Baseline + macro |
|---|---:|---:|
| Average precision | 0.2487 | 0.2519 |
| ROC-AUC | 0.6702 | 0.6744 |
| Brier score | 0.1210 | 0.1205 |
| Default precision at validation-selected threshold | 22.58% | 22.14% |
| Default recall at validation-selected threshold | 62.99% | 66.67% |
| Predicted default rate at validation-selected threshold | 41.42% | 44.72% |

Average precision increases by 0.0032 and ROC-AUC increases by 0.0042. The Brier score decreases by 0.0005. These are modest aggregate improvements. The macro-enhanced condition's default recall is higher, while its default precision is slightly lower, and it classifies a larger share of loans as potential defaults.

The changes are not uniform across test vintages. The per-vintage average-precision changes are approximately -0.00052 (2012), +0.00090 (2013), +0.00020 (2014), and +0.00134 (2015). Thus, the macro features do not improve every chronological fold.

No statistical significance test is reported for these small metric differences. They should be described as observed improvements in this experiment, not as proof of a statistically significant or universally generalizable benefit.

## 6. Discussion

The chronological experiments indicate that borrower and loan characteristics contain predictive signal for distinguishing observed default outcomes in the selected resolved cohorts. Performance is moderate rather than high, and results vary across loan terms and origination periods.

The macroeconomic comparison suggests that broad economic conditions around origination add some incremental predictive information. The improvement in average precision and ROC-AUC is small, and the fold-level comparison shows that the effect is not consistent across every test year. Macroeconomic variables should therefore be treated as supplementary context rather than a substitute for borrower and loan information.

The threshold results also illustrate a practical credit-risk trade-off. A recall-oriented policy identifies more actual defaults but flags many non-defaulting loans. The appropriate threshold depends on the costs of false positives and false negatives in the intended lending workflow. The project does not establish a universal business-optimal threshold.

## 7. Limitations

1. **Resolved-outcome selection:** unresolved loans are excluded. The 99.5% cohort-resolution heuristic does not guarantee an exact fixed-horizon outcome for every loan.
2. **Historical scope:** the data covers origination vintages through 2018. Results do not establish performance on present-day lending populations.
3. **Prediction timing:** grade, sub-grade, and interest rate are predictors. The intended prediction point is after these underwriting attributes have been assigned, not before underwriting.
4. **Macroeconomic revisions:** the current FRED historical series may incorporate revisions made after the original publication dates. A stricter point-in-time experiment would use ALFRED real-time vintages.
5. **Causal interpretation:** segment and macroeconomic comparisons are predictive associations. They do not establish that a particular economic factor caused an individual loan to default.
6. **Decision costs:** a threshold targeting recall can yield many false positives. No lender-specific cost matrix or operational decision rule is established in this project.
7. **External validity:** chronological testing on historical LendingClub cohorts is a temporal stress test, not evidence that the model will perform similarly on other lenders, countries, or current market conditions.

## 8. Conclusion

This project builds a time-aware credit-risk prediction workflow using LendingClub accepted-loan data. It applies outcome-resolution screening, engineers borrower and loan features, evaluates pooled and term-specific models using chronological validation, and tests the incremental predictive value of historical macroeconomic indicators.

The latest macroeconomic experiment shows small aggregate improvements in average precision, ROC-AUC, and Brier score after adding lagged unemployment, inflation, and federal funds rate indicators. The improvement is not consistent across all test vintages, and overall model discrimination remains moderate. The evidence therefore supports a cautious conclusion: macroeconomic context contributes some additional predictive information in this historical dataset, but does not fundamentally change the model's predictive performance.

## 9. Reproducibility

Use Python 3.10 or newer. From the project root, create and activate a virtual environment, install `requirements.txt`, and run the notebooks in order:

1. `01_data_quality_and_vintage_analysis.ipynb`
2. `02_feature_engineering.ipynb`
3. `03_modeling_and_temporal_validation.ipynb`
4. `04_macroeconomic_features_and_incremental_value.ipynb`

Notebook 4 requires internet access to retrieve FRED data. The raw LendingClub file and generated loan-level processed CSVs are not included in the Git repository.

## References and Data Attribution

- LendingClub accepted-loan data, distributed through Kaggle: [All Lending Club Loan Data](https://www.kaggle.com/datasets/wordsforthewise/lending-club).
- Federal Reserve Bank of St. Louis, Federal Reserve Economic Data (FRED): [https://fred.stlouisfed.org/](https://fred.stlouisfed.org/).
- Project source code is distributed under the MIT License. The MIT License does not grant rights to redistribute the LendingClub source dataset; users must comply with the dataset provider's terms.
