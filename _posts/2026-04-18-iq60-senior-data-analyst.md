---
title: "IQ60 Senior Data Analyst"
date: 2026-04-18 05:22:39 
author: shajeebhameed
layout: page
slug: iq60-senior-data-analyst
status: publish
---

## 🎯 Senior Data Analyst – Interview Preparation Guide

> All answers follow the **STAR format** : **S** ituation → **T** ask → **A** ction → **R** esult

* * *

## 🟡 Round 1 – Technical 1: SQL | Python | Data Engineering Basics

* * *

**Q1. Write a SQL query to find duplicate records and remove them efficiently.**

**Answer:**

**Situation:** At J&J Global Services, the consolidated BigQuery model ingested data from 4 legacy sources, frequently producing duplicate rows on policy and claims records.

**Task:** I needed to identify and remove duplicates without truncating the table, preserving the most recent valid record for each key.

**Action:**
    
    
    -- Step 1: Identify duplicates using ROW_NUMBER()
    WITH ranked AS (
      SELECT *,
        ROW_NUMBER() OVER (
          PARTITION BY policy_id, claim_date
          ORDER BY updated_at DESC
        ) AS rn
      FROM claims_staging
    )
    -- Step 2: Delete all but the latest record
    DELETE FROM claims_staging
    WHERE policy_id IN (
      SELECT policy_id FROM ranked WHERE rn > 1
    );
    
    -- In BigQuery (no DELETE with subquery), use MERGE or recreate:
    CREATE OR REPLACE TABLE claims_staging AS
    SELECT * EXCEPT(rn)
    FROM ranked
    WHERE rn = 1;
    
    
    

**Result:** Removed ~15% duplicate rows from the staging layer, improving actuarial query accuracy and reducing downstream reconciliation errors by 40%.

* * *

**Q2. Difference between ROW_NUMBER(), RANK(), DENSE_RANK() with real examples.**

**Answer:**

**Situation:** While building the claims data warehouse at Travelers Insurance, I needed to rank claims handlers by volume and flag ties correctly for the compliance report.

**Task:** Explain and apply the right window function depending on business need.

**Action:**
    
    
    SELECT
      handler_id,
      claim_count,
      ROW_NUMBER() OVER (ORDER BY claim_count DESC) AS row_num,
      -- Always unique: 1, 2, 3, 4 (no ties)
    
      RANK()       OVER (ORDER BY claim_count DESC) AS rnk,
      -- Skips after tie: 1, 2, 2, 4 (gap after tie)
    
      DENSE_RANK() OVER (ORDER BY claim_count DESC) AS dense_rnk
      -- No skip: 1, 2, 2, 3 (continuous after tie)
    FROM handler_summary;
    
    
    

handler_id| claim_count| ROW_NUMBER| RANK| DENSE_RANK  
---|---|---|---|---  
A| 100| 1| 1| 1  
B| 90| 2| 2| 2  
C| 90| 3| 2| 2  
D| 80| 4| 4| 3  
  
**Result:** Used `DENSE_RANK()` for fair performance banding and `ROW_NUMBER()` for deduplication. This distinction was key in the actuarial reporting logic and was later adopted as a team standard.

* * *

**Q3. Given a large dataset, how would you optimize a slow-running SQL query?**

**Answer:**

**Situation:** At Refinitiv (LSEG), a daily FX reconciliation query on SQL Server was taking 45+ minutes, blocking the morning trading reports.

**Task:** Diagnose and fix the bottleneck without changing the business logic.

**Action:** I followed a structured optimisation approach:

  1. **Execution Plan** – Used `SET STATISTICS IO ON` and reviewed the query plan; found a full table scan on a 200M-row table.
  2. **Indexing** – Added a composite non-clustered index on `(trade_date, currency_pair, status)` — the three most-filtered columns.
  3. **Partitioning** – Partitioned the table by `trade_date` to enable partition elimination.
  4. **Query rewrite** – Replaced a correlated subquery with a `JOIN`, eliminating row-by-row processing.
  5. **Avoided SELECT *** – Restricted to only required columns to reduce I/O.


    
    
    -- Before (slow): correlated subquery
    SELECT * FROM trades t
    WHERE status = (SELECT MAX(status) FROM trades WHERE trade_id = t.trade_id)
    
    -- After (fast): JOIN with indexed columns
    SELECT t.trade_id, t.currency_pair, t.amount, t.trade_date
    FROM trades t
    INNER JOIN (
      SELECT trade_id, MAX(status) AS latest_status
      FROM trades
      GROUP BY trade_id
    ) latest ON t.trade_id = latest.trade_id AND t.status = latest.latest_status
    WHERE t.trade_date >= DATEADD(day, -1, GETDATE())
    
    
    

**Result:** Query time dropped from 45 minutes to under 3 minutes — a 93% reduction. This eliminated the morning reporting delay and was flagged as a best practice in the team's SQL playbook.

* * *

**Q4. How do you handle missing or null values in a dataset?**

**Answer:**

**Situation:** During the AXA Insurance data migration, incoming claims data had significant null values in key actuarial fields like `loss_amount` and `reserve_date`.

**Task:** Build a robust null-handling strategy that preserved data integrity for downstream loss reserving calculations.

**Action:**
    
    
    import pandas as pd
    import numpy as np
    
    df = pd.read_parquet("claims_raw.parquet")
    
    # 1. Audit nulls first
    null_summary = df.isnull().sum() / len(df) * 100
    print(null_summary[null_summary > 0])
    
    # 2. Strategy per column type:
    
    # Numeric – fill with median (resistant to outliers)
    df['loss_amount'] = df['loss_amount'].fillna(df['loss_amount'].median())
    
    # Categorical – fill with mode or 'UNKNOWN' sentinel
    df['claim_type'] = df['claim_type'].fillna(df['claim_type'].mode()[0])
    
    # Date – flag as missing, don't impute blindly
    df['reserve_date_missing'] = df['reserve_date'].isnull().astype(int)
    df['reserve_date'] = df['reserve_date'].fillna(pd.Timestamp('1900-01-01'))
    
    # 3. Drop rows where critical key fields are null
    df = df.dropna(subset=['policy_id', 'claim_id'])
    
    # 4. Log what was imputed for audit trail
    print(f"Rows after cleaning: {len(df)}")
    
    
    

**Result:** Reduced null rate in critical fields from 12% to under 0.5%, improving loss reserving model accuracy and passing Microsoft Purview data quality checks. The imputation logic was documented in the STTM for auditor review.

* * *

**Q5. Write a Python/PySpark script to read a large CSV file and process it efficiently.**

**Answer:**

**Situation:** At AXA, raw claims files arrived daily as CSV exports — some exceeding 5GB — and needed to be ingested into Microsoft Fabric for actuarial analysis.

**Task:** Build an efficient ingestion script that wouldn't exhaust memory or timeout.

**Action:**
    
    
    # PySpark approach (used in Microsoft Fabric notebooks)
    from pyspark.sql import SparkSession
    from pyspark.sql.functions import col, to_date, when, lit
    
    spark = SparkSession.builder \
        .appName("AXA_Claims_Ingestion") \
        .config("spark.sql.shuffle.partitions", "200") \
        .getOrCreate()
    
    # Read with schema inference disabled for speed — use explicit schema
    from pyspark.sql.types import StructType, StructField, StringType, DoubleType, DateType
    
    schema = StructType([
        StructField("claim_id",    StringType(), False),
        StructField("policy_id",   StringType(), False),
        StructField("loss_amount", DoubleType(), True),
        StructField("claim_date",  StringType(), True),
        StructField("status",      StringType(), True),
    ])
    
    df = spark.read.csv(
        "abfss://raw@storageaccount.dfs.core.windows.net/claims/",
        schema=schema,
        header=True,
        inferSchema=False   # explicit schema = faster
    )
    
    # Transformations
    df_clean = df \
        .withColumn("claim_date", to_date(col("claim_date"), "yyyy-MM-dd")) \
        .withColumn("loss_amount", when(col("loss_amount") < 0, lit(0)).otherwise(col("loss_amount"))) \
        .filter(col("claim_id").isNotNull()) \
        .dropDuplicates(["claim_id"])
    
    # Write as Delta (ACID, queryable, versioned)
    df_clean.write \
        .format("delta") \
        .mode("overwrite") \
        .partitionBy("claim_date") \
        .save("abfss://silver@storageaccount.dfs.core.windows.net/claims_clean/")
    
    print(f"Processed {df_clean.count():,} records")
    
    
    

**Result:** Ingestion time for a 5GB file dropped from 40 minutes (pandas) to under 4 minutes with PySpark. The Delta output enabled incremental Power BI refresh with sub-second dashboard load times.

* * *

**Q6. What are the different types of SQL joins and when would you use each?**

**Answer:**

**Situation:** At J&J, I regularly joined claims, policy, and handler tables across multiple source systems for actuarial reporting. Choosing the wrong join type caused either data loss or row explosion.

**Task:** Apply the right join type for each reporting scenario.

**Action:**

