# Beta Bank — Customer Churn Prediction

This is a project I worked on in the TripleTen Data Science program.

### Objective

To build a classification model that predicts whether a **Beta Bank** customer will leave the bank soon. The bank wants to retain existing customers rather than acquire new ones — so identifying at-risk clients early is critical.

The model must achieve an **F1 score of at least 0.59** on the test set. **AUC-ROC** is also measured and compared with F1.

### Background

Beta Bank customers have been leaving gradually. Using historical data on client behavior and contract terminations, this project trains and compares several classifiers, addresses **class imbalance**, tunes hyperparameters, and evaluates the best model on a held-out test set.

The data is provided by TripleTen (Kaggle-style bank churn dataset).

### Data Dictionary

File: `Churn.csv` (also included locally as `Beta Bank.csv`) — **10,000** customer records.

| Column | Description |
|--------|-------------|
| `RowNumber` | Row index (dropped — not predictive) |
| `CustomerId` | Unique customer ID (dropped — not predictive) |
| `Surname` | Customer surname (dropped — not predictive) |
| `CreditScore` | Credit score |
| `Geography` | Country of residence (France, Germany, Spain) |
| `Gender` | Gender |
| `Age` | Age |
| `Tenure` | Years as a customer (fixed-deposit maturation period) |
| `Balance` | Account balance |
| `NumOfProducts` | Number of banking products used |
| `HasCrCard` | Has credit card (1/0) |
| `IsActiveMember` | Active member (1/0) |
| `EstimatedSalary` | Estimated salary |
| **`Exited`** | **Target:** 1 = customer left, 0 = stayed |

**Class balance:** ~**79.6%** stayed, ~**20.4%** exited — significant imbalance favoring the negative class.

### Process

**Data preparation**
* Dropped `RowNumber`, `CustomerId`, and `Surname` (unique or irrelevant identifiers)
* Imputed missing **`Tenure`** values (~9%) with the **median** (no clear pattern in missing rows; distribution is right-skewed)
* **One-hot encoded** `Geography` and `Gender` (`drop_first=True`)
* **StandardScaler** applied to numeric features: `CreditScore`, `Age`, `Tenure`, `Balance`, `NumOfProducts`, `HasCrCard`, `IsActiveMember`, `EstimatedSalary`

**Train / validation / test split**
* **60% / 20% / 20%** via stratified `train_test_split` (`random_state=12345`)

**Model comparison (validation F1, no imbalance handling)**

| Model | Validation F1 |
|-------|---------------|
| Decision tree | 0.50 |
| **Random forest** | **0.61** |
| Logistic regression | 0.31 |

Random forest performed best even before addressing imbalance; logistic regression struggled most.

**Class imbalance strategies** (at least two required)
1. **`class_weight='balanced'`** — helped logistic regression slightly; hurt random forest
2. **Upsampling** — duplicated positive (exited) class 4× to match negatives; **best overall**
3. **Downsampling** — undersampled majority class; modest improvement for logistic regression only

**Hyperparameter tuning**
* `GridSearchCV` on upsampled training data (5-fold CV, scoring=`f1`)
* Best random forest: **`max_depth=10`**, **`n_estimators=43`**
* Threshold tuning (0.00–0.98): default **0.5** gave the highest validation F1

**Final testing**
* Merged validation set back into training (with upsampling), retrained best model, evaluated on **test set**

### Results

**Final model:** `RandomForestClassifier(max_depth=10, n_estimators=43)` trained on upsampled data

| Metric | Test set | Project threshold |
|--------|----------|-------------------|
| **F1** | **0.60** | ≥ 0.59 ✓ |
| **AUC-ROC** | **0.85** | — |

**F1 vs AUC-ROC**
* **AUC-ROC (0.85)** — strong ability to **rank** customers by exit risk; well above a random baseline
* **F1 (0.60)** — moderate hard-classification performance; more sensitive to imbalance and misclassifications on the minority (exited) class

Upsampling improved recall on exited customers but is not a perfect substitute for more real exit data.

### Conclusion

The **random forest** classifier with **upsampling** and tuned hyperparameters is the best approach for Beta Bank churn prediction. It **passes the F1 ≥ 0.59 requirement** on the test set.

The high AUC-ROC suggests the model is useful for **prioritizing high-risk customers** for retention campaigns. The moderate F1 reflects the difficulty of correctly classifying rare exit cases — further gains may come from more exit data or additional behavioral features.

Please see the included Jupyter Notebook (`Beta Bank.ipynb`) for the full analysis.
