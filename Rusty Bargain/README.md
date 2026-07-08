# Rusty Bargain — Used Car Price Prediction

This is a project I worked on in the TripleTen Data Science program.

### Objective

To build a machine learning model that estimates the **market value** of used cars for Rusty Bargain’s customer-facing app. The company cares about three factors:

1. **Prediction quality** (RMSE)
2. **Prediction speed** (inference time)
3. **Training time**

### Background

Rusty Bargain is developing an app where users can quickly find out what their car is worth. Historical listings include technical specs, trim details, and sale prices.

The dataset is provided by TripleTen (~354K listings; cleaned to a smaller working set).

### Data Dictionary

File: `car_data.csv` (also included locally as `Rusty Bargain car_data.csv`)

| Column | Description |
|--------|-------------|
| `DateCrawled` | Date profile was downloaded (dropped) |
| `DateCreated` | Profile creation date (dropped) |
| `LastSeen` | Last user activity date (dropped) |
| `RegistrationMonth` | Registration month (dropped) |
| `NumberOfPictures` | Picture count — all zeros (dropped) |
| `VehicleType` | Body type (sedan, SUV, etc.) |
| `RegistrationYear` | Registration year (proxy for model year) |
| `Gearbox` | Manual / automatic |
| `Power` | Horsepower (hp) |
| `Model` | Vehicle model |
| `Mileage` | Odometer (km) |
| `FuelType` | Fuel type |
| `Brand` | Manufacturer |
| `NotRepaired` | Whether damage was repaired |
| `PostalCode` | Seller postal code |
| **`Price`** | **Target:** sale price (EUR) |

### Data preparation

* Dropped duplicates and non-predictive date columns
* Fixed invalid `RegistrationYear` (kept 1900–2025)
* Filtered impossible `Power` values (kept 10–1000 hp)
* Merged `gasoline` / `petrol` fuel types
* Imputed missing categoricals (`Model` → `other`; `Gearbox`, `FuelType`, `NotRepaired`, `VehicleType` → `unknown`)
* Dropped rows with zero price (~3% of data)
* EDA: price is right-skewed; **power** and **registration year** most correlated with price; mileage capped at 150K km

**Train / validation / test split:** 70% / 15% / 15%

**Encoding pipelines:**
| Model family | Encoding |
|--------------|----------|
| Linear Regression | One-hot encoding + `StandardScaler` |
| Decision Tree / Random Forest | Ordinal encoding (high cardinality) |
| XGBoost | One-hot encoding |
| CatBoost | Native categorical handling |
| LightGBM | Native categorical (`astype('category')`) |

Cell runtimes tracked with **`%%time`** (Jupyter magic).

---

### Models trained

Six models compared with hyperparameter tuning (`GridSearchCV` where applicable):

| Model | CV / Val RMSE (€) | Best hyperparameters (summary) | Tuning wall time |
|-------|-------------------|-------------------------------|------------------|
| **Linear Regression** | Val **2,797** | (sanity check — no tuning) | ~18.5 s |
| Decision Tree | CV **1,964** | GridSearch on depth/splits | ~25 s |
| Random Forest | CV **1,775** | `max_depth=14`, `n_estimators=100` | ~3 min |
| XGBoost | Val **1,678** | `lr=0.1`, `max_depth=6`, `n_estimators=300` | ~7 min |
| CatBoost | Val **1,698** | `depth=6`, `iterations=300`, `lr=0.1` | ~6 min |
| LightGBM | Val **1,650** | `lr=0.1`, `max_depth=6`, `n_estimators=300`, `num_leaves=63` | ~1.5 min |

Linear regression confirms gradient boosting models are learning meaningful patterns (all GBMs beat LR by a wide margin).

---

### Final test comparison (retrained on train + validation)

| Model | Train time (s) | Predict time (s) | RMSE per row (s) | **Test RMSE (€)** |
|-------|----------------|------------------|------------------|-------------------|
| **LightGBM** | 5.17 | 0.800 | 0.00001582 | **1,681** |
| XGBoost | 48.83 | 0.445 | 0.00000879 | 1,717 |
| **CatBoost** | 83.20 | **0.193** | **0.00000381** | 1,741 |
| Random Forest | 53.58 | 0.572 | 0.00001130 | 1,756 |
| Decision Tree | 1.05 | 0.009 | 0.00000018 | 1,888 |
| Linear Regression | 17.77 | **0.001** | 0.00000002 | 2,835 |

---

### Recommendation

**LightGBM** achieves the **lowest test RMSE** (~€1,681). **CatBoost** is recommended for production:

- Only ~€60 higher RMSE than LightGBM (~3.5%)
- **Fastest inference** among top gradient boosting models (0.19 s vs 0.80 s for LightGBM on test set)
- Native handling of categorical features and missing values — better for real-world messy listings
- Lower server cost and easier scaling for a customer-facing app

Gradient boosting models clearly outperform tree ensembles and linear regression on this task.

Please see the included Jupyter Notebook (`Rusty Bargain.ipynb`) for the full analysis.
