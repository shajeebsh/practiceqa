---
title: "Data profiling"
date: 2026-04-08 10:06:41 
author: shajeebhameed
layout: page
slug: data-profiling
status: publish
---

> Claude [link](<https://claude.ai/share/3281eb24-e261-44a4-ac1f-f537381d182d>)

Here's a realistic, interactive P&C Insurance Claims data profiling report — the kind you'd actually produce and present to a data governance team:

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-4.png)

This is a realistic P&C Insurance Claims data profiling report — exactly the kind you'd build and present in this role. Here's what each section shows:

**Top KPIs** — overall DQ score, failed rules, warnings, and rows scanned at a glance.

**DQ radar chart** — scores across the four standard dimensions (Completeness, Accuracy, Consistency, Timeliness, Validity). Timeliness at 79% is the weakest — a common finding in insurance where FNOL reporting lags.

**Column-level table** — you can filter by Pass / Warning / Fail. Notice `LOSS_RESERVE_AMT` at 11.4% null and `BROKER_CODE` at 14.6% null — exactly the kind of issue you described in your interview answer for Q6.

**Null distribution bar chart** — colour-coded red/amber/green so a stakeholder can spot the worst columns instantly.

**Critical issues panel** — each issue has a root cause, business impact, and remediation recommendation. This is what you'd walk an actuarial or governance team through in a sprint review.

In an interview, if they ask "what does your profiling output look like?" — this is the answer. You'd typically build this in Power BI or a Fabric notebook and schedule it to refresh after each CDC batch run.

Q1: Explain the biggest data quality issues in this profiling report and how to fix them

Based on the profiling report, here are the five biggest data quality issues ranked by business impact, with detailed fixes for each:

* * *

### 1\. LOSS_RESERVE_AMT — 11.4% null (Critical)

**The problem:** 142,402 settled and closed claims have NULL or zero reserve amounts. In P&C insurance, every closed claim must have a reserve history — this is a regulatory requirement under Solvency II and a core input to actuarial loss development models. The profiling report estimates this understates total reserves by ~$18.3M.

**Root cause:** The ETL extraction missed a JOIN to a separate reserve adjustment table in the legacy system. The main claims table only holds the initial reserve; all subsequent reserve movements live in a child table that was never included in the pipeline.

**How to fix it:**

  * Add the missing JOIN in the ETL: `LEFT JOIN RESERVE_ADJUSTMENTS ra ON ra.CLAIM_ID = c.CLAIM_ID` and take the latest reserve amount per claim using a `ROW_NUMBER()` window function ordered by adjustment date descending.
  * Re-extract the 142K affected records and back-fill the Silver layer.
  * Add a DQ rule to the pipeline: any record with `CLAIM_STATUS IN ('SETTLED','CLOSED') AND LOSS_RESERVE_AMT IS NULL` should fail and route to a quarantine table, never land in the Gold layer.
  * Add a daily monitoring alert so this never silently reoccurs.



* * *

### 2\. BROKER_CODE — 14.6% null (Critical)

**The problem:** 182,118 records have no broker code. Downstream commission calculation logic requires this field — without it, these records are excluded from broker reporting and commission runs entirely. This is both a financial and relationship risk with brokers.

**Root cause:** A new direct-to-consumer (DTC) sales channel was launched but no default broker code was assigned to DTC policies. The business rule was never updated to handle this channel.

**How to fix it:**

  * Work with the business to agree a reserved broker code for DTC policies — e.g. `BRK-0000` meaning "direct channel." Update the STTM to document this transformation rule.
  * Apply a CASE statement in the ETL: `COALESCE(broker_code, CASE WHEN channel = 'DIRECT' THEN 'BRK-0000' ELSE NULL END)`.
  * Add a NOT NULL validation rule to the DQ framework — any null that isn't a DTC record should fail and alert immediately.
  * Back-fill historical DTC records using the channel flag already present in the policy table.



* * *

### 3\. VEHICLE_VIN — 22.1% null (Critical)

**The problem:** 275,672 records have no VIN. Without VIN, you cannot do cross-claim vehicle matching — which is one of the most effective fraud detection signals (same vehicle appearing in multiple claims under different policyholders). This effectively creates a blind spot in your fraud detection coverage for nearly a quarter of the dataset.

