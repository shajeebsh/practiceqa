---
title: "sql-pandas-basic-coding"
date: 2026-04-28 18:50:19 
author: shajeebhameed
layout: page
slug: sql-pandas-basic-coding__trashed
status: trash
---

## Python & SQL — Basic Practice Challenges with Answers

**Insurance Claims Dataset · 50 challenges · Beginner to Intermediate**

* * *

## How to use this guide

Each challenge has:

  * A clear **scenario** (what the business is asking)
  * A **hint** to guide your thinking
  * A full **answer** in SQL and/or Python
  * A **what to remember** note for the interview



Work through SQL and Python sections side by side — they cover the same concepts in both languages.

* * *

## Synthetic data setup

Run this once at the top of every Python session. All challenges use this dataset.
    
    
    import pandas as pd
    import numpy as np
    import sqlite3
    import warnings
    warnings.filterwarnings('ignore')
    
    # ── Sample data ──────────────────────────────────────────────
    records = [
        ('CLM001','CUST01','Motor', 'Open',  'ADJ01','Dublin',  'PRV01', 5200.00, 6000.00, 0.00,    '2024-01-15','2024-01-10', None,         0, 1, 1800.00),
        ('CLM002','CUST02','Property','Closed','ADJ02','Cork',   'PRV02',15600.00,16000.00,14800.00, '2023-04-05','2023-03-20','2023-12-15', 0, 1, 3800.00),
        ('CLM003','CUST03','Health', 'Closed','ADJ03','Galway',  None,    2800.00, 3000.00, 2600.00, '2023-05-09','2023-05-05','2023-12-01', 0, 0,  950.00),
        ('CLM004','CUST04','Motor', 'Closed','ADJ01','Dublin',  'PRV03',14335.00, 5579.00,  341.00, '2023-01-19','2023-01-15','2023-10-21', 0, 5, 2846.00),
        ('CLM005','CUST05','Liability','Open','ADJ02','Limerick','PRV01',44000.00,46000.00, 0.00,   '2024-01-05','2023-12-01', None,         1, 5, 4900.00),
        ('CLM006','CUST06','Travel', 'Closed','ADJ04','Dublin',  'PRV04',  900.00,  950.00,  850.00,'2023-08-05','2023-08-03','2023-10-15', 0, 0,  500.00),
        ('CLM007','CUST07','Motor', 'Open',  'ADJ03','Cork',    'PRV02', 8100.00, 8500.00,  0.00,   '2024-04-12','2024-04-10', None,         0, 6, 2700.00),
        ('CLM008','CUST08','Property','Closed','ADJ05','Waterford',None, 22300.00,23000.00,21000.00,'2023-03-28','2023-03-25','2023-11-10', 0, 2, 4500.00),
        ('CLM009','CUST09','Health', 'Pending','ADJ01','Galway','PRV01',  3400.00, 3600.00,  800.00,'2023-09-18','2023-09-10', None,         0, 0, 1100.00),
        ('CLM010','CUST10','Motor', 'Closed','ADJ02','Dublin',  'PRV03', 9200.00, 9800.00, 8700.00,'2023-08-22','2023-08-20','2024-04-10', 0, 2, 2700.00),
        ('CLM011','CUST01','Liability','Open','ADJ04','Dublin',  'PRV02',29500.00,30000.00, 0.00,   '2024-02-14','2024-02-10', None,         1, 4, 4800.00),
        ('CLM012','CUST02','Motor', 'Closed','ADJ05','Cork',    'PRV01', 4200.00, 4400.00, 3900.00,'2023-07-16','2023-07-14','2024-03-22', 0, 0, 1800.00),
        ('CLM013','CUST03','Property','Open','ADJ01','Galway',  'PRV03',35000.00,37000.00, 0.00,   '2024-01-30','2024-01-28', None,         1, 3, 4600.00),
        ('CLM014','CUST04','Health', 'Closed','ADJ02','Dublin',  None,    5500.00, 5800.00, 5100.00,'2023-10-29','2023-10-25','2024-06-15', 0, 3, 2200.00),
        ('CLM015','CUST05','Motor', 'Closed','ADJ03','Limerick','PRV04', 7800.00, 8200.00, 7100.00,'2024-03-10','2024-03-08','2024-11-01', 1, 2, 2300.00),
        ('CLM016','CUST06','Property','Closed','ADJ04','Cork',   'PRV02',6700.00, 7000.00, 6200.00,'2023-06-14','2023-06-10','2024-01-20', 0, 1, 2100.00),
        ('CLM017','CUST07','Liability','Open','ADJ05','Cork',    None,   55000.00,58000.00, 0.00,   '2024-03-25','2024-03-20', None,         1, 4, 4900.00),
        ('CLM018','CUST08','Motor', 'Closed','ADJ01','Waterford','PRV03',6300.00, 6600.00, 5900.00,'2023-09-11','2023-09-09','2024-05-08', 0, 2, 2400.00),
        ('CLM019','CUST09','Health', 'Pending','ADJ02','Galway','PRV01', 4100.00, 4300.00,  900.00,'2023-11-14','2023-11-10', None,         0, 0, 1300.00),
        ('CLM020','CUST10','Motor', 'Open',  'ADJ03','Dublin',  'PRV04', None,    8700.00,  0.00,   '2024-05-03','2024-05-01', None,         0, 0, 1500.00),
        # DQ issues injected
        ('CLM021','CUST01','Motor', 'Open',  'ADJ01','Dublin',  'PRV01', -500.00, 2000.00,  0.00,   '2024-03-15','2024-03-14', None,         0, 0,  800.00),
        ('CLM022','CUST02','Liability','Closed','ADJ02','Cork', 'PRV02',17907.00, 1834.00, 4900.00,'2026-08-15','2023-08-01','2023-04-03', 0, 3, 4746.00),
        ('CLM023','CUST03','Health', 'Closed','ADJ03','Galway', None,    6798.00, 2977.00, 1238.00,'2023-06-01','2023-06-01','2022-01-01', 1, 2, 4820.00),
        # Duplicate
        ('CLM001','CUST01','Motor', 'Open',  'ADJ01','Dublin',  'PRV01', 5200.00, 6000.00,  0.00,  '2024-01-15','2024-01-10', None,         0, 1, 1800.00),
    ]
    
    cols = ['claim_id','claimant_id','claim_type','claim_status','adjuster_id',
            'region','provider_id','claim_amount','reserve_amount','paid_amount',
            'fnol_date','loss_date','closure_date','fraud_flag','num_prev_claims','premium']
    
    df = pd.DataFrame(records, columns=cols)
    df['fnol_date']    = pd.to_datetime(df['fnol_date'])
    df['loss_date']    = pd.to_datetime(df['loss_date'])
    df['closure_date'] = pd.to_datetime(df['closure_date'], errors='coerce')
    
    # Load into SQLite for SQL practice
    conn = sqlite3.connect(':memory:')
    df.to_sql('claims', conn, index=False, if_exists='replace')
    print(f"Dataset ready: {len(df)} rows, {len(df.columns)} columns")
    print(df[['claim_id','claim_type','claim_status','claim_amount','fnol_date']].head(5).to_string())
    
    

