---
title: "DataScientist Core"
date: 2026-04-22 21:33:07 
author: shajeebhameed
layout: page
slug: datascientist-core
status: publish
---

## Data Analyst / Data Science — Comprehensive Interview Preparation Guide  
  
Exl Learn Links - [claude](<https://claude.ai/share/8795dc57-e867-4141-bf2f-3b334c3a232f>)

* * *

## Synthetic Dataset Reference

All examples throughout this guide use a consistent set of linked tables — mirroring datasets from Google Street View, J&J Global Services, AXA Insurance, and Travelers Insurance projects.

Table| Key Columns  
---|---  
`sales_data`| `customer_id, product, amount, region, sale_date, sales_rep`  
`customers`| `customer_id, segment, acquisition_channel, signup_date, email, region`  
`marketing_events`| `campaign_id, customer_id, event_type, event_date`  
`pipeline_logs`| `run_id, table_name, check_name, status, rows_loaded, run_date`  
  
* * *

## AREA 1 — Core Technical Methods

* * *

## Topic 1 — Data Cleaning & Handling Missing Values

**Why it matters:** Raw data from enterprise sources (Salesforce, Oracle, legacy claims systems) is almost always dirty. Cleaning is the most time-consuming and critical step before any analysis.

* * *

### Interview Question 1 (Behavioural)

> _"Tell me about a time you discovered data quality issues that materially affected an analysis. What did you do?"_

|   
---|---  
**Situation**|  At J&J Global Services, I was building a cost-savings analysis across claims and underwriting data pulled from Oracle and SQL Server. We were combining 4 legacy sources into a new BigQuery model.  
**Task**|  I needed to identify and resolve data quality issues before the analysis could be used by actuarial teams and executives as a source of truth.  
**Action**|  I ran a full data profiling pass: checked NULL rates per column, identified duplicate claim IDs, found date fields stored as VARCHAR, and detected outlier amounts that were clearly data entry errors. I documented 200+ transformation rules in the STTM and applied them in the ETL pipeline.  
**Result**|  The cleaned dataset became the single source of truth for 150+ analysts and executives. The analysis subsequently identified $2M+ in potential cost savings — savings that would have been invisible or misleading in the dirty source data.  
  
* * *

### Interview Question 2 (Conceptual)

> _"What is the difference between dropping nulls and imputing them? When would you choose each?"_

  * **Drop nulls** when the column is a primary key, a join key, or when the NULL rate is under ~1% and the mechanism is random (MCAR — Missing Completely At Random). Dropping is safe when missingness won't bias your results.
  * **Impute with median/mode** when data is Missing At Random (MAR) and you can't afford to lose rows — e.g., imputing `amount` with the product-group median when the NULL rate is 8%.
  * **Impute with a constant** (e.g., `'Unknown'` for region) when NULL itself is an informative category you want to preserve in aggregations.
  * **Never** impute a target variable or any column that is NULL because the event didn't happen (e.g., `claim_date` is NULL because no claim was made — that's meaningful, not missing).



* * *

### SQL Implementation
    
    
    -- Synthetic table: sales_data
    -- (customer_id, product, amount, region, sale_date, sales_rep)
    
    -- 1. Audit NULLs across key columns
    SELECT
      COUNT(*)                           AS total_rows,
      COUNT(*) - COUNT(customer_id)      AS null_customer_id,
      COUNT(*) - COUNT(amount)           AS null_amount,
      COUNT(*) - COUNT(region)           AS null_region,
      COUNT(*) - COUNT(sale_date)        AS null_sale_date
    FROM sales_data;
    
    -- 2. Replace NULL region with 'Unknown', cap extreme amounts, enforce types
    SELECT
      customer_id,
      product,
      COALESCE(region, 'Unknown')          AS region,
      CASE
        WHEN amount < 0     THEN 0
        WHEN amount > 50000 THEN 50000     -- business-rule cap
        ELSE amount
      END                                  AS amount_clean,
      CAST(sale_date AS DATE)              AS sale_date
    FROM sales_data
    WHERE customer_id IS NOT NULL          -- drop unresolvable rows
      AND amount IS NOT NULL;
    
    -- 3. Check for duplicates
    SELECT customer_id, sale_date, COUNT(*) AS n
    FROM sales_data
    GROUP BY customer_id, sale_date
    HAVING COUNT(*) > 1;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    import numpy as np
    
    data = {
      'customer_id': [1, 2, None, 4, 5],
      'product':     ['Widget A', 'Widget B', 'Widget A', 'Widget C', 'Widget B'],
      'amount':      [1200, -50, 800, 99000, None],
      'region':      ['EMEA', None, 'APAC', 'EMEA', 'NA'],
      'sale_date':   ['2024-01-15', '2024-01-18', '2024-01-20', '2024-01-22', '2024-01-25']
    }
    df = pd.DataFrame(data)
    
    # Audit: NULL rates as percentages
    print(df.isnull().sum() / len(df) * 100)
    
    # Clean
    df = df[df['customer_id'].notna()]                    # drop unresolvable
    df['region']    = df['region'].fillna('Unknown')      # impute with constant
    df['amount']    = df['amount'].fillna(df['amount'].median())  # median imputation
    df['amount']    = df['amount'].clip(lower=0, upper=50000)     # cap outliers
    df['sale_date'] = pd.to_datetime(df['sale_date'])     # enforce dtype
    
    # Deduplication
    df = df.drop_duplicates(subset=['customer_id', 'sale_date'], keep='last')
    
    print(df)
    
    
    

> 💡 **Key Insight:** Interviewers want to see that you **audit BEFORE you clean** — NULL rates, type mismatches, and business-rule violations are all different problems requiring different fixes. Never just drop NULLs blindly.

* * *

## Topic 2 — Exploratory Data Analysis (EDA)

**Why it matters:** EDA is how you understand the shape, distribution, and relationships in data before modelling or reporting. Senior analysts are expected to spot patterns and anomalies quickly.

* * *

### Interview Question (Conceptual + Technical)

> _"Walk me through how you would approach EDA on a new sales dataset you've never seen before."_

|   
---|---  
**Situation**|  When onboarding the consolidated BigQuery model at J&J Global Services, I received a merged claims + underwriting dataset with 50+ columns and ~5M rows I had never worked with before.  
**Task**|  I needed to produce an EDA summary that would inform the analytical direction and surface any residual data quality issues before handing the model to 150+ downstream users.  
**Action**|  I applied a structured EDA framework: (1) schema inspection, (2) statistical summaries, (3) distribution analysis by category, (4) correlation analysis, (5) anomaly flagging. I specifically looked at amount distributions by region and product, claim frequency over time, and correlation between claim amount and underwriting score.  
**Result**|  The EDA revealed a bi-modal distribution in claim amounts (indicating two distinct customer segments) and strong regional skew being averaged away in prior reporting. These findings changed the segmentation strategy for the $2M savings analysis.  
  
* * *

### SQL Implementation
    
    
    -- Statistical summary per region
    SELECT
      region,
      COUNT(*)                                           AS n_transactions,
      ROUND(AVG(amount), 2)                              AS mean_amount,
      ROUND(STDDEV(amount), 2)                           AS stddev_amount,
      MIN(amount)                                        AS min_amount,
      MAX(amount)                                        AS max_amount,
      PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount) AS median_amount,
      PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) AS p25,
      PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) AS p75
    FROM sales_data
    GROUP BY region
    ORDER BY mean_amount DESC;
    
    -- Distribution of amounts by bucket
    SELECT
      CASE
        WHEN amount <  500  THEN '< 500'
        WHEN amount <  1500 THEN '500–1500'
        WHEN amount <  5000 THEN '1500–5000'
        ELSE '5000+'
      END AS amount_bucket,
      COUNT(*) AS n,
      ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS pct
    FROM sales_data
    GROUP BY 1
    ORDER BY MIN(amount);
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    import numpy as np
    
    # Build the consistent synthetic dataset used throughout this guide
    np.random.seed(42)
    n = 500
    
    df = pd.DataFrame({
      'customer_id': np.random.randint(1000, 2000, n),
      'product':     np.random.choice(['Widget A', 'Widget B', 'Widget C'], n),
      'amount':      np.random.exponential(scale=1500, size=n).round(2),
      'region':      np.random.choice(['EMEA', 'NA', 'APAC'], n, p=[0.5, 0.3, 0.2]),
      'sale_date':   pd.date_range('2024-01-01', periods=n, freq='D')[:n],
      'sales_rep':   np.random.choice(['Alice', 'Bob', 'Carol', 'Dave'], n)
    })
    
    # Step 1: Schema inspection
    print(df.dtypes)
    print(df.shape)
    
    # Step 2: Full descriptive stats including 95th percentile
    print(df.describe(percentiles=[.25, .5, .75, .95]))
    
    # Step 3: By-group EDA
    eda = df.groupby('region')['amount'].agg(
      n='count',
      mean='mean',
      median='median',
      std='std',
      min='min',
      max='max',
      p25=lambda x: x.quantile(0.25),
      p75=lambda x: x.quantile(0.75)
    ).round(2)
    print(eda)
    
    # Step 4: Correlation matrix (numeric columns only)
    print(df.select_dtypes(include='number').corr())
    
    # Step 5: Value counts for categoricals
    for col in ['product', 'region', 'sales_rep']:
      print(df[col].value_counts(normalize=True).round(3))
    
    
    

> 💡 **Key Insight:** Interviewers test whether you go beyond `.describe()` — they want percentile analysis, group-level distributions, and correlation checks that reveal business-relevant patterns, not just averages.

* * *

## Topic 3 — Aggregations, Grouping & Window Functions

**Why it matters:** Window functions are the single most tested advanced SQL topic in data analyst interviews. Ranking, running totals, LAG/LEAD comparisons, and moving averages are core to real analytics work.

* * *

### Interview Question (Technical)

> _"Write a query to calculate each sales rep's monthly revenue, their rank within the region, and the revenue growth vs. the prior month."_

|   
---|---  
**Situation**|  At Google Street View, leadership wanted a monthly performance dashboard for R&D teams across global regions — showing not just raw metrics but relative ranking and trend direction.  
**Task**|  I needed to build a reusable query pattern in BigQuery that could power Looker Studio dashboard tiles showing rep-level ranking and month-over-month growth.  
**Action**|  I used window functions — `RANK() OVER PARTITION BY` for regional ranking, `SUM() OVER` for running totals, and `LAG()` to calculate MoM change. The query was wrapped in a CTE for readability and reuse.  
**Result**|  The pattern was adopted as a standard template across 5 dashboard tiles, eliminating 100+ manual hours/month of Excel-based reporting.  
  
* * *

