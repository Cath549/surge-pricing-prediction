# Surge Pricing Prediction — NYC Taxi Demand Forecasting

A machine learning project that predicts surge-level taxi demand by zone and time window, using NYC TLC Yellow Taxi data.

---

## What This Project Is About

Ride-hailing platforms like Grab use surge pricing to balance driver supply and passenger demand. But the best-run platforms don't just _react_ to surges — they try to _predict_ them before they happen, so they can pre-position drivers, reduce wait times, and activate pricing gradually instead of reactively.

This project builds that prediction system using real NYC taxi trip data. The core question is: **given what's happened in a pickup zone over the last hour or so, can we predict whether that zone is about to hit surge-level demand in the next 15–30 minutes?**

It's not a toy problem. NYC generates millions of taxi trips per month, demand is highly uneven across zones and time-of-day, and the "surge" label doesn't exist in the raw data — we have to engineer it ourselves. That makes this more interesting and more realistic than most tutorial-style projects.

---

## The Data

**Source:** NYC Taxi & Limousine Commission (TLC) public trip records

**Why yellow taxi?** The TLC publishes data for multiple vehicle types — yellow taxis, green boro taxis, and high-volume for-hire vehicles (Uber, Lyft). Yellow taxis were the right choice here for a few reasons: they operate across all of Manhattan and the airports (the highest-density surge zones), they have the most complete and consistent data quality, and they generate enough trip volume per zone to compute a meaningful demand signal. Green taxis are geographically limited to outer boroughs and upper Manhattan, which would exclude most of the interesting surge dynamics.

**Why Q1 2023?** 2023 is the first year where NYC taxi demand patterns returned to something genuinely normal after COVID. Earlier years are either fully disrupted (2020) or still in recovery (2021–2022). We used Q1 specifically — January through March — to stay within a single season and avoid needing to model annual seasonality patterns. January and February form the training set; March is held out as the test set.

**External data:** We also pulled in hourly weather data from NOAA (rain and temperature are strong demand drivers) and a simple holiday calendar flag for Q1 2023 federal holidays. These are small, joinable by timestamp, and add real signal.

---

## The Problem Setup

The raw data is individual trips. We aggregate them to **zone × 15-minute windows** — so each row in our modeling dataset represents one NYC pickup zone in one 15-minute slot.

The target label (surge = yes/no) doesn't exist in the raw data. We define it as: demand in this zone-window exceeds the zone's rolling historical baseline by more than a threshold we determine during EDA. This is the same basic logic that ride-hailing platforms use in practice.

This is a **binary classification** problem. The secondary extension is a regression model that predicts the actual demand volume instead of just the surge flag — that's useful for pre-positioning a specific number of drivers.

**What makes it non-trivial:**

- The label is engineered, so we have to defend the threshold choice
- Surge events are the minority class, so accuracy is useless as a metric — we use PR-AUC
- The data has temporal structure, so train/test split must be time-based, not random
- Features require lag engineering: "what was demand 15 minutes ago, 1 hour ago, same time last week?"

---

## Technical Approach

| Step                | What We Do                                                                             |
| ------------------- | -------------------------------------------------------------------------------------- |
| EDA                 | Explore demand patterns, spatial distribution, time-of-day trends, weather correlation |
| Feature Engineering | Aggregate to zone-window level, build lag features, join weather and holiday data      |
| Modeling            | Train logistic regression and random forest baselines, then LightGBM as the main model |
| Evaluation          | PR-AUC as primary metric; precision/recall tradeoff analysis; SHAP for explainability  |

**Key tools:** pandas, PySpark (for the aggregation pipeline — this is the kind of scale you'd see in production), LightGBM, SHAP, geopandas, folium.

---

## Project Structure

```
surge-pricing-prediction/
├── data/
│   ├── raw/           # TLC parquet files — not in Git (too large)
│   ├── processed/     # Cleaned, aggregated outputs
│   └── external/      # Zone shapefiles, weather data
├── notebooks/         # Jupyter notebooks — EDA, feature eng, modeling, evaluation
├── src/               # Reusable Python modules
│   ├── data/          # Data download and preprocessing scripts
│   ├── features/      # Feature engineering logic
│   ├── models/        # Training pipeline
│   └── utils/         # Spark session helpers, utilities
├── models/            # Saved model artifacts
├── reports/
│   ├── figures/       # Exported plots
│   └── project_report.md
├── requirements.txt
└── README.md
```

---

## How to Run It

```bash
# Clone the repo
git clone https://github.com/Cath549/surge-pricing-prediction.git
cd surge-pricing-prediction

# Set up the environment
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Download the raw data (TLC parquet files for Jan–Mar 2023)
# See src/data/download.py for instructions

# Open notebooks in order
jupyter notebook
```

---

## Results

_(To be updated as the project progresses)_

---

## What I Learned / What This Demonstrates

_(To be updated once modeling is complete)_

---

## Author

Built by [Cath549](https://github.com/Cath549) as a portfolio project targeting MLE roles in Southeast Asia.
