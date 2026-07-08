# Sweet Lift Taxi — Hourly Order Forecasting

This is a project I worked on in the TripleTen Data Science program.

### Objective

To build a model that predicts the **number of taxi orders in the next hour** at airport locations, so Sweet Lift Taxi can attract more drivers during peak demand.

**Success criterion:** test-set **RMSE ≤ 48**.

### Background

Sweet Lift Taxi collected historical order data at airports. The raw series is recorded every **10 minutes**; for this project it is aggregated to **hourly** totals. The goal is to compare machine learning and time-series forecasting approaches and select a model that meets the RMSE threshold.

The data is provided by TripleTen (March–August 2018).

### Data Dictionary

File: `taxi.csv` (also included locally as `Sweet Lift Taxi.csv`)

| Column | Description |
|--------|-------------|
| `datetime` (index) | Timestamp of each observation |
| **`num_orders`** | **Target:** number of taxi orders in the interval |

**Raw data:** 26,496 rows at 10-minute frequency  
**After hourly resample:** 4,416 rows (mean ~84 orders/hour, std ~45, max 462)

### Process

1. **Resample** 10-minute data to 1-hour sums (`resample('1H').sum()`)
2. **EDA:** time-series plots, seasonal decomposition, rolling means — data is **non-stationary** with upward trend (March→August) and clear **daily/weekly seasonality**
3. **Feature engineering** (`make_features`, max lag = 24):
   - Calendar: `hour`, `day of week`, `month`
   - Rolling stats (shifted): 24h and 168h mean and std
   - Lags: `lag_1` through `lag_24`
4. **Train/test split:** 10% test, **no shuffle** (chronological) — `test_size = int(0.1 × 4416) = 441` rows
5. **Models:** Linear Regression, Random Forest, LightGBM, CatBoost, XGBoost (with `GridSearchCV` where applicable), plus **auto_arima** (SARIMAX with daily seasonality m=24)
6. **Evaluation:** RMSE on held-out test set

---

### Model comparison (test set)

| Model | Best hyperparameters (summary) | **Test RMSE** | Meets ≤ 48? |
|-------|-------------------------------|---------------|-------------|
| **Random Forest** | `max_depth=14`, `n_estimators=100` | **41.0** | ✓ |
| XGBoost | `lr=0.05`, `max_depth=4`, `n_estimators=300`, `subsample=0.8` | 41.6 | ✓ |
| LightGBM | `lr=0.05`, `max_depth=6`, `n_estimators=100`, `num_leaves=31` | 41.5 | ✓ |
| **Linear Regression** | (no tuning — sanity baseline) | 45.1 | ✓ |
| CatBoost | `depth=4`, `iterations=300`, `lr=0.05` | 42.5 | ✓ |
| SARIMAX (auto_arima) | ARIMA(5,1,0)(1,0,0)[24] | **59.9** | ✗ |

All machine learning models beat the project threshold. The best **forecasting** model (auto_arima) did **not** meet the RMSE requirement.

---

### Key findings

- **Feature engineering mattered:** calendar features, rolling statistics, and 24-hour lags captured daily/weekly patterns that pure ARIMA struggled with on this short, trending series.
- **ML outperformed classical forecasting** on the test set — likely because only ~6 months of data were available and the series is non-stationary with a summer uptrend.
- **Random Forest** achieved the lowest test RMSE (~41.0), but all tuned ML models were within a few points of each other.

### Conclusion

**Random Forest** is the best model by test RMSE and clears the ≤ 48 requirement comfortably.

For production, a **gradient boosting model** (LightGBM or XGBoost) is a practical alternative: RMSE is only slightly higher (~41.5) with better speed and scalability for retraining at scale.

Collecting **more data** (a full year or multiple years) would help capture seasonal patterns beyond the March–August window and likely improve all models.

Please see the included Jupyter Notebook (`Sweet Lift Taxi.ipynb`) for the full analysis.
