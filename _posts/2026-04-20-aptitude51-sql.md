---
title: "Aptitude51 - SQL"
date: 2026-04-20 15:37:53 
author: shajeebhameed
layout: page
slug: aptitude51-sql
status: publish
---

## 50 Toughest SQL Technical Interview Questions — STAR Format

### Senior / Business Data Analyst – EDW / Banking / SQL Deep-Dive

_All answers reference real experience: LSEG/Refinitiv, J &J Global Services, Google, Travelers Insurance, AXA Insurance, Cisco_

* * *

> **STAR Key:**
> 
>   * 🔵 **S** ituation — The context and background
>   * 🟡 **T** ask — Your specific responsibility
>   * 🟢 **A** ction — Exactly what you did, with SQL where applicable
>   * 🔴 **R** esult — The measurable outcome
> 


* * *

## Section 1: Advanced Window Functions & Analytics (Q1–10)

* * *

### Q1. How would you calculate a year-over-year percentage change using window functions, without using a self-join?

🔵 **Situation** At J&J Global Services, the actuarial team needed a rolling year-over-year comparison of incurred claims by product line — previously done using a self-join that was taking over 8 minutes on a 200M-row claims table.

🟡 **Task** Rewrite the YoY calculation using window functions to eliminate the self-join and bring runtime within the 2-minute SLA.

🟢 **Action** Used `LAG()` partitioned by `product_line` to pull the prior year's value within the same window pass:
    
    
    WITH yearly_claims AS (
      SELECT
        product_line,
        EXTRACT(YEAR FROM claim_date)          AS claim_year,
        SUM(incurred_amount)                   AS total_incurred
      FROM claims
      GROUP BY product_line, EXTRACT(YEAR FROM claim_date)
    ),
    yoy AS (
      SELECT
        product_line,
        claim_year,
        total_incurred,
        LAG(total_incurred) OVER (
          PARTITION BY product_line
          ORDER BY claim_year
        )                                        AS prior_year_incurred
      FROM yearly_claims
    )
    SELECT
      product_line,
      claim_year,
      total_incurred,
      prior_year_incurred,
      ROUND(
        (total_incurred - prior_year_incurred) / NULLIF(prior_year_incurred, 0) * 100,
        2
      )                                          AS yoy_pct_change
    FROM yoy
    ORDER BY product_line, claim_year;
    
    
    

Key points:

  * `NULLIF(prior_year_incurred, 0)` prevents divide-by-zero for new product lines.
  * `LAG()` with no offset defaults to 1 — previous partition row.
  * The CTE pre-aggregates before windowing to reduce the row set the window function operates on.



🔴 **Result** Query runtime dropped from 8+ minutes to **47 seconds**. Actuarial team got the YoY trend report daily rather than weekly. The pattern was adopted for 4 other trend reports on the same platform.

* * *

### Q2. Explain how you would find the top N records per group without using a subquery or `LIMIT`.

🔵 **Situation** At LSEG, the risk team needed the top 3 largest open positions per currency pair per trading day — a report that ran 6 times daily. The existing LIMIT-based subquery approach broke when different currency pairs had the same position size (ties not handled correctly).