Join Type| Returns| Use When  
---|---|---  
`INNER JOIN`| Only matching rows in both| You need complete records from both sides  
`LEFT JOIN`| All left rows + matched right| Preserving all policies even if no claims filed  
`RIGHT JOIN`| All right rows + matched left| Rare; usually rewrite as LEFT JOIN  
`FULL OUTER JOIN`| All rows from both, nulls where no match| Reconciling two systems — finding gaps on either side  
`CROSS JOIN`| Cartesian product| Generating date/dimension combinations deliberately  
`SELF JOIN`| Table joined to itself| Hierarchy queries (e.g., manager → employee)  
      
    
    -- Real example: LEFT JOIN to find policies with no claims (lapsed)
    SELECT p.policy_id, p.holder_name, c.claim_id
    FROM policies p
    LEFT JOIN claims c ON p.policy_id = c.policy_id
    WHERE c.claim_id IS NULL  -- policies with zero claims
    
    
    

**Result:** This pattern was used to identify lapsed policies for the actuarial churn model, directly contributing to the $2M+ cost-saving analysis at J&J.

* * *

**Q7. Explain window functions and give a practical example from your work.**

**Answer:**

**Situation:** At Travelers Insurance, the fraud team needed a running total of claim amounts per policy within each quarter to detect abnormal surges.

**Task:** Build a query that could calculate running aggregates without collapsing rows (which `GROUP BY` would do).

**Action:**
    
    
    SELECT
      policy_id,
      claim_date,
      claim_amount,
      SUM(claim_amount) OVER (
        PARTITION BY policy_id, DATE_TRUNC('quarter', claim_date)
        ORDER BY claim_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
      ) AS running_total_qtd,
    
      AVG(claim_amount) OVER (
        PARTITION BY policy_id
        ORDER BY claim_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
      ) AS rolling_7_day_avg,
    
      LAG(claim_amount, 1) OVER (
        PARTITION BY policy_id ORDER BY claim_date
      ) AS prev_claim_amount
    
    FROM claims
    ORDER BY policy_id, claim_date;
    
    
    

**Result:** The fraud team used the running total and rolling average to flag policies where the QTD total exceeded 3x the 7-day rolling average — catching 12 suspicious accounts in the first month of deployment.

* * *

**Q8. How would you design a slowly changing dimension (SCD) in a data warehouse?**

**Answer:**

**Situation:** At Travelers Insurance, the Snowflake data warehouse tracked insurance agents whose territories and commission tiers changed over time. Historical claims needed to reflect the agent's tier _at the time of claim_ , not today's tier.

**Task:** Design an SCD Type 2 to preserve historical accuracy.

**Action:**
    
    
    -- SCD Type 2 agent dimension in Snowflake
    CREATE TABLE dim_agent (
      agent_sk       INT AUTOINCREMENT PRIMARY KEY,   -- surrogate key
      agent_id       VARCHAR,                          -- natural/business key
      agent_name     VARCHAR,
      territory      VARCHAR,
      commission_tier VARCHAR,
      effective_date DATE,
      expiry_date    DATE DEFAULT '9999-12-31',        -- open-ended current record
      is_current     BOOLEAN DEFAULT TRUE
    );
    
    -- On change: close old record, insert new
    UPDATE dim_agent
    SET expiry_date = CURRENT_DATE - 1, is_current = FALSE
    WHERE agent_id = 'AG001' AND is_current = TRUE;
    
    INSERT INTO dim_agent (agent_id, agent_name, territory, commission_tier, effective_date)
    VALUES ('AG001', 'John Murphy', 'Dublin South', 'Tier 2', CURRENT_DATE);
    
    
    

**Result:** Historical claims reports now correctly join on `effective_date <= claim_date <= expiry_date`, giving the actuarial team accurate commission attribution across 5 years of historical data.

* * *

**Q9. What is indexing in SQL and how did you apply it?**

**Answer:**

**Situation:** At Refinitiv, a high-frequency FX trading database was experiencing slow lookups on the `trades` table (200M+ rows) during real-time reconciliation windows.

**Task:** Implement an indexing strategy that supported millisecond-level query response for live trading.

**Action:**

  * Applied a **clustered index** on `trade_id` (unique, sequential — good for range scans).
  * Added **non-clustered composite indexes** on the most-filtered column combinations: `(trade_date, currency_pair)` and `(status, settlement_date)`.
  * Used **covering indexes** (including projected columns) to avoid key lookups:


    
    
    CREATE NONCLUSTERED INDEX IX_trades_date_currency
    ON trades (trade_date, currency_pair)
    INCLUDE (amount, status, trader_id);  -- covering — no extra lookup needed
    
    
    

  * Monitored index usage with `sys.dm_db_index_usage_stats` and dropped unused indexes to reduce write overhead.



**Result:** Query response dropped to under 50ms for real-time lookups. The framework achieved 99.9% data consistency with the Oracle source across all trading sessions.

* * *

**Q10. How do you write efficient CTEs vs subqueries? When do you choose each?**

**Answer:**

**Situation:** At J&J, complex reports joining claims, policy, and handler data had deeply nested subqueries that were unreadable and hard to maintain across a team of 150+ analysts.

**Task:** Refactor for readability and performance without sacrificing correctness.

**Action:**
    
    
    -- Before: nested subqueries (hard to read, re-executes inner query)
    SELECT h.handler_name, sub.total_claims
    FROM handlers h
    JOIN (
      SELECT handler_id, COUNT(*) AS total_claims
      FROM (
        SELECT handler_id FROM claims WHERE status = 'CLOSED'
      ) closed
      GROUP BY handler_id
    ) sub ON h.handler_id = sub.handler_id;
    
    -- After: CTEs (readable, maintainable, single-pass)
    WITH closed_claims AS (
      SELECT handler_id
      FROM claims
      WHERE status = 'CLOSED'
    ),
    handler_totals AS (
      SELECT handler_id, COUNT(*) AS total_claims
      FROM closed_claims
      GROUP BY handler_id
    )
    SELECT h.handler_name, ht.total_claims
    FROM handlers h
    JOIN handler_totals ht ON h.handler_id = ht.handler_id;
    
    
    

**Rule of thumb I applied:**

  * **CTE** – multi-step logic, reused subsets, recursive queries, readability-first
  * **Subquery** – simple one-off filter, correlated lookups
  * **Temp table** – large intermediate results reused many times, statistics needed



**Result:** Refactoring 20+ core reports to CTEs reduced peer review time and onboarding time for new analysts by ~30%, and the pattern was adopted as a coding standard in the team's SQL style guide.

* * *

## 🔵 Round 2 – Technical 2: Cloud | Data Warehousing | BI

* * *

**Q11. How do you design an end-to-end ETL pipeline in a cloud environment?**

**Answer:**

**Situation:** At Travelers Insurance, raw Medicare/Medicaid claims data arrived daily from 3 external systems in different formats (CSV, JSON, XML). The actuarial team needed clean, analytics-ready data by 7am each morning.

**Task:** Design a reliable, auditable, end-to-end pipeline in Snowflake.

**Action:** I implemented a **Bronze-Silver-Gold medallion architecture** :
    
    
    [Source Systems] → [Raw Ingest: Bronze] → [Cleansed: Silver] → [Aggregated: Gold]
         CSV/JSON           Snowflake stage       Validated tables     BI-ready marts
    
    
    

  * **Bronze:** Raw files landed in S3/Azure Blob → loaded via `COPY INTO` Snowflake staging tables (no transformation, full audit trail).
  * **Silver:** Snowflake Tasks triggered cleaning jobs — null handling, deduplication, type casting, business rule validation.
  * **Gold:** Aggregated fact/dim tables built via scheduled Snowflake Tasks, partitioned by `claim_date`.
  * **Monitoring:** Used Snowflake's `QUERY_HISTORY` and `TASK_HISTORY` views to alert on failures via email notification.



**Result:** Pipeline ran end-to-end in under 30 minutes. Data latency reduced from 24 hours to under 15 minutes for the fraud detection team using CDC (Streams & Tasks). Zero SLA breaches in 18 months.

* * *

**Q12. Explain incremental data loading strategies — how did you implement CDC?**

**Answer:**

**Situation:** At Travelers Insurance, the initial full-load pipeline for claims was processing 50M+ rows nightly, taking 4+ hours and consuming excessive Snowflake compute credits.

**Task:** Switch to incremental loading using Change Data Capture to process only new/changed records.

**Action:**
    
    
    -- Step 1: Create a Snowflake Stream on the source table
    CREATE OR REPLACE STREAM claims_stream ON TABLE claims_bronze;
    
    -- Step 2: Snowflake Task reads the stream and merges into Silver
    CREATE OR REPLACE TASK silver_claims_refresh
      WAREHOUSE = COMPUTE_WH
      SCHEDULE = '15 MINUTE'
    AS
    MERGE INTO claims_silver AS target
    USING (
      SELECT *
      FROM claims_stream
      WHERE METADATA$ACTION = 'INSERT'     -- new records only
         OR METADATA$ACTION = 'UPDATE'     -- changed records
    ) AS source
    ON target.claim_id = source.claim_id
    WHEN MATCHED THEN
      UPDATE SET target.status = source.status,
                 target.loss_amount = source.loss_amount,
                 target.updated_at = CURRENT_TIMESTAMP()
    WHEN NOT MATCHED THEN
      INSERT (claim_id, policy_id, loss_amount, status, claim_date, updated_at)
      VALUES (source.claim_id, source.policy_id, source.loss_amount, source.status, source.claim_date, CURRENT_TIMESTAMP());
    
    ALTER TASK silver_claims_refresh RESUME;
    
    
    