* * *

## Part A — SQL basics (25 challenges)

* * *

### A1. Select all columns

**Scenario:** Pull every record from the claims table to review the full dataset.

**Hint:** The most basic SQL statement — SELECT everything FROM the table.
    
    
    SELECT * FROM claims;
    
    

> **What to remember:** `SELECT *` is fine for exploration. In production always name your columns explicitly to avoid breaking downstream code when new columns are added.

* * *

### A2. Select specific columns

**Scenario:** The reporting team only needs claim_id, claim_type, claim_status, and claim_amount. Return just those four columns.

**Hint:** List the column names separated by commas after SELECT.
    
    
    SELECT claim_id, claim_type, claim_status, ROUND(claim_amount, 2) AS claim_amount
    FROM claims;
    
    

> **What to remember:** Always ROUND financial amounts to 2 decimal places in output. Use AS to give columns readable names.

* * *

### A3. Filter with WHERE — single condition

**Scenario:** Show only open claims.

**Hint:** Use WHERE claim_status = 'value'. String values need single quotes.
    
    
    SELECT claim_id, claim_type, claim_amount, fnol_date
    FROM claims
    WHERE claim_status = 'Open';
    
    

> **What to remember:** SQL is case-sensitive for string values in most databases. 'Open' ≠ 'open'. Always check how your source data stores status values.

* * *

### A4. Filter with WHERE — multiple conditions (AND)

**Scenario:** Find all open Motor claims — both conditions must be true.

**Hint:** Use AND to combine conditions. Both must be true for a row to be included.
    
    
    SELECT claim_id, claimant_id, claim_amount, fnol_date
    FROM claims
    WHERE claim_status = 'Open'
      AND claim_type   = 'Motor';
    
    

> **What to remember:** AND narrows results (fewer rows). OR widens results (more rows). Always wrap each condition in parentheses when mixing AND and OR together.

* * *

### A5. Filter with WHERE — OR condition

**Scenario:** Return all claims from Dublin or Cork.

**Hint:** Use OR when either condition is enough to include the row.
    
    
    SELECT claim_id, region, claim_type, claim_amount
    FROM claims
    WHERE region = 'Dublin'
       OR region = 'Cork';
    
    

> **What to remember:** A cleaner alternative for multiple OR values on the same column is `WHERE region IN ('Dublin', 'Cork')` — see A7.

* * *

### A6. Filter with comparison operators

**Scenario:** Find all claims where claim_amount is greater than £10,000.

**Hint:** Use > for greater than. Other operators: < >= <= != (not equal).
    
    
    SELECT claim_id, claim_type, region, ROUND(claim_amount, 2) AS claim_amount
    FROM claims
    WHERE claim_amount > 10000
    ORDER BY claim_amount DESC;
    
    

> **What to remember:** Always add `ORDER BY` when looking for high/low values — otherwise the result order is unpredictable.

* * *

### A7. IN operator — multiple values

**Scenario:** Return claims from Dublin, Cork, and Galway in one query. Avoid writing three OR conditions.

**Hint:** `WHERE column IN (val1, val2, val3)` is cleaner than multiple OR conditions.
    
    
    SELECT claim_id, region, claim_type, claim_status
    FROM claims
    WHERE region IN ('Dublin', 'Cork', 'Galway')
    ORDER BY region;
    
    

> **What to remember:** `NOT IN` is the inverse — returns rows where the column does NOT match any value in the list. Careful: `NOT IN` behaves unexpectedly if the list contains a NULL.

* * *

### A8. IS NULL and IS NOT NULL

**Scenario:** Find all claims where provider_id is missing (null).

**Hint:** You cannot use `= NULL` in SQL — it never matches. Always use `IS NULL` or `IS NOT NULL`.
    
    
    -- Find nulls
    SELECT claim_id, claim_type, provider_id
    FROM claims
    WHERE provider_id IS NULL;
    
    -- Find non-nulls
    SELECT claim_id, claim_type, provider_id
    FROM claims
    WHERE provider_id IS NOT NULL;
    
    

> **What to remember:** `WHERE x = NULL` always returns 0 rows — NULL is not equal to anything, including itself. This is one of the most common SQL beginner mistakes.

* * *

### A9. ORDER BY — sort results

**Scenario:** Return all claims sorted by claim_amount from highest to lowest.

**Hint:** ORDER BY column DESC for descending. ASC (default) for ascending.
    
    
    SELECT claim_id, claim_type, ROUND(claim_amount, 2) AS claim_amount
    FROM claims
    WHERE claim_amount IS NOT NULL
    ORDER BY claim_amount DESC;
    
    

> **What to remember:** NULL values sort to the top with `DESC` in most databases (SQL Server puts them at bottom). Add `WHERE x IS NOT NULL` if you do not want nulls in the output.

* * *

### A10. LIMIT — top N rows

**Scenario:** Return the 5 most expensive claims only.

**Hint:** ORDER BY first, then LIMIT to take the top rows. Without ORDER BY, LIMIT is random.
    
    
    SELECT claim_id, claim_type, region, ROUND(claim_amount, 2) AS claim_amount
    FROM claims
    WHERE claim_amount IS NOT NULL AND claim_amount > 0
    ORDER BY claim_amount DESC
    LIMIT 5;
    
    

> **What to remember:** In SQL Server use `SELECT TOP 5` instead. In BigQuery use `LIMIT`. `ORDER BY + LIMIT` together is always the correct pattern for top-N queries.

* * *

### A11. COUNT — count rows

**Scenario:** How many total claims are in the table? How many have a non-null claim_amount?

**Hint:** `COUNT(*)` counts all rows. `COUNT(column)` skips nulls.
    
    
    SELECT
        COUNT(*)             AS total_rows,
        COUNT(claim_amount)  AS non_null_amounts,
        COUNT(*) - COUNT(claim_amount) AS null_amounts
    FROM claims;
    
    

> **What to remember:** `COUNT(*)` and `COUNT(column)` are different. `COUNT(*)` = total rows. `COUNT(column)` = non-null values in that column. This distinction is crucial for completeness checks.

* * *

### A12. SUM and AVG

**Scenario:** What is the total claim_amount and average claim_amount across all valid claims?

**Hint:** SUM() adds all values, AVG() divides sum by count. Both skip NULLs automatically.
    
    
    SELECT
        ROUND(SUM(claim_amount), 2) AS total_claimed,
        ROUND(AVG(claim_amount), 2) AS avg_claimed,
        ROUND(MIN(claim_amount), 2) AS min_claimed,
        ROUND(MAX(claim_amount), 2) AS max_claimed
    FROM claims
    WHERE claim_amount IS NOT NULL AND claim_amount > 0;
    
    