🟡 **Task** Rewrite the query using window functions to correctly handle ties and work within BigQuery (which doesn't support `LIMIT` inside subqueries at the time).

🟢 **Action**
    
    
    WITH ranked_positions AS (
      SELECT
        trade_date,
        currency_pair,
        account_id,
        open_position,
        DENSE_RANK() OVER (
          PARTITION BY trade_date, currency_pair
          ORDER BY open_position DESC
        ) AS pos_rank
      FROM open_positions
      WHERE position_status = 'OPEN'
    )
    SELECT
      trade_date,
      currency_pair,
      account_id,
      open_position,
      pos_rank
    FROM ranked_positions
    WHERE pos_rank <= 3
    ORDER BY trade_date, currency_pair, pos_rank;
    
    
    

Why `DENSE_RANK()` over `ROW_NUMBER()`:

  * `ROW_NUMBER()` would arbitrarily pick one record when two accounts have equal positions — producing inconsistent results.
  * `DENSE_RANK()` returns all tied records at the same rank, ensuring no genuine top position is missed.
  * `RANK()` was rejected because it skips rank numbers on ties (rank 1, 1, 3) — the business wanted consecutive rank labels.



🔴 **Result** The rewrite correctly surfaced tied positions that the previous query was silently dropping. Two risk positions that had been invisible for weeks were immediately flagged to the trading desk. Report runtime improved by 35%.

* * *

### Q3. How do you calculate a running total that resets at the start of each month?

🔵 **Situation** At Travelers Insurance, the fraud team needed a month-to-date running total of flagged claims — resetting to zero at the start of each calendar month — to track whether fraud volumes were on track against monthly targets.

🟡 **Task** Build the MTD running total in the Gold layer using a single window function expression without procedural logic.

🟢 **Action**
    
    
    SELECT
      claim_date,
      account_id,
      claim_amount,
      fraud_flag,
      SUM(CASE WHEN fraud_flag = 1 THEN claim_amount ELSE 0 END) OVER (
        PARTITION BY
          EXTRACT(YEAR  FROM claim_date),
          EXTRACT(MONTH FROM claim_date)   -- reset per month
        ORDER BY claim_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
      ) AS mtd_fraud_amount,
    
      COUNT(CASE WHEN fraud_flag = 1 THEN 1 END) OVER (
        PARTITION BY
          EXTRACT(YEAR  FROM claim_date),
          EXTRACT(MONTH FROM claim_date)
        ORDER BY claim_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
      ) AS mtd_fraud_count
    FROM claims
    ORDER BY claim_date;
    
    
    

Key design decisions:

  * `PARTITION BY YEAR, MONTH` forces the window to restart at month boundaries.
  * `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` ensures the running total accumulates correctly even if multiple claims share the same date.
  * `CASE WHEN fraud_flag = 1` filters within the SUM — no need for a pre-filtered CTE.



🔴 **Result** The fraud team's MTD dashboard was rebuilt around this query. The pattern also supported a `rolling_7d_fraud_amount` variant (ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) that fed a real-time anomaly detection alert.

* * *

### Q4. How would you calculate a median value in SQL without a built-in MEDIAN function?

🔵 **Situation** At Google (Street View), the research team needed median image processing latency by region and week. BigQuery at the time had `PERCENTILE_CONT` as an aggregate window function but it could not be used directly inside a GROUP BY — an issue that confused the junior analysts on the team.

🟡 **Task** Implement median latency calculation in BigQuery SQL correctly, explaining the syntax constraints to the team.

🟢 **Action**
    
    
    -- BigQuery approach: PERCENTILE_CONT as analytic function
    WITH latency_data AS (
      SELECT
        region,
        DATE_TRUNC(capture_ts, WEEK)             AS week_start,
        processing_latency_mins
      FROM image_processing_log
      WHERE processing_latency_mins IS NOT NULL
    ),
    medians AS (
      SELECT
        region,
        week_start,
        processing_latency_mins,
        PERCENTILE_CONT(processing_latency_mins, 0.5) OVER (
          PARTITION BY region, week_start
        ) AS median_latency
      FROM latency_data
    )
    SELECT DISTINCT
      region,
      week_start,
      median_latency
    FROM medians
    ORDER BY region, week_start;
    
    -- Alternative (T-SQL / standard SQL) using ROW_NUMBER for databases
    -- without PERCENTILE_CONT:
    WITH numbered AS (
      SELECT
        region,
        processing_latency_mins,
        ROW_NUMBER() OVER (PARTITION BY region ORDER BY processing_latency_mins) AS rn,
        COUNT(*)     OVER (PARTITION BY region)                                  AS total_rows
      FROM image_processing_log
    )
    SELECT
      region,
      AVG(CAST(processing_latency_mins AS FLOAT)) AS median_latency
    FROM numbered
    WHERE rn IN (
      (total_rows + 1) / 2,
      (total_rows + 2) / 2
    )
    GROUP BY region;
    
    
    

The `ROW_NUMBER` approach works by selecting the middle row(s): for odd counts the middle row; for even counts the two middle rows and averaging them.

🔴 **Result** Median latency analysis identified a regional outlier (APAC) where median was 3x the global median — invisible in mean calculations because of a small number of extremely fast images skewing the average. The finding fed directly into the 25% pipeline optimisation.

* * *

### Q5. How would you detect gaps in a sequential series (e.g., missing trade IDs or dates) using SQL?

🔵 **Situation** At LSEG, the compliance team suspected that some trade records were being dropped silently by the Oracle-to-SQL Server sync — trade IDs should be sequential within a trading day, and any gap indicated a missing record.

🟡 **Task** Write a query to detect all gaps in the sequential trade ID series for any given trading day.

🟢 **Action**
    
    
    -- Method 1: LAG to find where next expected ID != actual next ID
    WITH sequenced AS (
      SELECT
        trade_date,
        trade_id,
        LEAD(trade_id) OVER (
          PARTITION BY trade_date
          ORDER BY trade_id
        ) AS next_trade_id
      FROM trades
      WHERE trade_date = '2024-03-15'
    )
    SELECT
      trade_date,
      trade_id            AS gap_after_id,
      next_trade_id       AS next_found_id,
      next_trade_id - trade_id - 1 AS missing_count
    FROM sequenced
    WHERE next_trade_id - trade_id > 1   -- gap detected
    ORDER BY trade_id;
    
    -- Method 2: Generate expected sequence and anti-join
    WITH expected AS (
      SELECT generate_series(
        (SELECT MIN(trade_id) FROM trades WHERE trade_date = '2024-03-15'),
        (SELECT MAX(trade_id) FROM trades WHERE trade_date = '2024-03-15')
      ) AS expected_id
    )
    SELECT e.expected_id AS missing_trade_id
    FROM expected e
    LEFT JOIN trades t
      ON  t.trade_id  = e.expected_id
      AND t.trade_date = '2024-03-15'
    WHERE t.trade_id IS NULL;
    
    
    

Method 1 using `LEAD()` is preferable for large datasets as it avoids the generate_series overhead. Method 2 is clearer when the full list of missing IDs (not just gap boundaries) is needed.

🔴 **Result** The gap detection script found 4 missing trade records over a 2-week audit period — all from a 3-minute window during a network interruption that the sync service had logged as a warning but not a failure. The missing trades were recovered from the Oracle archive. The script was scheduled as a nightly audit job thereafter.

* * *

### Q6. How would you pivot rows into columns dynamically without knowing the column values in advance?

🔵 **Situation** At J&J Global Services, the finance team needed a pivot report showing monthly claim totals per product line as columns — but product lines were added regularly and the report needed to work without code changes each time a new product launched.

🟡 **Task** Build a dynamic pivot in SQL (T-SQL on SQL Server) that automatically included any new product lines without a stored procedure rewrite.

🟢 **Action**
    
    
    -- Step 1: Build the dynamic column list
    DECLARE @cols      NVARCHAR(MAX);
    DECLARE @sql       NVARCHAR(MAX);
    DECLARE @pivot_sql NVARCHAR(MAX);
    
    -- Fetch distinct product lines ordered alphabetically
    SELECT @cols = STRING_AGG(
      QUOTENAME(product_line), ', '
    ) WITHIN GROUP (ORDER BY product_line)
    FROM (SELECT DISTINCT product_line FROM claims) AS c;
    
    -- Step 2: Build the dynamic PIVOT query
    SET @pivot_sql = N'
    SELECT claim_month, ' + @cols + '
    FROM (
      SELECT
        FORMAT(claim_date, ''yyyy-MM'') AS claim_month,
        product_line,
        incurred_amount
      FROM claims
    ) AS src
    PIVOT (
      SUM(incurred_amount)
      FOR product_line IN (' + @cols + ')
    ) AS pvt
    ORDER BY claim_month;';
    
    -- Step 3: Execute
    EXEC sp_executesql @pivot_sql;
    
    
    

Key considerations:

  * `QUOTENAME()` wraps column names in brackets — prevents SQL injection and handles names with spaces.
  * `STRING_AGG` (SQL Server 2017+) replaces the older FOR XML PATH trick.
  * For BigQuery: use `EXECUTE IMMEDIATE` with a similar pattern, or use a conditional aggregation approach:


    
    
    -- BigQuery conditional aggregation (no native PIVOT):
    SELECT
      FORMAT_DATE('%Y-%m', claim_date) AS claim_month,
      SUM(CASE WHEN product_line = 'Motor'    THEN incurred_amount END) AS Motor,
      SUM(CASE WHEN product_line = 'Property' THEN incurred_amount END) AS Property,
      SUM(CASE WHEN product_line = 'Health'   THEN incurred_amount END) AS Health
      -- columns added as new product lines launch
    FROM claims
    GROUP BY 1
    ORDER BY 1;
    
    
    

🔴 **Result** The dynamic pivot report ran daily without any code maintenance overhead. When 2 new product lines launched in Q2, the report automatically included them with zero developer involvement — saving an estimated 2 days of rework per product launch.

* * *

### Q7. How would you identify the first and last transaction for each customer within a session using SQL?

🔵 **Situation** At Travelers Insurance, the fraud analytics team needed to identify the first and last claim submitted per policyholder per calendar year — to detect "burst" fraud patterns where a policyholder submits multiple claims rapidly before going dormant.

🟡 **Task** Write a query to return both the first and last claim per policyholder per year in a single pass without self-joins.

🟢 **Action**
    
    
    WITH claim_ordered AS (
      SELECT
        policyholder_id,
        claim_id,
        claim_date,
        claim_amount,
        fraud_flag,
        EXTRACT(YEAR FROM claim_date)                            AS claim_year,
    
        ROW_NUMBER() OVER (
          PARTITION BY policyholder_id, EXTRACT(YEAR FROM claim_date)
          ORDER BY claim_date ASC
        )                                                        AS rn_first,
    
        ROW_NUMBER() OVER (
          PARTITION BY policyholder_id, EXTRACT(YEAR FROM claim_date)
          ORDER BY claim_date DESC
        )                                                        AS rn_last,
    
        COUNT(*)       OVER (
          PARTITION BY policyholder_id, EXTRACT(YEAR FROM claim_date)
        )                                                        AS total_claims,
    
        DATEDIFF(
          MAX(claim_date) OVER (PARTITION BY policyholder_id, EXTRACT(YEAR FROM claim_date)),
          MIN(claim_date) OVER (PARTITION BY policyholder_id, EXTRACT(YEAR FROM claim_date))
        )                                                        AS days_span
      FROM claims
    ),
    first_last AS (
      SELECT
        policyholder_id,
        claim_year,
        total_claims,
        days_span,
        MAX(CASE WHEN rn_first = 1 THEN claim_id   END) AS first_claim_id,
        MAX(CASE WHEN rn_first = 1 THEN claim_date END) AS first_claim_date,
        MAX(CASE WHEN rn_first = 1 THEN claim_amount END) AS first_claim_amount,
        MAX(CASE WHEN rn_last  = 1 THEN claim_id   END) AS last_claim_id,
        MAX(CASE WHEN rn_last  = 1 THEN claim_date END) AS last_claim_date,
        MAX(CASE WHEN rn_last  = 1 THEN claim_amount END) AS last_claim_amount
      FROM claim_ordered
      GROUP BY policyholder_id, claim_year, total_claims, days_span
    )
    SELECT *,
      CASE WHEN total_claims >= 3 AND days_span <= 30
           THEN 'HIGH RISK BURST'
           ELSE 'NORMAL'
      END AS fraud_pattern_flag
    FROM first_last
    ORDER BY fraud_pattern_flag DESC, total_claims DESC;
    
    
    

🔴 **Result** The burst fraud flag immediately identified 47 policyholders with 3+ claims within a 30-day window — a cohort the fraud team had not previously been able to systematically identify. Six were escalated for investigation; two resulted in fraud case referrals.

* * *

### Q8. How would you calculate a cumulative distribution (percentile rank) across a dataset?

🔵 **Situation** At Google, the performance team needed to understand how each Street View image's processing time ranked against the full distribution — identifying which images were in the slowest 10% (95th percentile and above) for targeted optimisation.

🟡 **Task** Build a percentile ranking query in BigQuery that returned each image's latency alongside its percentile position in the overall distribution.

🟢 **Action**
    
    
    SELECT
      image_id,
      region,
      processing_latency_mins,
    
      -- Percentile rank: what % of images are faster than this one?
      PERCENT_RANK() OVER (
        ORDER BY processing_latency_mins
      )                                     AS overall_pct_rank,
    
      -- Cumulative distribution: what % of images are <= this latency?
      CUME_DIST() OVER (
        ORDER BY processing_latency_mins
      )                                     AS cumulative_dist,
    
      -- Decile bucket (1 = fastest 10%, 10 = slowest 10%)
      NTILE(10) OVER (
        ORDER BY processing_latency_mins
      )                                     AS latency_decile,
    
      -- Region-specific percentile (how does this image rank within its region?)
      PERCENT_RANK() OVER (
        PARTITION BY region
        ORDER BY processing_latency_mins
      )                                     AS region_pct_rank
    
    FROM image_processing_log
    WHERE processing_date = CURRENT_DATE - 1;
    
    
    

Difference between `PERCENT_RANK` and `CUME_DIST`:

  * `PERCENT_RANK`: (rank - 1) / (total rows - 1) → ranges 0 to 1, excludes current row from denominator.
  * `CUME_DIST`: rank / total rows → always > 0, includes current row. More appropriate for "what percentage are at or below this value."



🔴 **Result** The decile analysis showed the slowest 10% of images (decile 10) had latency 14x the median. Cross-referencing with the region column revealed 80% were from a single ingestion node — the bottleneck that drove the 25% pipeline optimisation.

* * *

### Q9. How do you calculate a ratio-to-report (each row as a percentage of its group total)?

🔵 **Situation** At J&J Global Services, the executive team wanted a claims report showing each region's contribution as a percentage of the total — both globally and within each product line — in a single query without post-processing in Excel.

🟡 **Task** Write a SQL query that simultaneously calculated each region's share of global total claims and share of product-line total claims.

🟢 **Action**
    
    
    SELECT
      region,
      product_line,
      SUM(incurred_amount)                    AS region_product_amount,
    
      -- Share of global total
      ROUND(
        SUM(incurred_amount) /
        SUM(SUM(incurred_amount)) OVER ()     -- OVER() = grand total window
        * 100, 2
      )                                       AS pct_of_global_total,
    
      -- Share of product line total
      ROUND(
        SUM(incurred_amount) /
        SUM(SUM(incurred_amount)) OVER (
          PARTITION BY product_line           -- window resets per product
        )
        * 100, 2
      )                                       AS pct_of_product_total,
    
      -- Share of region total
      ROUND(
        SUM(incurred_amount) /
        SUM(SUM(incurred_amount)) OVER (
          PARTITION BY region
        )
        * 100, 2
      )                                       AS pct_of_region_total
    
    FROM claims
    WHERE claim_year = 2024
    GROUP BY region, product_line
    ORDER BY region, product_line;
    
    
    

Key technique: `SUM(SUM(...)) OVER (PARTITION BY ...)` — the inner SUM is the GROUP BY aggregate; the outer SUM with OVER is the window function running over those aggregated rows. This avoids a subquery or second CTE pass.

🔴 **Result** The executive dashboard was rebuilt using this pattern, eliminating an Excel post-processing step that had been taking an analyst 2 hours each month. The report surfaced that one product line contributed 60% of incurred claims but only 35% of earned premium — a finding that directly fed into the $2M+ cost savings analysis.

* * *

### Q10. How would you solve the "islands and gaps" problem — grouping consecutive rows that share the same value?

🔵 **Situation** At LSEG, the operations team needed to group consecutive minutes of trading activity where the FX volatility flag was in the same state (HIGH/NORMAL) — to calculate how long each "volatility episode" lasted and how many trades occurred during it.

🟡 **Task** Implement the classic "islands and gaps" SQL pattern to group consecutive rows sharing the same volatility state into contiguous episodes.

🟢 **Action**
    
    
    -- Step 1: Assign a group ID to each island using the
    --         ROW_NUMBER difference technique
    WITH flagged AS (
      SELECT
        minute_ts,
        currency_pair,
        volatility_flag,
        trade_count,
        ROW_NUMBER() OVER (
          PARTITION BY currency_pair
          ORDER BY minute_ts
        )                                         AS rn_overall,
    
        ROW_NUMBER() OVER (
          PARTITION BY currency_pair, volatility_flag
          ORDER BY minute_ts
        )                                         AS rn_within_flag
      FROM trading_minutes
    ),
    islands AS (
      SELECT
        minute_ts,
        currency_pair,
        volatility_flag,
        trade_count,
        -- When overall RN and within-flag RN move together,
        -- their DIFFERENCE stays constant within the same island
        rn_overall - rn_within_flag               AS island_id
      FROM flagged
    )
    SELECT
      currency_pair,
      volatility_flag,
      island_id,
      MIN(minute_ts)                              AS episode_start,
      MAX(minute_ts)                              AS episode_end,
      DATEDIFF(MINUTE, MIN(minute_ts), MAX(minute_ts)) + 1
                                                  AS duration_minutes,
      SUM(trade_count)                            AS total_trades,
      COUNT(*)                                    AS minute_count
    FROM islands
    GROUP BY currency_pair, volatility_flag, island_id
    ORDER BY currency_pair, episode_start;
    
    
    

The genius of the technique: for consecutive rows with the same flag, `rn_overall` increments by 1 and `rn_within_flag` also increments by 1 — their difference stays constant, identifying the island. When the flag changes, `rn_within_flag` resets — creating a new constant difference (new island).

🔴 **Result** The volatility episode analysis revealed that 73% of reconciliation breaks at LSEG occurred during HIGH volatility episodes longer than 20 minutes — a critical insight that led to implementing enhanced sync frequency specifically during these windows.

* * *

## Section 2: Query Optimisation & Performance (Q11–18)

* * *

### Q11. How do you approach optimising a query that uses multiple CTEs and runs slowly?

🔵 **Situation** At J&J, a key actuarial query used 7 chained CTEs and took 22 minutes to run — it was re-run after every parameter change, making iterative analysis near-impossible.

🟡 **Task** Diagnose and optimise the query without changing the business logic or output.

🟢 **Action**

  * Ran the execution plan — found that the query optimiser was materialising 3 of the 7 CTEs multiple times because they were referenced in more than one downstream CTE.
  * In SQL Server, CTEs are not cached — each reference re-executes the CTE definition. Replaced the 3 multi-referenced CTEs with **temp tables** to force materialisation:


    
    
    -- Before: CTE referenced 3 times downstream (executed 3x)
    WITH claims_base AS (
      SELECT ... FROM claims WHERE claim_year = 2024
    )
    SELECT ... FROM claims_base ...   -- ref 1
    UNION ALL
    SELECT ... FROM claims_base ...   -- ref 2 (re-executes!)
    JOIN claims_base ...               -- ref 3 (re-executes again!)
    
    -- After: materialise into temp table (executed once)
    CREATE TABLE #claims_base AS
    SELECT ... FROM claims WHERE claim_year = 2024;
    
    CREATE INDEX idx_cb ON #claims_base(product_line, region);
    
    SELECT ... FROM #claims_base ...  -- reads cached temp table
    UNION ALL
    SELECT ... FROM #claims_base ...
    JOIN #claims_base ...
    
    
    

  * Additionally, pushed filter predicates earlier: moved `WHERE claim_year = 2024` from the outer query into the base CTE/temp table, reducing the working set by 80%.
  * Replaced a correlated subquery in CTE 5 with a window function.



🔴 **Result** Runtime dropped from 22 minutes to **2 minutes 40 seconds**. Actuarial analysts could now iterate on parameter changes in near real-time. The temp table + early filter pattern was documented as a team standard.

* * *

### Q12. How do you handle query performance when joining two very large tables?

🔵 **Situation** At LSEG, a reconciliation query joined the full Oracle trade table (~500M rows) with the SQL Server positions table (~300M rows) — a join that ran for over an hour and frequently caused memory pressure on the reporting server.

🟡 **Task** Redesign the join strategy to make the reconciliation query run within the 15-minute SLA.

🟢 **Action** Applied a multi-stage optimisation approach:
    
    
    -- Stage 1: Pre-filter BOTH sides to only today's data BEFORE joining
    -- (Filter pushdown — reduce join inputs from 500M to ~2M rows)
    WITH today_trades AS (
      SELECT trade_id, currency_pair, amount, status
      FROM trades
      WHERE trade_date = CAST(GETDATE() AS DATE)   -- ~2M rows vs 500M
    ),
    today_positions AS (
      SELECT trade_id, net_position, account_id
      FROM positions
      WHERE position_date = CAST(GETDATE() AS DATE) -- ~1.5M rows vs 300M
    )
    -- Stage 2: Join the filtered subsets only
    SELECT
      t.trade_id,
      t.currency_pair,
      t.amount,
      t.status,
      p.net_position,
      p.account_id
    FROM today_trades  t
    JOIN today_positions p ON t.trade_id = p.trade_id;
    
    
    

Additional actions:

  * Added **partitioning** on both tables by `trade_date`/`position_date` — each partition ~2M rows; the join now only scanned today's partition.
  * Changed join hint from default HASH JOIN to MERGE JOIN after confirming both sides were ordered on `trade_id` — merge join is faster when inputs are pre-sorted.
  * Disabled auto-statistics update during the join window; refreshed statistics manually post-load instead.



🔴 **Result** Reconciliation query runtime dropped from 60+ minutes to **11 minutes** — within SLA. Server memory pressure incidents dropped to zero. Partitioning also improved all other queries on these tables by 40–60%.

* * *

### Q13. How do you identify and resolve a parameter sniffing problem in SQL Server?

🔵 **Situation** At LSEG, a stored procedure for generating daily position reports ran in 30 seconds when called with common currency pairs (USD/EUR) but took 18 minutes for exotic pairs with low trade volumes — the same execution plan was being reused for both cases due to parameter sniffing.

🟡 **Task** Diagnose and fix the parameter sniffing issue causing catastrophic plan reuse for uncommon parameter values.

🟢 **Action**
    
    
    -- Diagnosis: check cached plans for the procedure
    SELECT
      qs.execution_count,
      qs.total_elapsed_time / qs.execution_count AS avg_elapsed_ms,
      qs.min_elapsed_time,
      qs.max_elapsed_time,
      SUBSTRING(st.text, (qs.statement_start_offset/2)+1,
        ((qs.statement_end_offset - qs.statement_start_offset)/2)+1) AS query_text
    FROM sys.dm_exec_query_stats qs
    CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
    WHERE st.text LIKE '%daily_position_report%'
    ORDER BY qs.max_elapsed_time DESC;
    
    -- Fix 1: OPTION(RECOMPILE) forces fresh plan per execution
    -- Use when parameter distributions vary wildly
    CREATE PROCEDURE dbo.get_daily_positions
      @currency_pair VARCHAR(10),
      @trade_date    DATE
    AS
    BEGIN
      SELECT ...
      FROM positions
      WHERE currency_pair = @currency_pair
      AND   trade_date    = @trade_date
      OPTION (RECOMPILE);  -- new plan compiled each call
    END;
    
    -- Fix 2: Local variable assignment breaks sniffing
    -- Use when RECOMPILE overhead is unacceptable
    CREATE PROCEDURE dbo.get_daily_positions_v2
      @currency_pair VARCHAR(10),
      @trade_date    DATE
    AS
    BEGIN
      DECLARE @cp VARCHAR(10) = @currency_pair;  -- copy to local var
      DECLARE @td DATE        = @trade_date;     -- optimizer can't sniff local vars
    
      SELECT ...
      FROM positions
      WHERE currency_pair = @cp
      AND   trade_date    = @td;
    END;
    
    -- Fix 3: OPTIMIZE FOR UNKNOWN — tell optimiser to use average stats
    -- Use for general-purpose procedures with moderate variation
      OPTION (OPTIMIZE FOR (@currency_pair UNKNOWN, @trade_date UNKNOWN));
    
    
    

The chosen fix was `OPTION(RECOMPILE)` for the exotic pairs procedure (called infrequently, fresh plan worth the compilation cost) and local variables for the high-frequency common pairs procedure.

🔴 **Result** Exotic currency pair report runtime dropped from 18 minutes to 45 seconds. The common pair procedure maintained 30-second performance. Both procedures now used optimal plans for their respective data distributions.

* * *

### Q14. How would you rewrite a cursor-based ETL process in SQL Server to a set-based operation?

🔵 **Situation** At Cisco (via Wipro), the legacy TelePresence ETL pipeline used a cursor to loop through 500,000+ conference records one-by-one, applying a status classification rule. The nightly ETL was taking 5+ hours.

🟡 **Task** Rewrite the cursor-based processing as a set-based operation to bring the ETL within the 2-hour batch window.

🟢 **Action**
    
    
    -- BEFORE: Cursor approach (one row at a time — catastrophically slow)
    DECLARE @conf_id INT, @duration INT, @quality_score FLOAT;
    DECLARE cur CURSOR FOR
      SELECT conference_id, duration_mins, quality_score FROM staging_conferences;
    
    OPEN cur;
    FETCH NEXT FROM cur INTO @conf_id, @duration, @quality_score;
    
    WHILE @@FETCH_STATUS = 0
    BEGIN
      UPDATE target_conferences
      SET status_category =
        CASE
          WHEN @duration < 5  AND @quality_score < 3.0 THEN 'FAILED_SHORT'
          WHEN @duration < 5                            THEN 'SHORT'
          WHEN @quality_score < 3.0                     THEN 'POOR_QUALITY'
          WHEN @quality_score >= 4.5                    THEN 'HIGH_QUALITY'
          ELSE                                               'STANDARD'
        END
      WHERE conference_id = @conf_id;
    
      FETCH NEXT FROM cur INTO @conf_id, @duration, @quality_score;
    END;
    CLOSE cur; DEALLOCATE cur;
    
    -- AFTER: Set-based UPDATE with CASE (processes all rows in one operation)
    UPDATE t
    SET t.status_category =
      CASE
        WHEN s.duration_mins < 5  AND s.quality_score < 3.0 THEN 'FAILED_SHORT'
        WHEN s.duration_mins < 5                             THEN 'SHORT'
        WHEN s.quality_score < 3.0                           THEN 'POOR_QUALITY'
        WHEN s.quality_score >= 4.5                          THEN 'HIGH_QUALITY'
        ELSE                                                      'STANDARD'
      END
    FROM target_conferences t
    JOIN staging_conferences s ON t.conference_id = s.conference_id;
    -- One UPDATE statement replaces 500,000 individual UPDATE calls
    
    
    

For complex multi-step cursor logic, staged temp table updates are still set-based:
    
    
    -- Batch MERGE for more complex upsert logic
    MERGE target_conferences AS tgt
    USING staging_conferences AS src
    ON tgt.conference_id = src.conference_id
    WHEN MATCHED AND src.load_ts > tgt.load_ts THEN
      UPDATE SET tgt.status_category = src.status_category,
                 tgt.load_ts         = src.load_ts
    WHEN NOT MATCHED BY TARGET THEN
      INSERT (conference_id, status_category, load_ts)
      VALUES (src.conference_id, src.status_category, src.load_ts);
    
    
    

🔴 **Result** ETL runtime dropped from 5 hours to **under 90 minutes** — a 70% improvement. The set-based approach also eliminated deadlock contention that the cursor had been causing on the target table during batch processing.

* * *

### Q15. How do you use query hints and when should you avoid them?

🔵 **Situation** At LSEG, a critical reconciliation query was consistently choosing a nested loop join plan that performed well for small datasets but degraded catastrophically when end-of-month trade volumes were 10x normal.

🟡 **Task** Stabilise the query plan for high-volume month-end processing while preserving normal performance for standard days.

🟢 **Action**
    
    
    -- Diagnostic: view current plan hints and statistics
    SELECT * FROM sys.dm_exec_query_plan(
      (SELECT plan_handle FROM sys.dm_exec_requests WHERE session_id = @@SPID)
    );
    
    -- Apply HASH JOIN hint for large datasets (overrides nested loop choice)
    SELECT
      t.trade_id,
      t.amount,
      p.net_position
    FROM trades t
    JOIN positions p
      ON t.trade_id = p.trade_id
    OPTION (HASH JOIN);             -- forces hash join for large inputs
    
    -- For month-end specifically — use MAXDOP to parallelise
    SELECT ...
    FROM trades t
    JOIN positions p ON t.trade_id = p.trade_id
    OPTION (HASH JOIN, MAXDOP 4);  -- 4 parallel threads for month-end batch
    
    -- AVOID hints in these scenarios:
    -- 1. NOLOCK (READ UNCOMMITTED) — reads dirty data; dangerous in trading/finance
    --    SELECT * FROM trades WITH (NOLOCK)  ← NEVER in financial contexts
    
    -- 2. FORCE ORDER — prevents optimiser from reordering joins; only use if
    --    you have profiled and are certain of the optimal join order
    
    -- 3. INDEX hints — brittle; breaks if index is renamed or rebuilt
    --    SELECT * FROM trades WITH (INDEX = idx_trade_date)  ← fragile
    
    
    

Resolution: implemented a **Plan Guide** to apply the HASH JOIN hint only when row estimates exceeded a threshold — preserving the nested loop plan for normal days:
    
    
    EXEC sp_create_plan_guide
      @name = N'PG_reconciliation_monthend',
      @stmt = N'SELECT t.trade_id... FROM trades t JOIN positions p...',
      @type = N'SQL',
      @hints = N'OPTION(HASH JOIN, MAXDOP 4)';
    
    
    

🔴 **Result** Month-end reconciliation runtime stabilised at under 15 minutes regardless of volume. Normal-day performance unchanged. The plan guide approach was documented as preferred over inline hints as it could be managed centrally without code changes.

* * *

### Q16. How do you handle NULL values correctly in complex SQL expressions to avoid silent errors?

🔵 **Situation** At J&J, an actuarial loss ratio report was producing subtly incorrect results — the loss ratio for a small number of product lines was showing as NULL rather than a number. The issue had gone unnoticed for 2 months because NULL was silently excluded from aggregate calculations.

🟡 **Task** Audit the query for all NULL propagation risks and implement defensive NULL handling throughout.

🟢 **Action**
    
    
    -- PROBLEM: NULL propagates silently through arithmetic
    -- incurred / earned_premium → NULL if either is NULL
    -- NULL in SUM() → excluded from total silently
    
    -- SOLUTION: Comprehensive NULL handling
    SELECT
      product_line,
    
      -- COALESCE for default values (prefer over ISNULL — ANSI standard)
      COALESCE(SUM(incurred_amount), 0)             AS total_incurred,
      COALESCE(SUM(earned_premium),  0)             AS total_earned,
    
      -- NULLIF prevents divide-by-zero (returns NULL instead of error)
      -- then COALESCE handles the NULL result
      COALESCE(
        SUM(incurred_amount) / NULLIF(SUM(earned_premium), 0),
        0
      )                                             AS loss_ratio,
    
      -- NULL-safe equality comparison (IS NOT DISTINCT FROM in standard SQL)
      -- For: "match rows where region is the same, including both-NULL case"
      COUNT(CASE
        WHEN a.region IS NOT DISTINCT FROM b.region THEN 1
      END)                                          AS region_matches,
    
      -- Aggregation: NULL excluded from COUNT(*) vs COUNT(col)
      COUNT(*)                                      AS total_rows,       -- includes NULLs
      COUNT(incurred_amount)                        AS non_null_incurred, -- excludes NULLs
      COUNT(*) - COUNT(incurred_amount)             AS null_incurred_count,
    
      -- String concatenation: NULL + anything = NULL
      -- Use CONCAT() which treats NULL as empty string
      CONCAT(
        COALESCE(region, 'UNKNOWN'), ' - ',
        COALESCE(product_line, 'UNCLASSIFIED')
      )                                             AS region_product_label
    
    FROM claims
    GROUP BY product_line
    HAVING COALESCE(SUM(incurred_amount), 0) > 0;  -- exclude zero-claim products
    
    
    

🔴 **Result** The audit identified 6 locations in the query where NULLs were silently affecting results. After fixing, 3 product lines that had been showing NULL loss ratios now displayed correct values. One had a loss ratio of 145% — a critical data point that had been invisible for 2 months.

* * *

### Q17. How do you use `EXCEPT`, `INTERSECT`, and `MINUS` for data comparison, and when is each most useful?

🔵 **Situation** At LSEG, after each nightly Oracle-to-SQL Server sync, the team needed a quick way to identify records present in one system but not the other — a reconciliation pattern needed multiple times per day.

🟡 **Task** Build a set-based reconciliation query using EXCEPT/INTERSECT that was fast, readable, and easily adapted to different table comparisons.

🟢 **Action**
    
    
    -- EXCEPT: rows in LEFT that do NOT exist in RIGHT
    -- Use: find trades in Oracle that haven't synced to SQL Server yet
    SELECT trade_id, trade_date, currency_pair, amount, status
    FROM oracle_trades_staging      -- source of truth
    EXCEPT
    SELECT trade_id, trade_date, currency_pair, amount, status
    FROM sql_server_trades;         -- target
    
    -- INTERSECT: rows that exist in BOTH sets (exact match on all columns)
    -- Use: confirm which records synced perfectly (all fields identical)
    SELECT trade_id, trade_date, currency_pair, amount, status
    FROM oracle_trades_staging
    INTERSECT
    SELECT trade_id, trade_date, currency_pair, amount, status
    FROM sql_server_trades;
    
    -- Reconciliation summary: combine both in one query
    SELECT 'In Oracle only (not synced)'      AS category, COUNT(*) AS record_count
    FROM (
      SELECT trade_id FROM oracle_trades_staging
      EXCEPT
      SELECT trade_id FROM sql_server_trades
    ) x
    UNION ALL
    SELECT 'In SQL Server only (orphan)'      AS category, COUNT(*)
    FROM (
      SELECT trade_id FROM sql_server_trades
      EXCEPT
      SELECT trade_id FROM oracle_trades_staging
    ) x
    UNION ALL
    SELECT 'Matched in both (full row match)' AS category, COUNT(*)
    FROM (
      SELECT trade_id, currency_pair, amount, status FROM oracle_trades_staging
      INTERSECT
      SELECT trade_id, currency_pair, amount, status FROM sql_server_trades
    ) x;
    
    
    

Important nuances:

  * `EXCEPT`/`INTERSECT` treat **NULLs as equal** — unlike `=` operator where NULL ≠ NULL.
  * Both operators implicitly deduplicate (like UNION). Use `EXCEPT ALL` / `INTERSECT ALL` to retain duplicates where supported.
  * For large tables, a LEFT JOIN with NULL check on the right side is often faster as it's more optimiser-friendly.



🔴 **Result** The reconciliation query ran in under 2 minutes on 2M daily trade records and became the first check run after every sync. It detected the 4 missing trade records in the gap analysis and confirmed the 99.9% consistency rate.

* * *

### Q18. How would you write a SQL query to detect outliers using statistical methods?

🔵 **Situation** At Travelers Insurance, the fraud analytics team needed a systematic way to flag claims whose amounts were statistically unusual — not just exceeding a fixed threshold, but outliers relative to each claim category's own distribution.

🟡 **Task** Implement a statistical outlier detection approach in SQL that adapted to the distribution of each claim category separately.

🟢 **Action**
    
    
    -- Method 1: Z-Score (standard deviations from the mean)
    -- Flag records > 3 standard deviations from their category mean
    WITH category_stats AS (
      SELECT
        claim_category,
        AVG(claim_amount)                       AS cat_mean,
        STDDEV(claim_amount)                    AS cat_stddev,
        COUNT(*)                                AS cat_count
      FROM claims
      GROUP BY claim_category
      HAVING COUNT(*) >= 30   -- Z-score unreliable for small samples
    ),
    z_scored AS (
      SELECT
        c.claim_id,
        c.claim_category,
        c.claim_amount,
        s.cat_mean,
        s.cat_stddev,
        (c.claim_amount - s.cat_mean) / NULLIF(s.cat_stddev, 0)   AS z_score
      FROM claims c
      JOIN category_stats s ON c.claim_category = s.claim_category
    )
    SELECT *,
      CASE WHEN ABS(z_score) > 3 THEN 'OUTLIER' ELSE 'NORMAL' END AS outlier_flag
    FROM z_scored
    ORDER BY ABS(z_score) DESC;
    
    -- Method 2: IQR (Interquartile Range) — more robust for skewed distributions
    -- Common in insurance where claim amounts are heavily right-skewed
    WITH quartiles AS (
      SELECT
        claim_category,
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY claim_amount) AS q1,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY claim_amount) AS q3
      FROM claims
      GROUP BY claim_category
    ),
    iqr_bounds AS (
      SELECT
        claim_category,
        q1,
        q3,
        q3 - q1                    AS iqr,
        q1 - 1.5 * (q3 - q1)      AS lower_bound,
        q3 + 1.5 * (q3 - q1)      AS upper_bound
      FROM quartiles
    )
    SELECT
      c.claim_id,
      c.claim_category,
      c.claim_amount,
      b.lower_bound,
      b.upper_bound,
      CASE
        WHEN c.claim_amount < b.lower_bound THEN 'LOW OUTLIER'
        WHEN c.claim_amount > b.upper_bound THEN 'HIGH OUTLIER'
        ELSE 'NORMAL'
      END AS outlier_flag
    FROM claims c
    JOIN iqr_bounds b ON c.claim_category = b.claim_category
    ORDER BY outlier_flag, c.claim_amount DESC;
    
    
    

