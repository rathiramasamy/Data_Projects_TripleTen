# Ice Videogames — Forecasting 2017 Video Game Sales

This is a project I worked on in the TripleTen Data Science program.

### Objective

To analyze historical video game sales, reviews, genres, and platforms to identify patterns that predict commercial success — and use those insights to plan Ice’s **2017 advertising campaign** (imagining it is December 2016).

### Background

**Ice** is a global online store selling video games. The dataset includes user and expert reviews, genres, platforms (Xbox, PlayStation, PC, Nintendo, etc.), regional sales, and ESRB age ratings.

**Important context:** Data for 2016 may be incomplete. The analysis treats 2013–2016 as the most relevant window for forecasting 2017.

The data is provided by TripleTen (modified from open sources / Kaggle-style game sales data).

### Data Dictionary

Single file: `games.csv` — one row per game (per platform release).

| Column | Description |
|--------|-------------|
| `name` | Game title |
| `platform` | Platform (e.g. PS4, XOne, PC, 3DS) |
| `year_of_release` | Year the game was released |
| `genre` | Game genre (Action, Sports, RPG, etc.) |
| `na_sales` | North America sales (USD millions) |
| `eu_sales` | Europe sales (USD millions) |
| `jp_sales` | Japan sales (USD millions) |
| `other_sales` | Other regions sales (USD millions) |
| `critic_score` | Professional review score (0–100) |
| `user_score` | User review score (0–10) |
| `rating` | ESRB content rating (E, E10+, T, M, AO, etc.) |

A **`total_sales`** column was engineered as the sum of all regional sales.

### Process

**Data preparation**
* Lowercased column names and converted types (`year_of_release` → int, `user_score` → float)
* Dropped 2 rows missing both `name` and `genre` (GEN platform, 1993)
* Handled missing values:
  - **`user_score`:** converted `"tbd"` (to be determined) to NaN; left other missing scores as-is
  - **`critic_score` / `user_score`:** not imputed (subjective scores unique per game)
  - **`rating`:** pre-1994 games → `"not rated"`; remaining missing → `"unknown"` (ESRB did not exist before 1994; many JP-only titles lack ESRB)
  - **`year_of_release`:** imputed via random sampling within each platform’s existing release-year distribution
* Built **`total_sales`** = `na_sales` + `eu_sales` + `jp_sales` + `other_sales`

**Analysis period**
* Explored release trends by year and platform lifecycles (~10-year cycles)
* Focused on **2013–2016** data for 2017 forecasting (excluded legacy/defunct platforms)

**Exploratory analysis**
1. Platform sales over time; identified leaders and declining platforms
2. Boxplots of global sales by platform (high variance, positive skew, outlier-driven hits)
3. PS4 deep dive: scatter plots + Pearson correlation for user vs critic scores vs sales
4. Cross-platform comparison for the same titles (PS4, 3DS, XOne)
5. Genre profitability (Action, Shooter, Sports, RPG, etc.)
6. **Regional profiles (NA, EU, JP):** top 5 platforms and genres per region; ESRB rating vs sales

**Hypothesis tests** (two-sided independent t-tests, α = 0.05)
1. Average **user ratings** on **Xbox One** vs **PC** are the same
2. Average **user ratings** for **Action** vs **Sports** genres are different

### Key findings

**Platforms (2013–2016)**
- Leaders: **PS4**, **PS3**, **XOne**, **3DS**, **X360**
- Declining: **PS3**, **X360**, **Wii**, **DS**, **PSP**
- Potentially profitable for 2017: **PS4** (clear leader), **3DS**, **XOne**
- Sales are **highly skewed** — a few blockbuster titles drive most revenue per platform

**Reviews (PS4)**
- **Critic score** vs sales: moderate positive correlation (~**0.41**)
- **User score** vs sales: very weak correlation (~**−0.03**)
- Early critic reviews may influence purchases more than user scores

**Genres**
- Highest total sales: **Action**, then **Shooter**, **Sports**, **RPG**
- Highest median sales: **Shooter**, then **Sports**

**Regional differences**
| Region | Top platforms | Top genres |
|--------|---------------|------------|
| **NA** | PS4, XOne, X360, PS3, 3DS | Action, Shooter, Sports |
| **EU** | PS4, PS3, XOne, X360, 3DS | Action, Shooter, Sports |
| **JP** | 3DS, PS3, PSV, PS4, WiiU | Role-Playing, Action, Sports |

Japan favors **handhelds** and **RPGs**; NA/EU favor **home consoles** and action/shooter/sports genres.

**ESRB ratings**
- Weak correlation with sales overall; **M-rated** games show higher sales ceilings in NA/EU
- ESRB is a NA system — use cautiously as a proxy in EU/JP (PEGI/CERO regions)

**Hypothesis test results**

| Hypothesis | Result | p-value |
|------------|--------|---------|
| XOne vs PC user ratings equal | **Cannot reject** null (no significant difference) | ~0.164 |
| Action vs Sports user ratings equal | **Reject** null (ratings differ) | ~4.6×10⁻²⁸ |

### 2017 advertising priorities (conclusion)

* **PS4 & XOne** games in **North America and Europe**
* **3DS** games in **Japan**
* Reduce spend on legacy platforms (**X360**, **PS3**, **Wii**, **DS**)
* Promote **Action** globally; **Action / Shooter / Sports** in the West; **RPG** in Japan
* Favor **less restrictive** ESRB ratings for mass-market campaigns (higher sales ceiling)
* Use **critic scores** for early marketing; track actual sales and adjust over time

Please see the included Jupyter Notebook (`Ice Videogames.ipynb`) for the full analysis.
