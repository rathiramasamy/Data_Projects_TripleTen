# Zyfra Gold Recovery — Predicting Gold Recovery from Ore

This is a project I worked on in the TripleTen Data Science program.

### Objective

To build a machine learning prototype for **Zyfra** that predicts gold recovery rates during ore extraction and purification. The model helps optimize production, eliminate unprofitable parameters, and flag input conditions that lead to low recovery.

### Background

Zyfra develops efficiency solutions for heavy industry. This project uses time-indexed process data from a gold recovery plant. Parameters measured close together in time are often similar; some features are missing because they were calculated later.

The data is synthetic (contract details undisclosed). Provided by TripleTen.

### Data Dictionary

Three files (also included locally in this folder):

| File | Description |
|------|-------------|
| `gold_recovery_train.csv` | Training set with targets (~16,860 rows, 87 columns) |
| `gold_recovery_test.csv` | Test set — no targets; fewer features |
| `gold_recovery_full.csv` | Combined source with all features |

**Key columns:**
- `date` — timestamp of measurement
- Process stages: **rougher** (flotation), **primary_cleaner**, **secondary_cleaner**, **final**
- Feature types: `input.*`, `state.*` (available at prediction time); `output.*` and `calculation.*` (dropped from training except targets)
- Metal concentrations: **Au** (gold), **Ag** (silver), **Pb** (lead), plus solids (`sol`)
- **Targets:** `rougher.output.recovery` and `final.output.recovery` (%)

### Process

**1. Data preparation**
* Loaded train, test, and full datasets; converted `date` to datetime and sorted chronologically
* **Verified recovery formula** for rougher stage:

  `recovery = (C × (F − T)) / (F × (C − T)) × 100`

  where C = concentrate Au, F = feed Au, T = tail Au  
  **MAE ≈ 9.3×10⁻¹⁵** — calculated values match stored feature (negligible error)

* **Features missing from test set:** all `output.*` and `calculation.*` parameters (measured later). Merged targets from `gold_recovery_full.csv` into test for evaluation; dropped other output/calculation columns from training
* Dropped rows with missing target values
* Imputed remaining missing features with **cubic spline interpolation** (order 3) — preserves time-series patterns
* Removed anomalies where **total metal concentration = 0** at any purification stage

**2. Exploratory analysis**
* **Metal concentrations by stage:** Au increases through purification; Ag rises after flotation then decreases; Pb stabilizes ~10% in final concentrate
* **Feed particle size (train vs test):** KDE plots for `rougher.input.feed_size` and `primary_cleaner.input.feed_size` show similar distributions — evaluation is valid
* **Total concentration anomalies:** zero-sum rows removed from both train and test

**3. Model building**
* **Multi-output regression** predicting both recovery targets
* **sMAPE** metric with weighted final score: `0.25 × rougher + 0.75 × final`
* **5-fold cross-validation** on training set
* **DummyRegressor** sanity check (mean/median baseline)

### Model comparison (5-fold CV, training set)

| Model | CV sMAPE |
|-------|----------|
| Decision Tree | 9.80% |
| **Random Forest** | **8.87%** |
| Linear Regression | 11.55% |

**Best model:** `RandomForestRegressor(n_estimators=50, max_depth=5)`  
`GridSearchCV` over `n_estimators`, `max_depth`, `min_samples_split`, `min_samples_leaf` did not improve CV score (best params matched defaults).

### Results

| Evaluation | Final sMAPE |
|------------|-------------|
| **Test set (Random Forest)** | **7.62%** |
| Mean baseline | 7.78% |
| Median baseline | 7.78% |

The Random Forest beats naive baselines by ~**0.15 percentage points** on test sMAPE. Test performance is slightly better than CV, suggesting reasonable generalization.

### Conclusion

The **Random Forest** prototype achieves **7.6% sMAPE** on held-out test data for gold recovery prediction. Zyfra can use it to:

- Simulate production scenarios and identify unprofitable conditions
- Avoid processing ore with unfavorable input characteristics
- Support real-time decisions before adjusting process parameters

Further gains may come from continuous retraining on new data or exploring additional model types.

Please see the included Jupyter Notebook (`Zyfra.ipynb`) for the full analysis.
