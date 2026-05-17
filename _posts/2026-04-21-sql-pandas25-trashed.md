---
title: "sql_pandas25"
date: 2026-04-21 19:18:15 
author: shajeebhameed
layout: page
slug: sql_pandas25__trashed
status: trash
---

## Core Data Analyst Methods — Complete Q&A Guide

### STAR Format | Real-Time Reference Queries | SQL + Pandas with Synthetic Data

* * *

> **STAR Key:**
> 
>   * 🔵 **S** ituation — Context and background
>   * 🟡 **T** ask — Your specific responsibility
>   * 🟢 **A** ction — What you did with full working code
>   * 🔴 **R** esult — Measurable outcome
> 


> **Synthetic Dataset Used Throughout:** Tables: `claims`, `policies`, `adjusters`, `regions` Scenario: Insurance Claims Analytics — J&J Global Services / Travelers Insurance context

* * *

## 📦 Synthetic Data Setup
    
    
    # ─── Python: Create all synthetic datasets ───────────────────────────────────
    import pandas as pd
    import numpy as np
    
    np.random.seed(42)
    n = 200
    
    claims = pd.DataFrame({
        'claim_id':        range(1001, 1001 + n),
        'policy_id':       np.random.choice(range(201, 221), n),
        'adjuster_id':     np.random.choice(range(301, 311), n),
        'region_id':       np.random.choice(range(401, 406), n),
        'product_line':    np.random.choice(['Motor','Property','Health','Travel'], n),
        'claim_amount':    np.round(np.random.lognormal(9, 1.2, n), 2),
        'reserve_amount':  np.round(np.random.lognormal(9.3, 1.1, n), 2),
        'claim_date':      pd.date_range('2023-01-01', periods=n, freq='D').tolist()[:n],
        'settlement_date': pd.date_range('2023-02-01', periods=n, freq='D').tolist()[:n],
        'claim_status':    np.random.choice(['Open','Settled','Rejected','Pending'], n,
                                             p=[0.3,0.4,0.15,0.15]),
        'fraud_flag':      np.random.choice([0, 1], n, p=[0.92, 0.08]),
    })
    # Inject some NULLs
    claims.loc[claims.sample(10).index, 'reserve_amount'] = np.nan
    claims.loc[claims.sample(5).index,  'settlement_date'] = pd.NaT
    
    policies = pd.DataFrame({
        'policy_id':     range(201, 221),
        'policyholder':  [f'Holder_{i}' for i in range(201, 221)],
        'premium':       np.round(np.random.uniform(500, 5000, 20), 2),
        'start_date':    pd.date_range('2022-01-01', periods=20, freq='ME'),
        'end_date':      pd.date_range('2023-01-01', periods=20, freq='ME'),
        'risk_tier':     np.random.choice(['Low','Medium','High'], 20),
        'region_id':     np.random.choice(range(401, 406), 20),
    })
    
    adjusters = pd.DataFrame({
        'adjuster_id':   range(301, 311),
        'adjuster_name': [f'Adjuster_{i}' for i in range(301, 311)],
        'experience_yrs':np.random.randint(1, 20, 10),
        'team':          np.random.choice(['Alpha','Beta','Gamma'], 10),
        'region_id':     np.random.choice(range(401, 406), 10),
    })
    
    regions = pd.DataFrame({
        'region_id':   range(401, 406),
        'region_name': ['EMEA','APAC','NA','LATAM','MEA'],
        'country':     ['UK','Singapore','USA','Brazil','UAE'],
    })
    
    
    
    
    
    -- ─── SQL: DDL + INSERT (SQLite/PostgreSQL compatible) ────────────────────────
    CREATE TABLE claims (
        claim_id        INT PRIMARY KEY,
        policy_id       INT,
        adjuster_id     INT,
        region_id       INT,
        product_line    VARCHAR(20),
        claim_amount    DECIMAL(12,2),
        reserve_amount  DECIMAL(12,2),
        claim_date      DATE,
        settlement_date DATE,
        claim_status    VARCHAR(20),
        fraud_flag      INT
    );
    
    CREATE TABLE policies (
        policy_id     INT PRIMARY KEY,
        policyholder  VARCHAR(50),
        premium       DECIMAL(10,2),
        start_date    DATE,
        end_date      DATE,
        risk_tier     VARCHAR(10),
        region_id     INT
    );
    
    CREATE TABLE adjusters (
        adjuster_id    INT PRIMARY KEY,
        adjuster_name  VARCHAR(50),
        experience_yrs INT,
        team           VARCHAR(20),
        region_id      INT
    );
    
    CREATE TABLE regions (
        region_id   INT PRIMARY KEY,
        region_name VARCHAR(20),
        country     VARCHAR(30)
    );
    
    -- Sample inserts (abbreviated — full data matches Python synthetic set)
    INSERT INTO claims VALUES
    (1001,201,301,401,'Motor',   8432.50, 9200.00,'2023-01-01','2023-02-15','Settled',0),
    (1002,202,302,402,'Property',45231.80,50000.00,'2023-01-02','2023-03-01','Settled',1),
    (1003,203,301,403,'Health',  1200.00, 1500.00,'2023-01-03', NULL,       'Open',   0),
    (1004,204,305,401,'Travel',  3800.75, 4000.00,'2023-01-04','2023-02-20','Pending',0),
    (1005,205,302,404,'Motor',  22000.00,   NULL, '2023-01-05','2023-04-01','Rejected',1);
    -- ... (200 rows total)
    
    
    

* * *

## Section 1: FILTERING — Selecting Specific Rows

* * *

### Q1. How do you filter rows based on a single condition?

🔵 **Situation** At Travelers Insurance, the fraud team needed to pull only `Settled` claims from the claims table daily for their reconciliation report — the full dataset had all statuses mixed together.

🟡 **Task** Filter the claims DataFrame/table to return only settled claims.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT  claim_id,
            policy_id,
            claim_amount,
            claim_status,
            claim_date
    FROM    claims
    WHERE   claim_status = 'Settled';
    
    -- Output (sample):
    -- claim_id | policy_id | claim_amount | claim_status | claim_date
    -- ---------|-----------|--------------|--------------|------------
    --     1001 |       201 |      8432.50 | Settled      | 2023-01-01
    --     1002 |       202 |     45231.80 | Settled      | 2023-01-02
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    settled = claims[claims['claim_status'] == 'Settled']
    
    # Alternative: .query() — more readable for complex filters
    settled = claims.query("claim_status == 'Settled'")
    
    print(settled[['claim_id','policy_id','claim_amount','claim_status','claim_date']])
    # Output:
    #    claim_id  policy_id  claim_amount claim_status  claim_date
    #        1001        201       8432.50      Settled  2023-01-01
    #        1002        202      45231.80      Settled  2023-01-02
    
    
    

🔴 **Result** Filtering reduced the working dataset from 200 rows to ~80 rows (40% of total — matching the `Settled` probability), enabling the fraud team's daily reconciliation to run in under 30 seconds.

* * *

### Q2. How do you filter rows using multiple AND conditions?

🔵 **Situation** At LSEG, the risk team needed to isolate high-value Motor claims that were still open — to prioritise reserve reviews before month-end.

🟡 **Task** Return only `Open` Motor claims with `claim_amount > 10,000`.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT  claim_id,
            product_line,
            claim_amount,
            claim_status,
            reserve_amount
    FROM    claims
    WHERE   claim_status  = 'Open'
      AND   product_line  = 'Motor'
      AND   claim_amount  > 10000;
    
    -- Output (sample):
    -- claim_id | product_line | claim_amount | claim_status | reserve_amount
    -- ---------|--------------|--------------|--------------|---------------
    --     1021 | Motor        |     22150.00 | Open         |      25000.00
    --     1047 | Motor        |     18340.75 | Open         |      20000.00
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Method 1: Boolean mask with & operator
    mask = (
        (claims['claim_status'] == 'Open') &
        (claims['product_line'] == 'Motor') &
        (claims['claim_amount'] > 10_000)
    )
    high_risk_open = claims[mask]
    
    # Method 2: .query() — cleaner syntax
    high_risk_open = claims.query(
        "claim_status == 'Open' and product_line == 'Motor' and claim_amount > 10000"
    )
    
    print(high_risk_open[['claim_id','product_line','claim_amount',
                           'claim_status','reserve_amount']])
    # Output:
    #    claim_id product_line  claim_amount claim_status  reserve_amount
    #        1021        Motor      22150.00         Open        25000.00
    #        1047        Motor      18340.75         Open        20000.00
    
    
    

🔴 **Result** The multi-condition filter narrowed 200 claims to ~12 high-priority cases, allowing the reserve review team to focus their month-end effort and complete reviews 3 hours faster.

* * *

### Q3. How do you filter using OR conditions?

🔵 **Situation** At Travelers Insurance, compliance needed claims that were either `Rejected` or had `fraud_flag = 1` for a quarterly fraud audit report.

🟡 **Task** Return all claims where status is Rejected OR fraud was flagged.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT  claim_id,
            claim_status,
            fraud_flag,
            claim_amount,
            product_line
    FROM    claims
    WHERE   claim_status = 'Rejected'
       OR   fraud_flag   = 1
    ORDER BY fraud_flag DESC, claim_amount DESC;
    
    -- Output (sample):
    -- claim_id | claim_status | fraud_flag | claim_amount | product_line
    -- ---------|--------------|------------|--------------|-------------
    --     1002 | Settled      |          1 |     45231.80 | Property
    --     1005 | Rejected     |          1 |     22000.00 | Motor
    --     1018 | Rejected     |          0 |      9800.00 | Travel
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Method 1: | (OR) operator — must wrap each condition in parentheses
    audit = claims[
        (claims['claim_status'] == 'Rejected') |
        (claims['fraud_flag'] == 1)
    ]
    
    # Method 2: .query()
    audit = claims.query("claim_status == 'Rejected' or fraud_flag == 1")
    
    audit_sorted = audit.sort_values(
        ['fraud_flag','claim_amount'], ascending=[False, False]
    )
    
    print(audit_sorted[['claim_id','claim_status','fraud_flag',
                         'claim_amount','product_line']].head(5))
    
    
    

🔴 **Result** The OR filter captured 47 claims for the fraud audit (23.5% of total) — 16 purely rejected and 31 fraud-flagged — giving the compliance team a complete picture without missing any fraud-positive settled claims.

* * *

### Q4. How do you filter using `IN` / `NOT IN` for a list of values?

🔵 **Situation** At J&J Global Services, the actuarial team needed claims from only Motor and Property product lines for a combined reserve analysis — and separately needed to exclude Rejected and Pending statuses from the loss ratio report.

🟡 **Task** Use `IN` to include specific product lines, and `NOT IN` to exclude specific statuses.

🟢 **Action**
    
    
    -- ─── SQL: IN ─────────────────────────────────────────────────────────────────
    SELECT  claim_id,
            product_line,
            claim_amount,
            claim_status
    FROM    claims
    WHERE   product_line IN ('Motor', 'Property')
    ORDER BY product_line, claim_amount DESC;
    
    -- ─── SQL: NOT IN ──────────────────────────────────────────────────────────────
    -- CAUTION: NOT IN returns no rows if the list contains NULL
    -- Always use NOT EXISTS or filter NULLs first when the list may have NULLs
    SELECT  claim_id,
            claim_status,
            claim_amount
    FROM    claims
    WHERE   claim_status NOT IN ('Rejected', 'Pending')
      AND   claim_status IS NOT NULL;   -- defensive NULL guard
    
    -- Output (sample — NOT IN):
    -- claim_id | claim_status | claim_amount
    -- ---------|--------------|-------------
    --     1001 | Settled      |      8432.50
    --     1003 | Open         |      1200.00
    
    
    
    
    
    # ─── Pandas: isin() ──────────────────────────────────────────────────────────
    motor_property = claims[claims['product_line'].isin(['Motor', 'Property'])]
    
    # NOT IN: use ~ (tilde) to negate isin()
    active_claims = claims[
        ~claims['claim_status'].isin(['Rejected', 'Pending']) &
        claims['claim_status'].notna()   # defensive NULL guard
    ]
    
    # With .query()
    motor_property = claims.query("product_line in ['Motor','Property']")
    active_claims  = claims.query("claim_status not in ['Rejected','Pending']")
    
    print(f"Motor+Property: {len(motor_property)} rows")
    print(f"Active (not Rejected/Pending): {len(active_claims)} rows")
    
    # Output:
    # Motor+Property: 98 rows
    # Active (not Rejected/Pending): 145 rows
    
    
    

🔴 **Result** `IN` filtering replaced 4 separate queries for each product line, reducing code complexity by 75%. The `NOT IN` with NULL guard prevented a silent data loss bug that had been affecting the loss ratio calculation — 5 NULL-status claims were now correctly included.

* * *

### Q5. How do you filter on NULL values?

🔵 **Situation** At AXA Insurance, the data quality team needed to find all claims with missing `reserve_amount` (injected NULLs from the legacy system) — and separately, all claims that did NOT have a NULL `settlement_date`.

🟡 **Task** Filter for NULLs and non-NULLs across two different columns.

🟢 **Action**
    
    
    -- ─── SQL: IS NULL ────────────────────────────────────────────────────────────
    SELECT  claim_id,
            claim_amount,
            reserve_amount,
            claim_status
    FROM    claims
    WHERE   reserve_amount IS NULL;
    
    -- Output:
    -- claim_id | claim_amount | reserve_amount | claim_status
    -- ---------|--------------|----------------|-------------
    --     1005 |     22000.00 |           NULL | Rejected
    --     1023 |      3420.00 |           NULL | Open
    -- (10 rows — matching our synthetic NULL injection)
    
    -- ─── SQL: IS NOT NULL ────────────────────────────────────────────────────────
    SELECT  claim_id,
            claim_amount,
            settlement_date
    FROM    claims
    WHERE   settlement_date IS NOT NULL;
    -- Returns 195 rows (200 - 5 NaT injected)
    
    -- COALESCE: replace NULL with a default for reporting
    SELECT  claim_id,
            claim_amount,
            COALESCE(reserve_amount, claim_amount * 1.1) AS reserve_estimate,
            CASE WHEN reserve_amount IS NULL THEN 'Estimated' ELSE 'Actual' END AS reserve_source
    FROM    claims;
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # IS NULL
    null_reserves = claims[claims['reserve_amount'].isna()]
    print(f"Claims with NULL reserve: {len(null_reserves)}")
    
    # IS NOT NULL
    with_settlement = claims[claims['settlement_date'].notna()]
    print(f"Claims with settlement date: {len(with_settlement)}")
    
    # NULL audit across all columns
    null_audit = pd.DataFrame({
        'column':        claims.columns,
        'null_count':    claims.isnull().sum().values,
        'null_pct':      (claims.isnull().mean() * 100).round(2).values
    }).query("null_count > 0")
    print(null_audit)
    
    # Output:
    #            column  null_count  null_pct
    #    reserve_amount          10      5.00
    #   settlement_date           5      2.50
    
    # COALESCE equivalent: fillna()
    claims['reserve_estimate'] = claims['reserve_amount'].fillna(
        claims['claim_amount'] * 1.1
    )
    claims['reserve_source'] = claims['reserve_amount'].apply(
        lambda x: 'Estimated' if pd.isna(x) else 'Actual'
    )
    
    
    

🔴 **Result** NULL filtering identified 10 claims with missing reserves (5% of total) — all from the legacy Oracle system. The COALESCE/fillna estimate prevented zero-reserve records from distorting the loss ratio calculation by providing a conservative 110% estimate as a proxy.

* * *

### Q6. How do you filter using BETWEEN for ranges?

🔵 **Situation** At LSEG, the operations team needed claims filed within Q1 2023 (Jan–Mar) with amounts between £5,000 and £50,000 for a targeted portfolio review.

🟡 **Task** Filter on both a date range and a numeric range simultaneously.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT  claim_id,
            claim_date,
            claim_amount,
            product_line,
            claim_status
    FROM    claims
    WHERE   claim_date    BETWEEN '2023-01-01' AND '2023-03-31'   -- inclusive both ends
      AND   claim_amount  BETWEEN 5000          AND 50000
    ORDER BY claim_date;
    
    -- Output (sample):
    -- claim_id | claim_date | claim_amount | product_line | claim_status
    -- ---------|------------|--------------|--------------|-------------
    --     1001 | 2023-01-01 |      8432.50 | Motor        | Settled
    --     1004 | 2023-01-04 |      3800.75 | Travel       | Pending
    -- Note: 3800.75 is NOT returned (below 5000 threshold — correct)
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Date range filter using .between() — inclusive by default
    q1_filter = (
        claims['claim_date'].between('2023-01-01', '2023-03-31') &
        claims['claim_amount'].between(5_000, 50_000)
    )
    q1_claims = claims[q1_filter]
    
    # Alternative: explicit comparison operators
    q1_claims = claims[
        (claims['claim_date'] >= '2023-01-01') &
        (claims['claim_date'] <= '2023-03-31') &
        (claims['claim_amount'] >= 5_000) &
        (claims['claim_amount'] <= 50_000)
    ]
    
    print(q1_claims[['claim_id','claim_date','claim_amount',
                      'product_line','claim_status']])
    print(f"\nQ1 claims in £5K-£50K range: {len(q1_claims)}")
    
    # Output:
    # Q1 claims in £5K-£50K range: 48
    
    
    

🔴 **Result** The combined range filter narrowed the portfolio from 200 claims to 48 — a targeted cohort for the Q1 review. Processing time for the downstream reserve analysis dropped from 8 minutes to under 2 minutes on the reduced dataset.

* * *

### Q7. How do you filter using pattern matching (LIKE / str.contains)?

🔵 **Situation** At J&J Global Services, the data quality team needed to find all policies held by policyholders whose name contained "Motor" in the description field — a fuzzy match requirement after a system migration left inconsistent naming conventions.

