# OilyGiant — Selecting the Best Region for New Oil Wells

This is a project I worked on in the TripleTen Data Science program.

### Objective

To help **OilyGiant** choose the best region for developing new oil wells by:

1. Building **linear regression** models to predict oil reserve volume at each site
2. Selecting the top 200 wells from a 500-site exploration study
3. Estimating profit and evaluating **risk** using **bootstrapping** (1,000 samples)

**Selection rule:** Keep only regions with **loss risk below 2.5%**; from those, pick the region with the **highest average profit**.

### Background

OilyGiant is deciding where to drill next. Geological exploration data is available for **three regions**. Each region has ~**100,000** oil well sites with three feature measurements and known reserve volumes.

The workflow mirrors OilyGiant’s practice: study **500** candidate sites, develop the best **200**, with a **$100M** budget.

The data is synthetic (contract details and well characteristics are not disclosed). Provided by TripleTen.

### Data Dictionary

Three files: `geo_data_0.csv`, `geo_data_1.csv`, `geo_data_2.csv` (also included locally in this folder).

| Column | Description |
|--------|-------------|
| `id` | Unique oil well identifier (dropped before modeling) |
| `f0`, `f1`, `f2` | Three geological features (specific meaning undisclosed) |
| `product` | Volume of reserves (**thousand barrels**) — target variable |

**100,000 rows per region**, no missing values, no duplicate IDs.

### Business assumptions

| Parameter | Value |
|-----------|-------|
| Wells studied per region | 500 |
| Wells developed | 200 |
| Development budget | **$100 million** |
| Revenue per barrel | **$4.50** |
| Revenue per unit of `product` | **$4,500** (1 unit = 1,000 barrels) |
| Break-even volume per well | **~111.1** thousand barrels |

### Process

**Data preparation**
* Loaded and explored all three regional datasets
* Dropped `id` (unique identifier, not predictive)
* Compared distributions across regions via histograms and summary statistics

**Model training (per region)**
* Features: `f0`, `f1`, `f2` → Target: `product`
* **75% train / 25% validation** split (`random_state=12345`)
* **`LinearRegression`** only (per project requirements)
* Saved validation predictions and actual values; reported mean predicted volume and **RMSE**

**Profit calculation**
* Defined `select_top_wells()` — pick top 200 sites by predicted reserves, sum **actual** volumes
* Defined `calculate_profit()` — `profit = total_volume × $4,500 − $100,000,000`

**Risk analysis (bootstrapping)**
* 1,000 bootstrap samples per region
* Each sample: draw 500 sites (with replacement) from validation set → top 200 by prediction → profit
* Reported **average profit**, **95% confidence interval**, and **risk of losses** (% of samples with negative profit)

### Model performance (validation set)

| Region | Avg predicted volume (k barrels) | RMSE |
|--------|----------------------------------|------|
| 1 | 92.59 | 37.58 |
| 2 | 68.73 | **0.89** |
| 3 | 94.97 | 40.03 |

Region 3 has the highest predicted reserves but the **least reliable** model. Region 2 has the lowest average volume but **by far the lowest RMSE** — predictions are highly accurate.

**Break-even check:** No region’s *average* well volume exceeds ~111.1k barrels (Region 3 closest at 95.0). Profitability depends on selecting the best 200 of 500 sites.

### Profit (top 200 wells, full validation set)

| Region | Total volume (k barrels) | Profit |
|--------|--------------------------|--------|
| 1 | 29,602 | **$33.2M** |
| 2 | 27,589 | $24.2M |
| 3 | 28,245 | $27.1M |

Region 1 looks best on this naive calculation — but bootstrapping tells a different story.

### Bootstrapping results (1,000 samples)

| Region | Avg profit | 95% CI (profit) | Risk of loss |
|--------|------------|-----------------|--------------|
| 1 | $6.01M | $0.13M – $12.31M | **2.0%** |
| **2** | **$6.64M** | **$2.06M – $11.91M** | **0.1%** |
| 3 | $5.97M | $0.02M – $12.46M | **2.5%** ✗ |

**Region 3** is disqualified (risk = 2.5%, not strictly below 2.5%).

Between Regions 1 and 2, **Region 2** has higher average bootstrap profit and far lower loss risk (0.1% vs 2.0%).

### Conclusion

**Recommended region: Region 2 (`geo_data_1.csv`)**

Region 2 offers the best balance of:
- **Highest average bootstrap profit** (~$6.64M)
- **Lowest model error** (RMSE ≈ 0.89) — reliable predictions for site selection
- **Lowest risk of losses** (0.1%) — well below the 2.5% threshold

Region 3’s high reserves are offset by poor model accuracy and elevated risk. Region 1’s headline profit on the full validation set proved inconsistent under bootstrapping.

Please see the included Jupyter Notebook (`OilyGiant.ipynb`) for the full analysis.