**Result:** Nightly processing time dropped from 4 hours to 12 minutes. Compute costs reduced by ~65%. Data latency cut from 24 hours to under 15 minutes, enabling near-real-time fraud alerting.

* * *

**Q13. Difference between Data Lake vs Data Warehouse vs Lakehouse.**

**Answer:**

**Situation:** At AXA Insurance, the architecture team debated whether to use Azure Data Lake, Synapse Analytics (DWH), or Microsoft Fabric (Lakehouse) for the claims platform. I was asked to prepare a recommendation.

**Task:** Clearly articulate the trade-offs and recommend the right approach.

**Action:**

Dimension| Data Lake| Data Warehouse| Lakehouse  
---|---|---|---  
**Data type**|  Raw, unstructured/semi-structured| Structured, curated| Both  
**Schema**|  Schema-on-read| Schema-on-write| Both  
**ACID**|  No (unless Delta/Iceberg)| Yes| Yes (Delta Lake)  
**Cost**|  Low storage| High compute| Balanced  
**Query speed**|  Slow (no indexing)| Fast| Fast (with Delta)  
**Use case**|  Data science, ML, raw archive| BI, reporting, finance| Unified analytics platform  
**Tools**|  ADLS, S3, GCS| Snowflake, BigQuery, Synapse| Microsoft Fabric, Databricks  
  
**My recommendation for AXA:** Microsoft Fabric Lakehouse — it unified the raw storage (OneLake) with structured analytics (Direct Lake Power BI) and provided ACID guarantees via Delta Lake, avoiding the need to maintain two separate systems.

**Result:** Recommendation was approved and implemented. Power BI dashboards loaded in sub-second via Direct Lake mode, and the team avoided ~£200K/year in Synapse licensing costs.

* * *

**Q14. How does Delta Lake ensure ACID transactions?**

**Answer:**

**Situation:** At AXA, the Microsoft Fabric implementation used Delta Lake as the storage format. Stakeholders were concerned about concurrent writes from multiple PySpark notebooks corrupting data.

**Task:** Explain and demonstrate how Delta Lake handles ACID compliance.

**Action:**

Delta Lake achieves ACID via a **transaction log** (`_delta_log/`) — a JSON-based audit trail of every operation:

  * **Atomicity** — writes either fully commit or roll back. No partial files.
  * **Consistency** — schema enforcement prevents invalid data being written.
  * **Isolation** — optimistic concurrency control; conflicts are detected and retried.
  * **Durability** — committed transactions persist in the log even if the cluster crashes.


    
    
    # Concurrent-safe upsert using Delta MERGE
    from delta.tables import DeltaTable
    
    delta_table = DeltaTable.forPath(spark, "abfss://silver/.../claims_clean/")
    
    delta_table.alias("target").merge(
        source=df_updates.alias("source"),
        condition="target.claim_id = source.claim_id"
    ).whenMatchedUpdateAll() \
     .whenNotMatchedInsertAll() \
     .execute()
    
    # View transaction history
    delta_table.history(10).show()
    
    # Time travel — query as of yesterday
    df_yesterday = spark.read.format("delta") \
        .option("timestampAsOf", "2026-04-17") \
        .load("abfss://silver/.../claims_clean/")
    
    
    

**Result:** No data corruption incidents across 18 months of concurrent notebook execution. Time travel was used twice to recover from accidental overwrites, saving hours of reprocessing.

* * *

**Q15. How do you design and maintain BI dashboards for executive stakeholders?**

**Answer:**

**Situation:** At Google Street View (via Cognizant), senior R&D leadership needed real-time visibility into the image ingestion pipeline — from collection to processing to publishing — across a 10-month window.

**Task:** Build and maintain executive dashboards in Looker Studio that were self-serve, accurate, and updated in real time.

**Action:**

  1. **Requirements gathering** – Ran structured sessions with PM and VP-level stakeholders to define the 5–7 KPIs that mattered most (latency rates, publish %, defect counts, SSD utilisation).
  2. **Data modelling** – Built a clean BigQuery view layer to abstract raw tables from the presentation layer, so dashboard SQL never touched raw tables directly.
  3. **Design** – Followed a pyramid layout: headline KPIs at top, trend charts in the middle, drill-down tables at the bottom. Colour-coded RAG status for instant scan-ability.
  4. **Refresh cadence** – Set up streaming inserts via Pub/Sub → BigQuery for near real-time data.
  5. **Governance** – Used Looker Studio's access controls to ensure only authorised teams could view sensitive R&D metrics.



**Result:** Dashboard adoption across 3 global R&D teams. Leadership reported 40% reduction in ad-hoc data requests. The SSD Life Cycle Dashboard I built was specifically cited in a quarterly business review as a model for operational monitoring.

* * *

**Q16. How do you monitor and troubleshoot pipeline failures?**

**Answer:**

**Situation:** At Travelers Insurance, a nightly Snowflake Task pipeline silently failed mid-run, causing the fraud team's morning dashboard to show stale data — discovered only when a flagged claim was missed.

**Task:** Diagnose the root cause and build a monitoring layer to prevent silent failures in future.

**Action:**

  1. **Immediate diagnosis:**


    
    
    -- Check Task run history
    SELECT *
    FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
      SCHEDULED_TIME_RANGE_START => DATEADD('hour', -24, CURRENT_TIMESTAMP()),
      TASK_NAME => 'SILVER_CLAIMS_REFRESH'
    ))
    ORDER BY SCHEDULED_TIME DESC;
    -- Found: FAILED with "Statement reached its statement timeout" error
    
    
    

  2. **Root cause:** A sudden data volume spike (3× normal) caused the task to hit its 30-minute timeout.
  3. **Fix:**
     * Increased task timeout and right-sized the virtual warehouse to `LARGE` for peak windows.
     * Added a fallback: if the stream hasn't been consumed in 20 minutes, a monitoring task sends an alert via Snowflake's `SYSTEM$SEND_EMAIL`.
  4. **Proactive monitoring layer:**


    
    
    -- Data freshness check — alert if Silver hasn't updated in 30 min
    CREATE OR REPLACE TASK monitor_freshness
      SCHEDULE = '10 MINUTE'
    AS
      SELECT SYSTEM$SEND_EMAIL(
        'alert-list@company.com',
        'Pipeline Delay Alert',
        'Claims Silver table has not refreshed in 30 minutes.'
      )
      WHERE (SELECT DATEDIFF('minute', MAX(updated_at), CURRENT_TIMESTAMP()) FROM claims_silver) > 30;
    
    
    

**Result:** Zero undetected failures in the following 12 months. The monitoring pattern was applied to all 8 pipelines in the warehouse. Mean time to detection (MTTD) reduced from hours to under 10 minutes.

* * *

**Q17. How have you used BigQuery for large-scale analytics?**

**Answer:**

**Situation:** At J&J Global Services, 4 legacy databases were consolidated into BigQuery to serve 150+ analysts. Queries ran against tables with 500M+ rows and initial query costs were high.

**Task:** Optimise the BigQuery environment for cost and performance at scale.

**Action:**

  * **Partitioning** – Partitioned fact tables by `event_date` (date-based queries only scan the relevant partition).
  * **Clustering** – Clustered on `policy_type` and `region` (the most-filtered dimensions), reducing bytes scanned by 60–70%.
  * **Materialised views** – Pre-aggregated daily summary tables that dashboards queried instead of raw tables.
  * **Slot reservations** – Moved high-priority analytical workloads to reserved slots to avoid on-demand billing spikes.
  * **Column selection discipline** – Enforced `SELECT specific_columns` over `SELECT *` via code review, since BigQuery charges by bytes scanned.


    
    
    -- Partitioned + clustered table creation
    CREATE TABLE `project.dataset.policy_events`
    PARTITION BY DATE(event_date)
    CLUSTER BY policy_type, region
    AS
    SELECT * FROM `project.dataset.policy_events_raw`;
    
    
    

**Result:** Average query cost reduced by 55%. The BigQuery model served as the single source of truth for 150+ analysts and executives across 3 continents with sub-5-second dashboard response times.

* * *

**Q18. Explain Snowflake architecture — what makes it different?**

**Answer:**

**Situation:** At Travelers Insurance, I was the lead for the Snowflake data warehouse build. I had to present the architecture to the CTO and justify the platform choice over Redshift.

**Task:** Articulate Snowflake's unique architecture and map it to our specific use case.

**Action:**

Snowflake's key differentiator is its **three-layer separated architecture** :

  1. **Storage layer** – Data stored as compressed columnar files in cloud object storage (S3/Azure Blob). Independent of compute — you pay only for storage used.
  2. **Compute layer (Virtual Warehouses)** – Independent, scalable compute clusters. Multiple warehouses can query the same data simultaneously without contention.
  3. **Cloud Services layer** – Query optimisation, metadata management, access control, and transaction management handled centrally.



**How I leveraged this:**

  * **Separate warehouses per workload** — `XSMALL` for ELT jobs, `LARGE` for actuarial queries. No resource contention.
  * **Auto-suspend/resume** — warehouses shut down when idle, reducing idle compute cost to zero.
  * **Multi-cluster warehouse** — enabled during peak month-end reporting to auto-scale horizontally.
  * **Zero-copy cloning** — cloned production tables for UAT testing without duplicating storage costs.