> **What to remember:** All aggregate functions (SUM, AVG, MIN, MAX, COUNT) skip NULL values. This means AVG is calculated on non-null rows only — relevant when comparing completeness.

* * *

### A13. GROUP BY — aggregate by category

**Scenario:** How many claims and what is the average claim_amount for each claim_type?

**Hint:** GROUP BY collapses rows into groups. Every column in SELECT must either be in GROUP BY or inside an aggregate function.
    
    
    SELECT
        claim_type,
        COUNT(*)                    AS claim_count,
        ROUND(AVG(claim_amount), 2) AS avg_amount,
        ROUND(SUM(claim_amount), 2) AS total_amount
    FROM claims
    WHERE claim_amount > 0
    GROUP BY claim_type
    ORDER BY total_amount DESC;
    
    

> **What to remember:** The rule: if it is not being aggregated (SUM/AVG/COUNT etc.), it must be in GROUP BY. Forgetting this rule is the most common GROUP BY error.

* * *

### A14. HAVING — filter after GROUP BY

**Scenario:** Show only claim types where the total claim_amount exceeds £20,000.

**Hint:** WHERE filters rows before grouping. HAVING filters groups after aggregation. You cannot use WHERE with aggregate functions.
    
    
    SELECT
        claim_type,
        COUNT(*)                    AS claim_count,
        ROUND(SUM(claim_amount), 2) AS total_amount
    FROM claims
    WHERE claim_amount > 0
    GROUP BY claim_type
    HAVING SUM(claim_amount) > 20000
    ORDER BY total_amount DESC;
    
    

> **What to remember:** WHERE = filters rows (before aggregation). HAVING = filters groups (after aggregation). Never use a column alias in HAVING — use the full expression.

* * *

### A15. COUNT with CASE WHEN — conditional count

**Scenario:** In a single query, count how many claims are Open, Closed, and Pending without running three separate queries.

**Hint:** Wrap a condition in CASE WHEN inside COUNT or SUM. `SUM(CASE WHEN condition THEN 1 ELSE 0 END)` is equivalent to a conditional count.
    
    
    SELECT
        SUM(CASE WHEN claim_status = 'Open'    THEN 1 ELSE 0 END) AS open_count,
        SUM(CASE WHEN claim_status = 'Closed'  THEN 1 ELSE 0 END) AS closed_count,
        SUM(CASE WHEN claim_status = 'Pending' THEN 1 ELSE 0 END) AS pending_count,
        COUNT(*) AS total
    FROM claims;
    
    

> **What to remember:** This single-pass pattern is much faster than running three separate queries on a large table. It is also the foundation of the DQ scorecard pattern.

* * *

### A16. CASE WHEN — add a label column

**Scenario:** Add a column called amount_band that labels each claim as "Minor" (under £5k), "Standard" (£5k–£20k), or "Large" (over £20k).

**Hint:** CASE WHEN ... THEN ... WHEN ... THEN ... ELSE ... END creates a conditional column.
    
    
    SELECT
        claim_id, claim_type, ROUND(claim_amount, 2) AS claim_amount,
        CASE
            WHEN claim_amount < 5000  THEN 'Minor'
            WHEN claim_amount <= 20000 THEN 'Standard'
            WHEN claim_amount > 20000 THEN 'Large'
            ELSE 'Unknown'
        END AS amount_band
    FROM claims
    WHERE claim_amount IS NOT NULL AND claim_amount > 0
    ORDER BY claim_amount;
    
    

> **What to remember:** CASE WHEN evaluates conditions in order and stops at the first match. Always add an ELSE clause to handle unexpected values — without it, unmatched rows return NULL.

* * *

### A17. DISTINCT — unique values

**Scenario:** What are the unique claim_types and unique regions in the dataset?

**Hint:** SELECT DISTINCT removes duplicate rows from the result.
    
    
    -- Unique claim types
    SELECT DISTINCT claim_type FROM claims ORDER BY claim_type;
    
    -- Unique regions
    SELECT DISTINCT region FROM claims ORDER BY region;
    
    -- Count of unique values
    SELECT COUNT(DISTINCT claim_type) AS unique_types,
           COUNT(DISTINCT region)     AS unique_regions
    FROM claims;
    
    

> **What to remember:** `COUNT(DISTINCT column)` counts unique non-null values. Essential for DQ uniqueness checks.

* * *

### A18. LIKE — pattern matching

**Scenario:** Find all claims where claim_id starts with 'CLM0'. Also find all claimants from 'CUST0'.

**Hint:** `%` matches any sequence of characters. `_` matches exactly one character.
    
    
    -- Starts with CLM0
    SELECT claim_id, claim_type FROM claims WHERE claim_id LIKE 'CLM0%';
    
    -- Contains 'Motor'
    SELECT claim_id, claim_type FROM claims WHERE claim_type LIKE '%tor%';
    
    -- Exactly 6 characters
    SELECT claim_id FROM claims WHERE claim_id LIKE 'CLM___';
    
    

> **What to remember:** Avoid leading wildcards (`LIKE '%value'`) on large tables — they prevent index usage and cause full table scans. A leading `%` is a performance anti-pattern.

* * *

### A19. BETWEEN — range filter

**Scenario:** Find all claims where claim_amount is between £5,000 and £20,000 inclusive.

**Hint:** `BETWEEN a AND b` is inclusive on both ends — equivalent to `>= a AND <= b`.
    
    
    SELECT claim_id, claim_type, ROUND(claim_amount, 2) AS claim_amount
    FROM claims
    WHERE claim_amount BETWEEN 5000 AND 20000
    ORDER BY claim_amount;
    
    

> **What to remember:** BETWEEN is always inclusive. For dates, BETWEEN '2023-01-01' AND '2023-12-31' includes both the start and end date.

* * *

### A20. Date filtering

**Scenario:** Find all claims where fnol_date is in 2023. Also find claims filed in the last 12 months.

**Hint:** Use `>=` and `<` for date ranges — more reliable than BETWEEN for dates. Use YEAR() or STRFTIME() to extract year.
    
    
    -- Claims in 2023
    SELECT claim_id, claim_type, fnol_date
    FROM claims
    WHERE fnol_date >= '2023-01-01' AND fnol_date < '2024-01-01'
    ORDER BY fnol_date;
    
    -- Year extraction (SQLite syntax)
    SELECT claim_id, fnol_date, STRFTIME('%Y', fnol_date) AS claim_year
    FROM claims
    WHERE STRFTIME('%Y', fnol_date) = '2023';
    
    

> **What to remember:** Avoid applying functions to indexed date columns in WHERE — `WHERE YEAR(fnol_date) = 2023` disables index usage. The sargable (index-friendly) version is `WHERE fnol_date >= '2023-01-01' AND fnol_date < '2024-01-01'`.

* * *

### A21. NULL handling — COALESCE and NULLIF

