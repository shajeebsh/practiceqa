---
title: "Aptitude50-Teradata-tech"
date: 2026-04-20 10:01:54 
author: shajeebhameed
layout: page
slug: aptitude50-teradata-tech
status: publish
---

**Role & responsibilities**

  * Should be good at requirement gathering, documentation, presentation, and interaction with business stakeholders.
  * Should be capable to interact with developers and testers to build complex logic in EDW.
  * Technical knowledge of data models and database design.
  * Good knowledge of Teradata/SQL and writing little complex scripts proficiently
  * Develop and implement databases, data collection systems, data analytics and other strategies that optimize statistical efficiency and quality.
  * Working with different stake holders to identify process improvement opportunities, propose system modifications, and devise data strategies.
  * Stakeholder engagement to develop existing artefacts in a clear interpretation of Data usage across each business and function
  * Should be capable of doing Business Acceptance Test/User acceptance test and System Integration test. Will be required to transform the business requirement into scripts.
  * Should be capable of doing AS-IS, TO-BE and Gap analysis.
  * Having worked in corporate banking/finance in Data space in the past will be helpful.



**Experience & Skills:**

  * Proven working experience as a Data Analyst or Business Data Analyst for at least 5 years.
  * Technical proficiency regarding database design development, data models, techniques for data mining, and segmentation.
  * Experience in handling reporting packages like Business Objects, programming (JavaScript, XML, or ETL frameworks), databases
  * Strong analytical skills with the ability to collect, organize, analyze, and disseminate significant amounts of information with attention to detail and accuracy
  * Experience of working in both Agile & waterfall SDLCs
  * Strong communication skills with the ability to communicate technical issues and strategy to both technical and non-technical audiences at senior levels within the bank. Interpersonal skills - collaboration, facilitation, and negotiation skills



* * *

## 50 Technical Interview Questions & Answers

### Senior / Business Data Analyst – EDW / Banking / SQL Focus

_Tailored to the job specification and profile_

* * *

## Section 1: SQL & Query Optimisation (Q1–12)

* * *

**Q1. What is the difference between`WHERE` and `HAVING` in SQL?**

`WHERE` filters rows **before** aggregation; `HAVING` filters **after** aggregation.
    
    
    -- WHERE: filters raw rows first
    SELECT department, COUNT(*) AS headcount
    FROM employees
    WHERE status = 'Active'
    GROUP BY department
    HAVING COUNT(*) > 10;
    -- HAVING: filters the aggregated result (departments with more than 10 active staff)
    
    
    

* * *

**Q2. Explain the difference between`RANK()`, `DENSE_RANK()`, and `ROW_NUMBER()`.**

Function| Behaviour on ties| Gaps in sequence?  
---|---|---  
`ROW_NUMBER()`| Assigns unique sequential numbers| No  
`RANK()`| Same rank for ties| Yes (skips numbers)  
`DENSE_RANK()`| Same rank for ties| No (no gaps)  
      
    
    SELECT trade_id, amount,
      ROW_NUMBER() OVER (ORDER BY amount DESC) AS row_num,
      RANK()       OVER (ORDER BY amount DESC) AS rnk,
      DENSE_RANK() OVER (ORDER BY amount DESC) AS dense_rnk
    FROM trades;
    
    
    

* * *

**Q3. How would you identify and remove duplicate records from a table without deleting distinct ones?**

Use a CTE with `ROW_NUMBER()` partitioned by the business key:
    
    
    WITH CTE AS (
      SELECT *,
        ROW_NUMBER() OVER (PARTITION BY trade_id, trade_date ORDER BY load_ts DESC) AS rn
      FROM trades_staging
    )
    DELETE FROM CTE WHERE rn > 1;
    
    
    

Keep `rn = 1` (most recent load) and delete all others.

* * *

**Q4. What is a correlated subquery? Give a real-world example.**

A correlated subquery references a column from the **outer query** and re-executes for each outer row.
    
    
    -- Find customers whose latest trade amount exceeds their own average
    SELECT c.customer_id, c.name
    FROM customers c
    WHERE (
      SELECT MAX(t.amount)
      FROM trades t
      WHERE t.customer_id = c.customer_id
    ) > (
      SELECT AVG(t2.amount)
      FROM trades t2
      WHERE t2.customer_id = c.customer_id
    );
    
    
    

Correlated subqueries can be expensive — consider rewriting with window functions for large datasets.

* * *

**Q5. What is the difference between a clustered and non-clustered index?**

  * **Clustered index** : physically sorts the table data on disk by the index key. Only one per table. Best on columns used in `ORDER BY` and range queries (e.g., `trade_date`).
  * **Non-clustered index** : a separate structure pointing to row locations. Multiple allowed. Best for columns frequently used in `WHERE` or `JOIN` conditions.



_In the LSEG FX trading project, clustered indexes on`trade_date` and non-clustered on `account_id` gave millisecond-level retrieval on high-frequency transaction data._

* * *

**Q6. How do you optimise a slow-running SQL query?**

  1. **Check the execution plan** — identify table scans, hash joins, missing index warnings.
  2. **Add/tune indexes** — covering indexes for frequently filtered columns.
  3. **Avoid`SELECT *`** — fetch only required columns.
  4. **Replace correlated subqueries** with JOINs or CTEs.
  5. **Use set-based operations** instead of cursors/loops.
  6. **Partition large tables** — especially for date-range queries.
  7. **Update statistics** — stale stats lead to poor plan choices.
  8. **Filter early** — apply `WHERE` before joining to reduce row sets.



* * *

**Q7. What is a CTE and how does it differ from a subquery?**

A Common Table Expression (CTE) is a named temporary result set defined with `WITH`. It:

  * Improves **readability** and allows **self-referencing** (recursive CTEs).
  * Can be **referenced multiple times** in the same query (unlike a subquery).
  * Does not improve performance by itself — the engine may still materialise or inline it.


    
    
    WITH monthly_totals AS (
      SELECT account_id, DATE_TRUNC('month', trade_date) AS month, SUM(amount) AS total
      FROM trades
      GROUP BY 1, 2
    )
    SELECT account_id, month, total,
      LAG(total) OVER (PARTITION BY account_id ORDER BY month) AS prev_month
    FROM monthly_totals;
    
    
    

* * *

**Q8. Explain`INNER JOIN`, `LEFT JOIN`, `FULL OUTER JOIN`, and `CROSS JOIN`.**

Join Type| Returns  
---|---  
`INNER JOIN`| Only matching rows in both tables  
`LEFT JOIN`| All rows from left + matched rows from right (NULLs where no match)  
`RIGHT JOIN`| All rows from right + matched rows from left  
`FULL OUTER JOIN`| All rows from both tables; NULLs where no match on either side  
`CROSS JOIN`| Cartesian product — every row in left × every row in right  
  
* * *

**Q9. What is a recursive CTE? Give a use case in banking.**

A recursive CTE calls itself to traverse hierarchical data.
    
    
    -- Traverse a corporate hierarchy (subsidiary → parent chain)
    WITH RECURSIVE org_hierarchy AS (
      SELECT entity_id, parent_id, entity_name, 1 AS level
      FROM entities WHERE parent_id IS NULL  -- root
    
      UNION ALL
    
      SELECT e.entity_id, e.parent_id, e.entity_name, h.level + 1
      FROM entities e
      INNER JOIN org_hierarchy h ON e.parent_id = h.entity_id
    )
    SELECT * FROM org_hierarchy ORDER BY level;
    
    
    

Useful for credit risk consolidation where exposure rolls up through legal entity trees.

* * *

**Q10. How would you detect data anomalies in a large transaction table using SQL?**
    
    
    -- 1. Null checks
    SELECT COUNT(*) AS nulls FROM trades WHERE amount IS NULL OR account_id IS NULL;
    
    -- 2. Outlier detection (values beyond 3 standard deviations)
    SELECT trade_id, amount
    FROM trades
    WHERE ABS(amount - AVG(amount) OVER()) > 3 * STDDEV(amount) OVER();
    
    -- 3. Duplicate detection
    SELECT trade_id, COUNT(*) FROM trades GROUP BY trade_id HAVING COUNT(*) > 1;
    
    -- 4. Date anomalies (future-dated trades)
    SELECT * FROM trades WHERE trade_date > CURRENT_DATE;
    
    -- 5. Volume spike detection
    SELECT trade_date, COUNT(*) AS vol
    FROM trades
    GROUP BY trade_date
    HAVING COUNT(*) > 2 * AVG(COUNT(*)) OVER (ORDER BY trade_date ROWS BETWEEN 6 PRECEDING AND 1 PRECEDING);
    
    
    

* * *

**Q11. What is the difference between`UNION` and `UNION ALL`?**

  * `UNION` removes duplicates (performs a sort/hash to deduplicate) — slower.
  * `UNION ALL` keeps all rows including duplicates — faster.



Always prefer `UNION ALL` when you know the datasets are distinct, or when duplicates are acceptable, to avoid unnecessary overhead.

* * *