**Result:** 65% lower compute cost vs the previous SQL Server DWH. Zero performance degradation during simultaneous ETL + BI workloads. Actuarial queries that took 20 minutes on SQL Server ran in under 90 seconds on Snowflake.

* * *

**Q19. How do you ensure data quality across a pipeline?**

**Answer:**

**Situation:** At AXA, the claims data came from 3 source systems with inconsistent formats, duplicate IDs, and occasional corrupted numeric fields. Inaccurate data flowing to the actuarial model could result in incorrect reserve calculations with material financial impact.

**Task:** Build a systematic data quality framework that validated data at every layer.

**Action:** I implemented a **layered DQ approach** :
    
    
    from pyspark.sql.functions import col, count, when, isnan
    
    def run_dq_checks(df, table_name):
        results = {}
    
        # 1. Completeness — null rates per column
        for col_name in df.columns:
            null_count = df.filter(col(col_name).isNull() | isnan(col(col_name))).count()
            null_pct = null_count / df.count() * 100
            results[f"{col_name}_null_pct"] = round(null_pct, 2)
            if null_pct > 5:
                print(f"⚠️ WARN: {col_name} has {null_pct:.1f}% nulls")
    
        # 2. Uniqueness — duplicate key check
        dup_count = df.count() - df.dropDuplicates(["claim_id"]).count()
        results["duplicate_claim_ids"] = dup_count
        if dup_count > 0:
            raise ValueError(f"❌ FAIL: {dup_count} duplicate claim_ids found in {table_name}")
    
        # 3. Validity — business rule checks
        invalid_amounts = df.filter(col("loss_amount") < 0).count()
        results["negative_loss_amounts"] = invalid_amounts
    
        # 4. Referential integrity
        orphan_claims = df.filter(col("policy_id").isNull()).count()
        results["orphan_claims"] = orphan_claims
    
        return results
    
    dq_report = run_dq_checks(df_silver, "claims_silver")
    
    
    

Additionally, I used **Microsoft Purview** to establish data lineage and flagged DQ violations to a monitoring dashboard. A failed DQ check halted the pipeline and sent an alert before any bad data reached Gold/BI layers.

**Result:** Manual data review effort reduced by 40%. Loss reserving accuracy improved significantly. Zero data quality incidents reached the actuarial model in the final 12 months of the engagement.

* * *

**Q20. What is your approach to data modelling — star schema vs normalised?**

**Answer:**

**Situation:** At J&J, the BigQuery consolidation project required a modelling decision that would affect 150+ analysts and multiple BI tools consuming the data.

**Task:** Choose and implement the right data modelling approach for an insurance analytics use case.

**Action:**

I chose a **star schema** (dimensional model) for the analytics layer:
    
    
                        dim_policy
                            |
    dim_handler — fact_claims — dim_date
                            |
                        dim_claim_type
    
    
    

**Reasoning:**

  * **Star schema** — denormalised, fewer joins, faster BI queries, easier for non-technical analysts.
  * **Normalised (3NF)** — better for OLTP, storage-efficient, but too many joins for analytical workloads.
  * **Snowflake schema** — further normalised dims; I avoided this as it complicated Power BI relationships unnecessarily.



I kept the **Silver layer normalised** (staging) and the **Gold layer as a star schema** (presentation), giving us the best of both.

**Result:** Power BI reports that previously required 8-table joins in the normalised source now ran against 2-table star schema queries. Dashboard load times improved by 70%. New analysts could self-serve without needing to understand the source system complexity.

* * *

## 🔴 Round 3 – Managerial / Behavioural

* * *

**Q21. Tell me about the most challenging data project you worked on.**

**Answer:**

**Situation:** At J&J Global Services, I was tasked with consolidating 4 legacy data sources — each with different schemas, data quality standards, and business ownership — into a single BigQuery model that would serve as the source of truth for 150+ analysts globally.

**Task:** Deliver the migration within 6 months while maintaining zero disruption to existing actuarial and compliance reporting.

**Action:**

  * Conducted a full data audit of all 4 sources, profiling 200+ business attributes and documenting transformation rules in STTM.
  * Built a parallel-run approach: new BigQuery model ran alongside legacy systems for 8 weeks so discrepancies could be identified before cutover.
  * Held weekly alignment meetings with data owners from each legacy system and the actuarial team to resolve conflicting business rules.
  * Built automated reconciliation reports comparing row counts, aggregates, and key metric values between old and new systems.
  * Managed stakeholder expectations when a data quality issue in one source delayed the timeline by 3 weeks — communicated proactively and adjusted the phased cutover plan.



**Result:** Delivered the consolidated model on revised timeline with zero data loss. Post-migration, ad-hoc query volume from analysts dropped 35% because data was now discoverable and self-serve. The $2M+ cost-saving analysis that followed was only possible because all data was in one trusted model.

* * *

**Q22. How do you handle tight deadlines and multiple deliverables?**

**Answer:**

**Situation:** At Cognizant/Google, I was simultaneously supporting 3 active dashboard deliverables while also being asked to lead an urgent data quality investigation after an ingestion pipeline produced anomalous latency figures ahead of a quarterly R&D review.

**Task:** Manage all commitments without missing the executive review deadline.

**Action:**

  * Immediately triaged by impact: the pipeline investigation had the highest business risk (it would affect the executive presentation), so I reprioritised it to slot 1.
  * Communicated transparently with the dashboard stakeholders: gave a revised 48-hour estimate with a clear explanation of why.
  * Divided the investigation into parallel tracks — I took the BigQuery query analysis while a colleague checked the AppScript ingestion script — halving the resolution time.
  * Used JIRA to maintain visibility across all workstreams so nothing fell through the cracks.
  * Blocked two 2-hour deep work windows each day to protect focus time on the investigation.



**Result:** Identified and resolved the pipeline bottleneck within 18 hours. The executive review used accurate data. All three dashboards were delivered within the revised timeline. The experience led me to introduce a sprint-style work-in-progress limit for the team to prevent overloading in future.

* * *

**Q23. Describe a time you faced a production failure. How did you handle it?**

**Answer:**

**Situation:** At Travelers Insurance, the nightly Snowflake CDC pipeline silently failed at 2am due to a compute timeout caused by an unexpected 3× data volume spike. The fraud detection team discovered the issue at 8am when their dashboard showed day-old data — by which point a suspicious claim had gone unactioned for 6 hours.

**Task:** Restore data pipeline integrity immediately, minimise business impact, and prevent recurrence.

**Action:**

  1. **Immediate response (first 30 minutes):**
     * Identified the failed Task via `TASK_HISTORY` in Snowflake.
     * Manually triggered a catch-up run with an upgraded `LARGE` warehouse to process the backlog.
     * Notified the fraud team with an ETA and the list of claims that needed manual review in the interim.
  2. **Root cause analysis:**
     * The spike was caused by a source system batch that had been delayed 3 days and flushed all at once.
     * The task timeout of 30 minutes was too tight for exceptional volume scenarios.
  3. **Permanent fixes:**
     * Increased task timeout and set dynamic warehouse sizing based on stream volume.
     * Built a data freshness monitoring task that fires an email alert if Silver hasn't refreshed within 30 minutes of schedule.
     * Added a volume anomaly check: if incoming row count > 2× 7-day average, the task pauses and sends an alert for human review before proceeding.



**Result:** Dashboard data restored within 2 hours. No fraudulent claims slipped through undetected. The monitoring framework prevented 4 near-miss failures in the following 6 months, catching them within 10 minutes each time.

* * *

**Q24. How do you collaborate with business teams who aren't technical?**

**Answer:**

**Situation:** At J&J, the actuarial and claims teams had strong domain expertise but no SQL knowledge. They needed to specify business rules for the data transformation layer — rules that I then had to implement precisely in the pipeline.

**Task:** Bridge the gap between business logic and technical implementation without mistranslation.

**Action:**

  * Ran structured requirements workshops using plain-language templates: "When [event happens], the system should [do this], unless [exception]."
  * Translated every business rule into a testable data specification before writing any code, and had the business owner sign off on it.
  * Used the STTM (Source-to-Target Mapping) document as the shared contract — business used it to describe intent, I used it to implement logic.
  * Shared data previews (not code) with stakeholders during development — showing sample rows before and after transformation so they could validate visually.
  * Held monthly "data office hours" where analysts could ask questions and flag data anomalies directly, building trust and reducing email noise.



**Result:** Zero transformation disputes at UAT. The STTM I built covered 200+ attributes and became the reference document for the entire analytics org, used for onboarding new analysts 12 months after the project concluded.

* * *

**Q25. What would you do if your data pipeline fails just before a client presentation?**

**Answer:**

**Situation:** Hypothetical — but I faced a near-equivalent at Google, where the Street View publication pipeline showed incorrect figures 2 hours before a senior R&D review meeting.

**Task:** Resolve the data issue or mitigate its impact within the available window.

**Action:**