**Scenario:** Replace null provider_id values with the text 'No Provider'. Also prevent division by zero when calculating a ratio.

**Hint:** `COALESCE(col, default)` returns the first non-null value. `NULLIF(a, b)` returns NULL if a = b (used to prevent division by zero).
    
    
    -- Replace nulls with a default
    SELECT claim_id,
           COALESCE(provider_id, 'No Provider') AS provider_id
    FROM claims;
    
    -- Prevent division by zero
    SELECT claim_id,
           ROUND(paid_amount / NULLIF(reserve_amount, 0) * 100, 1) AS paid_pct
    FROM claims
    WHERE paid_amount IS NOT NULL;
    
    

> **What to remember:** `NULLIF(value, 0)` is the standard way to prevent division by zero — it returns NULL instead of throwing an error, and NULL divided by anything returns NULL gracefully.

* * *

### A22. JOIN — combine two tables

**Scenario:** Simulate joining claims to a reference table. Create an adjusters table and join it to claims to add adjuster names.

**Hint:** INNER JOIN returns only matching rows. LEFT JOIN keeps all rows from the left table even with no match.
    
    
    -- Create reference table inline using VALUES
    WITH adjusters(adjuster_id, adjuster_name, team) AS (
        VALUES
        ('ADJ01', 'Sarah Murphy',  'Motor'),
        ('ADJ02', 'James O Brien', 'Property'),
        ('ADJ03', 'Priya Nair',    'Health'),
        ('ADJ04', 'Tom Walsh',     'Liability'),
        ('ADJ05', 'Emma Doyle',    'Complex')
    )
    SELECT c.claim_id, c.claim_type, c.claim_amount,
           a.adjuster_name, a.team
    FROM claims c
    LEFT JOIN adjusters a ON c.adjuster_id = a.adjuster_id
    ORDER BY c.claim_id;
    
    

> **What to remember:** INNER JOIN — only matching rows from both tables. LEFT JOIN — all rows from left table, NULLs where no match. When in doubt, use LEFT JOIN and check for unmatched rows with WHERE a.adjuster_id IS NULL.

* * *

### A23. Subquery in WHERE

**Scenario:** Find all claims whose claim_amount is above the overall average claim_amount.

**Hint:** A subquery in WHERE calculates a value that the outer query filters on.
    
    
    SELECT claim_id, claim_type, ROUND(claim_amount, 2) AS claim_amount
    FROM claims
    WHERE claim_amount > (
        SELECT AVG(claim_amount)
        FROM claims
        WHERE claim_amount > 0
    )
    AND claim_amount > 0
    ORDER BY claim_amount DESC;
    
    

> **What to remember:** The inner subquery runs first, produces one value, and the outer query uses it in the filter. An alternative is a CTE (WITH clause) which is often more readable.

* * *

### A24. CTE — WITH clause

**Scenario:** Rewrite the above query using a CTE. Show claims above the average alongside the average itself.

**Hint:** WITH clause_name AS (subquery) — then reference it like a table. CTEs are more readable than nested subqueries.
    
    
    WITH avg_claim AS (
        SELECT AVG(claim_amount) AS avg_amount
        FROM claims
        WHERE claim_amount > 0
    )
    SELECT c.claim_id, c.claim_type,
           ROUND(c.claim_amount, 2)  AS claim_amount,
           ROUND(a.avg_amount, 2)    AS overall_avg,
           ROUND(c.claim_amount - a.avg_amount, 2) AS above_avg_by
    FROM claims c
    CROSS JOIN avg_claim a
    WHERE c.claim_amount > a.avg_amount
    ORDER BY c.claim_amount DESC;
    
    

> **What to remember:** CTEs are defined with WITH and can be referenced multiple times in the same query. They run top-to-bottom, and later CTEs can reference earlier ones. Always prefer CTEs over deeply nested subqueries for readability.

* * *

### A25. String functions

**Scenario:** Clean up the claim_type column — convert to uppercase, extract the first 3 characters, and concatenate claim_id and claim_type.

**Hint:** UPPER(), LOWER(), SUBSTR() (or LEFT()), LENGTH(), and || (concatenation) are the core string functions.
    
    
    SELECT
        claim_id,
        claim_type,
        UPPER(claim_type)           AS type_upper,
        LOWER(claim_type)           AS type_lower,
        SUBSTR(claim_type, 1, 3)    AS type_code,
        LENGTH(claim_type)          AS type_length,
        claim_id || ' - ' || claim_type AS full_label
    FROM claims
    ORDER BY claim_type;
    
    

> **What to remember:** Concatenation operator varies by database: `||` in SQLite/PostgreSQL, `+` in SQL Server, `CONCAT()` in MySQL/BigQuery. SUBSTR(string, start, length) — start index is 1 in SQL (not 0 like Python).

* * *

## Part B — Python / Pandas basics (25 challenges)

* * *

### B1. Load and inspect a DataFrame

**Scenario:** You have just received a claims CSV file. What is the first thing you do?

**Hint:** shape, dtypes, head(), info() — run all four immediately on any new dataset.
    
    
    # Using the df already created above
    print("Shape:", df.shape)           # (rows, columns)
    print("\nColumn types:")
    print(df.dtypes)
    print("\nFirst 5 rows:")
    print(df.head())
    print("\nDataset info:")
    df.info()
    
    

**Expected output:**
    
    
    Shape: (24, 16)
    Column types:
    claim_id         object
    claim_type       object
    claim_amount    float64   ← check this is float, not object
    fnol_date       datetime64
    ...
    
    

> **What to remember:** If `claim_amount` shows as `object` (string) instead of `float64`, your aggregations will fail silently. Always check dtypes immediately. Fix with `pd.to_numeric(df['col'], errors='coerce')`.

* * *

### B2. Select columns

**Scenario:** Return only claim_id, claim_type, claim_status, and claim_amount.

**Hint:** Use `df[['col1','col2',...]]` with double brackets for multiple columns.
    
    
    # Select multiple columns
    subset = df[['claim_id', 'claim_type', 'claim_status', 'claim_amount']]
    print(subset.head(10).to_string())
    
    # Single column returns a Series
    amounts = df['claim_amount']
    print(type(amounts))   # <class 'pandas.core.series.Series'>
    
    

> **What to remember:** `df['col']` returns a Series (one column). `df[['col']]` returns a DataFrame (one column). The double bracket is not a typo — it makes a list of column names.

* * *

### B3. Filter rows — single condition

**Scenario:** Show only open claims.

**Hint:** `df[df['column'] == 'value']` creates a boolean mask and applies it.
    
    
    open_claims = df[df['claim_status'] == 'Open']
    print(f"Open claims: {len(open_claims)}")
    print(open_claims[['claim_id','claim_type','claim_amount']].to_string(index=False))
    
    

> **What to remember:** `df[mask]` — the mask is a boolean Series (True/False per row). Only True rows are returned. You can save the mask as a variable: `mask = df['claim_status'] == 'Open'` then `df[mask]`.