🟡 **Task** Find records using partial string matching across name and description fields.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    -- % = any characters (wildcard)
    -- _ = exactly one character
    
    -- Starts with 'Holder_2'
    SELECT  policy_id, policyholder, risk_tier
    FROM    policies
    WHERE   policyholder LIKE 'Holder_2%';
    
    -- Contains '05' anywhere
    SELECT  policy_id, policyholder
    FROM    policies
    WHERE   policyholder LIKE '%05%';
    
    -- Exactly 8 characters: 'Holder_X'
    SELECT  policy_id, policyholder
    FROM    policies
    WHERE   policyholder LIKE 'Holder__';
    
    -- Case-insensitive (PostgreSQL: ILIKE; SQL Server: use LOWER())
    SELECT  policy_id, policyholder
    FROM    policies
    WHERE   LOWER(policyholder) LIKE '%holder%';
    
    -- Output (LIKE 'Holder_2%'):
    -- policy_id | policyholder | risk_tier
    -- ----------|--------------|----------
    --       201 | Holder_201   | Medium
    --       202 | Holder_202   | High
    
    
    
    
    
    # ─── Pandas: str.contains() ──────────────────────────────────────────────────
    # Default: case-sensitive, regex=True
    starts_with_2 = policies[policies['policyholder'].str.startswith('Holder_2')]
    
    contains_05   = policies[policies['policyholder'].str.contains('05', na=False)]
    
    # Case-insensitive search
    case_insensitive = policies[
        policies['policyholder'].str.contains('holder', case=False, na=False)
    ]
    
    # Regex pattern: policyholder ending in 1 or 2
    regex_match = policies[
        policies['policyholder'].str.contains(r'_2\d{2}$', regex=True, na=False)
    ]
    
    # str.match() — anchored at start (like LIKE 'pattern%')
    anchored = policies[policies['policyholder'].str.match(r'^Holder_20')]
    
    print(f"Starts with 'Holder_2': {len(starts_with_2)} rows")
    print(f"Contains '05': {len(contains_05)} rows")
    print(f"Regex ends in 2XX: {len(regex_match)} rows")
    
    # Output:
    # Starts with 'Holder_2': 20 rows (all — our 'Holder_201' to 'Holder_220')
    # Contains '05': 1 rows (Holder_205)
    # Regex ends in 2XX: 20 rows
    
    
    

🔴 **Result** Pattern matching identified 23 inconsistently named records from the legacy migration that exact-match filters had been missing. Correcting these improved the policy-to-claim join match rate from 91% to 99.5%.

* * *

## Section 2: AGGREGATION — Summarising Data

* * *

### Q8. How do you compute basic aggregations (COUNT, SUM, AVG, MIN, MAX)?

🔵 **Situation** At J&J Global Services, the weekly executive summary required five core metrics for each product line: total claims, total incurred amount, average claim size, smallest claim, and largest claim.

🟡 **Task** Compute all five aggregation metrics grouped by product line in a single query.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        product_line,
        COUNT(*)                        AS total_claims,
        COUNT(claim_amount)             AS non_null_amounts,   -- excludes NULLs
        SUM(claim_amount)               AS total_incurred,
        ROUND(AVG(claim_amount), 2)     AS avg_claim,
        MIN(claim_amount)               AS min_claim,
        MAX(claim_amount)               AS max_claim,
        ROUND(SUM(claim_amount) /
              NULLIF(SUM(reserve_amount), 0) * 100, 2) AS loss_ratio_pct
    FROM    claims
    GROUP BY product_line
    ORDER BY total_incurred DESC;
    
    -- Output:
    -- product_line | total_claims | total_incurred | avg_claim  | min_claim | max_claim | loss_ratio_pct
    -- -------------|--------------|----------------|------------|-----------|-----------|---------------
    -- Motor        |           52 |    1284320.45  |  24698.47  |   820.30  | 198430.00 |          87.3
    -- Property     |           48 |     980123.80  |  20419.25  |  1200.00  | 162000.00 |          89.1
    -- Health       |           51 |     421890.60  |   8272.37  |   350.50  |  89000.00 |          92.4
    -- Travel       |           49 |     201450.30  |   4111.23  |   120.00  |  45000.00 |          78.6
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Method 1: groupby + agg with named aggregations
    summary = claims.groupby('product_line').agg(
        total_claims    = ('claim_id',     'count'),
        non_null_amounts= ('claim_amount', 'count'),   # excludes NaN
        total_incurred  = ('claim_amount', 'sum'),
        avg_claim       = ('claim_amount', 'mean'),
        min_claim       = ('claim_amount', 'min'),
        max_claim       = ('claim_amount', 'max'),
        total_reserve   = ('reserve_amount','sum'),
    ).round(2)
    
    summary['loss_ratio_pct'] = (
        summary['total_incurred'] / summary['total_reserve'].replace(0, np.nan) * 100
    ).round(2)
    
    summary = summary.sort_values('total_incurred', ascending=False)
    print(summary)
    
    # Method 2: describe() for quick statistical summary
    print(claims.groupby('product_line')['claim_amount'].describe().round(2))
    
    # Output (describe):
    #               count         mean          std      min      25%       50%       75%       max
    # product_line
    # Health         51.0      8272.37     15234.21   350.50  2100.00   5400.00  12000.00  89000.00
    # Motor          52.0     24698.47     38421.90   820.30  5400.00  14200.00  38000.00 198430.00
    
    
    

🔴 **Result** Single-query aggregation replaced 5 separate reports. The executive summary was produced in under 3 seconds, and the loss ratio column immediately highlighted Health (92.4%) as a concern — triggering a pricing review that hadn't been on the agenda.

* * *

### Q9. How do you use GROUP BY with HAVING to filter aggregated results?

🔵 **Situation** At Travelers Insurance, the operations manager needed to identify adjusters who had handled more than 15 claims AND whose average claim amount exceeded £15,000 — to flag potential capacity or selection bias issues.

🟡 **Task** Aggregate by adjuster and filter the aggregated results using HAVING.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    -- WHERE filters rows BEFORE aggregation
    -- HAVING filters groups AFTER aggregation
    
    SELECT
        c.adjuster_id,
        a.adjuster_name,
        a.team,
        COUNT(*)                    AS claim_count,
        ROUND(AVG(c.claim_amount), 2) AS avg_claim,
        ROUND(SUM(c.claim_amount), 2) AS total_incurred,
        SUM(c.fraud_flag)           AS fraud_cases
    FROM    claims    c
    JOIN    adjusters a ON a.adjuster_id = c.adjuster_id
    WHERE   c.claim_status != 'Rejected'          -- pre-filter: exclude rejected
    GROUP BY c.adjuster_id, a.adjuster_name, a.team
    HAVING  COUNT(*)          > 15                -- AFTER aggregation
      AND   AVG(c.claim_amount) > 15000
    ORDER BY avg_claim DESC;
    
    -- Output:
    -- adjuster_id | adjuster_name | team  | claim_count | avg_claim  | total_incurred | fraud_cases
    -- ------------|---------------|-------|-------------|------------|----------------|------------
    --         302 | Adjuster_302  | Alpha |          18 |   28340.50 |      510129.00 |           3
    --         305 | Adjuster_305  | Beta  |          16 |   19870.25 |      317924.00 |           1
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # WHERE equivalent: pre-filter before groupby
    active_claims = claims[claims['claim_status'] != 'Rejected']
    
    # GROUP BY equivalent
    adjuster_stats = active_claims.groupby('adjuster_id').agg(
        claim_count   = ('claim_id',     'count'),
        avg_claim     = ('claim_amount', 'mean'),
        total_incurred= ('claim_amount', 'sum'),
        fraud_cases   = ('fraud_flag',   'sum'),
    ).round(2).reset_index()
    
    # HAVING equivalent: filter the aggregated DataFrame
    flagged_adjusters = adjuster_stats[
        (adjuster_stats['claim_count'] > 15) &
        (adjuster_stats['avg_claim']   > 15_000)
    ].merge(adjusters[['adjuster_id','adjuster_name','team']], on='adjuster_id')
    
    flagged_adjusters = flagged_adjusters.sort_values('avg_claim', ascending=False)
    print(flagged_adjusters)
    
    # Key insight: WHERE runs on rows → HAVING runs on groups
    # pd.query on the agg result = HAVING
    
    
    

🔴 **Result** HAVING filtering identified 2 adjusters handling disproportionately large claims. Investigation revealed one had a high-value commercial caseload (legitimate) while the other had a pattern of claim uplifts (flagged for review). The fraud_cases column provided immediate supporting context.

* * *

### Q10. How do you compute running totals and cumulative aggregations?

🔵 **Situation** At LSEG, the finance team needed a daily cumulative claims report showing the running total of incurred amounts — so the team could see at a glance how quickly annual reserves were being consumed.

🟡 **Task** Compute the daily claim total AND a running cumulative sum ordered by date.

🟢 **Action**
    
    
    -- ─── SQL: Window function for running total ───────────────────────────────────
    SELECT
        claim_date,
        COUNT(*)                                AS daily_claims,
        ROUND(SUM(claim_amount), 2)             AS daily_total,
        ROUND(SUM(SUM(claim_amount)) OVER (
            ORDER BY claim_date
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ), 2)                                   AS cumulative_total,
        ROUND(SUM(SUM(claim_amount)) OVER (
            ORDER BY claim_date
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) / 5000000 * 100, 2)                  AS pct_of_annual_budget
    FROM    claims
    GROUP BY claim_date
    ORDER BY claim_date;
    
    -- Output (sample):
    -- claim_date | daily_claims | daily_total | cumulative_total | pct_of_annual_budget
    -- -----------|--------------|-------------|------------------|---------------------
    -- 2023-01-01 |            1 |     8432.50 |         8432.50  |                 0.17
    -- 2023-01-02 |            1 |    45231.80 |        53664.30  |                 1.07
    -- 2023-01-03 |            1 |     1200.00 |        54864.30  |                 1.10
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Step 1: Daily aggregation
    daily = claims.groupby('claim_date').agg(
        daily_claims = ('claim_id',     'count'),
        daily_total  = ('claim_amount', 'sum'),
    ).reset_index().sort_values('claim_date')
    
    # Step 2: Cumulative sum (expanding window)
    daily['cumulative_total'] = daily['daily_total'].cumsum()
    
    # Step 3: Percentage of budget
    annual_budget = 5_000_000
    daily['pct_of_budget'] = (daily['cumulative_total'] / annual_budget * 100).round(2)
    
    # Cumulative min/max (useful for tracking record values)
    daily['running_max_daily'] = daily['daily_total'].cummax()
    daily['running_min_daily'] = daily['daily_total'].cummin()
    
    print(daily[['claim_date','daily_claims','daily_total',
                 'cumulative_total','pct_of_budget']].head(10))
    
    # Alternative: .expanding() for any aggregation
    daily['cumulative_avg'] = daily['daily_total'].expanding().mean()
    daily['cumulative_std'] = daily['daily_total'].expanding().std()
    
    
    

🔴 **Result** The cumulative report showed the annual budget would be exhausted by mid-September — 3.5 months ahead of year-end. This early warning triggered a reserve review in Q2 rather than Q4, providing 6 additional months to manage the exposure.

* * *

### Q11. How do you compute percentile and quantile aggregations?

🔵 **Situation** At J&J Global Services, the actuarial team needed to analyse the claim amount distribution for each product line — specifically the 25th, 50th, 75th percentiles and the 95th percentile (for tail risk assessment).

🟡 **Task** Compute multiple percentile values grouped by product line.

🟢 **Action**
    
    
    -- ─── SQL (PostgreSQL / SQL Server 2019+ syntax) ───────────────────────────────
    SELECT
        product_line,
        ROUND(PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY claim_amount), 2) AS p25,
        ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY claim_amount), 2) AS median,
        ROUND(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY claim_amount), 2) AS p75,
        ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY claim_amount), 2) AS p95,
        ROUND(AVG(claim_amount), 2)                                           AS mean_claim,
        ROUND(
            PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY claim_amount) -
            PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY claim_amount), 2
        )                                                                      AS iqr
    FROM    claims
    GROUP BY product_line
    ORDER BY median DESC;
    
    -- Output:
    -- product_line | p25      | median    | p75       | p95       | mean_claim | iqr
    -- -------------|----------|-----------|-----------|-----------|------------|----------
    -- Motor        | 5400.00  |  14200.00 |  38000.00 | 125000.00 |  24698.47  |  32600.00
    -- Property     | 4200.00  |  12800.00 |  32000.00 | 108000.00 |  20419.25  |  27800.00
    -- Health       | 1200.00  |   5400.00 |  14000.00 |  52000.00 |   8272.37  |  12800.00
    -- Travel       |  800.00  |   2800.00 |   9200.00 |  28000.00 |   4111.23  |   8400.00
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Method 1: groupby + quantile
    pct_25  = claims.groupby('product_line')['claim_amount'].quantile(0.25)
    pct_50  = claims.groupby('product_line')['claim_amount'].quantile(0.50)
    pct_75  = claims.groupby('product_line')['claim_amount'].quantile(0.75)
    pct_95  = claims.groupby('product_line')['claim_amount'].quantile(0.95)
    
    # Method 2: agg with multiple quantiles — more concise
    def percentile(n):
        """Factory function for named percentile aggregation."""
        def pct_fn(x):
            return x.quantile(n)
        pct_fn.__name__ = f'p{int(n*100)}'
        return pct_fn
    
    dist_analysis = claims.groupby('product_line')['claim_amount'].agg([
        percentile(0.25),
        percentile(0.50),
        percentile(0.75),
        percentile(0.95),
        'mean',
    ]).round(2)
    dist_analysis.columns = ['p25','median','p75','p95','mean']
    dist_analysis['iqr'] = (dist_analysis['p75'] - dist_analysis['p25']).round(2)
    
    print(dist_analysis.sort_values('median', ascending=False))
    
    # Quick way: describe() gives p25/p50/p75 automatically
    print(claims.groupby('product_line')['claim_amount'].describe(
        percentiles=[0.25, 0.5, 0.75, 0.95]
    ).round(2))
    
    
    

🔴 **Result** Percentile analysis revealed Motor's 95th percentile (£125K) was 9x its median — indicating extreme right-skew and high tail risk. This justified a separate large-loss reinsurance treaty for Motor, saving an estimated £800K annually in unprotected tail exposure.

* * *

## Section 3: SORTING — Ordering Results

* * *

### Q12. How do you sort by single and multiple columns?

🔵 **Situation** At Travelers Insurance, the daily claims triage report needed to present claims sorted by urgency: highest claim amount first, and within the same amount, by claim date (oldest first — longest waiting).

🟡 **Task** Sort the claims report by amount descending, then date ascending.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    -- Single column sort
    SELECT  claim_id, claim_amount, claim_date, claim_status
    FROM    claims
    ORDER BY claim_amount DESC;
    
    -- Multi-column sort: amount DESC, then date ASC (oldest first within same amount)
    SELECT
        claim_id,
        claim_amount,
        claim_date,
        claim_status,
        product_line
    FROM    claims
    WHERE   claim_status IN ('Open', 'Pending')
    ORDER BY claim_amount    DESC,    -- primary sort: highest first
             claim_date      ASC;     -- secondary: oldest first within same amount
    
    -- NULLs handling: NULLS LAST (PostgreSQL) or ISNULL trick (SQL Server)
    -- PostgreSQL:
    SELECT  claim_id, reserve_amount
    FROM    claims
    ORDER BY reserve_amount DESC NULLS LAST;
    
    -- SQL Server equivalent:
    SELECT  claim_id, reserve_amount
    FROM    claims
    ORDER BY CASE WHEN reserve_amount IS NULL THEN 1 ELSE 0 END,
             reserve_amount DESC;
    
    -- Output (Open/Pending, sorted):
    -- claim_id | claim_amount | claim_date | claim_status | product_line
    -- ---------|--------------|------------|--------------|-------------
    --     1021 |    198430.00 | 2023-01-21 | Open         | Motor
    --     1047 |    162000.00 | 2023-02-16 | Pending      | Property
    --     1003 |      1200.00 | 2023-01-03 | Open         | Health
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Single column
    sorted_single = claims.sort_values('claim_amount', ascending=False)
    
    # Multi-column: list of columns + list of ascending flags
    triage_report = (
        claims[claims['claim_status'].isin(['Open','Pending'])]
        .sort_values(
            by=['claim_amount', 'claim_date'],
            ascending=[False, True]    # amount DESC, date ASC
        )
        .reset_index(drop=True)
    )
    
    # NULLs last by default in pandas (ascending); NULLs first in descending
    # Control with na_position parameter
    sorted_with_nulls = claims.sort_values(
        'reserve_amount',
        ascending=False,
        na_position='last'     # NULLs at end regardless of sort direction
    )
    
    print(triage_report[['claim_id','claim_amount','claim_date',
                          'claim_status','product_line']].head(5))
    
    # Rank within groups (e.g., rank by claim amount within each product line)
    claims['amount_rank'] = claims.groupby('product_line')['claim_amount'].rank(
        method='dense', ascending=False
    )
    top3_per_product = claims[claims['amount_rank'] <= 3].sort_values(
        ['product_line','amount_rank']
    )
    
    
    

🔴 **Result** The correctly sorted triage report ensured that high-value, long-waiting claims were reviewed first. Average response time for high-value Open claims improved by 40% in the first month because adjusters no longer had to manually scan the list for priorities.

* * *

### Q13. How do you use RANK, DENSE_RANK, and ROW_NUMBER for ordered rankings?

🔵 **Situation** At J&J Global Services, the performance team needed to rank adjusters by total claims handled within each region — to identify top performers and support workload balancing decisions.

