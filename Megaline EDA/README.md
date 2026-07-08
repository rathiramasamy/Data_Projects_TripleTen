# Megaline EDA

This is a project I worked on in the TripleTen Data Science program.

### Objective

To analyze Megaline customer behavior on the **Surf** and **Ultimate** prepaid plans, calculate monthly revenue per user, and determine which plan brings in more revenue to help guide advertising budget decisions.

### Background

Megaline is a telecom operator offering two prepaid plans. The analysis uses data on **500 clients** who subscribed in **2018**, including who they are, where they live, which plan they use, and their call, text, and internet usage.

**Surf** — $20/month; 500 minutes, 50 texts, 15 GB included; overage: $0.03/min, $0.03/text, $10/GB.

**Ultimate** — $70/month; 3,000 minutes, 1,000 texts, 30 GB included; overage: $0.01/min, $0.01/text, $7/GB.

**Billing rules:** Each call is rounded up to a full minute. Monthly data usage is summed in megabytes, then rounded up to whole gigabytes (1 GB = 1,024 MB).

The data is provided by TripleTen.

### Data Dictionary

There are five tables in the dataset:

- `megaline_users.csv`: one row per customer
    - `'user_id'`: unique user identifier
    - `'first_name'`: user's first name
    - `'last_name'`: user's last name
    - `'age'`: user's age (years)
    - `'reg_date'`: subscription start date
    - `'churn_date'`: date the user left the service (missing if still active when data was extracted)
    - `'city'`: user's city of residence
    - `'plan'`: calling plan (`surf` or `ultimate`)

- `megaline_plans.csv`: one row per plan
    - `'plan_name'`: plan name
    - `'usd_monthly_pay'`: monthly subscription fee (USD)
    - `'minutes_included'`: monthly minute allowance
    - `'messages_included'`: monthly text allowance
    - `'mb_per_month_included'`: monthly data allowance (MB)
    - `'usd_per_minute'`: price per minute after exceeding limits
    - `'usd_per_message'`: price per text after exceeding limits
    - `'usd_per_gb'`: price per extra GB after exceeding limits

- `megaline_calls.csv`: one row per call
    - `'id'`: unique call identifier
    - `'call_date'`: call date
    - `'duration'`: call duration (minutes)
    - `'user_id'`: user who made the call

- `megaline_messages.csv`: one row per text message
    - `'id'`: unique message identifier
    - `'message_date'`: message date
    - `'user_id'`: user who sent the message

- `megaline_internet.csv`: one row per web session
    - `'id'`: unique session identifier
    - `'session_date'`: session date
    - `'mb_used'`: data used in the session (MB)
    - `'user_id'`: user identifier

### Process

First, I opened the data files and reviewed the general contents of each table.

Next, I preprocessed the data by:
* Setting `plan_name` as the index on the plans table and adding a GB allowance column
* Converting date columns to datetime (`reg_date`, `churn_date`, `call_date`, `message_date`, `session_date`)
* Filling missing `churn_date` values with `2018-12-31` (users still active at year-end)
* Adding an `ny_nj` flag for users in the New York–Newark–Jersey City metro area
* Renaming ID columns for clarity (`call_id`, `message_id`, `session_id`)
* Rounding call durations up to whole minutes per Megaline billing rules
* Aggregating monthly call minutes, message counts, and internet usage (MB → GB, rounded up) per user
* Merging usage tables and filling missing monthly usage with 0
* Building a complete monthly activity table aligned with each user's active subscription months
* Calculating **monthly revenue** per user based on plan limits and overage charges

I then performed exploratory data analysis (EDA), including:
1. Comparing active monthly users on Surf vs. Ultimate over 2018
2. Analyzing monthly call duration, texting, and internet usage by plan
3. Plotting histograms and boxplots of minutes, texts, and data used per month
4. Computing mean, variance, and standard deviation for usage and revenue by plan
5. Comparing average monthly revenue by plan over time

Finally, I tested two hypotheses using **two-sided independent t-tests** (α = 0.05):
1. Average revenue from Surf users differs from average revenue from Ultimate users
2. Average revenue from NY–NJ users differs from average revenue from users in other regions

### Results

**Usage behavior**
- Surf has more than twice as many users as Ultimate in this sample; active user counts grow through 2018 as new subscribers join.
- Call and text usage distributions are right-skewed; many months have low or zero usage.
- Ultimate users rarely approach their large minute/text allowances; Surf users more often exceed limits, especially for **data**.
- A subset of Surf users pays far above the $20 base fee—some more than the $70 Ultimate subscription.

**Revenue**
- Mean monthly revenue: **~$48** (Surf) vs **~$72** (Ultimate).
- Ultimate revenue is tightly clustered near the $70 base fee; Surf revenue has much higher variance (many users at $20, with heavy overage outliers).

**Hypothesis tests**
- **Surf vs. Ultimate revenue:** statistically significant difference (p ≈ 2.2×10⁻⁹⁴). Ultimate users generate higher average monthly revenue in this sample.
- **NY–NJ vs. other regions:** statistically significant difference (p ≈ 0.039). Average revenue differs by region.

**Business takeaway**

Although Ultimate has a higher **average** revenue per user, a large share of Surf customers who exceed package limits—especially on data—generate substantial overage charges. Some Surf users pay more than Ultimate's base price. Combined with Surf's larger customer base and likely higher conversion from a lower advertised price, **increasing Surf marketing** could be profitable if those customers continue to incur overage fees. Further analysis on churn, plan switching, and usage trends beyond 2018 would strengthen this recommendation.

Please see the included Jupyter Notebook (`Megaline EDA.ipynb`) for the full analysis.
