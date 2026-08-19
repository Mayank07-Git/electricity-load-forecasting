# Electricity Load Forecasting

Machine-learning based **24-hour ahead German electricity load forecasting** using historical ENTSO-E load data, calendar features, weather variables and LightGBM.

The project was developed for the **DDMO SoSe 2026 electricity-load forecasting challenge** and focuses on a robust, leakage-aware forecasting workflow with recent backtesting and benchmark comparison.

## Project overview

```text
ENTSO-E Actual Load
        +
Calendar Features
        +
Weather Features
        ↓
Data Cleaning / Outlier Handling
        ↓
Lagged Load Features
        ↓
LightGBM Recursive Forecaster
        ↓
24-Hour German Load Forecast
        ↓
MAE / RMSE / MAPE Evaluation
        ↓
Persistence Benchmark Comparison
```

## Key results

The final live-style backtest used a **7-day recent evaluation window**.

| Metric | LightGBM model | Weekly persistence |
|---|---:|---:|
| Mean MAE | **970.2 MW** | 1,727.9 MW |
| Mean RMSE | **1,174.4 MW** | 2,179.6 MW |
| Mean MAPE | **1.93%** | 3.42% |
| Days better by MAE | **6/7** | 1/7 |
| Median daily MAE | **0.90 GW** | 1.49 GW |
| Maximum daily MAE | **1.42 GW** | 3.04 GW |

For an additional forecast-vs-actual test day, the model achieved:

- **MAE:** 895.3 MW
- **RMSE:** 1,117.6 MW
- **MAPE:** 1.70%

> These metrics are from the project's recorded June 2026 backtesting workflow. They should be interpreted as a recent validation result rather than a guarantee of future forecasting performance.

## Model

The main forecasting model uses:

- **LightGBM**
- Recursive multi-step forecasting
- 168-hour weekly lag
- Calendar features
- Weather features
- Explicit outlier annotation / weighting
- Deterministic training configuration

Core estimator configuration:

```text
n_estimators       = 400
learning_rate      = 0.05
num_leaves         = 63
min_child_samples  = 20
random_state       = 2026
deterministic      = True
force_col_wise     = True
```

## Data and features

### Load data

Historical German electricity load is obtained from **ENTSO-E Actual Load** data.

The pipeline converts the source series to hourly resolution and trains only on actually published historical values.

### Calendar features

The forecasting workflow uses time-based information such as:

- hour of day
- day of week
- month / seasonal information
- weekday/weekend structure
- other calendar-derived features used by the forecasting pipeline

### Weather features

Weather variables are incorporated as exogenous predictors for the German load forecast.

### Lagged demand

A **168-hour weekly lag** captures the strong weekly structure in electricity demand. Additional historical information is handled through the forecasting framework and feature pipeline.

## Data-quality safeguards

The project contains explicit checks for:

- ENTSO-E data freshness
- frame coverage
- missing hourly values
- training/prediction feature completeness
- stale Actual Load data
- forecast horizon length
- NaN predictions

Outlier rows are annotated and handled through a weighting mechanism rather than silently replacing the underlying historical observations.

## Backtesting methodology

The evaluation is designed to avoid using future information.

For each backtest day:

1. Build the training set only from information available before the target day.
2. Prepare calendar and weather exogenous variables.
3. Fit the LightGBM recursive forecaster.
4. Predict the next 24 hours.
5. Compare predictions with actual ENTSO-E load.
6. Calculate MAE, RMSE and MAPE.
7. Compare against weekly persistence.

The recent backtest also records forecast bias and per-day performance.

## Project structure

```text
electricity-load-forecasting/
│
├── README.md
├── LICENSE
├── SECURITY.md
├── requirements.txt
├── pyproject.toml
├── .gitignore
│
├── notebooks/
│   └── live_preprocessing_v2_original_restored.ipynb
│
├── src/
│   ├── forecast_pipeline.py
│   ├── metrics.py
│   ├── preprocessing.py
│   └── ...
│
├── data/
│   └── README.md
│
├── submissions/
│   └── neura/
│       └── forecast CSV submissions
│
├── docs/
│   └── project documentation
│
└── .github/
    └── workflows/
        └── scorecard.yml
```

## Installation

Python 3.11+ is recommended.

```bash
python -m venv .venv
```

### Windows

```powershell
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### Linux / macOS

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

## ENTSO-E API configuration

The live notebook reads the ENTSO-E API key from an environment variable:

```text
ENTSOE_API_KEY
```

Do **not** commit API keys or `api.env` files.

Example Windows PowerShell configuration:

```powershell
$env:ENTSOE_API_KEY="YOUR_API_KEY"
```

The notebook checks whether the variable exists before attempting a live download.

## Running the notebook

Start Jupyter:

```bash
jupyter notebook
```

Open:

```text
notebooks/live_preprocessing_v2_original_restored.ipynb
```

The notebook contains the end-to-end live-style workflow, including:

- data acquisition
- preprocessing
- feature construction
- model training
- backtesting
- forecast-vs-actual validation
- persistence comparison
- provenance logging

## Submissions

The `submissions/neura/` directory contains forecast CSV files generated for the challenge workflow.

Raw training data, cached datasets and trained model artifacts are intentionally not committed.

## Security

The repository includes `SECURITY.md` and automated security-analysis configuration.

Never commit:

- ENTSO-E API keys
- `.env` files
- `api.env`
- downloaded raw datasets containing credentials or private information
- local cache directories
- trained model binaries unless explicitly intended for publication

## Limitations

- The recent validation window is relatively short.
- Electricity demand is affected by weather, holidays, market conditions and unusual events that are difficult to model perfectly.
- Backtest performance does not guarantee leaderboard or future live performance.
- The public repository intentionally excludes the large raw datasets and local cache used during development.

## Technologies

**Python · LightGBM · Pandas · NumPy · Scikit-learn · Jupyter · ENTSO-E · Time-Series Forecasting · Feature Engineering · Weather Data · Machine Learning**

## Author / Team

**Team Neura — DDMO SoSe 2026**

Project focused on data-driven electricity-load forecasting and machine-learning based forecasting evaluation.