🟡 **Task** Apply all three ranking functions and explain when each is appropriate.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    WITH adjuster_totals AS (
        SELECT
            c.adjuster_id,
            r.region_name,
            COUNT(*)                    AS claim_count,
            ROUND(SUM(c.claim_amount),2) AS total_handled
        FROM    claims    c
        JOIN    adjusters a ON a.adjuster_id = c.adjuster_id
        JOIN    regions   r ON r.region_id   = a.region_id
        GROUP BY c.adjuster_id, r.region_name
    )
    SELECT
        adjuster_id,
        region_name,
        claim_count,
        total_handled,
        ROW_NUMBER() OVER (PARTITION BY region_name ORDER BY claim_count DESC) AS row_num,
        -- Unique sequential — no ties, arbitrary tiebreak
        RANK()       OVER (PARTITION BY region_name ORDER BY claim_count DESC) AS rank_num,
        -- Gaps on ties: 1,1,3 (skips 2 if two share rank 1)
        DENSE_RANK() OVER (PARTITION BY region_name ORDER BY claim_count DESC) AS dense_rank_num
        -- No gaps on ties: 1,1,2
    FROM    adjuster_totals
    ORDER BY region_name, claim_count DESC;
    
    -- Output (EMEA region):
    -- adjuster_id | region_name | claim_count | row_num | rank_num | dense_rank_num
    -- ------------|-------------|-------------|---------|----------|----------------
    --         301 | EMEA        |          22 |       1 |        1 |              1
    --         302 | EMEA        |          22 |       2 |        1 |              1  ← tie
    --         305 | EMEA        |          18 |       3 |        3 |              2  ← gap vs no-gap
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Compute adjuster totals with region join
    adjuster_totals = (
        claims
        .merge(adjusters[['adjuster_id','region_id']], on='adjuster_id')
        .merge(regions[['region_id','region_name']],  on='region_id')
        .groupby(['adjuster_id','region_name'], as_index=False)
        .agg(claim_count=('claim_id','count'), total_handled=('claim_amount','sum'))
    )
    
    # ROW_NUMBER: unique sequential integer (no ties)
    adjuster_totals['row_num'] = adjuster_totals.groupby('region_name')['claim_count'].rank(
        method='first', ascending=False
    ).astype(int)
    
    # RANK: gaps on ties (1,1,3)
    adjuster_totals['rank_num'] = adjuster_totals.groupby('region_name')['claim_count'].rank(
        method='min', ascending=False
    ).astype(int)
    
    # DENSE_RANK: no gaps on ties (1,1,2)
    adjuster_totals['dense_rank'] = adjuster_totals.groupby('region_name')['claim_count'].rank(
        method='dense', ascending=False
    ).astype(int)
    
    result = adjuster_totals.sort_values(['region_name','claim_count'],
                                          ascending=[True, False])
    print(result[['adjuster_id','region_name','claim_count',
                  'row_num','rank_num','dense_rank']].head(10))
    
    # When to use which:
    # ROW_NUMBER → need unique IDs (pagination, deduplication)
    # RANK       → sports-style ranking (tied 1st = two 1sts, next is 3rd)
    # DENSE_RANK → business ranking (tied 1st = two 1sts, next is 2nd)
    
    
    

🔴 **Result** Dense ranking exposed that 3 adjusters in EMEA were tied for top performance by claim count — but the total-handled column revealed one processed 40% more value. This nuance drove a more accurate performance bonus allocation.

* * *

## Section 4: JOINS — Combining Tables

* * *

### Q14. How do you perform an INNER JOIN to match records across two tables?

🔵 **Situation** At LSEG, the daily risk report needed claims data enriched with the policyholder name and premium amount — requiring a join between `claims` and `policies`.

🟡 **Task** Join claims with policies to produce an enriched risk report.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    -- INNER JOIN: returns only rows where the key exists in BOTH tables
    SELECT
        c.claim_id,
        c.claim_amount,
        c.claim_status,
        c.product_line,
        p.policyholder,
        p.premium,
        p.risk_tier,
        ROUND(c.claim_amount / NULLIF(p.premium, 0), 2) AS claim_to_premium_ratio
    FROM    claims   c
    INNER JOIN policies p ON p.policy_id = c.policy_id
    ORDER BY claim_to_premium_ratio DESC;
    
    -- Output (sample):
    -- claim_id | claim_amount | claim_status | product_line | policyholder | premium  | risk_tier | ratio
    -- ---------|--------------|--------------|--------------|--------------|----------|-----------|-------
    --     1002 |    45231.80  | Settled      | Property     | Holder_202   | 2100.00  | High      |  21.54
    --     1021 |    22000.00  | Open         | Motor        | Holder_205   | 1850.00  | High      |  11.89
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Inner join (default how='inner')
    enriched = pd.merge(
        claims,
        policies[['policy_id','policyholder','premium','risk_tier']],
        on='policy_id',
        how='inner',
        validate='many_to_one'    # validate: many claims per policy (expected)
    )
    
    enriched['claim_to_premium_ratio'] = (
        enriched['claim_amount'] / enriched['premium'].replace(0, np.nan)
    ).round(2)
    
    result = enriched.sort_values('claim_to_premium_ratio', ascending=False)
    
    print(result[['claim_id','claim_amount','claim_status','product_line',
                  'policyholder','premium','risk_tier',
                  'claim_to_premium_ratio']].head(5))
    
    # Merge audit
    print(f"\nClaims before join: {len(claims)}")
    print(f"Claims after inner join: {len(enriched)}")
    print(f"Claims dropped (no matching policy): {len(claims) - len(enriched)}")
    # If dropped > 0 → orphan claims detected
    
    
    

🔴 **Result** The enriched join revealed 3 claims with no matching policy record (orphaned claims from the legacy system) — confirmed by the `validate='many_to_one'` check. The claim-to-premium ratio column immediately flagged 7 policies where claims had already exceeded 10x the annual premium — triggering a policy review.

* * *

### Q15. How do you perform LEFT JOIN to retain all records from the left table?

🔵 **Situation** At AXA Insurance, the data team needed to identify policies that had NO associated claims — to understand the zero-claim cohort for profitability analysis.

🟡 **Task** Use LEFT JOIN to keep all policies, including those with no claims, and identify zero-claim policies.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    -- LEFT JOIN: ALL rows from left table (policies), matched rows from right (claims)
    -- Where no match: right-side columns are NULL
    SELECT
        p.policy_id,
        p.policyholder,
        p.premium,
        p.risk_tier,
        COUNT(c.claim_id)               AS claim_count,
        COALESCE(SUM(c.claim_amount),0) AS total_incurred,
        CASE WHEN COUNT(c.claim_id) = 0 THEN 'Zero Claim' ELSE 'Has Claims' END AS claim_status
    FROM    policies p
    LEFT JOIN claims  c ON c.policy_id = p.policy_id
    GROUP BY p.policy_id, p.policyholder, p.premium, p.risk_tier
    ORDER BY claim_count DESC;
    
    -- Output (sample):
    -- policy_id | policyholder | premium | risk_tier | claim_count | total_incurred | claim_status
    -- ----------|--------------|---------|-----------|-------------|----------------|-------------
    --       201 | Holder_201   | 2450.00 | Medium    |          12 |      89432.00  | Has Claims
    --       215 | Holder_215   | 3200.00 | Low       |           0 |           0.00 | Zero Claim
    --       218 | Holder_218   |  890.00 | Low       |           0 |           0.00 | Zero Claim
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # LEFT JOIN: keep all policies
    policy_claims = pd.merge(
        policies,
        claims[['policy_id','claim_id','claim_amount']],
        on='policy_id',
        how='left'   # all policies retained; NaN where no claim
    )
    
    # Aggregate: count and sum (NaN-safe)
    policy_summary = policy_claims.groupby(
        ['policy_id','policyholder','premium','risk_tier'],
        as_index=False
    ).agg(
        claim_count   = ('claim_id',     'count'),
        total_incurred= ('claim_amount', 'sum'),
    )
    # 'count' excludes NaN — so zero-claim policies get count=0 correctly
    policy_summary['total_incurred'] = policy_summary['total_incurred'].fillna(0)
    policy_summary['claim_category'] = np.where(
        policy_summary['claim_count'] == 0, 'Zero Claim', 'Has Claims'
    )
    
    zero_claim_policies = policy_summary[policy_summary['claim_count'] == 0]
    print(f"Zero-claim policies: {len(zero_claim_policies)} / {len(policies)}")
    print(f"Their avg premium: £{zero_claim_policies['premium'].mean():.2f}")
    
    # Output:
    # Zero-claim policies: 3 / 20
    # Their avg premium: £2140.00
    
    
    

🔴 **Result** LEFT JOIN identified 3 zero-claim policies with an average premium of £2,140 — contributing £6,420 in pure premium profit. This zero-claim cohort was used to build a retention model, reducing lapse rate in this profitable segment by 18%.

* * *

### Q16. How do you use FULL OUTER JOIN and handle unmatched rows from both sides?

🔵 **Situation** At J&J Global Services, after a system migration, the team needed to identify: (a) claims with no matching policy, AND (b) policies with no claims — to detect data integrity issues on both sides of the relationship.

🟡 **Task** Use FULL OUTER JOIN to detect orphaned records on both sides simultaneously.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        p.policy_id          AS policy_policy_id,
        c.policy_id          AS claim_policy_id,
        p.policyholder,
        c.claim_id,
        c.claim_amount,
        CASE
            WHEN p.policy_id IS NULL THEN 'Orphan Claim (no policy)'
            WHEN c.claim_id  IS NULL THEN 'Policy with no claims'
            ELSE                          'Matched'
        END AS match_status
    FROM    policies p
    FULL OUTER JOIN claims c ON c.policy_id = p.policy_id
    WHERE   p.policy_id IS NULL OR c.claim_id IS NULL   -- show only unmatched
    ORDER BY match_status;
    
    -- Output:
    -- policy_policy_id | claim_policy_id | policyholder | claim_id | match_status
    -- -----------------|-----------------|--------------|----------|-------------------------
    --             NULL |             999 | NULL         |     1198 | Orphan Claim (no policy)
    --              215 |            NULL | Holder_215   |     NULL | Policy with no claims
    
    
    
    
    
    # ─── Pandas: FULL OUTER JOIN ──────────────────────────────────────────────────
    full_join = pd.merge(
        policies[['policy_id','policyholder']],
        claims[['claim_id','policy_id','claim_amount']],
        on='policy_id',
        how='outer',
        indicator=True    # adds _merge column: 'left_only','right_only','both'
    )
    
    # Categorise
    full_join['match_status'] = full_join['_merge'].map({
        'left_only':  'Policy with no claims',
        'right_only': 'Orphan Claim (no policy)',
        'both':       'Matched'
    })
    
    # Summary
    print(full_join['match_status'].value_counts())
    # Output:
    # Matched                      197
    # Policy with no claims          3
    # Orphan Claim (no policy)       1
    
    # Show problem records
    problems = full_join[full_join['match_status'] != 'Matched']
    print(problems[['policy_id','policyholder','claim_id','match_status']])
    
    
    

🔴 **Result** FULL OUTER JOIN found 1 orphan claim (£45,231 with no policy record — a serious data integrity issue) and 3 zero-claim policies. The orphan claim was traced to a policy that had been deleted during the migration, and a remediation ticket was raised to restore the policy record before financial close.

* * *

### Q17. How do you perform a SELF JOIN to compare rows within the same table?

🔵 **Situation** At Travelers Insurance, the fraud team needed to find pairs of claims from the same policy where the second claim was filed within 30 days of the first — a common rapid re-claim fraud pattern.

🟡 **Task** Use a self-join on the claims table to identify rapid re-claim pairs.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        c1.claim_id           AS first_claim_id,
        c2.claim_id           AS second_claim_id,
        c1.policy_id,
        c1.claim_date         AS first_claim_date,
        c2.claim_date         AS second_claim_date,
        c2.claim_date - c1.claim_date AS days_between,
        c1.claim_amount       AS first_amount,
        c2.claim_amount       AS second_amount,
        c1.product_line
    FROM    claims c1
    JOIN    claims c2
        ON  c2.policy_id  = c1.policy_id            -- same policy
        AND c2.claim_id   > c1.claim_id             -- avoid self-matching and duplicates
        AND c2.claim_date BETWEEN c1.claim_date AND c1.claim_date + INTERVAL '30 days'
    WHERE   c1.claim_status != 'Rejected'
      AND   c2.claim_status != 'Rejected'
    ORDER BY days_between ASC;
    
    -- Output:
    -- first_claim_id | second_claim_id | policy_id | days_between | first_amount | second_amount
    -- ---------------|-----------------|-----------|--------------|--------------|---------------
    --           1003 |            1028 |       203 |            7 |      1200.00 |       2800.00
    --           1015 |            1041 |       208 |           14 |     15000.00 |      22000.00
    
    
    
    
    
    # ─── Pandas: Self-join ───────────────────────────────────────────────────────
    # Merge claims with itself on policy_id
    self_join = pd.merge(
        claims[['claim_id','policy_id','claim_date','claim_amount','claim_status','product_line']],
        claims[['claim_id','policy_id','claim_date','claim_amount','claim_status']],
        on='policy_id',
        suffixes=('_first','_second')
    )
    
    # Filter: second claim ID > first (avoid self-match & duplicates)
    # AND second claim within 30 days of first
    self_join = self_join[
        (self_join['claim_id_second'] > self_join['claim_id_first']) &
        (self_join['claim_status_first']  != 'Rejected') &
        (self_join['claim_status_second'] != 'Rejected')
    ].copy()
    
    self_join['days_between'] = (
        pd.to_datetime(self_join['claim_date_second']) -
        pd.to_datetime(self_join['claim_date_first'])
    ).dt.days
    
    rapid_reclaims = self_join[self_join['days_between'] <= 30].sort_values('days_between')
    
    print(f"Rapid re-claim pairs (≤30 days): {len(rapid_reclaims)}")
    print(rapid_reclaims[['claim_id_first','claim_id_second','policy_id',
                            'days_between','claim_amount_first',
                            'claim_amount_second']].head(5))
    
    
    

🔴 **Result** Self-join identified 8 rapid re-claim pairs — 4 were subsequently confirmed as fraud (second claim was for the same incident repackaged as a new event). The pattern detection flagged £67,000 in potentially fraudulent claims.

* * *

## Section 5: DEDUPLICATION — Removing Duplicate Records

* * *

### Q18. How do you identify and remove exact duplicate rows?

🔵 **Situation** At J&J Global Services, the Salesforce batch export produced duplicate rows when an account was modified twice in the same hour — causing the claims count and total incurred to be overstated by ~8%.

🟡 **Task** Identify all exact duplicate rows in the claims staging table and remove them, keeping one copy.

🟢 **Action**
    
    
    -- ─── SQL: Identify duplicates ────────────────────────────────────────────────
    -- Count exact duplicates (all columns identical)
    SELECT
        claim_id,
        policy_id,
        claim_amount,
        claim_date,
        COUNT(*) AS occurrence_count
    FROM    claims
    GROUP BY claim_id, policy_id, claim_amount, claim_date
    HAVING  COUNT(*) > 1;
    
    -- ─── SQL: Remove duplicates — keep one per claim_id (most recent load) ────────
    -- Method 1: ROW_NUMBER (SQL Server / PostgreSQL)
    WITH ranked AS (
        SELECT  *,
                ROW_NUMBER() OVER (
                    PARTITION BY claim_id
                    ORDER BY claim_date DESC        -- keep most recently dated row
                ) AS rn
        FROM    claims
    )
    DELETE FROM ranked WHERE rn > 1;
    
    -- Method 2: CREATE TABLE AS SELECT DISTINCT (simpler for full duplicates)
    CREATE TABLE claims_deduped AS
    SELECT DISTINCT * FROM claims;
    
    -- Output before: 200 rows (with 16 duplicates)
    -- Output after:  184 unique rows
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Inject some duplicate rows for demonstration
    claims_with_dupes = pd.concat([claims, claims.sample(16)], ignore_index=True)
    
    print(f"Total rows (with dupes):  {len(claims_with_dupes)}")
    print(f"Exact duplicates:         {claims_with_dupes.duplicated().sum()}")
    
    # Show which rows are duplicated
    dupes = claims_with_dupes[claims_with_dupes.duplicated(keep=False)]
    print(f"Rows involved in duplication: {len(dupes)}")
    
    # Method 1: drop_duplicates() — all columns must match
    deduped_all = claims_with_dupes.drop_duplicates()
    print(f"After drop_duplicates():  {len(deduped_all)}")
    
    # Method 2: deduplicate on business key only (claim_id)
    # keep='first' → keep earliest occurrence
    # keep='last'  → keep most recent occurrence
    # keep=False   → drop ALL copies (use when any duplicate = data error)
    deduped_key = claims_with_dupes.drop_duplicates(
        subset=['claim_id'],
        keep='last'    # keep most recently loaded version
    )
    print(f"After key-based dedup:    {len(deduped_key)}")
    
    # Step 3: Audit — log what was dropped
    dropped = claims_with_dupes[claims_with_dupes.duplicated(subset=['claim_id'], keep='last')]
    dropped.to_csv('dropped_duplicates_audit.csv', index=False)
    print(f"Dropped rows logged: {len(dropped)}")
    
    
    

🔴 **Result** Deduplication removed 16 duplicate records (8% of the staging dataset). Correcting these reduced the total incurred amount from £3.24M (inflated) to £2.98M (correct) — a £260K overstatement that would have distorted the quarterly loss ratio by 8.7 percentage points.

* * *

### Q19. How do you deduplicate keeping the most recent record per business key?

🔵 **Situation** At LSEG, the FX trade feed occasionally sent the same trade twice with updated status — both records had the same `trade_id` but different `modified_ts`. Only the most recently modified version should be kept.

🟡 **Task** Keep only the most recent version of each claim per `claim_id`, ordered by `claim_date`.

🟢 **Action**
    
    
    -- ─── SQL: ROW_NUMBER deduplication ───────────────────────────────────────────
    WITH latest_claims AS (
        SELECT  *,
                ROW_NUMBER() OVER (
                    PARTITION BY claim_id           -- group by business key
                    ORDER BY claim_date DESC,       -- most recent date first
                             claim_id DESC          -- tiebreak on ID
                ) AS row_num
        FROM    claims_staging   -- assume this has duplicates
    )
    SELECT  claim_id,
            policy_id,
            claim_amount,
            claim_status,
            claim_date,
            row_num
    FROM    latest_claims
    WHERE   row_num = 1;    -- keep only the most recent per claim_id
    
    -- ─── Verify: no duplicate claim_ids in output ─────────────────────────────────
    SELECT claim_id, COUNT(*) FROM (
        SELECT claim_id FROM latest_claims WHERE row_num = 1
    ) x
    GROUP BY claim_id
    HAVING COUNT(*) > 1;
    -- Should return 0 rows
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Simulate: some claims have multiple versions with different dates
    claims_v2 = claims.copy()
    extra_rows = claims.sample(10).copy()
    extra_rows['claim_date'] = pd.to_datetime(extra_rows['claim_date']) + pd.Timedelta(days=5)
    extra_rows['claim_status'] = 'Updated'
    staging = pd.concat([claims_v2, extra_rows], ignore_index=True)
    
    print(f"Staging rows: {len(staging)}")
    print(f"Duplicate claim_ids: {staging.duplicated('claim_id').sum()}")
    
    # Sort by business key + version indicator (most recent first)
    staging_sorted = staging.sort_values(
        ['claim_id', 'claim_date'],
        ascending=[True, False]    # claim_id ASC, date DESC
    )
    
    # Keep first occurrence per claim_id (= most recent after sort)
    latest = staging_sorted.drop_duplicates(subset=['claim_id'], keep='first')
    
    # Validate: zero duplicates
    assert latest.duplicated('claim_id').sum() == 0, "Deduplication failed!"
    print(f"After dedup: {len(latest)} unique claims")
    
    # Version comparison: what changed in the updated records?
    updated_ids = extra_rows['claim_id'].tolist()
    original  = claims[claims['claim_id'].isin(updated_ids)][['claim_id','claim_status','claim_date']]
    updated   = latest[latest['claim_id'].isin(updated_ids)][['claim_id','claim_status','claim_date']]
    comparison = original.merge(updated, on='claim_id', suffixes=('_original','_latest'))
    print(comparison)
    
    
    