**Root cause:** Pre-2022 claims were migrated from a legacy system that stored VIN in a free-text notes field rather than a structured column, so it was never extracted during the original migration.

**How to fix it:**

  * Run a regex extraction on the legacy notes field to parse VINs: VINs follow a fixed 17-character alphanumeric pattern (`[A-HJ-NPR-Z0-9]{17}`) — a Python or SQL regex can recover a significant portion.
  * For records where regex fails, cross-reference against the policy system using policy number to retrieve the vehicle record and its VIN.
  * For genuinely unrecoverable records, flag them with a `VIN_AVAILABLE = FALSE` indicator so the fraud team knows not to rely on vehicle matching for those claims.
  * Going forward, add a NOT NULL constraint at ingestion for all claim types where a vehicle is involved (auto, commercial fleet lines).



* * *

### 4\. FNOL_DATE before LOSS_DATE — 3,841 records (Critical)

**The problem:** First Notice of Loss date is earlier than the actual loss date on 3,841 records. This is a logical impossibility — you cannot report a loss before it happens. These records will produce negative claims reporting lag values and directly distort IBNR (Incurred But Not Reported) calculations, which actuaries rely on to estimate future claims liabilities.

**Root cause:** Data entry errors in the legacy claims system — likely agents entering dates in wrong fields, or system date defaulting to today's date during manual entry.

**How to fix it:**

  * Add a referential integrity rule to the DQ framework: `FNOL_DATE >= LOSS_DATE` — any violation quarantines the record and raises an alert to the claims operations team for manual correction.
  * For the 3,841 existing records, route them to a data steward review queue. In many cases the dates are transposed and can be corrected by swapping FNOL and LOSS dates — but this needs human sign-off, not automated correction, because it affects IBNR.
  * In the source system, add a front-end validation to prevent FNOL date being entered before loss date at point of claim entry.
  * Document the remediation in the STTM with a transformation rule and audit trail.



* * *

### 5\. FRAUD_FLAG — 5.2% null (Warning)

**The problem:** 64,864 claims have never been evaluated for fraud. These records silently bypass the SIU (Special Investigations Unit) referral workflow. This is not just a data quality issue — it is an operational risk, because potentially fraudulent claims may be settled without any fraud review.

**Root cause:** The fraud scoring model only runs on claims above a certain payment threshold. Claims below the threshold, or claims settled very quickly, exit the workflow before the fraud flag is set.

**How to fix it:**

  * Apply a business rule default: set `FRAUD_FLAG = FALSE` for all records where the SIU workflow has completed (i.e. the claim is closed or settled) but no flag was ever written. Add a separate `FRAUD_REVIEWED = FALSE` indicator so the absence of review is still visible.
  * Do not default NULL to FALSE for open claims — that would hide the gap. Only apply the default on closed/settled records where the review window has passed.
  * Update the fraud scoring pipeline to run on all claims regardless of payment threshold and write the result flag back within 24 hours of FNOL.
  * Add a DQ rule: any claim with `CLAIM_STATUS = 'SETTLED' AND FRAUD_FLAG IS NULL` should appear on a daily exceptions report for the SIU team.



* * *

### Summary — remediation priority

Issue| Records affected| Business impact| Fix complexity  
---|---|---|---  
LOSS_RESERVE_AMT null| 142,402| Reserve understatement $18.3M| Medium — ETL join fix  
BROKER_CODE null| 182,118| Commission errors| Low — default rule  
VEHICLE_VIN null| 275,672| Fraud detection blind spot| High — regex + cross-ref  
FNOL before LOSS date| 3,841| IBNR distortion| Medium — manual review queue  
FRAUD_FLAG null| 64,864| SIU workflow gap| Low — default rule  
  
The reserve and FNOL issues should be fixed before the next actuarial run. The broker code and fraud flag fixes are lower complexity and can be deployed as pipeline rule changes within a sprint. The VIN back-fill is a longer-running remediation project — start with the regex extraction pass as a quick win.

> STTM - Source to Target Mapping

Here's a realistic, fully interactive STTM — modelled on the kind of document you built at J&J for 200+ attributes:

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-5.png)