* * *

### B4. Filter rows — multiple conditions (AND / OR)

**Scenario:** Find open Motor claims. Also find claims in Dublin OR Cork.

**Hint:** Use `&` for AND, `|` for OR. Always wrap each condition in parentheses.
    
    
    # AND — both must be true
    motor_open = df[(df['claim_status'] == 'Open') & (df['claim_type'] == 'Motor')]
    print(f"Open Motor claims: {len(motor_open)}")
    
    # OR — either is enough
    dublin_cork = df[(df['region'] == 'Dublin') | (df['region'] == 'Cork')]
    print(f"Dublin or Cork claims: {len(dublin_cork)}")
    
    

> **What to remember:** Never use Python's `and` / `or` keywords with DataFrames — they raise errors. Always use `&` / `|`. Always use parentheses around each condition.

* * *

### B5. isin() — filter on a list of values

**Scenario:** Return claims from Dublin, Cork, and Galway without writing three OR conditions.

**Hint:** `df[df['col'].isin([val1, val2, val3])]` — equivalent to SQL `IN`.
    
    
    regions = ['Dublin', 'Cork', 'Galway']
    filtered = df[df['region'].isin(regions)]
    print(f"Claims in selected regions: {len(filtered)}")
    
    # NOT isin — equivalent of SQL NOT IN
    other_regions = df[~df['region'].isin(regions)]
    print(f"Claims in other regions: {len(other_regions)}")
    
    

> **What to remember:** `~` is the NOT operator for boolean Series. `~df['col'].isin([...])` = `NOT IN`. Much cleaner than chaining multiple `!=` conditions.

* * *

### B6. isnull() and notna() — find nulls

**Scenario:** Find all claims where claim_amount is null. Count nulls per column.

**Hint:** `df['col'].isnull()` returns True where value is NaN/None. `.sum()` counts Trues.
    
    
    # Find null claim_amounts
    null_amounts = df[df['claim_amount'].isnull()]
    print(f"Null claim_amounts: {len(null_amounts)}")
    
    # Count nulls per column
    print("\nNull count per column:")
    print(df.isnull().sum()[df.isnull().sum() > 0])
    
    # Null percentage
    print("\nNull percentage:")
    print((df.isnull().sum() / len(df) * 100).round(2)[df.isnull().sum() > 0])
    
    

> **What to remember:** `isnull()` and `isna()` are identical — pick one and stay consistent. `notna()` and `notnull()` are also identical. Use `df.isnull().sum()` as your first DQ check.

* * *

### B7. sort_values() — sort rows

**Scenario:** Sort claims by claim_amount from highest to lowest.

**Hint:** `.sort_values('col', ascending=False)` for descending.
    
    
    sorted_df = (df[df['claim_amount'].notna() & (df['claim_amount'] > 0)]
                 .sort_values('claim_amount', ascending=False))
    print(sorted_df[['claim_id','claim_type','claim_amount']].head(10).to_string(index=False))
    
    # Sort by multiple columns
    multi = df.sort_values(['claim_type', 'claim_amount'], ascending=[True, False])
    print(multi[['claim_id','claim_type','claim_amount']].head(8).to_string(index=False))
    
    

> **What to remember:** `ascending` can be a list when sorting by multiple columns — `[True, False]` means first column ascending, second descending.

* * *

### B8. head() and nlargest() / nsmallest()

**Scenario:** Return the top 5 most expensive claims and the 3 cheapest valid claims.

**Hint:** `nlargest(n, col)` is cleaner than sort + head for top-N.
    
    
    # Top 5 by claim_amount
    top5 = df[df['claim_amount'] > 0].nlargest(5, 'claim_amount')
    print("Top 5 most expensive:")
    print(top5[['claim_id','claim_type','claim_amount']].to_string(index=False))
    
    # Bottom 3
    bottom3 = df[df['claim_amount'] > 0].nsmallest(3, 'claim_amount')
    print("\nBottom 3 cheapest:")
    print(bottom3[['claim_id','claim_type','claim_amount']].to_string(index=False))
    
    

> **What to remember:** `nlargest(n, col)` is equivalent to SQL `ORDER BY col DESC LIMIT n`. It is faster than `sort_values + head` for large DataFrames because it does not sort the entire dataset.

* * *

### B9. describe() — distribution summary

**Scenario:** Get a full statistical profile of claim_amount including the 95th and 99th percentiles.

**Hint:** Pass a list to `percentiles=` parameter. Clean data first.
    
    
    clean = df[df['claim_amount'].notna() & (df['claim_amount'] > 0)]
    
    print(clean['claim_amount'].describe(percentiles=[.25, .50, .75, .95, .99]).round(2))
    
    # By claim type
    print("\nProfile by type:")
    print(clean.groupby('claim_type')['claim_amount'].describe(
        percentiles=[.5, .95]).round(2))
    
    

**Expected output:**
    
    
    count       21.00
    mean      9716.00
    median    6300.00    ← median < mean → right-skewed
    p95      44000.00
    p99      55000.00
    max      55000.00
    
    

> **What to remember:** When mean > median, the data is right-skewed. For insurance claims this is normal — a few large claims pull the mean upward. Always report both in assessments.

* * *

### B10. value_counts() — frequency table

**Scenario:** How many claims of each type are there? Show count and percentage.

**Hint:** `.value_counts()` counts unique values. Add `normalize=True` for proportions.
    
    
    # Absolute counts
    print(df['claim_type'].value_counts())
    
    # With percentage
    pct = df['claim_type'].value_counts(normalize=True).mul(100).round(1)
    print("\nClaim type distribution (%):")
    print(pct)
    
    # Include nulls
    print("\nWith nulls:")
    print(df['provider_id'].value_counts(dropna=False))
    
    

> **What to remember:** `dropna=False` includes NaN in the count — essential for completeness checks. Without it, nulls are silently excluded and your counts look cleaner than they are.

* * *

### B11. groupby() + agg() — aggregate by group

**Scenario:** Calculate count, mean, and total claim_amount for each claim_type.

**Hint:** `groupby('col')['val_col'].agg(name='function')` — use named aggregations.
    
    
    result = (df[df['claim_amount'] > 0]
              .groupby('claim_type')['claim_amount']
              .agg(
                  count='count',
                  mean=lambda x: round(x.mean(), 2),
                  total=lambda x: round(x.sum(), 2),
                  median=lambda x: round(x.median(), 2)
              )
              .reset_index()
              .sort_values('total', ascending=False))
    print(result.to_string(index=False))
    
    

> **What to remember:** `.reset_index()` after groupby converts the grouped index back into regular columns — almost always needed. Without it, claim_type is the index, not a column.

* * *

### B12. groupby() + multiple columns

**Scenario:** Calculate average claim_amount grouped by both claim_type and region.

