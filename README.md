# Equities Weather Prediction Model

## Description

This project builds a monthly panel of US equities in three sectors (Agriculture & Food, Utilities & Telecom, Leisure & Tourism) that are more susceptible to weather shocks, maps each firm to the counties where it operates, attaches county-level rainfall and temperature anomalies and state-level solar irradiance anomalies, and tests whether those shocks add explanatory power on top of the Fama-French 3-factor model.

## Pipeline

Run the notebooks in order. Each stage writes intermediate files to `Data/Intermediate/` for the next stage.

| Stage | Notebook | What it does |
|---|---|---|
| 0 | `0_Final Project CRSP Setup` | Pulls CRSP monthly returns, Compustat fundamentals, the CCM link table and S&P 500 history from WRDS; flags S&P 500 / Russell 2000 (proxy) membership and atmospheric sectors; saves `master_crsp_merged.parquet` |
| 1 | `1_Final Project Company Operations` | Scrapes Item 2 (Properties) of each firm's 10-K from SEC EDGAR to find operating city/state locations |
| 1 | `1_Final Project Env Factors_Rainfall_Temp` | Parses NOAA nClimDiv county precipitation and temperature files; computes county-month z-scores and ±2σ anomaly flags |
| 1 | `1_Final Project Env Factors_Solar` | Computes state-month z-scores for NSRDB Direct Normal Irradiance (DNI) |
| 1 | `1_Final Project Env Factors_Geomagnetic` | Placeholder — not yet implemented |
| 1 | `1_Final Project Merge and Clean Operations and Environmental` | Maps firm locations to county FIPS, merges the weather and solar anomalies, and collapses to one row per firm-month; saves `panel.parquet` |
| 1 | `1_Final Project Merge FF3 Factors` | Adds monthly Fama-French 3 factors and the risk-free rate from WRDS |
| 2 | `2_Final Project Regression Analysis` | PanelOLS with firm fixed effects: FF3-only vs. FF3 + weather, across lags (T, T-1, T-3) and sample cuts (sector, large/small cap) |
| 2 | `2_Final Project Regression Analysis - IS OOS Test` | Same regression framework, with an in-sample / out-of-sample test |
| 2 | `2_Final Project Nonlinear Analysis` | Spline regression, quantile regression, gradient boosting + SHAP, and portfolio sorts on lag T-1 weather signals |

## Setup

```bash
pip install pandas numpy matplotlib seaborn pyarrow wrds edgartools pgeocode linearmodels statsmodels scikit-learn shap patsy scipy
```

Credentials are read from environment variables rather than stored in the notebooks:

```bash
export WRDS_USERNAME="your_wrds_username"
export SEC_EDGAR_IDENTITY="Your Name your.email@example.com"   # SEC asks for a name + email
```

## Data

Data files not included in this repository. The notebooks expect this layout next to the `notebooks/` folder:

```
Data/
├── Intermediate/                       # created by the pipeline
├── Prec.Temp Raw Data/
│   ├── climdiv-pcpncy-v1.0.0-20260205  # NOAA nClimDiv county precipitation
│   └── climdiv-tmpccy-v1.0.0-20260205  # NOAA nClimDiv county temperature
└── nsrdb solar data.csv                # NSRDB state-level monthly DNI
```

Sources: [WRDS](https://wrds-www.wharton.upenn.edu/) (CRSP, Compustat, Fama-French), [SEC EDGAR](https://www.sec.gov/edgar), [NOAA nClimDiv](https://www.ncei.noaa.gov/pub/data/cirs/climdiv/), [NREL NSRDB](https://nsrdb.nrel.gov/).