🔴 **Result** The ROW_NUMBER deduplication correctly kept the most recent version of 10 updated claims, ensuring the status field reflected the latest state. Without this, 3 claims that had moved from `Open` to `Settled` would have remained `Open` in the report, understating the settlement rate by 1.5%.

* * *

### Q20. How do you find and handle near-duplicates using fuzzy matching logic?

🔵 **Situation** At J&J Global Services, policyholder names were stored inconsistently across legacy systems — "John Smith", "J. Smith", "JOHN SMITH" referred to the same person. Standard exact deduplication missed these, causing the same policyholder to appear as multiple records.

🟡 **Task** Identify near-duplicate policyholder names using normalisation and fuzzy matching.

🟢 **Action**
    
    
    -- ─── SQL: Normalise and flag potential duplicates ────────────────────────────
    WITH normalised AS (
        SELECT
            policy_id,
            policyholder,
            UPPER(TRIM(
                REPLACE(REPLACE(policyholder, '.', ''), '_', ' ')
            ))                AS normalised_name,
            premium
        FROM policies
    ),
    grouped AS (
        SELECT
            normalised_name,
            COUNT(*)          AS duplicate_count,
            STRING_AGG(CAST(policy_id AS VARCHAR), ', ') AS policy_ids,
            STRING_AGG(policyholder, ' | ')              AS original_names
        FROM   normalised
        GROUP BY normalised_name
        HAVING COUNT(*) > 1
    )
    SELECT * FROM grouped ORDER BY duplicate_count DESC;
    
    -- Output (hypothetical):
    -- normalised_name | duplicate_count | policy_ids  | original_names
    -- ----------------|-----------------|-------------|-------------------
    -- HOLDER 201      |               2 | 201, 215    | Holder_201 | Holder 201
    
    
    
    
    
    # ─── Pandas: Normalisation + fuzzy matching ───────────────────────────────────
    import re
    
    def normalise_name(name):
        """Standardise name for fuzzy comparison."""
        if pd.isna(name): return ''
        name = str(name).upper().strip()
        name = re.sub(r'[._\-]', ' ', name)   # replace separators with space
        name = re.sub(r'\s+', ' ', name)       # collapse multiple spaces
        return name
    
    policies['normalised_name'] = policies['policyholder'].apply(normalise_name)
    
    # Group by normalised name to find exact matches after normalisation
    name_groups = policies.groupby('normalised_name').agg(
        count       = ('policy_id', 'count'),
        policy_ids  = ('policy_id', list),
        orig_names  = ('policyholder', list)
    ).reset_index()
    
    near_dupes = name_groups[name_groups['count'] > 1]
    print(f"Near-duplicate groups found: {len(near_dupes)}")
    
    # Advanced: SOUNDEX / phonetic matching using jellyfish library
    try:
        import jellyfish
        policies['soundex'] = policies['policyholder'].apply(
            lambda x: jellyfish.soundex(normalise_name(x)) if x else ''
        )
        soundex_groups = policies.groupby('soundex').filter(lambda g: len(g) > 1)
        print(f"Phonetically similar groups: {soundex_groups['soundex'].nunique()}")
    except ImportError:
        print("Install jellyfish for phonetic matching: pip install jellyfish")
    
    # Levenshtein distance for pairwise comparison
    from itertools import combinations
    names = policies['policyholder'].tolist()
    potential_dupes = []
    for n1, n2 in combinations(names, 2):
        # Simple character-level similarity (no external library needed)
        common = sum(c1 == c2 for c1, c2 in zip(normalise_name(n1), normalise_name(n2)))
        similarity = common / max(len(normalise_name(n1)), len(normalise_name(n2)), 1)
        if similarity > 0.85 and n1 != n2:
            potential_dupes.append({'name_1': n1, 'name_2': n2, 'similarity': round(similarity,3)})
    
    pd.DataFrame(potential_dupes).head(10)
    
    
    

🔴 **Result** Normalisation-based near-duplicate detection found 4 policyholder pairs that were the same person under different naming conventions — consolidating them reduced the policy count from 20 to 16 (unique policyholders) and corrected the per-policyholder claim frequency metric from 10.0 to 12.5 per holder.

* * *

## Section 6: WINDOW FUNCTIONS — Advanced Row-Level Analytics

* * *

### Q21. How do you use LAG and LEAD to compare the current row with previous/next rows?

🔵 **Situation** At LSEG, the trading team needed to compare each day's total claims volume with the previous day — to detect sudden spikes that might indicate a data ingestion issue.

🟡 **Task** Compute daily claim totals and add previous-day and next-day comparisons using LAG and LEAD.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    WITH daily AS (
        SELECT
            claim_date,
            COUNT(*)                    AS daily_claims,
            ROUND(SUM(claim_amount), 2) AS daily_total
        FROM   claims
        GROUP BY claim_date
    )
    SELECT
        claim_date,
        daily_claims,
        daily_total,
        LAG(daily_total,  1, 0) OVER (ORDER BY claim_date) AS prev_day_total,
        LEAD(daily_total, 1, 0) OVER (ORDER BY claim_date) AS next_day_total,
        ROUND(daily_total -
              LAG(daily_total, 1) OVER (ORDER BY claim_date), 2) AS day_on_day_change,
        ROUND((daily_total -
               LAG(daily_total, 1) OVER (ORDER BY claim_date)) /
              NULLIF(LAG(daily_total, 1) OVER (ORDER BY claim_date), 0) * 100,
        2)                                                  AS pct_change
    FROM   daily
    ORDER BY claim_date;
    
    -- Output (sample):
    -- claim_date | daily_total | prev_day_total | day_on_day_change | pct_change
    -- -----------|-------------|----------------|-------------------|------------
    -- 2023-01-01 |     8432.50 |           0.00 |          8432.50  |       NULL
    -- 2023-01-02 |    45231.80 |        8432.50 |         36799.30  |     436.4%
    -- 2023-01-03 |     1200.00 |       45231.80 |        -44031.80  |     -97.3%  ← spike
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    daily = (
        claims.groupby('claim_date')
        .agg(daily_claims=('claim_id','count'), daily_total=('claim_amount','sum'))
        .reset_index()
        .sort_values('claim_date')
        .assign(claim_date=lambda x: pd.to_datetime(x['claim_date']))
    )
    
    # LAG = shift(1) — previous row
    daily['prev_day_total'] = daily['daily_total'].shift(1)
    
    # LEAD = shift(-1) — next row
    daily['next_day_total'] = daily['daily_total'].shift(-1)
    
    # Day-on-day change
    daily['dod_change'] = daily['daily_total'] - daily['prev_day_total']
    daily['pct_change'] = (
        daily['dod_change'] / daily['prev_day_total'].replace(0, np.nan) * 100
    ).round(2)
    
    # Flag anomalies: change > 300% or < -80%
    daily['is_spike'] = (daily['pct_change'].abs() > 300) | (daily['pct_change'] < -80)
    
    print(daily[['claim_date','daily_total','prev_day_total',
                 'dod_change','pct_change','is_spike']].head(10))
    print(f"\nAnomaly days detected: {daily['is_spike'].sum()}")
    
    
    

🔴 **Result** LAG/LEAD analysis flagged 3 days with >300% day-on-day changes — all traced to batch ingestion events where multiple days' worth of claims were loaded at once. The anomaly flag became part of the daily data quality dashboard, catching similar issues within 15 minutes of ingestion.

* * *

### Q22. How do you use PARTITION BY for group-level calculations while keeping row grain?

🔵 **Situation** At J&J Global Services, the reporting team needed each claim's amount alongside its product line's total and its percentage contribution — all in one row without collapsing to group level.

🟡 **Task** Use PARTITION BY window functions to enrich each row with group-level statistics while preserving the row grain.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        claim_id,
        product_line,
        claim_amount,
        claim_status,
        -- Group total without collapsing rows
        SUM(claim_amount)   OVER (PARTITION BY product_line)    AS product_total,
        AVG(claim_amount)   OVER (PARTITION BY product_line)    AS product_avg,
        COUNT(*)            OVER (PARTITION BY product_line)    AS product_count,
        -- Share of product total
        ROUND(claim_amount /
              SUM(claim_amount) OVER (PARTITION BY product_line) * 100, 2) AS pct_of_product,
        -- Global rank within product
        RANK() OVER (PARTITION BY product_line ORDER BY claim_amount DESC) AS rank_in_product,
        -- Running total within product ordered by date
        SUM(claim_amount) OVER (
            PARTITION BY product_line
            ORDER BY claim_date
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        )                                                        AS product_running_total
    FROM   claims
    ORDER BY product_line, claim_amount DESC;
    
    -- Output (sample — Motor only):
    -- claim_id | product_line | claim_amount | product_total | pct_of_product | rank_in_product
    -- ---------|--------------|--------------|---------------|----------------|----------------
    --     1021 | Motor        |   198430.00  |  1284320.45   |          15.45 |               1
    --     1047 | Motor        |   162000.00  |  1284320.45   |          12.61 |               2
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # groupby + transform: returns same-length output aligned to original DataFrame
    claims['product_total']  = claims.groupby('product_line')['claim_amount'].transform('sum')
    claims['product_avg']    = claims.groupby('product_line')['claim_amount'].transform('mean')
    claims['product_count']  = claims.groupby('product_line')['claim_amount'].transform('count')
    
    # Share of product total (row contribution)
    claims['pct_of_product'] = (
        claims['claim_amount'] / claims['product_total'] * 100
    ).round(2)
    
    # Rank within product line
    claims['rank_in_product'] = claims.groupby('product_line')['claim_amount'].rank(
        method='min', ascending=False
    ).astype(int)
    
    # Running total within product ordered by date
    claims_sorted = claims.sort_values(['product_line','claim_date'])
    claims_sorted['product_running_total'] = claims_sorted.groupby('product_line')[
        'claim_amount'
    ].cumsum()
    
    # Z-score within product (outlier detection per product)
    claims['product_zscore'] = claims.groupby('product_line')['claim_amount'].transform(
        lambda x: (x - x.mean()) / x.std()
    ).round(3)
    
    print(claims[['claim_id','product_line','claim_amount','product_total',
                  'pct_of_product','rank_in_product','product_zscore']].head(8))
    
    
    

🔴 **Result** The PARTITION BY enrichment enabled a single-pass analysis that previously required 4 separate aggregation queries merged back together. The `pct_of_product` column immediately revealed that the top 5 Motor claims represented 48% of all Motor incurred losses — driving a targeted large-loss reinsurance review.

* * *

## Section 7: DATE & TIME — Temporal Analysis

* * *

### Q23. How do you extract date parts and compute date differences?

🔵 **Situation** At AXA Insurance, the settlement performance team needed to analyse how long claims took to settle — broken down by month filed, day of week, and product line.

🟡 **Task** Extract date parts from `claim_date` and compute settlement lag in days.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        claim_id,
        claim_date,
        settlement_date,
        -- Date part extraction
        EXTRACT(YEAR  FROM claim_date)              AS claim_year,
        EXTRACT(MONTH FROM claim_date)              AS claim_month,
        EXTRACT(DOW   FROM claim_date)              AS day_of_week_num,  -- 0=Sun
        TO_CHAR(claim_date, 'Month')                AS month_name,       -- PostgreSQL
        DATENAME(WEEKDAY, claim_date)               AS day_name,         -- SQL Server
        -- Date arithmetic
        DATEDIFF(DAY, claim_date, settlement_date)  AS settlement_lag_days,  -- SQL Server
        settlement_date - claim_date                AS settlement_lag_days,  -- PostgreSQL
        -- Quarter
        EXTRACT(QUARTER FROM claim_date)            AS claim_quarter,
        CONCAT('Q', EXTRACT(QUARTER FROM claim_date),
               '-', EXTRACT(YEAR FROM claim_date))  AS quarter_label,
        -- Age of claim in days from today
        CURRENT_DATE - claim_date                   AS claim_age_days
    FROM   claims
    WHERE  settlement_date IS NOT NULL
    ORDER BY settlement_lag_days DESC;
    
    -- Output (sample):
    -- claim_id | claim_date | settlement_lag | claim_year | claim_month | day_of_week_num
    -- ---------|------------|----------------|------------|-------------|----------------
    --     1002 | 2023-01-02 |             58 |       2023 |           1 |               1
    --     1001 | 2023-01-01 |             45 |       2023 |           1 |               0
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Ensure datetime type
    claims['claim_date']      = pd.to_datetime(claims['claim_date'])
    claims['settlement_date'] = pd.to_datetime(claims['settlement_date'])
    
    # Date part extraction using .dt accessor
    claims['claim_year']    = claims['claim_date'].dt.year
    claims['claim_month']   = claims['claim_date'].dt.month
    claims['claim_quarter'] = claims['claim_date'].dt.quarter
    claims['day_of_week']   = claims['claim_date'].dt.dayofweek   # 0=Mon, 6=Sun
    claims['day_name']      = claims['claim_date'].dt.day_name()  # 'Monday','Tuesday'...
    claims['month_name']    = claims['claim_date'].dt.strftime('%B')  # 'January'
    claims['week_number']   = claims['claim_date'].dt.isocalendar().week.astype(int)
    claims['quarter_label'] = ('Q' + claims['claim_quarter'].astype(str) +
                               '-' + claims['claim_year'].astype(str))
    
    # Date differences
    claims['settlement_lag_days'] = (
        claims['settlement_date'] - claims['claim_date']
    ).dt.days  # returns NaN where settlement_date is NaT
    
    # Age of claim from today
    claims['claim_age_days'] = (pd.Timestamp.today() - claims['claim_date']).dt.days
    
    # Monthly settlement performance
    monthly_perf = claims.groupby(['claim_year','claim_month'])['settlement_lag_days'].agg(
        avg_lag='mean', median_lag='median', max_lag='max'
    ).round(1).reset_index()
    
    print(claims[['claim_id','claim_date','settlement_date','settlement_lag_days',
                  'day_name','quarter_label']].head(5))
    
    # Output:
    #    claim_id claim_date settlement_date  settlement_lag_days  day_name quarter_label
    #        1001 2023-01-01      2023-02-15                 45.0    Sunday       Q1-2023
    #        1002 2023-01-02      2023-03-01                 58.0    Monday       Q1-2023
    
    
    

🔴 **Result** Date analysis revealed that claims filed on Fridays had an average settlement lag 12 days longer than mid-week claims — a process bottleneck traced to reduced adjuster capacity on Fridays. Scheduling adjustment reduced Friday claim settlement lag by 8 days.

* * *

### Q24. How do you group by time periods (weekly, monthly, quarterly)?

🔵 **Situation** At Travelers Insurance, the finance team needed a monthly summary of claims incurred — and wanted to see quarterly subtotals alongside the monthly detail in the same report.

🟡 **Task** Aggregate claims by month and quarter, producing a hierarchical time-period summary.

🟢 **Action**
    
    
    -- ─── SQL: Monthly aggregation ────────────────────────────────────────────────
    SELECT
        DATE_TRUNC('month', claim_date)             AS month_start,  -- PostgreSQL
        -- SQL Server: DATEADD(MONTH, DATEDIFF(MONTH, 0, claim_date), 0)
        EXTRACT(YEAR  FROM claim_date)              AS year,
        EXTRACT(MONTH FROM claim_date)              AS month,
        CONCAT('Q', EXTRACT(QUARTER FROM claim_date)) AS quarter,
        COUNT(*)                                    AS claim_count,
        ROUND(SUM(claim_amount), 2)                 AS total_incurred,
        ROUND(AVG(claim_amount), 2)                 AS avg_claim,
        SUM(fraud_flag)                             AS fraud_count
    FROM   claims
    GROUP BY
        DATE_TRUNC('month', claim_date),
        EXTRACT(YEAR  FROM claim_date),
        EXTRACT(MONTH FROM claim_date),
        EXTRACT(QUARTER FROM claim_date)
    ORDER BY month_start;
    
    -- ─── SQL: ROLLUP for hierarchical totals ──────────────────────────────────────
    SELECT
        EXTRACT(YEAR    FROM claim_date) AS year,
        EXTRACT(QUARTER FROM claim_date) AS quarter,
        EXTRACT(MONTH   FROM claim_date) AS month,
        COUNT(*)                         AS claim_count,
        ROUND(SUM(claim_amount), 2)      AS total_incurred
    FROM   claims
    GROUP BY ROLLUP(
        EXTRACT(YEAR    FROM claim_date),
        EXTRACT(QUARTER FROM claim_date),
        EXTRACT(MONTH   FROM claim_date)
    )
    ORDER BY year, quarter, month;
    -- ROLLUP produces subtotals at each level + grand total (NULLs mark subtotal rows)
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    claims['claim_date'] = pd.to_datetime(claims['claim_date'])
    
    # Monthly grouping using Period
    claims['month_period']   = claims['claim_date'].dt.to_period('M')
    claims['quarter_period'] = claims['claim_date'].dt.to_period('Q')
    claims['week_period']    = claims['claim_date'].dt.to_period('W')
    
    # Monthly summary
    monthly = claims.groupby('month_period').agg(
        claim_count   = ('claim_id',    'count'),
        total_incurred= ('claim_amount','sum'),
        avg_claim     = ('claim_amount','mean'),
        fraud_count   = ('fraud_flag',  'sum'),
    ).round(2)
    monthly['fraud_rate'] = (monthly['fraud_count'] / monthly['claim_count'] * 100).round(1)
    
    # Quarterly summary using resample (requires DatetimeIndex)
    claims_indexed = claims.set_index('claim_date')
    quarterly = claims_indexed.resample('QE').agg(
        claim_count   = ('claim_id',    'count'),
        total_incurred= ('claim_amount','sum'),
        avg_claim     = ('claim_amount','mean'),
    ).round(2)
    
    # ROLLUP equivalent: multi-level groupby with subtotals
    def rollup_summary(df):
        """Produce month + quarter + grand total in one DataFrame."""
        monthly_agg  = df.groupby('month_period')['claim_amount'].sum().rename('monthly')
        quarterly_agg = df.groupby('quarter_period')['claim_amount'].sum().rename('quarterly')
        grand_total  = df['claim_amount'].sum()
    
        print(f"Monthly totals:\n{monthly_agg.round(2)}\n")
        print(f"Quarterly totals:\n{quarterly_agg.round(2)}\n")
        print(f"Grand total: £{grand_total:,.2f}")
    
    rollup_summary(claims)
    
    
    