This is a production-grade STTM for the PC Claims Master table — exactly the kind of document you described building for 200+ attributes at J&J. Here's what each section shows:

**Header block** — captures source system (Oracle legacy), target system (Snowflake Gold layer), load type (CDC via Streams & Tasks every 15 mins), business owner, and version history. This is the first thing an auditor or actuary looks at.

**24 mapped attributes** — covering every column from the profiling report, with full transformation logic, validation rules, nullable flags, and DQ status. You can search by column name, filter by Mandatory / Conditional / Derived, or filter to DQ failures only.

**Key columns to note in an interview:**

  * `LOSS_RESERVE_AMT` (row 7) — shows the v3.2 fix: the JOIN to `CLM_RESERVE_ADJ` using `ROW_NUMBER()` to get the latest reserve. This directly addresses the $18.3M understatement issue.
  * `BROKER_CODE` (row 12) — shows the `COALESCE` defaulting logic for the DTC channel.
  * `CLAIM_STATUS` (row 6) — shows the full CASE decode from legacy 2-char codes to readable values.
  * `ETL_LOAD_DT` and `RECORD_SOURCE` (rows 23–24) — audit columns for Purview lineage tracking.



**Change log** — version-controlled with date, author, change description, and approver. This is what you mean when you say "version-controlled STTMs with traceability" in your interview.

Hit "Generate ETL SQL ↗" to get the actual Snowflake SQL code for this mapping.

> Medallion Architecture

Here's a fully interactive Medallion Architecture walkthrough — built around the AXA Insurance scenario from your CV, with a live data flow simulator you can step through:

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-6.png?w=621)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-7.png)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-8.png)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-9.png)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-10.png)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-11.png)

Here's exactly how to explain this in your interview, linked directly to your CV:

**The three layers in plain English:**

Bronze is your "never lose anything" layer — raw data lands exactly as it came from Oracle, NULLs and all, append-only. You can always replay from Bronze if something goes wrong downstream.

Silver is where intelligence happens — all 80+ DQ rules from the profiling report fire here. Records that fail (like the FNOL before LOSS_DATE issue) get quarantined and never pollute Gold. The STTM transformations execute here — broker code defaulting, status decoding, VIN regex extraction.

Gold is what the business actually sees — fully aggregated, business-logic applied, joined to the reserve adjustment table (the v3.2 STTM fix), and served directly to Power BI via Direct Lake for sub-second query performance.

**How to use the widget in your interview prep:**

Hit "Run CDC batch" to watch a 15-minute batch flow live through all four stages with real record counts. Step through the 6-step scenario to see exactly how claim CLM-1247383 flows from an Oracle FNOL entry all the way to an actuarial Power BI dashboard — including the quarantine scenario for a bad record. This is the story you told at AXA, now fully visualised.

> Overall Structural Layout of all Three Layers

I'll produce two diagrams — first the overall structural layout of all three layers, then the detailed ERD showing how the tables relate to each other within and across layers.

Here is the structural containment view showing what lives inside each layer:

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-12.png)

Now here is the ERD showing the actual column-level relationships between the key tables across all three layers — this is the kind of diagram you'd present to a data engineer or architect:

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-13.png)

Here is how to read these two diagrams together in your interview:

**Structural diagram (first)** shows the containment — what lives inside each layer, the CDC flow arrows between layers, and the quarantine branch off Silver. Click any table to go deeper.

**ERD (second)** shows the column-level relationships — the three key join paths an interviewer will ask about:

  * `CLM_MASTER_RAW` → `CLM_RESERVE_ADJ_RAW` on `CLM_ID` — this is the fix for the $18.3M reserve gap (the missing JOIN in the original ETL)
  * `PC_CLAIMS_CLEANSED` → `RESERVE_HISTORY` on `CLAIM_ID` — this is how the Silver layer builds the running reserve history from individual adjustments
  * `PC_CLAIMS_MASTER` → `BROKER_DIM` on `BROKER_CODE` — this drives the commission fact table, and `BRK-0000` is the default for DTC channel records



The `DQ_QUARANTINE` table has no downstream joins by design — failed records are isolated there and never flow to Gold without steward sign-off. That is the key governance point to emphasise in any interview question about data quality architecture.