Chose IQR over Z-score for claim amounts because claim distributions are heavily right-skewed — Z-score assumes normal distribution and would miss high outliers in skewed data.

🔴 **Result** IQR-based outlier flagging identified 312 claims in the first run — 0.4% of the dataset. Manual review of the top 50 flagged claims found 11 with clear fraud indicators that had not been caught by the existing rules-based system. The model was embedded into the daily Gold layer pipeline.

* * *

## Section 3: Advanced Joins, Subqueries & Set Operations (Q19–26)

* * *

### Q19. How would you write a query to find records with no matching row in another table — without using `NOT IN`?

🔵 **Situation** At LSEG, a daily check was needed to find trades in the Oracle system that had no corresponding settlement record in the SQL Server settlements table. The existing query used `NOT IN` and was failing silently when the settlements table contained any NULL `trade_id` — a common data quality issue.

🟡 **Task** Rewrite the orphan detection query to handle NULLs correctly and perform reliably at scale.

🟢 **Action**
    
    
    -- PROBLEM: NOT IN silently returns zero rows if subquery contains any NULL
    -- (NULL comparison logic: trade_id NOT IN (..., NULL, ...) → always UNKNOWN → no rows)
    SELECT trade_id FROM trades
    WHERE trade_id NOT IN (SELECT trade_id FROM settlements);
    -- ↑ Returns empty set if settlements.trade_id has even one NULL row
    
    -- FIX 1: NOT EXISTS (preferred — NULL-safe, often faster)
    SELECT t.trade_id, t.trade_date, t.currency_pair
    FROM trades t
    WHERE NOT EXISTS (
      SELECT 1
      FROM settlements s
      WHERE s.trade_id = t.trade_id
    )
    AND t.trade_date = CURRENT_DATE;
    
    -- FIX 2: LEFT JOIN with NULL check (transparent, optimiser-friendly)
    SELECT t.trade_id, t.trade_date, t.currency_pair
    FROM trades t
    LEFT JOIN settlements s ON t.trade_id = s.trade_id
    WHERE s.trade_id IS NULL    -- no match found in settlements
    AND   t.trade_date = CURRENT_DATE;
    
    -- FIX 3: EXCEPT (set-based, clean — deduplicates automatically)
    SELECT trade_id FROM trades WHERE trade_date = CURRENT_DATE
    EXCEPT
    SELECT trade_id FROM settlements WHERE settlement_date = CURRENT_DATE;
    
    -- Performance: NOT EXISTS typically fastest (stops on first match),
    -- LEFT JOIN is most optimiser-friendly for large tables with indexes,
    -- EXCEPT is cleanest syntax but slightly slower due to deduplication step.
    
    
    

🔴 **Result** The `NOT EXISTS` rewrite revealed 23 unsettled trades that the `NOT IN` version had been silently missing due to NULL `trade_id` values in the settlements table from a data entry issue. All 23 were reconciled the same day. The NULL data entry issue was fixed at source.

* * *

### Q20. How do you write a self-join and when is it useful?

🔵 **Situation** At J&J Global Services, the HR analytics team (using the same EDW) needed to identify manager-subordinate pairs where the manager's salary was lower than their direct report's — a data anomaly indicating either data errors or compensation band violations.

🟡 **Task** Write a self-join query on the employee table to compare manager and subordinate salaries within the same table.

🟢 **Action**
    
    
    -- Self-join: same table referenced twice with different aliases
    SELECT
      mgr.employee_id    AS manager_id,
      mgr.employee_name  AS manager_name,
      mgr.salary         AS manager_salary,
      mgr.job_grade      AS manager_grade,
    
      sub.employee_id    AS subordinate_id,
      sub.employee_name  AS subordinate_name,
      sub.salary         AS subordinate_salary,
      sub.job_grade      AS subordinate_grade,
    
      sub.salary - mgr.salary AS salary_gap
    
    FROM employees sub                               -- subordinate alias
    JOIN employees mgr                               -- manager alias
      ON sub.manager_id = mgr.employee_id
    WHERE sub.salary > mgr.salary                    -- anomaly condition
      AND sub.is_active = 1
      AND mgr.is_active = 1
    ORDER BY salary_gap DESC;
    
    -- Extension: recursive self-join for full hierarchy traversal
    WITH RECURSIVE hierarchy AS (
      -- Anchor: top-level managers (no manager above them)
      SELECT employee_id, employee_name, manager_id, salary, 1 AS depth
      FROM employees WHERE manager_id IS NULL
    
      UNION ALL
    
      -- Recursive: join each employee to their manager in the hierarchy
      SELECT e.employee_id, e.employee_name, e.manager_id, e.salary, h.depth + 1
      FROM employees e
      JOIN hierarchy h ON e.manager_id = h.employee_id
    )
    SELECT * FROM hierarchy ORDER BY depth, manager_id;
    
    
    

🔴 **Result** The self-join identified 14 manager-subordinate salary inversions — all confirmed as data entry errors in the source HR system. Corrections were made within the week. The query was added as a monthly data quality check.

* * *

### Q21. How would you implement a MERGE (UPSERT) statement and what are the common pitfalls?

🔵 **Situation** At J&J Global Services, the Salesforce-to-EDW pipeline needed to handle both new records (INSERT) and updates to existing ones (UPDATE) in a single atomic operation — previously done as separate DELETE + INSERT which caused brief data gaps visible to report users.

🟡 **Task** Redesign the load pattern using MERGE to eliminate the data gap and handle the insert/update/delete logic in one statement.