**Q12. How would you calculate a running total and a 7-day moving average in SQL?**
    
    
    SELECT
      trade_date,
      daily_volume,
      SUM(daily_volume) OVER (ORDER BY trade_date) AS running_total,
      AVG(daily_volume) OVER (ORDER BY trade_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7d
    FROM daily_trade_summary
    ORDER BY trade_date;
    
    
    

* * *

## Section 2: Data Modelling & EDW Design (Q13–20)

* * *

**Q13. What is the difference between a Star Schema and a Snowflake Schema?**

Aspect| Star Schema| Snowflake Schema  
---|---|---  
Dimension tables| Denormalised (flat)| Normalised (split into sub-dimensions)  
Query performance| Faster (fewer joins)| Slower (more joins)  
Storage| Higher redundancy| Lower redundancy  
Maintenance| Simpler| More complex  
  
In a banking EDW, a Star Schema is typically preferred for BI performance. Snowflake Schema suits scenarios where dimension data changes frequently and storage efficiency matters.

* * *

**Q14. What is a Slowly Changing Dimension (SCD)? What are the types?**

An SCD handles changes to dimension attributes over time.

Type| Behaviour| Use case  
---|---|---  
Type 1| Overwrite old value| Non-historical corrections (e.g., typo fix)  
Type 2| Add new row, track history with `effective_from`/`effective_to`| Customer address history, account status  
Type 3| Add `previous_value` column| Track one prior state only  
  
_Type 2 is most common in banking to preserve point-in-time accuracy for regulatory reporting._

* * *

**Q15. What is a surrogate key and why is it used in a data warehouse?**

A surrogate key is a system-generated, meaningless integer (e.g., `customer_key BIGINT IDENTITY`) used as the primary key in dimension tables.

**Why:** Source system keys can change, merge, or conflict across systems. Surrogate keys decouple the warehouse from source volatility and enable SCD Type 2 versioning.

* * *

**Q16. Explain the Bronze-Silver-Gold (Medallion) architecture.**

A layered data warehouse pattern:

Layer| Description  
---|---  
**Bronze (Raw)**|  Ingested as-is from source. No transformations. Preserves full audit trail.  
**Silver (Cleansed)**|  Deduplicated, standardised, validated. Business rules applied.  
**Gold (Curated)**|  Aggregated, business-ready. Powers BI dashboards and reporting.  
  
_Used in the Travelers Insurance claims warehouse — raw Medicare/Medicaid claims in Bronze, validated claims in Silver, fraud-flagged aggregates in Gold._

* * *

**Q17. What is a fact table and what types of facts exist?**

A fact table stores measurable business events/metrics. Types of facts:

  * **Additive** : Can be summed across all dimensions (e.g., sales amount).
  * **Semi-additive** : Can be summed across some dimensions but not all (e.g., account balance — summing across time is meaningless).
  * **Non-additive** : Cannot be summed at all (e.g., ratios, percentages, exchange rates).



* * *

**Q18. What is CDC (Change Data Capture) and how does it work?**

CDC captures row-level changes (inserts, updates, deletes) in source systems and streams them to the target, avoiding full table refreshes.

**Mechanisms:**

  * **Timestamp-based** : Query rows where `modified_date > last_run`.
  * **Log-based** : Read database transaction logs (e.g., SQL Server CDC, Oracle LogMiner).
  * **Trigger-based** : Database triggers write changes to a staging table.



_In the Travelers project, CDC pipelines cut claims data latency from 24 hours to under 15 minutes for the fraud detection team._

* * *

**Q19. What is data normalisation? What are 1NF, 2NF, and 3NF?**

Normalisation eliminates redundancy and ensures data integrity.

Form| Rule  
---|---  
**1NF**|  Atomic values; no repeating groups; each column holds a single value  
**2NF**|  1NF + no partial dependency (every non-key column depends on the **whole** primary key)  
**3NF**|  2NF + no transitive dependency (non-key columns depend only on the primary key, not on other non-key columns)  
  
Data warehouses are often intentionally **denormalised** for query performance.

* * *

**Q20. How would you design an EDW for a corporate banking loan portfolio?**

  1. **Identify subject areas** : Customers, Accounts, Loans, Repayments, Collateral, Counterparties.
  2. **Define grain** : one row per loan per reporting date.
  3. **Fact tables** : `fact_loan_balance`, `fact_repayment`, `fact_exposure`.
  4. **Dimension tables** : `dim_customer`, `dim_account`, `dim_product`, `dim_geography`, `dim_date`.
  5. **SCD Type 2** on `dim_customer` and `dim_product` for historical tracking.
  6. **Staging layer** : raw ingestion from core banking system.
  7. **Data quality checks** : referential integrity, balance reconciliation against source.
  8. **Governance** : data lineage documentation, RBAC, audit logging.



* * *

## Section 3: ETL/ELT Pipeline Design (Q21–26)

* * *

**Q21. What is the difference between ETL and ELT?**

Aspect| ETL| ELT  
---|---|---  
Order| Extract → Transform → Load| Extract → Load → Transform  
Transform location| Dedicated ETL server / middleware| Inside the target data warehouse  
Best for| Legacy systems, complex transformations| Cloud DWH (BigQuery, etc.) with scalable compute  
Latency| Higher| Lower (raw data lands first)  
  
* * *

**Q22. What data quality checks do you apply in a pipeline?**

  1. **Completeness** : NULL checks on mandatory fields.
  2. **Uniqueness** : Duplicate detection on business keys.
  3. **Referential integrity** : Orphan records (FK mismatches).
  4. **Validity** : Domain checks (e.g., amount > 0, valid currency codes).
  5. **Consistency** : Cross-system reconciliation (source count = target count).
  6. **Timeliness** : Freshness checks (max load timestamp within expected window).
  7. **Format checks** : Date formats, string lengths, numeric precision.



_At LSEG, 50+ automated validation checks enforced consistency between Oracle source and SQL Server target during live FX trade execution._

* * *

**Q23. What is an STTM and what does it contain?**

An **Source-to-Target Mapping (STTM)** document defines how data moves from source to target in an ETL pipeline. It typically contains:

  * Source system, table, and column names
  * Target table and column names
  * Data type mappings
  * Transformation rules (e.g., concatenation, lookup, derivation)
  * Business rules and validation logic
  * Default values and NULL handling
  * Notes on SCD handling



_At J &J Global Services, STTMs were created for 200+ business attributes covering underwriting and claims data._

* * *

**Q24. How would you handle a failed ETL job in production?**

  1. **Alert immediately** via monitoring/alerting tool (PagerDuty, Azure Monitor).
  2. **Identify failure point** — check logs for the exact step and error.
  3. **Assess impact** — which downstream reports/dashboards are affected?
  4. **Communicate** to stakeholders with ETA.
  5. **Rollback or re-run** — for idempotent pipelines, re-run from the failure point; for non-idempotent, roll back partial loads first.
  6. **Root cause analysis** — fix and add a preventive check.
  7. **Post-mortem documentation** — update runbook.



* * *

**Q25. What is idempotency in ETL pipelines and why does it matter?**

An idempotent pipeline produces the same result regardless of how many times it is run. This means re-running a failed job does not create duplicates or corrupt data.

**Techniques to achieve idempotency:**

  * **MERGE (UPSERT)** instead of INSERT — update if exists, insert if new.
  * **Partition-based overwrite** — delete/replace the relevant partition before writing.
  * **Staging tables** — load to staging first, then MERGE into target.



* * *

**Q26. How would you design a data pipeline for real-time fraud detection?**

  1. **Ingest** : Stream transactions via Kafka or event hub.
  2. **Validate** : Apply real-time schema and business rule checks.
  3. **Enrich** : Join with customer profile and historical behaviour from a fast-access store.
  4. **Score** : Apply fraud scoring logic (rules engine or ML model output).
  5. **Alert** : Flag high-risk transactions; push to alert queue.
  6. **Persist** : Write all transactions + scores to EDW for retrospective analysis.
  7. **Monitor** : Latency SLAs, throughput dashboards, anomaly alerting.



_In the Travelers project, CDC-based pipelines reduced claims data latency to under 15 minutes, enabling near-real-time fraud flagging._

* * *

## Section 4: Business Analysis & Requirements (Q27–33)

* * *

**Q27. How do you approach requirement gathering for a new data analytics project?**

  1. **Stakeholder identification** — map all business and technical stakeholders.
  2. **Initial workshops** — understand business objectives, pain points, current state.
  3. **AS-IS analysis** — document existing data flows, systems, and reports.
  4. **TO-BE design** — define the desired future state with stakeholders.
  5. **Gap analysis** — identify what needs to change: data, process, technology.
  6. **Document requirements** — functional (what the system should do) and non-functional (performance, security, SLA).
  7. **Sign-off** — validate with stakeholders before development begins.
  8. **Traceability** — maintain a requirements traceability matrix through delivery.



* * *

**Q28. What is AS-IS / TO-BE / Gap analysis? Walk me through an example.**

  * **AS-IS** : Documents the current state — existing systems, data flows, processes, and pain points.
  * **TO-BE** : Defines the desired future state — improved processes, new systems, target architecture.
  * **Gap** : Identifies what must change to move from AS-IS to TO-BE: missing data, process redesign, new integrations, tooling.



**Example (AXA Insurance migration):**

  * AS-IS: Claims data in 3 legacy systems with 24-hour batch updates, no lineage, manual reconciliation.
  * TO-BE: Unified EDW with near-real-time data, automated quality checks, full lineage tracking.
  * Gap: No CDC capability in legacy systems; no data dictionary; manual transformation logic undocumented.



* * *

**Q29. What is UAT and how do you manage it as a data analyst?**

**User Acceptance Testing (UAT)** validates that a delivered solution meets business requirements before go-live.

**As a data analyst:**

  1. Translate business requirements into structured test cases/scripts.
  2. Define expected outputs for each test scenario.
  3. Co-ordinate with business users to execute tests.
  4. Log defects with clear reproduction steps, expected vs actual output.
  5. Track defect resolution with developers.
  6. Obtain formal sign-off from business stakeholders.
  7. Document test evidence for audit trail.



* * *

**Q30. What is the difference between BAT, UAT, and SIT?**

Test Type| Who runs it| What it validates  
---|---|---  
**SIT** (System Integration Test)| Technical team| End-to-end data flow across integrated systems; interfaces work correctly  
**UAT** (User Acceptance Test)| Business users (analyst supports)| System meets business requirements; outputs are correct and usable  
**BAT** (Business Acceptance Test)| Business owners| Final sign-off that solution delivers business value and is fit for purpose  
  
* * *

**Q31. How do you handle conflicting requirements from different stakeholders?**

  1. **Document all requirements** explicitly — don't rely on verbal agreement.
  2. **Identify the conflict** clearly — present both requirements side by side.
  3. **Facilitate a joint session** — bring conflicting stakeholders to the table.
  4. **Prioritise by business impact** — which requirement delivers more value or is a regulatory must?
  5. **Escalate if needed** — involve a project sponsor or steering committee.
  6. **Record the decision** — document who decided, why, and the trade-offs accepted.



* * *

**Q32. How do you communicate complex data findings to non-technical stakeholders?**

  * **Lead with the insight, not the method** — "Claims fraud increased 18% in Q3" before explaining how.
  * **Use visualisations** — charts and dashboards over raw tables.
  * **Avoid jargon** — translate technical terms (e.g., "CDC pipeline" → "real-time data feed").
  * **Use analogies** — relate data concepts to business processes they already know.
  * **Tell a story** — context → finding → implication → recommendation.
  * **Confirm understanding** — invite questions; summarise decisions in writing.



* * *

**Q33. What is a data dictionary and why is it important?**

A data dictionary (or business glossary) is a centralised catalogue of all data assets, including:

  * Field names, descriptions, and data types.
  * Business definitions and ownership.
  * Source system, transformation rules.
  * Valid values and constraints.



**Why it matters:** Ensures consistent interpretation of data across teams; critical for onboarding, governance, audit, and avoiding conflicting KPI definitions across departments.

* * *

## Section 5: Data Quality & Governance (Q34–38)

* * *

**Q34. What are the key dimensions of data quality?**

Dimension| Definition  
---|---  
**Completeness**|  All required data is present  
**Accuracy**|  Data correctly represents reality  
**Consistency**|  Same data is consistent across systems  
**Timeliness**|  Data is available when needed  
**Uniqueness**|  No unintended duplicates  
**Validity**|  Data conforms to defined formats and rules  
**Integrity**|  Relationships between data are intact  
  
* * *

**Q35. How would you implement a data quality monitoring framework?**

  1. **Define rules** : Work with business to document quality thresholds (e.g., NULL rate < 0.1%, duplicate rate = 0%).
  2. **Automate checks** : Build SQL-based validation queries or use a DQ tool.
  3. **Schedule** : Run checks after each pipeline load.
  4. **Alert** : Notify data owners when thresholds are breached.
  5. **Dashboard** : Publish DQ score trends for each domain.
  6. **Remediation workflow** : Log failures, assign owners, track resolution.
  7. **Root cause tracking** : Link recurring failures to upstream source issues.



* * *

**Q36. What is data lineage and why does it matter in banking?**

Data lineage tracks the origin, movement, transformations, and consumption of data across systems — a full audit trail from source to report.

**Why it matters in banking:**

  * **Regulatory compliance** : BCBS 239, GDPR, MiFID II require firms to demonstrate data accuracy and traceability.
  * **Impact analysis** : Understand what downstream reports are affected by an upstream change.
  * **Incident investigation** : Quickly trace incorrect figures back to their source.
  * **Audit readiness** : Provide regulators with documented evidence of data flows.



* * *

**Q37. What is RBAC in the context of data governance?**

**Role-Based Access Control (RBAC)** restricts data access based on a user's role within the organisation.

  * Principle of least privilege: users access only what they need.
  * Applied at database, schema, table, and column level.
  * In banking: traders see trading data; actuaries see claims data; executives see aggregated dashboards — each role has a distinct permission set.
  * Combined with **Dynamic Data Masking** to show masked values (e.g., `****1234`) to lower-privileged roles without changing underlying data.



* * *

**Q38. How do you ensure reconciliation between source and target systems?**

  1. **Row count reconciliation** : Source count = Target count after each load.
  2. **Checksum/hash reconciliation** : Hash key columns to detect silent data corruption.
  3. **Aggregate reconciliation** : Sum of amounts in source = Sum in target for each partition/date.
  4. **Null/outlier checks** : Compare null rates between source and target.
  5. **Automated alerting** : Any breach of reconciliation thresholds triggers an alert.
  6. **Reconciliation report** : Published daily to data stewards.



_At LSEG, 50+ automated validation checks reduced FX trading reconciliation issues by 85%._

* * *

## Section 6: BI & Reporting (Q39–42)

* * *

**Q39. What is the difference between Direct Query and Import mode in Power BI?**

Mode| How it works| Best for  
---|---|---  
**Import**|  Data copied into Power BI's in-memory model| Fast performance; data up to last refresh  
**Direct Query**|  Every visual queries the source live| Real-time/near-real-time; large datasets that can't be imported  
**Composite**|  Mix of both| Some tables imported, some live  
  
_Direct Lake mode (used in the AXA project) is a hybrid that reads directly from the storage layer without import, combining the speed of Import with the freshness of Direct Query._

* * *

**Q40. What makes a good executive dashboard?**

  1. **Single screen** — key KPIs visible without scrolling.
  2. **Exception-based** — highlight what needs attention, not everything.
  3. **Consistent colour coding** — red/amber/green for status.
  4. **Clear labelling** — no jargon; units explicit (£M, %, bps).
  5. **Drill-down capability** — summary on top, detail accessible.
  6. **Trend context** — vs prior period, vs target.
  7. **Refresh timestamp** — users know how current the data is.
  8. **Actionable** — every metric should link to a business decision.



* * *

**Q41. How do you choose between a bar chart, line chart, and scatter plot?**

Chart Type| Best for  
---|---  
**Bar / Column**|  Comparing discrete categories  
**Line**|  Showing trends over time  
**Scatter**|  Showing correlation between two numeric variables  
**Pie / Donut**|  Part-to-whole (use sparingly; max 5 segments)  
**Heat map**|  Patterns across two categorical dimensions  
**KPI card**|  Single metric with target and trend  
  
* * *

**Q42. What is a Business Objects report and how does it differ from Power BI?**

**Business Objects (SAP BO)** is an enterprise BI suite widely used in banking and finance. Key differences from Power BI:

Aspect| Business Objects| Power BI  
---|---|---  
Primary users| IT/report developers| Business users + IT  
Data access| Universe semantic layer| Dataflow / Direct Query  
Deployment| On-premise (common in banks)| Cloud-native (Microsoft 365)  
Self-service| Limited| Strong  
Typical use| Pixel-perfect regulatory/financial reports| Interactive dashboards  
  
In heavily regulated banking environments, BO is often the approved tool for statutory and regulatory reporting due to its controlled universe layer and audit trail.

* * *

## Section 7: Agile & SDLC (Q43–46)

* * *

**Q43. How does a data analyst contribute in an Agile delivery model?**

  * **Sprint planning** : Estimate and size data-related user stories.
  * **Backlog grooming** : Decompose requirements into deliverable data tasks.
  * **Daily stand-up** : Report progress, flag blockers (e.g., access issues, data unavailability).
  * **Development support** : Clarify requirements for developers; review logic.
  * **Sprint demo** : Present data outputs/dashboards to stakeholders.
  * **Retrospective** : Identify process improvements for the next sprint.
  * **Continuous delivery** : Produce incremental working data products, not big-bang releases.



* * *

**Q44. What is the difference between Agile and Waterfall SDLC? When would you choose each?**

Aspect| Agile| Waterfall  
---|---|---  
Delivery| Iterative sprints| Sequential phases  
Requirements| Evolving| Fixed upfront  
Change tolerance| High| Low  
Client involvement| Continuous| At milestones  
Best for| Exploratory analytics, dashboards, evolving requirements| Regulatory projects with fixed specs, migration projects  
  
_In practice (J &J, Travelers, LSEG), both were used: Agile for dashboard development, Waterfall for formal EDW migrations with regulatory sign-off._

* * *

**Q45. How do you manage scope creep in a data project?**

  1. **Baseline requirements** formally documented and signed off before build begins.
  2. **Change control process** — any new request goes through formal assessment: impact on timeline, cost, and other deliverables.
  3. **Traceability matrix** — map all work items back to signed-off requirements.
  4. **Transparent communication** — proactively flag to the PM when new requests arrive.
  5. **Prioritisation with the business** — new requests may be deferred to the next sprint/phase.



* * *

**Q46. What artifacts does a data analyst produce in a typical project?**

  * Requirements documentation (BRD / FRD)
  * Source-to-Target Mapping (STTM)
  * Data dictionary / business glossary
  * AS-IS / TO-BE process diagrams
  * Gap analysis document
  * Data flow diagrams
  * UAT test scripts and sign-off documentation
  * Data quality reports
  * Executive dashboards and reports
  * Post-implementation review



* * *

## Section 8: Banking & Finance Domain (Q47–50)

* * *

**Q47. What is BCBS 239 and why is it relevant to a data analyst in banking?**

**BCBS 239** (Basel Committee on Banking Supervision Principle 239) sets standards for **Risk Data Aggregation and Risk Reporting** for global systemically important banks (G-SIBs).

Key principles relevant to data analysts:

  * **Data accuracy and integrity** : Risk data must be complete, accurate, and reconciled.
  * **Completeness** : All material risk positions must be captured.
  * **Timeliness** : Risk reports must be produced within agreed timescales.
  * **Adaptability** : Systems must be able to generate ad-hoc reports quickly.



As a data analyst, this means robust data lineage, quality frameworks, and documented transformation logic are regulatory requirements, not optional.

* * *

**Q48. What is an FX trade life cycle and where does data analysis fit?**

An FX trade moves through:

  1. **Trade capture** — deal booked in trading system.
  2. **Validation** — automated checks (counterparty limits, rate plausibility).
  3. **Confirmation** — matched with counterparty.
  4. **Settlement** — exchange of currencies on value date.
  5. **Reconciliation** — nostro/vostro account reconciliation.
  6. **Reporting** — regulatory (MiFID II, EMIR) and management reporting.



Data analysis is critical at **steps 2, 5, and 6** — ensuring data quality, detecting breaks in reconciliation, and producing accurate regulatory and P&L reports.

_This aligns directly with the LSEG/Refinitiv FX trading data engineering role, where 99.9% data consistency was maintained across Oracle and SQL Server._

* * *

**Q49. What is the difference between operational data and analytical data in a banking context?**

Aspect| Operational (OLTP)| Analytical (OLAP / EDW)  
---|---|---  
Purpose| Run the business (transactions)| Analyse the business (insights)  
Update frequency| Real-time / continuous| Batch or near-real-time  
Data scope| Current state| Historical + current  
Query type| Simple, high-frequency| Complex, aggregated  
Optimised for| Write speed| Read/query performance  
Examples| Core banking system, trade booking| EDW, data mart, BI dashboard  
  
* * *

**Q50. How would you approach migrating a legacy banking data warehouse to a modern cloud EDW?**

  1. **Discovery & AS-IS**: Inventory all source systems, existing reports, data volumes, and dependencies.
  2. **Stakeholder alignment** : Agree scope, priorities, and success criteria.
  3. **Target architecture** : Define the cloud EDW design (layered architecture, schemas, naming conventions).
  4. **STTM documentation** : Map all source tables/fields to target with transformation rules.
  5. **ETL/ELT build** : Develop pipelines; implement data quality checks at each layer.
  6. **Data reconciliation** : Validate target data against source — row counts, aggregates, checksums.
  7. **UAT/BAT** : Business users validate reports against the new platform vs legacy.
  8. **Parallel run** : Both systems run simultaneously for a defined period.
  9. **Cutover** : Decommission legacy; go-live on new platform.
  10. **Post-migration review** : Monitor for 30–60 days; address issues; update documentation.



_This approach was applied in the AXA Insurance legacy migration, delivering sub-second Power BI dashboards and near-real-time claims data versus the previous 24-hour batch cycle._

* * *

_End of Interview Q &A Document_ _Generated: April 2026 | Tailored to Senior Data Analyst / Business Data Analyst role – EDW / Banking / SQL_

* * *

* * *

## 50 Technical Aptitude Interview Questions — STAR Format

### Senior / Business Data Analyst – EDW / Banking / SQL Focus

_All answers reference real experience across Google, LSEG, J &J, AXA, Travelers, Cisco_

* * *

> **STAR Key:**
> 
>   * 🔵 **S** ituation — The context and background
>   * 🟡 **T** ask — Your specific responsibility
>   * 🟢 **A** ction — What you did, step by step
>   * 🔴 **R** esult — The measurable outcome
> 


* * *

## Section 1: SQL & Query Optimisation (Q1–10)

* * *

### Q1. Tell me about a time you optimised a slow-running SQL query in a production environment.

🔵 **Situation** At LSEG/Refinitiv, the FX trading platform ran a daily reconciliation query across Oracle and SQL Server. As transaction volumes grew, the query was taking 40+ minutes to complete, causing downstream reporting delays and breaching SLA windows.

🟡 **Task** I was responsible for identifying the performance bottleneck and optimising the query without changing the reconciliation logic or output.

🟢 **Action**

  * Ran the execution plan in SQL Server Management Studio — identified a full table scan on a 50-million-row transaction table due to a missing index on `trade_date` and `account_id`.
  * Replaced a correlated subquery that was executing once per row with a JOIN against a pre-aggregated CTE.
  * Added a composite clustered index on `(trade_date, account_id)` and a covering non-clustered index on `(trade_id, amount, status)`.
  * Rewrote the `SELECT *` to only fetch the 8 columns needed, reducing I/O significantly.
  * Tested in a staging environment before deploying to production.



🔴 **Result** Query execution time dropped from 40 minutes to under 4 minutes — a 90% improvement. Reconciliation reports were consistently delivered within the SLA window, and the optimisation was adopted as a template for 6 other batch jobs.

* * *

### Q2. Describe a situation where you had to write complex SQL to solve a difficult business problem.

🔵 **Situation** At J&J Global Services, actuarial teams needed a report showing rolling 12-month underwriting loss ratios by product line, region, and claims category — but the data was spread across 4 legacy systems with different schemas, date formats, and currency codes.

🟡 **Task** I was tasked with designing a single consolidated SQL query that accurately combined all 4 sources and computed the derived metrics, to be scheduled daily in the BigQuery EDW.

🟢 **Action**

  * Wrote a multi-step SQL pipeline using CTEs: first standardised date formats and currency conversions per source, then unioned the 4 sources into a single claims dataset.
  * Applied window functions (`SUM() OVER`) to compute rolling 12-month earned premiums and incurred losses.
  * Joined against a product dimension to classify claims by product line.
  * Added data quality flags for records with missing or implausible values.
  * Validated row counts and aggregate totals against each source system manually before sign-off.



🔴 **Result** The report replaced a manual Excel process that took actuarial analysts 3 days each month. The automated query ran in under 2 minutes daily, with zero reconciliation issues flagged in the 6 months post-launch.

* * *

### Q3. Give an example of when you used window functions to solve an analytical problem.

🔵 **Situation** At Google (Street View project), the research team needed to understand image processing pipeline performance — specifically, how long each image took to move from collection to publishing, and whether processing time was improving or worsening week-over-week.

🟡 **Task** I needed to build a trend analysis in BigQuery showing week-on-week change in median processing latency, broken down by region and image type.

🟢 **Action**

  * Used `PERCENTILE_CONT(0.5) WITHIN GROUP` (median) partitioned by week and region to compute latency distributions.
  * Applied `LAG()` window function to compare the current week's median against the prior week.
  * Used `AVG() OVER (ORDER BY week ROWS BETWEEN 3 PRECEDING AND CURRENT ROW)` for a 4-week moving average to smooth volatility.
  * Built the output into a Looker Studio dashboard with week-over-week trend lines for global leadership.



🔴 **Result** The analysis identified a specific regional ingestion bottleneck that had been invisible in aggregate metrics. Fixing it drove a **25% reduction in overall processing time** — one of the highest-impact optimisations in that programme.

* * *

### Q4. Tell me about a time you identified and fixed a data quality issue caused by a SQL logic error.

🔵 **Situation** At LSEG, after deploying a new reconciliation module, the finance team reported that the daily FX net position report was showing a 0.3% discrepancy versus the front-office trading system — small in percentage terms, but significant in absolute value given the trade volumes.

🟡 **Task** I was responsible for tracing the root cause and resolving it before the next business day reporting cycle.

🟢 **Action**

  * Ran a diff between source Oracle records and SQL Server target for the affected date.
  * Discovered that a `LEFT JOIN` in the sync logic was inadvertently including cancelled trades that should have been excluded — the `WHERE status != 'CANCELLED'` filter was placed after the join instead of inside a subquery, causing it to be ignored when the right-side rows were NULL.
  * Fixed by moving the filter into a derived table before the join.
  * Added a specific automated check to the validation framework: cancelled trades count in target = 0 after each sync.



🔴 **Result** The discrepancy was resolved before market open. The root cause fix was deployed within 4 hours, and the new validation check has caught 2 similar edge cases in subsequent loads, preventing recurrence.

* * *

### Q5. Describe a time you had to handle duplicate data in a large dataset.

🔵 **Situation** During the J&J Global Services EDW consolidation, when merging 4 legacy insurance data sources into a single BigQuery model, we discovered that roughly 8% of claim records appeared in more than one source system — the same claim referenced by different internal IDs depending on which regional system had processed it first.

🟡 **Task** I needed to deduplicate 150+ million records without losing legitimate multi-touch claim events, and without breaking the downstream actuarial models that depended on claim counts.

🟢 **Action**

  * Worked with the business to define the deduplication key: a composite of `claimant_id`, `incident_date`, `policy_number`, and `claim_type`.
  * Used `ROW_NUMBER() OVER (PARTITION BY [composite key] ORDER BY source_system_priority, load_timestamp DESC)` to rank duplicates, preserving the record from the authoritative source system.
  * Applied the deduplication in the Silver layer of the ETL pipeline, keeping raw duplicates intact in Bronze for audit purposes.
  * Built a daily reconciliation check comparing deduplicated counts against expected volumes from each source.



🔴 **Result** Deduplication removed 12 million duplicate records. Actuarial loss ratio reports became consistent across regions for the first time. The approach was documented as the standard deduplication pattern for all future source integrations.

* * *

### Q6. Tell me about a time you used SQL to investigate an unexpected trend in business data.

🔵 **Situation** At Travelers Insurance, the fraud analytics team flagged that Medicare claims approval rates had spiked 22% in a single week — far outside normal variance — and suspected a data issue rather than a genuine business trend.

🟡 **Task** I was asked to investigate whether the spike was real or an artefact of the data pipeline.

🟢 **Action**

  * Queried the claims fact table, segmenting the spike by load batch, source region, claim type, and adjuster ID.
  * Discovered the spike was concentrated in a single batch loaded on a Tuesday — isolated to one regional office code.
  * Traced back through the pipeline and found that a source file from that region had been re-submitted after a processing error, causing double-counting of approved claims in the Gold layer.
  * Confirmed by comparing raw Bronze counts against Gold layer — a 2:1 ratio for that batch confirmed the duplicate load.



🔴 **Result** The erroneous batch was rolled back and reloaded correctly within 2 hours. Downstream fraud reports were rerun. I added a batch-level duplicate detection check to the pipeline, preventing the same issue from recurring.

* * *

### Q7. Give an example of when you built a complex JOIN across multiple tables to produce a business report.

🔵 **Situation** At Cisco (via Wipro), the operations team needed a daily dashboard showing TelePresence system health across 50,000+ global endpoints — combining device status, conference volumes, failure events, and user satisfaction scores, which lived in 15 separate source tables.

🟡 **Task** I needed to design and build the SQL layer that consolidated all 15 tables into a clean reporting model for the SSRS dashboard.

🟢 **Action**

  * Designed a star schema with a central `fact_conference` table joined to `dim_device`, `dim_location`, `dim_user`, `dim_date`, and `dim_failure_type`.
  * Built ETL stored procedures using T-SQL to load each dimension daily before refreshing the fact table.
  * Used `OUTER APPLY` for the satisfaction scores (not all conferences had a score) rather than a LEFT JOIN, to avoid row multiplication.
  * Added a `data_completeness_flag` column to highlight records where one or more dimension lookups failed — keeping the report accurate without silently dropping rows.



🔴 **Result** The consolidated dashboard replaced 6 separate reports that operations teams were manually stitching together in Excel. Report accuracy was validated at 99.5% against source system counts, and the dashboard was used by senior Cisco executives globally.

* * *

### Q8. Tell me about a time you had to write SQL for a regulatory or compliance report.

🔵 **Situation** At LSEG/Refinitiv, the compliance team required a daily MiFID II transaction reporting extract — a structured file of all FX trades executed during the day, with specific field mappings, validations, and submission deadlines.

🟡 **Task** I was responsible for building the SQL extraction logic that produced the compliant output file, ensuring all mandatory fields were populated and all business rules were met.

🟢 **Action**

  * Analysed the MiFID II field specification and mapped each required field to the Oracle source columns, documenting transformation rules in the STTM.
  * Wrote T-SQL stored procedures to extract, transform, and validate the trade data — including LEI validation, instrument classification lookup, and price normalisation.
  * Implemented 50+ automated data quality checks within the extraction pipeline (null checks on mandatory fields, valid code lookups, duplicate transaction reference detection).
  * Built an exception report for any records failing validation, routing them to the compliance team for manual review before submission.



🔴 **Result** The automated pipeline replaced a semi-manual process that had a 2–3% error rate. Post-automation, error rate dropped to under 0.05%. The pipeline ran daily without incident for the 16-month duration of the project.

* * *

### Q9. Describe a situation where you used SQL to reconcile data between two systems.

🔵 **Situation** At LSEG, the Oracle trading system and the SQL Server reporting database needed to stay in sync in near-real-time. Any discrepancy between them would result in incorrect risk exposure figures being reported to traders and risk managers.

🟡 **Task** I was tasked with designing the reconciliation framework to continuously validate consistency between the two systems.

🟢 **Action**

  * Built a scheduled T-SQL reconciliation job running every 15 minutes, comparing row counts, sum of trade amounts, and count of open positions between Oracle (via linked server) and SQL Server.
  * Added hash-based record-level comparison for any partition where aggregate totals didn't match — identifying the exact records at variance.
  * Configured alerting to notify the data engineering team within 2 minutes of any breach exceeding defined thresholds.
  * Created a reconciliation dashboard showing real-time sync status, historical breach log, and time-to-resolve metrics.



🔴 **Result** Achieved **99.9% data consistency** across both systems. Reconciliation issues dropped by **85%** compared to the previous manual process. The framework became the standard for all new data feeds onboarded to the platform.

* * *

### Q10. Tell me about a time you transformed business requirements into SQL scripts for testing.

🔵 **Situation** At J&J Global Services, the actuarial team provided a 40-page requirements document describing the logic for a new underwriting performance scorecard. The document was written in business language with no technical specification.

🟡 **Task** My task was to translate those business rules into executable SQL validation scripts that could be used for both development testing and UAT.

🟢 **Action**

  * Ran a series of requirements workshops with the actuarial team to clarify ambiguous rules and edge cases.
  * Documented 60+ individual business rules in a structured format: rule ID, description, SQL expression, expected pass/fail condition.
  * Wrote parameterised SQL test scripts in T-SQL for each rule, designed to be re-run at any point in the pipeline.
  * Created a test results table that logged rule ID, execution timestamp, pass/fail status, and row-level failures for easy triage.
  * Walked the actuarial team through the scripts during UAT, translating SQL output back into business language for their sign-off.



🔴 **Result** All 60 rules were validated and signed off in UAT within 3 sprints. The scripts were later reused as the automated regression test suite, catching 4 logic regressions during subsequent pipeline changes before they reached production.

* * *

## Section 2: Data Modelling & EDW Design (Q11–18)

* * *

### Q11. Tell me about a time you designed a data warehouse from scratch.

🔵 **Situation** At Cisco (via Wipro), there was no centralised data warehouse for TelePresence operational data. Reports were produced ad hoc by querying 15+ production tables directly, causing performance issues and inconsistent metrics across teams.

🟡 **Task** I was tasked with architecting and building a SQL Server data warehouse that would serve as a single source of truth for all TelePresence operational analytics.

🟢 **Action**

  * Conducted discovery sessions with operations, engineering, and leadership to understand reporting needs and KPIs.
  * Designed a star schema: `fact_conference` at the centre, with `dim_device`, `dim_user`, `dim_location`, `dim_date`, and `dim_failure_type`.
  * Built ETL stored procedures in T-SQL and SSIS to load each layer daily.
  * Implemented data quality validation at each ETL stage before data was promoted to the reporting layer.
  * Documented all table designs, transformation logic, and data dictionary in the project artefacts.



🔴 **Result** The warehouse consolidated data from 15+ source tables into a clean, performant model. Report accuracy was validated at **99.5%** against source systems. The platform supported executive-level reporting for 50,000+ global endpoints and was delivered within the agreed Waterfall project timeline.

* * *

### Q12. Describe a time you consolidated multiple data sources into a single data model.

🔵 **Situation** At J&J Global Services, 150+ analysts and executives were drawing data from 4 separate legacy systems — each with different schemas, naming conventions, and update frequencies. Reports from different systems frequently contradicted each other, eroding trust in the data.

🟡 **Task** I led the technical effort to consolidate all 4 sources into a single, standardised BigQuery model that would become the authoritative source of truth.

🟢 **Action**

  * Profiled all 4 source systems to understand schema structures, data volumes, quality issues, and business rules.
  * Designed a unified data model with standardised naming conventions and a shared business calendar dimension.
  * Built ETL pipelines in Python and BigQuery SQL to ingest, cleanse, and merge each source daily.
  * Applied deduplication logic using composite business keys where the same record appeared in multiple sources.
  * Worked with business stakeholders to validate the unified model against their existing reports before cutover.



🔴 **Result** The consolidated model replaced 4 separate data silos. All **150+ analysts and executives** globally migrated to the single model within 6 weeks. Report discrepancy incidents fell to zero in the 3 months post-launch. This work also directly contributed to identifying **$2M+ in cost savings** through more accurate cross-system analysis.

* * *

### Q13. Tell me about a time you applied Slowly Changing Dimension (SCD) logic in a real project.

🔵 **Situation** At Travelers Insurance, the claims data warehouse needed to track changes to policyholder information over time — specifically, address, coverage tier, and risk classification. Without historical tracking, point-in-time actuarial analysis was impossible.

🟡 **Task** I was responsible for implementing SCD Type 2 logic on the `dim_policyholder` dimension to preserve the full history of attribute changes.

🟢 **Action**

  * Added `effective_from`, `effective_to`, and `is_current` columns to the dimension.
  * Built a daily MERGE procedure: compared incoming data against the current record using a hash of the key attributes; if changed, closed the existing record (set `effective_to` = yesterday) and inserted a new row as the current version.
  * Applied the same approach to `dim_policy_product` to support product tier change analysis.
  * Validated by testing a known policyholder address change end-to-end through the pipeline.



🔴 **Result** Actuarial teams could now run accurate point-in-time queries — "what was this policyholder's risk tier at the time of the claim?" — which had previously required manual lookups in the legacy system. This unblocked a regulatory reserving report that had been on hold for 2 months.

* * *

### Q14. Give an example of a time you performed data profiling on a new data source.

🔵 **Situation** When the J&J Global Services project began onboarding a new regional claims system as the fourth data source into the BigQuery EDW, the source team provided limited documentation and no data dictionary.

🟡 **Task** I was tasked with profiling the new source to assess data quality, understand the structure, and identify any issues before pipeline development began.

🟢 **Action**

  * Wrote SQL profiling scripts to analyse: row counts, null rates per column, distinct value counts, min/max ranges, and format consistency for all 80+ columns.
  * Identified 11 columns with >30% null rates, 3 columns with mixed date formats, and 2 columns that appeared to duplicate existing fields under different names.
  * Documented findings in a data profiling report and presented to the source system owner.
  * Used the findings to design targeted cleansing rules and raise 6 data quality issues with the source team for upstream fixes.



🔴 **Result** Profiling prevented 3 critical data quality issues from reaching the production pipeline. The upstream fixes were implemented before go-live, and the profiling scripts were reused as the data quality baseline monitor for that source.

* * *

### Q15. Tell me about a time the data model you designed had to change due to new business requirements.

🔵 **Situation** Midway through the Travelers Insurance claims warehouse project, the business introduced a new fraud scoring system that generated a risk score per claim — a data element not present in the original model design.

🟡 **Task** I needed to incorporate the fraud score into the existing Gold layer without breaking downstream reports or requiring a full remodel.

🟢 **Action**

  * Assessed impact: the fraud score was a new non-additive fact tied to individual claims — it belonged in the `fact_claims` table but couldn't simply be appended without a schema migration.
  * Proposed and implemented a zero-downtime approach: added the column as nullable first, backfilled historical scores from the scoring system's API using a Python script, then applied NOT NULL constraint after validation.
  * Updated the STTM documentation to reflect the new field, its derivation, and its refresh cadence.
  * Communicated the change and updated Power BI report definitions with the fraud analytics team.



🔴 **Result** The change was delivered within one sprint with no downtime or downstream report breakage. The fraud score became a key dimension in the executive dashboard, enabling the fraud team to identify high-risk claim clusters for the first time.

* * *

### Q16. Describe a situation where you had to document data lineage for an audit or compliance requirement.

🔵 **Situation** At LSEG, the internal audit team required a full end-to-end data lineage map for the FX reconciliation platform — tracing every field in the daily regulatory report back to its Oracle source, through all transformations, to the final SQL Server output.

🟡 **Task** I was responsible for producing the lineage documentation within a 2-week deadline.

🟢 **Action**

  * Walked through every step of the ETL pipeline, cataloguing each transformation: source table → staging → transformation logic → target column.
  * Built a lineage matrix in Excel mapping 120+ fields across 3 transformation stages with notes on business rules applied at each step.
  * Cross-referenced with the STTM to ensure all documented transformations matched the actual code.
  * Presented the lineage documentation to the audit team and resolved 4 queries about specific field derivations.



🔴 **Result** The audit was completed without any findings related to data lineage. The documentation was retained as the ongoing reference for the platform and was updated automatically as part of the change management process thereafter.

* * *

### Q17. Tell me about a time you had to choose between different data modelling approaches and justify your decision.

🔵 **Situation** At the start of the Travelers Insurance claims warehouse project, there was a debate between the team and the client architects about whether to use a Star Schema or a fully normalised 3NF model for the EDW.

🟡 **Task** I was asked to evaluate both options and make a recommendation based on the reporting requirements.

🟢 **Action**

  * Analysed the query patterns from the business: 90% of queries were aggregated reads (SUM of claims by product, region, and date) — classic OLAP workload.
  * Ran a proof of concept: built the same report query against a 3NF model and a Star Schema model using a sample dataset. The Star Schema query was 60% faster and used 3 fewer joins.
  * Documented the trade-offs: Star Schema had minor data redundancy but delivered significantly better query performance and was simpler for analysts to understand.
  * Presented findings to the client architecture team with the PoC benchmarks.



🔴 **Result** The Star Schema was approved. The warehouse delivered sub-second query performance for Power BI dashboards, which was a stated requirement that the 3NF model could not meet without materialised views. The decision was validated at go-live.

* * *

### Q18. Give an example of when you implemented a Bronze-Silver-Gold layered architecture.

🔵 **Situation** At Travelers Insurance, the existing claims data process loaded raw source files directly into a single reporting table with no staging or quality layers — making it impossible to reprocess data without losing history, and causing frequent report errors when source file formats changed.

🟡 **Task** I was tasked with redesigning the pipeline architecture to introduce proper layering, improve resilience, and enable reprocessing.

🟢 **Action**

  * **Bronze layer** : Designed raw ingestion tables mirroring source file structures exactly, with load timestamps and source file references. No transformations — full audit trail preserved.
  * **Silver layer** : Built cleansing and standardisation logic — deduplication, date format normalisation, currency conversion, business rule application. Quality checks ran here; failed records were quarantined.
  * **Gold layer** : Aggregated, business-ready tables feeding Power BI dashboards. Rebuilt from Silver on each run, so any Silver fix automatically propagated to Gold.
  * Scheduled each layer sequentially with dependency checks and alerting.



🔴 **Result** The layered architecture reduced data latency from 24 hours to under 15 minutes via CDC-based Silver loading. Data reprocessing — previously a manual half-day effort — became a one-click operation. Report error rate dropped to near zero post-implementation.

* * *

## Section 3: ETL / Pipeline & Data Engineering (Q19–25)

* * *

### Q19. Tell me about a time you built an end-to-end ETL pipeline from scratch.

🔵 **Situation** At J&J Global Services, the Salesforce CRM data was being manually exported to CSV and uploaded to the data warehouse weekly by the sales operations team — a fragile, error-prone process that caused 4–6 hour data gaps.

🟡 **Task** I was asked to design and build a fully automated Salesforce-to-EDW integration that would eliminate manual effort and reduce latency.

🟢 **Action**

  * Designed a C# background service that authenticated with the Salesforce API, extracted delta records (modified since last run), and staged them in SQL Server.
  * Built transformation logic to map Salesforce fields to the warehouse schema, applying cleansing rules and dimension lookups.
  * Implemented error handling: failed records were logged to a quarantine table with error codes; the service resumed from the last successful checkpoint on restart.
  * Scheduled the service to run every 30 minutes and added monitoring with email alerting on failure.
  * Documented the integration architecture in the STTM and produced a runbook for the operations team.



🔴 **Result** Manual data uploads were eliminated entirely. Data latency dropped from up to 6 hours to under 30 minutes. The pipeline ran without a production incident in its first 6 months.

* * *

### Q20. Describe a time you implemented CDC (Change Data Capture) to reduce data latency.

🔵 **Situation** At Travelers Insurance, the fraud detection team was working with claims data that was 24 hours old — loaded by a nightly batch job. This meant fraud patterns were only visible the following day, significantly reducing the team's ability to act quickly.

🟡 **Task** I was responsible for designing a near-real-time data pipeline to replace the nightly batch and reduce latency to under 15 minutes.

🟢 **Action**

  * Evaluated CDC approaches: chose a log-based CDC pattern using Streams and Tasks methodology to capture row-level inserts, updates, and deletes from the source as they occurred.
  * Redesigned the Silver layer pipeline to process CDC events incrementally rather than full table refreshes.
  * Implemented a micro-batch schedule: pipeline ran every 10 minutes, processing only changed records.
  * Added latency monitoring: a dashboard tracked time from source event to Gold layer availability; alerts fired if latency exceeded 15 minutes.



🔴 **Result** Data latency dropped from **24 hours to under 15 minutes**. The fraud team detected and acted on 3 fraud patterns in the first month that would previously have been missed due to the daily lag.

* * *

### Q21. Tell me about a time a data pipeline failed in production and how you handled it.

🔵 **Situation** At LSEG, the nightly Oracle-to-SQL Server sync failed silently at 2 AM — the job appeared to complete successfully, but a downstream validation check at 6 AM revealed that 3 hours of FX trades had not been loaded. The trading desk opened without complete position data.

🟡 **Task** As the data engineer on call, I needed to diagnose the failure, recover the missing data, and ensure the trading desk had complete data before market open at 8 AM.

🟢 **Action**

  * Immediately checked the sync service logs: found a deadlock in the SQL Server target caused by a long-running ad hoc query from a BI analyst running concurrently with the sync job.
  * Identified the exact time window of the gap (3:15 AM – 6:00 AM) from the Oracle transaction log.
  * Re-ran the sync for the affected time window using a parameterised backfill procedure, loading the missing trades into the target.
  * Ran the full reconciliation framework to confirm all counts and aggregates matched Oracle.
  * Communicated status updates to the trading desk manager at 30-minute intervals.



🔴 **Result** Missing data was fully recovered by 7:30 AM — 30 minutes before market open. I then implemented two preventive measures: a query governor to limit ad hoc query resource consumption during the sync window, and a post-sync record count validation that would alert immediately if the expected volume wasn't met.

* * *

### Q22. Give an example of when you automated a manual data process.

🔵 **Situation** At Google (Street View project), the compliance team was spending 100+ hours per month manually pulling data from 3 separate internal tools, combining them in Excel, and producing a regulatory-style coverage report for global leadership.

🟡 **Task** I was asked to automate the entire report generation process end-to-end.

🟢 **Action**

  * Mapped the existing manual process step-by-step with the compliance team to understand all data sources, transformations, and output formats.
  * Built Python scripts to extract data from each of the 3 source systems via API and ingest into BigQuery.
  * Created a scheduled BigQuery SQL pipeline that applied the same transformation and aggregation logic previously done in Excel.
  * Connected the output to a Looker Studio dashboard with automated daily refresh, replacing the manual report entirely.
  * Used AppScript to auto-generate the formatted PDF report version and email it to stakeholders on a schedule.



🔴 **Result** **100+ manual hours per month** were eliminated. The report was delivered daily instead of monthly, giving leadership fresher insight. The compliance team redeployed their time to analysis rather than data assembly.

* * *

### Q23. Tell me about a time you had to handle large volumes of data efficiently.

🔵 **Situation** At Cisco (via Wipro), the TelePresence data warehouse was processing 500,000+ conference records daily across 50,000 global endpoints. As the dataset grew, the nightly ETL was taking over 5 hours to complete — overrunning the batch window and delaying morning reports.

🟡 **Task** I was tasked with reducing the ETL runtime to under 2 hours without changing the report output.

🟢 **Action**

  * Profiled the ETL: identified that a full table scan was occurring on a 3-year history table at every run, even though only the previous day's data was new.
  * Redesigned the ETL to use incremental loading: only process records with `load_date = YESTERDAY`, then merge into the history table using a MERGE statement.
  * Partitioned the fact table by `conference_date` — queries against recent data no longer scanned historical partitions.
  * Moved expensive dimension lookups to pre-computed lookup tables refreshed once per day rather than inline during fact table load.



🔴 **Result** ETL runtime dropped from over 5 hours to **under 90 minutes** — within the batch window with headroom. Morning reports were consistently on time. The approach was documented as the ETL optimisation standard for future projects.

* * *

### Q24. Describe a time you implemented data quality checks in a pipeline.

🔵 **Situation** At LSEG, the FX reconciliation platform had no automated data quality framework when I joined. Data issues were only discovered when traders or risk managers spotted anomalies in reports — often hours after the data had been loaded.

🟡 **Task** I was tasked with designing and implementing an automated data quality validation framework that would catch issues at ingestion before they reached the reporting layer.

🟢 **Action**

  * Worked with the business and compliance team to catalogue all data quality rules: mandatory field checks, valid code lookups, range checks, duplicate detection, and cross-system aggregate reconciliation.
  * Implemented 50+ automated SQL-based validation checks running after each pipeline stage.
  * Built a data quality results table logging each check's pass/fail status, failure row counts, and timestamps.
  * Configured real-time alerting: any critical check failure triggered a PagerDuty alert within 2 minutes.
  * Created a DQ dashboard for the data steward team showing daily quality scores by data domain.



🔴 **Result** Reconciliation issues detected post-reporting dropped by **85%**. The framework caught 12 data quality failures in its first month that would previously have reached the trading desk undetected.

* * *

### Q25. Tell me about a time you had to migrate data from a legacy system to a modern platform.

🔵 **Situation** At AXA Insurance, the existing claims processing platform was a 15-year-old legacy system with no API access, limited documentation, and a batch-only data feed. The business needed to migrate to a modern cloud-based data platform while maintaining 100% data continuity.

🟡 **Task** I led the data migration workstream — responsible for extracting legacy data, transforming it to the new schema, loading it to the target platform, and validating completeness.

🟢 **Action**

  * Conducted AS-IS analysis: inventoried all legacy tables (200+ objects), documented data types, business rules, and known quality issues.
  * Designed the TO-BE schema on the new platform with improved normalisation and clearer naming conventions.
  * Built the migration ETL pipeline in stages: historical data (5 years) migrated in bulk first, then delta loads to keep in sync during the parallel-run period.
  * Implemented a comprehensive validation suite: row count reconciliation, field-level hash comparison, and business metric reconciliation (total claims value, approval rates by category).
  * Managed a 4-week parallel run where both systems ran simultaneously — daily comparison reports issued to the business.



🔴 **Result** Migration completed with **zero data loss** validated across all 200+ objects. The parallel run identified and resolved 7 data discrepancies before cutover. Post-migration, claims data latency dropped from 24 hours to under 15 minutes, and the manual review process was reduced by 40%.

* * *

## Section 4: Business Analysis & Stakeholder Management (Q26–33)

* * *

### Q26. Tell me about a time you gathered requirements for a complex data project.

🔵 **Situation** At J&J Global Services, the actuarial and finance teams needed a new underwriting performance scorecard — but the two teams had conflicting views on which metrics mattered and how they should be calculated.

🟡 **Task** I was responsible for facilitating the requirements gathering process, resolving the conflicts, and producing a signed-off business requirements document (BRD) before development began.

🟢 **Action**

  * Conducted separate discovery workshops with each team first to understand their individual needs and current pain points.
  * Identified 6 specific metric definitions where the teams disagreed (e.g., "earned premium" — actuarial used a different time-apportionment method than finance).
  * Facilitated a joint workshop, presenting both definitions side by side with examples of the different outputs each would produce.
  * Proposed a resolution: both definitions would be implemented as separate calculated fields with clear labels, satisfying both teams.
  * Documented 60+ business rules in a structured BRD and obtained formal sign-off from both stakeholders.



🔴 **Result** The BRD was signed off after a single revision cycle — unusually fast for a cross-departmental project. The scorecard launched on time with zero post-launch disputes about metric definitions.

* * *

### Q27. Describe a situation where you had to explain a complex data issue to a non-technical audience.

🔵 **Situation** At Google, a senior executive asked why the Street View coverage dashboard was showing different numbers than a slide deck presented by a different team the previous week. The discrepancy was caused by a difference in how "coverage" was defined and measured — a nuanced technical issue involving data freshness and geometry calculation methodology.

🟡 **Task** I needed to explain the discrepancy clearly to a non-technical executive in a way that built confidence in the data rather than undermining it.

🟢 **Action**

  * Avoided technical jargon entirely. Used an analogy: "Think of it like two speedometers in the same car — one shows current speed, one shows average speed for the last hour. Both are correct, but they're measuring different things."
  * Prepared a one-page visual showing the two measurement definitions side by side with example numbers.
  * Acknowledged the confusion clearly, explained the root cause, and proposed a solution: a single agreed definition going forward, published in a data dictionary.
  * Followed up with a written summary so the executive could share it with their team.



🔴 **Result** The executive appreciated the clarity and approved the proposal for a unified metric definition. The data dictionary entry was created and used as the reference for all subsequent coverage reporting. No further discrepancy queries were raised.

* * *

### Q28. Tell me about a time you managed conflicting priorities from multiple stakeholders.

🔵 **Situation** At J&J Global Services, I was simultaneously supporting the actuarial team (who wanted a new loss ratio dashboard urgently for a board presentation) and the IT compliance team (who had a deadline for data lineage documentation required for an internal audit).

🟡 **Task** Both tasks were high-priority and time-sensitive; I had to manage my capacity and stakeholder expectations without compromising either deliverable.

🟢 **Action**

  * Spoke to both stakeholders to understand their hard deadlines: actuarial needed the dashboard in 5 days; audit documentation was due in 8 days.
  * Negotiated with the actuarial team: delivered a functional first version of the dashboard in 3 days (covering the 5 most critical metrics) with a commitment to complete the remaining metrics after the audit deadline.
  * Communicated the plan transparently to both stakeholders, documenting the agreed scope and timeline in writing.
  * Allocated mornings to the dashboard build (focused development) and afternoons to the lineage documentation (more research-based).



🔴 **Result** Both deliverables were met on time. The actuarial team presented the dashboard at their board meeting. The audit documentation was submitted with one day to spare and received no findings. The structured communication approach was noted positively in my performance review.

* * *

### Q29. Give an example of when your data analysis directly influenced a business decision.

🔵 **Situation** At J&J Global Services, a senior director was planning to consolidate two regional claims processing teams based on the assumption that one region was underperforming.

🟡 **Task** I was asked to provide data to support the analysis. However, as I dug into the numbers, I found the picture was more complex than the initial assumption suggested.

🟢 **Action**

  * Built a multi-dimensional analysis comparing the two regions across: claims volume, processing time, approval rates, error rates, cost per claim, and claim complexity distribution.
  * Discovered the "underperforming" region actually handled a significantly higher proportion of complex, high-value claims — which naturally took longer and had different approval rates.
  * When normalised for claim complexity, the region's performance metrics were comparable or better than the other region.
  * Presented the findings with clear visualisations and a written narrative to the director.



🔴 **Result** The consolidation plan was reconsidered. Instead, the director used the analysis to identify **$2M+ in cost savings** from process improvements in claims categorisation and routing — a better outcome than consolidation would have achieved.

* * *

### Q30. Describe a time you had to conduct an AS-IS / TO-BE analysis.

🔵 **Situation** At AXA Insurance, the legacy claims data platform had never been formally documented. Before migrating to the new cloud platform, the project team needed a clear picture of what existed and what the target state should look like.

🟡 **Task** I led the AS-IS/TO-BE analysis workstream, covering data flows, system integrations, data quality issues, and reporting capabilities.

🟢 **Action**

  * **AS-IS** : Ran a 3-week discovery phase — interviewed system owners, traced data flows by following actual claims records through each system, documented all ETL processes (many undocumented), catalogued 200+ data objects and their inter-dependencies.
  * **Identified pain points** : 24-hour data latency, no data lineage, 3 conflicting definitions of "active policy" across systems, no automated quality checks.
  * **TO-BE** : Designed the target architecture — layered cloud EDW, sub-15-minute data latency, unified data dictionary, automated quality framework, full lineage tracking.
  * **Gap analysis** : Documented 14 gaps across data, process, and technology dimensions, each with an estimated effort and priority.



🔴 **Result** The gap analysis became the project delivery roadmap. All 14 gaps were addressed by go-live. The TO-BE architecture delivered the agreed improvements: latency reduced from 24 hours to under 15 minutes, zero lineage gaps at audit, and 40% reduction in manual data review effort.

* * *

### Q31. Tell me about a time you identified a process improvement opportunity through data analysis.

🔵 **Situation** At Google (Street View), the image ingestion pipeline had a known but unmeasured latency — the time from image capture to public availability was reported as "typically 2–4 weeks" but no one had measured it precisely at each stage.

🟡 **Task** I identified this as an opportunity to understand and optimise the pipeline, and proposed an analysis to the programme manager.

🟢 **Action**

  * Built a BigQuery analysis tracking individual images from capture timestamp through each processing stage — QA, stitching, geo-tagging, and publishing — using event logs.
  * Identified that 70% of total latency was occurring in a single geo-tagging step where a queue was backing up due to a resource constraint.
  * Quantified the impact: fixing the bottleneck would reduce overall pipeline time by approximately 25%.
  * Presented findings to the engineering team with the data to support a resource allocation request.



🔴 **Result** The bottleneck was resolved within one sprint. Overall pipeline processing time improved by **25%** , directly accelerating Street View coverage refresh rates globally.

* * *

### Q32. Describe a time you had to co-ordinate and manage a UAT cycle.

🔵 **Situation** At Cisco (via Wipro), the TelePresence data warehouse go-live required formal UAT sign-off from 4 business stakeholder groups across 3 time zones before the production cutover could proceed.

🟡 **Task** I was responsible for planning, co-ordinating, and managing the entire UAT process from test script preparation through to formal sign-off.

🟢 **Action**

  * Translated 40+ business requirements into structured UAT test cases with: test ID, scenario description, input data, expected output, and pass/fail criteria.
  * Created a shared UAT tracking spreadsheet with a status dashboard visible to all stakeholders.
  * Scheduled UAT sessions across time zones, providing each group with a UAT pack including test scripts, access details, and escalation contacts.
  * Logged all defects in JIRA with severity classifications; triaged with developers daily.
  * Produced daily status updates for the project manager and a final UAT sign-off report.



🔴 **Result** UAT completed within the 3-week window with all 40+ test cases passed. 8 defects were raised; 7 resolved before sign-off, 1 deferred to post-go-live with business approval. Production cutover proceeded on schedule.

* * *

### Q33. Give an example of when your documentation significantly helped a project or team.

🔵 **Situation** At J&J Global Services, the project had significant analyst turnover mid-programme — two experienced analysts left within 3 months. The complex ETL logic and business rules they had built were largely undocumented in their heads.

🟡 **Task** I took ownership of the documentation effort to capture all existing logic, prevent knowledge loss, and ensure the project could continue without disruption.

🟢 **Action**

  * Audited all existing ETL code and pipelines, reverse-engineering the transformation logic from the SQL and Python scripts.
  * Created STTM documentation for all 200+ business attributes: source field, transformation rule, business definition, validation logic, and owner.
  * Wrote a data dictionary covering all tables in the EDW with field-level descriptions and examples.
  * Produced process flow diagrams for the 6 main data pipelines.
  * Held knowledge transfer sessions with the incoming analysts using the documentation as the reference material.



🔴 **Result** The two new analysts were fully productive within 2 weeks — compared to an estimated 6–8 weeks without documentation. The STTM and data dictionary were later cited by the client as a project deliverable of lasting value, and were maintained as live documents through to programme completion.

* * *

## Section 5: Data Quality & Problem Solving (Q34–39)

* * *

### Q34. Tell me about the most challenging data quality problem you have faced.

🔵 **Situation** At LSEG, the FX platform intermittently reported net positions that were correct to within 0.001% — close enough to pass automated threshold checks but incorrect enough to distort risk exposure reports for very large positions. The issue was sporadic and had resisted investigation for 3 weeks before I was assigned to it.

🟡 **Task** I was tasked with finding the root cause of a data quality issue that had defeated previous attempts at diagnosis.

🟢 **Action**

  * Started from first principles: ignored the previous investigation notes to avoid anchoring bias.
  * Built a custom comparison query that hashed every field of every trade record in both Oracle and SQL Server and flagged any hash mismatch — not just aggregate differences.
  * Found that a tiny fraction of trades (0.003% of volume) had floating-point rounding differences caused by different precision settings between the Oracle NUMBER type and SQL Server FLOAT for the notional amount field.
  * The difference was too small to trigger the reconciliation threshold but cumulated to a visible discrepancy on large positions.
  * Fix: changed the target column to DECIMAL(18,8) to match Oracle's precision; rebuilt affected historical records.



🔴 **Result** The intermittent discrepancy was eliminated. The fix was implemented in under 4 hours once the root cause was identified. I updated the data quality framework to include a precision-level field comparison check, preventing similar issues on any future data type migrations.

* * *

### Q35. Describe a time you caught a data issue before it reached the business.

🔵 **Situation** At Travelers Insurance, during a routine post-load validation check, I noticed that the daily claims count in the Gold layer was 15% lower than the previous day's — a drop that was statistically unusual but had not triggered any alerts because the absolute number was still within the configured threshold range.

🟡 **Task** I needed to determine whether this was a genuine business trend or a pipeline issue, and resolve it before the fraud team began their morning analysis.

🟢 **Action**

  * Queried the Bronze layer: raw claim counts from the source file were normal — no drop.
  * Queried the Silver layer: counts dropped at the Silver quality check stage.
  * Identified that a new source file had introduced a previously unseen date format (`DD/MM/YYYY` instead of `YYYY-MM-DD`) causing 15% of records to fail the date validation check and be quarantined.
  * Fixed the date parsing logic in the Silver transformation, reprocessed the quarantined records, and reloaded to Gold.



🔴 **Result** The issue was resolved before the fraud team's 9 AM analysis run — with zero business impact. I updated the Silver validation to handle both date formats and added a quarantine rate alert: any batch where >2% of records are quarantined triggers an immediate notification.

* * *

### Q36. Tell me about a time you had to investigate root cause of a recurring data issue.

🔵 **Situation** At LSEG, the operations team reported that approximately once a fortnight, the reconciliation report showed a small number of trades with a status mismatch between Oracle and SQL Server — trades that were "settled" in Oracle but still showing as "pending" in SQL Server.

🟡 **Task** I was asked to identify why the issue kept recurring despite previous patches, and implement a permanent fix.

🟢 **Action**

  * Logged every occurrence over a 6-week period: time of day, volume, instrument type, and trade size.
  * Identified a pattern: all occurrences happened between 5:00–5:15 PM — the settlement cut-off window when Oracle processed a batch status update.
  * Diagnosed a race condition: the SQL Server sync job ran every 15 minutes and was occasionally reading Oracle mid-batch-update, capturing some trades before their status was finalised.
  * Fix: implemented a "lock check" — the sync job now verifies that the Oracle batch settlement process has completed before beginning its read, using a status flag in a control table.



🔴 **Result** The race condition was eliminated entirely. The recurring issue — which had appeared fortnightly for over 4 months — has not recurred since the fix was deployed.

* * *

### Q37. Give an example of when you used data to challenge an assumption made by the business.

🔵 **Situation** At J&J Global Services, the finance team assumed their actuarial cost allocation model was accurate to within 2% based on historical manual reviews. They were using this assumption to justify not investing in a more rigorous automated reconciliation process.

🟡 **Task** I offered to run an independent data analysis to test the assumption before the investment decision was finalised.

🟢 **Action**

  * Extracted 12 months of cost allocation records from 3 source systems and the actuarial model output.
  * Built a field-level comparison across 15 cost categories, calculating the variance between the model allocation and the actual source figures for each category.
  * Found that while the overall variance was indeed ~2%, specific cost categories had variances of up to 18% — masked by offsetting errors in other categories.
  * Presented the category-level breakdown with the financial impact quantified for the highest-variance items.



🔴 **Result** The finance team revised their position. The automated reconciliation investment was approved. The more accurate reconciliation process subsequently identified specific allocation errors that contributed to the **$2M+ cost saving** finding.

* * *

### Q38. Tell me about a time you had to present data findings that delivered bad news.

🔵 **Situation** At Cisco (via Wipro), after completing the root cause analysis of TelePresence system failures, my analysis showed that one of the 3 critical failure modes was caused by a design flaw in the recently deployed software version — meaning the engineering team's recent release was the primary driver of a 40% spike in support incidents.

🟡 **Task** I had to present these findings to the engineering director, whose team had authored the release, in a way that was factual, fair, and constructive rather than accusatory.

🟢 **Action**

  * Structured the presentation: led with the business impact (support ticket volume and cost), then walked through the data evidence methodically — correlation with the release date, failure type distribution, affected device models.
  * Acknowledged the complexity: noted that the failure was triggered by an unusual combination of device firmware version and the new software — not straightforward to have anticipated in testing.
  * Focused the second half of the presentation on the forward-looking fix options and recommended a patch approach.
  * Shared the data and methodology in advance so the engineering team could validate the findings before the meeting.



🔴 **Result** The engineering director accepted the findings without defensiveness — crediting the pre-shared data as what made the difference. A patch was fast-tracked and deployed within 2 weeks, **reducing repeat production incidents by 40%**.

* * *

### Q39. Describe a time you had to work with incomplete or messy data and still deliver an output.

🔵 **Situation** At J&J, when onboarding the fourth legacy data source, the source team could only provide a partial data extract for the first 3 months due to a system constraint — 30% of historical records were missing for that period.

🟡 **Task** I needed to produce an accurate actuarial report covering that period without the missing data, while being transparent about the data limitations.

🟢 **Action**

  * Quantified the gap precisely: identified which specific months, regions, and product lines were affected.
  * Used statistical imputation for the missing period: calculated the ratio of the fourth source's contribution to total claims in the periods where complete data was available, then applied that ratio to impute the missing records.
  * Built the imputed data as a separate, clearly labelled segment in the report — actual vs imputed — so actuaries could weight them appropriately.
  * Documented the imputation methodology fully and flagged the confidence intervals.



🔴 **Result** The actuarial team accepted the approach and used the report for their analysis with the documented caveats. The full data was subsequently received and reconciled against the imputed figures — the imputation error was within 3%, validating the methodology.

* * *

## Section 6: SDLC, Delivery & Collaboration (Q40–46)

* * *

### Q40. Tell me about a time you delivered a data project under significant time pressure.

🔵 **Situation** At LSEG, a new regulatory reporting requirement under MiFID II came into effect with 6 weeks' notice — requiring a daily automated trade reporting extract that did not yet exist on the platform.

🟡 **Task** I was the lead data analyst responsible for building the extraction pipeline, ensuring compliance with the MiFID II field specification, and meeting the regulatory deadline.

🟢 **Action**

  * Immediately convened a kick-off with compliance, IT, and the trading desk to scope the requirements.
  * Prioritised ruthlessly: the first 2 weeks were dedicated solely to understanding the field specification and mapping it to existing Oracle source data.
  * Built the pipeline in parallel streams: field mapping and transformation logic in week 2–3; validation framework in week 3–4; UAT with compliance in week 5; production deployment and parallel-run in week 6.
  * Flagged 2 data availability gaps early (2 required fields not in the current data model) and worked with the trading system team to expose them via a new Oracle view.



🔴 **Result** The pipeline was live and compliant by the regulatory deadline. The first submission had a **< 0.05% error rate** — well within the accepted threshold. The 6-week delivery was cited by the compliance director as one of the fastest regulatory implementations on record for the platform.

* * *

### Q41. Describe a situation where you had to work across teams to resolve a data issue.

🔵 **Situation** At J&J Global Services, a mismatch between the finance team's revenue figures and the actuarial team's claims cost figures was causing a deadlock in the quarterly board pack preparation — each team believed the other's data was wrong.

🟡 **Task** I was brought in as a neutral data party to investigate the discrepancy and help both teams reach a resolution.

🟢 **Action**

  * Obtained the data extracts from both teams and compared them field by field.
  * Identified that both datasets were technically correct — but used different currency conversion date conventions: finance used month-end FX rates, actuarial used transaction-date FX rates.
  * Convened a joint session with both teams, walked through 3 specific examples showing the identical transaction producing different values under each convention.
  * Facilitated agreement on a single FX rate convention for the board pack (month-end), and proposed that both conventions be documented in the data dictionary for future reference.



🔴 **Result** The board pack deadlock was resolved within 48 hours. Both teams adopted the agreed convention. The data dictionary entry was created and has prevented the same disagreement from recurring in subsequent quarters.

* * *

### Q42. Tell me about a time you mentored or supported a less experienced team member.

🔵 **Situation** At Cisco (via Wipro), two junior analysts joined the team mid-project with strong academic backgrounds but limited production SQL experience — they were making query errors that were causing incorrect ETL outputs during development.

🟡 **Task** As the technical lead, I was responsible for bringing them up to speed quickly without slowing down the project delivery.

🟢 **Action**

  * Introduced a peer review process: no SQL changes were merged without review — I reviewed their code for the first 4 weeks, giving written feedback on each query explaining why changes were made.
  * Ran two informal "SQL clinic" sessions (1 hour each) covering the most common mistakes I was seeing: implicit type conversions, missing NULL handling, incorrect join types, and set-based vs row-based thinking.
  * Assigned progressively more complex tasks as their confidence grew — starting with dimension table loads (simpler) and moving to fact table ETL (more complex).



🔴 **Result** Within 6 weeks, both analysts were producing production-quality SQL independently. Peer review reduced their query error rate by **35%**. One of the analysts later took ownership of a key ETL module without supervision — a significant step in their development.

* * *

### Q43. Give an example of a time you worked in both Agile and Waterfall environments on the same programme.

🔵 **Situation** At J&J Global Services, the overall EDW consolidation was governed under a Waterfall contract (fixed scope, fixed deadlines, formal change control) — but the individual dashboard development workstream was run as Agile sprints because the business requirements for reporting were still evolving.

🟡 **Task** I had to operate effectively in both delivery modes simultaneously — maintaining formal documentation and change control for the EDW work while running fast, iterative development for the dashboards.

🟢 **Action**

  * For the **Waterfall EDW workstream** : maintained a strict requirements traceability matrix, produced formal documentation artefacts (BRD, STTM, test scripts) at each phase gate, and managed scope changes through the formal change control board.
  * For the **Agile dashboard workstream** : ran 2-week sprints with the BI team, daily stand-ups, backlog grooming with the business, and sprint demos to gather iterative feedback.
  * Key challenge: new dashboard requirements sometimes revealed EDW data gaps. I designed a liaison process: dashboard requirements were assessed for EDW impact and formally raised through the Waterfall change control if a data model change was needed.



🔴 **Result** Both workstreams delivered on time. The liaison process prevented 3 instances of dashboard development outpacing the EDW model, which would have caused re-work. The client highlighted the dual-mode delivery as a strength of the project approach in the post-implementation review.

* * *

### Q44. Describe a time you identified and escalated a risk on a data project.

🔵 **Situation** At AXA Insurance, 6 weeks before the planned go-live of the legacy migration, I identified during validation testing that 18 months of historical claims data from one regional office had not been included in the data extract provided by the legacy system team — a gap that would make the actuarial reserving model inaccurate from day one.

🟡 **Task** I needed to escalate this risk clearly and quickly so it could be resolved before go-live, without causing panic or blame.

🟢 **Action**

  * Quantified the impact immediately: which reports would be affected, by how much, and what the business consequence would be (incorrect reserve calculations, potential regulatory issue).
  * Raised it formally in the project risk log with a clear description, severity (High), and proposed resolution options: (a) delay go-live by 4 weeks to obtain the missing data, or (b) go-live on schedule with the gap clearly documented and a 4-week remediation plan.
  * Escalated to the project manager and steering committee the same day with a one-page briefing note.
  * Worked with the legacy system team to explore how quickly the missing extract could be produced.



🔴 **Result** The steering committee chose option (b) with a documented caveat. The missing data was extracted and loaded within 3 weeks of go-live — one week ahead of the remediation deadline. The risk escalation was credited with giving the business time to prepare their stakeholders rather than discovering the gap post-launch.

* * *

### Q45. Tell me about a time you received critical feedback and how you responded.

🔵 **Situation** Early in the J&J Global Services project, I delivered a data model design to the client architect who gave strong feedback that the model was over-engineered for the current requirements — too many dimensions, too much normalisation, and it would be harder for the 150 analysts to query intuitively.

🟡 **Task** I had to decide whether the feedback was valid and, if so, how to redesign the model quickly without losing the analytical capability it was intended to provide.

🟢 **Action**

  * Sat with the feedback for a day before responding — resisted the urge to defend the design immediately.
  * Reviewed the model against the actual query patterns I had collected during requirements gathering: the architect was right — the majority of queries were straightforward aggregations that didn't need the complexity I had built in.
  * Set up a follow-up session with the architect to redesign collaboratively, using the query patterns as the guiding criteria.
  * Simplified the model significantly: reduced from 12 dimensions to 7, denormalised 3 lookup tables, added pre-computed summary tables for the most common query patterns.



🔴 **Result** The revised model was approved in the next review. Query performance improved because of the simpler join paths. More importantly, analyst adoption was higher than expected — the simplified model was cited by the business as one of the most intuitive EDW structures they had worked with.

* * *

### Q46. Give an example of when you proactively identified an improvement that was outside your formal remit.

🔵 **Situation** At Google (Street View), while building the compliance reporting pipeline, I noticed that the bug tracking process used by the QA team relied on a manual triage step where engineers checked a shared inbox and manually created tickets in the bug tracking system — a process prone to delays and missed items.

🟡 **Task** This was outside my formal remit (I was responsible for data pipelines, not QA tooling) — but I saw an opportunity to add value and reduce a bottleneck that was indirectly affecting data quality metrics.

🟢 **Action**

  * Raised the observation with the QA team lead informally — asked if they'd welcome a suggested improvement.
  * With their agreement, built an AppScript-based automation that monitored the shared inbox, parsed incoming bug reports, and automatically created structured tickets in the bug tracking system with pre-populated fields.
  * Tested with a subset of bug types, refined based on QA team feedback, then rolled out fully.



🔴 **Result** Bug-to-ticket creation time dropped from an average of 4+ hours to under 5 minutes. Average bug-to-resolution time fell by **18 hours** and QA team efficiency improved by **20%** — a meaningful impact on the programme's overall delivery speed.

* * *

## Section 7: Banking & Finance Domain (Q47–50)

* * *

### Q47. Tell me about your experience working in a corporate banking or finance data environment.

🔵 **Situation** At LSEG/Refinitiv, I was embedded in the FX trading data engineering team — working directly with the data that drove live trading decisions, risk management, and regulatory reporting for one of the world's largest financial data providers.

🟡 **Task** My core responsibility was ensuring that the Oracle-to-SQL Server data synchronisation platform maintained the data accuracy and timeliness needed to support live FX trading operations.

🟢 **Action**

  * Designed and built the C# background service for Oracle-to-SQL Server real-time sync, handling millions of trade records daily.
  * Implemented a 50+ check automated validation framework running every 15 minutes to detect any sync discrepancies before they could affect risk reports.
  * Worked directly with risk managers and traders to understand their data requirements and SLA needs — daily stand-ups with the trading desk during critical periods.
  * Produced all data artefacts to regulatory standards: STTM documentation, data lineage maps, audit-ready transformation logic.



🔴 **Result** Achieved **99.9% data consistency** across both systems throughout the 16-month engagement. Reconciliation issues reduced by 85%. The platform successfully supported the trading desk through two high-volatility market periods with zero data-related incidents.

* * *

### Q48. Describe a time you worked on a data project with significant regulatory compliance requirements.

🔵 **Situation** At LSEG, the FX platform was subject to MiFID II transaction reporting obligations — a European regulation requiring all financial firms to report detailed trade data to regulators within strict time windows with high accuracy requirements.

🟡 **Task** I was responsible for building and maintaining the automated pipeline that produced the daily MiFID II transaction report from raw trade data.

🟢 **Action**

  * Analysed the regulatory field specification (50+ mandatory fields) and mapped each to Oracle source columns, documenting transformation rules in a compliance-reviewed STTM.
  * Built the extraction, validation, and formatting pipeline with 50+ automated quality checks — including LEI validation, instrument classification, price normalisation, and mandatory field completeness.
  * Designed an exception workflow: any trade failing validation was routed to the compliance team for review before being included or excluded from the submission.
  * Maintained full data lineage documentation for every field in the report — a regulatory requirement under BCBS 239-aligned governance standards.



🔴 **Result** Error rate on regulatory submissions dropped from ~2% (with the previous semi-manual process) to **< 0.05%**. No regulatory findings were raised against the firm during the period I maintained the pipeline.

* * *

### Q49. Tell me about a time you worked on a data reconciliation problem in a financial context.

🔵 **Situation** At LSEG, the daily reconciliation between the Oracle front-office trading system and the SQL Server risk reporting database was a critical control — any unresolved breaks could result in incorrect VaR (Value at Risk) calculations being published to the risk management team.

🟡 **Task** I was responsible for designing the reconciliation framework from scratch and ensuring it met the speed and accuracy requirements of a live trading environment.

🟢 **Action**

  * Designed a multi-level reconciliation: first aggregate (total position by currency pair), then record-level for any aggregate mismatch (hash comparison of individual trades), then field-level for any record mismatch.
  * Implemented the framework to run every 15 minutes during trading hours, with results published to a live reconciliation dashboard visible to the risk management team.
  * Set tiered alerting: informational for minor breaks within defined tolerances; immediate PagerDuty alert for any break exceeding the risk threshold.
  * Produced a daily reconciliation sign-off report for the compliance team.



🔴 **Result** Reconciliation breaks reduced by **85%** versus the previous manual process. Time to detect and resolve breaks dropped from an average of 4 hours to under 30 minutes. The framework was later adopted as the reconciliation standard across 2 additional data feeds on the same platform.

* * *

### Q50. Give an example of a time you contributed to improving data governance in an organisation.

🔵 **Situation** At J&J Global Services, data governance was largely informal — different teams used different definitions for the same business terms, there was no central data dictionary, and data access was managed ad hoc with no documented ownership or lineage.

🟡 **Task** As part of the EDW consolidation, I was asked to establish a foundational data governance framework alongside the technical delivery.

🟢 **Action**

  * **Data dictionary** : Created a comprehensive glossary of 200+ business terms — field definitions, business owners, valid values, transformation rules — maintained in Confluence.
  * **Data lineage** : Documented end-to-end lineage for all 6 major data pipelines, mapping source-to-target transformations for every field.
  * **Data ownership** : Worked with the business to assign a named data steward to each data domain (underwriting, claims, finance, HR) — the steward was responsible for approving definition changes.
  * **Access governance** : Proposed and implemented a role-based access model — analysts accessed aggregated views; raw data access required data steward approval.
  * **Quality standards** : Established measurable data quality SLAs for each domain (e.g., <0.1% null rate on key fields) and published a weekly quality scorecard.



🔴 **Result** Within 3 months, the number of "data definition" disputes in executive reporting dropped to near zero. The data dictionary became the reference for all new project onboarding. The governance framework was formally adopted as the enterprise standard for the J&J Global Services data organisation.

* * *

_End of Document_ _50 STAR-Format Technical Aptitude Interview Questions | Senior Data Analyst / Business Data Analyst_ _Tailored to: EDW / Banking / SQL focus | Based on Shajeeb's career history_ _Generated: April 2026_