**Hint:** Pass a list to groupby(). The result has a multi-level index — use reset_index().
    
    
    pivot = (df[df['claim_amount'] > 0]
             .groupby(['claim_type', 'region'])['claim_amount']
             .mean().round(2)
             .reset_index()
             .rename(columns={'claim_amount': 'avg_claim'}))
    print(pivot.sort_values('avg_claim', ascending=False).head(10).to_string(index=False))
    
    

> **What to remember:** Multi-column groupby is equivalent to `GROUP BY col1, col2` in SQL. The `.reset_index()` call flattens the result so claim_type and region become regular columns you can filter or sort.

* * *

### B13. assign() — add a new column

**Scenario:** Add a column called loss_ratio (paid_amount / premium × 100) and a column called claim_band ('Small' / 'Medium' / 'Large').

**Hint:** `assign()` is chainable. Lambda functions receive the full DataFrame as `x`.
    
    
    df_enriched = df.assign(
        loss_ratio=lambda x: (x['paid_amount'] / x['premium'] * 100).round(1),
        claim_band=lambda x: pd.cut(
            x['claim_amount'].fillna(0),
            bins=[0, 5000, 20000, float('inf')],
            labels=['Small', 'Medium', 'Large'],
            right=True
        )
    )
    print(df_enriched[['claim_id','claim_type','claim_amount','loss_ratio','claim_band']].to_string(index=False))
    
    

> **What to remember:** `assign()` returns a new DataFrame — it does not modify the original. Chain multiple assign() calls together: `df.assign(col1=...).assign(col2=...)`. Later lambdas in the same assign() cannot see columns created in earlier lambdas — use two separate assign() calls for that.

* * *

### B14. drop_duplicates() — remove duplicates

**Scenario:** CLM001 appears twice in the dataset. Remove the duplicate, keeping the last occurrence.

**Hint:** `drop_duplicates(subset='col', keep='last')` — subset defines what makes a row unique.
    
    
    print(f"Before: {len(df)} rows")
    print(f"Duplicate claim_ids: {df['claim_id'].duplicated().sum()}")
    
    df_clean = df.drop_duplicates(subset='claim_id', keep='last')
    print(f"After: {len(df_clean)} rows")
    
    # Find which rows are duplicates
    dupes = df[df.duplicated(subset='claim_id', keep=False)]
    print(f"\nDuplicated records:\n{dupes[['claim_id','claim_type','fnol_date']].to_string(index=False)}")
    
    

> **What to remember:** `keep='last'` keeps the last occurrence (most recent load). `keep='first'` keeps the first. `keep=False` marks ALL duplicates as True (useful for finding them). The SQL equivalent is ROW_NUMBER() OVER (PARTITION BY claim_id ORDER BY fnol_date DESC).

* * *

### B15. fillna() — handle null values

**Scenario:** Fill null claim_amounts with the median. Fill null provider_id with 'No Provider'.

**Hint:** `fillna(value)` — value can be a scalar, a dict, or a computed value.
    
    
    # Fill with median (safer than mean for skewed data)
    median_amount = df['claim_amount'].median()
    df['claim_amount_filled'] = df['claim_amount'].fillna(median_amount)
    
    # Fill with a string constant
    df['provider_filled'] = df['provider_id'].fillna('No Provider')
    
    # Fill with forward fill (use previous row's value)
    df['provider_ffill'] = df['provider_id'].fillna(method='ffill')
    
    print(df[['claim_id','claim_amount','claim_amount_filled','provider_id','provider_filled']].head(8).to_string(index=False))
    
    

> **What to remember:** For insurance claim amounts, fill with median (not mean) — the mean is inflated by large claims. Always create a new column for the filled version rather than overwriting the original — you may need the null flag later.

* * *

### B16. apply() — custom function per row/column

**Scenario:** Create a risk_label column: "High" if claim_amount > 20k, "Medium" if > 5k, else "Low".

**Hint:** `apply(lambda x: ...)` runs a function on each element (Series) or each row (DataFrame with axis=1).
    
    
    def risk_label(amount):
        if pd.isna(amount) or amount <= 0:
            return 'Unknown'
        elif amount > 20000:
            return 'High'
        elif amount > 5000:
            return 'Medium'
        else:
            return 'Low'
    
    df['risk_label'] = df['claim_amount'].apply(risk_label)
    print(df[['claim_id','claim_type','claim_amount','risk_label']].to_string(index=False))
    
    # Simpler with np.where for binary conditions
    df['high_value'] = np.where(df['claim_amount'] > 20000, 1, 0)
    
    

> **What to remember:** `apply()` is flexible but slow on large DataFrames. Prefer vectorised alternatives where possible: `pd.cut()` for binning, `np.where()` for binary conditions, `map()` for dictionary lookups.

* * *

### B17. merge() — join two DataFrames

**Scenario:** Create an adjusters reference DataFrame and join it to claims to add adjuster names.

**Hint:** `pd.merge(left, right, on='key', how='left')` — equivalent to SQL LEFT JOIN.
    
    
    adjusters = pd.DataFrame({
        'adjuster_id':   ['ADJ01','ADJ02','ADJ03','ADJ04','ADJ05'],
        'adjuster_name': ['Sarah Murphy','James O Brien','Priya Nair','Tom Walsh','Emma Doyle'],
        'team':          ['Motor','Property','Health','Liability','Complex']
    })
    
    result = pd.merge(df, adjusters, on='adjuster_id', how='left')
    print(result[['claim_id','claim_type','adjuster_id','adjuster_name','team']].head(10).to_string(index=False))
    
    # Find unmatched rows (LEFT ONLY)
    unmatched = result[result['adjuster_name'].isna()]
    print(f"\nUnmatched claims: {len(unmatched)}")
    
    

> **What to remember:** `how='left'` keeps all rows from the left DataFrame. Missing matches get NaN in right-side columns. Check for unmatched rows after every LEFT JOIN — they are referential integrity violations.

* * *

### B18. pivot_table() — cross-dimensional analysis

**Scenario:** Create a table showing total claim_amount for each region (rows) by claim_type (columns).

**Hint:** `pivot_table(index=rows, columns=cols, values=data, aggfunc=function)`.
    
    
    pivot = df[df['claim_amount'] > 0].pivot_table(
        index='region',
        columns='claim_type',
        values='claim_amount',
        aggfunc='sum',
        fill_value=0
    ).round(0)
    
    print(pivot.to_string())
    
    # Find the most expensive region+type combination
    max_val = pivot.stack().idxmax()
    print(f"\nHighest spend: {max_val[1]} claims in {max_val[0]}: £{pivot.loc[max_val[0], max_val[1]]:,.0f}")
    
    

> **What to remember:** `fill_value=0` replaces NaN with 0 for combinations with no data. Without it you get NaN where no claims exist. `pivot_table` is equivalent to `GROUP BY region, claim_type` in SQL — but the output is a matrix, not a flat table.

* * *

### B19. to_datetime() and date operations