My approach follows a clear priority sequence:

  1. **Assess in 5 minutes** — is this a data issue (wrong values) or a display issue (dashboard rendering)? Check the underlying BigQuery table directly. If the data is actually correct, it's a dashboard refresh problem — easier to fix.
  2. **Communicate immediately** — notify the presenter and meeting organiser. Never let a stakeholder walk into a meeting with stale data unknowingly.
  3. **Determine recoverability** — can I re-run the pipeline and get fresh data in time? If yes, trigger it immediately with a higher-priority warehouse/resource allocation.
  4. **Prepare a fallback** — pull the last known good dataset from yesterday's run. Present with a clear caveat: "Data as of [date], refreshed figures available post-meeting."
  5. **Post-incident** — root cause analysis, monitoring improvement, and a brief write-up shared with the team to prevent recurrence.



**Result (real incident):** Identified a BigQuery view had been accidentally overwritten during a schema change. Restored the previous version using BigQuery's table snapshot in under 45 minutes. The R&D review proceeded with accurate data and the presenter was never aware of the near-miss.

* * *

**Q26. Describe a time you identified a cost saving or process improvement through data.**

**Answer:**

**Situation:** At J&J Global Services, I was asked to perform a routine analysis of claims processing costs across underwriting regions.

**Task:** Deliver a standard cost report, but I noticed anomalies in the data while building the model.

**Action:**

  * During data profiling, I identified that 3 specific claim categories were being routed through a premium third-party processing vendor despite meeting the criteria for the lower-cost internal pathway.
  * Dug deeper: the routing logic in the legacy system had a hard-coded threshold that hadn't been updated in 4 years — it was overriding the intended business rules.
  * Quantified the impact: pulled 3 years of routing data, calculated the cost differential per claim type, and projected annualised over-spend.
  * Presented the finding with a clear slide: "Current state cost → corrected routing cost → projected annual saving."
  * Worked with the business rules team to update the threshold and validate the fix with a 30-day parallel run.



**Result:** The corrected routing logic was implemented within 6 weeks. Projected annual saving: $2M+. The analysis was shared at VP level and resulted in a broader audit of 12 other legacy business rules, uncovering further optimisation opportunities.

* * *

**Q27. How do you handle disagreements with stakeholders over data interpretation?**

**Answer:**

**Situation:** At Google Street View, the operations team disputed my analysis showing a 25% bottleneck in the ingestion pipeline. Their view was that the latency figures were within acceptable thresholds and the issue lay elsewhere.

**Task:** Resolve the disagreement constructively and ensure the correct root cause was identified.

**Action:**

  * Listened fully to their interpretation first — I wanted to understand what evidence they were using and whether they had context I lacked.
  * Proposed a data-led resolution: rather than debating verbally, I said, "Let's agree on what the data should look like if my hypothesis is correct, and if it's wrong, we'll see that in the numbers."
  * Built a shared diagnostic query in BigQuery that both teams could inspect and validate — making the data the neutral arbiter.
  * When the query results confirmed the bottleneck, I framed it collaboratively: "The good news is we've now pinpointed exactly where to focus," rather than "I was right."



**Result:** The bottleneck was confirmed and addressed in the following quarter, delivering 25% process optimisation. The operations team became advocates for data-led decision making going forward. The relationship was strengthened, not damaged, by the disagreement.

* * *

**Q28. Tell me about a time you had to learn a new technology quickly.**

**Answer:**

**Situation:** When I joined the AXA Insurance engagement at Cognizant, the project had already committed to Microsoft Fabric as the platform — a product I had not used before, having worked primarily in BigQuery and Snowflake.

**Task:** Get productive in Microsoft Fabric within 3 weeks to meet the first sprint delivery date.

**Action:**

  * Spent the first week on structured self-study: Microsoft Learn paths for Fabric, focusing on OneLake, Lakehouse, and Direct Lake mode — the specific features we'd use.
  * Identified the 3 biggest differences from BigQuery: the Delta Lake format, the Fabric notebook PySpark environment, and Direct Lake vs Import mode in Power BI.
  * Built a small proof-of-concept pipeline on my own before writing any production code — ingesting a sample claims file end-to-end through Bronze to Silver to a Power BI report.
  * Paired with a Microsoft Fabric specialist on the client side for the first 2 weeks to accelerate the learning curve.
  * Documented my findings in a team wiki so the rest of the squad could ramp up faster.



**Result:** Delivered the first Fabric notebook for Bronze ingestion on schedule in week 3. By month 2, I was leading the architecture decisions for the Silver layer and training two junior analysts on the platform. The engagement completed successfully with Power BI dashboards achieving sub-second load via Direct Lake.

* * *

**Q29. How do you manage stakeholder expectations when a project is delayed?**

**Answer:**

**Situation:** During the J&J BigQuery migration, a data quality issue discovered in one of the legacy sources — missing policy IDs across ~8% of records — required a 3-week delay to the planned cutover date.

**Task:** Communicate the delay without damaging stakeholder trust or the project timeline beyond what was necessary.

**Action:**

  * Informed key stakeholders within 24 hours of discovering the issue — no waiting until the problem was solved.
  * Prepared a clear impact statement: what was delayed, why, by how long, and what was being done about it.
  * Offered two options: (1) delay the full cutover by 3 weeks, or (2) proceed with a partial cutover excluding the affected policy subset, with a follow-on patch in 2 weeks. Stakeholders chose option 2.
  * Set up a weekly 15-minute progress update for the duration of the delay so stakeholders always had current information without having to ask.
  * Delivered the partial cutover on time, and the fix for the affected subset 11 days later (ahead of the 2-week estimate).



**Result:** The project sponsors described the communication as "exemplary." Despite the delay, confidence in the team's delivery capability increased. The project was cited internally as a model for large-scale data migration governance.

* * *

**Q30. Why are you interested in this role / what motivates you as a data analyst?**

**Answer:**

**Situation:** After 11+ years working across Insurance, Finance, Geospatial, and Telecom, I've developed a clear sense of what I find most energising in my work.

**Task:** Reflect genuinely on what drives me and connect it to the role.

**Action/Reflection:** What motivates me most is the moment when messy, fragmented data becomes a clear, trusted insight that actually changes a decision. At J&J, that was a $2M cost-saving recommendation that came from noticing an anomaly in a claims routing rule. At Google, it was a pipeline latency chart that unlocked a 25% process optimisation nobody had spotted before.

I'm particularly drawn to roles where:

  * The data problems are complex and cross-functional (multiple source systems, stakeholders with competing needs)
  * The output has real business consequence — not just dashboards that get glanced at
  * There's an opportunity to build something that outlasts the project — a framework, a standard, a pattern the team keeps using



I'm also motivated by collaboration. My best work has happened in environments where analysts, engineers, and business teams are genuinely aligned — and I enjoy being the person who builds that alignment.

**Result:** I look for roles where the data challenges are real, the stakeholders are engaged, and there's room to deliver work I'm proud of. That's what draws me to this opportunity.

* * *

## 🟠 Round 4 – Advanced Technical | Architecture | Scenario-Based

* * *

**Q31. How would you build a real-time fraud detection data pipeline?**

**Answer:**

**Situation:** At Travelers Insurance, the fraud team needed near-real-time alerting on suspicious claims — not the next-day batch reports they were getting.

**Task:** Design a pipeline that reduced claims data latency from 24 hours to under 15 minutes.

**Action:**
    
    
    [Claims Source System]
            ↓
    [Snowflake Stream (CDC)]
            ↓
    [Snowflake Task — every 15 min]
            ↓
    [Silver: cleansed + validated claims]
            ↓
    [Gold: fraud_risk_scores view]
            ↓
    [Power BI Dashboard — DirectQuery]
            ↓
    [Email Alert if risk_score > threshold]
    
    
    

Key design decisions:

  * **Streams & Tasks** for CDC — lightweight, native Snowflake, no additional tooling needed.
  * **MERGE statement** in the task to handle inserts and updates idempotently.
  * **Risk score view** using SQL window functions to flag claims where current amount > 3× rolling 7-day average for that policy.
  * **DirectQuery mode** in Power BI so dashboards always show live Snowflake data without scheduled refresh lag.



**Result:** Data latency reduced from 24 hours to under 15 minutes. Fraud team flagged 12 high-risk accounts in the first month that would have gone undetected under the previous batch process.

* * *

**Q32. How do you approach data governance and compliance in your pipelines?**

**Answer:**

**Situation:** At AXA Insurance, the platform handled sensitive personal and medical claims data, subject to GDPR and internal audit requirements. Any data movement needed to be traceable and access-controlled.

**Task:** Implement governance and compliance controls without adding friction to the analytics workflow.

**Action:**

  1. **Data Lineage** – Implemented Microsoft Purview to auto-scan all assets (OneLake, SQL endpoints, Power BI datasets) and map data lineage from source to report automatically.
  2. **Access Control** – Applied RBAC at the Fabric workspace level: actuarial team → Gold layer only; engineering team → Bronze/Silver + Gold; no direct raw access for BI consumers.
  3. **Data Masking** – Used Dynamic Data Masking on PII fields (`patient_id`, `policy_holder_name`) in the Silver layer SQL endpoint — analysts saw masked values unless granted explicit unmask permissions.
  4. **Audit Trail** – Every pipeline run logged to a `pipeline_audit` table (run_id, source, rows_processed, rows_rejected, run_by, timestamp). Failed DQ checks logged with full detail.
  5. **Retention** – Delta Lake time travel configured for 30-day retention, allowing point-in-time recovery for audit and incident investigation.



