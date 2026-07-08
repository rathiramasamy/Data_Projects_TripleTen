# Interconnect — Customer Churn Prediction

This is a project I worked on in the TripleTen Data Science program.

### Objective

To build a model that predicts **customer churn** for telecom operator Interconnect, so at-risk users can be offered promotional codes and special plan options before they leave.

**Primary metric:** **ROC-AUC**.

### Background

Interconnect provides landline phone, internet (DSL/fiber), and add-on services (security, tech support, cloud backup, streaming). Customers can pay monthly or sign 1- or 2-year contracts. Contract data is valid as of **February 1, 2020**.

The marketing team collected personal data, plan details, and contract history across four source files, merged by `customerID`.

### Data Dictionary

| File | Key columns |
|------|-------------|
| `contract.csv` | `BeginDate`, `EndDate`, `Type`, `PaperlessBilling`, `PaymentMethod`, `MonthlyCharges`, `TotalCharges` |
| `personal.csv` | `gender`, `SeniorCitizen`, `Partner`, `Dependents` |
| `internet.csv` | `InternetService`, `OnlineSecurity`, `OnlineBackup`, `DeviceProtection`, `TechSupport`, `StreamingTV`, `StreamingMovies` |
| `phone.csv` | `MultipleLines` (renamed to `phone_lines`) |

**Target:** `churned` = 1 if `EndDate` ≠ `'No'` (customer left); 0 if still active  
**Initial dataset:** 7,043 customers → **6,829** after cleaning

### Data preparation

1. Merged all four tables on `customerID`
2. Created **`months_active`** from contract dates; dropped `BeginDate` / `EndDate`
3. Renamed `Type` → `contract_type`; converted `TotalCharges` to float (11 invalid rows)
4. Filled missing service fields: `InternetService` / `phone_lines` → `'None'`; add-ons → `'No'`
5. Dropped **`TotalCharges`** (highly correlated with `months_active`)
6. Removed **214 involuntary churners** on 1- or 2-year contracts (~3% of data)
7. Dropped **`gender`** (not predictive)
8. Encoded Yes/No columns as 0/1; set `customerID` as index

### EDA — Class imbalance

| Class | Share |
|-------|-------|
| Active (no churn) | ~73.5% |
| Churned | ~26.5% |

Moderate imbalance — **ROC-AUC** is preferred over accuracy.

**Key churn drivers:**
- Month-to-month contracts (vs. 1- or 2-year)
- Low add-on adoption (no security, backup, tech support)
- Electronic check payment method
- Shorter tenure (`months_active` negatively correlated with churn)

---

### Models trained

Five classifiers with `GridSearchCV` (5-fold stratified CV, `scoring='roc_auc'`):

| Model | Preprocessing | Best hyperparameters (summary) | **Val ROC-AUC** |
|-------|---------------|-------------------------------|-----------------|
| Logistic Regression | OHE + scaling | `C=0.1`, `penalty=l2`, `solver=lbfgs` | 0.905 |
| Random Forest | OHE | `max_depth=10`, `n_estimators=200` | 0.910 |
| CatBoost | Native categoricals | `depth=6`, `iterations=200`, `lr=0.1` | 0.943 |
| LightGBM | OHE | `lr=0.1`, `max_depth=6`, `n_estimators=200`, `num_leaves=30` | 0.947 |
| **XGBoost** | OHE | `lr=0.1`, `max_depth=6`, `n_estimators=200` | **0.949** |

**Train / validation / test split:** 70% / 15% / 15% (stratified)

### Final result

**XGBoost** retrained on train + validation:

| Set | ROC-AUC |
|-----|---------|
| **Test** | **0.936** |

Gradient boosting models clearly outperform logistic regression and random forest. XGBoost achieved the highest validation AUC and strong test-set generalization.

### Conclusion

**XGBoost** is recommended for Interconnect's churn prediction system. With test ROC-AUC ≈ **0.94**, the model reliably ranks customers by churn risk so marketing can target retention offers (promo codes, plan upgrades) at the right time.

Top actionable signals from EDA: prioritize **month-to-month** customers with **low service adoption** and **manual payment methods** for outreach.

Please see the included Jupyter Notebook (`Interconnect.ipynb`) for the full analysis.
