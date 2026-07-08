# Sure Tomorrow Insurance — ML Evaluation Project

This is a project I worked on in the TripleTen Data Science program.

### Objective

To evaluate whether machine learning can help **Sure Tomorrow Insurance** with four tasks:

1. **Find similar customers** for targeted marketing
2. **Predict** whether a customer will receive any insurance benefit (classification)
3. **Predict the number** of insurance benefits (regression)
4. **Obfuscate personal data** without degrading the regression model

### Background

Sure Tomorrow wants to use ML across marketing, risk prediction, and privacy protection. The dataset contains demographic and financial attributes for insured persons, with benefits received over the last five years as the target.

The data is provided by TripleTen.

### Data Dictionary

File: `insurance_us.csv` (also included locally as `Sure Tomorrow Insurance_us.csv`) — **5,000** customers.

| Column | Description |
|--------|-------------|
| `gender` | 0 = female, 1 = male |
| `age` | Age (18–65) |
| `income` | Annual salary (USD) |
| `family_members` | Number of family members (0–6) |
| `insurance_benefits` | **Target:** number of benefits received in the last 5 years |

**Summary stats:** ~50% male/female; mean age ~31; mean income ~$39,916; mean benefits ~0.15 (most customers received **0** benefits).

### Data preparation

* Loaded and renamed columns to lowercase
* No missing values; converted `age` from float to int
* Explored distributions via histograms and pairplots — no extreme anomalies requiring removal
* **Task 2 binary target:** `insurance_benefits_received` = `insurance_benefits > 0` (~**11.3%** positive class — imbalanced)

---

### Task 1: Similar customers (kNN)

Built `get_knn()` using `NearestNeighbors` on features `gender`, `age`, `income`, `family_members` (benefits excluded).

Tested **4 combinations:**
- Scaled vs unscaled (`MaxAbsScaler`)
- **Euclidean** vs **Manhattan** distance

**Findings:**
- **Scaling is critical** — without it, high-variance features like `income` dominate distance
- Euclidean and Manhattan metrics return **very similar neighbors** — metric choice is flexible

---

### Task 2: Benefit receipt prediction (classification)

**kNN classifier** vs **random dummy** model; evaluated with **F1** (70/30 stratified split).

| Model | Best F1 | Notes |
|-------|---------|-------|
| Dummy (P = 0) | 0.00 | Always predicts "no benefit" |
| Dummy (P = 0.11) | 0.12 | Matches class prevalence |
| Dummy (P = 0.5 or 1) | 0.20 | Random guessing |
| **kNN, unscaled (k=1)** | **0.59** | Better than dummy |
| **kNN, scaled (k=1)** | **0.96** | Strong performance |
| kNN, scaled (k=2–10) | 0.89–0.93 | Remains high |

**Can a trained model do worse than dummy?** Yes — an unscaled kNN with larger k (e.g. k=4, F1 ≈ 0.19) can underperform the best dummy (F1 = 0.20). **Scaling and appropriate k matter.**

**Can it do better?** Yes — scaled kNN dramatically outperforms all dummy baselines.

---

### Task 3: Benefit count prediction (regression)

Custom **`MyLinearRegression`** (normal equation: `w = (XᵀX)⁻¹Xᵀy`); **70/30** train/test split.

| Data | RMSE | R² |
|------|------|-----|
| Unscaled | **0.34** | **0.66** |
| Scaled (`MaxAbsScaler`) | **0.34** | **0.66** |

Scaling does **not** affect linear regression predictions or metrics (as expected for this closed-form solution).

---

### Task 4: Data obfuscation

**Method:** multiply feature matrix by a random **invertible** matrix P:

`X_obfuscated = X × P`

**Analytical proof:** For linear regression, predictions are unchanged because the learned weights absorb P⁻¹.

**Computational verification:**

| Data | RMSE | R² |
|------|------|-----|
| Original X | 0.34 | 0.66 |
| Obfuscated X × P | 0.34 | 0.66 |

Obfuscation **masks raw personal attributes** while preserving model quality — validated for linear regression.

---

### Conclusion

Machine learning is viable for Sure Tomorrow across all four tasks:

- **kNN + scaling** finds meaningful similar customers for marketing
- **kNN classification** (scaled) predicts benefit receipt far better than random baselines
- **Linear regression** estimates benefit counts with RMSE 0.34 and R² 0.66
- **Matrix obfuscation** protects client data without breaking the regression model

Please see the included Jupyter Notebook (`Sure Tomorrow Insurance.ipynb`) for the full analysis.
