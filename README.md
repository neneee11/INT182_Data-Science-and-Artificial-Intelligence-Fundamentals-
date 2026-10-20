# Store Sales Forecasting

**EDA, a leakage-safe ML pipeline and XGBoost for 16-day retail sales forecasting**

An end-to-end, reproducible forecasting project on the Kaggle [Store Sales – Time Series Forecasting](https://www.kaggle.com/competitions/store-sales-time-series-forecasting) dataset (Corporación Favorita, Ecuador). It covers a data-quality audit, exploratory analysis, leakage-safe feature engineering, rolling-origin validation of baselines, Decision Tree, Random Forest, XGBoost and AutoGluon, and a validated `submission.csv`.

![Recent history and 16-day test forecast](outputs/figures/16_test_forecast_total.png)

## Highlights

- **Forecasts 1,782 series** (54 stores × 33 product families) **16 days ahead** with a recursive multi-step XGBoost model.
- **Mean RMSLE 0.3986 on the two evaluation folds**, **18% better** than the strongest baseline (day-of-week mean of the last 4 weeks, RMSLE 0.4869).
- **Matches AutoGluon** (0.3981; the 0.0005 gap is smaller than the fold-to-fold spread) at about **1/8 of the training time**.
- **Leakage-safe by construction.** Every feature is built from a history masked at the forecast origin, and automated perturbation tests in the notebook check this.
- **One executed notebook** (24 sections) plus reusable modules in `src/`.

## Results

Mean over evaluation folds F2 + F3 (2017-07-15 → 2017-08-15), 16-day multi-step forecasts:

| Model | RMSLE ↓ | WAPE | MAE | RMSE | Bias | Fit time / fold |
|---|---|---|---|---|---|---|
| AutoGluon Tabular (LightGBM ×2 + XGBoost ensemble) | 0.3981 | 0.1356 | 64.2 | 236.7 | −0.9% | 403 s |
| **XGBoost (tuned) — selected** | **0.3986** | 0.1369 | 64.8 | 238.2 | −0.5% | 52 s |
| XGBoost (baseline config) | 0.4004 | 0.1375 | 65.1 | 237.3 | −0.9% | 90 s |
| XGBoost direct (lag ≥ 16) | 0.4129 | 0.1421 | 67.3 | 242.8 | −2.4% | 48 s |
| Random Forest (tuned) | 0.4178 | 0.1349 | 63.9 | 232.2 | −0.8% | 94 s |
| Decision Tree (tuned) | 0.4289 | 0.1477 | 69.9 | 255.1 | −0.8% | 35 s |
| Day-of-week mean, last 4 weeks (best baseline) | 0.4869 | 0.1556 | 73.7 | 260.1 | +1.6% | – |
| Historical mean, 28 days | 0.4952 | 0.2158 | 102.4 | 347.4 | +0.2% | – |
| Seasonal naive (t−7) | 0.5691 | 0.1732 | 81.9 | 307.6 | −0.1% | – |
| Naive (last value) | 0.6238 | 0.3551 | 167.8 | 599.8 | +12.1% | – |

**Why XGBoost over AutoGluon.**
- **Accuracy is a statistical tie.** The gap is 0.0005 RMSLE against a fold standard deviation of 0.006–0.010, and each model wins one fold.
- **XGBoost is cheaper to run.** It trains about 8× faster, and the model file is 2.6 MB instead of about 180 MB.
- **It is easier to explain.** A single model gives direct gain importance and SHAP values.
- **The rule was fixed in advance.** Before testing, we decided to take the lowest RMSLE, and to prefer the cheaper model when two are within 0.002.

**Ablations (XGBoost tuned).**
- **Promotion features matter.** Removing them worsens RMSLE to 0.4430, an 11% loss.
- **Lagged oil and transactions add nothing.** Without them the score is 0.3975, so they can be dropped.

**Evaluation protocol matters.** Scoring F3 one step ahead with actual lags would have reported RMSLE 0.3825, about 6% better than the valid 16-day recursive score of 0.4057.

**No test score is claimed.** The test set has no labels here. `outputs/submission.csv` (28,512 rows) passes every schema check against `sample_submission.csv`: same columns, row count and id order, and no missing, infinite or negative values.

<p align="center">
  <img src="outputs/figures/13_model_comparison.png" width="49%" alt="Model comparison">
  <img src="outputs/figures/14_xgb_feature_importance.png" width="49%" alt="XGBoost feature importance">
</p>

## Key EDA findings

- **Strong growth.** Average daily units rose +49%, +15%, +20% and +8% across 2014–2017, so the models train on recent data (from 2016).
- **Skewed, zero-inflated target.** 31% of rows are zero (mean 358, median 11), so the model fits `log1p(sales)`, which is equivalent to optimising RMSLE.
- **A dominant weekly cycle.** Sunday sales run at 1.30× trend and Thursday at 0.79×. Lag 7 and the same-weekday mean are top features.
- **Concentrated volume.** Five families make up 81% of units, and stores differ 9× in daily sales.
- **Holiday effects depend on the type of day.** "Additional" days run +21% and ordinary national holidays about ±0%. January 1 is near zero.
- **Seasonality sits at family level.** BOOKS peaks at ×5.1 in October, SCHOOL AND OFFICE SUPPLIES at ×4.5 in September and FROZEN FOODS at ×4.0 in December.
- **Promotions are strongly associated with sales.** This is observational: large stores and families run more promotions.

![Calendar effects](outputs/figures/07_calendar_effects.png)

## Method

```text
7 raw CSVs
  → data-quality audit (43 checks) + wide panel: 1,782 series × 1,704 days
  → 50 features, target = log1p(sales)
  → rolling-origin split: F1 (tuning) · F2, F3 (evaluation) · 16-day horizons
  → Pipeline(OrdinalEncoder → XGBRegressor), early stopping on the last 16 training days → refit
  → recursive 16-day forecast + zero rule (no sales in 56 days → 0)
  → RMSLE / WAPE / MAE / RMSE / bias, model selection
  → final fit on 2016-01-01 … 2017-08-15 (1,054,944 rows, 220 trees) → submission.csv
```

| Fold | Training data up to | 16-day forecast window | Used for |
|---|---|---|---|
| F1 | 2017-06-28 | 2017-06-29 → 07-14 | hyper-parameter tuning only |
| F2 | 2017-07-14 | 2017-07-15 → 07-30 | evaluation |
| F3 | 2017-07-30 | 2017-07-31 → 08-15 | evaluation |
| Test | 2017-08-15 | 2017-08-16 → 08-31 | submission |

**Features (50).**
- **Sales history (13).** Lags 1/7/14/28; rolling means over 7/14/28/56/364 days; rolling std over 7/28 days; same-weekday 4-week mean; 28-day zero share.
- **Calendar (14).** Date parts, weekend, month start/end, cyclical day-of-week/day-of-year encodings, payday.
- **Holidays (8).** National, regional and local, with transferred holidays handled.
- **Promotion (5).**
- **External (4).** Oil and store transactions, both lagged 16 or more days only.
- **Store / product (6).**

**Leakage control.**
- **Masked history.** Rolling windows end at d−1, and everything after the forecast origin is hidden.
- **External data lagged.** Oil and transactions are used only with values at least 16 days old, because future values are unknown at forecast time.
- **No row multiplication.** Holidays are aggregated before joining; a naive join on `date` would add 53,460 rows.
- **Training-only fitting.** Encoders are fitted on training rows only.
- **Untouched evaluation data.** Validation targets are never used for training or early stopping.

## Quick start

```bash
git clone <this-repo-url>
cd store-sales-forecasting
python3 -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
pip install "autogluon.tabular[lightgbm]==1.6.3"        # optional: AutoGluon experiment
```

1. Download `store-sales-time-series-forecasting.zip` from the [Kaggle competition page](https://www.kaggle.com/competitions/store-sales-time-series-forecasting/data) and put it in `data/` (or set `STORE_SALES_DATA_DIR` to a folder with the extracted CSVs).
2. Open the notebook:

```bash
jupyter lab notebooks/store_sales_eda_ml_pipeline.ipynb
```

Headless runs:

```bash
cd notebooks
# Quick-test mode (~5 min): short history, tiny searches -> outputs/quick_test/ (preliminary, not comparable)
STORE_SALES_QUICK_TEST=1 jupyter nbconvert --to notebook --execute store_sales_eda_ml_pipeline.ipynb \
    --output quick_run.ipynb --ExecutePreprocessor.timeout=-1
# Full mode (~45 min on 2 CPU cores): full searches, ablations, AutoGluon -> outputs/, models/
STORE_SALES_QUICK_TEST=0 jupyter nbconvert --to notebook --execute store_sales_eda_ml_pipeline.ipynb \
    --output full_run.ipynb --ExecutePreprocessor.timeout=-1
```

Reuse the trained model after a full run (`models/` is not committed, see below):

```python
import sys, joblib; sys.path.insert(0, "src")
from data_validation import load_data, preprocess_data
from predict import generate_predictions

state = preprocess_data(load_data("data"))
sub = generate_predictions(joblib.load("models/final_model.joblib"), state, mode="recursive")
```

**Setup notes.**
- **pandas must stay below 3.** AutoGluon 1.6 needs `pandas < 3`, which `requirements.txt` already pins.
- **macOS needs libomp for XGBoost.** Run `brew install libomp`.
- **AutoGluon and SHAP are optional.** If either is missing, the notebook prints a message, skips that step and carries on.
- **Tested environment:** Python 3.13.16 on Linux, 2 CPU cores, 7 GB RAM, no GPU.

## Project structure

```text
store-sales-forecasting/
├── data/README.md                          # how to get the dataset (raw CSVs are not committed)
├── notebooks/
│   └── store_sales_eda_ml_pipeline.ipynb   # main executed notebook, 24 sections
├── src/
│   ├── data_validation.py                  # extract_dataset, load_data, validate_data, preprocess_data
│   ├── feature_engineering.py              # build_features (recursive / direct), leakage-safe training frames
│   ├── train.py                            # splits, metrics, baselines, model fitting, recursive forecaster, comparison
│   ├── predict.py                          # generate_predictions, validate_submission, save_artifacts
│   └── eda.py                              # EDA and diagnostic plots
├── outputs/
│   ├── figures/                            # 18 figures: EDA, folds, comparison, importance, SHAP, diagnostics, forecast
│   ├── data_quality_report.csv             # 43 checks with magnitude / impact / treatment
│   ├── model_comparison.csv                # mean metrics on evaluation folds
│   ├── model_results_by_fold.csv           # every model × fold
│   ├── experiment_log.csv                  # every training run incl. hyper-parameter search
│   ├── xgb_search_results.csv, xgb_feature_importance.csv
│   ├── validation_predictions_best_model.csv
│   ├── run_config.json                     # package versions, selected params, runtime
│   └── submission.csv                      # final forecast (validated)
├── models/                                 # created by a full run; git-ignored
├── requirements.txt
└── README.md
```

**Not committed** (see `.gitignore`):
- **Data:** raw CSVs and the ZIP. The Kaggle competition rules apply to the data, so download it yourself.
- **Models:** `models/` (re-created by a full run).
- **Quick-test outputs:** `outputs/quick_test/`.

## Metrics

- **RMSLE** (primary; the competition metric): √mean[(log1p ŷ − log1p y)²]. It is a relative error, so small and large series count more evenly. Predictions are clipped at 0.
- **WAPE:** Σ|y − ŷ| / Σy, the share of total units mis-forecast.
- **MAE / RMSE:** errors in units, dominated by high-volume families. RMSE penalises large misses.
- **Bias:** Σ(ŷ − y) / Σy. Negative values mean under-forecasting.

## Limitations

- **Only two evaluation folds**, both in July–August 2017. Close models cannot be separated statistically, and other seasons (e.g. December) are untested.
- **Small hyper-parameter searches:** 6 XGBoost and 2 Random Forest configurations, because only 2 CPU cores were available.
- **Exposure bias in recursive forecasting.** The model trains on actual lags but forecasts from its own predictions.
- **Generic holiday flags.** The largest validation errors fell on the transferred-holiday long weekend of Aug 11–13, 2017, and SCHOOL AND OFFICE SUPPLIES was under-forecast by 55% during the back-to-school ramp-up.
- **AutoGluon ran as Tabular, not TimeSeries** (no PyTorch available), with a 480 s time limit per fold, so its results can vary slightly between runs.
- **`is_payday` comes from the competition description**, not from the data files.
- **Importance is not causation.** Feature importance and SHAP describe model behaviour, not causal effects.

## Future work

- **Validation:** more rolling-origin folds across a full year, with paired significance tests.
- **Holiday features:** holiday names, days to/from a holiday, and last year's sales on the same holiday.
- **Forecasting strategy:** hybrid direct/recursive forecasting, plus Tweedie or Poisson objectives for intermittent families.
- **Ensembling:** combine XGBoost, AutoGluon and Random Forest, whose error profiles differ.
- **Prediction intervals** for setting safety stock.

## Reproducibility

- **Seeds:** fixed (`RANDOM_STATE = 42`).
- **Package versions and selected hyper-parameters:** saved in `outputs/run_config.json`.
- **Model metadata:** a full run writes `models/final_model_metadata.json`.
- **Re-runs:** reproduce the Decision Tree, Random Forest and XGBoost results exactly. AutoGluon may differ slightly because of its time limit.

---

### สรุปภาษาไทย

โปรเจกต์นี้พยากรณ์ยอดขายรายวันล่วงหน้า 16 วัน ของ 1,782 series (54 ร้าน × 33 หมวดสินค้า) จากข้อมูล Kaggle Store Sales ใช้ XGBoost แบบ recursive บน features 50 ตัว ที่ออกแบบไม่ให้ข้อมูลอนาคตรั่ว และประเมินด้วยการแบ่งข้อมูลตามเวลา 3 ช่วง (F1 ใช้จูน F2 กับ F3 ใช้วัดผล) ได้ RMSLE เฉลี่ย 0.3986 ดีกว่า baseline ที่ดีที่สุด 18% และแม่นเท่ากับ AutoGluon แต่เทรนเร็วกว่าประมาณ 8 เท่า ขั้นตอนทั้งหมดอยู่ใน `notebooks/store_sales_eda_ml_pipeline.ipynb`