🟢 **Action**
    
    
    MERGE target_accounts AS tgt
    USING (
      SELECT
        account_id,
        account_name,
        account_status,
        annual_revenue,
        last_modified_date,
        -- Hash of all updateable fields for change detection
        HASHBYTES('SHA2_256',
          CONCAT(account_name, '|', account_status, '|',
                 CAST(annual_revenue AS VARCHAR))
        ) AS row_hash
      FROM salesforce_staging
      WHERE batch_date = CAST(GETDATE() AS DATE)
    ) AS src
    ON tgt.account_id = src.account_id
    
    -- Update only if something has actually changed (hash comparison)
    WHEN MATCHED AND tgt.row_hash <> src.row_hash THEN
      UPDATE SET
        tgt.account_name      = src.account_name,
        tgt.account_status    = src.account_status,
        tgt.annual_revenue    = src.annual_revenue,
        tgt.last_modified_date = src.last_modified_date,
        tgt.row_hash          = src.row_hash,
        tgt.updated_ts        = GETUTCDATE()
    
    -- Insert new accounts
    WHEN NOT MATCHED BY TARGET THEN
      INSERT (account_id, account_name, account_status,
              annual_revenue, last_modified_date, row_hash, created_ts)
      VALUES (src.account_id, src.account_name, src.account_status,
              src.annual_revenue, src.last_modified_date, src.row_hash, GETUTCDATE())
    
    -- Mark deleted accounts (soft delete — don't physically remove)
    WHEN NOT MATCHED BY SOURCE
      AND tgt.is_active = 1 THEN
      UPDATE SET tgt.is_active = 0, tgt.deleted_ts = GETUTCDATE()
    
    OUTPUT $action AS merge_action,
           inserted.account_id,
           deleted.account_id AS old_account_id
    INTO merge_audit_log (merge_action, new_id, old_id);
    
    
    

Common MERGE pitfalls:

  1. **Duplicate source rows** → MERGE throws an error if source has duplicate join keys. Always deduplicate source first.
  2. **No index on join key in target** → full table scan on target per source row. Always index the merge key.
  3. **WHEN NOT MATCHED BY SOURCE** → deletes/updates ALL unmatched target rows, not just today's. Always add a filter on the target side.
  4. **Race conditions** → MERGE is not atomic under default isolation. Use `WITH (HOLDLOCK)` hint on the target for concurrency safety.



🔴 **Result** The MERGE-based load eliminated the DELETE + INSERT data gap. Report users saw no more "dip to zero" in account counts during the nightly load window. The hash-based change detection also reduced unnecessary UPDATE operations by 70%, improving overall pipeline performance.

* * *

### Q22. How do you efficiently query a hierarchical table (adjacency list model)?

🔵 **Situation** At J&J Global Services, the corporate legal entity structure was stored as an adjacency list (entity_id, parent_entity_id) — a standard pattern that became a problem when credit risk analysts needed to roll up exposure across entire subsidiary trees, which could be 8–10 levels deep.

🟡 **Task** Write a query to traverse an arbitrarily deep entity hierarchy and aggregate credit exposure at every level.

🟢 **Action**
    
    
    -- Recursive CTE to traverse the hierarchy
    WITH RECURSIVE entity_tree AS (
      -- Anchor: start from a specific parent (or all root nodes)
      SELECT
        entity_id,
        parent_entity_id,
        entity_name,
        entity_id                     AS root_entity_id,
        entity_name                   AS root_entity_name,
        0                             AS depth,
        CAST(entity_id AS VARCHAR(500)) AS path    -- track traversal path
      FROM legal_entities
      WHERE parent_entity_id IS NULL    -- root nodes
    
      UNION ALL
    
      -- Recursive: join each entity to its parent in the tree
      SELECT
        e.entity_id,
        e.parent_entity_id,
        e.entity_name,
        t.root_entity_id,
        t.root_entity_name,
        t.depth + 1,
        CONCAT(t.path, ' > ', e.entity_id)
      FROM legal_entities e
      JOIN entity_tree t ON e.parent_entity_id = t.entity_id
      WHERE t.depth < 20              -- guard against circular references
    ),
    exposure_rollup AS (
      SELECT
        t.root_entity_id,
        t.root_entity_name,
        SUM(e.credit_exposure)        AS total_group_exposure,
        COUNT(DISTINCT t.entity_id)   AS entity_count,
        MAX(t.depth)                  AS max_depth
      FROM entity_tree t
      JOIN credit_exposure e ON t.entity_id = e.entity_id
      GROUP BY t.root_entity_id, t.root_entity_name
    )
    SELECT
      r.*,
      -- Flag groups exceeding concentration limit
      CASE WHEN r.total_group_exposure > 50000000
           THEN 'CONCENTRATION BREACH'
           ELSE 'WITHIN LIMIT'
      END AS limit_status
    FROM exposure_rollup r
    ORDER BY total_group_exposure DESC;
    
    
    

Guard against infinite loops: `WHERE t.depth < 20` stops runaway recursion from circular parent-child references (a common data quality issue in legal entity tables).

🔴 **Result** The recursive rollup identified 3 corporate groups where sub-entity exposure individually appeared within limits but group-level aggregation breached the concentration threshold — a regulatory concern that flat queries had been missing entirely.

* * *

### Q23. How do you write a query to compare two tables and show differences at the row and column level?

🔵 **Situation** At AXA Insurance, during the parallel run phase of the legacy migration, the team needed a daily comparison between the legacy system's output and the new platform's output — not just row counts, but identifying which specific fields differed when records matched by key but had different values.

🟡 **Task** Write a comprehensive data comparison query that identified both missing rows and field-level differences for matched rows.

🟢 **Action**
    
    
    -- Part 1: Missing rows (in legacy but not new platform)
    SELECT 'MISSING_IN_NEW' AS diff_type, claim_id, NULL AS field_name,
           NULL AS legacy_value, NULL AS new_value
    FROM legacy_claims
    WHERE claim_id NOT IN (SELECT claim_id FROM new_claims)
    
    UNION ALL
    
    -- Part 2: Extra rows (in new platform but not legacy)
    SELECT 'EXTRA_IN_NEW', claim_id, NULL, NULL, NULL
    FROM new_claims
    WHERE claim_id NOT IN (SELECT claim_id FROM legacy_claims)
    
    UNION ALL
    
    -- Part 3: Field-level differences for matched rows (UNPIVOT approach)
    SELECT
      'FIELD_MISMATCH'     AS diff_type,
      l.claim_id,
      v.field_name,
      v.legacy_value,
      v.new_value
    FROM legacy_claims l
    JOIN new_claims n ON l.claim_id = n.claim_id
    CROSS APPLY (
      VALUES
        ('claim_amount',   CAST(l.claim_amount   AS VARCHAR), CAST(n.claim_amount   AS VARCHAR)),
        ('claim_status',   l.claim_status,                    n.claim_status),
        ('claimant_name',  l.claimant_name,                   n.claimant_name),
        ('policy_number',  l.policy_number,                   n.policy_number),
        ('approval_date',  CAST(l.approval_date  AS VARCHAR), CAST(n.approval_date  AS VARCHAR))
    ) AS v(field_name, legacy_value, new_value)
    WHERE v.legacy_value <> v.new_value            -- only show actual differences
       OR (v.legacy_value IS NULL AND v.new_value IS NOT NULL)
       OR (v.legacy_value IS NOT NULL AND v.new_value IS NULL)
    
    ORDER BY diff_type, claim_id, field_name;
    
    
    

The `CROSS APPLY VALUES` pattern (UNPIVOT equivalent) is key — it turns column differences into rows, making the output easy to filter and analyse regardless of which field differs.

🔴 **Result** The comparison query ran daily throughout the 4-week parallel run. It identified 7 specific data discrepancies (mostly date format and currency rounding differences) before go-live, all resolved before cutover. The query became the standard migration validation tool for subsequent projects.

* * *

### Q24. How do you write an efficient query to find duplicate records based on business key, keeping only the most recent?

🔵 **Situation** At J&J Global Services, the Salesforce staging table accumulated duplicate account records when the same account was modified multiple times in a single day — each modification produced a new row. The nightly load needed to keep only the most recent version of each account.

🟡 **Task** Write a deduplication query that was idempotent (safe to re-run), scalable to 10M+ rows, and preserved the most recent record per account.

🟢 **Action**
    
    
    -- Method 1: ROW_NUMBER with DELETE (SQL Server)
    -- Most flexible — can incorporate complex tie-breaking logic
    WITH deduped AS (
      SELECT
        staging_id,
        account_id,
        last_modified_date,
        ROW_NUMBER() OVER (
          PARTITION BY account_id              -- business key
          ORDER BY last_modified_date DESC,    -- keep newest
                   staging_id DESC             -- tie-break on insertion order
        ) AS rn
      FROM salesforce_staging
      WHERE batch_date = CAST(GETDATE() AS DATE)
    )
    DELETE FROM salesforce_staging
    WHERE staging_id IN (
      SELECT staging_id FROM deduped WHERE rn > 1
    );
    
    -- Method 2: INSERT ... SELECT with deduplication (BigQuery / analytics DWH)
    -- Best for write-once immutable tables
    INSERT INTO salesforce_accounts_clean
    SELECT * EXCEPT(rn)
    FROM (
      SELECT
        *,
        ROW_NUMBER() OVER (
          PARTITION BY account_id
          ORDER BY last_modified_date DESC
        ) AS rn
      FROM salesforce_staging
      WHERE batch_date = CURRENT_DATE
    ) WHERE rn = 1;
    
    -- Method 3: QUALIFY (BigQuery native — cleanest syntax)
    SELECT *
    FROM salesforce_staging
    WHERE batch_date = CURRENT_DATE
    QUALIFY ROW_NUMBER() OVER (
      PARTITION BY account_id
      ORDER BY last_modified_date DESC
    ) = 1;
    
    
    

`QUALIFY` in BigQuery is the most elegant approach — it filters on window function results directly in the SELECT without a wrapping subquery.

🔴 **Result** Deduplication reduced the staging table load from 1.2M records to 800K records per day — a 33% reduction. The MERGE step downstream ran proportionally faster. The idempotent design meant accidental re-runs on the same batch had zero impact.

* * *

### Q25. How would you write a query to calculate sessionisation (grouping events into sessions based on a time gap threshold)?

🔵 **Situation** At Google (Street View), the QA team needed to group image review events into "review sessions" — consecutive review actions by the same analyst, where a gap of more than 30 minutes between actions defined a session boundary. This was needed to analyse analyst productivity and identify sessions where error rates spiked.

🟡 **Task** Write a SQL query that assigned each review event to a session ID, using a 30-minute inactivity gap as the session boundary.

🟢 **Action**
    
    
    WITH event_gaps AS (
      SELECT
        analyst_id,
        event_ts,
        event_type,
        image_id,
        review_outcome,
    
        -- Time since previous event for same analyst
        LAG(event_ts) OVER (
          PARTITION BY analyst_id
          ORDER BY event_ts
        )                                                AS prev_event_ts,
    
        -- Flag = 1 if this event starts a new session (gap > 30 min or first event)
        CASE
          WHEN LAG(event_ts) OVER (
                 PARTITION BY analyst_id ORDER BY event_ts
               ) IS NULL
            OR TIMESTAMPDIFF(MINUTE,
                 LAG(event_ts) OVER (PARTITION BY analyst_id ORDER BY event_ts),
                 event_ts
               ) > 30
          THEN 1
          ELSE 0
        END                                              AS is_session_start
      FROM review_events
    ),
    session_assigned AS (
      SELECT
        *,
        -- Cumulative sum of session_start flags = session number per analyst
        SUM(is_session_start) OVER (
          PARTITION BY analyst_id
          ORDER BY event_ts
          ROWS UNBOUNDED PRECEDING
        )                                                AS session_number
      FROM event_gaps
    )
    SELECT
      analyst_id,
      session_number,
      -- Derive a globally unique session ID
      CONCAT(analyst_id, '-', session_number)            AS session_id,
      MIN(event_ts)                                      AS session_start,
      MAX(event_ts)                                      AS session_end,
      COUNT(*)                                           AS events_in_session,
      TIMESTAMPDIFF(MINUTE, MIN(event_ts), MAX(event_ts)) AS session_duration_mins,
      AVG(CASE WHEN review_outcome = 'ERROR' THEN 1.0 ELSE 0 END) AS error_rate
    FROM session_assigned
    GROUP BY analyst_id, session_number
    ORDER BY analyst_id, session_number;
    
    
    

The key technique: `SUM(is_session_start) OVER (ORDER BY event_ts ROWS UNBOUNDED PRECEDING)` — cumulative sum of the start flags creates a monotonically increasing session counter that increments exactly at each session boundary.

🔴 **Result** Sessionisation analysis revealed that error rates were 3x higher in sessions immediately following a gap of 2+ hours — suggesting analyst fatigue or context-switching. QA team scheduling was adjusted to reduce long gaps, improving review accuracy by 15%.

* * *

### Q26. How do you handle slowly changing dimensions in SQL using Type 2 SCD logic?

🔵 **Situation** At Travelers Insurance, a critical compliance requirement mandated that actuarial models could be re-run against the exact policyholder profile that existed at any point in time — requiring full history tracking on the `dim_policyholder` table.

🟡 **Task** Implement Type 2 SCD logic in SQL to track all attribute changes on `dim_policyholder` with full temporal validity tracking.

🟢 **Action**
    
    
    -- SCD Type 2 implementation using MERGE
    MERGE dim_policyholder AS tgt
    USING (
      SELECT
        policyholder_id,
        full_name,
        address,
        risk_tier,
        coverage_level,
        -- Hash of tracked attributes for efficient change detection
        HASHBYTES('SHA2_256',
          CONCAT(full_name, '|', address, '|', risk_tier, '|', coverage_level)
        ) AS attr_hash
      FROM policyholder_staging
      WHERE batch_date = CAST(GETDATE() AS DATE)
    ) AS src
    ON tgt.policyholder_id = src.policyholder_id
    AND tgt.is_current = 1    -- only match against the current version
    
    -- CASE 1: Matched and attributes changed → expire current row
    WHEN MATCHED AND tgt.attr_hash <> src.attr_hash THEN
      UPDATE SET
        tgt.is_current      = 0,
        tgt.effective_to    = CAST(GETDATE() AS DATE) - 1,
        tgt.updated_ts      = GETUTCDATE()
    
    -- CASE 2: New policyholder → insert as current row
    WHEN NOT MATCHED BY TARGET THEN
      INSERT (policyholder_id, full_name, address, risk_tier,
              coverage_level, attr_hash, effective_from,
              effective_to, is_current, created_ts)
      VALUES (src.policyholder_id, src.full_name, src.address, src.risk_tier,
              src.coverage_level, src.attr_hash, CAST(GETDATE() AS DATE),
              '9999-12-31', 1, GETUTCDATE());
    
    -- CASE 3: Changed records → also need a new current row
    -- MERGE can't do UPDATE + INSERT in same match clause.
    -- Handle with a second INSERT after the MERGE:
    INSERT INTO dim_policyholder
      (policyholder_id, full_name, address, risk_tier,
       coverage_level, attr_hash, effective_from, effective_to,
       is_current, created_ts)
    SELECT
      src.policyholder_id, src.full_name, src.address, src.risk_tier,
      src.coverage_level, src.attr_hash, CAST(GETDATE() AS DATE),
      '9999-12-31', 1, GETUTCDATE()
    FROM policyholder_staging src
    JOIN dim_policyholder tgt
      ON  tgt.policyholder_id = src.policyholder_id
      AND tgt.is_current = 0                    -- was just expired by MERGE
      AND tgt.effective_to = CAST(GETDATE() AS DATE) - 1  -- expired today
    WHERE src.batch_date = CAST(GETDATE() AS DATE);
    
    -- Point-in-time query using the SCD Type 2 dimension:
    SELECT p.*, c.incurred_amount
    FROM claims c
    JOIN dim_policyholder p
      ON  p.policyholder_id = c.policyholder_id
      AND c.claim_date BETWEEN p.effective_from AND p.effective_to;
    -- Returns the policyholder's profile as it was on the claim date
    
    
    

🔴 **Result** The Type 2 SCD implementation unblocked a regulatory reserving report that had been on hold for 2 months — actuarial teams could now run point-in-time models accurately. The approach was validated by the external actuarial auditors as compliant with reserving methodology standards.

* * *

## Section 4: Data Reconciliation & Integrity (Q27–33)

* * *

### Q27. How do you write a SQL-based reconciliation framework for a financial data pipeline?

🔵 **Situation** At LSEG, the FX reconciliation between Oracle and SQL Server had no systematic framework — discrepancies were found ad hoc by traders. A formal automated framework was needed to meet the firm's data control standards.

🟡 **Task** Design and implement a SQL-based reconciliation framework that ran automatically and produced an auditable reconciliation report.

🟢 **Action**
    
    
    -- Reconciliation framework: multi-level checks in a single stored procedure
    
    CREATE PROCEDURE run_daily_reconciliation
      @recon_date DATE = NULL
    AS
    BEGIN
      SET @recon_date = COALESCE(@recon_date, CAST(GETDATE() AS DATE));
    
      -- Clear previous results for this date (idempotent)
      DELETE FROM recon_results WHERE recon_date = @recon_date;
    
      -- LEVEL 1: Row count check
      INSERT INTO recon_results (recon_date, check_level, check_name,
                                  source_value, target_value, variance, status)
      SELECT
        @recon_date,
        1,
        'Row Count',
        (SELECT COUNT(*) FROM oracle_trades  WHERE trade_date = @recon_date),
        (SELECT COUNT(*) FROM sqlsvr_trades  WHERE trade_date = @recon_date),
        ABS(
          (SELECT COUNT(*) FROM oracle_trades  WHERE trade_date = @recon_date) -
          (SELECT COUNT(*) FROM sqlsvr_trades  WHERE trade_date = @recon_date)
        ),
        CASE WHEN
          (SELECT COUNT(*) FROM oracle_trades WHERE trade_date = @recon_date) =
          (SELECT COUNT(*) FROM sqlsvr_trades WHERE trade_date = @recon_date)
        THEN 'PASS' ELSE 'FAIL' END;
    
      -- LEVEL 2: Sum of notional by currency pair
      INSERT INTO recon_results
      SELECT
        @recon_date, 2,
        CONCAT('Notional Sum - ', o.currency_pair),
        o.oracle_sum,
        s.sqlsvr_sum,
        ABS(o.oracle_sum - COALESCE(s.sqlsvr_sum, 0)),
        CASE WHEN ABS(o.oracle_sum - COALESCE(s.sqlsvr_sum, 0)) < 0.01
             THEN 'PASS' ELSE 'FAIL' END
      FROM (
        SELECT currency_pair, SUM(notional_amount) AS oracle_sum
        FROM oracle_trades WHERE trade_date = @recon_date
        GROUP BY currency_pair
      ) o
      FULL OUTER JOIN (
        SELECT currency_pair, SUM(notional_amount) AS sqlsvr_sum
        FROM sqlsvr_trades WHERE trade_date = @recon_date
        GROUP BY currency_pair
      ) s ON o.currency_pair = s.currency_pair;
    
      -- LEVEL 3: Record-level hash check for any Level 2 failures
      INSERT INTO recon_results
      SELECT
        @recon_date, 3,
        CONCAT('Record Hash - ', o.trade_id),
        CONVERT(VARCHAR, HASHBYTES('SHA2_256',
          CONCAT(o.currency_pair,'|',o.notional_amount,'|',o.status)), 2),
        CONVERT(VARCHAR, HASHBYTES('SHA2_256',
          CONCAT(s.currency_pair,'|',s.notional_amount,'|',s.status)), 2),
        NULL,
        CASE WHEN
          HASHBYTES('SHA2_256', CONCAT(o.currency_pair,'|',o.notional_amount,'|',o.status)) =
          HASHBYTES('SHA2_256', CONCAT(s.currency_pair,'|',s.notional_amount,'|',s.status))
        THEN 'PASS' ELSE 'FAIL' END
      FROM oracle_trades o
      JOIN sqlsvr_trades s ON o.trade_id = s.trade_id
      WHERE o.trade_date = @recon_date
      AND o.currency_pair IN (
        SELECT currency_pair FROM recon_results
        WHERE recon_date = @recon_date AND check_level = 2 AND status = 'FAIL'
      );
    
      -- Summary output
      SELECT check_level, check_name, status, variance
      FROM recon_results
      WHERE recon_date = @recon_date
      ORDER BY check_level, status DESC;
    END;
    
    
    

🔴 **Result** The 3-level reconciliation framework detected discrepancies within 2 minutes of each sync. Reconciliation issues fell by **85%** because problems were caught immediately rather than hours later. The framework's audit log was accepted by the internal control team as evidence of continuous data monitoring.

* * *

### Q28. How do you detect and handle data type mismatches between source and target systems?

🔵 **Situation** At AXA Insurance, the legacy-to-modern migration revealed intermittent precision mismatches in financial amounts — values that looked correct to 2 decimal places but differed at the 6th decimal place, causing reconciliation totals to diverge by small but unacceptable amounts.

🟡 **Task** Build a SQL diagnostic to identify all data type and precision mismatches across the 200+ columns being migrated.

🟢 **Action**
    
    
    -- Step 1: Compare schema metadata between source and target
    SELECT
      s.column_name,
      s.data_type       AS source_type,
      s.numeric_precision AS source_precision,
      s.numeric_scale   AS source_scale,
      t.data_type       AS target_type,
      t.numeric_precision AS target_precision,
      t.numeric_scale   AS target_scale,
      CASE
        WHEN s.data_type <> t.data_type              THEN 'TYPE MISMATCH'
        WHEN s.numeric_precision > t.numeric_precision THEN 'PRECISION LOSS RISK'
        WHEN s.numeric_scale > t.numeric_scale       THEN 'SCALE LOSS RISK'
        ELSE 'OK'
      END AS mismatch_type
    FROM information_schema.columns s
    JOIN information_schema.columns t
      ON  s.column_name  = t.column_name
      AND s.table_name   = 'legacy_claims'
      AND t.table_name   = 'new_claims'
    WHERE s.data_type <> t.data_type
       OR s.numeric_scale > t.numeric_scale
    ORDER BY mismatch_type, s.column_name;
    
    -- Step 2: Detect actual value-level precision differences on numeric columns
    SELECT
      l.claim_id,
      'claim_amount'                         AS field_name,
      l.claim_amount                         AS legacy_value,
      n.claim_amount                         AS new_value,
      l.claim_amount - n.claim_amount        AS raw_difference,
      ABS(l.claim_amount - n.claim_amount)   AS abs_difference
    FROM legacy_claims l
    JOIN new_claims    n ON l.claim_id = n.claim_id
    WHERE ABS(l.claim_amount - n.claim_amount) > 0.000001  -- below rounding tolerance
      AND ABS(l.claim_amount - n.claim_amount) < 1.00      -- but not a genuine error
    ORDER BY abs_difference DESC;
    
    
    

Root cause fix: the Oracle `NUMBER(18,8)` columns were being stored as SQL Server `FLOAT` — which uses binary floating-point and cannot exactly represent all decimal fractions. Fix: change target to `DECIMAL(18,8)`.

🔴 **Result** The precision mismatch was identified and fixed before go-live. Post-fix reconciliation showed zero variance on all previously affected columns. The schema comparison query became a standard pre-migration checklist item for all subsequent data migration projects.

* * *

### Q29. How would you write a SQL query to perform a full audit trail comparison between pipeline runs?

🔵 **Situation** At Travelers Insurance, regulators requested a full audit trail showing what changed in the claims data between two consecutive daily pipeline runs — to demonstrate that no unauthorised changes were being made to historical records.

🟡 **Task** Design a SQL query that compared two snapshots of the claims table and classified every change as INSERT, UPDATE, or DELETE.

🟢 **Action**
    
    
    -- Snapshot comparison using full row hash
    -- Assumes daily snapshots are stored with a snapshot_date partition
    
    WITH yesterday AS (
      SELECT claim_id, claim_amount, claim_status, adjuster_id, updated_ts,
        HASHBYTES('SHA2_256',
          CONCAT(claim_id,'|',claim_amount,'|',claim_status,'|',adjuster_id)
        ) AS row_hash
      FROM claims_snapshot WHERE snapshot_date = CAST(GETDATE()-1 AS DATE)
    ),
    today AS (
      SELECT claim_id, claim_amount, claim_status, adjuster_id, updated_ts,
        HASHBYTES('SHA2_256',
          CONCAT(claim_id,'|',claim_amount,'|',claim_status,'|',adjuster_id)
        ) AS row_hash
      FROM claims_snapshot WHERE snapshot_date = CAST(GETDATE() AS DATE)
    )
    SELECT
      COALESCE(t.claim_id, y.claim_id)     AS claim_id,
      CASE
        WHEN y.claim_id IS NULL            THEN 'INSERT'   -- new today
        WHEN t.claim_id IS NULL            THEN 'DELETE'   -- removed today
        WHEN t.row_hash <> y.row_hash      THEN 'UPDATE'   -- changed
        ELSE                                    'UNCHANGED'
      END                                  AS change_type,
      y.claim_amount                       AS prev_amount,
      t.claim_amount                       AS curr_amount,
      y.claim_status                       AS prev_status,
      t.claim_status                       AS curr_status,
      GETUTCDATE()                         AS audit_ts
    FROM today  t
    FULL OUTER JOIN yesterday y ON t.claim_id = y.claim_id
    WHERE NOT (t.claim_id IS NOT NULL AND y.claim_id IS NOT NULL
               AND t.row_hash = y.row_hash)  -- exclude unchanged rows
    ORDER BY change_type, claim_id;
    
    
    

🔴 **Result** The audit trail query satisfied the regulatory request within 24 hours. The output showed 0 unauthorised historical record modifications over the 3-month period reviewed. The query was formalised as a monthly automated audit report, retained for 7 years per the firm's data retention policy.

* * *

### Q30. How would you write SQL to check referential integrity across tables without relying on database-enforced foreign keys?

🔵 **Situation** At J&J Global Services, the BigQuery EDW intentionally had no foreign key constraints enforced (BigQuery doesn't enforce them) — but referential integrity violations were causing silent data gaps in actuarial reports where dimension lookups were failing.

🟡 **Task** Build a referential integrity validation layer in SQL that ran as part of the ETL pipeline and flagged all orphan records before they reached the Gold layer.

🟢 **Action**
    
    
    -- Comprehensive referential integrity checks as SQL assertions
    
    -- Check 1: Claims must reference a valid policy
    SELECT 'claims → policies' AS relationship,
           COUNT(*)             AS orphan_count,
           MIN(c.claim_id)      AS sample_orphan
    FROM claims c
    LEFT JOIN policies p ON c.policy_id = p.policy_id
    WHERE p.policy_id IS NULL
      AND c.claim_date >= CURRENT_DATE - 7;  -- recent data only
    
    -- Check 2: Policies must reference a valid product
    SELECT 'policies → products' AS relationship, COUNT(*), MIN(policy_id)
    FROM policies p
    LEFT JOIN products pr ON p.product_code = pr.product_code
    WHERE pr.product_code IS NULL;
    
    -- Check 3: Claims must reference a valid adjuster
    SELECT 'claims → adjusters' AS relationship, COUNT(*), MIN(claim_id)
    FROM claims c
    LEFT JOIN adjusters a ON c.adjuster_id = a.adjuster_id
    WHERE a.adjuster_id IS NULL;
    
    -- Consolidated integrity scorecard (runs all checks, summarises)
    WITH integrity_checks AS (
      SELECT 'claims→policies'   AS rel, COUNT(*) AS orphans
      FROM claims LEFT JOIN policies ON claims.policy_id = policies.policy_id
      WHERE policies.policy_id IS NULL
      UNION ALL
      SELECT 'policies→products', COUNT(*)
      FROM policies LEFT JOIN products ON policies.product_code = products.product_code
      WHERE products.product_code IS NULL
      UNION ALL
      SELECT 'claims→adjusters', COUNT(*)
      FROM claims LEFT JOIN adjusters ON claims.adjuster_id = adjusters.adjuster_id
      WHERE adjusters.adjuster_id IS NULL
    )
    SELECT
      rel,
      orphans,
      CASE WHEN orphans = 0 THEN '✓ PASS' ELSE '✗ FAIL' END AS status
    FROM integrity_checks
    ORDER BY orphans DESC;
    
    
    

🔴 **Result** The integrity scorecard ran after each ETL load and caught 3 recurring referential integrity violations that were being caused by out-of-order loads (claims arriving before their policy records). The load sequence was reordered to resolve the root cause, and orphan counts dropped to zero.

* * *

## Section 5: ETL SQL Patterns & Advanced Techniques (Q31–40)

* * *

### Q31. How do you implement an incremental load pattern in SQL that is both efficient and idempotent?

🔵 **Situation** At LSEG, the full-table reload of the 500M-row trades table was taking 3 hours nightly. Moving to an incremental load was necessary but the implementation had to be idempotent — a re-run on the same day should produce the same result without duplicates.

🟡 **Task** Design an incremental load SQL pattern that processed only changed records, was safe to re-run, and handled late-arriving data.

🟢 **Action**
    
    
    -- Step 1: Identify the high-water mark (last successfully loaded timestamp)
    DECLARE @hwm DATETIME;
    SELECT @hwm = MAX(last_load_ts) FROM etl_control WHERE table_name = 'trades';
    
    -- Step 2: Extract only records modified since the high-water mark
    -- with a 5-minute overlap buffer to catch late-arriving records
    INSERT INTO trades_staging (trade_id, currency_pair, amount, status, modified_ts)
    SELECT trade_id, currency_pair, amount, status, modified_ts
    FROM oracle_trades_link   -- linked server to Oracle
    WHERE modified_ts > DATEADD(MINUTE, -5, @hwm)  -- overlap buffer
      AND modified_ts <= GETUTCDATE();              -- don't load future records
    
    -- Step 3: MERGE staging into target (idempotent — re-run safe)
    MERGE trades AS tgt
    USING trades_staging AS src ON tgt.trade_id = src.trade_id
    WHEN MATCHED AND src.modified_ts > tgt.modified_ts THEN
      UPDATE SET tgt.currency_pair = src.currency_pair,
                 tgt.amount        = src.amount,
                 tgt.status        = src.status,
                 tgt.modified_ts   = src.modified_ts
    WHEN NOT MATCHED BY TARGET THEN
      INSERT (trade_id, currency_pair, amount, status, modified_ts)
      VALUES (src.trade_id, src.currency_pair, src.amount,
              src.status, src.modified_ts);
    
    -- Step 4: Update high-water mark ONLY on success
    UPDATE etl_control
    SET last_load_ts = (SELECT MAX(modified_ts) FROM trades_staging),
        last_run_ts  = GETUTCDATE(),
        rows_loaded  = @@ROWCOUNT
    WHERE table_name = 'trades';
    
    -- Step 5: Clean up staging
    TRUNCATE TABLE trades_staging;
    
    
    

Key idempotency mechanisms:

  * MERGE with `modified_ts > tgt.modified_ts` guard — a re-run won't overwrite with an older record.
  * High-water mark updated ONLY after successful MERGE — a failure leaves the HWM unchanged, so re-run picks up the same window.
  * 5-minute overlap buffer catches records whose `modified_ts` was backdated slightly by the source system.



🔴 **Result** Nightly load time dropped from 3 hours to **11 minutes**. Late-arriving records were caught by the overlap buffer with zero manual intervention required. The pattern was adopted as the standard for all 8 incremental feeds on the platform.

* * *

### Q32. How do you use `CROSS APPLY` and `OUTER APPLY` and when do they outperform standard JOINs?

🔵 **Situation** At Cisco (via Wipro), the TelePresence reporting query needed to attach the most recent quality score event to each conference — but a conference could have 0, 1, or many quality score events, and only the latest mattered. A standard LEFT JOIN caused row duplication for multi-scored conferences.

🟡 **Task** Rewrite the join using APPLY to return exactly one quality score per conference without post-deduplication.

🟢 **Action**
    
    
    -- PROBLEM: Standard LEFT JOIN duplicates conferences with multiple scores
    SELECT c.conference_id, c.start_ts, q.quality_score
    FROM conferences c
    LEFT JOIN quality_scores q ON q.conference_id = c.conference_id;
    -- ↑ Returns 3 rows for a conference with 3 quality score events
    
    -- SOLUTION: OUTER APPLY with TOP 1 (returns exactly 1 row per conference)
    SELECT
      c.conference_id,
      c.start_ts,
      c.duration_mins,
      c.participant_count,
      qs.quality_score,
      qs.scored_at
    FROM conferences c
    OUTER APPLY (             -- OUTER: keeps conferences with no quality score (like LEFT JOIN)
      SELECT TOP 1
        quality_score,
        scored_at
      FROM quality_scores
      WHERE conference_id = c.conference_id
      ORDER BY scored_at DESC    -- most recent score
    ) qs
    WHERE c.conference_date = CAST(GETDATE()-1 AS DATE);
    
    -- CROSS APPLY vs OUTER APPLY:
    -- CROSS APPLY: only returns conference rows where the subquery returns ≥1 row (like INNER JOIN)
    -- OUTER APPLY: returns all conference rows, NULLs where subquery returns 0 rows (like LEFT JOIN)
    
    -- When APPLY outperforms JOIN:
    -- 1. Top-N per group (avoids window function + filter wrapper)
    -- 2. Calling a table-valued function per row
    -- 3. Unpivoting a fixed set of columns (CROSS APPLY VALUES)
    -- 4. Correlated subquery that needs to return multiple columns
    
    -- CROSS APPLY VALUES (column unpivot) — used in reconciliation queries
    SELECT c.claim_id, v.field_name, v.field_value
    FROM claims c
    CROSS APPLY (
      VALUES
        ('claim_amount',  CAST(c.claim_amount  AS VARCHAR(50))),
        ('claim_status',  c.claim_status),
        ('policy_number', c.policy_number)
    ) AS v(field_name, field_value)
    WHERE v.field_value IS NOT NULL;
    
    
    

🔴 **Result** The OUTER APPLY rewrite eliminated conference row duplication, reducing the daily report row count from 1.8M to the correct 600K. The SSRS dashboard loaded 65% faster as a result. The CROSS APPLY VALUES pattern was used in the migration comparison query (Q23).

* * *

### Q33. How do you implement a Type 1 (overwrite) vs Type 2 (history) SCD in a single pipeline?

🔵 **Situation** At Travelers Insurance, the `dim_policyholder` dimension had mixed SCD requirements: some attributes (email address) were Type 1 (correct in place, no history needed) while others (risk tier, coverage level) were Type 2 (full history required for actuarial point-in-time queries).

🟡 **Task** Design a single MERGE-based pipeline that handled Type 1 and Type 2 attributes simultaneously without running two separate processes.

🟢 **Action**
    
    
    -- Combined Type 1 + Type 2 SCD in a two-step process
    
    -- Step 1: Handle Type 2 changes — expire current row if tracked attributes changed
    MERGE dim_policyholder AS tgt
    USING policyholder_staging AS src
    ON tgt.policyholder_id = src.policyholder_id AND tgt.is_current = 1
    WHEN MATCHED AND (
      tgt.risk_tier      <> src.risk_tier OR       -- Type 2 attribute
      tgt.coverage_level <> src.coverage_level     -- Type 2 attribute
    ) THEN
      UPDATE SET
        tgt.is_current   = 0,
        tgt.effective_to = CAST(GETDATE()-1 AS DATE);
    
    -- Insert new current row for Type 2 changes
    INSERT INTO dim_policyholder
      (policyholder_id, full_name, email,              -- includes Type 1
       risk_tier, coverage_level, effective_from,      -- includes Type 2
       effective_to, is_current, created_ts)
    SELECT
      src.policyholder_id, src.full_name, src.email,
      src.risk_tier, src.coverage_level, CAST(GETDATE() AS DATE),
      '9999-12-31', 1, GETUTCDATE()
    FROM policyholder_staging src
    JOIN dim_policyholder tgt
      ON  tgt.policyholder_id = src.policyholder_id
      AND tgt.is_current = 0
      AND tgt.effective_to = CAST(GETDATE()-1 AS DATE);  -- just expired
    
    -- Step 2: Apply Type 1 changes — update ALL versions for non-tracked attributes
    -- (email correction should be visible across all historical rows)
    UPDATE tgt
    SET tgt.email     = src.email,     -- Type 1: update all versions
        tgt.full_name = src.full_name  -- Type 1: correct across history
    FROM dim_policyholder tgt
    JOIN policyholder_staging src ON tgt.policyholder_id = src.policyholder_id
    WHERE tgt.email <> src.email      -- only if Type 1 attribute actually changed
       OR tgt.full_name <> src.full_name;
    
    
    

The crucial design: Type 1 UPDATE runs **after** Type 2 INSERT — ensuring the newly inserted current row also gets the Type 1 correction applied in the same batch.

🔴 **Result** The combined SCD pipeline eliminated the need for two separate ETL jobs. Actuarial point-in-time queries continued to work correctly against Type 2 history, while Type 1 corrections propagated immediately across all versions — satisfying both the compliance and actuarial teams' requirements simultaneously.

* * *

### Q34. How would you design a SQL-based data lineage tracking system?

🔵 **Situation** At J&J Global Services, the internal audit team required proof that every field in the executive dashboard could be traced back to its source system. There was no automated lineage tracking — it relied entirely on documentation that was frequently out of sync with the actual ETL code.

🟡 **Task** Design a lightweight SQL-based lineage tracking system that captured lineage automatically as part of the ETL pipeline execution.

🟢 **Action**
    
    
    -- Lineage metadata tables
    CREATE TABLE lineage_objects (
      object_id       INT IDENTITY PRIMARY KEY,
      object_name     VARCHAR(200),
      object_type     VARCHAR(50),   -- 'TABLE', 'VIEW', 'REPORT'
      layer           VARCHAR(20),   -- 'SOURCE', 'STAGING', 'SILVER', 'GOLD'
      system_name     VARCHAR(100),
      created_ts      DATETIME DEFAULT GETUTCDATE()
    );
    
    CREATE TABLE lineage_edges (
      edge_id         INT IDENTITY PRIMARY KEY,
      source_object_id   INT REFERENCES lineage_objects(object_id),
      target_object_id   INT REFERENCES lineage_objects(object_id),
      transformation_type VARCHAR(50),  -- 'COPY', 'AGGREGATE', 'DERIVE', 'JOIN'
      transformation_sql  NVARCHAR(MAX), -- the actual SQL expression
      etl_job_name       VARCHAR(200),
      recorded_ts        DATETIME DEFAULT GETUTCDATE()
    );
    
    -- ETL jobs log their lineage automatically at runtime:
    -- (Example: ETL job that loads claims_gold from claims_silver)
    INSERT INTO lineage_edges
      (source_object_id, target_object_id, transformation_type,
       transformation_sql, etl_job_name)
    VALUES (
      (SELECT object_id FROM lineage_objects WHERE object_name = 'claims_silver'),
      (SELECT object_id FROM lineage_objects WHERE object_name = 'claims_gold'),
      'AGGREGATE',
      'SUM(incurred_amount) GROUP BY product_line, region, claim_year',
      'etl_claims_gold_daily'
    );
    
    -- Lineage query: trace any Gold field back to its source
    WITH lineage_path AS (
      SELECT
        source_object_id,
        target_object_id,
        transformation_type,
        transformation_sql,
        etl_job_name,
        1 AS depth,
        CAST(etl_job_name AS VARCHAR(2000)) AS full_path
      FROM lineage_edges
      WHERE target_object_id = (
        SELECT object_id FROM lineage_objects WHERE object_name = 'claims_gold'
      )
    
      UNION ALL
    
      SELECT
        e.source_object_id,
        e.target_object_id,
        e.transformation_type,
        e.transformation_sql,
        e.etl_job_name,
        lp.depth + 1,
        CONCAT(e.etl_job_name, ' → ', lp.full_path)
      FROM lineage_edges e
      JOIN lineage_path lp ON e.target_object_id = lp.source_object_id
    )
    SELECT DISTINCT
      s.object_name  AS source_object,
      s.system_name  AS source_system,
      lp.full_path   AS lineage_chain,
      lp.depth       AS hops_from_source
    FROM lineage_path lp
    JOIN lineage_objects s ON s.object_id = lp.source_object_id
    ORDER BY lp.depth;
    
    
    

🔴 **Result** The lineage system provided the audit team with an automated, always-current lineage map for all 6 ETL pipelines. It replaced 200+ pages of manually maintained STTM documentation as the live reference. The internal audit completed without a single lineage-related finding.

* * *

### Q35. How do you implement a slowly changing dimension Type 6 (combined Type 1+2+3)?

🔵 **Situation** At Travelers Insurance, the risk management team needed the `dim_policyholder` dimension to support three simultaneous query patterns: current value lookup (Type 1), full historical traversal (Type 2), and previous-to-current comparison (Type 3) — all in a single dimension table.

🟡 **Task** Implement SCD Type 6 (also called "hybrid SCD") that simultaneously supported all three access patterns.

🟢 **Action**
    
    
    -- SCD Type 6 table structure: combines Type 1 + Type 2 + Type 3
    -- Type 1: current_* columns always show the current value (updated across all rows)
    -- Type 2: effective_from / effective_to track full history
    -- Type 3: previous_* columns track the immediate prior value
    
    ALTER TABLE dim_policyholder ADD
      -- Type 2 columns (full history)
      effective_from    DATE    NOT NULL DEFAULT GETDATE(),
      effective_to      DATE    NOT NULL DEFAULT '9999-12-31',
      is_current        BIT     NOT NULL DEFAULT 1,
      -- Type 3 column (one prior value)
      previous_risk_tier    VARCHAR(20) NULL,
      previous_coverage     VARCHAR(50) NULL,
      -- Type 1 columns (always current, even on historical rows)
      current_risk_tier     VARCHAR(20) NULL,
      current_coverage      VARCHAR(50) NULL;
    
    -- Load procedure: implement all three types in one pass
    UPDATE tgt
    SET
      -- Type 2: expire the old row
      tgt.effective_to   = CAST(GETDATE()-1 AS DATE),
      tgt.is_current     = 0,
      -- Type 3: capture prior value before expiry
      tgt.previous_risk_tier = tgt.risk_tier,
      tgt.previous_coverage  = tgt.coverage_level
    FROM dim_policyholder tgt
    JOIN policyholder_staging src ON tgt.policyholder_id = src.policyholder_id
    WHERE tgt.is_current = 1
    AND (tgt.risk_tier <> src.risk_tier OR tgt.coverage_level <> src.coverage_level);
    
    -- Insert new Type 2 row
    INSERT INTO dim_policyholder ...  -- (as per Q26 SCD2 pattern)
    
    -- Type 1: update current_* columns across ALL rows for this policyholder
    UPDATE dim_policyholder
    SET current_risk_tier = src.risk_tier,
        current_coverage  = src.coverage_level
    FROM dim_policyholder tgt
    JOIN policyholder_staging src ON tgt.policyholder_id = src.policyholder_id;
    
    
    

Query examples for each access pattern:
    
    
    -- Type 1 access (current value, single row per policyholder):
    SELECT * FROM dim_policyholder WHERE is_current = 1;
    
    -- Type 2 access (point-in-time):
    SELECT * FROM dim_policyholder
    WHERE policyholder_id = 12345
    AND claim_date BETWEEN effective_from AND effective_to;
    
    -- Type 3 access (what changed most recently?):
    SELECT policyholder_id, previous_risk_tier, risk_tier AS new_risk_tier
    FROM dim_policyholder
    WHERE is_current = 1
    AND previous_risk_tier <> risk_tier;
    
    
    

🔴 **Result** The Type 6 SCD implementation satisfied three separate analytical use cases from a single dimension table — eliminating the need for separate "current" and "history" views that had previously caused confusion and query errors.

* * *

## Section 6: Recursive SQL, Graph Traversal & Advanced Patterns (Q36–43)

* * *

### Q36. How do you detect circular references in a hierarchical table?

🔵 **Situation** At J&J Global Services, the recursive CTE traversing the legal entity hierarchy occasionally ran indefinitely — consuming all CPU — due to circular parent-child references (Entity A → Entity B → Entity A) caused by data entry errors.

🟡 **Task** Write a SQL query to detect all circular references in the hierarchy before the recursive CTE ran, and alert on them.

🟢 **Action**
    
    
    -- Detect circular references using path tracking
    WITH RECURSIVE hierarchy_check AS (
      -- Anchor: all root nodes (no parent)
      SELECT
        entity_id,
        parent_entity_id,
        CAST(entity_id AS VARCHAR(4000)) AS path,  -- track visited nodes
        0                                AS depth,
        0                                AS is_cycle
      FROM legal_entities
      WHERE parent_entity_id IS NULL
    
      UNION ALL
    
      SELECT
        e.entity_id,
        e.parent_entity_id,
        CONCAT(h.path, ',', e.entity_id),
        h.depth + 1,
        -- Cycle detected if current entity_id already appears in the path
        CASE WHEN CHARINDEX(CAST(e.entity_id AS VARCHAR), h.path) > 0
             THEN 1 ELSE 0 END
      FROM legal_entities e
      JOIN hierarchy_check h ON e.parent_entity_id = h.entity_id
      WHERE h.is_cycle = 0      -- stop recursion once cycle detected
        AND h.depth < 50        -- safety depth limit
    )
    SELECT
      entity_id,
      parent_entity_id,
      path             AS cycle_path,
      depth
    FROM hierarchy_check
    WHERE is_cycle = 1
    ORDER BY depth;
    
    -- Simpler: find direct reciprocal relationships (A is parent of B, B is parent of A)
    SELECT
      a.entity_id   AS entity_a,
      a.parent_entity_id AS claims_parent_is_b,
      b.entity_id   AS entity_b,
      b.parent_entity_id AS but_b_claims_parent_is_a
    FROM legal_entities a
    JOIN legal_entities b
      ON a.entity_id = b.parent_entity_id
      AND b.entity_id = a.parent_entity_id;
    
    
    

🔴 **Result** The cycle detection script found 2 circular references caused by data entry errors. Both were corrected in the source system. The check was added as a pre-condition gate in the ETL pipeline — the recursive CTE only runs if the cycle check returns zero rows, preventing any future infinite loop scenario.

* * *

### Q37. How would you write a query to calculate the nth highest salary or nth largest value without using `LIMIT`/`TOP`?

🔵 **Situation** At Cisco (via Wipro), a database compatibility constraint meant the reporting query had to run on SQL Server 2008 (no `FETCH OFFSET`) and on Oracle simultaneously — requiring a solution that worked on both platforms without platform-specific syntax.

🟡 **Task** Write a cross-platform query to return the 3rd highest value from a numeric column using only ANSI SQL.

🟢 **Action**
    
    
    -- Method 1: Correlated subquery (ANSI SQL — works on any database)
    -- "Find rows where exactly N-1 rows have a strictly higher value"
    SELECT DISTINCT salary
    FROM employees e1
    WHERE 2 = (           -- N-1 = 2 for 3rd highest (0 rows are higher for the max)
      SELECT COUNT(DISTINCT e2.salary)
      FROM employees e2
      WHERE e2.salary > e1.salary
    );
    
    -- Method 2: DENSE_RANK window function (SQL Server 2005+, Oracle, BigQuery)
    SELECT salary
    FROM (
      SELECT
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
      FROM employees
    ) ranked
    WHERE salary_rank = 3;   -- change to any N
    
    -- Method 3: Double NOT EXISTS (pure ANSI, academic but useful to know)
    -- "Find a value for which no N distinct higher values exist"
    SELECT DISTINCT salary
    FROM employees e1
    WHERE (
      SELECT COUNT(DISTINCT salary)
      FROM employees e2
      WHERE e2.salary > e1.salary
    ) = 2;   -- exactly 2 values are higher → this is the 3rd highest
    
    -- Extension: Top N per department
    SELECT department_id, employee_id, salary
    FROM (
      SELECT
        department_id,
        employee_id,
        salary,
        DENSE_RANK() OVER (
          PARTITION BY department_id
          ORDER BY salary DESC
        ) AS dept_salary_rank
      FROM employees
    ) ranked
    WHERE dept_salary_rank <= 3;
    
    
    

🔴 **Result** The `DENSE_RANK()` approach was chosen for the production query — it handled ties correctly and ran on both SQL Server and Oracle without modification. The cross-platform compatibility was critical as the same query fed reports on 3 different database platforms across the Cisco estate.

* * *

### Q38. How do you write SQL to transpose or unpivot a wide table into a long format?

🔵 **Situation** At J&J Global Services, the actuarial source system delivered monthly claims data as a wide table with one column per month (`jan_2023`, `feb_2023`, ... `dec_2023`). The EDW required a long format (one row per month) for all time-series analysis and window function calculations.

🟡 **Task** Transform the wide 12-column monthly format into a normalised long format without hard-coding column names in a way that would break when a new year's columns were added.

🟢 **Action**
    
    
    -- Method 1: CROSS APPLY VALUES (SQL Server) — explicit but flexible
    SELECT
      policy_id,
      product_line,
      v.month_label,
      v.incurred_amount
    FROM wide_claims
    CROSS APPLY (
      VALUES
        ('2023-01', jan_2023),
        ('2023-02', feb_2023),
        ('2023-03', mar_2023),
        ('2023-04', apr_2023),
        ('2023-05', may_2023),
        ('2023-06', jun_2023),
        ('2023-07', jul_2023),
        ('2023-08', aug_2023),
        ('2023-09', sep_2023),
        ('2023-10', oct_2023),
        ('2023-11', nov_2023),
        ('2023-12', dec_2023)
    ) AS v(month_label, incurred_amount)
    WHERE v.incurred_amount IS NOT NULL   -- exclude nulls (sparse data)
    ORDER BY policy_id, month_label;
    
    -- Method 2: UNPIVOT operator (T-SQL)
    SELECT policy_id, product_line, month_label, incurred_amount
    FROM wide_claims
    UNPIVOT (
      incurred_amount FOR month_label IN (
        jan_2023, feb_2023, mar_2023, apr_2023,
        may_2023, jun_2023, jul_2023, aug_2023,
        sep_2023, oct_2023, nov_2023, dec_2023
      )
    ) AS unpvt;
    
    -- Method 3: Dynamic UNPIVOT — auto-detects month columns from metadata
    DECLARE @month_cols NVARCHAR(MAX);
    SELECT @month_cols = STRING_AGG(QUOTENAME(column_name), ', ')
    FROM information_schema.columns
    WHERE table_name = 'wide_claims'
    AND column_name LIKE '[a-z][a-z][a-z]_[0-9][0-9][0-9][0-9]';  -- pattern match
    
    DECLARE @sql NVARCHAR(MAX) = '
    SELECT policy_id, product_line, month_label, incurred_amount
    FROM wide_claims
    UNPIVOT (incurred_amount FOR month_label IN (' + @month_cols + ')) u;';
    
    EXEC sp_executesql @sql;
    
    
    

🔴 **Result** The long-format transformation enabled the first-ever time-series analysis on the claims data — including seasonality detection (Q3 consistently 30% above average) and the rolling YoY window function analysis (Q1 in this document). The dynamic UNPIVOT automatically included new year columns when added by the source system.

* * *

### Q39. How do you write a SQL query to find all paths between two nodes in a graph stored in a relational table?

🔵 **Situation** At J&J Global Services, the compliance team needed to trace all data flows between the source claims systems and the executive dashboard — stored as a graph in the lineage_edges table — to demonstrate that no uncontrolled transformation paths existed.

🟡 **Task** Write a recursive SQL query to enumerate all possible paths between a source system node and the executive dashboard node in the lineage graph.

🟢 **Action**
    
    
    WITH RECURSIVE all_paths AS (
      -- Anchor: start from the source node
      SELECT
        source_object_id,
        target_object_id,
        etl_job_name,
        CAST(source_object_id AS VARCHAR(2000))    AS path_ids,
        CAST(etl_job_name     AS VARCHAR(4000))    AS path_jobs,
        1                                           AS hops,
        0                                           AS is_cycle
      FROM lineage_edges
      WHERE source_object_id = (
        SELECT object_id FROM lineage_objects WHERE object_name = 'oracle_claims_source'
      )
    
      UNION ALL
    
      -- Recursive: extend the path one hop at a time
      SELECT
        e.source_object_id,
        e.target_object_id,
        e.etl_job_name,
        CONCAT(ap.path_ids, ' → ', e.target_object_id),
        CONCAT(ap.path_jobs, ' → ', e.etl_job_name),
        ap.hops + 1,
        -- Cycle detection: check if target already in path
        CASE WHEN CHARINDEX(CAST(e.target_object_id AS VARCHAR), ap.path_ids) > 0
             THEN 1 ELSE 0 END
      FROM lineage_edges e
      JOIN all_paths ap ON e.source_object_id = ap.target_object_id
      WHERE ap.is_cycle = 0
        AND ap.hops < 15   -- max path length guard
    )
    SELECT
      path_ids  AS node_path,
      path_jobs AS transformation_path,
      hops
    FROM all_paths
    WHERE target_object_id = (
      SELECT object_id FROM lineage_objects WHERE object_name = 'executive_dashboard'
    )
    ORDER BY hops;
    
    
    

🔴 **Result** The path traversal query revealed 3 distinct data flow paths to the executive dashboard — the compliance team was aware of 2 but had not known about the third (an undocumented legacy feed). The undocumented path was reviewed, validated, and formally documented. The audit was completed with clean findings.

* * *

### Q40. How do you write a SQL query to perform fuzzy matching between two name columns to find near-duplicate records?

🔵 **Situation** At J&J Global Services, when merging 4 legacy claims systems, the same claimant appeared with slightly different name spellings across systems (`"John Smith"`, `"Jon Smith"`, `"J. Smith"`). Standard exact-match joins missed these duplicates, resulting in over-counting in the actuarial model.

🟡 **Task** Write a SQL-based fuzzy matching approach to identify potential name duplicates for human review before deduplication.

🟢 **Action**
    
    
    -- Method 1: SOUNDEX — phonetic matching (fast, built-in to T-SQL)
    -- Catches homophones and common misspellings
    SELECT
      a.claimant_id  AS id_a,
      a.full_name    AS name_a,
      b.claimant_id  AS id_b,
      b.full_name    AS name_b,
      'SOUNDEX_MATCH' AS match_type
    FROM claimants a
    JOIN claimants b
      ON  SOUNDEX(a.last_name)  = SOUNDEX(b.last_name)
      AND SOUNDEX(a.first_name) = SOUNDEX(b.first_name)
      AND a.claimant_id < b.claimant_id      -- avoid self-matches and duplicates
      AND a.claimant_id <> b.claimant_id
      AND a.date_of_birth = b.date_of_birth  -- anchor on exact DOB to reduce false positives
    WHERE a.claimant_id < b.claimant_id;
    
    -- Method 2: DIFFERENCE function (T-SQL) — 0-4 score, 4 = highest similarity
    SELECT
      a.claimant_id, a.full_name,
      b.claimant_id, b.full_name,
      DIFFERENCE(a.last_name,  b.last_name)  AS last_name_diff,
      DIFFERENCE(a.first_name, b.first_name) AS first_name_diff
    FROM claimants a
    JOIN claimants b ON a.claimant_id < b.claimant_id
    WHERE DIFFERENCE(a.last_name,  b.last_name)  >= 3   -- high phonetic similarity
      AND DIFFERENCE(a.first_name, b.first_name) >= 3
      AND a.date_of_birth = b.date_of_birth;
    
    -- Method 3: Levenshtein distance via tally table (T-SQL UDF approach)
    -- More accurate but computationally expensive — run on pre-filtered candidate pairs
    -- In BigQuery: use ML.DISTANCE or EDIT_DISTANCE built-in functions
    SELECT
      a.claimant_id, a.full_name,
      b.claimant_id, b.full_name,
      dbo.fn_levenshtein(a.full_name, b.full_name) AS edit_distance
    FROM claimants a
    JOIN claimants b
      ON a.claimant_id < b.claimant_id
      AND a.date_of_birth = b.date_of_birth   -- pre-filter with exact match first
    WHERE dbo.fn_levenshtein(a.full_name, b.full_name) <= 3;  -- 3 character edits
    
    
    

🔴 **Result** The fuzzy matching identified 1,847 potential duplicate claimants — 6.2% of the claimant base. After manual review, 1,203 were confirmed duplicates and merged. Correcting these removed 8,400 duplicate claim records from the actuarial model, materially improving reserve accuracy.

* * *

## Section 7: EDW-Specific SQL Patterns (Q41–50)

* * *

### Q41. How do you implement a slowly changing fact (measuring cumulative totals that can decrease)?

🔵 **Situation** At Travelers Insurance, the claims reserve table stored running reserve balances that could increase (new reserve set) or decrease (reserve released after settlement) — a semi-additive fact that couldn't simply be summed across time like regular claims.

🟡 **Task** Design the fact table and query pattern to correctly report reserve balances at any point in time without double-counting increases and decreases.

🟢 **Action**
    
    
    -- Fact table design: store reserve MOVEMENTS (deltas), not snapshots
    -- This avoids point-in-time balance calculation complexity
    
    -- Fact table: fact_reserve_movement
    -- claim_id | movement_date | movement_type | movement_amount
    -- (positive = reserve increase, negative = reserve release/decrease)
    
    -- Query: balance at any point in time (cumulative sum of movements)
    SELECT
      claim_id,
      movement_date,
      movement_amount,
      SUM(movement_amount) OVER (
        PARTITION BY claim_id
        ORDER BY movement_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
      )                                         AS running_reserve_balance
    FROM fact_reserve_movement
    WHERE claim_id = 98765
    ORDER BY movement_date;
    
    -- Query: total reserve balance across ALL claims as of a specific date
    -- (semi-additive: SUM across claims but NOT across time)
    SELECT
      claim_id,
      SUM(movement_amount) AS reserve_balance_at_eom
    FROM fact_reserve_movement
    WHERE movement_date <= '2024-03-31'   -- as-of date
    GROUP BY claim_id
    HAVING SUM(movement_amount) > 0;      -- exclude fully settled claims
    
    -- Reserve balance trend by month-end:
    WITH month_ends AS (
      SELECT DISTINCT
        LAST_DAY(movement_date) AS month_end_date
      FROM fact_reserve_movement
    ),
    balances AS (
      SELECT
        me.month_end_date,
        SUM(CASE WHEN frm.movement_date <= me.month_end_date
                 THEN frm.movement_amount END) AS total_reserve
      FROM month_ends me
      CROSS JOIN fact_reserve_movement frm
      GROUP BY me.month_end_date
    )
    SELECT * FROM balances ORDER BY month_end_date;
    
    
    

🔴 **Result** The movement-based design correctly handled reserve releases — a pattern the previous snapshot-based approach had been getting wrong by summing across the time dimension. Actuarial reserve reports became reconcilable with the source system for the first time, removing a long-standing audit caveat.

* * *

### Q42. How do you implement a bridge table for many-to-many relationships in a data warehouse?

🔵 **Situation** At J&J Global Services, the underwriting model had a many-to-many relationship between policies and agents — one policy could have multiple agents, and one agent could manage multiple policies. The standard star schema couldn't represent this without losing data.

🟡 **Task** Design and implement a bridge table pattern to correctly model the many-to-many relationship and produce accurate commission allocation reports.

🟢 **Action**
    
    
    -- Bridge table: sits between fact_policy and dim_agent
    CREATE TABLE bridge_policy_agent (
      policy_key          INT NOT NULL,    -- FK to dim_policy
      agent_key           INT NOT NULL,    -- FK to dim_agent
      allocation_weight   DECIMAL(5,4),    -- proportion of commission (sums to 1.0 per policy)
      role_type           VARCHAR(50),     -- 'PRIMARY', 'SECONDARY', 'REFERRAL'
      effective_from      DATE,
      effective_to        DATE,
      PRIMARY KEY (policy_key, agent_key, effective_from)
    );
    
    -- Query: policy fact with correct commission allocation via bridge
    SELECT
      a.agent_name,
      a.region,
      SUM(p.earned_premium * b.allocation_weight) AS allocated_premium,
      SUM(p.commission_amount * b.allocation_weight) AS allocated_commission,
      COUNT(DISTINCT p.policy_key)                AS policy_count
    FROM fact_policy p
    JOIN bridge_policy_agent b
      ON  p.policy_key    = b.policy_key
      AND p.policy_date BETWEEN b.effective_from AND b.effective_to
    JOIN dim_agent a ON b.agent_key = a.agent_key
    WHERE p.policy_year = 2024
      AND a.region = 'EMEA'
    GROUP BY a.agent_name, a.region
    ORDER BY allocated_premium DESC;
    
    -- Validate: allocation_weights must sum to exactly 1.0 per policy
    SELECT policy_key,
           SUM(allocation_weight) AS total_weight,
           CASE WHEN ABS(SUM(allocation_weight) - 1.0) > 0.0001
                THEN 'WEIGHT ERROR' ELSE 'OK' END AS validation
    FROM bridge_policy_agent
    WHERE effective_to = '9999-12-31'   -- current allocations
    GROUP BY policy_key
    HAVING ABS(SUM(allocation_weight) - 1.0) > 0.0001;
    
    
    

🔴 **Result** The bridge table pattern resolved a 6-month-old commission reporting discrepancy where multi-agent policies were double-counting commission. The corrected report was used to recalculate commission payments for 2 prior quarters — identifying $180K in overpayments that were adjusted.

* * *

### Q43. How do you handle late-arriving data in a data warehouse pipeline?

🔵 **Situation** At Travelers Insurance, claims data arrived late approximately 3% of the time — a claim event with a `claim_date` of Monday would sometimes only arrive in Wednesday's feed, causing Monday's Gold layer aggregates to be understated.

🟡 **Task** Design a SQL pipeline pattern that correctly handled late-arriving facts without requiring a full table reload of historical partitions.

🟢 **Action**
    
    
    -- Step 1: Identify late-arriving records in the staging table
    SELECT
      claim_id,
      claim_date,             -- the business date this claim belongs to
      load_date,              -- the date we actually received it
      DATEDIFF(DAY, claim_date, load_date)  AS arrival_lag_days
    FROM claims_staging
    WHERE load_date = CURRENT_DATE
      AND claim_date < CURRENT_DATE   -- late-arriving: event date < load date
    ORDER BY arrival_lag_days DESC;
    
    -- Step 2: Apply late-arrivals to the correct historical partition
    -- (rather than the current day's partition)
    MERGE fact_claims AS tgt
    USING (
      SELECT *
      FROM claims_staging
      WHERE load_date = CURRENT_DATE   -- today's batch
    ) AS src
    ON tgt.claim_id = src.claim_id
    -- Late arrival: insert into the correct historical date partition
    WHEN NOT MATCHED BY TARGET THEN
      INSERT (claim_id, claim_date, load_date, claim_amount,
              claim_status, is_late_arrival, original_load_date)
      VALUES (src.claim_id, src.claim_date, src.load_date,
              src.claim_amount, src.claim_status,
              CASE WHEN src.claim_date < src.load_date THEN 1 ELSE 0 END,
              src.load_date);
    
    -- Step 3: Recompute affected Gold layer aggregates for historical dates
    -- Only re-aggregate the specific historical dates that received late arrivals
    WITH affected_dates AS (
      SELECT DISTINCT claim_date
      FROM claims_staging
      WHERE load_date = CURRENT_DATE
        AND claim_date < CURRENT_DATE
    )
    MERGE gold_daily_claims_summary AS tgt
    USING (
      SELECT
        f.claim_date,
        COUNT(*)          AS total_claims,
        SUM(claim_amount) AS total_incurred
      FROM fact_claims f
      WHERE f.claim_date IN (SELECT claim_date FROM affected_dates)
      GROUP BY f.claim_date
    ) AS src
    ON tgt.summary_date = src.claim_date
    WHEN MATCHED THEN
      UPDATE SET
        tgt.total_claims   = src.total_claims,
        tgt.total_incurred = src.total_incurred,
        tgt.last_updated   = GETUTCDATE();
    
    
    

🔴 **Result** Late-arrival handling ensured that historical Gold layer aggregates were automatically corrected when late data arrived — without reprocessing entire partitions. The fraud team confirmed that previously "missing" Monday claims were now correctly appearing in Monday's totals, improving the weekly fraud pattern analysis accuracy.

* * *

### Q44. How do you write a query to identify data skew in a distributed data warehouse?

🔵 **Situation** At J&J Global Services on BigQuery, certain queries against the 200M-row claims table were running disproportionately slowly despite having correct indexes. Investigation suggested data skew — one or more partition/distribution keys had vastly more rows than others, causing hot-spot processing.

🟡 **Task** Write SQL diagnostic queries to identify skew in the data distribution and propose a resolution.

🟢 **Action**
    
    
    -- Diagnose partition skew: row count distribution across partition values
    SELECT
      claim_year,
      claim_month,
      COUNT(*)                                        AS row_count,
      ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS pct_of_total,
      -- Flag if partition holds > 5x the average partition size
      CASE WHEN COUNT(*) > 5 * AVG(COUNT(*)) OVER ()
           THEN 'SKEWED' ELSE 'OK' END                AS skew_flag
    FROM claims
    GROUP BY claim_year, claim_month
    ORDER BY row_count DESC;
    
    -- Diagnose JOIN skew: identify high-frequency join keys
    -- (keys with extreme row counts cause hotspots in hash joins)
    SELECT
      product_line,
      COUNT(*)                              AS row_count,
      ROUND(COUNT(*) * 100.0 /
        (SELECT COUNT(*) FROM claims), 4)   AS pct_of_table
    FROM claims
    GROUP BY product_line
    ORDER BY row_count DESC;
    -- If top 1 product_line = 60% of all rows → severe JOIN skew risk
    
    -- BigQuery-specific: check for shuffle skew in INFORMATION_SCHEMA
    SELECT
      job_id,
      total_bytes_processed,
      total_slot_ms,
      -- High slot_ms with low bytes = skewed execution
      ROUND(total_slot_ms / NULLIF(total_bytes_processed, 0) * 1e9, 2) AS slot_ms_per_gb
    FROM `region-europe-west1`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
    WHERE statement_type = 'SELECT'
      AND creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
    ORDER BY slot_ms_per_gb DESC
    LIMIT 20;
    
    
    

Skew resolution approaches:

  1. **Re-partition** on a higher-cardinality column (`claim_id % 100` instead of `product_line`).
  2. **Salting** : add a random suffix to skewed join keys to distribute load, then aggregate post-join.
  3. **Pre-aggregation** : summarise the skewed side before the join.
  4. **Broadcast join hint** : for small dimension tables joining large fact tables, broadcast the small table to all nodes.



🔴 **Result** The skew diagnosis revealed one product line representing 62% of all claim rows. A salted join approach reduced the worst-case query runtime from 18 minutes to 4 minutes. The insight was used to redesign the BigQuery table clustering key from `product_line` to `(claim_year, claim_month)` — improving all queries by an average of 40%.

* * *

### Q45–50: Scenario-Based SQL Challenges

* * *

### Q45. You are asked to find all customers who made a purchase in every month of the year. How do you write this in SQL?

🔵 **Situation** At J&J Global Services, the commercial team needed to identify "consistently active" accounts — those with at least one transaction in every single month of 2023 — to target with a loyalty programme.

🟡 **Task** Write a query that correctly identified accounts active in all 12 months, handling sparse activity months correctly.

🟢 **Action**
    
    
    -- Approach: count distinct active months per account, filter for 12
    SELECT
      account_id,
      COUNT(DISTINCT DATE_TRUNC('MONTH', transaction_date)) AS active_months
    FROM transactions
    WHERE EXTRACT(YEAR FROM transaction_date) = 2023
    GROUP BY account_id
    HAVING COUNT(DISTINCT DATE_TRUNC('MONTH', transaction_date)) = 12
    ORDER BY account_id;
    
    -- More robust: cross-join with all 12 months, then find accounts present in all
    WITH months AS (
      SELECT generate_series(1, 12) AS month_num   -- or a date spine
    ),
    account_months AS (
      SELECT
        account_id,
        EXTRACT(MONTH FROM transaction_date) AS txn_month
      FROM transactions
      WHERE EXTRACT(YEAR FROM transaction_date) = 2023
      GROUP BY account_id, EXTRACT(MONTH FROM transaction_date)
    )
    SELECT account_id
    FROM account_months
    GROUP BY account_id
    HAVING COUNT(DISTINCT txn_month) = (SELECT COUNT(*) FROM months);
    
    
    

🔴 **Result** Query identified 847 "consistently active" accounts from 12,000 total — a targeted cohort that received the loyalty programme outreach. Conversion rate for this cohort was 3.2x higher than the broad-based campaign.

* * *

### Q46. How would you write a query to calculate customer lifetime value (LTV) using SQL?

🔵 **Situation** At Travelers Insurance, the product team needed customer LTV to rank policyholders for a retention campaign — prioritising high-value customers at risk of lapsing.

🟡 **Task** Compute LTV from the claims and policy transaction tables.

🟢 **Action**
    
    
    WITH customer_history AS (
      SELECT
        p.policyholder_id,
        MIN(p.policy_start_date)              AS first_policy_date,
        MAX(p.policy_end_date)                AS last_policy_date,
        SUM(p.premium_amount)                 AS total_premiums_paid,
        COALESCE(SUM(c.claim_amount), 0)      AS total_claims_incurred,
        COUNT(DISTINCT p.policy_id)           AS policy_count,
        DATEDIFF(MONTH,
          MIN(p.policy_start_date),
          COALESCE(MAX(p.policy_end_date), CURRENT_DATE)
        )                                     AS tenure_months
      FROM policies p
      LEFT JOIN claims c ON p.policy_id = c.policy_id
      GROUP BY p.policyholder_id
    )
    SELECT
      policyholder_id,
      tenure_months,
      total_premiums_paid,
      total_claims_incurred,
      total_premiums_paid - total_claims_incurred               AS net_value,
      ROUND((total_premiums_paid - total_claims_incurred) /
        NULLIF(tenure_months, 0) * 12, 2)                      AS annualised_ltv,
      NTILE(10) OVER (ORDER BY
        (total_premiums_paid - total_claims_incurred) DESC
      )                                                         AS ltv_decile
    FROM customer_history
    ORDER BY annualised_ltv DESC;
    
    
    

🔴 **Result** LTV decile analysis identified the top 10% of customers generating 68% of net underwriting margin. The retention campaign was focused on deciles 1–3 at risk of lapse — reducing churn in that segment by 22% versus the untargeted baseline.

* * *

### Q47. How do you write a SQL query to calculate month-over-month retention rate?

🔵 **Situation** At Google (Street View project), the programme manager needed to track analyst retention in the QA pipeline — what percentage of analysts active in month N were still active in month N+1.

🟡 **Task** Write a retention cohort query without using any BI tool computation.

🟢 **Action**
    
    
    WITH monthly_active AS (
      SELECT
        analyst_id,
        DATE_TRUNC(event_ts, MONTH)   AS active_month
      FROM review_events
      GROUP BY analyst_id, DATE_TRUNC(event_ts, MONTH)
    ),
    retention AS (
      SELECT
        curr.active_month             AS cohort_month,
        COUNT(DISTINCT curr.analyst_id) AS current_active,
        COUNT(DISTINCT nxt.analyst_id)  AS retained_next_month
      FROM monthly_active curr
      LEFT JOIN monthly_active nxt
        ON  curr.analyst_id   = nxt.analyst_id
        AND nxt.active_month  = DATE_ADD(curr.active_month, INTERVAL 1 MONTH)
      GROUP BY curr.active_month
    )
    SELECT
      cohort_month,
      current_active,
      retained_next_month,
      ROUND(retained_next_month * 100.0 /
        NULLIF(current_active, 0), 1)  AS retention_rate_pct
    FROM retention
    ORDER BY cohort_month;
    
    
    

🔴 **Result** Retention analysis revealed a consistent dip in analyst activity in weeks following long public holiday periods — not previously visible in aggregate productivity metrics. Scheduling adjustments improved the dip from 34% drop to 18% drop.

* * *

### Q48. How do you write a query to detect time-series anomalies using SQL alone (no ML tools)?

🔵 **Situation** At LSEG, the operations team needed an automated alert when daily trade volumes deviated significantly from the expected range — without relying on an external anomaly detection system.

🟡 **Task** Implement a statistical anomaly detection approach purely in SQL using rolling statistics.

🟢 **Action**
    
    
    WITH daily_volumes AS (
      SELECT
        trade_date,
        currency_pair,
        COUNT(*)            AS daily_count,
        SUM(notional_amount) AS daily_notional
      FROM trades
      WHERE trade_date >= CURRENT_DATE - 90
      GROUP BY trade_date, currency_pair
    ),
    rolling_stats AS (
      SELECT
        trade_date,
        currency_pair,
        daily_count,
        AVG(daily_count) OVER (
          PARTITION BY currency_pair
          ORDER BY trade_date
          ROWS BETWEEN 29 PRECEDING AND 1 PRECEDING  -- 30-day lookback, exclude today
        )                     AS rolling_avg,
        STDDEV(daily_count) OVER (
          PARTITION BY currency_pair
          ORDER BY trade_date
          ROWS BETWEEN 29 PRECEDING AND 1 PRECEDING
        )                     AS rolling_stddev
      FROM daily_volumes
    )
    SELECT
      trade_date,
      currency_pair,
      daily_count,
      ROUND(rolling_avg, 0)   AS expected_avg,
      ROUND(rolling_stddev, 0) AS expected_stddev,
      ROUND((daily_count - rolling_avg) /
        NULLIF(rolling_stddev, 0), 2) AS z_score,
      CASE
        WHEN ABS((daily_count - rolling_avg) /
             NULLIF(rolling_stddev, 0)) > 3   THEN 'ANOMALY - CRITICAL'
        WHEN ABS((daily_count - rolling_avg) /
             NULLIF(rolling_stddev, 0)) > 2   THEN 'ANOMALY - WARNING'
        ELSE                                       'NORMAL'
      END                     AS anomaly_flag
    FROM rolling_stats
    WHERE trade_date = CURRENT_DATE - 1  -- yesterday's results
      AND rolling_stddev > 0             -- need enough history
    ORDER BY ABS((daily_count - rolling_avg) / NULLIF(rolling_stddev, 0)) DESC;
    
    
    

🔴 **Result** The anomaly query caught a volume spike 3 days before the end-of-quarter that turned out to be caused by a duplicate feed from a new data provider — not genuine trade volume. Catching it early prevented an incorrect risk report from being submitted to the regulator.

* * *

### Q49. How do you optimise a GROUP BY query that runs slowly on a very large table?

🔵 **Situation** At LSEG, a daily summary query grouping 500M trade records by currency pair, status, and date was running for 45 minutes — far exceeding the 5-minute window before the morning report run.

🟡 **Task** Optimise the GROUP BY query to run within the time constraint.

🟢 **Action** Four-stage optimisation:
    
    
    -- STAGE 1: Filter early (most impactful)
    -- Move predicates as close to the data source as possible
    SELECT currency_pair, status, trade_date, SUM(notional) AS total
    FROM trades
    WHERE trade_date >= DATEADD(DAY, -90, GETDATE())  -- applied at scan time
    GROUP BY currency_pair, status, trade_date;
    
    -- STAGE 2: Pre-aggregate in a CTE before joining to dimensions
    -- Reduces rows before the dimension join
    WITH agg AS (
      SELECT currency_pair, status, trade_date, SUM(notional) AS total
      FROM trades
      WHERE trade_date >= DATEADD(DAY, -90, GETDATE())
      GROUP BY currency_pair, status, trade_date  -- aggregate raw fact first
    )
    SELECT a.*, d.currency_name, d.region
    FROM agg a
    JOIN dim_currency d ON a.currency_pair = d.currency_pair;  -- join aggregated rows, not raw
    
    -- STAGE 3: Covering index on GROUP BY columns
    CREATE NONCLUSTERED INDEX idx_trades_group
    ON trades (trade_date, currency_pair, status)
    INCLUDE (notional_amount);  -- covering: avoids key lookup for SUM column
    
    -- STAGE 4: Incremental pre-aggregation (materialise daily summary table)
    -- Run intraday to keep summary current; GROUP BY runs on summary, not raw
    MERGE daily_trade_summary AS tgt
    USING (
      SELECT trade_date, currency_pair, status, SUM(notional) AS total
      FROM trades
      WHERE trade_date = CAST(GETDATE()-1 AS DATE)  -- only yesterday's data
      GROUP BY trade_date, currency_pair, status
    ) src
    ON tgt.trade_date = src.trade_date
    AND tgt.currency_pair = src.currency_pair
    AND tgt.status = src.status
    WHEN MATCHED THEN UPDATE SET tgt.total = src.total
    WHEN NOT MATCHED THEN INSERT VALUES (src.trade_date, src.currency_pair, src.status, src.total);
    
    
    

🔴 **Result** Combined optimisations: covering index alone reduced runtime from 45 to 12 minutes. Pre-aggregation pattern reduced it further to **90 seconds**. The covering index improved 6 other queries on the same table as a side benefit.

* * *

### Q50. How do you write a SQL query to calculate the number of working days between two dates, excluding weekends and public holidays?

🔵 **Situation** At LSEG, the compliance team needed to calculate the regulatory settlement lag for each trade — the number of working days between trade date and settlement date — to flag trades breaching the T+2 settlement obligation. Standard `DATEDIFF` counted calendar days and failed compliance requirements.

🟡 **Task** Write a working-day calculation in SQL that excluded weekends and a reference table of public holidays.

🟢 **Action**
    
    
    -- Method 1: Date spine approach (accurate, handles all holiday tables)
    -- Requires a pre-built dim_date table with is_working_day flag
    SELECT
      t.trade_id,
      t.trade_date,
      t.settlement_date,
      COUNT(d.date_key)                        AS working_days_to_settlement,
      CASE WHEN COUNT(d.date_key) > 2
           THEN 'T+2 BREACH'
           ELSE 'COMPLIANT'
      END                                      AS settlement_status
    FROM trades t
    JOIN dim_date d
      ON  d.calendar_date BETWEEN DATEADD(DAY, 1, t.trade_date)
                              AND t.settlement_date
      AND d.is_working_day = 1    -- flag set false for weekends and public holidays
    WHERE t.trade_date >= '2024-01-01'
    GROUP BY t.trade_id, t.trade_date, t.settlement_date
    ORDER BY working_days_to_settlement DESC;
    
    -- Method 2: Calculate inline without a date dimension table
    -- (weekends only — no public holiday table)
    SELECT
      trade_id,
      trade_date,
      settlement_date,
      -- Total calendar days minus weekend days in the range
      DATEDIFF(DAY, trade_date, settlement_date)
      -- Subtract full weeks × 2 weekend days
      - (DATEDIFF(WEEK, trade_date, settlement_date) * 2)
      -- Subtract partial weekend days at start/end
      - CASE WHEN DATENAME(WEEKDAY, trade_date)      = 'Sunday'   THEN 1 ELSE 0 END
      - CASE WHEN DATENAME(WEEKDAY, settlement_date) = 'Saturday' THEN 1 ELSE 0 END
                                                  AS working_days
    FROM trades;
    
    -- Method 3: Public holidays subtraction
    SELECT
      t.trade_id,
      DATEDIFF(DAY, t.trade_date, t.settlement_date)
      - (DATEDIFF(WEEK, t.trade_date, t.settlement_date) * 2)
      - (
          SELECT COUNT(*)
          FROM public_holidays ph
          WHERE ph.holiday_date BETWEEN DATEADD(DAY,1,t.trade_date)
                                   AND t.settlement_date
            AND DATENAME(WEEKDAY, ph.holiday_date) NOT IN ('Saturday','Sunday')
        )                                         AS working_days_net_holidays
    FROM trades t;
    
    
    

🔴 **Result** Working-day settlement lag analysis identified 127 trades breaching the T+2 regulatory obligation over a 3-month period — 0.008% of total volume but a compliance exposure that had not been visible with the previous calendar-day calculation. The trades were escalated to the compliance team and reported as required under MiFID II.

* * *

* * *

_End of Document_ _50 Toughest SQL Technical Interview Questions — STAR Format_ _Senior / Business Data Analyst | EDW / Banking / SQL Deep-Dive_ _All answers grounded in real project experience_ _Generated: April 2026_

* * *

**Q51. What is a recursive CTE? Give a use case in banking.**

A recursive CTE calls itself to traverse hierarchical data.
    
    
    WITH RECURSIVE entities AS (
      SELECT 1 AS entity_id, NULL AS parent_id, 'Global Holding Corp' AS entity_name
      UNION ALL SELECT 2, 1, 'Europe Division'
      UNION ALL SELECT 3, 1, 'Asia Division'
      UNION ALL SELECT 4, 2, 'Ireland Ltd'
      UNION ALL SELECT 5, 2, 'Germany GmbH'
      UNION ALL SELECT 6, 4, 'Dublin Software Hub'
      UNION ALL SELECT 7, 3, 'India Tech Center'
      UNION ALL SELECT 8, 7, 'Kochi R&D Lab'
    ),
    hierarchy_entities AS (
      Select entity_id, parent_id, 1 as Level, entity_name from entities where parent_id is NOT NULL
    	UNION ALL
      Select E.entity_id, E.parent_id, H.Level + 1 as Level, E.entity_name from entities AS E
      INNER JOIN hierarchy_entities H ON H.entity_id = E.parent_id
      )
    select H.entity_id, H.parent_id, H.entity_name, E.entity_name as Parent, H.Level from hierarchy_entities AS H
    Inner JOIN entities AS E ON H.parent_id = E.entity_id

Useful for credit risk consolidation where exposure rolls up through legal entity trees.