### SQL Implementation
    
    
    WITH monthly_sales AS (
      SELECT
        sales_rep,
        region,
        DATE_TRUNC('month', sale_date)   AS month,
        SUM(amount)                      AS monthly_revenue
      FROM sales_data
      GROUP BY 1, 2, 3
    ),
    ranked AS (
      SELECT
        sales_rep,
        region,
        month,
        monthly_revenue,
        RANK() OVER (
          PARTITION BY region, month
          ORDER BY monthly_revenue DESC
        )                                AS region_rank,
        LAG(monthly_revenue) OVER (
          PARTITION BY sales_rep
          ORDER BY month
        )                                AS prev_month_revenue,
        SUM(monthly_revenue) OVER (
          PARTITION BY sales_rep
          ORDER BY month
          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        )                                AS ytd_revenue
      FROM monthly_sales
    )
    SELECT
      *,
      ROUND(
        100.0 * (monthly_revenue - prev_month_revenue)
        / NULLIF(prev_month_revenue, 0),
      1)                                 AS mom_pct_change
    FROM ranked
    ORDER BY region, month, region_rank;
    
    
    

* * *

### Pandas Implementation
    
    
    # Uses df from Topic 2
    df['month'] = df['sale_date'].dt.to_period('M')
    
    monthly = (
      df.groupby(['sales_rep', 'region', 'month'])['amount']
      .sum()
      .reset_index()
      .rename(columns={'amount': 'monthly_revenue'})
      .sort_values(['sales_rep', 'month'])
    )
    
    # MoM change — equivalent to LAG()
    monthly['prev_rev'] = monthly.groupby('sales_rep')['monthly_revenue'].shift(1)
    monthly['mom_pct']  = (
      (monthly['monthly_revenue'] - monthly['prev_rev'])
      / monthly['prev_rev'] * 100
    ).round(1)
    
    # Rank within region per month — equivalent to RANK() OVER PARTITION BY
    monthly['region_rank'] = (
      monthly.groupby(['region', 'month'])['monthly_revenue']
      .rank(method='min', ascending=False)
      .astype(int)
    )
    
    # YTD running total — equivalent to SUM() OVER UNBOUNDED PRECEDING
    monthly['ytd_revenue'] = monthly.groupby('sales_rep')['monthly_revenue'].cumsum()
    
    print(monthly.head(15))
    
    
    