**Result:** Passed first GDPR compliance audit with zero findings. Microsoft Purview lineage report was submitted directly to the auditors — no manual documentation needed. Full data traceability from source system to Power BI visual in a single lineage graph.

* * *

**Q33. How have you used Python for data automation beyond basic scripting?**

**Answer:**

**Situation:** At Google Street View, the defect reporting process required engineers to manually copy bug details from internal tools into a tracking spreadsheet — a 3-hour weekly task prone to transcription errors.

**Task:** Automate the end-to-end defect reporting pipeline using Python and internal APIs.

**Action:**
    
    
    import requests
    import gspread
    from google.oauth2.service_account import Credentials
    from datetime import datetime
    
    # Authenticate to internal Bug Tracking API
    headers = {"Authorization": f"Bearer {API_TOKEN}"}
    response = requests.get(
        "https://internal-bugtracker.google.com/api/v1/bugs",
        headers=headers,
        params={"status": "OPEN", "project": "street-view-ingestion"}
    )
    bugs = response.json()["bugs"]
    
    # Transform to flat structure
    rows = []
    for bug in bugs:
        rows.append([
            bug["id"],
            bug["title"],
            bug["severity"],
            bug["assignee"],
            bug["created_at"],
            bug["days_open"],
            bug.get("resolution_time_hours", "")
        ])
    
    # Write to Google Sheet via AppScript-compatible Sheets API
    creds = Credentials.from_service_account_file("service_account.json",
        scopes=["https://www.googleapis.com/auth/spreadsheets"])
    gc = gspread.authorize(creds)
    sheet = gc.open("Street View Defect Tracker").sheet1
    sheet.clear()
    sheet.append_rows(rows, value_input_option="USER_ENTERED")
    
    # Trigger Looker Studio refresh via URL parameter
    print(f"Updated {len(rows)} bugs at {datetime.now()}")
    
    
    

This ran via a scheduled Cloud Function, firing every 6 hours.

**Result:** Eliminated 3 hours/week of manual reporting. Average bug-to-resolution time reduced by 18 hours because engineers saw new defects within 6 hours of creation rather than at the next weekly sync. Resolution efficiency improved by 20%.

* * *

**Q34. Explain a time you had to present complex data findings to a non-technical audience.**

**Answer:**

**Situation:** After my deep-dive analysis at J&J identified $2M+ in potential cost savings from misconfigured claims routing logic, I needed to present the finding to a VP-level audience with no data background.

**Task:** Translate a technically complex root cause into a compelling, actionable business case.

**Action:**

  * **Started with the "so what":** Opened with: "We've identified $2M+ in annual over-spend that can be recovered with a single routing rule change." — not with the SQL or the data model.
  * **Used an analogy:** Described the routing logic issue like a toll road with the wrong speed limit signs — vehicles were being sent to the expensive lane even though they qualified for the free lane.
  * **Visualised, not tabled:** Built a 3-slide summary with a Sankey flow chart (where claims were going vs where they should go), a before/after cost bar chart, and a simple implementation timeline.
  * **Avoided jargon:** No mention of legacy thresholds, stored procedures, or STTM. Instead: "a rule that hadn't been reviewed since 2020."
  * **Anticipated questions:** Prepared a 1-page appendix with the full data methodology for the CFO's team to validate independently.



**Result:** The VP approved the fix in the same meeting. The routing correction was implemented within 6 weeks. The presentation style became the template for all future insight reports in the analytics team.

* * *

**Q35. How do you prioritise when you have competing stakeholder requests?**

**Answer:**

**Situation:** At Cognizant/Google, I regularly had requests from 3–4 stakeholder groups simultaneously: the R&D team needed pipeline metrics, compliance needed a data quality report, and the engineering team needed an ad-hoc investigation into a latency spike.

**Task:** Triage and sequence the work fairly without losing any stakeholder's trust.

**Action:** I use a simple **impact × urgency** matrix to triage:

Request| Impact| Urgency| Priority  
---|---|---|---  
Latency spike investigation| High (affects publishing SLA)| High (live issue)| **1st**  
Compliance DQ report| High (audit deadline)| Medium (3 days)| **2nd**  
R&D pipeline metrics| Medium| Low (weekly cadence)| **3rd**  
  
  * Communicated my triage decision to each stakeholder with a brief explanation and ETA — so no one felt ignored.
  * For the lower-priority items, I gave a firm commitment date (not "when I have time").
  * Used JIRA to make my workload visible to my manager and the team, enabling re-allocation if needed.



**Result:** All three deliverables were completed within their respective deadlines. Stakeholders appreciated the transparency — several commented that knowing where they stood in the queue was more valuable than being deprioritised without explanation.

* * *

**Q36. Describe your approach to documentation and knowledge sharing.**

**Answer:**

**Situation:** At J&J, I inherited a data warehouse with no documentation. When a query broke, it took days to understand the original intent because the logic existed only in the heads of analysts who had since left.

**Task:** Build documentation practices that would survive team changes and scale to 150+ users.

**Action:**

  * Created a **STTM (Source-to-Target Mapping)** for every table — capturing source system, field name, transformation rule, data type, and business definition for 200+ attributes.
  * Wrote **SQL inline comments** on all stored procedures and views explaining the "why," not just the "what."
  * Built a **data dictionary** in Confluence that non-technical stakeholders could use to understand what each field meant in business terms.
  * Ran monthly **"data office hours"** sessions where analysts could ask questions — these were recorded and added to the knowledge base.
  * Introduced a **PR review checklist** requiring documentation updates alongside any code change.



**Result:** When 3 team members left within 6 months, onboarding their replacements took 2 weeks instead of the previous 8 weeks. The STTM document was still in active use 18 months after I created it.

* * *

**Q37. Have you ever pushed back on a stakeholder request? How did you handle it?**

**Answer:**

**Situation:** At Google Street View, a senior PM requested a dashboard metric that would show the publishing success rate for a specific 3-day window in isolation — without context of the full trend. The metric would have made performance look significantly better than it actually was by excluding a problematic batch.

**Task:** Decline or reshape the request without damaging the relationship.

**Action:**

  * Did not refuse outright. Instead, asked a clarifying question: "I want to make sure I build the right thing — can you help me understand the decision this metric will support?"
  * When the PM explained they needed to demonstrate progress to leadership, I offered an alternative: show the 3-day window _in context_ of the 30-day trend, with an annotation explaining the improvement. This told the same positive story honestly.
  * Explained my concern directly but diplomatically: "If the 30-day trend is excluded, the metric could be questioned in the Q&A — this framing protects the narrative."



**Result:** The PM agreed to the contextual view. The dashboard passed leadership review without challenge. The PM later told me the annotated trend chart was more persuasive than the isolated metric would have been. The relationship was strengthened by the honest pushback.

* * *

**Q38. How do you stay current with developments in data and analytics?**

**Answer:**

**Situation:** The data ecosystem evolves rapidly — tools I used 3 years ago (e.g., traditional ETL, on-prem SQL Server) have been substantially replaced or augmented by cloud-native and AI-assisted alternatives.

**Task:** Maintain technical relevance across cloud platforms, BI tools, and data engineering patterns.

**Action:** My learning approach is structured into three layers:

  1. **Deliberate practice** — I learn by building. When Microsoft Fabric was announced, I built a personal end-to-end lakehouse project before using it professionally. Same with Snowflake Streams & Tasks.
  2. **Curated reading** — I follow: the Snowflake blog, Google Cloud data engineering updates, dbt Community, and the Databricks engineering blog. I skim for what's production-relevant vs what's hype.
  3. **Community & peer learning** — I'm active in the Data Engineering Dublin meetup community and have built relationships with peers across different platforms. When Purview came up at AXA, I already had a contact who'd implemented it at another firm.



**Current interests:** I'm closely watching the evolution of AI-assisted data quality (e.g., dbt + LLM-powered anomaly detection) and the shift toward semantic layers in BI (e.g., Looker's LookML vs Power BI's unified semantic model).

**Result:** I've successfully delivered projects on 5 distinct cloud data platforms in the last 6 years, each time ramping to production quality within 3–4 weeks. Continuous learning has been a direct enabler of that adaptability.

* * *

**Q39. Tell me about a time you mentored or developed a junior team member.**

**Answer:**

**Situation:** At Wipro/Cisco, I was the technical lead and had 2 junior analysts on my team who were strong in Excel but had limited SQL and ETL experience. They were expected to contribute to complex data warehouse work from month 2.

**Task:** Accelerate their technical development without it becoming a bottleneck on delivery.

**Action:**

  * **Week 1:** Ran a 2-day SQL intensive with them using real Cisco TMS data — hands-on, not slides. Focused on joins, aggregations, and window functions because those covered 80% of the day-to-day work.
  * **Pair coding:** For the first month, they wrote SQL drafts and I reviewed them with tracked changes and comments explaining the reasoning, not just the fix.
  * **Graduated responsibility:** Started them on well-defined reporting queries, moved to ETL transformation logic, then to writing their own validation frameworks.
  * **Code review standard:** Introduced a 2-person peer review for all production SQL — this caught errors but also forced explanation, which deepened their understanding.
  * **Recognition:** Made sure their contributions were visible in sprint reviews and stakeholder updates — building their confidence alongside their skills.



