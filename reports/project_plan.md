# Project Plan — Dynamic Surge Pricing Prediction

## Problem Statement

We predict whether ride-hailing demand in a given city zone will surge beyond its historical normal level within the next 15–30 minutes. The operations team at an e-hailing company uses these predictions to proactively reposition nearby idle drivers toward high-demand zones and optionally notify riders of expected price increases.

False negatives (missing a real surge) are costlier than false positives (false alarms), because a missed surge causes long rider wait times, customer churn, and lost revenue — while a false positive only causes minor driver repositioning overhead.

## Dataset

- **Primary:** NYC TLC Yellow Taxi Trip Records (January–February 2023)
  - ~6 million individual trip records
  - Aggregated into zone-level demand counts per 15-minute time window
  - Source: https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- **External — Weather:** Historical hourly weather for NYC (temperature, precipitation, snow) via Open-Meteo API
- **External — Calendar:** US public holidays, weekend flags

## Target Variable

We define "surge" as demand in a given zone exceeding its historical baseline for that zone and time-of-day by a threshold (e.g., 1.5×). This target is constructed from raw data, not provided pre-labelled.

## Pipeline

```
Data Ingestion → Cleaning → Aggregation → Target Creation
    → Feature Engineering → Temporal Train/Test Split
    → Modelling → Evaluation → Explainability → Reporting
```

### Stage details

| Stage | Input | Output |
|---|---|---|
| 1. Data Ingestion | NYC TLC parquet files + weather API | Raw DataFrames |
| 2. Data Cleaning | Raw trips | Clean trips (nulls, outliers removed) |
| 3. Aggregation | Clean trips | Demand counts per zone per 15-min slot |
| 4. Target Creation | Demand counts | Binary surge labels |
| 5. Feature Engineering | Demand + weather + calendar | Feature matrix |
| 6. Train/Test Split | Feature matrix | Temporal split (Jan train, Feb test) |
| 7. Modelling | Train set | Trained models (LR → RF → LightGBM) |
| 8. Evaluation | Models + test set | Metrics and comparison |
| 9. Explainability | Best model + test data | SHAP analysis |
| 10. Reporting | All results | Findings report + README |

## Feature categories

- **Lag features:** Demand 15 min, 30 min, 1 hour, 2 hours ago
- **Rolling statistics:** Mean and std of demand over past 1h, 3h windows
- **Time features:** Hour of day (cyclical encoded), day of week, is_weekend, is_holiday
- **Weather:** Temperature, precipitation, snow indicator
- **Zone-level:** Historical average demand for this zone at this hour

## Models (progressive complexity)

1. **Logistic Regression** — interpretable baseline
2. **Random Forest** — ensemble baseline
3. **LightGBM** — gradient boosting (expected best performer)

## Evaluation

| Metric | Target |
|---|---|
| PR-AUC (primary) | ≥ 0.50 |
| Recall | ≥ 0.70 |
| Precision | ≥ 0.40 |
| Lift over baseline | Meaningful improvement |

## Key design decisions

- **Temporal validation only** — no random train/test splits (prevents data leakage)
- **Features use only past data** — lag and rolling features strictly backward-looking
- **LightGBM over XGBoost** — faster iteration for time-constrained project
- **SHAP for explainability** — game-theory grounded, shows direction and magnitude of feature effects

## Tools

`pandas`, `numpy`, `pyarrow`, `matplotlib`, `seaborn`, `scikit-learn`, `lightgbm`, `shap`, `jupyter`

## Timeline

- **Day 1:** Data acquisition, EDA, data cleaning
- **Day 2:** Feature engineering, modelling, evaluation
- **Day 3:** SHAP explainability, findings report, README, final polish