**Scenario:** Convert fnol_date to datetime, extract year and month, and calculate how many days each open claim has been open.

**Hint:** `pd.to_datetime()` converts strings. `.dt` accessor gives date parts.
    
    
    # Convert (already done in setup, but here is the pattern)
    df['fnol_date'] = pd.to_datetime(df['fnol_date'], errors='coerce')
    
    # Extract date parts
    df['fnol_year']  = df['fnol_date'].dt.year
    df['fnol_month'] = df['fnol_date'].dt.month
    df['fnol_qtr']   = df['fnol_date'].dt.quarter
    
    # Days open for open claims
    today = pd.Timestamp('2025-04-24')
    open_c = df[df['claim_status'] == 'Open'].copy()
    open_c['days_open'] = (today - open_c['fnol_date']).dt.days
    
    print(open_c[['claim_id','fnol_date','fnol_year','fnol_month','days_open']].to_string(index=False))
    
    

> **What to remember:** Always use `errors='coerce'` when converting dates — invalid date strings become NaT (Not a Time) instead of crashing. `.dt.days` is the key method to extract integer day count from a Timedelta.

* * *

### B20. String methods — .str accessor

**Scenario:** Convert claim_type to uppercase, check which claim_ids start with 'CLM0', and extract the numeric part of claim_id.

**Hint:** `.str.upper()`, `.str.startswith()`, `.str.contains()`, `.str[start:end]` — all work on string Series.
    
    
    print(df['claim_type'].str.upper().head())
    print(df['claim_type'].str.lower().head())
    
    # Filter by string pattern
    clm0 = df[df['claim_id'].str.startswith('CLM0')]
    print(f"CLM0* claims: {len(clm0)}")
    
    # Extract numeric part of claim_id
    df['claim_num'] = df['claim_id'].str[3:].astype(int)
    print(df[['claim_id','claim_num']].head())
    
    # Contains check
    motor_type = df[df['claim_type'].str.contains('Motor', case=False, na=False)]
    
    

> **What to remember:** Always pass `na=False` to `.str.contains()` and `.str.startswith()` — without it, NaN values raise errors or produce NaN in the result. The `.str` accessor applies the string method element-wise to the whole Series.

* * *

### B21. groupby() with transform() — add group stats back

**Scenario:** Add a column showing the average claim_amount for that claim_type alongside each row. Then flag rows that are above their type's average.

**Hint:** `transform()` returns a Series with the same index as the original — perfect for adding group statistics back to individual rows.
    
    
    # Add group average to every row
    df['type_avg'] = (df[df['claim_amount'] > 0]
                      .groupby('claim_type')['claim_amount']
                      .transform('mean').round(2))
    
    # Flag above-average within type
    df['above_type_avg'] = (df['claim_amount'] > df['type_avg']).astype(int)
    
    print(df[['claim_id','claim_type','claim_amount','type_avg','above_type_avg']]
          .dropna(subset=['claim_amount']).to_string(index=False))
    
    

> **What to remember:** `transform()` vs `agg()`: `agg()` collapses to one row per group (like GROUP BY). `transform()` keeps the same number of rows and aligns the result back to the original index. SQL equivalent: `AVG(x) OVER (PARTITION BY claim_type)`.

* * *

### B22. cumsum() and cumcount() — running totals

**Scenario:** Sort claims by fnol_date and add a running total of claim_amount and a sequential claim number.

**Hint:** `cumsum()` produces a running sum. `cumcount()` produces a running count (starts at 0).
    
    
    sorted_claims = df[df['claim_amount'] > 0].sort_values('fnol_date').copy()
    sorted_claims['running_total'] = sorted_claims['claim_amount'].cumsum().round(2)
    sorted_claims['claim_seq']     = sorted_claims.groupby('claimant_id').cumcount() + 1
    
    print(sorted_claims[['claim_id','claimant_id','fnol_date','claim_amount',
                          'running_total','claim_seq']].head(12).to_string(index=False))
    
    

> **What to remember:** `cumcount() + 1` is the Pandas equivalent of `ROW_NUMBER()` in SQL. Add the `+1` because `cumcount()` starts at 0. Use `groupby().cumcount()` to restart the counter for each group.

* * *

### B23. shift() — LAG and LEAD

**Scenario:** For each claimant, find the date of their previous claim and calculate the gap in days between claims.

**Hint:** Sort by claimant_id + fnol_date first. `shift(1)` inside groupby = LAG(1). `shift(-1)` = LEAD(1).
    
    
    lag_df = df[df['fnol_date'].notna()].sort_values(['claimant_id','fnol_date']).copy()
    
    lag_df['prev_fnol']   = lag_df.groupby('claimant_id')['fnol_date'].shift(1)
    lag_df['days_since']  = (lag_df['fnol_date'] - lag_df['prev_fnol']).dt.days
    lag_df['next_fnol']   = lag_df.groupby('claimant_id')['fnol_date'].shift(-1)
    
    print(lag_df[['claim_id','claimant_id','fnol_date','prev_fnol','days_since']]
          .dropna(subset=['prev_fnol']).to_string(index=False))
    
    # Flag repeat within 30 days
    repeats = lag_df[lag_df['days_since'] <= 30]
    print(f"\nRepeat claims within 30 days: {len(repeats)}")
    
    

> **What to remember:** `shift(1) = LAG(1)` | `shift(-1) = LEAD(1)`. Always sort by the group key + order key BEFORE calling shift — the result is position-based. Forgetting to sort is the most common shift() bug.

* * *

### B24. IQR outlier detection

**Scenario:** Detect outlier claim amounts using the IQR method. Flag them rather than removing them.

**Hint:** Q1 = 25th percentile, Q3 = 75th percentile, IQR = Q3 − Q1. Fences = Q1 − 1.5×IQR and Q3 + 1.5×IQR.
    
    
    clean = df[df['claim_amount'].notna() & (df['claim_amount'] > 0)].copy()
    
    Q1  = clean['claim_amount'].quantile(0.25)
    Q3  = clean['claim_amount'].quantile(0.75)
    IQR = Q3 - Q1
    lower_fence = Q1 - 1.5 * IQR
    upper_fence = Q3 + 1.5 * IQR
    
    print(f"Q1={Q1:,.0f}  Q3={Q3:,.0f}  IQR={IQR:,.0f}")
    print(f"Lower fence={lower_fence:,.0f}  Upper fence={upper_fence:,.0f}")
    
    clean['outlier_flag'] = ((clean['claim_amount'] < lower_fence) |
                             (clean['claim_amount'] > upper_fence)).astype(int)
    
    print(f"\nOutliers: {clean['outlier_flag'].sum()}")
    print(clean[clean['outlier_flag']==1][['claim_id','claim_type','claim_amount']].to_string(index=False))
    
    