**Result:** Both analysts were independently producing and deploying production SQL within 3 months. Query error rates in their code dropped by 35% after introducing peer review. One of them went on to lead their own analytics workstream within 12 months.

* * *

**Q40. Where do you see yourself in 3–5 years?**

**Answer:**

**Situation:** After 11+ years across multiple industries and platforms, I have a clear sense of the direction I want to develop in.

**Task:** Give an honest, grounded answer that reflects genuine ambition without overpromising.

**Action/Response:**

In the near term, I want to deepen my impact as a Senior Data Analyst — specifically in environments where I can work on harder problems: complex multi-source data estates, real-time analytics, and closer integration between data and business decision-making.

Over 3–5 years, I see myself moving into a **Lead Data Analyst or Analytics Engineering** role — one where I'm setting standards, shaping the data architecture, and developing the analytical capability of a team around me, not just delivering individual workstreams.

I'm also increasingly interested in the intersection of data quality, observability, and AI-assisted analytics — areas where I think the next wave of value creation in data will come from.

What I'm _not_ looking to do is drift into pure management divorced from technical work. The analysts I most respect are those who maintain hands-on depth while expanding their influence. That's the model I want to follow.

**Result:** I share this because I think it's relevant to what you're hiring for — you need someone who will grow with the role and contribute beyond the immediate brief, and that's genuinely what I'm looking for too.

* * *

## 🟢 Round 4 – Scenario / Problem Solving

* * *

**Q41. You discover that a dashboard used by the CFO has been showing incorrect revenue figures for 3 months. What do you do?**

**Answer:**

**Situation (hypothetical, based on real experience patterns):** A data quality issue affecting executive reporting — the highest-stakes scenario in data analytics.

**Task:** Handle it with urgency, transparency, and a clear remediation path.

**Action — my step-by-step response:**

  1. **Verify before escalating** — confirm the error is real, quantify the magnitude, and identify the exact time period and fields affected. Don't raise a fire alarm on a suspicion.
  2. **Notify immediately** — inform my manager and the CFO's data point-of-contact within the hour. The cardinal rule: no executive should discover a data error from someone else before hearing it from the data team.
  3. **Contain** — take the incorrect dashboard offline or add a prominent "Data Under Review" banner. A wrong number with a banner is better than a wrong number presented as fact.
  4. **Root cause** — trace the lineage backwards: dashboard → BI semantic model → Gold table → transformation logic → source. Find the exact point of failure.
  5. **Quantify the impact** — produce a corrected dataset and provide the CFO's team with the accurate figures for all affected periods.
  6. **Fix and validate** — correct the underlying logic, run reconciliation against a known-good source, add automated DQ checks so this class of error is caught in future.
  7. **Post-mortem** — document what happened, why it wasn't caught earlier, and what systemic changes will prevent recurrence.



**Result mindset:** The goal is not to minimise blame — it's to restore trust as quickly as possible through transparency and decisive action. How a data team handles an error matters as much as the error itself.

* * *

**Q42. How would you handle a situation where two data sources give conflicting numbers for the same metric?**

**Answer:**

**Situation:** At J&J, during the 4-source BigQuery consolidation, the legacy policy system and the Salesforce CRM showed different total active policy counts — a 7% discrepancy that neither business owner could explain.

**Task:** Resolve the discrepancy and establish which number was authoritative.

**Action:**

  1. **Define the metric precisely** — "active policy" meant different things: one system used `status = 'ACTIVE'`, the other used `renewal_date >= today`. Different filters, not necessarily wrong data.
  2. **Trace at row level** — exported both datasets with their primary keys and did a full outer join to identify: records in A but not B, records in B but not A, and records in both with different values.


    
    
    SELECT
      COALESCE(a.policy_id, b.policy_id) AS policy_id,
      a.status AS legacy_status,
      b.status AS crm_status,
      CASE
        WHEN a.policy_id IS NULL THEN 'Only in CRM'
        WHEN b.policy_id IS NULL THEN 'Only in Legacy'
        ELSE 'In both'
      END AS source_flag
    FROM legacy_policies a
    FULL OUTER JOIN crm_policies b ON a.policy_id = b.policy_id
    WHERE a.status != b.status OR a.policy_id IS NULL OR b.policy_id IS NULL;
    
    
    

  3. **Escalate the definition question** — brought both data owners together with the business definition in writing. Within one meeting, the discrepancy was explained: 3,200 policies were in a "pending renewal" state that one system counted as active and the other didn't.
  4. **Document the agreed definition** — updated the STTM with the aligned business rule and made the BigQuery model the single authoritative source going forward.



**Result:** Discrepancy resolved without either system being "wrong." The new BigQuery model used the agreed business definition and became the number that finance, actuarial, and compliance all reported from. No more conflicting numbers.

* * *

**Q43. You are given a dataset with 50 columns and asked to build a report by end of day. Where do you start?**

**Answer:**

**Situation:** Common scenario in fast-moving analytics environments — asked to deliver quickly on an unfamiliar dataset.

**Task:** Deliver a useful, accurate report within a tight deadline without cutting corners on data integrity.

**Action:**

**Step 1 — Clarify the business question first (15 minutes)** Before touching the data: what decision does this report support? Who is the audience? What are the 3–5 most important things they need to know? This prevents building the wrong thing fast.

**Step 2 — Profile the data (20 minutes)**
    
    
    import pandas as pd
    
    df = pd.read_csv("dataset.csv")
    print(df.shape)           # rows × columns
    print(df.dtypes)          # data types
    print(df.isnull().sum())  # null rates
    print(df.describe())      # numeric distributions
    print(df.nunique())       # cardinality per column
    
    
    

**Step 3 — Identify the relevant columns** With the business question in mind, identify the 5–10 columns that actually answer it. The other 40 are noise for now.

**Step 4 — Build the core metric first, then add context** Deliver the primary answer (e.g., total revenue by region) before adding trend lines, breakdowns, and comparisons. If time runs out, the core is always ready.

**Step 5 — Validate before sharing** Spot-check 3–5 individual rows end-to-end. Sense-check totals against any known benchmarks. Never share without a sanity check.

**Result (applied at Google):** Using this approach, I delivered an ad-hoc pipeline latency analysis within 4 hours that was used directly in a senior R&D review — accurate, focused, and without the noise of the 30 irrelevant pipeline columns in the raw dataset.

* * *

**Q44. How would you migrate a legacy on-premises data warehouse to the cloud?**

**Answer:**

**Situation:** At AXA, the legacy on-premises SQL Server data warehouse (10 years old, 2TB, 300+ tables) needed to be migrated to Microsoft Fabric to enable modern analytics and reduce infrastructure costs.

**Task:** Plan and deliver the migration with zero data loss and minimal disruption to existing reporting.

**Action — phased approach:**

**Phase 1: Assessment (weeks 1–2)**

  * Catalogued all 300+ tables: active vs unused, dependencies, data volumes, refresh frequencies.
  * Identified 40 tables that drove 90% of reporting — these got migrated first.
  * Documented all ETL jobs, transformation logic, and scheduled reports.



**Phase 2: Foundation (weeks 3–4)**

  * Set up Microsoft Fabric workspace, OneLake structure, and access controls.
  * Migrated Bronze layer — raw data dumps from SQL Server into OneLake via ADF copy pipelines.



**Phase 3: Parallel run (weeks 5–8)**

  * Rebuilt Silver and Gold layers in Fabric, matching the logic of the legacy SQL Server transformations.
  * Ran both systems simultaneously, comparing output daily via automated reconciliation queries.
  * Fixed discrepancies iteratively — most were timezone handling and NULL treatment differences.



**Phase 4: Cutover**

  * Business-hours freeze on legacy writes → final delta loaded → Power BI reports reconnected to Fabric endpoints.
  * Legacy system kept in read-only mode for 30 days as a fallback.



**Result:** Zero data loss. Power BI dashboard load times improved from 8 seconds (SQL Server DirectQuery) to sub-second (Direct Lake). Infrastructure costs reduced by ~60%. 20M+ records migrated successfully.

* * *

**Q45. How do you approach building a KPI framework for a business from scratch?**

**Answer:**

**Situation:** At J&J, after the BigQuery consolidation, there was no agreed definition of what "good performance" looked like for the claims operation. Every team had their own version of core metrics.

**Task:** Build a single KPI framework that all teams — actuarial, compliance, operations — would trust and use.

**Action:**

  1. **Discovery workshops** — ran structured sessions with each team to surface their top 5 metrics and the business decisions they supported.
  2. **Identified overlaps and conflicts** — mapped where teams used the same metric name for different calculations (e.g., "claim cycle time" measured from 3 different starting events across 3 teams).
  3. **Agreed definitions in plain language** — for each KPI: what it measures, how it's calculated, what data source feeds it, what the target/threshold is, and who owns it.
  4. **Built a tiered KPI hierarchy:**
     * **L1 (Executive):** 5 metrics — total claims volume, average settlement time, fraud rate, reserve accuracy, cost per claim
     * **L2 (Operational):** 15 metrics — breakdowns of L1 by region, claim type, handler
     * **L3 (Analytical):** Drill-down diagnostics for specific investigations
  5. **Implemented in BigQuery** — each KPI as a certified view, versioned, with change history.
  6. **Published to a single Power BI governance dashboard** — one source, one number, for all teams.