🔴 **Result** Monthly aggregation revealed Q1 claims (Jan-Mar) were 35% higher than Q3 — a seasonal pattern that had previously been hidden in annual aggregate reports. This drove a seasonal pricing adjustment for Motor renewals, improving loss ratio by 4.2 percentage points in the following year.

* * *

## Section 8: STRING OPERATIONS — Text Data Manipulation

* * *

### Q25. How do you clean, trim, and standardise string columns?

🔵 **Situation** At J&J Global Services, the policyholder name field from the legacy system had leading/trailing spaces, mixed case, and special characters — causing join failures when matching against the CRM system.

🟡 **Task** Clean and standardise all string columns before the data warehouse load.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        policy_id,
        policyholder,
        TRIM(policyholder)                          AS trimmed,           -- remove spaces
        UPPER(TRIM(policyholder))                   AS uppercased,
        LOWER(TRIM(policyholder))                   AS lowercased,
        REPLACE(TRIM(policyholder), '_', ' ')       AS underscores_removed,
        LENGTH(TRIM(policyholder))                  AS name_length,
        LEFT(TRIM(policyholder), 6)                 AS first_6_chars,
        SUBSTRING(policyholder, 8, 3)               AS chars_8_to_10,
        CONCAT(TRIM(policyholder), ' [', CAST(policy_id AS VARCHAR), ']') AS labelled
    FROM   policies
    ORDER BY policy_id;
    
    -- Output (sample):
    -- policy_id | policyholder | trimmed    | uppercased | underscores_removed
    -- ----------|--------------|------------|------------|--------------------
    --       201 | Holder_201   | Holder_201 | HOLDER_201 | Holder 201
    --       202 | Holder_202   | Holder_202 | HOLDER_202 | Holder 202
    
    
    
    
    
    # ─── Pandas: String accessor methods ─────────────────────────────────────────
    # All via .str accessor — operates element-wise, NaN-safe
    p = policies.copy()
    
    p['trimmed']       = p['policyholder'].str.strip()          # remove whitespace
    p['upper']         = p['policyholder'].str.upper()
    p['lower']         = p['policyholder'].str.lower()
    p['title']         = p['policyholder'].str.title()          # Title Case
    p['no_underscore'] = p['policyholder'].str.replace('_', ' ', regex=False)
    p['length']        = p['policyholder'].str.len()
    p['first_6']       = p['policyholder'].str[:6]              # slice
    p['last_3']        = p['policyholder'].str[-3:]
    p['padded']        = p['policyholder'].str.zfill(12)        # zero-pad
    p['labelled']      = p['policyholder'] + ' [' + p['policy_id'].astype(str) + ']'
    
    # Regex-based cleaning
    p['alpha_only']    = p['policyholder'].str.replace(r'[^a-zA-Z\s]', '', regex=True)
    p['digits_only']   = p['policyholder'].str.extract(r'(\d+)', expand=False)  # extract numbers
    
    # Split on delimiter
    p[['holder_prefix','holder_num']] = p['policyholder'].str.split('_', expand=True, n=1)
    
    # Standardisation pipeline function
    def standardise_name(series):
        return (
            series
            .str.strip()
            .str.upper()
            .str.replace(r'[._\-]', ' ', regex=True)
            .str.replace(r'\s+', ' ', regex=True)
        )
    
    p['standardised'] = standardise_name(p['policyholder'])
    print(p[['policy_id','policyholder','standardised','no_underscore','digits_only']].head())
    
    
    

🔴 **Result** String standardisation increased the policy-to-CRM match rate from 84% to 99.2% — recovering 30 policies that had been failing to join due to whitespace and case inconsistencies. This prevented £189,000 in policy premium from being unreconciled in the quarterly report.

* * *

## Section 9: SUBQUERIES & CTEs — Modular Query Building

* * *

### Q26. How do you use CTEs to make complex queries readable and maintainable?

🔵 **Situation** At LSEG, the monthly risk report required a multi-step calculation: first aggregate claims by adjuster, then rank adjusters, then flag the top 20% as "high performers" — a 3-step logic that was previously one unreadable nested query.

🟡 **Task** Refactor the nested query into a clear CTE chain.

🟢 **Action**
    
    
    -- ─── SQL: CTE chain ───────────────────────────────────────────────────────────
    -- Step 1: Aggregate claims per adjuster
    WITH adjuster_totals AS (
        SELECT
            adjuster_id,
            COUNT(*)                    AS claim_count,
            ROUND(SUM(claim_amount),2)  AS total_handled,
            ROUND(AVG(claim_amount),2)  AS avg_claim,
            SUM(fraud_flag)             AS fraud_cases
        FROM   claims
        WHERE  claim_status != 'Rejected'
        GROUP BY adjuster_id
    ),
    -- Step 2: Rank adjusters by volume
    adjuster_ranked AS (
        SELECT
            *,
            PERCENT_RANK() OVER (ORDER BY claim_count DESC) AS performance_percentile,
            NTILE(5)       OVER (ORDER BY claim_count DESC) AS performance_quintile
        FROM   adjuster_totals
    ),
    -- Step 3: Classify and join with adjuster details
    final_report AS (
        SELECT
            ar.*,
            a.adjuster_name,
            a.team,
            a.experience_yrs,
            CASE
                WHEN ar.performance_quintile = 1 THEN 'Top 20%'
                WHEN ar.performance_quintile = 2 THEN 'Upper Mid'
                WHEN ar.performance_quintile = 3 THEN 'Middle'
                WHEN ar.performance_quintile = 4 THEN 'Lower Mid'
                ELSE                                  'Bottom 20%'
            END AS performance_band
        FROM   adjuster_ranked ar
        JOIN   adjusters a ON a.adjuster_id = ar.adjuster_id
    )
    SELECT * FROM final_report ORDER BY performance_quintile, claim_count DESC;
    
    -- Output (sample):
    -- adjuster_id | claim_count | total_handled | performance_band | adjuster_name | team
    -- ------------|-------------|---------------|------------------|---------------|------
    --         302 |          22 |    584320.00  | Top 20%          | Adjuster_302  | Alpha
    --         305 |          19 |    421890.00  | Top 20%          | Adjuster_305  | Beta
    
    
    
    
    
    # ─── Pandas: CTE equivalent using variable assignment ─────────────────────────
    # Step 1: Aggregate (CTE 1 equivalent)
    adjuster_totals = (
        claims[claims['claim_status'] != 'Rejected']
        .groupby('adjuster_id')
        .agg(
            claim_count   = ('claim_id',    'count'),
            total_handled = ('claim_amount','sum'),
            avg_claim     = ('claim_amount','mean'),
            fraud_cases   = ('fraud_flag',  'sum'),
        )
        .round(2)
        .reset_index()
    )
    
    # Step 2: Rank (CTE 2 equivalent)
    adjuster_totals['performance_pct'] = (
        adjuster_totals['claim_count'].rank(pct=True, ascending=False)
    )
    adjuster_totals['performance_quintile'] = pd.qcut(
        adjuster_totals['claim_count'],
        q=5, labels=[1,2,3,4,5]
    ).astype(int)
    
    # Step 3: Classify + join (CTE 3 equivalent)
    band_map = {1:'Top 20%', 2:'Upper Mid', 3:'Middle', 4:'Lower Mid', 5:'Bottom 20%'}
    adjuster_totals['performance_band'] = adjuster_totals['performance_quintile'].map(band_map)
    
    final_report = (
        adjuster_totals
        .merge(adjusters[['adjuster_id','adjuster_name','team','experience_yrs']],
               on='adjuster_id')
        .sort_values(['performance_quintile','claim_count'], ascending=[True,False])
    )
    
    print(final_report[['adjuster_id','claim_count','total_handled',
                         'performance_band','adjuster_name','team']].to_string())
    
    
    

🔴 **Result** The CTE refactor reduced the query from 67 lines of nested code to a clearly readable 3-step chain. Review time for the monthly report query dropped from 20 minutes to 5 minutes — and a previously hidden bug (rejected claims being included in totals) was caught during refactoring.

* * *

## Section 10: PIVOTING & RESHAPING — Restructuring Data

* * *

### Q27. How do you pivot data from long format to wide format?

🔵 **Situation** At J&J Global Services, the executive dashboard needed a cross-tab showing total claim amounts for each `product_line` (rows) × `claim_status` (columns) — data that was stored in long format (one row per claim).

🟡 **Task** Pivot the long-format claims data into a wide cross-tab for the dashboard.

🟢 **Action**
    
    
    -- ─── SQL: CASE-based pivot (works in all SQL dialects) ───────────────────────
    SELECT
        product_line,
        ROUND(SUM(CASE WHEN claim_status = 'Open'     THEN claim_amount ELSE 0 END), 2) AS Open,
        ROUND(SUM(CASE WHEN claim_status = 'Settled'  THEN claim_amount ELSE 0 END), 2) AS Settled,
        ROUND(SUM(CASE WHEN claim_status = 'Rejected' THEN claim_amount ELSE 0 END), 2) AS Rejected,
        ROUND(SUM(CASE WHEN claim_status = 'Pending'  THEN claim_amount ELSE 0 END), 2) AS Pending,
        ROUND(SUM(claim_amount), 2)                                                      AS Grand_Total
    FROM   claims
    GROUP BY product_line
    ORDER BY Grand_Total DESC;
    
    -- SQL Server PIVOT syntax (alternative):
    SELECT  product_line, [Open], [Settled], [Rejected], [Pending]
    FROM (
        SELECT product_line, claim_status, claim_amount FROM claims
    ) src
    PIVOT (
        SUM(claim_amount)
        FOR claim_status IN ([Open], [Settled], [Rejected], [Pending])
    ) pvt;
    
    -- Output:
    -- product_line | Open       | Settled    | Rejected  | Pending   | Grand_Total
    -- -------------|------------|------------|-----------|-----------|-------------
    -- Motor        | 312450.00  | 821300.00  | 89430.00  | 61140.45  | 1284320.45
    -- Property     | 245100.00  | 612800.00  | 72023.80  | 50200.00  |  980123.80
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Method 1: pivot_table — most flexible
    pivot = pd.pivot_table(
        claims,
        values='claim_amount',
        index='product_line',
        columns='claim_status',
        aggfunc='sum',
        fill_value=0,
        margins=True,          # adds Grand Total row/column
        margins_name='TOTAL'
    ).round(2)
    
    print(pivot)
    
    # Method 2: crosstab — for frequency counts
    ct_count = pd.crosstab(
        claims['product_line'],
        claims['claim_status'],
        margins=True, margins_name='TOTAL'
    )
    print(ct_count)
    
    # Method 3: crosstab with values (for amounts)
    ct_amount = pd.crosstab(
        claims['product_line'],
        claims['claim_status'],
        values=claims['claim_amount'],
        aggfunc='sum'
    ).fillna(0).round(2)
    
    # Normalise: percentage of row total
    ct_pct = pd.crosstab(
        claims['product_line'],
        claims['claim_status'],
        normalize='index'   # 'index'=row%, 'columns'=col%, True=overall%
    ).round(3) * 100
    
    print("\nProduct-Status mix (% of product total):")
    print(ct_pct)
    
    # Output (crosstab count):
    # claim_status  Open  Settled  Rejected  Pending  TOTAL
    # product_line
    # Health          15       22         8       6     51
    # Motor           16       21         9       6     52
    # Property        14       19         8       7     48
    # Travel          15       18         9       7     49
    # TOTAL           60       80        34      26    200
    
    
    

🔴 **Result** The pivot table revealed that Property claims had the highest Rejected rate (16.7%) — more than double Motor (17.3%) — suggesting overly aggressive underwriting controls. This drove a policy review that reduced unnecessary rejections by 30% in the following quarter, improving customer satisfaction scores.

* * *

## Section 11: NULL HANDLING — Working with Missing Data

* * *

### Q28. How do you handle NULLs in calculations using COALESCE, NULLIF, and ISNULL?

🔵 **Situation** At Travelers Insurance, the loss ratio calculation was returning NULL for 10 claims where `reserve_amount` was missing — causing the aggregate loss ratio to be understated because NULLs were excluded from SUM calculations.

🟡 **Task** Use NULL-handling functions to provide default values for missing reserves and prevent division-by-zero errors.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        claim_id,
        claim_amount,
        reserve_amount,
        -- COALESCE: return first non-NULL value
        COALESCE(reserve_amount, claim_amount * 1.10, 0)    AS reserve_estimate,
    
        -- ISNULL (SQL Server) / IFNULL (MySQL) — two-arg shorthand for COALESCE
        ISNULL(reserve_amount, claim_amount * 1.10)         AS reserve_isnull,
    
        -- NULLIF: return NULL if two values are equal (useful to avoid div/0)
        -- NULLIF(denominator, 0) returns NULL instead of causing error
        ROUND(claim_amount / NULLIF(reserve_amount, 0), 4)  AS claim_reserve_ratio,
    
        -- Safe loss ratio with NULL and zero protection
        ROUND(
            COALESCE(claim_amount, 0) /
            NULLIF(COALESCE(reserve_amount, claim_amount * 1.10), 0)
            * 100, 2
        )                                                   AS safe_loss_ratio_pct,
    
        -- Flag whether the reserve was imputed
        CASE WHEN reserve_amount IS NULL THEN 1 ELSE 0 END  AS is_imputed
    FROM   claims
    ORDER BY is_imputed DESC, claim_amount DESC;
    
    -- Output (NULL reserve rows):
    -- claim_id | claim_amount | reserve_amount | reserve_estimate | is_imputed
    -- ---------|--------------|----------------|------------------|------------
    --     1005 |    22000.00  |           NULL |        24200.00  |          1
    --     1023 |     3420.00  |           NULL |         3762.00  |          1
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # COALESCE equivalent: fillna() with multiple fallbacks
    claims['reserve_estimate'] = claims['reserve_amount'].fillna(
        claims['claim_amount'] * 1.10   # first fallback
    ).fillna(0)                          # second fallback (if claim_amount also null)
    
    # NULLIF equivalent: replace a specific value with NaN before division
    # np.where or .replace(0, np.nan)
    claims['claim_reserve_ratio'] = (
        claims['claim_amount'] /
        claims['reserve_amount'].replace(0, np.nan)   # NULLIF(reserve_amount, 0)
    ).round(4)
    
    # Safe loss ratio with full null/zero protection
    claims['safe_loss_ratio_pct'] = (
        claims['claim_amount'].fillna(0) /
        claims['reserve_estimate'].replace(0, np.nan)
        * 100
    ).round(2)
    
    # Flag imputed values
    claims['is_imputed'] = claims['reserve_amount'].isna().astype(int)
    
    # NULL summary by column
    print("NULL audit:")
    print(claims[['claim_amount','reserve_amount','settlement_date']].isnull().sum())
    print(f"\nImputed reserves: {claims['is_imputed'].sum()} claims")
    
    # COALESCE with multiple columns (pick first non-null across columns)
    claims['best_contact'] = claims['settlement_date'].combine_first(claims['claim_date'])
    # combine_first: returns left value where not NaN, else right value
    
    # Output:
    # NULL audit:
    # claim_amount      0
    # reserve_amount   10
    # settlement_date   5
    # Imputed reserves: 10 claims
    
    
    

🔴 **Result** NULL-safe calculations corrected the aggregate loss ratio from 84.3% (NULLs excluded from reserve total) to 87.1% (NULLs imputed at 110% of claim amount). This 2.8 percentage point correction was material for the quarterly regulatory reserve adequacy test.

* * *

## Section 12: CASE WHEN — Conditional Logic

* * *

### Q29. How do you use CASE WHEN for conditional classification and bucketing?

🔵 **Situation** At Travelers Insurance, claims needed to be classified into risk tiers based on amount and fraud history — for routing to different adjuster teams (standard, complex, or fraud investigation).

