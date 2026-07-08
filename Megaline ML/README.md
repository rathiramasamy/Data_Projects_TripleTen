# Megaline ML — Plan Recommendation Model

This is a project I worked on in the TripleTen Data Science program.

### Objective

To build a classification model that analyzes subscriber behavior and recommends one of Megaline's newer prepaid plans — **Smart** or **Ultra** — for customers still on legacy plans. The model should achieve **at least 75% accuracy** on the test set.

### Background

Mobile carrier **Megaline** offers two newer plans:

- **Smart** (`is_ultra = 0`)
- **Ultra** (`is_ultra = 1`)

The model is trained on behavior data from **3,214 subscribers** who have already switched to Smart or Ultra. Each row represents one user's **monthly** usage. Data preprocessing was completed in an earlier statistical analysis project; this notebook focuses on model building and evaluation.

The data is provided by TripleTen.

### Data Dictionary

Single file: `users_behavior.csv` — one row per user per month.

| Column | Description |
|--------|-------------|
| `calls` | Number of calls |
| `minutes` | Total call duration (minutes) |
| `messages` | Number of text messages |
| `mb_used` | Internet traffic used (MB) |
| `is_ultra` | Target: **1** = Ultra plan, **0** = Smart plan |

**Class balance:** ~70% Smart, ~30% Ultra (985 Ultra users in the full dataset).

### Process

First, I loaded and explored the dataset, confirming feature types and target distribution.

Next, I split the data into **training (60%)**, **validation (20%)**, and **test (20%)** sets using stratified `train_test_split` to preserve class balance.

I then compared three classifiers, tuning hyperparameters on the training set and evaluating on the validation set:

1. **Decision tree** — tested `max_depth` from 1 to 10; best depth = **10**
2. **Random forest** — `GridSearchCV` over `n_estimators` (1–20) and `max_depth` (1–10), 5-fold CV
3. **Logistic regression** — baseline with `solver='liblinear'`

After selecting the best model, I **merged the validation set back into training**, retrained the final model, and evaluated on the held-out **test set**.

Finally, I ran a **sanity check** (weighted random predictions at 70% Smart / 30% Ultra) and reviewed the **confusion matrix**.

### Model comparison (validation set)

| Model | Validation accuracy | Notes |
|-------|---------------------|--------|
| Decision tree | **76.2%** | Moderate overfitting (train >> val) |
| **Random forest** | **78.2%** | Best performer; slight overfitting |
| Logistic regression | **70.6%** | Below 75% threshold |

**Best random forest parameters:** `n_estimators=15`, `max_depth=9`

### Results

**Final model:** `RandomForestClassifier(n_estimators=15, max_depth=9)`

| Set | Accuracy |
|-----|----------|
| Training (train + val) | **88.4%** |
| **Test** | **81.6%** |

Test accuracy **exceeds the 75% project threshold**. The small drop from training to test suggests reasonable generalization.

**Sanity check:** Weighted random guessing achieved **~55.8%** accuracy vs **81.6%** for the model — the model performs substantially better than chance.

**Confusion matrix (test set):** `[[425, 21], [97, 100]]`
- Strong at identifying **Smart** users
- Misses more **Ultra** users (lower recall for Ultra)
- When the model predicts **Ultra**, predictions are typically correct (high precision)

### Conclusion

The **random forest** classifier is the best choice for recommending Smart vs. Ultra based on monthly usage (calls, minutes, messages, data). It meets the accuracy requirement and beats a random baseline by a wide margin. Ultra recommendations from the model are likely reliable when the model assigns that class, though some Ultra users may be missed.

With more data or feature engineering (e.g., additional months, derived usage ratios), performance could likely improve further.

Please see the included Jupyter Notebook (`Megaline ML.ipynb`) for the full analysis.
