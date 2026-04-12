# Dynamic Surge Pricing Prediction

> Predicting ride-hailing demand surges before they happen — enabling proactive driver positioning and improved rider experience.

---

## Why this project?

Surge pricing is one of the most visible and impactful features of any ride-hailing platform. When demand suddenly spikes in a zone — after a concert ends, during rush hour in the rain, on New Year's Eve — the platform faces a critical choice: react after the fact (riders wait, drivers miss out) or predict it early and act proactively.

Most portfolio projects in this space predict *trip duration* or *fare price* — problems with clean, pre-labelled datasets. This project is different. **There is no "surge" column in the raw data.** We construct the entire ML problem from scratch: defining what surge means, building the target variable from raw trip records, and engineering features that a real operations team could use to reposition drivers before demand peaks.

This reflects how ML problems are actually framed in industry — not downloaded from Kaggle, but reasoned through from a business need.

---

## Business Problem

**Who:** The operations team at an e-hailing company.

**What they need:** A signal — 15 to 30 minutes ahead — that a specific city zone is about to experience abnormally high demand.

**What they do with it:** Reposition idle drivers toward that zone, or notify riders that prices may rise soon.

**Why it matters:**
- A missed surge (false negative) → riders wait too long → churn → lost revenue
- A false alarm (false positive) → drivers reposition unnecessarily → minor fuel/time cost

False negatives are more costly. Our model optimises accordingly.

---

## What We're Predicting

**Target:** Binary classification — will demand in zone Z, in the next 15-minute window, exceed its historical average for that zone and time-of-day by a defined threshold?

- `1` → Surge (act now: reposition drivers)
- `0` → Normal (no action needed)

---

## Dataset

- **NYC TLC Yellow Taxi Trip Records** (Jan–Feb 2023) — ~6 million individual trips
  - Source: [NYC TLC Open Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
  - Aggregated into zone-level demand counts per 15-minute time window
- **Weather data** — hourly temperature, precipitation, snowfall for NYC (Open-Meteo API)
- **Calendar features** — US public holidays, weekday/weekend flags

> Raw data files are not committed to this repository. See `data/README.md` for download instructions.

---

## Approach

```
Raw Trips → Clean → Aggregate by Zone & Time → Define Surge Target
    → Engineer Features → Temporal Split → Model → Evaluate → Explain
```

### Feature engineering highlights
- **Lag features** — demand 15 min, 30 min, 1 hr ago in the same zone
- **Rolling statistics** — mean and std of demand over past 1h and 3h windows
- **Time features** — hour of day (cyclically encoded), day of week, weekend, holiday flags
- **Weather** — rain, snow, temperature at pickup time
- **Zone baseline** — historical average demand for this zone at this hour

### Modelling
Progressive complexity:
1. Logistic Regression — interpretable baseline
2. Random Forest — ensemble baseline
3. LightGBM — gradient boosting (primary model)

### Explainability
SHAP analysis to answer: *which conditions most strongly predict a surge?*

---

## Key Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Train/test split | Temporal (not random) | Prevents data leakage — time order must be respected |
| Primary metric | PR-AUC | Imbalanced classes — ROC-AUC misleads; PR-AUC focuses on the minority class |
| Processing | PySpark + pandas | PySpark for scalable aggregation; pandas for exploration |
| Explainability | SHAP | Model-agnostic, shows direction + magnitude of each feature |

---

## Success Criteria

| Metric | Target |
|---|---|
| PR-AUC | ≥ 0.50 |
| Recall | ≥ 0.70 |
| Precision | ≥ 0.40 |

---

## Project Structure

```
surge-pricing-prediction/
├── notebooks/
│   ├── 01_eda.ipynb                  ← Data exploration and cleaning
│   ├── 02_feature_engineering.ipynb  ← Aggregation, target creation, features
│   ├── 03_modelling.ipynb            ← Training and comparison
│   └── 04_evaluation_and_insights.ipynb ← Metrics + SHAP
├── src/
│   ├── data_loader.py
│   ├── features.py
│   ├── train.py
│   └── evaluate.py
├── reports/
│   ├── project_plan.md               ← Full planning document
│   └── findings.md                   ← Results and business insights
├── data/
│   └── README.md                     ← Data sources (raw files not committed)
└── requirements.txt
```

---

## How to Run

```bash
# Install dependencies
pip install -r requirements.txt

# Download data (see data/README.md for instructions)

# Run notebooks in order
jupyter notebook notebooks/01_eda.ipynb
```

---

## Skills Demonstrated

- **Problem framing** — constructing an ML problem from raw business data
- **Large-scale data processing** — PySpark for aggregation at scale
- **Time-series feature engineering** — lag features, rolling windows, cyclical encoding
- **Temporal validation** — proper train/test splitting for time-ordered data
- **Gradient boosting** — LightGBM with hyperparameter tuning
- **Imbalanced classification** — PR-AUC, recall/precision trade-off analysis
- **Model explainability** — SHAP values for business-actionable insights
- **Business communication** — findings translated into operational recommendations