🟡 **Task** Apply multi-condition CASE WHEN logic to classify each claim into a routing tier.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        claim_id,
        claim_amount,
        fraud_flag,
        product_line,
        claim_status,
        -- Simple CASE: single column comparison
        CASE claim_status
            WHEN 'Settled'  THEN 'Closed'
            WHEN 'Rejected' THEN 'Closed'
            ELSE                 'Active'
        END AS status_group,
    
        -- Searched CASE: arbitrary conditions
        CASE
            WHEN fraud_flag = 1 AND claim_amount > 20000 THEN 'Priority-1: Fraud + High Value'
            WHEN fraud_flag = 1                          THEN 'Priority-2: Fraud'
            WHEN claim_amount > 50000                    THEN 'Priority-3: High Value'
            WHEN claim_amount > 10000                    THEN 'Priority-4: Medium Value'
            WHEN claim_status = 'Pending'                THEN 'Priority-5: Pending Review'
            ELSE                                              'Priority-6: Standard'
        END AS routing_tier,
    
        -- Amount banding
        CASE
            WHEN claim_amount <  1000              THEN 'XS (<£1K)'
            WHEN claim_amount <  5000              THEN 'S  (£1K-£5K)'
            WHEN claim_amount < 20000              THEN 'M  (£5K-£20K)'
            WHEN claim_amount < 50000              THEN 'L  (£20K-£50K)'
            ELSE                                        'XL (>£50K)'
        END AS amount_band
    FROM   claims
    ORDER BY
        CASE routing_tier
            WHEN 'Priority-1: Fraud + High Value' THEN 1
            WHEN 'Priority-2: Fraud'              THEN 2
            WHEN 'Priority-3: High Value'         THEN 3
            ELSE                                       4
        END;
    
    -- Output (sample):
    -- claim_id | claim_amount | fraud_flag | routing_tier                   | amount_band
    -- ---------|--------------|------------|--------------------------------|-------------
    --     1002 |    45231.80  |          1 | Priority-1: Fraud + High Value | L (£20K-£50K)
    --     1005 |    22000.00  |          1 | Priority-1: Fraud + High Value | L (£20K-£50K)
    --     1021 |   198430.00  |          0 | Priority-3: High Value         | XL (>£50K)
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Simple CASE equivalent: map()
    claims['status_group'] = claims['claim_status'].map({
        'Settled': 'Closed', 'Rejected': 'Closed',
        'Open': 'Active', 'Pending': 'Active'
    }).fillna('Unknown')
    
    # Searched CASE: np.select (vectorised — preferred for large DataFrames)
    conditions = [
        (claims['fraud_flag'] == 1) & (claims['claim_amount'] > 20_000),
        claims['fraud_flag'] == 1,
        claims['claim_amount'] > 50_000,
        claims['claim_amount'] > 10_000,
        claims['claim_status'] == 'Pending',
    ]
    choices = [
        'Priority-1: Fraud + High Value',
        'Priority-2: Fraud',
        'Priority-3: High Value',
        'Priority-4: Medium Value',
        'Priority-5: Pending Review',
    ]
    claims['routing_tier'] = np.select(conditions, choices, default='Priority-6: Standard')
    
    # Amount banding: pd.cut (cleaner than np.select for range bins)
    claims['amount_band'] = pd.cut(
        claims['claim_amount'],
        bins=[0, 1_000, 5_000, 20_000, 50_000, np.inf],
        labels=['XS (<£1K)', 'S (£1K-£5K)', 'M (£5K-£20K)', 'L (£20K-£50K)', 'XL (>£50K)'],
        include_lowest=True
    )
    
    # Sort by priority tier
    tier_order = {
        'Priority-1: Fraud + High Value': 1,
        'Priority-2: Fraud': 2,
        'Priority-3: High Value': 3,
        'Priority-4: Medium Value': 4,
        'Priority-5: Pending Review': 5,
        'Priority-6: Standard': 6,
    }
    claims['tier_sort'] = claims['routing_tier'].map(tier_order)
    prioritised = claims.sort_values('tier_sort')
    
    # Summary
    print(claims['routing_tier'].value_counts().sort_index())
    print(claims['amount_band'].value_counts())
    
    
    

🔴 **Result** The routing tier classification automated manual claim triage that had taken 2 hours each morning. Priority-1 claims (fraud + high value) were now routed to specialist adjusters within 15 minutes of ingestion — improving fraud detection response time by 73%.

* * *

## Section 13: DATA TYPE CONVERSIONS

* * *

### Q30. How do you cast and convert data types safely?

🔵 **Situation** At AXA Insurance, the legacy system export stored all fields as VARCHAR — `claim_amount` was a string with currency symbols (e.g., "£8,432.50"), and dates were in non-standard formats ("01-Jan-2023"). These needed conversion before any numeric or date calculation.

🟡 **Task** Strip, parse, and cast all malformed string columns to their correct data types.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    -- Safe cast with error handling
    SELECT
        claim_id,
        raw_amount,
        -- Remove currency symbol and commas, then cast
        CAST(REPLACE(REPLACE(raw_amount, '£', ''), ',', '') AS DECIMAL(12,2)) AS clean_amount,
    
        -- Safe cast: TRY_CAST returns NULL on failure (SQL Server 2012+)
        TRY_CAST(raw_amount AS DECIMAL(12,2))                                  AS safe_amount,
    
        -- Date parsing: convert 'DD-Mon-YYYY' to DATE
        CONVERT(DATE, raw_date, 106)                                           AS parsed_date, -- SQL Server
        TO_DATE(raw_date, 'DD-Mon-YYYY')                                       AS parsed_date, -- PostgreSQL
    
        -- Integer to string (for concatenation)
        CAST(claim_id AS VARCHAR(10)) + '-' + raw_status                       AS claim_label,
    
        -- Boolean-like: integer flag to text
        CASE CAST(fraud_flag AS INT) WHEN 1 THEN 'Yes' ELSE 'No' END          AS is_fraud
    
    FROM   claims_staging;
    
    -- Type-check columns in information_schema
    SELECT column_name, data_type, character_maximum_length
    FROM   information_schema.columns
    WHERE  table_name = 'claims'
    ORDER BY ordinal_position;
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Simulate dirty data from legacy export
    dirty = pd.DataFrame({
        'claim_id':     ['1001', '1002', '1003'],
        'raw_amount':   ['£8,432.50', '£45,231.80', 'N/A'],
        'raw_date':     ['01-Jan-2023', '02-Jan-2023', 'INVALID'],
        'raw_flag':     ['True', 'False', 'true'],
        'raw_status':   ['  Settled ', 'OPEN', 'rejected  '],
    })
    
    # 1. Numeric: strip symbols and cast
    dirty['clean_amount'] = (
        dirty['raw_amount']
        .str.replace('£', '', regex=False)
        .str.replace(',', '', regex=False)
        .pipe(pd.to_numeric, errors='coerce')  # errors='coerce' → NaN on failure
    )
    
    # 2. Date parsing with format
    dirty['clean_date'] = pd.to_datetime(
        dirty['raw_date'],
        format='%d-%b-%Y',   # day-MonthAbbrev-Year
        errors='coerce'       # 'INVALID' → NaT
    )
    
    # 3. Boolean: multiple string representations
    bool_map = {'true': True, 'True': True, 'false': False, 'False': False}
    dirty['clean_flag'] = dirty['raw_flag'].str.lower().map(bool_map)
    
    # 4. String normalisation
    dirty['clean_status'] = dirty['raw_status'].str.strip().str.title()
    
    # 5. Integer from string
    dirty['clean_id'] = dirty['claim_id'].astype(int)
    
    # 6. Categorical (memory efficient for low-cardinality strings)
    dirty['status_cat'] = dirty['clean_status'].astype('category')
    
    # Type audit
    print(dirty.dtypes)
    print(dirty)
    
    # Output:
    # clean_amount: float64  (NaN for 'N/A')
    # clean_date:   datetime64[ns]  (NaT for 'INVALID')
    # clean_flag:   object (bool)
    # clean_status: object
    # Output data:
    #    clean_id  clean_amount clean_date  clean_flag clean_status
    #        1001       8432.50 2023-01-01        True      Settled
    #        1002      45231.80 2023-01-02       False         Open
    #        1003           NaN        NaT        True     Rejected
    
    
    

🔴 **Result** Type conversion cleaned 100% of the legacy export, replacing 3 separate manual scripts (one per field type) with a single automated pipeline. The `errors='coerce'` pattern identified 12 genuinely unparseable records that were routed to a data quality exception queue for manual review.

* * *

## _[Continued in Part 2: UNION/UNION ALL, Subqueries, Stored Procedures, Index Optimisation, EXISTS, MERGE, Pivot/Unpivot, Statistical Functions, Data Profiling, and Final Best Practices]_

## Section 14: UNION & SET OPERATIONS — Combining Result Sets

* * *

### Q31. How do you combine result sets using UNION, UNION ALL, INTERSECT, and EXCEPT?

🔵 **Situation** At J&J Global Services, the monthly claims report needed to combine records from two separate regional databases (EMEA and APAC) that had identical schemas. The team also needed to identify claims that appeared in both systems (potential duplicates) and claims unique to each system.

🟡 **Task** Demonstrate all four set operations with appropriate use cases.

🟢 **Action**
    
    
    -- ─── SQL: UNION (removes duplicates — slower) ────────────────────────────────
    SELECT claim_id, claim_amount, claim_status, 'EMEA'  AS source_region FROM claims_emea
    UNION
    SELECT claim_id, claim_amount, claim_status, 'APAC'  AS source_region FROM claims_apac;
    -- Returns distinct rows only — use when duplicates must be eliminated
    
    -- ─── SQL: UNION ALL (keeps all rows — faster) ────────────────────────────────
    SELECT claim_id, claim_amount, claim_status, 'EMEA' AS source FROM claims_emea
    UNION ALL
    SELECT claim_id, claim_amount, claim_status, 'APAC' AS source FROM claims_apac;
    -- Always prefer UNION ALL when you know datasets are distinct or duplicates are acceptable
    
    -- ─── SQL: INTERSECT — rows present in BOTH sets ──────────────────────────────
    SELECT claim_id FROM claims_emea
    INTERSECT
    SELECT claim_id FROM claims_apac;
    -- Returns claim_ids that appear in both regional databases (potential duplicates)
    
    -- ─── SQL: EXCEPT (MINUS in Oracle) — rows in left NOT in right ────────────────
    SELECT claim_id FROM claims_emea
    EXCEPT
    SELECT claim_id FROM claims_apac;
    -- Returns claim_ids in EMEA that are NOT in APAC
    
    -- ─── SQL: Practical combined reconciliation ────────────────────────────────────
    SELECT 'EMEA only'       AS category, COUNT(*) AS count FROM (
        SELECT claim_id FROM claims_emea
        EXCEPT
        SELECT claim_id FROM claims_apac
    ) x
    UNION ALL
    SELECT 'APAC only',       COUNT(*) FROM (
        SELECT claim_id FROM claims_apac
        EXCEPT
        SELECT claim_id FROM claims_emea
    ) x
    UNION ALL
    SELECT 'Both systems',    COUNT(*) FROM (
        SELECT claim_id FROM claims_emea
        INTERSECT
        SELECT claim_id FROM claims_apac
    ) x;
    
    -- Output:
    -- category       | count
    -- ---------------|-------
    -- EMEA only      |   142
    -- APAC only      |    98
    -- Both systems   |    12  ← potential duplicates to investigate
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Simulate two regional DataFrames
    claims_emea = claims[claims['region_id'].isin([401, 405])].copy()
    claims_emea['source'] = 'EMEA'
    claims_apac = claims[claims['region_id'].isin([402, 404])].copy()
    claims_apac['source'] = 'APAC'
    
    # UNION ALL: pd.concat (always use — fastest, explicit)
    combined = pd.concat([claims_emea, claims_apac], ignore_index=True)
    print(f"UNION ALL: {len(combined)} rows")
    
    # UNION: pd.concat + drop_duplicates
    unioned = pd.concat([claims_emea, claims_apac]).drop_duplicates()
    print(f"UNION (distinct): {len(unioned)} rows")
    
    # INTERSECT: merge with indicator
    emea_ids = set(claims_emea['claim_id'])
    apac_ids = set(claims_apac['claim_id'])
    
    intersect_ids = emea_ids & apac_ids          # Python set intersection
    except_ids    = emea_ids - apac_ids          # Python set difference
    union_ids     = emea_ids | apac_ids          # Python set union
    
    print(f"\nINTERSECT (both systems): {len(intersect_ids)}")
    print(f"EXCEPT (EMEA only):       {len(except_ids)}")
    print(f"UNION (either system):    {len(union_ids)}")
    
    # Using merge with indicator (more powerful — keeps full row context)
    merged = pd.merge(
        claims_emea[['claim_id','claim_amount']],
        claims_apac[['claim_id','claim_amount']],
        on='claim_id', how='outer', indicator=True
    )
    recon_summary = merged['_merge'].value_counts().rename({
        'left_only':  'EMEA only',
        'right_only': 'APAC only',
        'both':       'Both systems'
    })
    print(f"\nReconciliation:\n{recon_summary}")
    
    
    

🔴 **Result** Set operations identified 12 claims present in both regional databases — confirmed as duplicates from a batch reprocessing event. Removing them corrected the cross-regional total incurred from £2.98M (inflated) to £2.81M. The reconciliation pattern became the standard monthly cross-regional data quality check.

* * *

## Section 15: SUBQUERIES — Nested Queries

* * *

### Q32. How do you use correlated and non-correlated subqueries?

🔵 **Situation** At Travelers Insurance, the risk team needed to find claims that were above their own product line's average — without using a window function (for compatibility with an older database version).

🟡 **Task** Use subqueries (both types) to identify above-average claims within each product line.

🟢 **Action**
    
    
    -- ─── SQL: Non-correlated subquery (executes once) ────────────────────────────
    -- Find all claims above the GLOBAL average
    SELECT  claim_id, claim_amount, product_line
    FROM    claims
    WHERE   claim_amount > (
        SELECT AVG(claim_amount) FROM claims   -- executes once for entire query
    )
    ORDER BY claim_amount DESC;
    
    -- ─── SQL: Correlated subquery (executes per row) ──────────────────────────────
    -- Find claims above their OWN product line's average
    SELECT
        c.claim_id,
        c.claim_amount,
        c.product_line,
        ROUND((
            SELECT AVG(c2.claim_amount)
            FROM   claims c2
            WHERE  c2.product_line = c.product_line  -- references outer query
        ), 2)                          AS product_avg,
        ROUND(c.claim_amount - (
            SELECT AVG(c2.claim_amount)
            FROM   claims c2
            WHERE  c2.product_line = c.product_line
        ), 2)                          AS above_avg_by
    FROM   claims c
    WHERE  c.claim_amount > (
        SELECT AVG(c2.claim_amount)
        FROM   claims c2
        WHERE  c2.product_line = c.product_line
    )
    ORDER BY above_avg_by DESC;
    
    -- Output (sample):
    -- claim_id | claim_amount | product_line | product_avg | above_avg_by
    -- ---------|--------------|--------------|-------------|-------------
    --     1021 |   198430.00  | Motor        |   24698.47  |  173731.53
    --     1047 |   162000.00  | Property     |   20419.25  |  141580.75
    
    -- ─── Performance note: Replace correlated subqueries with JOIN+CTE when possible
    WITH product_avgs AS (
        SELECT product_line, AVG(claim_amount) AS avg_amount
        FROM   claims GROUP BY product_line
    )
    SELECT c.claim_id, c.claim_amount, c.product_line, pa.avg_amount
    FROM   claims c
    JOIN   product_avgs pa ON pa.product_line = c.product_line
    WHERE  c.claim_amount > pa.avg_amount;   -- same result, much faster
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Non-correlated: global average filter
    global_avg = claims['claim_amount'].mean()
    above_global = claims[claims['claim_amount'] > global_avg].copy()
    print(f"Global avg: £{global_avg:,.2f}")
    print(f"Claims above global avg: {len(above_global)}")
    
    # Correlated: product line average filter (use transform — vectorised equivalent)
    claims['product_avg'] = claims.groupby('product_line')['claim_amount'].transform('mean')
    claims['above_avg_by'] = claims['claim_amount'] - claims['product_avg']
    
    above_product_avg = claims[claims['claim_amount'] > claims['product_avg']].copy()
    above_product_avg = above_product_avg.sort_values('above_avg_by', ascending=False)
    
    print(f"\nClaims above their product avg: {len(above_product_avg)}")
    print(above_product_avg[['claim_id','claim_amount','product_line',
                               'product_avg','above_avg_by']].round(2).head(5))
    
    # The transform() approach is the Pandas equivalent of a correlated subquery
    # and is vectorised (much faster than row-wise apply)
    
    # Scalar subquery equivalent: single-value comparison
    top_10_pct_threshold = claims['claim_amount'].quantile(0.90)
    top_decile = claims[claims['claim_amount'] >= top_10_pct_threshold]
    print(f"\nTop 10% threshold: £{top_10_pct_threshold:,.2f}")
    print(f"Top decile claims: {len(top_decile)}")
    
    
    

🔴 **Result** The correlated subquery (rewritten as a CTE join) identified 87 above-product-average claims — 43.5% of total volume. The Motor segment's above-average claims had a median amount 7x the product average, confirming the bimodal distribution hypothesis and justifying separate pricing for high-value Motor policies.

* * *

### Q33. How do you use EXISTS and NOT EXISTS for semi-joins?

🔵 **Situation** At J&J Global Services, the compliance team needed to find all policies that had at least one fraud-flagged claim (for an enhanced review programme) — and separately, all policies with NO fraud-flagged claims (for a loyalty discount).

🟡 **Task** Use EXISTS and NOT EXISTS to efficiently identify policies based on the existence of related fraud claims.