**Result:** "Which dashboard should I use?" questions dropped to zero within 2 months. The KPI framework was presented at a J&J global analytics summit as a model for other regions to adopt.

* * *

## 🔵 Round 5 – Culture Fit, Values & Role-Specific

* * *

**Q46. What does "data-driven decision making" mean to you in practice?**

**Answer:**

**Situation:** It's a phrase used everywhere, but the reality varies enormously across organisations.

**Task/Reflection:** Share what it genuinely looks like in practice based on my experience.

**Action/Answer:**

To me, data-driven decision making means three specific things:

  1. **The question comes before the data** — the best analyses start with "what decision are we trying to make?" not "what does our data say?" At J&J, the $2M saving only emerged because I started with "where are we spending more than we should?" — not by exploring data aimlessly.
  2. **Data informs, people decide** — I've seen organisations mistake data for a decision-making replacement. Data reduces uncertainty; it doesn't eliminate judgment. I always present findings with confidence intervals, assumptions, and limitations — never as the single truth.
  3. **The loop closes** — a decision made with data should be tracked against the outcome. At Travelers, the fraud detection pipeline wasn't "done" when it went live — it was done when we measured whether the fraud flag rate improved and what happened to false positives. Closing the loop is what separates analytics from reporting.



**Result:** This philosophy is why I document assumptions, version KPI definitions, and invest in monitoring. The insight is only as valuable as the decision it changes and the outcome it improves.

* * *

**Q47. How do you handle working with imperfect or incomplete data?**

**Answer:**

**Situation:** In every project I've worked on — from AXA claims to Google Street View — the data has never been perfectly clean or complete at the start.

**Task:** Deliver reliable analysis despite data imperfections, without overstating confidence.

**Action:**

My approach has three components:

  1. **Quantify the imperfection** — before any analysis, I measure what's missing and how much it matters. A 2% null rate in a low-priority field is different from a 2% null rate in the primary key. I document both.
  2. **Make assumptions explicit** — any imputation, exclusion, or approximation I make is documented and disclosed in the output. I never silently clean data and present results as if the original data was perfect.
  3. **Assess the impact on conclusions** — if I exclude records with missing values, I run a sensitivity check: does the conclusion change if I include them with a conservative assumption? If yes, I flag it as a limitation. If no, the conclusion is robust.


    
    
    # Example: Sensitivity check
    # Primary analysis: exclude nulls
    result_excl = df.dropna(subset=['loss_amount'])['loss_amount'].mean()
    
    # Sensitivity: fill nulls with conservative lower bound
    result_low = df['loss_amount'].fillna(df['loss_amount'].quantile(0.25)).mean()
    
    print(f"Excluding nulls: {result_excl:.2f}")
    print(f"Conservative fill: {result_low:.2f}")
    print(f"Difference: {abs(result_excl - result_low):.2f} ({abs(result_excl - result_low)/result_excl*100:.1f}%)")
    
    
    

**Result:** Stakeholders consistently trust my analysis because they know I won't hide data quality problems. At AXA, the transparent DQ report I produced as part of the migration actually accelerated the source system cleanup — because the business owners could see the impact in plain numbers for the first time.

* * *

**Q48. Describe your experience with agile delivery in a data context.**

**Answer:**

**Situation:** At Cognizant across multiple engagements, I worked within agile delivery frameworks — 2-week sprints, Jira for backlog management, and daily standups with cross-functional teams including engineers, analysts, and business stakeholders.

**Task:** Adapt traditional agile practices to the specific rhythms of data and analytics work.

**Action:**

Data work has unique characteristics that require adapting agile:

  * **Stories are harder to size** — data investigations often expand unexpectedly. I learned to timebox exploration (e.g., "2 hours of root cause analysis, then we decide if it's a sprint story or a spike").
  * **"Done" needs a data-specific definition** — I established a Definition of Done for analytics stories: code reviewed, data validated against source, documented in STTM or Confluence, and a business stakeholder has confirmed the output makes sense.
  * **Demos with real data** — in sprint reviews, I always showed the actual dashboard or query output, not slides. Stakeholders give much better feedback when they see the real thing.
  * **Tracking tech debt** — data pipelines accumulate tech debt fast. I maintained a dedicated "data debt" section of the backlog and ensured at least 20% of each sprint capacity was allocated to it.



**Result:** In the J&J engagement, sprint velocity stabilised within 6 weeks after introducing these practices. Story acceptance rate at review improved from 70% to 95% — because business stakeholders were involved earlier and the Definition of Done was explicit.

* * *

**Q49. How do you ensure your dashboards are actually used and not just built?**

**Answer:**

**Situation:** At Google Street View, I had seen dashboards built by previous teams that were technically impressive but had zero adoption — nobody used them because they didn't match how the teams actually worked.

**Task:** Build dashboards for the R&D and operations teams that became genuinely embedded in their workflow.

**Action:**

  1. **Start with the workflow, not the data** — I shadowed the operations team for a day before writing a line of SQL or designing a chart. I wanted to see which questions they asked each other, what they Googled, and where they felt blind.
  2. **Prototype in 48 hours** — built a rough first version quickly and got it in front of users. Early feedback (before I'd invested weeks) was far more honest and useful.
  3. **Design for the user's resolution** — the dashboards were used on large screens in the operations room, so I used larger fonts, high-contrast colours (monochrome at their request), and minimal text.
  4. **Embedded in the workflow** — arranged for the dashboard to be the default browser home page on ops workstations. Removed the friction of having to navigate to it.
  5. **Feedback loop** — added a "Report an issue" button on every dashboard page linking to a JIRA ticket template. This made it easy for users to flag problems without emailing me.



**Result:** The Street View dashboard went from zero adoption to being viewed 40+ times per day within 6 weeks. Leadership cited it in two quarterly business reviews. The SSD Life Cycle Dashboard I built was explicitly requested by a sister team in a different region.

* * *

**Q50. Tell me about a time you used data to influence a strategic decision.**

**Answer:**

**Situation:** At J&J Global Services, there was ongoing debate about whether to renew a premium third-party claims processing contract that was due for renegotiation. The account team believed it was essential; finance believed it was overpriced but couldn't quantify the gap.

**Task:** Provide a data-based perspective to inform the negotiation.

**Action:**

  * Pulled 3 years of claims routing data to understand exactly which claim types went through the third-party vendor vs the internal pathway.
  * Discovered that 38% of claims sent to the premium vendor met the eligibility criteria for the internal pathway — they were being routed externally due to a stale business rule.
  * Calculated the cost differential per claim type and annualised the over-spend: $2M+.
  * Modelled two scenarios: (a) correct the routing rule — eliminates most of the over-spend; (b) renegotiate the contract using the analysis as leverage — partial saving but maintains vendor relationship.
  * Presented both options with financial projections to the VP and CFO.



**Result:** The CFO chose option (a) — the routing fix. Delivered within 6 weeks. The analysis directly drove a $2M+ annual saving. The vendor contract was subsequently renegotiated from a position of strength, producing an additional 15% cost reduction on the retained volume.

* * *

## 🟣 Round 5 – Questions for You to Ask the Interviewer

* * *

**Q51–60: Smart questions to ask your interviewer**

These demonstrate strategic thinking, genuine interest, and seniority:

* * *

**Q51.** _"What does the data estate currently look like — how many source systems are analysts working with, and what are the biggest data quality or trust challenges the team faces today?"_

> Shows you understand that data quality is the foundation, and signals you've thought about real-world complexity.

* * *

**Q52.** _"How do business stakeholders currently engage with data — are they self-serve on dashboards, or do most requests come through the analytics team?"_

> Reveals the maturity of the data culture and where you'll spend your time.

* * *

**Q53.** _"What does success look like for this role in the first 90 days versus the first year?"_

> Shows you're thinking about impact and delivery, not just the job description.

* * *

**Q54.** _"What are the biggest technical bottlenecks in the current data pipeline or reporting infrastructure that you're hoping this hire will help address?"_

> Positions you as someone who solves problems, not just executes tasks.

* * *

**Q55.** _"How does the data team collaborate with engineering, product, and the business — what's the model and how has it evolved?"_

> Signals you understand that data work is cross-functional and relationship-dependent.

* * *

**Q56.** _"What cloud data platforms and BI tools are in the stack today, and are there any architectural changes being evaluated in the near term?"_

> Practical and forward-looking — shows you're thinking about where you'll invest your learning.

* * *

**Q57.** _"Can you tell me about a recent analysis or dashboard that had a significant impact on a business decision? What made it successful?"_

> Invites the interviewer to share a success story, revealing what "good" looks like in their context.

* * *

**Q58.** _"How does the team approach data governance and documentation — is there a shared standard, or is it more ad hoc?"_

> Signals that you care about sustainability and quality, not just delivery speed.

* * *

**Q59.** _"What's the biggest data challenge facing the business in the next 12 months that you'd expect a Senior Data Analyst in this role to contribute to?"_

> Elevates the conversation from job description to strategic contribution.

* * *

**Q60.** _"What do the most effective people in this team have in common — what separates good from great here?"_

> Helps you understand the culture and shows you're thinking about how to perform at the top level, not just meet expectations.

* * *