> 💡 **Key Insight:** `LAG/LEAD` and `RANK() OVER PARTITION BY` are the two most frequently tested window function patterns. Be able to explain `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — it comes up as a follow-up in nearly every interview.

* * *

## Topic 4 — Joins (Inner, Left, Right, Full, Self, Cross)

**Why it matters:** Every real-world analysis requires combining datasets. Choosing the wrong join type silently corrupts results — often without any error message.

* * *

### Interview Question (Technical)

> _"What is the difference between INNER and LEFT JOIN? When would a LEFT JOIN give you misleading results?"_

|   
---|---  
**Situation**|  At J&J Global Services, I was consolidating 4 legacy sources. Marketing data had customers who never made purchases; sales data had transactions with no matching customer record (orphaned due to migration errors).  
**Task**|  I needed to build a 360-degree customer view that preserved ALL customers including those with no purchases, while flagging orphaned transactions for data quality remediation.  
**Action**|  I used LEFT JOIN from customers to sales to retain all customers, then a second LEFT JOIN from sales to customers to surface orphaned transactions. I used FULL OUTER JOIN with `indicator=True` to get a complete diagnostic view and compared row counts to validate.  
**Result**|  Found 3,200 orphaned transactions and 8,500 customers with zero purchases — both groups were invisible in prior INNER JOIN-based reports, distorting conversion rate calculations by 12%.  
  
* * *

### SQL Implementation
    
    
    -- customers(customer_id, segment, acquisition_channel, signup_date)
    -- sales_data(customer_id, product, amount, region, sale_date)
    
    -- LEFT JOIN: all customers, even those with no purchases
    SELECT
      c.customer_id,
      c.segment,
      c.acquisition_channel,
      COALESCE(SUM(s.amount), 0)  AS total_spend,
      COUNT(s.customer_id)        AS purchase_count
    FROM customers c
    LEFT JOIN sales_data s ON c.customer_id = s.customer_id
    GROUP BY c.customer_id, c.segment, c.acquisition_channel;
    
    -- FULL OUTER JOIN: find orphans on both sides
    SELECT
      COALESCE(c.customer_id, s.customer_id) AS customer_id,
      CASE
        WHEN c.customer_id IS NULL THEN 'Orphan Sale'
        WHEN s.customer_id IS NULL THEN 'No Purchase'
        ELSE 'Matched'
      END                                     AS status
    FROM customers c
    FULL OUTER JOIN sales_data s ON c.customer_id = s.customer_id
    WHERE c.customer_id IS NULL OR s.customer_id IS NULL;
    
    -- SELF JOIN: find same-day purchases by same customer (duplicate/fraud check)
    SELECT
      a.customer_id,
      a.sale_date,
      a.amount        AS amount_1,
      b.amount        AS amount_2
    FROM sales_data a
    JOIN sales_data b
      ON  a.customer_id = b.customer_id
      AND a.sale_date   = b.sale_date
      AND a.rowid < b.rowid;  -- avoids matching a row with itself
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    customers = pd.DataFrame({
      'customer_id':        [101, 102, 103, 104, 105],
      'segment':            ['Enterprise', 'SMB', 'Enterprise', 'Consumer', 'SMB'],
      'acquisition_channel': ['Paid', 'Organic', 'Referral', 'Paid', 'Organic']
    })
    
    sales = pd.DataFrame({
      'customer_id': [101, 102, 101, 103, 999],   # 999 is an orphan
      'product':     ['Widget A', 'Widget B', 'Widget C', 'Widget A', 'Widget B'],
      'amount':      [1200, 800, 500, 2100, 400]
    })
    
    # LEFT JOIN — all customers, NaN where no sale exists
    left  = customers.merge(sales, on='customer_id', how='left')
    print('LEFT JOIN rows:', len(left))    # 5+ customers preserved
    
    # INNER JOIN — only matched records
    inner = customers.merge(sales, on='customer_id', how='inner')
    print('INNER JOIN rows:', len(inner))  # orphan customer_id=999 dropped
    
    # FULL OUTER — find all mismatches using indicator
    full = customers.merge(sales, on='customer_id', how='outer', indicator=True)
    print(full[full['_merge'] != 'both'][['customer_id', '_merge']])
    
    
    

> 💡 **Key Insight:** INNER JOIN silently drops unmatched rows — always validate row counts before and after any join. The `indicator=True` parameter in Pandas is your best diagnostic tool for join quality.

* * *

## Topic 5 — Date/Time & String Operations

**Why it matters:** Dates and strings are the messiest data types in real-world datasets — stored inconsistently across systems. Mastery here separates senior analysts from juniors.

* * *

### Interview Question (Technical)

> _"How would you calculate the number of days between a customer's signup and their first purchase, and flag customers who purchased within 7 days?"_

|   
---|---  
**Situation**|  At AXA Insurance, I was analysing real-time claims data to understand time-to-first-claim patterns after policy activation — a key fraud indicator used by the claims team.  
**Task**|  I needed to calculate time-to-first-event metrics and flag anomalously fast claimants for the fraud detection team's review queue.  
**Action**|  I computed DATEDIFF between `policy_start_date` and `first_claim_date` using window functions and DATE_TRUNC. I then bucketed claimants into cohorts: same-day, 1–7 days, 8–30 days, 30+ days.  
**Result**|  The analysis fed directly into the fraud detection model. Combined with CDC pipelines, this reduced data latency from 24 hours to under 15 minutes, giving the fraud team near-real-time flagging capability.  
  
* * *

### SQL Implementation
    
    
    -- Date arithmetic: days from signup to first purchase
    WITH first_purchase AS (
      SELECT
        customer_id,
        MIN(sale_date) AS first_purchase_date
      FROM sales_data
      GROUP BY customer_id
    )
    SELECT
      c.customer_id,
      c.signup_date,
      fp.first_purchase_date,
      DATEDIFF('day', c.signup_date, fp.first_purchase_date) AS days_to_first_purchase,
      CASE
        WHEN DATEDIFF('day', c.signup_date, fp.first_purchase_date) = 0 THEN 'Same Day'
        WHEN DATEDIFF('day', c.signup_date, fp.first_purchase_date) <= 7 THEN 'Early Buyer'
        WHEN DATEDIFF('day', c.signup_date, fp.first_purchase_date) <= 30 THEN '8–30 Days'
        ELSE '30+ Days'
      END                                                    AS buyer_cohort,
      DATE_TRUNC('month', c.signup_date)                     AS signup_month,
      EXTRACT(DOW FROM c.signup_date)                        AS signup_day_of_week
    FROM customers c
    LEFT JOIN first_purchase fp ON c.customer_id = fp.customer_id;
    
    -- String operations: cleaning and extracting from text fields
    SELECT
      customer_id,
      email,
      SPLIT_PART(email, '@', 2)            AS email_domain,
      UPPER(TRIM(region))                  AS region_clean,
      REGEXP_REPLACE(phone, '[^0-9]', '')  AS phone_digits_only,
      LEFT(segment, 3)                     AS segment_code
    FROM customers;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    customers = pd.DataFrame({
      'customer_id':  [101, 102, 103, 104, 105],
      'signup_date':  pd.to_datetime(['2024-01-01', '2024-01-05', '2024-01-10',
                                      '2024-01-15', '2024-01-20']),
      'email':        ['a@gmail.com', 'b@corp.ie', 'c@yahoo.com', 'd@corp.ie', 'e@gmail.com'],
      'region':       [' emea ', 'na', 'apac', ' EMEA ', 'na']
    })
    
    sales = pd.DataFrame({
      'customer_id': [101, 102, 103, 101, 104],
      'sale_date':   pd.to_datetime(['2024-01-03', '2024-01-05', '2024-01-25',
                                     '2024-02-01', '2024-01-16'])
    })
    
    # First purchase date per customer
    first_purchase = (
      sales.groupby('customer_id')['sale_date']
      .min()
      .reset_index()
      .rename(columns={'sale_date': 'first_purchase_date'})
    )
    
    df = customers.merge(first_purchase, on='customer_id', how='left')
    
    # Date arithmetic
    df['days_to_first'] = (df['first_purchase_date'] - df['signup_date']).dt.days
    df['buyer_cohort']  = pd.cut(
      df['days_to_first'],
      bins=[-1, 0, 7, 30, float('inf')],
      labels=['Same Day', 'Early Buyer', '8–30 Days', '30+ Days']
    )
    df['signup_month'] = df['signup_date'].dt.to_period('M')
    df['signup_dow']   = df['signup_date'].dt.day_name()
    
    # String operations
    df['email_domain'] = df['email'].str.split('@').str[1]
    df['region_clean'] = df['region'].str.strip().str.upper()
    
    print(df[['customer_id', 'days_to_first', 'buyer_cohort', 'email_domain', 'region_clean']])
    
    
    

> 💡 **Key Insight:** Always convert date columns explicitly with `pd.to_datetime()` or `CAST` — implicit string-to-date coercion is the #1 source of silent calculation errors in production pipelines. Never assume dates are stored correctly.

* * *

## Topic 6 — CTEs, Subqueries & Pandas Method Chaining

**Why it matters:** CTEs (Common Table Expressions) are the SQL equivalent of modular functions — they make complex logic readable, testable, and reusable. Pandas method chaining achieves the same goal.

* * *

### Interview Question

> _"What are the advantages of CTEs over subqueries? When would you choose one over the other?"_

|   
---|---  
**Situation**|  At Travelers Insurance, I built a Snowflake data warehouse using Bronze-Silver-Gold lakehouse architecture. Complex actuarial queries involved 5–7 levels of aggregation and transformation.  
**Task**|  I needed query patterns that actuarial analysts (not engineers) could read, validate, and modify without breaking them — the queries were business-critical for loss reserving.  
**Action**|  I refactored all complex nested subqueries into named CTEs. Each CTE represented one business concept: `raw_claims` → `cleaned_claims` → `monthly_aggregates` → `ranked_by_severity`. This mirrored the Gold layer transformations exactly.  
**Result**|  Query readability reviews went from hour-long sessions to 10 minutes. The patterns became the standard template for all sub-second actuarial queries in Power BI Direct Lake mode.  
  
* * *

### SQL Implementation — CTE vs Subquery
    
    
    -- ❌ Subquery approach — hard to read, hard to debug, impossible to unit-test
    SELECT region, avg_amount
    FROM (
      SELECT region, AVG(amount) AS avg_amount
      FROM (SELECT * FROM sales_data WHERE amount > 0) clean
      GROUP BY region
    ) agg
    WHERE avg_amount > 1000;
    
    -- ✅ CTE approach — each step is named, readable, and independently testable
    WITH
    clean_sales AS (
      -- Step 1: Remove invalid records
      SELECT * FROM sales_data WHERE amount > 0
    ),
    regional_avg AS (
      -- Step 2: Aggregate by region
      SELECT
        region,
        AVG(amount)   AS avg_amount,
        COUNT(*)      AS n_transactions,
        MAX(amount)   AS max_amount
      FROM clean_sales
      GROUP BY region
    ),
    high_value_regions AS (
      -- Step 3: Filter to high-value regions only
      SELECT * FROM regional_avg WHERE avg_amount > 1000
    )
    -- Final: join enrichment data
    SELECT
      h.region,
      h.avg_amount,
      h.n_transactions,
      COUNT(DISTINCT c.customer_id) AS n_customers
    FROM high_value_regions h
    JOIN customers c ON c.region = h.region
    GROUP BY h.region, h.avg_amount, h.n_transactions
    ORDER BY h.avg_amount DESC;
    
    
    

* * *

### Pandas Method Chaining (Equivalent Pattern)
    
    
    # Each method call in the chain = one named CTE step
    result = (
      df                                        # Raw data
      .query('amount > 0')                      # clean_sales CTE
      .groupby('region')                        # regional_avg CTE
      .agg(
        avg_amount    = ('amount', 'mean'),
        n_transactions = ('amount', 'count'),
        max_amount    = ('amount', 'max')
      )
      .reset_index()
      .query('avg_amount > 1000')               # high_value_regions CTE
      .merge(                                   # JOIN step
        customers[['customer_id', 'region', 'segment']],
        on='region', how='left'
      )
      .groupby(['region', 'segment'])           # final aggregation
      .agg(
        avg_amount    = ('avg_amount', 'first'),
        n_customers   = ('customer_id', 'nunique')
      )
      .reset_index()
      .sort_values('avg_amount', ascending=False)
    )
    print(result)
    
    
    

> 💡 **Key Insight:** CTEs don't inherently improve SQL performance (most optimisers inline them), but they dramatically improve maintainability. In interviews, emphasise that each CTE represents **one business concept** — that's the real value, not performance.

* * *

## Topic 7 — Data Transformation & Reshaping (Pivot, Melt, Stack)

**Why it matters:** Dashboard and reporting queries constantly require rotating rows to columns (pivot) and vice versa (unpivot/melt). These transformations are essential for executive-ready matrix formats.

* * *

### Interview Question

> _"How would you reshape a long-format sales dataset into a wide-format report showing each product as a column?"_

|   
---|---  
**Situation**|  At Google Street View, leadership needed a monthly executive dashboard in Looker Studio showing R&D metrics per region in a matrix format — regions as rows, months as columns.  
**Task**|  The raw data was in long format (one row per region/month). The BI tool required wide format for the matrix visualisation.  
**Action**|  I used conditional aggregation (CASE WHEN + SUM) in BigQuery for the pivot — preferred over native PIVOT syntax for cross-platform portability. In Python I used `pd.pivot_table()` for the transformation layer before loading.  
**Result**|  The reshaped query powered a real-time dashboard that eliminated 100+ manual hours/month of Excel pivoting for research and compliance teams.  
  
* * *

### SQL Implementation
    
    
    -- Conditional aggregation (portable across BigQuery, Snowflake, SQL Server, PostgreSQL)
    SELECT
      region,
      SUM(CASE WHEN product = 'Widget A' THEN amount ELSE 0 END) AS widget_a_revenue,
      SUM(CASE WHEN product = 'Widget B' THEN amount ELSE 0 END) AS widget_b_revenue,
      SUM(CASE WHEN product = 'Widget C' THEN amount ELSE 0 END) AS widget_c_revenue,
      SUM(amount)                                                 AS total_revenue
    FROM sales_data
    GROUP BY region;
    
    -- Monthly pivot: months as columns (BigQuery-style with CASE WHEN)
    SELECT
      region,
      SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-01-01' THEN amount ELSE 0 END) AS jan_2024,
      SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-02-01' THEN amount ELSE 0 END) AS feb_2024,
      SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-03-01' THEN amount ELSE 0 END) AS mar_2024
    FROM sales_data
    GROUP BY region;
    
    -- UNPIVOT: wide back to long (SQL Server / Snowflake)
    SELECT region, product, revenue
    FROM wide_sales_table
    UNPIVOT (
      revenue FOR product IN (
        widget_a_revenue AS 'Widget A',
        widget_b_revenue AS 'Widget B',
        widget_c_revenue AS 'Widget C'
      )
    ) AS unpivoted;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    # PIVOT: long to wide
    pivot = pd.pivot_table(
      df,
      values='amount',
      index='region',
      columns='product',
      aggfunc='sum',
      fill_value=0
    )
    pivot['total'] = pivot.sum(axis=1)
    print("Wide format:\n", pivot)
    
    # MELT: wide back to long (equivalent to UNPIVOT)
    wide = pivot.drop(columns='total').reset_index()
    long = wide.melt(
      id_vars='region',
      var_name='product',
      value_name='revenue'
    )
    print("\nLong format:\n", long.head(10))
    
    # CROSSTAB: count-based pivot
    ct = pd.crosstab(df['region'], df['product'], values=df['amount'], aggfunc='sum')
    print("\nCrosstab:\n", ct)
    
    # STACK / UNSTACK (hierarchical index approach)
    stacked   = pivot.stack()    # wide → long
    unstacked = stacked.unstack() # long → wide
    
    
    

> 💡 **Key Insight:** Conditional aggregation (`CASE WHEN` \+ `SUM`) is safer than native `PIVOT` because it works across all SQL dialects — BigQuery, Snowflake, SQL Server, and PostgreSQL — without vendor-specific syntax limitations.

* * *

## Topic 8 — Performance Optimisation (Indexing & Vectorisation)

**Why it matters:** At enterprise scale (20M+ rows, high-frequency transactions), unoptimised queries kill dashboards and pipelines. Performance tuning is a key differentiator for senior roles.

* * *

### Interview Question

> _"How did you optimise query performance on a large dataset? What techniques did you apply?"_

|   
---|---  
**Situation**|  At Refinitiv/LSEG, I worked on a high-frequency FX trading platform with millisecond-level data retrieval requirements on SQL Server, handling millions of trade records daily.  
**Task**|  Reconciliation queries were taking 45+ seconds, causing downstream trading reports to lag. I needed to bring retrieval to sub-second levels.  
**Action**|  I analysed execution plans, added composite indexes on the most-filtered columns (`trade_date`, `instrument_id`), implemented table partitioning by date, rewrote correlated subqueries as JOINs, and used covering indexes. In Python I replaced row-wise `apply()` loops with vectorised NumPy operations.  
**Result**|  Query performance improved from 45 seconds to under 200 milliseconds — a 225× improvement. Reconciliation issues dropped by 85%.  
  
* * *

### SQL Implementation
    
    
    -- ❌ Slow: function applied to indexed column defeats the index
    SELECT * FROM sales_data
    WHERE YEAR(sale_date) = 2024;
    
    -- ✅ Fast: use sargable range scan — index is used
    SELECT * FROM sales_data
    WHERE sale_date >= '2024-01-01'
      AND sale_date <  '2025-01-01';
    
    -- ❌ Slow: correlated subquery (runs once per row)
    SELECT customer_id, amount
    FROM sales_data s
    WHERE amount > (SELECT AVG(amount) FROM sales_data WHERE region = s.region);
    
    -- ✅ Fast: pre-aggregate with CTE, then join once
    WITH regional_avg AS (
      SELECT region, AVG(amount) AS avg_amount
      FROM sales_data GROUP BY region
    )
    SELECT s.customer_id, s.amount
    FROM sales_data s
    JOIN regional_avg r ON s.region = r.region
    WHERE s.amount > r.avg_amount;
    
    -- Composite index: most selective / most-filtered columns first
    CREATE INDEX idx_sales_region_date
    ON sales_data (region, sale_date, customer_id);
    
    -- Covering index: include projected columns — avoids table lookup entirely
    CREATE INDEX idx_sales_covering
    ON sales_data (region, sale_date)
    INCLUDE (amount, product);   -- SQL Server / PostgreSQL syntax
    
    -- BigQuery / Snowflake: partitioning + clustering
    CREATE TABLE sales_data
    PARTITION BY DATE_TRUNC(sale_date, MONTH)
    CLUSTER BY region, customer_id AS (...);
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    import numpy as np
    import time
    
    n = 1_000_000
    df_large = pd.DataFrame({
      'amount': np.random.uniform(100, 5000, n),
      'region': np.random.choice(['EMEA', 'NA', 'APAC'], n)
    })
    
    # ❌ Slow: row-wise apply() — Python for-loop under the hood
    t0 = time.time()
    df_large['tier_slow'] = df_large['amount'].apply(
      lambda x: 'High' if x > 3000 else ('Mid' if x > 1500 else 'Low')
    )
    print(f'apply():   {time.time() - t0:.2f}s')
    
    # ✅ Fast: vectorised np.select — operates on entire array at once
    t0 = time.time()
    conditions = [df_large['amount'] > 3000, df_large['amount'] > 1500]
    choices    = ['High', 'Mid']
    df_large['tier_fast'] = np.select(conditions, choices, default='Low')
    print(f'np.select: {time.time() - t0:.2f}s')  # typically 10–50× faster
    
    # Memory optimisation: categorical dtype for low-cardinality string columns
    df_large['region'] = df_large['region'].astype('category')
    print(df_large['region'].dtype)  # category — 4× memory saving vs object
    
    # Use query() with index for fast filtering
    df_large = df_large.set_index('region')
    emea = df_large.loc['EMEA']    # index lookup — much faster than boolean mask on large df
    
    
    

> 💡 **Key Insight:** Know the difference between `Seq Scan` and `Index Scan` in a query EXPLAIN plan. In Pandas, `apply()` on large DataFrames is a red flag in code reviews — always reach for `np.where`, `np.select`, or vectorised string methods (`.str.contains()`) first.

* * *

## Topic 9 — Feature Engineering & Statistical Summaries

**Why it matters:** Feature engineering transforms raw transactional data into the signals that power predictive models, customer segmentation, and executive scorecards.

* * *

### Interview Question

> _"What features would you engineer from a raw sales transaction table to predict customer churn?"_

|   
---|---  
**Situation**|  At J&J Global Services, I was tasked with identifying leading indicators of customer inactivity (churn proxy) before building a formal predictive model.  
**Task**|  I needed to transform raw transactional data into engineered features meaningful to the actuarial and data science team — features with clear business intuition.  
**Action**|  I engineered: recency (days since last transaction), frequency (transaction count in last 90 days), monetary value (total spend), spend volatility (std dev), average inter-purchase gap, product diversity score, and a 'declining trend' flag comparing 30-day vs 90-day rolling averages.  
**Result**|  The feature set validated the approach: recency alone had 0.71 correlation with the churn label. The declining trend flag captured an additional 14% of at-risk customers not identified by recency alone.  
  
* * *

### SQL Implementation
    
    
    WITH customer_features AS (
      SELECT
        customer_id,
        COUNT(*)                                            AS frequency,
        SUM(amount)                                         AS total_spend,
        AVG(amount)                                         AS avg_spend,
        STDDEV(amount)                                      AS spend_stddev,
        MAX(sale_date)                                      AS last_purchase_date,
        DATEDIFF('day', MAX(sale_date), CURRENT_DATE)       AS recency_days,
        COUNT(DISTINCT product)                             AS product_diversity,
        PERCENTILE_CONT(0.5)
          WITHIN GROUP (ORDER BY amount)                    AS median_spend,
        AVG(amount) FILTER (
          WHERE sale_date >= CURRENT_DATE - INTERVAL '30 days'
        )                                                   AS avg_spend_30d,
        AVG(amount) FILTER (
          WHERE sale_date >= CURRENT_DATE - INTERVAL '90 days'
        )                                                   AS avg_spend_90d,
        MIN(sale_date)                                      AS first_purchase_date
      FROM sales_data
      WHERE sale_date >= CURRENT_DATE - INTERVAL '365 days'
      GROUP BY customer_id
    )
    SELECT
      *,
      DATEDIFF('day', first_purchase_date, last_purchase_date)
        / NULLIF(frequency - 1, 0)                          AS avg_inter_purchase_gap,
      total_spend / NULLIF(frequency, 0)                   AS avg_order_value,
      CASE
        WHEN avg_spend_30d < avg_spend_90d * 0.7 THEN 1
        ELSE 0
      END                                                   AS declining_trend_flag
    FROM customer_features;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    import numpy as np
    
    today = pd.Timestamp('2024-12-31')
    
    # Base RFM-style features
    feats = (
      df.groupby('customer_id')
      .agg(
        frequency     = ('amount', 'count'),
        total_spend   = ('amount', 'sum'),
        avg_spend     = ('amount', 'mean'),
        spend_std     = ('amount', 'std'),
        last_date     = ('sale_date', 'max'),
        first_date    = ('sale_date', 'min'),
        product_div   = ('product', 'nunique'),
        median_spend  = ('amount', 'median')
      )
      .reset_index()
    )
    
    feats['recency_days']    = (today - feats['last_date']).dt.days
    feats['avg_order_value'] = feats['total_spend'] / feats['frequency']
    feats['tenure_days']     = (feats['last_date'] - feats['first_date']).dt.days
    feats['avg_gap_days']    = feats['tenure_days'] / (feats['frequency'] - 1).clip(lower=1)
    
    # Rolling window features: 30-day vs 90-day trend
    cutoff_30 = today - pd.Timedelta(days=30)
    cutoff_90 = today - pd.Timedelta(days=90)
    
    avg_30d = (
      df[df['sale_date'] >= cutoff_30]
      .groupby('customer_id')['amount'].mean()
      .rename('avg_30d')
    )
    avg_90d = (
      df[df['sale_date'] >= cutoff_90]
      .groupby('customer_id')['amount'].mean()
      .rename('avg_90d')
    )
    
    feats = feats.merge(avg_30d, on='customer_id', how='left')
    feats = feats.merge(avg_90d, on='customer_id', how='left')
    feats['declining_flag'] = (feats['avg_30d'] < feats['avg_90d'] * 0.7).astype(int)
    
    print(feats.head())
    
    
    

> 💡 **Key Insight:** Always articulate **WHY** a feature is meaningful, not just how to compute it. Recency matters because customers who haven't bought recently are more likely to churn. Spend volatility matters because erratic buyers are less loyal than consistent ones. Business intuition = senior analyst credibility.

* * *

# AREA 2 — Core Problem-Solving Techniques

* * *

## Technique 1 — Z-Score Analysis & Standardisation

**What it answers:** _"Is this value unusually high or low relative to the distribution?"_

Used for outlier flagging, comparing values across different scales, and normalising features for ML models.

**When to use:** Z-score when data is approximately normally distributed. Values with |z| > 3 are statistical outliers. Use **IQR method** instead for heavily skewed distributions (e.g., transaction amounts, claim values).

* * *

### Interview Question

> _"How would you identify and flag anomalous transactions in a sales dataset using Z-score?"_

|   
---|---  
**Situation**|  At AXA Insurance, the fraud detection team needed a programmatic way to flag claims that were statistically unusual — not just over a fixed threshold, but relative to the distribution within each product line.  
**Task**|  I needed to implement z-score based flagging that could run in near-real-time within Fabric notebooks against streaming claims data.  
**Action**|  I calculated mean and standard deviation of claim amounts per product line, computed a z-score for each claim, and flagged those with |z| > 3 for manual review. I combined this with IQR-based detection for the heavily skewed tail.  
**Result**|  The system flagged 0.4% of claims for review but captured 23% of confirmed fraud cases in the validation dataset — a 57× precision improvement over the previous fixed-threshold approach.  
  
* * *

### SQL Implementation
    
    
    WITH stats AS (
      SELECT
        product,
        AVG(amount)    AS mean_amount,
        STDDEV(amount) AS std_amount
      FROM sales_data
      GROUP BY product
    ),
    zscores AS (
      SELECT
        s.customer_id,
        s.product,
        s.amount,
        st.mean_amount,
        st.std_amount,
        (s.amount - st.mean_amount) / NULLIF(st.std_amount, 0)  AS z_score
      FROM sales_data s
      JOIN stats st ON s.product = st.product
    )
    SELECT
      *,
      CASE
        WHEN ABS(z_score) > 3 THEN 'OUTLIER'
        WHEN ABS(z_score) > 2 THEN 'REVIEW'
        ELSE 'NORMAL'
      END                             AS flag
    FROM zscores
    ORDER BY ABS(z_score) DESC;
    
    
    

* * *

### Pandas Implementation
    
    
    from scipy import stats as scipy_stats
    
    # Method 1: scipy zscore — global (use only for homogeneous populations)
    df['z_score_global'] = scipy_stats.zscore(df['amount'])
    
    # Method 2: z-score WITHIN product group — much more accurate
    df['z_score_product'] = df.groupby('product')['amount'].transform(
      lambda x: (x - x.mean()) / x.std()
    )
    
    df['flag'] = 'NORMAL'
    df.loc[df['z_score_product'].abs() > 2, 'flag'] = 'REVIEW'
    df.loc[df['z_score_product'].abs() > 3, 'flag'] = 'OUTLIER'
    
    print(df['flag'].value_counts())
    print(df[df['flag'] == 'OUTLIER'][['customer_id', 'product', 'amount', 'z_score_product']])
    
    
    

> 💡 **Key Insight:** Always compute z-scores **within a meaningful group** (product, region, customer segment) — a global z-score on mixed populations is statistically meaningless and produces false positives. A £50,000 transaction is normal for Enterprise but anomalous for Consumer.

* * *

## Technique 2 — Cohort Analysis

**What it answers:** _"How do different groups of users/customers behave over time?"_

Critical for understanding retention, engagement degradation, and the effectiveness of acquisition channels.

**When to use:** Whenever you need to compare the behaviour of groups defined by a common starting event (signup month, first purchase month, first claim date). It controls for recency bias that plagues simple averages.

* * *

### Interview Question

> _"How would you build a monthly retention cohort table showing what percentage of customers return to purchase each subsequent month?"_

|   
---|---  
**Situation**|  At J&J Global Services, marketing needed to understand whether recently acquired customers (via a new Paid campaign) retained better than Organic customers — but simple averages were confounded by acquisition recency.  
**Task**|  I needed to build a cohort retention matrix showing, for each acquisition month, what percentage of customers made a repeat purchase in months 1, 2, 3, etc. after acquisition.  
**Action**|  I defined cohorts by `signup_month`, tracked `purchase_month` relative to acquisition using DATEDIFF in months, and computed retention rates at each offset. The matrix was loaded to Power BI.  
**Result**|  The analysis revealed Paid customers had 40% Month-1 retention vs 28% for Organic, but by Month-3 the gap closed to 5% — informing a decision to reduce Paid spend by 30% and redirect to retention campaigns.  
  
* * *

### SQL Implementation
    
    
    WITH cohorts AS (
      -- Each customer assigned to their signup cohort
      SELECT
        customer_id,
        DATE_TRUNC('month', signup_date)  AS cohort_month
      FROM customers
    ),
    customer_activity AS (
      -- All distinct months of purchase activity per customer
      SELECT DISTINCT
        customer_id,
        DATE_TRUNC('month', sale_date)    AS activity_month
      FROM sales_data
    ),
    cohort_size AS (
      SELECT cohort_month, COUNT(DISTINCT customer_id) AS cohort_customers
      FROM cohorts GROUP BY 1
    ),
    retention_raw AS (
      SELECT
        c.cohort_month,
        DATEDIFF('month', c.cohort_month, a.activity_month)  AS month_number,
        COUNT(DISTINCT a.customer_id)                        AS active_customers
      FROM cohorts c
      JOIN customer_activity a ON c.customer_id = a.customer_id
      GROUP BY 1, 2
    )
    SELECT
      r.cohort_month,
      r.month_number,
      r.active_customers,
      cs.cohort_customers,
      ROUND(100.0 * r.active_customers / cs.cohort_customers, 1) AS retention_pct
    FROM retention_raw r
    JOIN cohort_size cs ON r.cohort_month = cs.cohort_month
    ORDER BY r.cohort_month, r.month_number;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    # Merge customers (signup_date) with sales activity
    activity = df.merge(customers[['customer_id', 'signup_date']], on='customer_id', how='left')
    
    activity['cohort_month']   = activity['signup_date'].dt.to_period('M')
    activity['activity_month'] = activity['sale_date'].dt.to_period('M')
    activity['month_number']   = (
      (activity['activity_month'] - activity['cohort_month'])
      .apply(lambda x: x.n)
    )
    
    # Cohort size (Month 0 = all customers acquired in that cohort)
    cohort_size = (
      activity[activity['month_number'] == 0]
      .groupby('cohort_month')['customer_id']
      .nunique()
      .rename('cohort_size')
    )
    
    # Retention counts at each month offset
    retention = (
      activity
      .groupby(['cohort_month', 'month_number'])['customer_id']
      .nunique()
      .reset_index()
      .rename(columns={'customer_id': 'active_customers'})
    )
    
    retention = retention.merge(cohort_size, on='cohort_month')
    retention['retention_pct'] = (
      retention['active_customers'] / retention['cohort_size'] * 100
    ).round(1)
    
    # Pivot into a readable cohort matrix
    cohort_matrix = retention.pivot(
      index='cohort_month',
      columns='month_number',
      values='retention_pct'
    )
    print(cohort_matrix)
    
    
    

> 💡 **Key Insight:** The most common cohort analysis mistake is using calendar month instead of relative month offset. **Month 0 = acquisition month. Month 3 = 3 months AFTER acquisition** — not March. Get this wrong and the entire matrix is meaningless.

* * *

## Technique 3 — Funnel Analysis

**What it answers:** _"Where in the conversion journey are we losing customers?"_

Essential for e-commerce, SaaS, and any multi-step customer flow (quote → bind → claim in insurance; lead → qualified → closed in sales).

**When to use:** When there is an ordered sequence of events that users must complete. The largest drop-off rate at any stage is the highest-priority optimisation target.

* * *

### Interview Question

> _"How would you build a conversion funnel report for a marketing campaign showing drop-off at each stage?"_

|   
---|---  
**Situation**|  At J&J Global Services, a Salesforce-integrated marketing campaign had 5 stages: Lead → Qualified → Proposal → Negotiation → Closed Won. Leadership suspected the Proposal stage was broken.  
**Task**|  I needed to build a funnel analysis from the Salesforce event data that quantified conversion rates between each stage and identified which sales reps had above-average Proposal conversion rates.  
**Action**|  I used the `marketing_events` table, counted distinct customers at each stage, computed stage-to-stage conversion rates using LAG(), and cross-referenced with `sales_data` to identify rep-level patterns.  
**Result**|  Confirmed the Proposal stage had 34% conversion vs 61% industry benchmark. Root cause: proposals were sent without pricing customisation. Fixing this lifted Proposal → Negotiation conversion to 52% in the next quarter.  
  
* * *

### SQL Implementation
    
    
    -- marketing_events(campaign_id, customer_id, event_type, event_date)
    -- event_type IN ('Lead','Qualified','Proposal','Negotiation','Closed Won')
    
    WITH funnel AS (
      SELECT
        event_type,
        COUNT(DISTINCT customer_id) AS users_at_stage
      FROM marketing_events
      WHERE campaign_id = 'CAMP_2024_Q1'
      GROUP BY event_type
    ),
    ordered AS (
      SELECT
        event_type,
        users_at_stage,
        CASE event_type
          WHEN 'Lead'        THEN 1
          WHEN 'Qualified'   THEN 2
          WHEN 'Proposal'    THEN 3
          WHEN 'Negotiation' THEN 4
          WHEN 'Closed Won'  THEN 5
        END                    AS stage_order
      FROM funnel
    )
    SELECT
      stage_order,
      event_type,
      users_at_stage,
      LAG(users_at_stage) OVER (ORDER BY stage_order)  AS prev_stage_users,
      -- Stage-to-stage conversion
      ROUND(
        100.0 * users_at_stage
        / NULLIF(LAG(users_at_stage) OVER (ORDER BY stage_order), 0),
      1)                                               AS stage_conv_pct,
      -- Drop-off from previous stage
      LAG(users_at_stage) OVER (ORDER BY stage_order)
        - users_at_stage                               AS drop_off,
      -- Overall conversion from top of funnel
      ROUND(
        100.0 * users_at_stage
        / FIRST_VALUE(users_at_stage) OVER (ORDER BY stage_order),
      1)                                               AS overall_conv_pct
    FROM ordered
    ORDER BY stage_order;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    events = pd.DataFrame({
      'customer_id': [1,  1,  1,  2,  2,  3,  3,  3,  3,  4,  5,  5],
      'event_type':  ['Lead','Qualified','Proposal',
                      'Lead','Qualified',
                      'Lead','Qualified','Proposal','Negotiation',
                      'Lead',
                      'Lead','Qualified']
    })
    
    stage_order = ['Lead', 'Qualified', 'Proposal', 'Negotiation', 'Closed Won']
    
    funnel = (
      events.groupby('event_type')['customer_id']
      .nunique()
      .reindex(stage_order, fill_value=0)
      .reset_index()
      .rename(columns={'customer_id': 'users_at_stage'})
    )
    
    top = funnel['users_at_stage'].iloc[0]
    
    funnel['prev_stage']      = funnel['users_at_stage'].shift(1)
    funnel['stage_conv_pct']  = (funnel['users_at_stage'] / funnel['prev_stage'] * 100).round(1)
    funnel['overall_conv_pct'] = (funnel['users_at_stage'] / top * 100).round(1)
    funnel['drop_off']        = (funnel['prev_stage'] - funnel['users_at_stage']).fillna(0).astype(int)
    
    print(funnel[['event_type', 'users_at_stage', 'drop_off', 'stage_conv_pct', 'overall_conv_pct']])
    
    
    

> 💡 **Key Insight:** Always report **both** stage-to-stage conversion AND overall conversion from top of funnel. A 50% drop-off at Stage 3 of 5 means only the top 12.5% of leads ever reach Stage 5 — the compound effect is what matters to the business.

* * *

## Technique 4 — RFM Segmentation

**What it answers:** _"Which customers are our most valuable, most at-risk, and most likely to respond to campaigns?"_

RFM (Recency, Frequency, Monetary) is the gold standard for behavioural customer segmentation in any industry.

**When to use:** When you need to prioritise customers for campaigns, identify churn risk, or target high-value customers for retention. Works on any dataset with customer IDs, transaction dates, and amounts.

* * *

### Interview Question

> _"Walk me through how you would build an RFM segmentation model and define actionable customer segments."_

|   
---|---  
**Situation**|  At J&J Global Services, the marketing team wanted to move away from mass-email campaigns to targeted outreach — but had no segmentation framework beyond basic demographic splits.  
**Task**|  I needed to build an RFM model from CRM transaction data and define actionable segments the marketing team could act on directly in Salesforce.  
**Action**|  I computed R (days since last purchase), F (purchase count in 12 months), M (total spend) for each customer. I assigned 1–5 scores to each dimension using NTILE(5) quintiles, then combined scores to define 6 behavioural segments.  
**Result**|  Segmentation enabled personalised campaigns that increased email open rates by 34% and reduced churn by 12%, with the framework adopted as a standing monthly report in Power BI.  
  
* * *

### SQL Implementation
    
    
    WITH rfm_raw AS (
      SELECT
        customer_id,
        DATEDIFF('day', MAX(sale_date), CURRENT_DATE)  AS recency,
        COUNT(*)                                        AS frequency,
        SUM(amount)                                     AS monetary
      FROM sales_data
      WHERE sale_date >= CURRENT_DATE - INTERVAL '365 days'
      GROUP BY customer_id
    ),
    rfm_scored AS (
      SELECT
        customer_id, recency, frequency, monetary,
        -- Lower recency (more recent) = higher score, so ORDER BY ASC
        NTILE(5) OVER (ORDER BY recency ASC)    AS r_score,
        NTILE(5) OVER (ORDER BY frequency DESC) AS f_score,
        NTILE(5) OVER (ORDER BY monetary DESC)  AS m_score
      FROM rfm_raw
    )
    SELECT
      customer_id, recency, frequency, monetary,
      r_score, f_score, m_score,
      (r_score + f_score + m_score)    AS rfm_total,
      CASE
        WHEN r_score >= 4 AND f_score >= 4           THEN 'Champion'
        WHEN r_score >= 3 AND f_score >= 3           THEN 'Loyal'
        WHEN r_score <= 2 AND f_score >= 3           THEN 'At Risk'
        WHEN r_score <= 2 AND f_score <= 2           THEN 'Hibernating'
        WHEN r_score >= 4 AND f_score <= 2           THEN 'New Customer'
        ELSE 'Needs Attention'
      END                              AS segment
    FROM rfm_scored;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    import numpy as np
    
    today = pd.Timestamp('2024-12-31')
    
    rfm = (
      df.groupby('customer_id')
      .agg(
        recency   = ('sale_date', lambda x: (today - x.max()).days),
        frequency = ('amount', 'count'),
        monetary  = ('amount', 'sum')
      )
      .reset_index()
    )
    
    # Quintile scoring: 1 = worst, 5 = best for each dimension
    # NOTE: Recency is reversed — lower days = more recent = better score
    rfm['r_score'] = pd.qcut(rfm['recency'], 5, labels=[5, 4, 3, 2, 1]).astype(int)
    rfm['f_score'] = pd.qcut(rfm['frequency'].rank(method='first'), 5, labels=[1, 2, 3, 4, 5]).astype(int)
    rfm['m_score'] = pd.qcut(rfm['monetary'].rank(method='first'),  5, labels=[1, 2, 3, 4, 5]).astype(int)
    
    rfm['rfm_total'] = rfm['r_score'] + rfm['f_score'] + rfm['m_score']
    
    def assign_segment(row):
      r, f = row['r_score'], row['f_score']
      if r >= 4 and f >= 4: return 'Champion'
      if r >= 3 and f >= 3: return 'Loyal'
      if r <= 2 and f >= 3: return 'At Risk'
      if r <= 2 and f <= 2: return 'Hibernating'
      if r >= 4 and f <= 2: return 'New Customer'
      return 'Needs Attention'
    
    rfm['segment'] = rfm.apply(assign_segment, axis=1)
    
    # Summary by segment
    print(
      rfm.groupby('segment')
      .agg(
        count       = ('customer_id', 'count'),
        avg_recency = ('recency', 'mean'),
        avg_freq    = ('frequency', 'mean'),
        avg_spend   = ('monetary', 'mean')
      )
      .round(1)
      .sort_values('avg_spend', ascending=False)
    )
    
    
    

> 💡 **Key Insight:** Recency scoring is **inverted** — a customer who bought yesterday has LOW recency in days but deserves a HIGH RFM score. This trips up many candidates. Always clarify the scoring direction explicitly in your explanation.

* * *

## Technique 5 — Anomaly Detection (IQR Method)

**What it answers:** _"Are there data points inconsistent with the rest of the distribution — due to fraud, error, or genuinely unusual events?"_

IQR is robust to non-normal distributions where Z-score fails.

**When to use:** IQR-based detection for skewed financial data (transaction amounts, claim values). Z-score for symmetric, normally distributed data. Combine both for production fraud detection systems.

* * *

### Interview Question

> _"How would you detect and flag outlier transactions in a skewed sales amount distribution?"_

|   
---|---  
**Situation**|  At AXA Insurance, claim amounts are highly right-skewed. Z-score flagging was generating too many false positives on legitimate large claims. The fraud team needed a statistically robust alternative.  
**Task**|  I needed to implement IQR-based outlier detection robust to the skewed distribution, embeddable in the Fabric notebooks pipeline for near-real-time flagging.  
**Action**|  I computed Q1, Q3, and IQR per product line, then set an upper fence at Q3 + 1.5×IQR and a strict fence at Q3 + 3×IQR. Claims between fences went to a review queue; above the strict fence were auto-flagged.  
**Result**|  The dual-fence approach reduced false positives by 67% compared to the previous Z-score method while maintaining 89% recall on confirmed fraud cases.  
  
* * *

### SQL Implementation
    
    
    WITH quartiles AS (
      SELECT
        product,
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) AS q1,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) AS q3
      FROM sales_data
      GROUP BY product
    ),
    iqr_bounds AS (
      SELECT
        product, q1, q3,
        (q3 - q1)                  AS iqr,
        q1 - 1.5 * (q3 - q1)      AS lower_fence,
        q3 + 1.5 * (q3 - q1)      AS upper_fence,
        q3 + 3.0 * (q3 - q1)      AS strict_upper_fence
      FROM quartiles
    )
    SELECT
      s.customer_id,
      s.product,
      s.amount,
      b.lower_fence,
      b.upper_fence,
      b.strict_upper_fence,
      CASE
        WHEN s.amount > b.strict_upper_fence THEN 'AUTO_FLAG'
        WHEN s.amount > b.upper_fence        THEN 'REVIEW_QUEUE'
        WHEN s.amount < b.lower_fence        THEN 'LOW_OUTLIER'
        ELSE 'NORMAL'
      END                          AS anomaly_flag
    FROM sales_data s
    JOIN iqr_bounds b ON s.product = b.product
    ORDER BY s.amount DESC;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    def compute_iqr_bounds(series):
      q1, q3 = series.quantile(0.25), series.quantile(0.75)
      iqr = q3 - q1
      return pd.Series({
        'q1': q1, 'q3': q3, 'iqr': iqr,
        'lower':  q1 - 1.5 * iqr,
        'upper':  q3 + 1.5 * iqr,
        'strict': q3 + 3.0 * iqr
      })
    
    # Compute bounds per product group
    bounds = (
      df.groupby('product')['amount']
      .apply(compute_iqr_bounds)
      .unstack()
      .reset_index()
    )
    
    df_flagged = df.merge(bounds, on='product', how='left')
    
    def flag_row(row):
      if row['amount'] > row['strict']: return 'AUTO_FLAG'
      if row['amount'] > row['upper']:  return 'REVIEW_QUEUE'
      if row['amount'] < row['lower']:  return 'LOW_OUTLIER'
      return 'NORMAL'
    
    df_flagged['anomaly_flag'] = df_flagged.apply(flag_row, axis=1)
    
    print(df_flagged['anomaly_flag'].value_counts())
    print(df_flagged[df_flagged['anomaly_flag'] == 'AUTO_FLAG'][
      ['customer_id', 'product', 'amount', 'upper', 'strict']
    ])
    
    
    

> 💡 **Key Insight:** Always perform outlier detection **within meaningful groups** — a £50,000 transaction is normal for Enterprise but anomalous for Consumer. Global IQR on heterogeneous data produces the same flaw as global Z-score: statistically meaningless flags.

* * *

## Technique 6 — Trend Decomposition & Moving Averages

**What it answers:** _"Is the trend I'm seeing real, or is it seasonal noise?"_

Decomposing time series into trend, seasonality, and residual components is essential for accurate reporting and prevents premature leadership interventions.

**When to use:** When reporting on weekly/monthly KPI metrics to distinguish genuine business trend from repeating seasonal patterns. Essential for any executive dashboard that drives resource allocation decisions.

* * *

### Interview Question

> _"How would you separate genuine revenue growth from seasonal effects in a monthly sales report?"_

|   
---|---  
**Situation**|  At Google Street View, the executive dashboard showed a sharp decline in processed image volume in Q4 2023. Leadership was concerned. The data covered 3 years.  
**Task**|  I needed to determine whether the decline was a genuine trend or a recurring seasonal pattern before escalating to leadership.  
**Action**|  I applied 3-month and 12-month moving averages to smooth noise, then used Python's `statsmodels` to formally decompose the time series into trend, seasonal, and residual components. I visualised all three layers separately.  
**Result**|  The decomposition confirmed Q4 always showed a 15% seasonal dip across all 3 years. The genuine trend was flat-to-slightly-positive. This prevented a premature leadership intervention and changed the quarterly reporting template to always show both raw and seasonally-adjusted figures.  
  
* * *

### SQL Implementation
    
    
    WITH monthly AS (
      SELECT
        DATE_TRUNC('month', sale_date)  AS month,
        SUM(amount)                     AS monthly_revenue
      FROM sales_data
      GROUP BY 1
    )
    SELECT
      month,
      monthly_revenue,
      -- 3-month moving average
      AVG(monthly_revenue) OVER (
        ORDER BY month
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
      )                               AS ma_3month,
      -- 12-month moving average (seasonal baseline)
      AVG(monthly_revenue) OVER (
        ORDER BY month
        ROWS BETWEEN 11 PRECEDING AND CURRENT ROW
      )                               AS ma_12month,
      -- Deviation from seasonal baseline
      ROUND(
        monthly_revenue / NULLIF(
          AVG(monthly_revenue) OVER (
            ORDER BY month
            ROWS BETWEEN 11 PRECEDING AND CURRENT ROW
          ), 0
        ) - 1,
      3)                              AS deviation_from_ma
    FROM monthly
    ORDER BY month;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    # Monthly aggregation with a datetime index
    monthly = (
      df.set_index('sale_date')
      .resample('M')['amount']
      .sum()
      .rename('monthly_revenue')
    )
    
    df_trend = monthly.to_frame()
    df_trend['ma_3m']      = df_trend['monthly_revenue'].rolling(window=3).mean()
    df_trend['ma_12m']     = df_trend['monthly_revenue'].rolling(window=12).mean()
    df_trend['deviation']  = (df_trend['monthly_revenue'] / df_trend['ma_12m'] - 1).round(3)
    df_trend['mom_change'] = df_trend['monthly_revenue'].pct_change().round(3)
    
    print(df_trend.tail(12))
    
    # Formal seasonal decomposition (requires 2+ complete cycles)
    from statsmodels.tsa.seasonal import seasonal_decompose
    
    # decomp = seasonal_decompose(monthly, model='additive', period=12)
    # trend    = decomp.trend     # underlying direction
    # seasonal = decomp.seasonal  # repeating pattern
    # residual = decomp.resid     # unexplained noise
    # decomp.plot()
    
    
    

> 💡 **Key Insight:** The 12-month moving average is your seasonal baseline. Any point above/below represents genuine trend or random noise. Always ask _"Does this dip happen every year?"_ before presenting a declining trend to leadership.

* * *

## Technique 7 — A/B Test Result Interpretation

**What it answers:** _"Is the difference between two versions statistically real, or could it be random chance?"_

A/B testing is the gold standard for measuring the causal impact of changes. Correlation in observational data is never sufficient to prove causation.

**When to use:** Whenever you want to make a causal claim — that X caused Y. Properly randomised A/B tests are the only way to prove causation from data.

* * *

### Interview Question

> _"How do you determine if an A/B test result is statistically significant? Walk me through the full analysis."_

|   
---|---  
**Situation**|  At J&J Global Services, marketing ran a 4-week A/B test: Group A received the standard email template, Group B received a personalised version generated from the CRM segmentation data I had built.  
**Task**|  I needed to analyse the test results rigorously — not just compare open rates, but determine statistical significance and quantify revenue lift.  
**Action**|  I ran a two-sample z-test for proportions (for open rate) and a Welch t-test for revenue per recipient. I calculated p-value, effect size (Cohen's d), confidence intervals, and projected annual revenue lift. I also checked for sample ratio mismatch (equal group sizes).  
**Result**|  Group B showed 34% higher open rate (p = 0.003, statistically significant) and €47 higher revenue per recipient. Projected annually: €28M incremental revenue — directly informing the full rollout decision.  
  
* * *

### SQL Implementation
    
    
    -- ab_test(customer_id, group_label, converted, revenue)
    
    WITH summary AS (
      SELECT
        group_label,
        COUNT(*)                                     AS n_users,
        SUM(converted)                               AS conversions,
        ROUND(AVG(converted::FLOAT) * 100, 2)        AS conversion_rate_pct,
        ROUND(AVG(revenue), 2)                       AS avg_revenue,
        ROUND(STDDEV(revenue), 2)                    AS stddev_revenue,
        SUM(revenue)                                 AS total_revenue
      FROM ab_test
      GROUP BY group_label
    ),
    -- Check sample ratio mismatch (groups should be roughly equal)
    srm_check AS (
      SELECT
        MAX(n_users) - MIN(n_users)       AS size_diff,
        ROUND(MAX(n_users) * 1.0 / MIN(n_users), 3) AS size_ratio,
        CASE WHEN MAX(n_users) * 1.0 / MIN(n_users) > 1.05
             THEN 'SRM DETECTED' ELSE 'OK' END AS srm_flag
      FROM summary
    )
    SELECT
      s.*,
      srm.srm_flag,
      -- Lift calculation
      t.conversion_rate_pct - c.conversion_rate_pct             AS lift_ppt,
      ROUND(100.0 * (t.conversion_rate_pct - c.conversion_rate_pct)
        / c.conversion_rate_pct, 1)                             AS relative_lift_pct,
      t.avg_revenue - c.avg_revenue                             AS revenue_lift_per_user
    FROM summary s
    CROSS JOIN summary c
    CROSS JOIN summary t
    CROSS JOIN srm_check srm
    WHERE c.group_label = 'Control'
      AND t.group_label = 'Treatment'
      AND s.group_label IN ('Control','Treatment');
    
    
    

* * *

### Pandas + SciPy Implementation
    
    
    import pandas as pd
    import numpy as np
    from scipy import stats
    from statsmodels.stats.proportion import proportions_ztest
    
    np.random.seed(42)
    n = 2500
    
    control = pd.DataFrame({
      'group':     'Control',
      'converted': np.random.binomial(1, 0.12, n),
      'revenue':   np.random.exponential(200, n)
    })
    treatment = pd.DataFrame({
      'group':     'Treatment',
      'converted': np.random.binomial(1, 0.16, n),   # +4 percentage point lift
      'revenue':   np.random.exponential(247, n)     # +€47 per user lift
    })
    
    df_ab = pd.concat([control, treatment])
    
    # Summary statistics
    summary = df_ab.groupby('group').agg(
      n           = ('revenue', 'count'),
      conversions = ('converted', 'sum'),
      conv_rate   = ('converted', 'mean'),
      avg_revenue = ('revenue', 'mean'),
      std_revenue = ('revenue', 'std')
    ).round(4)
    print(summary)
    
    # 1. Two-proportion z-test: conversion rate
    counts = df_ab.groupby('group')['converted'].sum().values
    nobs   = df_ab.groupby('group')['converted'].count().values
    z_stat, p_conv = proportions_ztest(counts, nobs)
    print(f'\nConversion z-test: z={z_stat:.3f}, p={p_conv:.4f}, significant={p_conv < 0.05}')
    
    # 2. Welch t-test: revenue per user (unequal variances)
    ctrl_rev = df_ab[df_ab['group'] == 'Control']['revenue']
    trt_rev  = df_ab[df_ab['group'] == 'Treatment']['revenue']
    t_stat, p_rev = stats.ttest_ind(ctrl_rev, trt_rev, equal_var=False)
    print(f'Revenue t-test:    t={t_stat:.3f}, p={p_rev:.4f}, significant={p_rev < 0.05}')
    
    # 3. Effect size: Cohen's d
    pooled_std = np.sqrt((ctrl_rev.std()**2 + trt_rev.std()**2) / 2)
    cohens_d   = (trt_rev.mean() - ctrl_rev.mean()) / pooled_std
    print(f"Cohen's d:         {cohens_d:.3f}  ({'small' if cohens_d < 0.2 else 'medium' if cohens_d < 0.5 else 'large'})")
    
    # 4. 95% confidence interval for revenue lift
    ci = stats.t.interval(0.95, df=len(ctrl_rev)+len(trt_rev)-2,
                          loc=trt_rev.mean()-ctrl_rev.mean(),
                          scale=stats.sem(trt_rev - trt_rev.mean()))
    print(f"Revenue lift 95% CI: ({ci[0]:.1f}, {ci[1]:.1f})")
    
    # 5. Sample ratio mismatch check
    group_sizes = df_ab['group'].value_counts()
    srm_ratio = group_sizes.max() / group_sizes.min()
    print(f"SRM check: ratio={srm_ratio:.3f} ({'OK' if srm_ratio < 1.05 else 'SRM DETECTED'})")
    
    
    

> 💡 **Key Insight:** Never report just the p-value — always pair it with **effect size** (Cohen's d or relative lift %) and **confidence intervals**. A p=0.001 result with 0.2% lift is statistically significant but practically worthless. Business decisions require both statistical AND practical significance.

* * *

## Technique 8 — Root Cause Analysis (Drill-Down & Slice-and-Dice)

**What it answers:** _"Revenue dropped 15% this month — WHY?"_

Root cause analysis is the systematic process of decomposing a top-level metric change into its component parts to find the true driver.

**When to use:** Whenever a KPI moves unexpectedly. Framework: (1) Confirm the metric moved, (2) Decompose by all dimensions, (3) Find where the change is concentrated, (4) Drill into that dimension, (5) Hypothesise and test root cause.

* * *

### Interview Question

> _"Revenue dropped 20% this month. Walk me through exactly how you would find the root cause."_

|   
---|---  
**Situation**|  At Google Street View, the monthly executive dashboard showed a 22% drop in processed image volume — a key R&D pipeline metric. Leadership suspected infrastructure. I suspected data quality.  
**Task**|  I was asked to perform a root cause analysis and present findings to the VP of R&D within 48 hours.  
**Action**|  I followed structured binary decomposition: confirmed the drop was real (not a dashboard bug), then sliced by region, pipeline stage, and image type. The drop was 100% concentrated in one region (APAC) and only in the 'Forest/Non-Forest' image layer. I traced this to a single upstream data source that had changed its schema after a vendor update, silently producing NULLs in 3 key fields.  
**Result**|  Root cause identified in 6 hours. Fix deployed in 18 hours vs the 2-week estimate for a full infrastructure investigation. The defect reporting pipeline improvement subsequently cut average bug-to-resolution time by 18 hours.  
  
* * *

### SQL Implementation
    
    
    -- Step 1: Confirm the metric change is real (not a dashboard rendering bug)
    SELECT
      DATE_TRUNC('month', sale_date)             AS month,
      SUM(amount)                                AS total_revenue,
      LAG(SUM(amount))
        OVER (ORDER BY DATE_TRUNC('month', sale_date)) AS prev_month_revenue,
      ROUND(
        SUM(amount) /
        NULLIF(LAG(SUM(amount)) OVER (ORDER BY DATE_TRUNC('month', sale_date)), 0) - 1,
      3)                                         AS mom_pct_change
    FROM sales_data
    GROUP BY 1 ORDER BY 1;
    
    -- Step 2: Decompose by region — find where the change is concentrated
    SELECT
      region,
      SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-11-01'
          THEN amount ELSE 0 END)                AS nov_revenue,
      SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-10-01'
          THEN amount ELSE 0 END)                AS oct_revenue,
      ROUND(
        SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-11-01' THEN amount ELSE 0 END) /
        NULLIF(SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-10-01'
                   THEN amount ELSE 0 END), 0) - 1,
      3)                                         AS region_mom_change
    FROM sales_data
    WHERE DATE_TRUNC('month', sale_date) IN ('2024-10-01', '2024-11-01')
    GROUP BY region
    ORDER BY region_mom_change ASC;  -- worst region at top
    
    -- Step 3: Drill into worst region by product
    SELECT
      product,
      SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-11-01' THEN amount ELSE 0 END) AS nov,
      SUM(CASE WHEN DATE_TRUNC('month', sale_date) = '2024-10-01' THEN amount ELSE 0 END) AS oct
    FROM sales_data
    WHERE region = 'APAC'    -- most affected region from Step 2
      AND DATE_TRUNC('month', sale_date) IN ('2024-10-01', '2024-11-01')
    GROUP BY product
    ORDER BY nov ASC;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    df['month'] = df['sale_date'].dt.to_period('M')
    
    # Step 1: Confirm metric change
    monthly_total = df.groupby('month')['amount'].sum()
    mom_change = monthly_total.pct_change().rename('mom_change')
    report = pd.DataFrame({'revenue': monthly_total, 'mom_change': mom_change})
    print("Step 1 — Monthly totals:\n", report.tail(6))
    
    # Step 2: Decompose by region
    region_month = df.groupby(['region', 'month'])['amount'].sum().unstack('month')
    last_two = region_month.iloc[:, -2:].copy()
    last_two.columns = ['prev', 'curr']
    last_two['mom_pct'] = (last_two['curr'] / last_two['prev'] - 1).round(3)
    print("\nStep 2 — By region:\n", last_two.sort_values('mom_pct'))
    
    # Step 3: Drill into worst region
    worst_region = last_two['mom_pct'].idxmin()
    print(f"\nWorst region: {worst_region}")
    
    product_month = (
      df[df['region'] == worst_region]
      .groupby(['product', 'month'])['amount']
      .sum()
      .unstack('month')
      .iloc[:, -2:]
      .copy()
    )
    product_month.columns = ['prev', 'curr']
    product_month['mom_pct'] = (product_month['curr'] / product_month['prev'] - 1).round(3)
    print(f"\nStep 3 — Drill-down in {worst_region}:\n", product_month.sort_values('mom_pct'))
    
    
    

> 💡 **Key Insight:** The key to root cause analysis is **binary decomposition** — at each step, find the ONE dimension where the change is most concentrated, then drill exclusively into that. "Boil the ocean" approaches (checking everything simultaneously) waste time and confuse stakeholders.

* * *

## Technique 9 — Data Validation & Attribution Analysis

**What it answers:** _"Can I trust this data?"_ and _"Which touchpoint deserves credit for the conversion?"_

These two techniques are inseparable in practice — attribution without data validation produces confidently wrong answers.

**When to use:** Data validation as a standard automated gate before ANY analysis that drives a business decision. Attribution when multiple marketing channels touch a customer before conversion and you need to allocate budget rationally.

* * *

### Interview Question

> _"How would you validate a new dataset before using it in a business-critical report?"_

|   
---|---  
**Situation**|  At AXA Insurance, a new claims data stream was being handed over from the legacy system to Microsoft Fabric. I was responsible for validating the migrated data before it could be used by the actuarial team for loss reserving — a process with regulatory implications.  
**Task**|  I needed a systematic, automated validation framework that could run on each pipeline load and produce a clear pass/fail report for the data engineering team.  
**Action**|  I built a 4-layer validation framework: (1) row count reconciliation vs source, (2) NULL rate checks against thresholds, (3) referential integrity checks, (4) business rule validation (claim date cannot precede policy start date). All results were logged to `pipeline_logs`.  
**Result**|  The framework detected a 0.3% row discrepancy in the first load (caused by timezone handling) and 4 referential integrity failures — both invisible without automated checks. Reduced manual review efforts by 40% and became the standard validation template for all Fabric notebook pipelines at AXA.  
  
* * *

### SQL Implementation
    
    
    -- Layer 1: Row count reconciliation vs source system
    SELECT
      'row_count'                              AS check_name,
      COUNT(*)                                 AS actual_rows,
      500000                                   AS expected_rows,
      ROUND(ABS(COUNT(*) - 500000) * 100.0 / 500000, 2) AS pct_deviation,
      CASE WHEN ABS(COUNT(*) - 500000) * 100.0 / 500000 > 1
           THEN 'FAIL' ELSE 'PASS' END         AS status
    FROM sales_data;
    
    -- Layer 2: NULL rate per key column
    SELECT
      'null_rate_customer_id'                  AS check_name,
      ROUND(100.0 * (COUNT(*) - COUNT(customer_id)) / COUNT(*), 2) AS null_pct,
      CASE WHEN (COUNT(*) - COUNT(customer_id)) * 100.0 / COUNT(*) > 0.5
           THEN 'FAIL' ELSE 'PASS' END         AS status
    FROM sales_data;
    
    -- Layer 3: Referential integrity (all sales must have valid customer)
    SELECT
      'ref_integrity_customer'                 AS check_name,
      COUNT(s.customer_id)                     AS orphan_count,
      CASE WHEN COUNT(s.customer_id) > 0 THEN 'FAIL' ELSE 'PASS' END AS status
    FROM sales_data s
    LEFT JOIN customers c ON s.customer_id = c.customer_id
    WHERE c.customer_id IS NULL;
    
    -- Layer 4: Business rule — sale date must be after signup date
    SELECT
      'biz_rule_sale_after_signup'             AS check_name,
      COUNT(*)                                 AS violations,
      CASE WHEN COUNT(*) > 0 THEN 'FAIL' ELSE 'PASS' END AS status
    FROM sales_data s
    JOIN customers c ON s.customer_id = c.customer_id
    WHERE s.sale_date < c.signup_date;
    
    -- Layer 5: Statistical distribution check (compare to baseline)
    SELECT
      'distribution_check'                     AS check_name,
      ROUND(AVG(amount), 2)                    AS current_mean,
      1450.00                                  AS baseline_mean,  -- from prior period
      ROUND(ABS(AVG(amount) - 1450) / 1450 * 100, 2) AS pct_drift,
      CASE WHEN ABS(AVG(amount) - 1450) / 1450 > 0.10
           THEN 'FAIL' ELSE 'PASS' END         AS status
    FROM sales_data;
    
    
    

* * *

### Pandas Implementation
    
    
    import pandas as pd
    
    def validate_dataset(df, customers, expected_rows=500, baseline_mean=1450.0):
      """
      Automated data validation framework — 4 layers.
      Returns a DataFrame with check name, value, threshold, and pass/fail status.
      """
      results = []
    
      # Layer 1: Row count reconciliation
      pct_dev = abs(len(df) - expected_rows) / expected_rows * 100
      results.append({
        'check':     'row_count',
        'actual':    len(df),
        'threshold': expected_rows,
        'pct_dev':   round(pct_dev, 2),
        'status':    'PASS' if pct_dev <= 1 else 'FAIL'
      })
    
      # Layer 2: NULL rates per key column
      for col in ['customer_id', 'amount', 'region', 'sale_date']:
        null_pct = df[col].isnull().mean() * 100
        results.append({
          'check':     f'null_rate_{col}',
          'actual':    round(null_pct, 2),
          'threshold': 0.5,
          'status':    'PASS' if null_pct <= 0.5 else 'FAIL'
        })
    
      # Layer 3: Referential integrity
      valid_ids = set(customers['customer_id'])
      orphans   = (~df['customer_id'].isin(valid_ids)).sum()
      results.append({
        'check':     'ref_integrity_customer',
        'actual':    int(orphans),
        'threshold': 0,
        'status':    'PASS' if orphans == 0 else 'FAIL'
      })
    
      # Layer 4: Business rule validation
      merged = df.merge(customers[['customer_id', 'signup_date']], on='customer_id', how='left')
      if 'signup_date' in merged.columns:
        violations = (merged['sale_date'] < merged['signup_date']).sum()
        results.append({
          'check':     'sale_after_signup',
          'actual':    int(violations),
          'threshold': 0,
          'status':    'PASS' if violations == 0 else 'FAIL'
        })
    
      # Layer 5: Distribution drift check
      mean_drift_pct = abs(df['amount'].mean() - baseline_mean) / baseline_mean * 100
      results.append({
        'check':     'distribution_mean_drift',
        'actual':    round(df['amount'].mean(), 2),
        'threshold': baseline_mean,
        'pct_dev':   round(mean_drift_pct, 2),
        'status':    'PASS' if mean_drift_pct <= 10 else 'FAIL'
      })
    
      report = pd.DataFrame(results).fillna('')
      overall = 'ALL PASS ✓' if (report['status'] == 'PASS').all() else 'FAILURES DETECTED ✗'
      print(report.to_string(index=False))
      print(f"\nOverall validation: {overall}")
      return report
    
    # Run validation
    report = validate_dataset(df, customers, expected_rows=500)
    
    
    

> 💡 **Key Insight:** Data validation is not optional — it **is** part of the analysis. Mentioning that you built automated validation checks (not manual spot-checks) signals production-grade engineering thinking that distinguishes senior candidates from analysts who just "look at the data."

* * *

## Quick Reference — Interview Cheat Sheet

## SQL Window Functions — Know Cold
    
    
    -- Ranking
    ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC)
    RANK()       OVER (PARTITION BY region ORDER BY amount DESC)  -- gaps on ties
    DENSE_RANK() OVER (PARTITION BY region ORDER BY amount DESC)  -- no gaps
    
    -- Offset
    LAG(amount, 1, 0) OVER (PARTITION BY customer_id ORDER BY sale_date)  -- prev row
    LEAD(amount, 1)   OVER (PARTITION BY customer_id ORDER BY sale_date)  -- next row
    
    -- Running totals
    SUM(amount) OVER (
      PARTITION BY customer_id
      ORDER BY sale_date
      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )
    
    -- Moving average
    AVG(amount) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
    
    -- Percentiles
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount)   -- median (interpolated)
    PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY amount)   -- median (exact value)
    NTILE(5) OVER (ORDER BY amount)                        -- quintile buckets
    
    
    

## Pandas One-Liners — Know Cold
    
    
    # Groupby + multi-agg
    df.groupby('region').agg(n=('amount','count'), avg=('amount','mean')).reset_index()
    
    # Window equivalent
    df['prev']    = df.groupby('customer_id')['amount'].shift(1)      # LAG
    df['next']    = df.groupby('customer_id')['amount'].shift(-1)     # LEAD
    df['cumsum']  = df.groupby('customer_id')['amount'].cumsum()      # running total
    df['rank']    = df.groupby('region')['amount'].rank(method='min', ascending=False)
    df['ma_3m']   = df['amount'].rolling(3).mean()                    # moving avg
    
    # Fast conditionals (no apply)
    import numpy as np
    np.where(df['amount'] > 3000, 'High', 'Low')
    np.select([df['amount'] > 3000, df['amount'] > 1000], ['High','Mid'], default='Low')
    
    # Pivot & melt
    pd.pivot_table(df, values='amount', index='region', columns='product', aggfunc='sum')
    df.melt(id_vars=['id'], var_name='col', value_name='val')
    
    # Date
    pd.to_datetime(col)  |  .dt.to_period('M')  |  .dt.days  |  .dt.day_name()
    
    # String
    .str.strip().str.upper()  |  .str.split('@').str[1]  |  .str.contains('pattern')
    
    # Performance
    df['col'] = df['col'].astype('category')   # 4× memory saving for low-cardinality
    
    
    

* * *

## Technique Decision Framework

Business Question| Technique| Key Method  
---|---|---  
Is this value unusually high?| Z-Score / IQR Outlier Detection| `STDDEV`, `NTILE`, `scipy.stats.zscore`  
Why did the KPI drop this month?| Root Cause Analysis| `LAG`, `CASE` slicing, `.pct_change()`  
Are customer groups behaving differently over time?| Cohort Analysis| `DATE_TRUNC`, `DATEDIFF`, `pivot_table`  
Where are customers dropping off?| Funnel Analysis| `COUNT DISTINCT`, `LAG`, `.reindex()`  
Who are our most valuable customers?| RFM Segmentation| `NTILE`, `pd.qcut`, `groupby.agg`  
Did our campaign actually work?| A/B Test Interpretation| `proportions_ztest`, `ttest_ind`  
Is this trend real or seasonal?| Trend Decomposition| Rolling `AVG`, `seasonal_decompose`  
Can I trust this dataset?| Data Validation Framework| Row count, NULL checks, ref integrity  
How to allocate marketing budget?| Attribution Analysis| Event sequencing, touch counting  
  
* * *

## Common Interview Follow-Up Questions — Prepared Answers

**"What's the difference between RANK() and DENSE_RANK()?"**  
`RANK()` leaves gaps after ties (1, 2, 2, 4). `DENSE_RANK()` does not (1, 2, 2, 3). Use DENSE_RANK when you need consecutive rankings without gaps — e.g., "Top 3 regions" where two regions are tied for 2nd.

**"When would you use a CTE vs a temp table?"**  
CTEs are inline, readable, and go out of scope immediately — ideal for modular queries. Temp tables persist in the session, can be indexed, and are better for very large intermediate results that are referenced multiple times. In Snowflake/BigQuery, CTEs are generally preferred.

**"How do you handle skewed data in a model?"**  
Log transformation (`np.log1p`), capping at percentile (e.g., 99th), using median instead of mean for central tendency, or choosing tree-based models (which are scale-invariant). Always plot the distribution first.

**"What's the difference between HAVING and WHERE?"**  
`WHERE` filters rows before aggregation. `HAVING` filters groups after aggregation. You cannot use aggregate functions in a `WHERE` clause.

**"How would you test if two distributions are different?"**  
Welch t-test (unequal variances, continuous), Mann-Whitney U (non-parametric), two-proportion z-test (binary/conversion rate), or chi-square test (categorical frequencies). Always check assumptions before choosing the test.

**"What is p-hacking and how do you avoid it?"**  
Running multiple tests and only reporting the significant one inflates the false positive rate. Avoid by pre-registering your hypothesis, using Bonferroni correction for multiple comparisons, and reporting all tests run — not just the significant ones.

* * *

_Guide prepared for Shajeeb S — Senior Data Analyst | Data Science Specialist — Dublin, Ireland_  
 _Covers 9 Core Technical Methods + 9 Core Problem-Solving Techniques with dual SQL + Pandas implementations_