🟢 **Action**
    
    
    -- ─── SQL: EXISTS (semi-join — stops at first match, very efficient) ───────────
    -- Policies with at least ONE fraud-flagged claim
    SELECT
        p.policy_id,
        p.policyholder,
        p.premium,
        p.risk_tier
    FROM   policies p
    WHERE  EXISTS (
        SELECT 1           -- SELECT 1: no data needed, just checking existence
        FROM   claims c
        WHERE  c.policy_id  = p.policy_id    -- correlated condition
          AND  c.fraud_flag = 1
    )
    ORDER BY p.policy_id;
    
    -- ─── SQL: NOT EXISTS ──────────────────────────────────────────────────────────
    -- Policies with NO fraud-flagged claims (clean policies)
    SELECT
        p.policy_id,
        p.policyholder,
        p.premium
    FROM   policies p
    WHERE  NOT EXISTS (
        SELECT 1
        FROM   claims c
        WHERE  c.policy_id  = p.policy_id
          AND  c.fraud_flag = 1
    )
    ORDER BY p.policy_id;
    
    -- ─── NOT EXISTS vs NOT IN — critical difference ───────────────────────────────
    -- NOT IN fails silently when the subquery contains NULLs:
    -- WHERE policy_id NOT IN (SELECT policy_id FROM claims) ← breaks if any NULLs
    -- NOT EXISTS handles NULLs correctly — always use NOT EXISTS
    
    -- Output (EXISTS):
    -- policy_id | policyholder | premium | risk_tier
    -- ----------|--------------|---------|----------
    --       202 | Holder_202   | 2100.00 | High
    --       205 | Holder_205   | 1850.00 | High
    
    -- Output (NOT EXISTS — clean policies):
    -- policy_id | policyholder | premium
    -- ----------|--------------|--------
    --       215 | Holder_215   | 3200.00
    --       218 | Holder_218   |  890.00
    
    
    
    
    
    # ─── Pandas: EXISTS equivalent ────────────────────────────────────────────────
    # Fraud-flagged claim IDs per policy
    fraud_policy_ids = claims[claims['fraud_flag'] == 1]['policy_id'].unique()
    
    # EXISTS: policies IN the fraud set
    policies_with_fraud = policies[policies['policy_id'].isin(fraud_policy_ids)]
    
    # NOT EXISTS: policies NOT IN the fraud set (NULL-safe — isin handles NaN correctly)
    clean_policies = policies[~policies['policy_id'].isin(fraud_policy_ids)]
    
    print(f"Policies with fraud claims: {len(policies_with_fraud)}")
    print(f"Clean policies (no fraud):  {len(clean_policies)}")
    
    # More complex: policies with fraud claims in CURRENT year only (conditional EXISTS)
    current_year = pd.Timestamp.today().year
    fraud_current_yr = claims[
        (claims['fraud_flag'] == 1) &
        (pd.to_datetime(claims['claim_date']).dt.year == current_year)
    ]['policy_id'].unique()
    
    policies['has_current_fraud'] = policies['policy_id'].isin(fraud_current_yr)
    
    # EXISTS with multiple conditions using merge
    fraud_high_value = claims[
        (claims['fraud_flag'] == 1) & (claims['claim_amount'] > 10_000)
    ][['policy_id']].drop_duplicates()
    
    policies_fraud_high = policies.merge(
        fraud_high_value, on='policy_id', how='inner'
    )
    print(f"\nPolicies with fraud AND high-value claims: {len(policies_fraud_high)}")
    
    
    

🔴 **Result** EXISTS identified 6 policies with fraud-flagged claims (30% of the 20-policy portfolio) — these were enrolled in the enhanced monitoring programme. NOT EXISTS correctly identified 11 clean policies (excluding 3 zero-claim policies separately), who received a 5% loyalty discount — targeted accurately because NOT EXISTS correctly handles the NULL policy_id edge case that NOT IN would silently mishandle.

* * *

## Section 16: STATISTICAL FUNCTIONS — Descriptive Analytics

* * *

### Q34. How do you compute standard deviation, variance, and correlation in SQL and Pandas?

🔵 **Situation** At J&J Global Services, the actuarial team needed to assess the statistical spread and correlation of claim amounts and reserve amounts — to validate that reserves were being set consistently proportional to claims.

🟡 **Task** Compute standard deviation, variance, and Pearson correlation for the claims dataset grouped by product line.

🟢 **Action**
    
    
    -- ─── SQL ─────────────────────────────────────────────────────────────────────
    SELECT
        product_line,
        COUNT(*)                                AS n,
        ROUND(AVG(claim_amount), 2)             AS mean_claim,
        ROUND(STDDEV(claim_amount), 2)          AS stddev_claim,      -- population STDDEV
        ROUND(STDDEV_SAMP(claim_amount), 2)     AS stddev_samp_claim, -- sample STDDEV (n-1)
        ROUND(VARIANCE(claim_amount), 2)        AS variance_claim,
        -- Coefficient of Variation (relative spread)
        ROUND(STDDEV(claim_amount) /
              NULLIF(AVG(claim_amount), 0) * 100, 2) AS cv_pct,
        -- Correlation between claim and reserve amounts (PostgreSQL)
        ROUND(CORR(claim_amount, reserve_amount)::NUMERIC, 4) AS claim_reserve_corr
    FROM    claims
    WHERE   reserve_amount IS NOT NULL
    GROUP BY product_line
    ORDER BY cv_pct DESC;
    
    -- Output:
    -- product_line | n  | mean_claim | stddev_claim | cv_pct | claim_reserve_corr
    -- -------------|----|-----------:|-------------:|-------:|-------------------
    -- Motor        | 52 |   24698.47 |    38421.90  | 155.6% |             0.8923
    -- Property     | 48 |   20419.25 |    29834.71  | 146.1% |             0.9142
    -- Health       | 51 |    8272.37 |    15234.21  | 184.2% |             0.7834
    -- Travel       | 49 |    4111.23 |     7892.45  | 192.0% |             0.8291
    
    
    
    
    
    # ─── Pandas ──────────────────────────────────────────────────────────────────
    # Per-group statistical summary
    stats = claims.dropna(subset=['reserve_amount']).groupby('product_line').agg(
        n             = ('claim_amount', 'count'),
        mean_claim    = ('claim_amount', 'mean'),
        std_claim     = ('claim_amount', 'std'),       # sample std (ddof=1)
        std_pop       = ('claim_amount', lambda x: x.std(ddof=0)),  # population std
        var_claim     = ('claim_amount', 'var'),
        mean_reserve  = ('reserve_amount', 'mean'),
        std_reserve   = ('reserve_amount', 'std'),
    ).round(2)
    
    stats['cv_pct'] = (stats['std_claim'] / stats['mean_claim'] * 100).round(1)
    
    # Correlation per group
    stats['claim_reserve_corr'] = claims.dropna(subset=['reserve_amount']).groupby(
        'product_line'
    ).apply(lambda g: g['claim_amount'].corr(g['reserve_amount'])).round(4)
    
    print(stats[['n','mean_claim','std_claim','cv_pct','claim_reserve_corr']])
    
    # Full correlation matrix across numeric columns
    numeric_cols = ['claim_amount','reserve_amount','fraud_flag']
    corr_matrix = claims[numeric_cols].corr(method='pearson')  # or 'spearman','kendall'
    print("\nCorrelation matrix:")
    print(corr_matrix.round(4))
    
    # Statistical significance of correlation
    from scipy.stats import pearsonr
    r, p_value = pearsonr(
        claims['claim_amount'].dropna(),
        claims['reserve_amount'].dropna()
    )
    print(f"\nOverall claim-reserve correlation: r={r:.4f}, p={p_value:.2e}")
    print(f"Statistically significant: {'Yes' if p_value < 0.05 else 'No'}")
    
    # Output (correlation matrix):
    #                claim_amount  reserve_amount  fraud_flag
    # claim_amount         1.0000          0.8734      0.1823
    # reserve_amount       0.8734          1.0000      0.1241
    # fraud_flag           0.1823          0.1241      1.0000
    
    
    

🔴 **Result** The correlation analysis confirmed reserves were strongly correlated with claims (r=0.87) — validating the reserving methodology. However, the Health product line's lower correlation (0.78) indicated inconsistent reserve-setting, prompting an adjuster training programme that improved Health reserve accuracy by 23%.

* * *

## Section 17: DATA PROFILING — Understanding Your Dataset

* * *

### Q35. How do you perform a comprehensive data profile of a new dataset?

🔵 **Situation** At AXA Insurance, when onboarding the fourth legacy data source, the team had no documentation. A full data profile was needed before building the ETL pipeline to understand data types, completeness, cardinality, and distribution.

🟡 **Task** Write a comprehensive data profiling query/script that covered all key dimensions of data quality.

🟢 **Action**
    
    
    -- ─── SQL: Column-level data profile ──────────────────────────────────────────
    -- For claim_amount specifically:
    SELECT
        'claim_amount'                          AS column_name,
        COUNT(*)                                AS total_rows,
        COUNT(claim_amount)                     AS non_null_count,
        COUNT(*) - COUNT(claim_amount)          AS null_count,
        ROUND((COUNT(*) - COUNT(claim_amount))::NUMERIC /
              COUNT(*) * 100, 2)               AS null_pct,
        COUNT(DISTINCT claim_amount)            AS distinct_values,
        ROUND(AVG(claim_amount), 2)             AS mean,
        ROUND(STDDEV(claim_amount), 2)          AS std_dev,
        MIN(claim_amount)                       AS min_val,
        MAX(claim_amount)                       AS max_val,
        ROUND(PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY claim_amount), 2) AS p25,
        ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY claim_amount), 2) AS median,
        ROUND(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY claim_amount), 2) AS p75
    FROM claims;
    
    -- String column profile:
    SELECT
        'claim_status'                          AS column_name,
        COUNT(*)                                AS total_rows,
        COUNT(DISTINCT claim_status)            AS distinct_values,
        MAX(LENGTH(claim_status))               AS max_length,
        MIN(LENGTH(claim_status))               AS min_length
    FROM claims;
    
    -- Value frequency distribution:
    SELECT
        claim_status,
        COUNT(*)                                AS frequency,
        ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS pct
    FROM claims
    GROUP BY claim_status
    ORDER BY frequency DESC;
    
    -- Output (frequency):
    -- claim_status | frequency | pct
    -- -------------|-----------|------
    -- Settled      |        80 | 40.0
    -- Open         |        60 | 30.0
    -- Pending      |        30 | 15.0
    -- Rejected     |        30 | 15.0
    
    
    
    
    
    # ─── Pandas: Full automated data profiling ────────────────────────────────────
    def data_profile(df, dataset_name='Dataset'):
        """
        Comprehensive data profile report for any DataFrame.
        Returns a summary DataFrame covering all key quality dimensions.
        """
        print(f"\n{'='*60}")
        print(f"DATA PROFILE: {dataset_name}")
        print(f"{'='*60}")
        print(f"Shape: {df.shape[0]:,} rows × {df.shape[1]} columns")
        print(f"Memory: {df.memory_usage(deep=True).sum() / 1e6:.2f} MB\n")
    
        profile = []
        for col in df.columns:
            series = df[col]
            row = {
                'column':         col,
                'dtype':          str(series.dtype),
                'total_rows':     len(series),
                'non_null':       series.count(),
                'null_count':     series.isnull().sum(),
                'null_pct':       round(series.isnull().mean() * 100, 2),
                'distinct':       series.nunique(),
                'distinct_pct':   round(series.nunique() / len(series) * 100, 2),
            }
    
            # Numeric-specific stats
            if pd.api.types.is_numeric_dtype(series):
                row.update({
                    'mean':    round(series.mean(), 2),
                    'std':     round(series.std(), 2),
                    'min':     round(series.min(), 2),
                    'p25':     round(series.quantile(0.25), 2),
                    'median':  round(series.median(), 2),
                    'p75':     round(series.quantile(0.75), 2),
                    'max':     round(series.max(), 2),
                    'zeros':   int((series == 0).sum()),
                    'negatives': int((series < 0).sum()),
                })
            # String-specific stats
            elif pd.api.types.is_object_dtype(series) or isinstance(series.dtype, pd.CategoricalDtype):
                row.update({
                    'top_value':    series.value_counts().index[0] if series.count() > 0 else None,
                    'top_freq':     series.value_counts().iloc[0]  if series.count() > 0 else 0,
                    'max_length':   series.dropna().astype(str).str.len().max(),
                    'min_length':   series.dropna().astype(str).str.len().min(),
                })
            # Date-specific stats
            elif pd.api.types.is_datetime64_any_dtype(series):
                row.update({
                    'min_date': series.min(),
                    'max_date': series.max(),
                    'date_range_days': (series.max() - series.min()).days if series.count() > 0 else None,
                })
    
            profile.append(row)
    
        profile_df = pd.DataFrame(profile)
    
        # Print key summary
        print("NULL SUMMARY (columns with missing data):")
        nulls = profile_df[profile_df['null_count'] > 0][['column','null_count','null_pct']]
        print(nulls.to_string(index=False) if len(nulls) > 0 else "  No missing values ✓")
    
        print("\nCARDINALITY (columns with < 20 distinct values — potential categoricals):")
        low_card = profile_df[profile_df['distinct'] < 20][['column','distinct','distinct_pct']]
        print(low_card.to_string(index=False))
    
        print("\nNUMERIC DISTRIBUTION:")
        num_cols = profile_df[profile_df['dtype'].str.contains('int|float')][
            ['column','mean','std','min','median','max','zeros','negatives']
        ]
        print(num_cols.to_string(index=False))
    
        return profile_df
    
    # Run the profiler
    profile = data_profile(claims, 'Claims Dataset')
    
    # Value frequency for categorical columns
    for col in ['claim_status','product_line','fraud_flag']:
        freq = claims[col].value_counts(normalize=False)
        pct  = claims[col].value_counts(normalize=True) * 100
        print(f"\n{col} distribution:")
        print(pd.DataFrame({'count': freq, 'pct': pct.round(1)}).to_string())
    
    
    

🔴 **Result** The automated data profiler ran in 4 seconds and generated a complete quality report for all 11 columns. It immediately flagged 2 issues: (1) `claim_amount` had 0 zeros but `reserve_amount` had 3 zeros — indicating reserve-setting failures, and (2) `fraud_flag` had only 2 distinct values — confirming binary nature and triggering automatic conversion to `int8` (7x memory saving).

* * *

## Section 18: INDEXING & PERFORMANCE — Query Optimisation Fundamentals

* * *

### Q36. How do you understand and improve query performance using indexes and execution plans?

🔵 **Situation** At LSEG, the daily reconciliation query was taking 18 minutes on a 500M-row table. An execution plan review was needed to identify the bottleneck and apply the correct indexing strategy.

🟡 **Task** Explain how to read an execution plan, identify missing indexes, and create the optimal index structure.

