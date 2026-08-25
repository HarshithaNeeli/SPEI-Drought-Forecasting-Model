# 🌦️ SPEI-3 Drought Forecasting — Ninh Thuận Province, Vietnam

Hydroclimatic drought forecasting pipeline for three districts in Ninh Thuận Province, Vietnam (Ninh Sơn District, Phan Rang-Tháp Chàm, Thuận Bắc District), using the Standardized Precipitation Evapotranspiration Index (SPEI-3).

An NSE-weighted ensemble of **Ridge Regression, LightGBM, XGBoost, and SARIMA** is used to forecast monthly SPEI-3 values from **January 2026 to December 2030**, with **90% conformal prediction intervals** and **moving-block bootstrap** uncertainty quantification.

---

## 📌 Project Overview

| | |
|---|---|
| **Index** | SPEI-3 (3-month Standardized Precipitation Evapotranspiration Index) |
| **Locations** | Ninh Sơn District, Phan Rang-Tháp Chàm, Thuận Bắc District |
| **Data period** | January 1981 – December 2025 (monthly) |
| **Train / Test split** | 80 / 20 chronological split |
| **Forecast horizon** | January 2026 – December 2030 (60 months) |
| **Models** | Ridge, LightGBM, XGBoost, SARIMA → NSE-weighted ensemble |
| **Uncertainty** | 90% Conformal Prediction intervals, Moving-Block Bootstrap |
| **Evaluation metrics** | R², NSE (Nash–Sutcliffe Efficiency), Index of Agreement (d), RMSE, MAE |

---

## 🗂️ Repository Structure

```
├── notebooks/
│   └── SPEI_Drought_Forecasting_Model.ipynb   # Full end-to-end pipeline
├── data/
│   └── dataToHarshitha.csv                    # Raw monthly SPEI-3 input data (1981–2025)
├── outputs/
│   ├── SPEI3_Monthly_Forecast_2026_2030.csv   # Final 60-month forecast output
│   └── figures/                               # Exported plots (EDA, diagnostics, forecasts)
├── docs/
│   └── pipeline_architecture.md               # Mermaid diagram of the pipeline
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

---

## 🔬 Methodology

1. **Data validation & EDA** — Wet/dry bar charts, distribution (KDE + normal fit) analysis, ACF/PACF per location.
2. **Feature engineering** — Lags 1–4 (SPEI-3 has short hydrological memory, so lags are intentionally capped), 3- and 6-month rolling mean/std, cyclical month encoding (sin/cos).
3. **Train/test split** — Chronological 80/20 split per location, `StandardScaler` fit on training data only.
4. **Model training** — Ridge, LightGBM, XGBoost, SARIMA(1,0,1)(1,0,1,12) trained per location; both training and test metrics computed.
5. **Evaluation** — R², NSE, Index of Agreement (d), RMSE, MAE, with a train-vs-test performance heatmap and residual diagnostics.
6. **Uncertainty quantification** — 90% conformal prediction intervals (calibrated on held-out residuals) and moving-block bootstrap (block size 12).
7. **5-fold time-series cross-validation** for Ridge, LightGBM, XGBoost.
8. **60-month recursive forecast (2026–2030)** — Each step's ensemble prediction is fed back as the next lag input. Final ensemble = NSE-softmax-weighted blend of Ridge, LightGBM, and SARIMA, with prediction intervals that widen over the horizon to reflect growing uncertainty.

---

## 📊 Output

`outputs/SPEI3_Monthly_Forecast_2026_2030.csv` contains, per location and month:

| Column | Description |
|---|---|
| `Location`, `Year`, `Month`, `Month_Name`, `Date` | Time identifiers |
| `SPEI3_Ridge`, `SPEI3_LightGBM`, `SPEI3_SARIMA` | Individual model forecasts |
| `SPEI3_Ensemble` | NSE-weighted ensemble forecast |
| `PI_Lower_90`, `PI_Upper_90` | 90% conformal prediction interval bounds |
| `SPEI_Category` | Drought/wet classification (e.g., Moderate Drought, Near Normal, Very Wet) |
| `Ensemble_Weights` | Per-location model weights used in the ensemble |

---

## ▶️ How to Run

1. Open `notebooks/SPEI_Drought_Forecasting_Model.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Upload `data/dataToHarshitha.csv` when prompted (or place it in the Colab working directory).
3. Run all cells top to bottom (`Runtime → Run all`).
4. The forecast CSV and all figures will be generated in the Colab environment — download and place them under `outputs/` if regenerating.

**Requirements:** see `requirements.txt`. Core libraries: `xgboost`, `lightgbm`, `statsmodels`, `scikit-learn`, `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`.

---

## 🧠 Key Modeling Notes

- **Lag features restricted to 1–4 months** — SPEI-3 does not carry long historical memory, so longer lags risk spurious correlation.
- **StandardScaler fit only on training data** to avoid leakage, especially important given the training period skews drier/negative while the recent period trends wetter/positive.
- **NSE and Index of Agreement (d)** are used alongside R² because standard R² alone is insufficient for a bounded hydrological index like SPEI (typically −2.5 to 2.5).
- **Recursive multi-step forecasting** — each forecasted month is fed back into the feature set as a lag input for the next step.
- **Prediction interval widths increase over the 60-month horizon**, reflecting accumulating forecast uncertainty.

---

## 📄 License

This project is released under the [MIT License](LICENSE).