> **What to remember:** IQR is anchored to Q1 and Q3 — values that outliers cannot distort. This makes IQR robust for right-skewed insurance data where z-score would be unreliable. Always flag rather than remove in insurance — a large claim may be entirely legitimate.

* * *

### B25. Full DQ pipeline — putting it all together

**Scenario:** Write a single function that accepts a DataFrame and returns a clean DataFrame and a DQ report dictionary. This is the assessment capstone question.

**Hint:** Chain all the individual skills: dedup → validity → nulls → outliers → report.
    
    
    def run_dq_pipeline(df):
        report = {}
        result = df.copy()
    
        # Step 1: Deduplicate
        before = len(result)
        result = result.drop_duplicates(subset='claim_id', keep='last')
        report['duplicates_removed'] = before - len(result)
    
        # Step 2: Count nulls
        report['null_claim_amount']  = int(result['claim_amount'].isnull().sum())
        report['null_closure_date']  = int(result['closure_date'].isnull().sum())
    
        # Step 3: Flag validity issues
        result['flag_negative'] = (result['claim_amount'] <= 0).astype(int)
        result['flag_future_fnol'] = (result['fnol_date'] > pd.Timestamp('2025-01-01')).astype(int)
        result['flag_closure_before_fnol'] = (
            result['closure_date'].notna() & (result['closure_date'] < result['fnol_date'])
        ).astype(int)
        report['negative_amounts']     = int(result['flag_negative'].sum())
        report['future_fnol_dates']    = int(result['flag_future_fnol'].sum())
        report['closure_before_fnol']  = int(result['flag_closure_before_fnol'].sum())
    
        # Step 4: IQR outlier flag
        clean_mask = result['claim_amount'].notna() & (result['claim_amount'] > 0)
        Q1 = result.loc[clean_mask, 'claim_amount'].quantile(0.25)
        Q3 = result.loc[clean_mask, 'claim_amount'].quantile(0.75)
        IQR = Q3 - Q1
        result['flag_outlier'] = (
            clean_mask & ((result['claim_amount'] > Q3 + 1.5*IQR) |
                          (result['claim_amount'] < Q1 - 1.5*IQR))
        ).astype(int)
        report['outliers_flagged'] = int(result['flag_outlier'].sum())
    
        # Step 5: Clean subset ready for analysis
        clean_df = result[
            result['claim_amount'].notna() &
            (result['claim_amount'] > 0) &
            (result['flag_future_fnol'] == 0) &
            (result['flag_closure_before_fnol'] == 0)
        ].copy()
        report['clean_rows_for_analysis'] = len(clean_df)
    
        return clean_df, report
    
    
    clean_df, report = run_dq_pipeline(df)
    print("DQ Report:")
    for k, v in report.items():
        print(f"  {k}: {v}")
    print(f"\nClean DataFrame: {len(clean_df)} rows ready for analysis")
    
    

**Expected output:**
    
    
    DQ Report:
      duplicates_removed: 1
      null_claim_amount: 1
      null_closure_date: 13
      negative_amounts: 1
      future_fnol_dates: 1
      closure_before_fnol: 1
      outliers_flagged: 2
      clean_rows_for_analysis: 19
    
    

> **What to remember:** Writing a reusable pipeline function is production thinking. It can be unit tested, scheduled, and versioned. The report dictionary gives stakeholders a summary of what was found without needing to read code. This pattern is exactly what EXL builds for insurance clients.

* * *

## Quick reference — SQL to Python mapping

Concept| SQL| Python / Pandas  
---|---|---  
Select all| `SELECT *`| `df`  
Select columns| `SELECT col1, col2`| `df[['col1','col2']]`  
Filter rows| `WHERE x = 'val'`| `df[df['x'] == 'val']`  
AND| `AND`| `&` (with parentheses)  
OR| `OR`| `|` (with parentheses)  
IN list| `WHERE x IN (a,b,c)`| `df[df['x'].isin([a,b,c])]`  
NOT IN| `WHERE x NOT IN (...)`| `df[~df['x'].isin([...])]`  
IS NULL| `WHERE x IS NULL`| `df[df['x'].isnull()]`  
IS NOT NULL| `WHERE x IS NOT NULL`| `df[df['x'].notna()]`  
Sort| `ORDER BY x DESC`| `.sort_values('x', ascending=False)`  
Top N| `LIMIT 5`| `.head(5)` or `.nlargest(5,'col')`  
Count| `COUNT(*)`| `len(df)` or `.count()`  
Count non-null| `COUNT(col)`| `df['col'].count()`  
Sum| `SUM(col)`| `df['col'].sum()`  
Average| `AVG(col)`| `df['col'].mean()`  
Group by| `GROUP BY col`| `.groupby('col')`  
Filter groups| `HAVING SUM(x) > 100`| filter after `.agg()`  
Distinct| `SELECT DISTINCT`| `.unique()` or `.nunique()`  
Case when| `CASE WHEN ... THEN ...`| `np.where()` or `pd.cut()`  
Join| `LEFT JOIN b ON a.id = b.id`| `pd.merge(a, b, on='id', how='left')`  
Add column| `col AS alias`| `.assign(col=...)`  
Pattern match| `LIKE '%val%'`| `.str.contains('val')`  
Replace null| `COALESCE(col, default)`| `.fillna(default)`  
Running total| `SUM() OVER (ORDER BY ...)`| `.cumsum()`  
Row number| `ROW_NUMBER() OVER (...)`| `.groupby().cumcount() + 1`  
Lag| `LAG(col,1) OVER (...)`| `.groupby().shift(1)`  
Lead| `LEAD(col,1) OVER (...)`| `.groupby().shift(-1)`  
Rank| `DENSE_RANK() OVER (...)`| `.rank(method='dense')`  
Percentile rank| `PERCENT_RANK() OVER (...)`| `.rank(pct=True)`  
Quartile band| `NTILE(4) OVER (...)`| `pd.qcut(col, q=4)`  
Rolling avg| `AVG() OVER (ROWS BETWEEN 2 ...)`| `.rolling(3).mean()`  
Dedup| `ROW_NUMBER() ... WHERE rn=1`| `.drop_duplicates(subset='col', keep='last')`  
  
* * *

## Key phrases for your assessment

> "I always run `df.info()` and `df.describe()` first — the dtype check and null count give me the most important DQ signals before writing a single query."

> "I use `COALESCE` in SQL and `fillna()` in Python when I need to replace nulls — but I only fill after I have profiled why they are null. Sometimes a null is correct pipeline behaviour."

> "In SQL I use `SUM(CASE WHEN condition THEN 1 ELSE 0 END)` to count multiple conditions in a single table scan — this is much faster than separate queries."

> "I use `&` and `|` in Pandas with parentheses around each condition — never Python's `and`/`or` keywords, which raise errors on DataFrames."

> "For top-N queries I always pair `ORDER BY + LIMIT` in SQL, and `nlargest()` or `sort_values + head()` in Pandas. Never use LIMIT without ORDER BY — the result order is undefined."