🟢 **Action**
    
    
    -- ─── SQL: Execution plan analysis ────────────────────────────────────────────
    -- SQL Server: view execution plan
    SET STATISTICS IO ON;    -- shows logical reads
    SET STATISTICS TIME ON;  -- shows execution time
    EXPLAIN ANALYZE           -- PostgreSQL: EXPLAIN ANALYZE
    EXPLAIN                   -- SQLite: EXPLAIN QUERY PLAN
    
    -- Query BEFORE optimisation (full table scan)
    -- EXPLAIN output shows: "Seq Scan on claims (cost=0..8500 rows=200000)"
    SELECT claim_id, claim_amount, claim_status
    FROM   claims
    WHERE  claim_status = 'Open'
      AND  claim_date >= '2023-01-01';
    -- Statistics IO output: Table 'claims': Scan count 1, logical reads 45823
    -- Execution time: 18 minutes
    
    -- ─── Create covering index (includes all columns needed by the query) ─────────
    -- Non-clustered index on filter columns, INCLUDE output columns
    CREATE NONCLUSTERED INDEX idx_claims_status_date
    ON claims (claim_status, claim_date)         -- filter columns (WHERE clause)
    INCLUDE   (claim_id, claim_amount);          -- output columns (SELECT list)
    -- INCLUDE avoids key lookup — all needed data in one index read
    
    -- PostgreSQL equivalent:
    CREATE INDEX idx_claims_status_date
    ON claims (claim_status, claim_date)
    INCLUDE (claim_id, claim_amount);
    
    -- ─── After index: execution plan shows index seek instead of table scan ────────
    -- "Index Seek using idx_claims_status_date (cost=0..120 rows=3000)"
    -- Statistics IO: Table 'claims': Scan count 1, logical reads 42  ← 99.9% reduction
    
    -- ─── Index guidelines ─────────────────────────────────────────────────────────
    -- 1. Index columns used in WHERE, JOIN ON, ORDER BY
    -- 2. Put most selective column first in composite index
    -- 3. INCLUDE output columns to create covering indexes
    -- 4. Avoid indexing columns updated frequently (index maintenance cost)
    -- 5. Max ~5 indexes per table for write-heavy tables
    
    -- ─── Check existing indexes ───────────────────────────────────────────────────
    -- SQL Server:
    SELECT name, type_desc, is_unique
    FROM sys.indexes
    WHERE object_id = OBJECT_ID('claims');
    
    -- PostgreSQL:
    SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'claims';
    
    -- Find missing indexes (SQL Server):
    SELECT
        mig.equality_columns,
        mig.inequality_columns,
        migs.avg_user_impact,
        migs.user_seeks
    FROM sys.dm_db_missing_index_groups   mig
    JOIN sys.dm_db_missing_index_group_stats migs ON migs.group_handle = mig.index_group_handle
    WHERE database_id = DB_ID()
    ORDER BY migs.avg_user_impact DESC;
    
    
    
    
    
    # ─── Pandas: Performance profiling and optimisation ───────────────────────────
    import time
    
    # Timing decorator
    def time_it(func):
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            result = func(*args, **kwargs)
            elapsed = time.perf_counter() - start
            print(f"{func.__name__}: {elapsed*1000:.2f}ms")
            return result
        return wrapper
    
    # Benchmark: boolean index vs query() vs loc
    n = 1_000_000
    large_claims = pd.concat([claims] * (n // len(claims)), ignore_index=True)
    
    # Method 1: Boolean indexing
    start = time.perf_counter()
    result1 = large_claims[
        (large_claims['claim_status'] == 'Open') &
        (large_claims['claim_amount'] > 5000)
    ]
    t1 = time.perf_counter() - start
    
    # Method 2: query() — numexpr-backed
    start = time.perf_counter()
    result2 = large_claims.query("claim_status == 'Open' and claim_amount > 5000",
                                  engine='python')
    t2 = time.perf_counter() - start
    
    print(f"Boolean mask: {t1*1000:.1f}ms")
    print(f"query():      {t2*1000:.1f}ms")
    print(f"Both return same rows: {len(result1) == len(result2)}")
    
    # Optimisation: categorical dtype for low-cardinality string columns
    print(f"\nMemory BEFORE: {large_claims.memory_usage(deep=True).sum() / 1e6:.1f} MB")
    large_claims['claim_status']  = large_claims['claim_status'].astype('category')
    large_claims['product_line']  = large_claims['product_line'].astype('category')
    large_claims['fraud_flag']    = large_claims['fraud_flag'].astype('int8')
    large_claims['claim_amount']  = large_claims['claim_amount'].astype('float32')
    print(f"Memory AFTER:  {large_claims.memory_usage(deep=True).sum() / 1e6:.1f} MB")
    
    # Set index for fast lookups (equivalent to database clustered index)
    claims_indexed = claims.set_index('claim_id').sort_index()
    # O(log n) lookup with sorted index vs O(n) with default RangeIndex
    specific_claim = claims_indexed.loc[1001]   # direct lookup
    
    
    

🔴 **Result** Adding the covering index reduced the reconciliation query from 18 minutes to **11 minutes** (39% improvement from the index alone). Combined with the query rewrite from Q11 (removing correlated subqueries), total runtime dropped to 2 minutes 40 seconds. The categorical dtype optimisation reduced the Pandas DataFrame memory from 18.4 MB to 4.2 MB (77% reduction).

* * *

## Section 19: MERGE / UPSERT — Incremental Data Loading

* * *

### Q37. How do you implement UPSERT (insert if new, update if exists) using MERGE?

🔵 **Situation** At J&J Global Services, the daily Salesforce pipeline needed to update existing claim records (status changes) and insert new ones — previously done as DELETE + INSERT which caused brief data gaps visible to dashboard users.

🟡 **Task** Replace the DELETE + INSERT pattern with an atomic MERGE (UPSERT) operation.

🟢 **Action**
    
    
    -- ─── SQL: MERGE (UPSERT) ──────────────────────────────────────────────────────
    MERGE INTO claims AS tgt            -- target table
    USING claims_staging AS src         -- source (new/updated data)
    ON    tgt.claim_id = src.claim_id   -- match condition (business key)
    
    -- When the key exists in both: UPDATE changed fields
    WHEN MATCHED AND (
        tgt.claim_status  <> src.claim_status  OR
        tgt.claim_amount  <> src.claim_amount  OR
        tgt.reserve_amount <> src.reserve_amount
    ) THEN UPDATE SET
        tgt.claim_status   = src.claim_status,
        tgt.claim_amount   = src.claim_amount,
        tgt.reserve_amount = src.reserve_amount,
        tgt.settlement_date = src.settlement_date
    
    -- When key exists only in source: INSERT new record
    WHEN NOT MATCHED BY TARGET THEN
        INSERT (claim_id, policy_id, adjuster_id, region_id, product_line,
                claim_amount, reserve_amount, claim_date, settlement_date,
                claim_status, fraud_flag)
        VALUES (src.claim_id, src.policy_id, src.adjuster_id, src.region_id,
                src.product_line, src.claim_amount, src.reserve_amount,
                src.claim_date, src.settlement_date, src.claim_status, src.fraud_flag)
    
    -- When key exists only in target: soft-delete (mark as inactive)
    WHEN NOT MATCHED BY SOURCE THEN
        UPDATE SET tgt.claim_status = 'Archived'
    
    -- Output clause: capture what changed
    OUTPUT
        $action                AS merge_action,    -- 'INSERT','UPDATE','DELETE'
        inserted.claim_id      AS claim_id,
        inserted.claim_status  AS new_status,
        deleted.claim_status   AS old_status
    INTO merge_audit_log (merge_action, claim_id, new_status, old_status);
    
    -- COMMON PITFALLS:
    -- 1. Duplicate source rows → MERGE throws error. Always deduplicate source first.
    -- 2. Only match on indexed columns → ensure ON clause uses indexed keys
    -- 3. WHEN NOT MATCHED BY SOURCE affects ALL unmatched target rows → add filter
    
    
    
    
    
    # ─── Pandas: UPSERT equivalent ───────────────────────────────────────────────
    import hashlib
    
    # Simulate staging data: 10 updates + 5 new records
    staging = claims.sample(10).copy()
    staging['claim_status'] = 'Updated'      # simulate status change
    new_records = pd.DataFrame({
        'claim_id':     range(1201, 1206),
        'policy_id':    [201]*5,
        'adjuster_id':  [301]*5,
        'region_id':    [401]*5,
        'product_line': ['Motor']*5,
        'claim_amount': [5000, 8000, 12000, 3000, 25000],
        'reserve_amount': [5500, 8800, 13200, 3300, 27500],
        'claim_date':   pd.date_range('2023-07-01', periods=5, freq='D'),
        'settlement_date': [None]*5,
        'claim_status': ['Open']*5,
        'fraud_flag':   [0]*5,
    })
    staging_full = pd.concat([staging, new_records], ignore_index=True)
    
    # UPSERT logic
    def upsert(target: pd.DataFrame, source: pd.DataFrame, key: str) -> pd.DataFrame:
        """
        Merge source into target:
        - UPDATE rows where key matches
        - INSERT rows where key is new
        Returns updated target + audit log.
        """
        audit = []
    
        # Deduplicate source on key
        source = source.drop_duplicates(subset=key, keep='last')
    
        # Identify updates and inserts
        existing_keys = set(target[key])
        update_mask   = source[key].isin(existing_keys)
        inserts       = source[~update_mask]
        updates       = source[update_mask]
    
        # Apply updates
        target_indexed = target.set_index(key)
        for _, row in updates.iterrows():
            old_status = target_indexed.loc[row[key], 'claim_status']
            target_indexed.loc[row[key]] = row.drop(key)
            audit.append({'action': 'UPDATE', key: row[key],
                          'old_status': old_status, 'new_status': row['claim_status']})
    
        # Apply inserts
        target_updated = pd.concat(
            [target_indexed.reset_index(), inserts], ignore_index=True
        )
        for _, row in inserts.iterrows():
            audit.append({'action': 'INSERT', key: row[key],
                          'old_status': None, 'new_status': row['claim_status']})
    
        audit_df = pd.DataFrame(audit)
        return target_updated, audit_df
    
    claims_updated, merge_audit = upsert(claims, staging_full, 'claim_id')
    
    print(f"Rows before UPSERT: {len(claims)}")
    print(f"Rows after UPSERT:  {len(claims_updated)}")
    print(f"\nAudit log:")
    print(merge_audit['action'].value_counts())
    
    
    

🔴 **Result** The MERGE pattern eliminated the DELETE + INSERT data gap. Dashboard users no longer experienced the 2-minute window where claim counts dropped to zero during nightly loads. The audit log captured 10 status updates and 5 new inserts, providing a complete change trail for the compliance team.

* * *

## Section 20: BEST PRACTICES — Quick Reference Summary

* * *

### Q38. What are the most important SQL and Pandas best practices every data analyst should follow?

🔵 **Situation** At every project — LSEG, J&J, Travelers, AXA — consistent application of best practices prevented the most common and costly data analysis errors.

🟡 **Task** Compile a comprehensive best practices checklist covering the most critical habits.

🟢 **Action**
    
    
    -- ─── SQL BEST PRACTICES ───────────────────────────────────────────────────────
    
    -- ✅ 1. ALWAYS specify column names in SELECT — never use SELECT *
    --    SELECT * → brittle if schema changes; downloads unnecessary data
    SELECT claim_id, claim_amount, claim_status   -- explicit columns
    FROM   claims;
    
    -- ✅ 2. Use table aliases for readability in multi-table queries
    SELECT c.claim_id, p.policyholder, a.adjuster_name
    FROM   claims    c
    JOIN   policies  p ON p.policy_id    = c.policy_id
    JOIN   adjusters a ON a.adjuster_id  = c.adjuster_id;
    
    -- ✅ 3. Filter early — apply WHERE before JOIN to reduce row sets
    SELECT c.claim_id, p.policyholder
    FROM   claims c
    JOIN   policies p ON p.policy_id = c.policy_id
    WHERE  c.claim_status = 'Open'     -- filter BEFORE or DURING join, not after
      AND  c.claim_amount > 5000;
    
    -- ✅ 4. Use COALESCE defensively for NULL-safe aggregations
    SELECT product_line, COALESCE(SUM(claim_amount), 0) AS total
    FROM   claims GROUP BY product_line;
    
    -- ✅ 5. Prefer NOT EXISTS over NOT IN for NULL safety
    -- WRONG: NOT IN fails silently if subquery has NULLs
    -- RIGHT:
    WHERE NOT EXISTS (SELECT 1 FROM settlements s WHERE s.claim_id = c.claim_id);
    
    -- ✅ 6. Use CTE chains instead of deeply nested subqueries
    WITH base AS (SELECT ... FROM claims WHERE ...),
         agg  AS (SELECT ... FROM base GROUP BY ...)
    SELECT * FROM agg;
    
    -- ✅ 7. Always validate JOIN cardinality (check for unexpected row multiplication)
    -- Before a merge: COUNT source and target; after: verify output count is expected
    
    -- ✅ 8. Use NULLIF to prevent division-by-zero errors
    ROUND(numerator / NULLIF(denominator, 0), 2)
    
    -- ✅ 9. Always use UNION ALL unless duplicate removal is explicitly required
    -- UNION performs an expensive sort/hash; UNION ALL does not
    
    -- ✅ 10. Comment your queries — future you will thank you
    -- Actuarial loss ratio: incurred / earned premium × 100
    SELECT ROUND(SUM(claim_amount) / NULLIF(SUM(premium), 0) * 100, 2) AS loss_ratio
    FROM   claims c JOIN policies p ON p.policy_id = c.policy_id;
    
    
    
    
    
    # ─── PANDAS BEST PRACTICES ────────────────────────────────────────────────────
    
    # ✅ 1. Always check shape, dtypes, and head() when loading new data
    df = pd.read_csv('claims.csv')
    print(df.shape, df.dtypes, df.head(3), df.describe())
    
    # ✅ 2. Specify dtypes on load to save memory and avoid type inference errors
    df = pd.read_csv('claims.csv', dtype={
        'claim_status': 'category',
        'fraud_flag':   'int8',
        'claim_amount': 'float32'
    })
    
    # ✅ 3. Use .copy() when creating subsets to avoid SettingWithCopyWarning
    subset = df[df['claim_status'] == 'Open'].copy()   # .copy() prevents slice issues
    subset['new_col'] = 1   # safe on copy
    
    # ✅ 4. Prefer vectorised operations over row-wise apply()
    # SLOW: df.apply(lambda row: row['a'] + row['b'], axis=1)
    # FAST: df['a'] + df['b']
    
    # ✅ 5. Always use na=False in string operations to handle NaN gracefully
    df['match'] = df['claim_status'].str.contains('Open', na=False)
    
    # ✅ 6. Chain operations with parentheses for readability
    result = (
        df
        .query("claim_status != 'Rejected'")
        .groupby('product_line')
        .agg(total=('claim_amount','sum'), count=('claim_id','count'))
        .round(2)
        .sort_values('total', ascending=False)
        .reset_index()
    )
    
    # ✅ 7. Validate merges with validate= and indicator=True
    merged = pd.merge(claims, policies, on='policy_id',
                      how='left', validate='many_to_one', indicator=True)
    assert (merged['_merge'] == 'right_only').sum() == 0, "Orphan policies detected!"
    
    # ✅ 8. Use assert for data validation checkpoints
    assert df['claim_id'].nunique() == len(df), "Duplicate claim_ids found!"
    assert df['claim_amount'].min() >= 0, "Negative claim amounts detected!"
    assert df['fraud_flag'].isin([0,1]).all(), "Invalid fraud_flag values!"
    
    # ✅ 9. Profile before processing — know your NULLs, types, and cardinality
    null_report = df.isnull().sum()
    type_report = df.dtypes
    card_report = df.nunique()
    
    # ✅ 10. Document with docstrings and inline comments
    def compute_loss_ratio(claims_df, policies_df):
        """
        Compute loss ratio = total incurred claims / total earned premium.
        Returns: float (percentage, e.g. 87.3 means 87.3%)
        """
        total_claims  = claims_df['claim_amount'].sum()
        total_premium = policies_df['premium'].sum()
        return round(total_claims / total_premium * 100, 2)
    
    
    

🔴 **Result** Systematic application of these best practices across all projects at LSEG, J&J, Travelers, and AXA prevented recurring issues: `NOT IN` NULL bugs (caught by practice #5), merge row multiplication (practice #7 — caught a 300% row inflation in a J&J pipeline), and SettingWithCopyWarning cascades (practice #3). Code review time dropped by 50% and production incidents from SQL/Pandas logic errors fell to near zero.

* * *

## 📋 Complete Quick Reference Card

* * *

### Method Reference Table — SQL vs Pandas

Operation| SQL Syntax| Pandas Equivalent  
---|---|---  
**Filter single**| `WHERE col = val`| `df[df['col'] == val]`  
**Filter multiple AND**| `WHERE a = x AND b = y`| `df[(df.a==x) & (df.b==y)]`  
**Filter OR**| `WHERE a = x OR b = y`| `df[(df.a==x) | (df.b==y)]`  
**Filter IN list**| `WHERE col IN (a, b)`| `df[df.col.isin([a,b])]`  
**Filter NOT IN**| `WHERE col NOT IN (a,b)`| `df[~df.col.isin([a,b])]`  
**NULL filter**| `WHERE col IS NULL`| `df[df.col.isna()]`  
**NOT NULL**| `WHERE col IS NOT NULL`| `df[df.col.notna()]`  
**Range filter**| `WHERE col BETWEEN a AND b`| `df[df.col.between(a,b)]`  
**Pattern match**| `WHERE col LIKE '%val%'`| `df[df.col.str.contains('val')]`  
**Count rows**| `COUNT(*)`| `len(df)` or `df.shape[0]`  
**Count non-null**| `COUNT(col)`| `df.col.count()`  
**Sum**| `SUM(col)`| `df.col.sum()`  
**Average**| `AVG(col)`| `df.col.mean()`  
**Min / Max**| `MIN(col)` / `MAX(col)`| `df.col.min()` / `.max()`  
**Group aggregate**| `GROUP BY col`| `df.groupby('col').agg()`  
**Filter groups**| `HAVING count > n`| `groupby → filter aggregated df`  
**Running total**| `SUM() OVER (ORDER BY)`| `df.col.cumsum()`  
**Rank**| `RANK() OVER (ORDER BY)`| `df.col.rank(method='min')`  
**Dense rank**| `DENSE_RANK()`| `df.col.rank(method='dense')`  
**Row number**| `ROW_NUMBER()`| `df.col.rank(method='first')`  
**Lag**| `LAG(col, 1)`| `df.col.shift(1)`  
**Lead**| `LEAD(col, 1)`| `df.col.shift(-1)`  
**Partition stat**| `AVG() OVER (PARTITION BY g)`| `groupby.transform('mean')`  
**Inner join**| `INNER JOIN t ON key`| `pd.merge(df1,df2,on=key,how='inner')`  
**Left join**| `LEFT JOIN t ON key`| `pd.merge(...,how='left')`  
**Full outer join**| `FULL OUTER JOIN`| `pd.merge(...,how='outer')`  
**Anti-join**| `WHERE NOT EXISTS (...)`| `df[~df.key.isin(other.key)]`  
**Union all**| `UNION ALL`| `pd.concat([df1,df2])`  
**Union distinct**| `UNION`| `pd.concat([df1,df2]).drop_duplicates()`  
**Intersect**| `INTERSECT`| `set(df1.key) & set(df2.key)`  
**Deduplicate**| `SELECT DISTINCT`| `df.drop_duplicates()`  
**Pivot**| `PIVOT / CASE WHEN`| `df.pivot_table()`  
**Unpivot**| `UNPIVOT / CROSS APPLY`| `df.melt()`  
**Sort**| `ORDER BY col DESC`| `df.sort_values('col',ascending=False)`  
**Top N**| `TOP 10 / LIMIT 10`| `df.head(10)` or `df.nlargest(10,col)`  
**Conditional**| `CASE WHEN x THEN y`| `np.select(conditions,choices)`  
**Null replace**| `COALESCE(col, default)`| `df.col.fillna(default)`  
**Null divide safe**| `NULLIF(col, 0)`| `df.col.replace(0, np.nan)`  
**Type cast**| `CAST(col AS INT)`| `df.col.astype(int)`  
**String upper**| `UPPER(col)`| `df.col.str.upper()`  
**String trim**| `TRIM(col)`| `df.col.str.strip()`  
**String replace**| `REPLACE(col,'a','b')`| `df.col.str.replace('a','b')`  
**Date extract**| `EXTRACT(MONTH FROM date)`| `df.date_col.dt.month`  
**Date diff**| `DATEDIFF(day, d1, d2)`| `(df.d2 - df.d1).dt.days`  
**Date truncate**| `DATE_TRUNC('month', date)`| `df.date.dt.to_period('M')`  
**Resample time**| `GROUP BY DATE_TRUNC()`| `df.resample('ME').agg()`  
**Std deviation**| `STDDEV(col)`| `df.col.std()`  
**Correlation**| `CORR(col1, col2)`| `df[['c1','c2']].corr()`  
**Percentile**| `PERCENTILE_CONT(0.95)`| `df.col.quantile(0.95)`  
**Upsert**| `MERGE INTO target USING source`| `concat + drop_duplicates + mask`  
**Profile**| `SELECT COUNT, AVG, MIN, MAX`| `df.describe()` / custom profile fn  
  
* * *

### Common Pitfalls & How to Avoid Them

Pitfall| Wrong| Right  
---|---|---  
**NULL in NOT IN**| `WHERE id NOT IN (SELECT id FROM t)` if t has NULLs → returns 0 rows| `WHERE NOT EXISTS (SELECT 1 FROM t WHERE t.id = c.id)`  
**WHERE vs HAVING**| `WHERE COUNT(*) > 5` → error| `HAVING COUNT(*) > 5`  
**Row multiplication in JOIN**|  No cardinality check| `validate='many_to_one'` in Pandas; `COUNT(*)` check in SQL  
**SettingWithCopyWarning**| `df[mask]['col'] = val`| `df.loc[mask,'col'] = val`  
**apply() on large DF**| `df.apply(row_func, axis=1)`| `np.select()` or vectorised ops  
**UNION when UNION ALL suffices**| `UNION` (sorts+deduplicates)| `UNION ALL` (faster, no dedup)  
**Not guarding division**| `a / b` (errors on b=0)| `a / NULLIF(b,0)` or `np.where(b!=0, a/b, np.nan)`  
**SELECT * in production**| `SELECT *`| Explicit column list  
**Not using .copy()**| `subset = df[mask]` then modify| `subset = df[mask].copy()`  
**Skipping NULL audit**|  Processing data without checking NULLs| Always run `df.isnull().sum()` first  
  
* * *

_End of Document_ _Core Data Analyst Methods — Complete Q &A with SQL + Pandas + Synthetic Data_ _STAR Format | 38 Questions | 18 Method Categories_ _All examples reference insurance analytics context (J &J, Travelers, AXA, LSEG)_ _Generated: April 2026_
