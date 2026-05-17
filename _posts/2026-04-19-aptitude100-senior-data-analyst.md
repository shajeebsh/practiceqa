---
title: "Aptitude100 Senior Data Analyst"
date: 2026-04-19 17:18:42 
author: shajeebhameed
layout: page
slug: aptitude100-senior-data-analyst
status: publish
---

## 🤖 100 Automation Aptitude Interview Questions & Answers

**Sass | Senior Data Analyst | STAR Format**

> **Format:** Every answer follows **S** ituation → **T** ask → **A** ction → **R** esult **Coverage:** Pipeline Automation • Reporting Automation • Data Quality Automation • Cloud & Workflow Automation • Python Scripting • API & Integration Automation • Monitoring & Alerting • Testing & Validation Automation • Process & Scheduling Automation • Behavioural / Strategy

* * *

## 📂 Section 1: Pipeline & ETL Automation (Q1–Q15)

* * *

**Q1. Describe a data pipeline you automated end-to-end. What was the impact?**

**Situation:** At Travelers Insurance, raw Medicare/Medicaid claims arrived daily from 3 source systems in CSV, JSON, and XML formats. The ingestion process was entirely manual — an analyst downloaded files, ran scripts, and loaded data by hand each morning, taking 3–4 hours.

**Task:** Fully automate the ingestion-to-analytics pipeline so the fraud team had clean data without human intervention.

**Action:**

  * Built a Bronze-Silver-Gold medallion pipeline in Snowflake using `COPY INTO` for raw ingestion, Tasks for transformation scheduling, and Streams for CDC.
  * Configured Snowflake Tasks to trigger every 15 minutes, reading new data from the stream and merging it into the Silver layer via a `MERGE` statement.
  * Set up automated DQ checks post-load: null rate checks, duplicate key detection, and business rule validation — all running as part of the Task chain.
  * Pipeline failures triggered email alerts via `SYSTEM$SEND_EMAIL`.



**Result:** End-to-end pipeline runtime reduced from 4 hours (manual) to 12 minutes (automated). Data latency cut from 24 hours to under 15 minutes. Zero manual intervention required. The fraud team gained near-real-time alerting that detected 12 high-risk accounts in the first month.

* * *

**Q2. How did you automate incremental data loading to avoid full reloads?**

**Situation:** At Travelers, the initial full-load pipeline processed 50M+ rows nightly, consuming excessive Snowflake compute credits and taking 4+ hours — unsustainable as data volumes grew.

**Task:** Replace full reloads with intelligent incremental loading using Change Data Capture.

**Action:**
    
    
    -- Snowflake Stream captures inserts, updates, deletes automatically
    CREATE OR REPLACE STREAM claims_stream ON TABLE claims_bronze;
    
    -- Scheduled Task reads only changed rows
    CREATE OR REPLACE TASK silver_claims_refresh
      WAREHOUSE = COMPUTE_WH
      SCHEDULE = '15 MINUTE'
    AS
    MERGE INTO claims_silver AS target
    USING (
      SELECT * FROM claims_stream
      WHERE METADATA$ACTION IN ('INSERT', 'UPDATE')
    ) AS source
    ON target.claim_id = source.claim_id
    WHEN MATCHED THEN UPDATE SET
      target.status      = source.status,
      target.loss_amount = source.loss_amount,
      target.updated_at  = CURRENT_TIMESTAMP()
    WHEN NOT MATCHED THEN INSERT VALUES (
      source.claim_id, source.policy_id,
      source.loss_amount, source.status,
      source.claim_date,  CURRENT_TIMESTAMP()
    );
    
    ALTER TASK silver_claims_refresh RESUME;
    
    
    
    

**Result:** Processing time dropped from 4 hours to 12 minutes. Compute costs reduced by ~65%. The CDC approach also enabled audit trails — every change was recorded with its action type and timestamp.

* * *

**Q3. How did you automate defect tracking and reporting at Google?**

**Situation:** At Google Street View (via Cognizant), engineers manually compiled bug reports each week — copying defect IDs, severities, and assignees from the internal bug tracker into a spreadsheet, then sharing via email. This took ~3 hours weekly and introduced transcription errors.

**Task:** Automate the full defect reporting pipeline from data extraction to dashboard population.

**Action:**
    
    
    import requests
    import gspread
    from google.oauth2.service_account import Credentials
    from datetime import datetime
    
    # Pull bugs from internal Bug Tracking API
    headers = {"Authorization": f"Bearer {API_TOKEN}"}
    bugs = requests.get(
        "https://internal-bugtracker/api/v1/bugs",
        headers=headers,
        params={"status": "OPEN", "project": "street-view-ingestion"}
    ).json()["bugs"]
    
    # Transform to flat rows
    rows = [[b["id"], b["title"], b["severity"], b["assignee"],
             b["created_at"], b["days_open"],
             b.get("resolution_time_hours", "")] for b in bugs]
    
    # Push to Google Sheet
    creds = Credentials.from_service_account_file("sa.json",
        scopes=["https://www.googleapis.com/auth/spreadsheets"])
    gc = gspread.authorize(creds)
    sheet = gc.open("Street View Defect Tracker").sheet1
    sheet.clear()
    sheet.append_rows(rows, value_input_option="USER_ENTERED")
    
    print(f"[{datetime.now()}] Updated {len(rows)} bugs")
    
    
    
    

Deployed as a Cloud Function triggered every 6 hours via Cloud Scheduler.

**Result:** Eliminated 3 hours/week of manual reporting. Defect data was current within 6 hours of creation vs the previous 7-day cycle. Bug-to-resolution time dropped by 18 hours; resolution efficiency improved 20%.

* * *

**Q4. Walk me through how you automated data quality checks in a pipeline.**

**Situation:** At AXA Insurance, incoming claims data had inconsistent quality — nulls in critical actuarial fields, duplicated claim IDs, and negative loss amounts that slipped through to the Gold layer and corrupted reserve calculations.

**Task:** Build an automated DQ gate that validated data at every layer and blocked bad data from progressing.

**Action:**
    
    
    from pyspark.sql.functions import col, isnan, count, when
    
    def run_dq_checks(df, layer, table_name):
        failures = []
    
        # Rule 1: Completeness — no nulls in key fields
        for key_col in ["claim_id", "policy_id", "loss_amount"]:
            null_count = df.filter(col(key_col).isNull()).count()
            if null_count > 0:
                failures.append(f"FAIL | {key_col}: {null_count} nulls in {table_name}")
    
        # Rule 2: Uniqueness — no duplicate claim IDs
        total = df.count()
        unique = df.dropDuplicates(["claim_id"]).count()
        if total != unique:
            failures.append(f"FAIL | Duplicate claim_ids: {total - unique} in {table_name}")
    
        # Rule 3: Validity — no negative loss amounts
        negatives = df.filter(col("loss_amount") < 0).count()
        if negatives > 0:
            failures.append(f"FAIL | Negative loss_amount: {negatives} rows in {table_name}")
    
        # Gate: stop pipeline if any failures
        if failures:
            for f in failures: print(f)
            raise ValueError(f"DQ gate failed at {layer} layer. Pipeline halted.")
        else:
            print(f"✓ All DQ checks passed for {table_name} at {layer} layer.")
    
    run_dq_checks(df_silver, "Silver", "claims_silver")
    
    
    
    

**Result:** Zero DQ incidents reached the Gold/BI layer post-implementation. Manual review effort reduced by 40%. The automated checks ran in under 90 seconds per layer and were logged to a pipeline audit table for compliance review.

* * *

**Q5. How did you automate schema validation across source-to-target mappings?**

**Situation:** At J&J Global Services, 4 legacy sources fed the BigQuery consolidation model. Source schemas occasionally changed without notice — new columns appeared, types changed, or fields were renamed — breaking downstream transformations silently.

**Task:** Automate schema drift detection so any change was caught before it reached analysts.

**Action:**
    
    
    from google.cloud import bigquery
    import json
    
    client = bigquery.Client()
    
    def get_schema_fingerprint(table_id):
        table = client.get_table(table_id)
        return {field.name: field.field_type for field in table.schema}
    
    def detect_schema_drift(source_table, target_table, expected_schema_path):
        current = get_schema_fingerprint(source_table)
        with open(expected_schema_path) as f:
            expected = json.load(f)
    
        added   = {k: v for k, v in current.items() if k not in expected}
        removed = {k: v for k, v in expected.items() if k not in current}
        changed = {k: (expected[k], current[k])
                   for k in current if k in expected and current[k] != expected[k]}
    
        if added or removed or changed:
            alert = f"""Schema Drift Detected on {source_table}:
      Added:   {added}
      Removed: {removed}
      Changed: {changed}"""
            send_alert(alert)  # triggers email/Slack notification
            return False
        return True
    
    # Run in Cloud Function on a schedule
    detect_schema_drift("project.raw.claims", "project.silver.claims",
                        "schemas/claims_expected.json")
    
    
    
    

**Result:** Schema drift caught within minutes of source changes rather than hours later when pipelines failed mid-run. Saved an estimated 2–3 hours of debugging per incident. The pattern was applied to all 12 source tables in the BigQuery model.

* * *

**Q6. Describe how you automated a batch data migration process.**

**Situation:** At AXA Insurance, 20M+ customer and claims records needed to be migrated from a legacy SQL Server DWH to Microsoft Fabric. Doing this as a single bulk load was too risky — any failure would require a complete re-run.

**Task:** Design an automated, resumable batch migration that could handle failures gracefully.

**Action:**
    
    
    import pyodbc
    import pandas as pd
    from delta import DeltaTable
    from pyspark.sql import SparkSession
    
    spark = SparkSession.builder.appName("AXA_Migration").getOrCreate()
    
    BATCH_SIZE = 100_000
    CHECKPOINT_TABLE = "migration_audit.batch_log"
    
    def get_last_processed_id():
        df = spark.sql(f"SELECT MAX(last_id) AS last_id FROM {CHECKPOINT_TABLE}")
        return df.collect()[0]["last_id"] or 0
    
    def log_batch(batch_num, start_id, end_id, rows, status):
        spark.sql(f"""
            INSERT INTO {CHECKPOINT_TABLE}
            VALUES ({batch_num}, {start_id}, {end_id}, {rows}, '{status}',
                    CURRENT_TIMESTAMP())
        """)
    
    last_id = get_last_processed_id()
    batch_num = 0
    
    while True:
        # Read next batch from SQL Server
        query = f"""SELECT TOP {BATCH_SIZE} * FROM claims
                    WHERE claim_id > {last_id}
                    ORDER BY claim_id"""
        batch_df = spark.read.format("jdbc").options(
            url="jdbc:sqlserver://...", query=query
        ).load()
    
        if batch_df.count() == 0:
            print("Migration complete.")
            break
    
        # Write to Delta (append)
        batch_df.write.format("delta").mode("append").save("/fabric/silver/claims")
        end_id = batch_df.agg({"claim_id": "max"}).collect()[0][0]
        log_batch(batch_num, last_id, end_id, batch_df.count(), "SUCCESS")
        last_id = end_id
        batch_num += 1
        print(f"Batch {batch_num}: {batch_df.count()} rows migrated up to ID {end_id}")
    
    
    
    

**Result:** 20M+ records migrated successfully over 3 nights. Any failure resumed from the last successful batch. Zero data loss. Post-migration reconciliation confirmed 100% row counts matched between SQL Server source and Fabric Delta target.

* * *

**Q7. How did you automate report generation and distribution?**

**Situation:** At J&J, the compliance team needed a weekly data quality report covering null rates, duplicate counts, and rule violations across 12 BigQuery tables — currently compiled manually in Excel and emailed every Friday morning.

**Task:** Fully automate the report generation, formatting, and distribution so no analyst time was needed.

**Action:**
    
    
    from google.cloud import bigquery
    import pandas as pd
    from jinja2 import Template
    import smtplib
    from email.mime.multipart import MIMEMultipart
    from email.mime.text import MIMEText
    from datetime import datetime
    
    client = bigquery.Client()
    
    # DQ metrics query
    DQ_QUERY = """
    SELECT
      table_name,
      column_name,
      COUNT(*) AS total_rows,
      COUNTIF(column_value IS NULL) AS null_count,
      ROUND(COUNTIF(column_value IS NULL) / COUNT(*) * 100, 2) AS null_pct
    FROM `project.dq_monitoring.column_stats`
    WHERE run_date = CURRENT_DATE()
    GROUP BY 1, 2
    ORDER BY null_pct DESC
    """
    
    df = client.query(DQ_QUERY).to_dataframe()
    
    # HTML report from template
    html_template = Template("""
    <h2>J&J Data Quality Report – {{ date }}</h2>
    <p>Tables monitored: {{ table_count }}</p>
    <table border="1">
      <tr><th>Table</th><th>Column</th><th>Null %</th><th>Status</th></tr>
      {% for row in rows %}
      <tr>
        <td>{{ row.table_name }}</td>
        <td>{{ row.column_name }}</td>
        <td>{{ row.null_pct }}%</td>
        <td>{{ '🔴 FAIL' if row.null_pct > 5 else '✅ PASS' }}</td>
      </tr>
      {% endfor %}
    </table>
    """)
    
    html = html_template.render(
        date=datetime.today().strftime("%d %b %Y"),
        table_count=df['table_name'].nunique(),
        rows=df.to_dict('records')
    )
    
    # Send email
    msg = MIMEMultipart("alternative")
    msg["Subject"] = f"Weekly DQ Report – {datetime.today().strftime('%d %b %Y')}"
    msg["From"] = "data-ops@jnj.com"
    msg["To"] = "compliance-team@jnj.com"
    msg.attach(MIMEText(html, "html"))
    
    with smtplib.SMTP("smtp.jnj.com") as server:
        server.send_message(msg)
    
    print("Report sent.")
    
    
    
    

Scheduled via Cloud Scheduler every Friday at 7am.

**Result:** Eliminated 2 hours of manual Friday morning work for 3 analysts. Report arrived in inboxes before the 8am compliance standup — earlier than the manual process ever achieved. Flagged issues were being actioned before the working week began.

* * *

**Q8. How have you used Snowflake Tasks and Streams to automate data workflows?**

**Situation:** At Travelers Insurance, the data engineering team was manually running SQL scripts in sequence each morning to move data through Bronze → Silver → Gold layers — error-prone and dependent on an individual being available.

**Task:** Replace the manual sequence with a fully automated, dependency-aware task chain.

**Action:**
    
    
    -- Root Task: ingest raw data every 15 mins
    CREATE OR REPLACE TASK root_ingest
      WAREHOUSE = XSMALL_WH
      SCHEDULE  = '15 MINUTE'
    AS
      COPY INTO claims_bronze
      FROM @claims_stage
      FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1);
    
    -- Child Task 1: Silver transformation (depends on root)
    CREATE OR REPLACE TASK silver_transform
      WAREHOUSE = SMALL_WH
      AFTER root_ingest
    AS
      MERGE INTO claims_silver AS t
      USING (SELECT * FROM claims_stream WHERE METADATA$ACTION = 'INSERT') AS s
      ON t.claim_id = s.claim_id
      WHEN NOT MATCHED THEN INSERT VALUES (s.*);
    
    -- Child Task 2: Gold aggregation (depends on silver)
    CREATE OR REPLACE TASK gold_aggregate
      WAREHOUSE = MEDIUM_WH
      AFTER silver_transform
    AS
      CREATE OR REPLACE TABLE claims_gold AS
      SELECT
        policy_id,
        DATE_TRUNC('month', claim_date) AS claim_month,
        COUNT(*)                        AS claim_count,
        SUM(loss_amount)                AS total_loss,
        AVG(loss_amount)                AS avg_loss
      FROM claims_silver
      GROUP BY 1, 2;
    
    -- Activate the chain
    ALTER TASK silver_transform RESUME;
    ALTER TASK gold_aggregate   RESUME;
    ALTER TASK root_ingest      RESUME;
    
    
    
    

**Result:** Full Bronze-Silver-Gold pipeline ran automatically every 15 minutes without manual intervention. Task failure logs were monitored via `TASK_HISTORY` and alerted within 10 minutes of any failure. Engineering team reclaimed ~1.5 hours/day previously spent on manual pipeline management.

* * *

**Q9. How did you automate data reconciliation between two systems?**

**Situation:** At Refinitiv (LSEG), the FX trading platform required daily reconciliation between Oracle (the source) and SQL Server (the reporting replica) to ensure no trade records were lost or corrupted in transit.

**Task:** Automate the reconciliation so discrepancies were detected and flagged without a manual count comparison each morning.

**Action:**
    
    
    import pyodbc
    import cx_Oracle
    import pandas as pd
    from datetime import date
    
    # Connect to both systems
    oracle_conn = cx_Oracle.connect("user/pass@oracle-host/FXDB")
    sqlsrv_conn = pyodbc.connect("DRIVER={ODBC Driver 17};SERVER=sql-host;DATABASE=FXDB;...")
    
    trade_date = date.today().isoformat()
    
    # Pull aggregate counts and sums from both
    oracle_query = f"""
        SELECT currency_pair, COUNT(*) AS trade_count, SUM(amount) AS total_amount
        FROM trades WHERE trade_date = DATE '{trade_date}'
        GROUP BY currency_pair"""
    
    sqlsrv_query = f"""
        SELECT currency_pair, COUNT(*) AS trade_count, SUM(amount) AS total_amount
        FROM trades WHERE trade_date = '{trade_date}'
        GROUP BY currency_pair"""
    
    df_oracle = pd.read_sql(oracle_query,  oracle_conn)
    df_sqlsrv = pd.read_sql(sqlsrv_query, sqlsrv_conn)
    
    # Join and compare
    df_recon = df_oracle.merge(df_sqlsrv, on="currency_pair",
                               suffixes=("_oracle", "_sqlsrv"))
    df_recon["count_diff"]  = df_recon["trade_count_oracle"]  - df_recon["trade_count_sqlsrv"]
    df_recon["amount_diff"] = df_recon["total_amount_oracle"] - df_recon["total_amount_sqlsrv"]
    
    discrepancies = df_recon[(df_recon["count_diff"].abs() > 0) |
                             (df_recon["amount_diff"].abs() > 0.01)]
    
    if not discrepancies.empty:
        print("⚠️ RECONCILIATION FAILURES:")
        print(discrepancies.to_string())
        send_alert(discrepancies)  # email/Slack
    else:
        print(f"✓ Reconciliation passed for {trade_date}. All pairs balanced.")
    
    
    
    

Ran via Windows Task Scheduler at 6am daily.

**Result:** Reconciliation issues reduced by 85%. Mean time to detect a discrepancy dropped from 4 hours (manual morning check) to under 10 minutes (automated alert). The C# background service achieving 99.9% consistency was validated by this automated reconciliation layer.

* * *

**Q10. How did you automate the handling of pipeline failures and retries?**

**Situation:** At Travelers, a Snowflake Task failed silently overnight due to a timeout. The fraud team discovered stale data at 8am — 6 hours after the failure. No alert had fired.

**Task:** Build an automated failure detection, alerting, and retry system so failures were caught within minutes.

**Action:**
    
    
    -- Monitoring Task: runs every 10 mins, checks freshness
    CREATE OR REPLACE TASK monitor_pipeline_health
      WAREHOUSE = XSMALL_WH
      SCHEDULE  = '10 MINUTE'
    AS
    CALL SYSTEM$SEND_EMAIL(
      'data-alerts@company.com',
      'Pipeline Alert: Claims Silver Stale',
      'Claims Silver table has not refreshed in the last 30 minutes. Check TASK_HISTORY.'
    )
    WHERE (
      SELECT DATEDIFF('minute', MAX(updated_at), CURRENT_TIMESTAMP())
      FROM claims_silver
    ) > 30;
    
    -- Auto-retry stored procedure
    CREATE OR REPLACE PROCEDURE retry_failed_task(task_name STRING)
    RETURNS STRING
    LANGUAGE JAVASCRIPT
    AS $$
      var max_retries = 3;
      for (var i = 0; i < max_retries; i++) {
        try {
          snowflake.execute({sqlText: `ALTER TASK ${TASK_NAME} SUSPEND`});
          snowflake.execute({sqlText: `ALTER TASK ${TASK_NAME} RESUME`});
          return "Retry " + (i+1) + " triggered successfully.";
        } catch(e) {
          if (i === max_retries - 1) return "All retries failed: " + e.message;
        }
      }
    $$;
    
    
    
    

**Result:** Mean time to detection dropped from 6 hours to under 10 minutes. Automated retries resolved 3 of 4 transient failures in the following 6 months without human intervention. Only genuine infrastructure issues required manual involvement.

* * *

**Q11. How did you automate data profiling at the start of a new project?**

**Situation:** At the start of every new engagement — J&J, AXA, Travelers — I received unfamiliar datasets with unknown quality levels. Manual profiling of 50+ column tables took days.

**Task:** Build a reusable automated data profiling script deployable on any new dataset within minutes.

**Action:**
    
    
    import pandas as pd
    import numpy as np
    
    def auto_profile(df, dataset_name="dataset"):
        print(f"\n{'='*60}")
        print(f"AUTO PROFILE REPORT: {dataset_name}")
        print(f"Shape: {df.shape[0]:,} rows × {df.shape[1]} columns")
        print(f"{'='*60}\n")
    
        report = []
        for col in df.columns:
            stats = {
                "Column":       col,
                "DType":        str(df[col].dtype),
                "Null_Count":   df[col].isnull().sum(),
                "Null_Pct":     round(df[col].isnull().mean() * 100, 2),
                "Unique_Count": df[col].nunique(),
                "Cardinality":  round(df[col].nunique() / len(df) * 100, 2),
            }
            if pd.api.types.is_numeric_dtype(df[col]):
                stats.update({
                    "Min":    df[col].min(),
                    "Max":    df[col].max(),
                    "Mean":   round(df[col].mean(), 2),
                    "Median": df[col].median(),
                    "Std":    round(df[col].std(), 2),
                })
            elif pd.api.types.is_object_dtype(df[col]):
                stats["Top_Value"] = df[col].mode()[0] if not df[col].mode().empty else "N/A"
                stats["Top_Freq"]  = df[col].value_counts().iloc[0] if len(df[col].value_counts()) > 0 else 0
    
            report.append(stats)
    
        df_report = pd.DataFrame(report)
        print(df_report.to_string(index=False))
        df_report.to_csv(f"{dataset_name}_profile.csv", index=False)
        return df_report
    
    profile = auto_profile(df_raw, "axa_claims_bronze")
    
    
    
    

**Result:** Profiling time for a new 50-column dataset reduced from 2 days (manual) to under 5 minutes. Used at project kick-off for every engagement since J&J. At AXA, the profile report was shared with the source system owners in the first project meeting — producing the first data quality conversation before any pipeline was built.

* * *

**Q12. How did you automate testing of ETL transformations?**

**Situation:** At J&J, the BigQuery transformation layer had 200+ business rules. Changes to one rule frequently broke other downstream transformations undetected until analysts noticed wrong numbers in reports.

**Task:** Build an automated regression test suite that caught transformation errors before they reached production.

**Action:**
    
    
    from google.cloud import bigquery
    import pandas as pd
    
    client = bigquery.Client()
    
    def run_transformation_tests():
        tests = [
            {
                "name": "Total claim count matches source",
                "query": """
                    SELECT
                      (SELECT COUNT(*) FROM `project.raw.claims`) AS source_count,
                      (SELECT COUNT(*) FROM `project.silver.claims`) AS silver_count
                """,
                "assert": lambda r: r["source_count"][0] == r["silver_count"][0]
            },
            {
                "name": "No negative loss amounts in Silver",
                "query": "SELECT COUNT(*) AS n FROM `project.silver.claims` WHERE loss_amount < 0",
                "assert": lambda r: r["n"][0] == 0
            },
            {
                "name": "All claim_ids are non-null in Gold",
                "query": "SELECT COUNT(*) AS n FROM `project.gold.claims_summary` WHERE claim_id IS NULL",
                "assert": lambda r: r["n"][0] == 0
            },
            {
                "name": "Gold total_loss equals Silver sum",
                "query": """
                    SELECT
                      (SELECT SUM(total_loss)  FROM `project.gold.claims_summary`) AS gold_sum,
                      (SELECT SUM(loss_amount) FROM `project.silver.claims`)        AS silver_sum
                """,
                "assert": lambda r: abs(r["gold_sum"][0] - r["silver_sum"][0]) < 0.01
            }
        ]
    
        passed, failed = 0, 0
        for test in tests:
            result = client.query(test["query"]).to_dataframe()
            if test["assert"](result):
                print(f"  ✅ PASS: {test['name']}")
                passed += 1
            else:
                print(f"  ❌ FAIL: {test['name']}")
                print(f"     Result: {result.to_dict()}")
                failed += 1
    
        print(f"\n{passed} passed / {failed} failed")
        if failed > 0:
            raise SystemExit("ETL tests failed. Deployment blocked.")
    
    run_transformation_tests()
    
    
    
    

**Result:** Caught 6 regression bugs in the first month that would have reached production. Transformation deployment confidence increased significantly. The test suite was integrated into the CI/CD pipeline via Jenkins, running automatically on every PR merge.

* * *

**Q13. Explain how you automated a source-to-target mapping validation.**

**Situation:** At J&J, the STTM (Source-to-Target Mapping) document defined the transformation rules for 200+ fields. Manually verifying that the BigQuery implementation matched the STTM took a full week before each release.

**Task:** Automate the STTM validation so the check ran in minutes rather than days.

**Action:**
    
    
    import pandas as pd
    from google.cloud import bigquery
    
    # STTM loaded from Excel maintained by the business
    sttm = pd.read_excel("STTM_JnJ.xlsx")
    client = bigquery.Client()
    
    def validate_sttm_field(row):
        field  = row["target_field"]
        rule   = row["transformation_rule"]
        sample = row["sample_expected_value"]
        table  = row["target_table"]
    
        query = f"SELECT {field} FROM `{table}` WHERE claim_id = 'TEST-001' LIMIT 1"
        result = client.query(query).to_dataframe()
    
        if result.empty:
            return {"field": field, "status": "NO_DATA", "actual": None}
    
        actual = str(result[field].iloc[0])
        status = "PASS" if actual == str(sample) else "FAIL"
        return {"field": field, "status": status,
                "expected": sample, "actual": actual, "rule": rule}
    
    validation_results = sttm.apply(validate_sttm_field, axis=1).tolist()
    df_results = pd.DataFrame(validation_results)
    
    failures = df_results[df_results["status"] == "FAIL"]
    print(f"STTM Validation: {len(df_results) - len(failures)}/{len(df_results)} passed")
    if not failures.empty:
        print("Failures:\n", failures.to_string())
    
    
    
    

**Result:** STTM validation time reduced from 5 days to under 20 minutes. The script ran as part of the release checklist, catching 3 mapping errors before they reached production in the final 6 months of the project. Business stakeholder confidence in the pipeline increased significantly.

* * *

**Q14. How did you automate handling of late-arriving data in your pipelines?**

**Situation:** At Travelers Insurance, source systems occasionally delivered claims files hours or days after the expected schedule due to upstream processing delays. The current pipeline had no mechanism for late data — it either missed records or required manual re-runs.

**Task:** Build a pipeline that could automatically detect and process late-arriving records without manual intervention.

**Action:**
    
    
    -- Watermark table tracks the latest processed timestamp per source
    CREATE TABLE pipeline_watermarks (
      source_name   VARCHAR,
      last_processed TIMESTAMP,
      updated_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
    );
    
    -- Late arrival detection: finds records newer than watermark
    CREATE OR REPLACE TASK process_late_arrivals
      WAREHOUSE = SMALL_WH
      SCHEDULE  = '1 HOUR'
    AS
    MERGE INTO claims_silver AS target
    USING (
      SELECT s.*
      FROM claims_bronze s
      JOIN pipeline_watermarks w ON w.source_name = 'claims_bronze'
      WHERE s.source_timestamp > w.last_processed   -- only late records
        AND s.source_timestamp <= CURRENT_TIMESTAMP()
    ) AS late_data
    ON target.claim_id = late_data.claim_id
    WHEN NOT MATCHED THEN INSERT VALUES (late_data.*);
    
    -- Update watermark after successful run
    UPDATE pipeline_watermarks
    SET last_processed = CURRENT_TIMESTAMP(),
        updated_at     = CURRENT_TIMESTAMP()
    WHERE source_name = 'claims_bronze';
    
    
    
    

**Result:** Late-arriving records were automatically ingested within 1 hour of arrival — without manual re-runs. This eliminated 3–4 ad-hoc pipeline restarts per month that previously required engineer involvement. Data completeness improved and the fraud team's morning dashboard always reflected all available data.

* * *

**Q15. How did you automate data archiving and retention management?**

**Situation:** At J&J, the BigQuery environment had no formal data retention policy. Raw tables were growing unchecked, inflating storage costs and creating compliance risk as GDPR required data older than 7 years to be deleted.

**Task:** Automate data archiving and deletion in line with retention policy without manual intervention.

**Action:**
    
    
    from google.cloud import bigquery, storage
    from datetime import datetime, timedelta
    import json
    
    bq_client  = bigquery.Client()
    gcs_client = storage.Client()
    
    RETENTION_YEARS    = 7
    ARCHIVE_BUCKET     = "jnj-data-archive"
    TABLES_TO_MANAGE   = ["raw.claims", "raw.policies", "raw.handlers"]
    
    def archive_and_delete(table_id, cutoff_date):
        # Export to GCS before deletion
        export_uri = f"gs://{ARCHIVE_BUCKET}/{table_id}/{cutoff_date}/*.parquet"
        job_config  = bigquery.ExtractJobConfig(destination_format="PARQUET")
        extract_job = bq_client.extract_table(table_id, export_uri, job_config=job_config)
        extract_job.result()
        print(f"Archived {table_id} to {export_uri}")
    
        # Delete old rows
        delete_query = f"""
            DELETE FROM `project.{table_id}`
            WHERE DATE(created_at) < '{cutoff_date}'
        """
        bq_client.query(delete_query).result()
        print(f"Deleted records before {cutoff_date} from {table_id}")
    
    cutoff = (datetime.now() - timedelta(days=365 * RETENTION_YEARS)).strftime("%Y-%m-%d")
    for table in TABLES_TO_MANAGE:
        archive_and_delete(table, cutoff)
    
    
    
    

Scheduled via Cloud Scheduler on the 1st of each month.

**Result:** Storage costs reduced by ~30% in the first quarter after implementation. GDPR compliance for data retention was automated and auditable — the archive GCS bucket served as the evidence trail for auditors. No manual intervention required.

* * *

## 📂 Section 2: Reporting & Dashboard Automation (Q16–Q28)

* * *

**Q16. How did you automate executive dashboard updates at Google?**

**Situation:** At Google Street View, R&D leadership dashboards in Looker Studio showed pipeline metrics that were manually refreshed each morning by an analyst running BigQuery queries and pasting results into a linked sheet.

**Task:** Make dashboards fully live with no manual refresh required.

**Action:**

  * Replaced the manual sheet with a BigQuery materialised view that refreshed automatically every hour via a scheduled query.
  * Connected Looker Studio directly to the BigQuery view (not the Sheet), eliminating the intermediate step entirely.
  * Set Looker Studio data source to "auto refresh" every 4 hours — sufficient for leadership decision-making cadence.
  * For near-real-time metrics (latency, defect counts), implemented a Pub/Sub → Dataflow → BigQuery streaming pipeline so data arrived within seconds of events.


    
    
    -- Scheduled BigQuery query: refreshes materialised view hourly
    CREATE OR REPLACE SCHEDULED QUERY streetview_kpi_refresh
    OPTIONS (
      schedule = 'every 1 hours',
      destination_table = 'project.dashboards.streetview_kpis'
    )
    AS
    SELECT
      DATE(publish_timestamp)           AS publish_date,
      pipeline_stage,
      COUNT(*)                          AS total_images,
      AVG(latency_minutes)              AS avg_latency_mins,
      COUNTIF(status = 'PUBLISHED')
        / COUNT(*)                      AS publish_success_rate
    FROM `project.raw.pipeline_events`
    WHERE publish_timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
    GROUP BY 1, 2;
    
    
    
    

**Result:** Dashboard refresh became fully automated. Leadership saw up-to-date data without any analyst involvement. The 2 hours/week previously spent on manual refresh was redirected to analytical work. Dashboard adoption increased as executives trusted the data was always current.

* * *

**Q17. Describe a time you automated a KPI calculation that was previously done in Excel.**

**Situation:** At J&J, the compliance team calculated 8 KPIs each month from 4 different CSV extracts — manually merging them in Excel with VLOOKUP formulas, spending ~5 hours per month and producing different results depending on who ran it.

**Task:** Automate the KPI calculation in BigQuery so it was consistent, auditable, and zero-touch.

**Action:**
    
    
    -- Single BigQuery view replacing 5 Excel sheets
    CREATE OR REPLACE VIEW `project.gold.compliance_kpis` AS
    
    WITH claims_base AS (
      SELECT * FROM `project.silver.claims`
      WHERE claim_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 12 MONTH)
    ),
    
    monthly_summary AS (
      SELECT
        DATE_TRUNC(claim_date, MONTH) AS report_month,
        COUNT(*)                       AS total_claims,
        SUM(loss_amount)               AS total_loss,
        AVG(loss_amount)               AS avg_claim_value,
        COUNTIF(status = 'FRAUD')
          / COUNT(*)                   AS fraud_rate,
        AVG(DATEDIFF(settled_date, claim_date)) AS avg_cycle_days,
        COUNTIF(loss_amount > 50000)
          / COUNT(*)                   AS large_claim_rate
      FROM claims_base
      GROUP BY 1
    )
    
    SELECT
      *,
      total_loss / LAG(total_loss) OVER (ORDER BY report_month) - 1 AS loss_mom_growth,
      fraud_rate * 100 AS fraud_rate_pct
    FROM monthly_summary
    ORDER BY report_month DESC;
    
    
    
    

Power BI connected to this view via DirectQuery — always live, never stale.

**Result:** 5 hours/month of Excel work eliminated. KPI figures became consistent across all teams (previously there were 3 different versions of the "fraud rate" depending on who ran the Excel). The view was adopted as the official compliance reporting source.

* * *

**Q18. How did you automate Power BI report refresh and distribution?**

**Situation:** At AXA Insurance, Power BI reports needed to be refreshed nightly and emailed to 6 regional actuarial leads by 7am each morning. Currently, a team member had to manually trigger the refresh and then screenshot the dashboard into an email.

**Task:** Automate the refresh, rendering, and distribution without manual involvement.

**Action:**

  * Configured **Power BI scheduled refresh** via the Power BI Service to run at 4am using the Fabric Direct Lake connection — no gateway needed.
  * Used the **Power BI REST API** to trigger a refresh programmatically and check its completion status:


    
    
    import requests
    import time
    
    # Get OAuth token
    token_response = requests.post(
        f"https://login.microsoftonline.com/{TENANT_ID}/oauth2/v2.0/token",
        data={"grant_type": "client_credentials",
              "client_id": CLIENT_ID, "client_secret": CLIENT_SECRET,
              "scope": "https://analysis.windows.net/powerbi/api/.default"}
    )
    token = token_response.json()["access_token"]
    headers = {"Authorization": f"Bearer {token}"}
    
    # Trigger refresh
    requests.post(
        f"https://api.powerbi.com/v1.0/myorg/datasets/{DATASET_ID}/refreshes",
        headers=headers
    )
    
    # Poll until complete
    while True:
        history = requests.get(
            f"https://api.powerbi.com/v1.0/myorg/datasets/{DATASET_ID}/refreshes",
            headers=headers
        ).json()
        latest = history["value"][0]
        if latest["status"] == "Completed":
            print("Refresh complete.")
            break
        elif latest["status"] == "Failed":
            raise Exception(f"Refresh failed: {latest.get('serviceExceptionJson')}")
        time.sleep(60)
    
    
    
    

  * Used Power BI **Subscribe to Report** feature to auto-email PDF snapshots to regional leads at 7am.



**Result:** Eliminated a daily manual task. Reports arrived in actuarial inboxes 30 minutes earlier than before. Zero missed refreshes in 12 months. The team member previously responsible reclaimed 30 minutes/day for analytical work.

* * *

**Q19. How have you automated the generation of data lineage documentation?**

**Situation:** At AXA Insurance, auditors required a full data lineage map showing how claims data moved from source systems through to the Power BI dashboards used in actuarial decisions — a document that had never existed and would take weeks to build manually.

**Task:** Automate lineage capture so it was always up to date without manual documentation effort.

**Action:**

  * Deployed **Microsoft Purview** to auto-scan all assets: Azure Data Lake Storage, Microsoft Fabric Lakehouse, SQL endpoints, and Power BI workspaces.
  * Configured Purview's automated scanning on a daily schedule for all registered sources.
  * Used Purview's lineage API to pull the lineage graph programmatically for the compliance report:


    
    
    from azure.purview.catalog import PurviewCatalogClient
    from azure.identity import ClientSecretCredential
    
    credential = ClientSecretCredential(TENANT_ID, CLIENT_ID, CLIENT_SECRET)
    client = PurviewCatalogClient(
        endpoint=f"https://{PURVIEW_ACCOUNT}.purview.azure.com",
        credential=credential
    )
    
    # Get lineage for a specific asset (e.g., Power BI dataset)
    lineage = client.lineage.get_lineage_graph(
        guid="<asset-guid>",
        direction="BOTH",
        depth=3
    )
    
    # Export to JSON for compliance report
    import json
    with open("lineage_report.json", "w") as f:
        json.dump(lineage, f, indent=2)
    
    print(f"Lineage captured: {len(lineage['guidEntityMap'])} assets mapped")
    
    
    
    

**Result:** Complete data lineage from SQL Server source → Bronze → Silver → Gold → Power BI captured automatically within 48 hours of deployment. Audit submission took 2 hours instead of the estimated 3 weeks. Zero manual documentation required going forward. Passed first GDPR audit with no findings.

* * *

**Q20. How did you build an automated SSD lifecycle monitoring dashboard?**

**Situation:** At Google Street View, the Street View capture fleet used SSDs in data collection vehicles. Tracking SSD health, utilisation, and replacement schedules was done via manual spreadsheets that were always out of date, leading to unexpected hardware failures mid-collection run.

**Task:** Build an automated, real-time SSD lifecycle monitoring system with predictive alerting.

**Action:**

  * Pulled SSD telemetry data (SMART metrics, utilisation %, age in days) from the fleet management API into BigQuery via a Python Cloud Function running every hour.
  * Built a BigQuery model that calculated:
    * **Days to end-of-life** based on current utilisation rate vs manufacturer lifespan
    * **Utilisation trend** using a 7-day rolling average
    * **Alert threshold:** flag SSDs projected to fail within 30 days


    
    
    # Cloud Function: pulls SSD telemetry and writes to BigQuery
    from google.cloud import bigquery
    import requests
    
    client = bigquery.Client()
    
    def pull_ssd_telemetry(event, context):
        fleet_data = requests.get(
            "https://fleet-api.internal/ssd-metrics",
            headers={"Authorization": f"Bearer {API_KEY}"}
        ).json()
    
        rows = [{
            "device_id":      d["id"],
            "utilisation_pct": d["util_pct"],
            "age_days":       d["age_days"],
            "health_score":   d["smart_health"],
            "timestamp":      d["ts"]
        } for d in fleet_data["devices"]]
    
        client.insert_rows_json("project.fleet.ssd_telemetry", rows)
    
    # BigQuery view: lifecycle projection
    """
    SELECT
      device_id,
      utilisation_pct,
      age_days,
      ROUND(1000 - age_days * (utilisation_pct / 100), 0) AS projected_days_remaining,
      CASE WHEN (1000 - age_days * (utilisation_pct / 100)) < 30
           THEN 'REPLACE SOON' ELSE 'OK' END               AS status
    FROM `project.fleet.ssd_telemetry`
    WHERE timestamp = (SELECT MAX(timestamp) FROM `project.fleet.ssd_telemetry`)
    """
    
    
    
    

**Result:** SSD-related collection failures reduced significantly as replacements became proactive rather than reactive. The dashboard was cited by operations leadership as an example of automated infrastructure monitoring done right. Maintenance planning became predictable instead of crisis-driven.

* * *

**Q21. How did you automate compliance reporting for GDPR or audit purposes?**

**Situation:** At AXA Insurance, quarterly GDPR compliance reports required manual extraction of data access logs, retention period checks, and PII field audits from 3 separate systems — taking 2 days to compile.

**Task:** Automate the compliance report generation so it was produced on-demand in minutes.

**Action:**

  * Created a unified BigQuery view joining access logs from Purview, retention metadata from the pipeline audit table, and PII field inventory from the data dictionary.
  * Built a Python script that queried the view and rendered a structured HTML compliance report.
  * Scheduled it to run on the 1st of each month, with the report automatically sent to the compliance officer.


    
    
    -- Unified compliance view
    CREATE OR REPLACE VIEW `project.compliance.quarterly_report` AS
    SELECT
      a.table_name,
      a.field_name,
      a.pii_flag,
      r.retention_days,
      r.data_older_than_threshold,
      al.last_accessed_by,
      al.last_access_timestamp,
      CASE WHEN r.data_older_than_threshold = TRUE
                AND r.deletion_confirmed = FALSE
           THEN 'ACTION REQUIRED'
           ELSE 'COMPLIANT' END AS compliance_status
    FROM `project.metadata.pii_inventory` a
    LEFT JOIN `project.metadata.retention_status` r USING (table_name)
    LEFT JOIN `project.audit.access_log_summary`  al USING (table_name, field_name)
    ORDER BY compliance_status DESC, a.table_name;
    
    
    
    

**Result:** Quarterly compliance report generation reduced from 2 days to 15 minutes. Report was always consistent and auditable. First automated report was submitted to the GDPR audit team — they noted it was the most complete compliance submission they had received from any internal team.

* * *

**Q22. How have you automated anomaly detection in datasets?**

**Situation:** At Refinitiv, large FX trade volumes meant that anomalous trades — potentially erroneous or fraudulent — could appear at any point in the day and needed to be flagged before settlement.

**Task:** Build an automated anomaly detection system that didn't require a data scientist to define every rule manually.

**Action:**
    
    
    import pandas as pd
    import numpy as np
    from scipy import stats
    
    def detect_anomalies(df, column, threshold_zscore=3.0):
        """Flag rows where value is more than N standard deviations from the mean."""
        z_scores = np.abs(stats.zscore(df[column].dropna()))
        anomaly_mask = z_scores > threshold_zscore
        df["z_score"]   = np.nan
        df.loc[df[column].notna(), "z_score"] = z_scores
        df["is_anomaly"] = anomaly_mask.reindex(df.index, fill_value=False)
        return df
    
    # Rolling window anomaly detection (more robust for time series)
    def rolling_anomaly(df, column, window=7, n_std=3):
        rolling_mean = df[column].rolling(window=window).mean()
        rolling_std  = df[column].rolling(window=window).std()
        upper = rolling_mean + (n_std * rolling_std)
        lower = rolling_mean - (n_std * rolling_std)
        df["anomaly"] = (df[column] > upper) | (df[column] < lower)
        return df
    
    df_trades = pd.read_sql("SELECT * FROM trades WHERE trade_date = CURRENT_DATE", conn)
    df_flagged = rolling_anomaly(df_trades, "amount", window=7, n_std=3)
    
    anomalies = df_flagged[df_flagged["anomaly"]]
    if not anomalies.empty:
        print(f"⚠️ {len(anomalies)} anomalous trades detected:")
        print(anomalies[["trade_id", "currency_pair", "amount"]].to_string())
        send_alert(anomalies)
    
    
    
    

**Result:** Automated anomaly detection flagged 12 suspicious trades in the first week — 3 of which were genuine errors that would have caused settlement failures. Reconciliation issues reduced by 85%. Manual review was now focused only on flagged records rather than scanning full datasets.

* * *

**Q23. How did you automate the generation of data dictionaries?**

**Situation:** At J&J, a data dictionary for the BigQuery model had never existed. Analysts spent significant time reverse-engineering what each field meant by reading transformation code — slowing down onboarding and causing incorrect usage of fields.

**Task:** Automatically generate and maintain a data dictionary directly from the BigQuery schema and STTM documentation.

**Action:**
    
    
    from google.cloud import bigquery
    import pandas as pd
    
    client = bigquery.Client()
    DATASET = "project.gold"
    
    def generate_data_dictionary(dataset_id):
        tables = list(client.list_tables(dataset_id))
        dictionary = []
    
        for table in tables:
            full_table = client.get_table(f"{dataset_id}.{table.table_id}")
            for field in full_table.schema:
                dictionary.append({
                    "Table":        table.table_id,
                    "Field":        field.name,
                    "Type":         field.field_type,
                    "Mode":         field.mode,
                    "Description":  field.description or "⚠️ Not documented",
                    "Is_PII":       "YES" if any(pii in field.name.lower()
                                    for pii in ["name","email","phone","dob","address"])
                                    else "NO"
                })
    
        df_dict = pd.DataFrame(dictionary)
    
        # Flag undocumented fields
        undoc = df_dict[df_dict["Description"] == "⚠️ Not documented"]
        print(f"Data Dictionary: {len(df_dict)} fields | {len(undoc)} undocumented")
    
        # Export
        df_dict.to_excel("data_dictionary_gold.xlsx", index=False)
        df_dict.to_html("data_dictionary_gold.html", index=False)
        return df_dict
    
    generate_data_dictionary("project.gold")
    
    
    
    

**Result:** A 200+ field data dictionary was generated in under 2 minutes. Undocumented fields were flagged for owners to fill in. After 2 weeks of crowdsourced updates, coverage reached 95%. Analyst onboarding time reduced from 3 weeks to 1 week for the data comprehension component.

* * *

**Q24. How did you automate A/B comparison of pipeline outputs after a change?**

**Situation:** At J&J, whenever a transformation rule changed in the BigQuery pipeline, the QA process required manually comparing the old and new outputs across hundreds of fields — a 2-day exercise before any release.

**Task:** Automate the before/after comparison so it ran in minutes and caught regressions automatically.

**Action:**
    
    
    from google.cloud import bigquery
    import pandas as pd
    
    client = bigquery.Client()
    
    def compare_pipeline_versions(table_a, table_b, key_col, compare_cols, sample_n=10000):
        """Compare two table snapshots and report all field-level differences."""
    
        query = f"""
        WITH a AS (SELECT {key_col}, {', '.join(compare_cols)} FROM `{table_a}` LIMIT {sample_n}),
             b AS (SELECT {key_col}, {', '.join(compare_cols)} FROM `{table_b}` LIMIT {sample_n})
        SELECT
          a.{key_col},
          {', '.join([f"a.{c} AS {c}_old, b.{c} AS {c}_new, a.{c} != b.{c} AS {c}_changed"
                      for c in compare_cols])}
        FROM a FULL OUTER JOIN b USING ({key_col})
        """
    
        df = client.query(query).to_dataframe()
        change_cols = [c for c in df.columns if c.endswith("_changed")]
        summary = {col.replace("_changed", ""): df[col].sum() for col in change_cols}
    
        print("Pipeline Version Comparison:")
        for field, change_count in summary.items():
            pct = change_count / len(df) * 100
            flag = "⚠️ " if pct > 1 else "✅ "
            print(f"  {flag}{field}: {change_count} rows changed ({pct:.1f}%)")
    
        return summary
    
    compare_pipeline_versions(
        table_a="project.silver.claims_v1_snapshot",
        table_b="project.silver.claims_v2",
        key_col="claim_id",
        compare_cols=["loss_amount", "status", "reserve_amount", "handler_id"]
    )
    
    
    
    

**Result:** Before/after comparison time reduced from 2 days to 8 minutes. Caught 2 unintended regressions in the final 6 months that would have silently changed loss reserve calculations. The tool became a standard release gate for the data engineering team.

* * *

**Q25. How did you automate monitoring of BI dashboard data freshness?**

**Situation:** At Google Street View, executives used Looker Studio dashboards for operational decisions. On two occasions, stale cached data was presented in leadership reviews without anyone realising the underlying pipeline had failed.

**Task:** Build an automated freshness check that would flag any dashboard showing data older than the agreed threshold.

**Action:**
    
    
    -- BigQuery: data freshness monitoring table
    CREATE OR REPLACE TABLE `project.monitoring.dashboard_freshness` AS
    SELECT
      'street_view_kpis'               AS dashboard_name,
      MAX(publish_timestamp)           AS latest_data_timestamp,
      CURRENT_TIMESTAMP()              AS check_timestamp,
      TIMESTAMP_DIFF(
        CURRENT_TIMESTAMP(),
        MAX(publish_timestamp),
        MINUTE
      )                                AS minutes_since_refresh,
      CASE
        WHEN TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), MAX(publish_timestamp), MINUTE) > 60
        THEN 'STALE'
        ELSE 'FRESH'
      END                              AS freshness_status
    FROM `project.raw.pipeline_events`;
    
    
    
    
    
    
    # Cloud Function: checks freshness and alerts if stale
    from google.cloud import bigquery
    import smtplib
    
    def check_dashboard_freshness(event, context):
        client = bigquery.Client()
        df = client.query("""
            SELECT * FROM `project.monitoring.dashboard_freshness`
            WHERE freshness_status = 'STALE'
        """).to_dataframe()
    
        if not df.empty:
            alert_body = df.to_string()
            send_alert("Dashboard Freshness Alert", alert_body)
            print(f"⚠️ {len(df)} stale dashboards detected. Alert sent.")
        else:
            print("✅ All dashboards are fresh.")
    
    
    
    

**Result:** Stale dashboard presentations were eliminated. The freshness check ran every 30 minutes and alerted within 10 minutes of a pipeline failure. Leadership always saw accurate data. The system was extended to cover all 6 Looker Studio dashboards in the Street View analytics portfolio.

* * *

**Q26. How have you automated data masking for PII in analytics pipelines?**

**Situation:** At AXA Insurance, the Silver layer SQL endpoint was accessible to a broad group of analysts — but it contained unmasked PII fields such as `policy_holder_name`, `date_of_birth`, and `national_id_number`. GDPR required that only authorised roles could see unmasked values.

**Task:** Automate PII masking so that the right data was automatically masked for the right roles, without two separate copies of the data.

**Action:**
    
    
    -- Snowflake Dynamic Data Masking (also applicable in BigQuery / Fabric)
    -- Step 1: Create masking policies
    CREATE OR REPLACE MASKING POLICY mask_name AS (val STRING) RETURNS STRING ->
      CASE
        WHEN CURRENT_ROLE() IN ('ACTUARIAL_ADMIN', 'DATA_ENGINEER') THEN val
        ELSE REGEXP_REPLACE(val, '.', '*')   -- mask all characters
      END;
    
    CREATE OR REPLACE MASKING POLICY mask_dob AS (val DATE) RETURNS DATE ->
      CASE
        WHEN CURRENT_ROLE() IN ('ACTUARIAL_ADMIN') THEN val
        ELSE DATE_FROM_PARTS(YEAR(val), 1, 1)  -- show year only
      END;
    
    -- Step 2: Apply to PII columns
    ALTER TABLE claims_silver
      MODIFY COLUMN policy_holder_name SET MASKING POLICY mask_name;
    
    ALTER TABLE claims_silver
      MODIFY COLUMN date_of_birth SET MASKING POLICY mask_dob;
    
    -- Analyst sees:   policy_holder_name = '****** *****'
    -- Admin sees:     policy_holder_name = 'John Murphy'
    
    
    
    

For BigQuery, implemented column-level security using **BigQuery Column-Level Encryption** and **VPC Service Controls**.

**Result:** PII masking was enforced automatically at query time — no copies of masked/unmasked data to maintain. Passed GDPR audit with zero PII exposure findings. Analyst productivity was unaffected as they could still use all non-PII fields for analysis.

* * *

**Q27. How did you automate the deployment of SQL objects in a CI/CD pipeline?**

**Situation:** At J&J, SQL changes (views, stored procedures, table definitions) were deployed manually by developers — copying scripts into BigQuery's console, running them in sequence, and hoping nothing was missed. This caused environment drift between dev, staging, and production.

**Task:** Implement automated SQL deployment via CI/CD so all environments were consistent and changes were tracked.

**Action:**

  * Stored all SQL objects in Git (Bitbucket) as `.sql` files, organised by schema and object type.
  * Wrote a Jenkins pipeline that:
    1. Detected changed SQL files on PR merge
    2. Validated syntax using BigQuery's dry-run API
    3. Applied changes to staging, ran the regression test suite, then promoted to production


    
    
    // Jenkinsfile (simplified)
    pipeline {
      agent any
      stages {
        stage('Detect Changes') {
          steps {
            script {
              changedFiles = sh(returnStdout: true,
                script: 'git diff --name-only HEAD~1 HEAD -- "*.sql"').trim().split('\n')
              echo "Changed SQL files: ${changedFiles}"
            }
          }
        }
        stage('Validate') {
          steps {
            sh '''
              for f in ${CHANGED_SQL_FILES}; do
                python validate_sql.py --file $f --project my-project
              done
            '''
          }
        }
        stage('Deploy to Staging') {
          steps { sh 'python deploy_sql.py --env staging' }
        }
        stage('Run Tests') {
          steps { sh 'python run_etl_tests.py --env staging' }
        }
        stage('Deploy to Production') {
          when { branch 'main' }
          steps { sh 'python deploy_sql.py --env production' }
        }
      }
    }
    
    
    
    

**Result:** Manual deployment errors eliminated. All 3 environments (dev/staging/prod) stayed in sync. Deployment time reduced from ~2 hours (manual) to 15 minutes (automated). Every SQL change was auditable via Git history with author, timestamp, and PR link.

* * *

**Q28. How did you automate Salesforce-to-data-warehouse data synchronisation?**

**Situation:** At J&J, Salesforce CRM data (accounts, contacts, opportunities, service cases) needed to flow daily into the BigQuery data warehouse for 360-degree customer analytics. The existing process was a manual nightly export of CSVs from Salesforce, uploaded by a team member each morning.

**Task:** Automate the Salesforce-to-BigQuery sync to run reliably without human involvement.

**Action:**

  * Built a C# service using the **Salesforce Bulk API 2.0** to query Salesforce objects incrementally (only records modified since the last run) and load them into BigQuery via the Storage Write API.


    
    
    // C# service: incremental Salesforce → BigQuery sync
    public async Task SyncSalesforceObjectAsync(string objectName, DateTime lastSync)
    {
        // Step 1: Create Bulk API job in Salesforce
        var soql = $"SELECT Id, Name, Status, LastModifiedDate " +
                   $"FROM {objectName} " +
                   $"WHERE LastModifiedDate > {lastSync:yyyy-MM-ddTHH:mm:ssZ}";
    
        var jobId = await _sfClient.CreateBulkQueryJob(soql);
        await _sfClient.WaitForJobCompletion(jobId);
        var records = await _sfClient.GetJobResults(jobId);
    
        // Step 2: Transform to BigQuery rows
        var rows = records.Select(r => new BigQueryInsertRow {
            { "sf_id",         r["Id"] },
            { "name",          r["Name"] },
            { "status",        r["Status"] },
            { "last_modified", r["LastModifiedDate"] },
            { "ingested_at",   DateTime.UtcNow }
        }).ToList();
    
        // Step 3: Upsert into BigQuery
        await _bqClient.InsertRowsAsync(_dataset, $"{objectName}_raw", rows);
    
        // Step 4: Update watermark
        await UpdateSyncWatermark(objectName, DateTime.UtcNow);
        _logger.LogInformation($"Synced {rows.Count} {objectName} records from Salesforce.");
    }
    
    
    
    

Deployed as a Windows Service triggered nightly via Task Scheduler.

**Result:** Manual CSV export process eliminated. Salesforce data arrived in BigQuery by 3am each morning — 5 hours earlier than the manual process. Data reliability improved significantly as the automated incremental sync never missed records. The service ran without intervention for the entire 30-month engagement.

* * *

## 📂 Section 3: Python & Scripting Automation (Q29–Q42)

* * *

**Q29. How did you use Python to automate repetitive data analyst tasks?**

**Situation:** At Google Street View, analysts spent 2–3 hours each Monday aggregating the previous week's pipeline performance stats from 3 BigQuery tables into a standard weekly summary email sent to 5 team leads.

**Task:** Automate the aggregation, formatting, and delivery of the weekly summary.

**Action:**
    
    
    from google.cloud import bigquery
    from datetime import datetime, timedelta
    import smtplib
    from email.mime.text import MIMEText
    
    client = bigquery.Client()
    
    def weekly_summary():
        last_week = (datetime.now() - timedelta(days=7)).strftime("%Y-%m-%d")
        today     = datetime.now().strftime("%Y-%m-%d")
    
        query = f"""
        SELECT
          pipeline_stage,
          COUNT(*)                             AS total_events,
          ROUND(AVG(latency_minutes), 1)       AS avg_latency,
          ROUND(COUNTIF(status='PUBLISHED')
                / COUNT(*) * 100, 1)           AS success_rate_pct,
          SUM(CASE WHEN status='FAILED' THEN 1 ELSE 0 END) AS failures
        FROM `project.raw.pipeline_events`
        WHERE DATE(event_timestamp) BETWEEN '{last_week}' AND '{today}'
        GROUP BY 1
        ORDER BY 1
        """
        df = client.query(query).to_dataframe()
    
        body = f"Street View Pipeline Weekly Summary ({last_week} to {today})\n\n"
        body += df.to_string(index=False)
        body += f"\n\nGenerated automatically at {datetime.now()}"
    
        msg = MIMEText(body)
        msg["Subject"] = f"Weekly Pipeline Summary – w/e {today}"
        msg["From"]    = "data-ops@google-sv.com"
        msg["To"]      = "sv-leads@google.com"
    
        with smtplib.SMTP("smtp.google.com", 587) as s:
            s.send_message(msg)
    
    weekly_summary()
    
    
    
    

**Result:** 2–3 hours/week of manual aggregation and email writing eliminated. Report arrived in team lead inboxes by 6am Monday — before the weekly planning meeting. Consistent format across all weeks made trend spotting easier.

* * *

**Q30. How did you use Python to automate large-scale data cleaning?**

**Situation:** At AXA Insurance, incoming claims files from 3 source systems had inconsistent date formats, mixed case in categorical fields, special characters in name fields, and negative values in amount columns — all of which broke downstream transformations if not cleaned.

**Task:** Build a reusable automated cleaning pipeline that handled all known issues without manual intervention.

**Action:**
    
    
    import pandas as pd
    import re
    from datetime import datetime
    
    def clean_claims_file(filepath):
        df = pd.read_csv(filepath, low_memory=False)
        original_rows = len(df)
        cleaning_log  = []
    
        # 1. Standardise date formats
        for date_col in ["claim_date", "settled_date", "reserve_date"]:
            if date_col in df.columns:
                df[date_col] = pd.to_datetime(df[date_col], errors="coerce",
                                               dayfirst=True)
                bad_dates = df[date_col].isnull().sum()
                cleaning_log.append(f"{date_col}: {bad_dates} unparseable dates → NaT")
    
        # 2. Standardise categoricals to uppercase
        for cat_col in ["status", "claim_type", "region"]:
            if cat_col in df.columns:
                df[cat_col] = df[cat_col].str.strip().str.upper()
    
        # 3. Remove special characters from name fields
        for name_col in ["policy_holder_name", "handler_name"]:
            if name_col in df.columns:
                df[name_col] = df[name_col].apply(
                    lambda x: re.sub(r"[^a-zA-Z\s\-]", "", str(x)) if pd.notna(x) else x
                )
    
        # 4. Cap negative amounts to zero
        df["loss_amount"] = df["loss_amount"].clip(lower=0)
    
        # 5. Drop exact duplicates
        before = len(df)
        df = df.drop_duplicates()
        cleaning_log.append(f"Duplicates removed: {before - len(df)}")
    
        # 6. Log summary
        print(f"Cleaned {filepath}: {original_rows} → {len(df)} rows")
        for entry in cleaning_log:
            print(f"  {entry}")
    
        # Write clean file
        out = filepath.replace(".csv", "_clean.parquet")
        df.to_parquet(out, index=False)
        return df
    
    df_clean = clean_claims_file("axa_claims_raw.csv")
    
    
    
    

**Result:** Manual pre-processing before each pipeline run eliminated. Clean file was produced in under 3 minutes for a 500K-row file. Downstream transformation failures due to data format issues reduced from weekly to zero over the following 6 months.

* * *

**Q31. Describe a Python script you built to automate data validation between two databases.**

_(See Q9 for the Refinitiv reconciliation script — this answer extends that with a Python focus.)_

**Situation:** At Travelers Insurance, the Silver layer in Snowflake needed to be validated against the Bronze layer after every load to confirm no records were dropped or corrupted during transformation.

**Task:** Automate a row-level validation that ran after each Task execution and blocked the Gold refresh if validation failed.

**Action:**
    
    
    import snowflake.connector
    import sys
    
    conn = snowflake.connector.connect(
        user=USER, password=PASSWORD,
        account=ACCOUNT, warehouse="XSMALL_WH",
        database="TRAVELERS_DW"
    )
    cursor = conn.cursor()
    
    def validate_layer(source_table, target_table, key_col, amount_col):
        cursor.execute(f"""
            SELECT
              (SELECT COUNT(*) FROM {source_table}) AS src_count,
              (SELECT COUNT(*) FROM {target_table}) AS tgt_count,
              (SELECT SUM({amount_col}) FROM {source_table}) AS src_sum,
              (SELECT SUM({amount_col}) FROM {target_table}) AS tgt_sum
        """)
        row = cursor.fetchone()
        src_count, tgt_count, src_sum, tgt_sum = row
    
        results = {
            "row_count_match":  src_count == tgt_count,
            "amount_sum_match": abs((src_sum or 0) - (tgt_sum or 0)) < 0.01
        }
    
        for check, passed in results.items():
            status = "✅ PASS" if passed else "❌ FAIL"
            print(f"  {status}: {check} | src={src_count if 'count' in check else src_sum} "
                  f"| tgt={tgt_count if 'count' in check else tgt_sum}")
    
        if not all(results.values()):
            sys.exit(1)   # Halt pipeline — Gold layer will not refresh
    
    validate_layer("BRONZE.CLAIMS", "SILVER.CLAIMS", "CLAIM_ID", "LOSS_AMOUNT")
    
    
    
    

**Result:** Automated validation ran after every Silver refresh — under 30 seconds. Caught 2 silent data loss events in 12 months caused by Snowflake Task timeouts mid-run, both before they reached the Gold layer or impacted any reports.

* * *

**Q32. How did you automate API-based data ingestion?**

**Situation:** At Google Street View, operational data from the internal fleet management system was only accessible via a REST API — no direct database connection was available. An analyst was manually calling the API and downloading JSON files daily.

**Task:** Automate the API ingestion pipeline to run on a schedule with error handling and retry logic.

**Action:**
    
    
    import requests
    import time
    import json
    from google.cloud import bigquery
    from datetime import datetime
    
    client = bigquery.Client()
    
    def fetch_with_retry(url, headers, params, max_retries=3, backoff=2):
        """Fetch API data with exponential backoff on failure."""
        for attempt in range(max_retries):
            try:
                resp = requests.get(url, headers=headers, params=params, timeout=30)
                resp.raise_for_status()
                return resp.json()
            except requests.exceptions.RequestException as e:
                if attempt < max_retries - 1:
                    wait = backoff ** attempt
                    print(f"Attempt {attempt+1} failed: {e}. Retrying in {wait}s...")
                    time.sleep(wait)
                else:
                    raise
    
    def ingest_fleet_data():
        headers = {"Authorization": f"Bearer {API_TOKEN}"}
        all_records = []
    
        # Paginate through all fleet records
        page, page_size = 1, 500
        while True:
            data = fetch_with_retry(
                "https://fleet-api.internal/v1/vehicles",
                headers=headers,
                params={"page": page, "size": page_size}
            )
            records = data.get("vehicles", [])
            if not records:
                break
    
            all_records.extend([{
                "vehicle_id":    r["id"],
                "ssd_health":    r["ssd"]["health_score"],
                "utilisation":   r["ssd"]["utilisation_pct"],
                "last_seen":     r["last_heartbeat"],
                "ingested_at":   datetime.utcnow().isoformat()
            } for r in records])
    
            page += 1
            if len(records) < page_size:
                break
    
        # Load to BigQuery
        errors = client.insert_rows_json("project.fleet.vehicle_telemetry", all_records)
        if errors:
            raise RuntimeError(f"BigQuery insert errors: {errors}")
        print(f"Ingested {len(all_records)} vehicle records.")
    
    ingest_fleet_data()
    
    
    
    

Deployed as a Cloud Function triggered by Cloud Scheduler every hour.

**Result:** Manual API download eliminated. Fleet data was always available within 1 hour of real-world events. API retry logic handled intermittent failures gracefully — no manual re-runs required in 6 months of operation.

* * *

**Q33. How did you automate PySpark data processing for large-scale ingestion?**

**Situation:** At AXA Insurance, claims files arrived daily as 5GB+ CSVs. Processing them with pandas crashed the notebook environment. The team was splitting files manually into chunks before loading.

**Task:** Automate large-scale ingestion using PySpark in Microsoft Fabric so files of any size were handled without memory issues.

**Action:**
    
    
    from pyspark.sql import SparkSession
    from pyspark.sql.functions import col, to_date, when, lit, current_timestamp
    from pyspark.sql.types import *
    
    spark = SparkSession.builder \
        .appName("AXA_Claims_Ingestion") \
        .config("spark.sql.shuffle.partitions", "200") \
        .config("spark.sql.adaptive.enabled", "true") \
        .getOrCreate()
    
    # Explicit schema = faster than inferSchema
    schema = StructType([
        StructField("claim_id",    StringType(),  False),
        StructField("policy_id",   StringType(),  False),
        StructField("claim_date",  StringType(),  True),
        StructField("loss_amount", DoubleType(),  True),
        StructField("status",      StringType(),  True),
        StructField("region",      StringType(),  True),
    ])
    
    # Read entire folder — handles multiple daily files at once
    df_raw = spark.read.csv(
        "abfss://raw@axa-storage.dfs.core.windows.net/claims/",
        schema=schema, header=True
    )
    
    # Transform
    df_clean = df_raw \
        .withColumn("claim_date",  to_date(col("claim_date"), "yyyy-MM-dd")) \
        .withColumn("loss_amount", when(col("loss_amount") < 0, lit(0.0))
                                   .otherwise(col("loss_amount"))) \
        .withColumn("status",      col("status").cast("string").alias("status")) \
        .withColumn("ingested_at", current_timestamp()) \
        .dropDuplicates(["claim_id"]) \
        .filter(col("claim_id").isNotNull())
    
    # Write as partitioned Delta (incremental append)
    df_clean.write \
        .format("delta") \
        .mode("append") \
        .partitionBy("claim_date") \
        .option("mergeSchema", "true") \
        .save("abfss://silver@axa-storage.dfs.core.windows.net/claims_clean/")
    
    print(f"Processed {df_clean.count():,} records.")
    
    
    
    

**Result:** Processing time for a 5GB daily file reduced from 40+ minutes (pandas with manual splitting) to under 4 minutes (PySpark). The pipeline handled any file size without memory errors. Manual file splitting was eliminated entirely.

* * *

**Q34. How did you automate data type enforcement across a pipeline?**

**Situation:** At J&J, source systems occasionally sent `loss_amount` as a string (e.g., `"1,234.56"`) instead of a numeric type, silently turning SUM aggregations into NULL in BigQuery.

**Task:** Automate data type detection and enforcement before any transformation ran, so type mismatches were caught at ingestion.

**Action:**
    
    
    import pandas as pd
    from google.cloud import bigquery
    
    EXPECTED_TYPES = {
        "claim_id":    "STRING",
        "policy_id":   "STRING",
        "loss_amount": "FLOAT64",
        "claim_date":  "DATE",
        "status":      "STRING",
        "region":      "STRING"
    }
    
    def enforce_types(df, expected_types):
        issues = []
        for col, expected in expected_types.items():
            if col not in df.columns:
                issues.append(f"MISSING: {col}")
                continue
    
            if expected == "FLOAT64":
                # Strip commas and cast
                df[col] = df[col].astype(str).str.replace(",", "").str.replace("£", "")
                df[col] = pd.to_numeric(df[col], errors="coerce")
                coerced = df[col].isnull().sum()
                if coerced > 0:
                    issues.append(f"TYPE COERCION: {col} — {coerced} values could not be converted to FLOAT64")
    
            elif expected == "DATE":
                df[col] = pd.to_datetime(df[col], errors="coerce", dayfirst=True)
                unparseable = df[col].isnull().sum()
                if unparseable > 0:
                    issues.append(f"DATE PARSE FAIL: {col} — {unparseable} unparseable dates → NaT")
    
        if issues:
            for i in issues: print(f"  ⚠️  {i}")
        else:
            print("  ✅ All types enforced successfully.")
        return df, issues
    
    df, issues = enforce_types(df_raw, EXPECTED_TYPES)
    
    
    
    

**Result:** Type-related transformation failures eliminated. The 6 incidents per quarter where `loss_amount` arrived as a string were caught automatically at ingestion — no downstream impact. Analysts stopped receiving "unexpected NULL" queries in their aggregation reports.

* * *

**Q35. How did you automate log analysis to identify pipeline performance trends?**

**Situation:** At Google Street View, the engineering team wanted to understand whether pipeline latency was trending up or down over time — but the only data was raw log files in Cloud Storage that no one had time to analyse manually.

**Task:** Build an automated log analysis pipeline that turned raw logs into actionable performance trends.

**Action:**
    
    
    from google.cloud import storage, bigquery
    import re
    from datetime import datetime
    
    gcs_client = storage.Client()
    bq_client  = bigquery.Client()
    
    def parse_log_line(line):
        """Extract timestamp, stage, status, and latency from a structured log line."""
        pattern = r'\[(\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2})\] STAGE=(\w+) STATUS=(\w+) LATENCY_MS=(\d+)'
        match = re.search(pattern, line)
        if match:
            return {
                "timestamp":  match.group(1),
                "stage":      match.group(2),
                "status":     match.group(3),
                "latency_ms": int(match.group(4))
            }
        return None
    
    def process_log_bucket(bucket_name, prefix):
        rows = []
        bucket = gcs_client.bucket(bucket_name)
        for blob in bucket.list_blobs(prefix=prefix):
            content = blob.download_as_text()
            for line in content.splitlines():
                parsed = parse_log_line(line)
                if parsed:
                    rows.append(parsed)
    
        if rows:
            errors = bq_client.insert_rows_json("project.monitoring.pipeline_logs", rows)
            print(f"Loaded {len(rows)} log events. Errors: {errors}")
    
    process_log_bucket("streetview-logs", "pipeline/")
    
    
    
    

BigQuery then powered a Looker Studio trend dashboard showing latency by stage over time.

**Result:** Latency trend analysis that previously required an ad-hoc request to engineering was now self-serve for any stakeholder. The analysis directly supported the identification of the ingestion bottleneck that drove the 25% process optimisation.

* * *

**Q36. How did you automate partition management in BigQuery or Snowflake?**

**Situation:** At J&J, BigQuery tables were not partitioned when the model was first built. As the model grew to 500M+ rows, full table scans were consuming excessive slot hours and the monthly BigQuery bill was increasing unsustainably.

**Task:** Automate the migration of existing tables to partitioned equivalents and enforce partitioning on all new tables.

**Action:**
    
    
    from google.cloud import bigquery
    
    client = bigquery.Client()
    
    def migrate_to_partitioned(source_table, dest_table, partition_col, cluster_cols):
        """Recreate table with partitioning and clustering."""
    
        query = f"""
        CREATE OR REPLACE TABLE `{dest_table}`
        PARTITION BY DATE({partition_col})
        CLUSTER BY {', '.join(cluster_cols)}
        AS SELECT * FROM `{source_table}`
        """
        job = client.query(query)
        job.result()
    
        # Verify row counts match
        src_count  = client.query(f"SELECT COUNT(*) AS n FROM `{source_table}`").to_dataframe()["n"][0]
        dest_count = client.query(f"SELECT COUNT(*) AS n FROM `{dest_table}`").to_dataframe()["n"][0]
    
        if src_count == dest_count:
            print(f"✅ Migrated {source_table}: {src_count:,} rows | Partitioned on {partition_col}")
        else:
            raise ValueError(f"Row count mismatch: source={src_count}, dest={dest_count}")
    
    # Apply to all fact tables
    tables_to_partition = [
        ("project.raw.claims",    "project.partitioned.claims",    "claim_date",    ["policy_type", "region"]),
        ("project.raw.policies",  "project.partitioned.policies",  "effective_date", ["policy_type"]),
        ("project.raw.handlers",  "project.partitioned.handlers",  "hire_date",      ["region"]),
    ]
    
    for src, dest, part_col, cluster in tables_to_partition:
        migrate_to_partitioned(src, dest, part_col, cluster)
    
    
    
    

**Result:** Post-partitioning, bytes scanned per query reduced by 60–70% as BigQuery only scanned the relevant partition. Monthly BigQuery compute costs reduced by ~55%. The automation was reused at AXA Insurance to migrate 8 additional tables.

* * *

**Q37. How did you automate environment configuration and secrets management?**

**Situation:** At J&J and AXA, connection strings, API keys, and service account paths were hardcoded in scripts — a security risk and a maintenance headache when credentials rotated.

**Task:** Centralise secrets management and automate credential injection into pipelines without hardcoding.

**Action:**
    
    
    # Using Google Secret Manager for credential injection
    from google.cloud import secretmanager
    import os
    
    def get_secret(secret_id, project_id="my-project"):
        """Retrieve a secret value from Google Secret Manager."""
        client  = secretmanager.SecretManagerServiceClient()
        name    = f"projects/{project_id}/secrets/{secret_id}/versions/latest"
        response = client.access_secret_version(request={"name": name})
        return response.payload.data.decode("UTF-8")
    
    # Usage: no hardcoded credentials anywhere
    SNOWFLAKE_PASSWORD  = get_secret("snowflake-prod-password")
    BQ_SERVICE_ACCOUNT  = get_secret("bigquery-sa-key-json")
    SALESFORCE_TOKEN    = get_secret("salesforce-api-token")
    
    # Environment-aware configuration
    ENV = os.environ.get("PIPELINE_ENV", "dev")
    CONFIG = {
        "dev":  {"project": "jnj-dev",   "dataset": "claims_dev"},
        "staging": {"project": "jnj-stg", "dataset": "claims_staging"},
        "prod": {"project": "jnj-prod",  "dataset": "claims_prod"}
    }
    active_config = CONFIG[ENV]
    print(f"Running in {ENV} environment: {active_config}")
    
    
    
    

**Result:** Zero hardcoded credentials across all pipelines. When the Snowflake password rotated, only the Secret Manager entry was updated — no code changes required. Security audit confirmed no credentials in version control. Environment switching (dev → staging → prod) was a single environment variable change.

* * *

**Q38. How did you use AppScript to automate data workflows at Google?**

**Situation:** At Google Street View, the defect management process required engineers to manually create structured bug reports in a specific format using the internal Bug Tracking system. The copy-paste process was error-prone and inconsistent across team members.

**Task:** Automate bug report creation from a structured input form, reducing manual effort and ensuring consistent format.

**Action:**
    
    
    // Google Apps Script: auto-create bug reports from Google Form submissions
    function onFormSubmit(e) {
      const responses = e.namedValues;
    
      const bugPayload = {
        title:       responses["Bug Title"][0],
        severity:    responses["Severity"][0],
        component:   "street-view-ingestion",
        description: formatDescription(responses),
        assignee:    autoAssignByComponent(responses["Affected Stage"][0]),
        labels:      ["automated-report", responses["Pipeline Stage"][0].toLowerCase()]
      };
    
      // Call internal Bug Tracking API
      const options = {
        method:      "post",
        contentType: "application/json",
        headers:     { Authorization: "Bearer " + getApiToken() },
        payload:     JSON.stringify(bugPayload)
      };
    
      const response = UrlFetchApp.fetch(
        "https://internal-bugtracker/api/v1/bugs", options
      );
      const bug = JSON.parse(response.getContentText());
    
      // Log to tracking sheet
      const sheet = SpreadsheetApp.openById(TRACKER_SHEET_ID).getActiveSheet();
      sheet.appendRow([
        bug.id, bug.title, bug.severity, bug.assignee,
        new Date(), "OPEN"
      ]);
    
      // Notify assignee
      GmailApp.sendEmail(
        bug.assignee + "@company.com",
        `New Bug Assigned: ${bug.title}`,
        `Bug ${bug.id} has been assigned to you.\n\nSeverity: ${bug.severity}`
      );
    }
    
    function formatDescription(responses) {
      return `**Steps to Reproduce:**\n${responses["Reproduction Steps"][0]}\n\n` +
             `**Expected:**\n${responses["Expected Behaviour"][0]}\n\n` +
             `**Actual:**\n${responses["Actual Behaviour"][0]}`;
    }
    
    
    
    

**Result:** Bug report creation time reduced by 80%. Consistency across all bug reports improved to 100% (previously ~60% had missing required fields). The automated assignment logic routed bugs to the correct engineer immediately, reducing the time-in-triage from 4 hours to zero. Average bug-to-resolution time reduced by 18 hours overall.

* * *

**Q39. How did you automate data format standardisation across multiple source systems?**

**Situation:** At Travelers Insurance, 3 source systems sent claims data in different formats: System A used `DD/MM/YYYY` dates and EUR amounts; System B used ISO dates and decimal USD; System C used epoch timestamps and amounts as strings with currency symbols.

**Task:** Build an automated normalisation layer that converted all formats to a single canonical standard before any transformation logic ran.

**Action:**
    
    
    import pandas as pd
    import re
    from datetime import datetime
    
    CANONICAL_DATE_FORMAT   = "%Y-%m-%d"
    CANONICAL_CURRENCY      = "USD"
    EXCHANGE_RATES          = {"EUR": 1.08, "GBP": 1.27, "USD": 1.0}
    
    def normalise_date(value, source_format):
        """Convert any date format to YYYY-MM-DD."""
        try:
            if str(value).isdigit():  # epoch timestamp
                return datetime.fromtimestamp(int(value)).strftime(CANONICAL_DATE_FORMAT)
            return pd.to_datetime(value, format=source_format,
                                  dayfirst=True).strftime(CANONICAL_DATE_FORMAT)
        except:
            return None
    
    def normalise_amount(value, source_currency):
        """Strip symbols, convert to float, apply exchange rate."""
        cleaned = re.sub(r"[£€$,\s]", "", str(value))
        try:
            amount = float(cleaned)
            return round(amount * EXCHANGE_RATES.get(source_currency, 1.0), 2)
        except:
            return None
    
    def normalise_source(df, source_config):
        df["claim_date_std"]  = df[source_config["date_col"]].apply(
            lambda x: normalise_date(x, source_config["date_format"])
        )
        df["loss_amount_usd"] = df[source_config["amount_col"]].apply(
            lambda x: normalise_amount(x, source_config["currency"])
        )
        return df
    
    # Config per source system
    SOURCE_CONFIGS = {
        "system_a": {"date_col": "ClaimDate",  "date_format": "%d/%m/%Y", "amount_col": "Amount",       "currency": "EUR"},
        "system_b": {"date_col": "claim_date", "date_format": "%Y-%m-%d", "amount_col": "loss_amount",  "currency": "USD"},
        "system_c": {"date_col": "epoch_ts",   "date_format": "epoch",    "amount_col": "amount_str",   "currency": "GBP"},
    }
    
    df_a_norm = normalise_source(df_a, SOURCE_CONFIGS["system_a"])
    df_b_norm = normalise_source(df_b, SOURCE_CONFIGS["system_b"])
    df_c_norm = normalise_source(df_c, SOURCE_CONFIGS["system_c"])
    
    # Union into canonical dataset
    df_unified = pd.concat([df_a_norm, df_b_norm, df_c_norm], ignore_index=True)
    
    
    
    

**Result:** Data format normalisation that previously required manual pre-processing by an analyst was fully automated. The canonical layer ran in under 5 minutes regardless of which source system changed its output format. Downstream transformation rules became source-agnostic for the first time.

* * *

**Q40. How did you automate notification and alerting for data teams?**

**Situation:** At Travelers Insurance, pipeline failures and data quality issues were discovered reactively — only when analysts or business users noticed incorrect data. There was no proactive alerting system in place.

**Task:** Build a comprehensive automated alerting system covering pipeline health, data freshness, quality gates, and volume anomalies.

**Action:**
    
    
    import smtplib
    import json
    import requests
    from email.mime.multipart import MIMEMultipart
    from email.mime.text import MIMEText
    from dataclasses import dataclass
    from enum import Enum
    
    class AlertSeverity(Enum):
        INFO    = "ℹ️"
        WARNING = "⚠️"
        CRITICAL = "🔴"
    
    @dataclass
    class Alert:
        title:    str
        message:  str
        severity: AlertSeverity
        context:  dict = None
    
    class AlertManager:
        def __init__(self, email_config, slack_webhook=None):
            self.email_config  = email_config
            self.slack_webhook = slack_webhook
    
        def send(self, alert: Alert):
            self._send_email(alert)
            if self.slack_webhook:
                self._send_slack(alert)
    
        def _send_email(self, alert):
            msg = MIMEMultipart()
            msg["Subject"] = f"{alert.severity.value} {alert.title}"
            msg["From"]    = self.email_config["from"]
            msg["To"]      = self.email_config["to"]
            body = f"{alert.message}\n\nContext: {json.dumps(alert.context, indent=2)}"
            msg.attach(MIMEText(body))
            with smtplib.SMTP(self.email_config["host"]) as s:
                s.send_message(msg)
    
        def _send_slack(self, alert):
            requests.post(self.slack_webhook, json={
                "text": f"{alert.severity.value} *{alert.title}*\n{alert.message}"
            })
    
    # Usage
    alerter = AlertManager(
        email_config={"from": "data-ops@company.com",
                      "to": "data-team@company.com",
                      "host": "smtp.company.com"},
        slack_webhook=SLACK_WEBHOOK_URL
    )
    
    # Call in pipeline checks
    alerter.send(Alert(
        title="Claims Silver — Data Stale",
        message="Claims Silver has not refreshed in 45 minutes.",
        severity=AlertSeverity.CRITICAL,
        context={"last_refresh": "2026-04-19 08:15:00", "current_time": "2026-04-19 09:00:00"}
    ))
    
    
    
    

**Result:** Mean time to detection for pipeline issues reduced from 6 hours to under 10 minutes. The alert manager was adopted across all 8 pipelines in the Snowflake environment. Reactive fire-fighting decreased dramatically — 4 near-miss incidents were caught automatically in the following 6 months before any business impact occurred.

* * *

**Q41. How did you automate model output validation in a data pipeline?**

**Situation:** At Travelers Insurance, the Gold layer aggregation (fraud risk scores per policy) occasionally produced score values outside the expected 0–1 range due to edge cases in the calculation logic. These invalid scores were silently passed to the fraud team's dashboard.

**Task:** Add automated validation of model outputs before they reached any downstream consumer.

**Action:**
    
    
    def validate_model_outputs(df, output_col, expected_min=0.0, expected_max=1.0):
        """Validate that model outputs fall within expected bounds."""
        violations = []
    
        # Range check
        out_of_range = df[(df[output_col] < expected_min) | (df[output_col] > expected_max)]
        if not out_of_range.empty:
            violations.append({
                "check":   "Range",
                "count":   len(out_of_range),
                "examples": out_of_range[["policy_id", output_col]].head(5).to_dict("records")
            })
    
        # Null check
        nulls = df[df[output_col].isnull()]
        if not nulls.empty:
            violations.append({
                "check": "Nulls",
                "count": len(nulls)
            })
    
        # Distribution check: mean should be between 0.1 and 0.9
        mean_score = df[output_col].mean()
        if not (0.05 <= mean_score <= 0.95):
            violations.append({
                "check": "Distribution",
                "detail": f"Mean score = {mean_score:.4f} (expected 0.05–0.95)"
            })
    
        if violations:
            print(f"❌ Model output validation FAILED ({len(violations)} issues):")
            for v in violations:
                print(f"   {v}")
            raise ValueError("Model output validation failed. Gold layer blocked.")
        else:
            print(f"✅ Model output validation passed. Mean={mean_score:.4f}")
    
    validate_model_outputs(df_fraud_scores, "fraud_risk_score")
    
    
    
    

**Result:** Invalid fraud scores no longer reached the dashboard. The validation gate caught 3 edge-case calculation errors in the first 3 months. Fraud team confidence in the scores improved significantly, increasing their adoption of the automated flagging system.

* * *

**Q42. How did you automate the monitoring of Snowflake credit consumption?**

**Situation:** At Travelers Insurance, Snowflake compute credits were being consumed unpredictably — some months 40% over budget — because no one monitored warehouse activity between billing cycles until the invoice arrived.

**Task:** Build an automated credit consumption monitor that tracked usage in real time and alerted before overspend occurred.

**Action:**
    
    
    -- Daily credit consumption summary
    CREATE OR REPLACE VIEW monitoring.daily_credit_usage AS
    SELECT
      TO_DATE(start_time)         AS usage_date,
      warehouse_name,
      SUM(credits_used)           AS credits_used,
      SUM(SUM(credits_used)) OVER (
        PARTITION BY TO_DATE(start_time)
      )                           AS daily_total_credits,
      SUM(SUM(credits_used)) OVER (
        ORDER BY TO_DATE(start_time)
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
      )                           AS cumulative_mtd_credits
    FROM snowflake.account_usage.warehouse_metering_history
    WHERE start_time >= DATE_TRUNC('month', CURRENT_DATE())
    GROUP BY 1, 2
    ORDER BY 1 DESC, credits_used DESC;
    
    -- Alert task: fires when MTD credits > 80% of monthly budget
    CREATE OR REPLACE TASK alert_credit_threshold
      WAREHOUSE = XSMALL_WH
      SCHEDULE  = 'USING CRON 0 8 * * * UTC'
    AS
    CALL SYSTEM$SEND_EMAIL(
      'data-ops@company.com',
      'Snowflake Credit Alert',
      'Monthly credit usage has exceeded 80% of budget. Review warehouse activity.'
    )
    WHERE (
      SELECT MAX(cumulative_mtd_credits) FROM monitoring.daily_credit_usage
    ) > 400;  -- 80% of 500-credit monthly budget
    
    ALTER TASK alert_credit_threshold RESUME;
    
    
    
    

**Result:** Over-budget months eliminated after the first quarter. The team could see credit consumption by warehouse in real time and adjust configuration (e.g., auto-suspend timeout, warehouse size) proactively. Monthly credit usage stabilised at 15% below budget on average.

* * *

## 📂 Section 4: Monitoring, Alerting & Governance Automation (Q43–Q55)

* * *

**Q43. How do you automate the detection of data drift in source systems?**

**Situation:** At J&J, source systems occasionally changed their data distributions without warning — for example, a previously uniform `claim_type` field suddenly showed 80% of records as `UNKNOWN` after an upstream system update. This broke downstream segmentation reports.

**Task:** Detect distribution changes in key fields automatically and alert before they propagated to analytics layers.

**Action:**
    
    
    import pandas as pd
    from scipy.stats import ks_2samp
    from google.cloud import bigquery
    
    client = bigquery.Client()
    
    def detect_data_drift(table, column, historical_days=30, threshold_pvalue=0.05):
        """Use Kolmogorov-Smirnov test to detect distribution shift."""
    
        # Historical sample (baseline)
        q_hist = f"""
            SELECT {column} FROM `{table}`
            WHERE DATE(created_at) BETWEEN
              DATE_SUB(CURRENT_DATE(), INTERVAL {historical_days + 7} DAY)
              AND DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
            AND {column} IS NOT NULL LIMIT 50000
        """
        # Recent sample (current)
        q_curr = f"""
            SELECT {column} FROM `{table}`
            WHERE DATE(created_at) >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
            AND {column} IS NOT NULL LIMIT 50000
        """
    
        hist = client.query(q_hist).to_dataframe()[column]
        curr = client.query(q_curr).to_dataframe()[column]
    
        if pd.api.types.is_numeric_dtype(hist):
            stat, p_value = ks_2samp(hist, curr)
            drifted = p_value < threshold_pvalue
            print(f"{'⚠️ DRIFT' if drifted else '✅ STABLE'}: {column} | "
                  f"KS stat={stat:.4f} | p={p_value:.4f}")
            return drifted, p_value
        else:
            # Categorical: compare top-5 value distribution
            hist_dist = hist.value_counts(normalize=True).head(5)
            curr_dist = curr.value_counts(normalize=True).head(5)
            print(f"\nCategorical Distribution — {column}:")
            print(pd.DataFrame({"Historical": hist_dist, "Current": curr_dist}))
            return False, None
    
    detect_data_drift("project.silver.claims", "claim_type")
    detect_data_drift("project.silver.claims", "loss_amount")
    
    
    
    

**Result:** Source system distribution changes were detected within 24 hours of occurrence — before they affected any reports. The J&J `claim_type` UNKNOWN spike was caught within 6 hours and traced to a missing lookup table in the source system. Issue resolved before analysts noticed.

* * *

**Q44. How did you automate row count reconciliation across pipeline layers?**

**Situation:** At Travelers Insurance, there was no systematic check that the number of records entering the Bronze layer matched what came out of the Gold layer after all transformations. Silent row drops during transformation had occurred twice without detection.

**Task:** Build an automated row count audit that tracked record counts at every layer.

**Action:**
    
    
    -- Automated row count tracking table
    CREATE OR REPLACE TABLE monitoring.layer_row_counts (
      run_id      VARCHAR DEFAULT UUID_STRING(),
      run_time    TIMESTAMP DEFAULT CURRENT_TIMESTAMP(),
      layer       VARCHAR,
      table_name  VARCHAR,
      row_count   INT,
      run_date    DATE DEFAULT CURRENT_DATE()
    );
    
    -- Procedure: captures counts across all layers
    CREATE OR REPLACE PROCEDURE capture_layer_counts()
    RETURNS STRING
    LANGUAGE SQL
    AS
    $$
      INSERT INTO monitoring.layer_row_counts (layer, table_name, row_count)
      SELECT 'BRONZE', 'CLAIMS_BRONZE', COUNT(*) FROM CLAIMS_BRONZE;
    
      INSERT INTO monitoring.layer_row_counts (layer, table_name, row_count)
      SELECT 'SILVER', 'CLAIMS_SILVER', COUNT(*) FROM CLAIMS_SILVER;
    
      INSERT INTO monitoring.layer_row_counts (layer, table_name, row_count)
      SELECT 'GOLD', 'CLAIMS_GOLD', COUNT(*) FROM CLAIMS_GOLD;
    
      RETURN 'Counts captured.';
    $$;
    
    -- Cross-layer reconciliation view
    CREATE OR REPLACE VIEW monitoring.layer_reconciliation AS
    SELECT
      b.run_date,
      b.row_count AS bronze_count,
      s.row_count AS silver_count,
      g.row_count AS gold_count,
      b.row_count - s.row_count AS bronze_to_silver_delta,
      s.row_count - g.row_count AS silver_to_gold_delta,
      CASE WHEN (b.row_count - s.row_count) > b.row_count * 0.01
           THEN 'INVESTIGATE' ELSE 'OK' END AS silver_status
    FROM monitoring.layer_row_counts b
    JOIN monitoring.layer_row_counts s ON b.run_date = s.run_date AND s.layer = 'SILVER'
    JOIN monitoring.layer_row_counts g ON b.run_date = g.run_date AND g.layer = 'GOLD'
    WHERE b.layer = 'BRONZE'
    ORDER BY b.run_date DESC;
    
    
    
    

**Result:** Row count reconciliation ran automatically after every pipeline execution. The 2 historical silent-drop incidents would have been caught within 15 minutes under this system. Subsequent 12 months: zero undetected row drops across any layer.

* * *

**Q45. How did you automate SLA monitoring for data delivery?**

**Situation:** At J&J, there were SLA commitments: actuarial teams would receive refreshed data by 7am daily. However, there was no automated way to know whether the SLA had been met or missed — someone had to manually check the dashboard each morning.

**Task:** Build an automated SLA monitoring system that tracked compliance and escalated violations.

**Action:**
    
    
    from google.cloud import bigquery
    import smtplib
    from datetime import datetime, time
    
    client = bigquery.Client()
    
    SLA_DEFINITIONS = [
        {
            "name":            "Claims Silver Daily Refresh",
            "table":           "project.silver.claims",
            "timestamp_field": "updated_at",
            "sla_time_utc":    time(7, 0),   # must refresh by 07:00 UTC
            "owner":           "data-ops@jnj.com",
            "escalation":      "head-of-analytics@jnj.com"
        }
    ]
    
    def check_sla(sla_def):
        query = f"""
            SELECT MAX({sla_def['timestamp_field']}) AS last_refresh
            FROM `{sla_def['table']}`
        """
        df = client.query(query).to_dataframe()
        last_refresh = df["last_refresh"].iloc[0]
        today_sla    = datetime.combine(datetime.today().date(), sla_def["sla_time_utc"])
    
        if last_refresh and last_refresh.replace(tzinfo=None) >= today_sla:
            print(f"✅ SLA MET: {sla_def['name']} — refreshed at {last_refresh}")
            return True
        else:
            delay_mins = (datetime.utcnow() - today_sla).seconds // 60
            print(f"⚠️ SLA BREACHED: {sla_def['name']} — {delay_mins} mins late")
            send_sla_breach_alert(sla_def, last_refresh, delay_mins)
            return False
    
    def send_sla_breach_alert(sla_def, last_refresh, delay_mins):
        subject = f"⚠️ SLA Breach: {sla_def['name']}"
        body    = (f"The SLA for {sla_def['name']} has been breached.\n"
                   f"Expected by: {sla_def['sla_time_utc']}\n"
                   f"Last refresh: {last_refresh}\n"
                   f"Delay: {delay_mins} minutes\n"
                   f"Owner: {sla_def['owner']}")
        send_email(sla_def["escalation"], subject, body)
    
    for sla in SLA_DEFINITIONS:
        check_sla(sla)
    
    
    
    

**Result:** SLA compliance became measurable and visible for the first time. Breach notifications reached the pipeline owner within 5 minutes of an SLA miss. SLA adherence rate improved from ~85% (estimated, never tracked) to 98% (measured) within 3 months as the team became aware of and accountable for their commitments.

* * *

**Q46. How did you automate RBAC (Role-Based Access Control) provisioning?**

**Situation:** At AXA Insurance, new analysts joined regularly and needed access to specific BigQuery datasets based on their role. Access provisioning was manual — a ticket raised, processed by IT in 2–3 days, often with incorrect permissions.

**Task:** Automate access provisioning so new users got the right access immediately upon role assignment.

**Action:**
    
    
    from google.cloud import bigquery, iam
    
    ROLE_PERMISSIONS = {
        "ANALYST": {
            "datasets": ["silver", "gold"],
            "access":   "roles/bigquery.dataViewer"
        },
        "DATA_ENGINEER": {
            "datasets": ["bronze", "silver", "gold"],
            "access":   "roles/bigquery.dataEditor"
        },
        "ACTUARIAL_LEAD": {
            "datasets": ["gold"],
            "access":   "roles/bigquery.dataViewer",
            "extra":    ["roles/bigquery.jobUser"]
        }
    }
    
    def provision_access(user_email, role, project_id="axa-analytics"):
        client    = bigquery.Client(project=project_id)
        perms     = ROLE_PERMISSIONS.get(role)
    
        if not perms:
            raise ValueError(f"Unknown role: {role}")
    
        for dataset_id in perms["datasets"]:
            dataset = client.get_dataset(f"{project_id}.{dataset_id}")
            entries = list(dataset.access_entries)
            entries.append(bigquery.AccessEntry(
                role        = perms["access"],
                entity_type = "userByEmail",
                entity_id   = user_email
            ))
            dataset.access_entries = entries
            client.update_dataset(dataset, ["access_entries"])
            print(f"✅ {user_email} granted {perms['access']} on {dataset_id}")
    
        print(f"Access provisioning complete for {user_email} ({role})")
    
    # Triggered via HR system webhook on new employee creation
    provision_access("new.analyst@axa.com", "ANALYST")
    
    
    
    

**Result:** Access provisioning time reduced from 2–3 days to under 2 minutes. Incorrect permission assignments eliminated — role-based templates ensured consistency. Analyst onboarding satisfaction improved. Security audit noted the automated provisioning as a control improvement over the previous manual process.

* * *

**Q47. How did you automate database index health monitoring?**

**Situation:** At Refinitiv (LSEG), the SQL Server trading database relied on specific indexes for millisecond-level query response. Index fragmentation grew over time as high-frequency trades were inserted, but nobody monitored fragmentation — until queries started slowing down mid-session.

**Task:** Automate index health monitoring and maintenance to prevent performance degradation.

**Action:**
    
    
    -- SQL Server: automated index health check and rebuild
    CREATE PROCEDURE [dba].[AutomatedIndexMaintenance]
    AS
    BEGIN
      DECLARE @TableName    NVARCHAR(256)
      DECLARE @IndexName    NVARCHAR(256)
      DECLARE @Fragmentation FLOAT
      DECLARE @SQL          NVARCHAR(MAX)
    
      -- Identify fragmented indexes
      DECLARE index_cursor CURSOR FOR
        SELECT
          OBJECT_NAME(ps.object_id)  AS table_name,
          i.name                     AS index_name,
          ps.avg_fragmentation_in_percent
        FROM sys.dm_db_index_physical_stats(
          DB_ID(), NULL, NULL, NULL, 'LIMITED'
        ) ps
        JOIN sys.indexes i ON ps.object_id = i.object_id
                           AND ps.index_id = i.index_id
        WHERE ps.avg_fragmentation_in_percent > 10   -- threshold: 10%
          AND ps.page_count > 1000                   -- only sizable indexes
        ORDER BY ps.avg_fragmentation_in_percent DESC;
    
      OPEN index_cursor
      FETCH NEXT FROM index_cursor INTO @TableName, @IndexName, @Fragmentation
    
      WHILE @@FETCH_STATUS = 0
      BEGIN
        IF @Fragmentation > 30
          SET @SQL = N'ALTER INDEX [' + @IndexName + N'] ON [' + @TableName + N'] REBUILD'
        ELSE
          SET @SQL = N'ALTER INDEX [' + @IndexName + N'] ON [' + @TableName + N'] REORGANIZE'
    
        EXEC sp_executesql @SQL
        PRINT 'Maintained index: ' + @IndexName + ' on ' + @TableName +
              ' (' + CAST(CAST(@Fragmentation AS INT) AS NVARCHAR) + '% frag)'
    
        FETCH NEXT FROM index_cursor INTO @TableName, @IndexName, @Fragmentation
      END
    
      CLOSE index_cursor
      DEALLOCATE index_cursor
    END;
    
    -- Schedule via SQL Agent Job: every Sunday 2am
    
    
    
    

**Result:** Index fragmentation maintained below 10% automatically. The query slowdown incidents that previously occurred monthly were eliminated. Real-time trade execution maintained millisecond-level retrieval consistently. The 99.9% data consistency target was sustained without manual DBA intervention.

* * *

**Q48. How did you automate the detection and resolution of orphaned records?**

**Situation:** At J&J, the BigQuery consolidation model occasionally produced orphaned claim records — claims referencing `policy_id` values that didn't exist in the policy dimension. These caused incorrect aggregations in the actuarial reports.

**Task:** Automate orphan detection and flagging as part of the pipeline DQ layer.

**Action:**
    
    
    -- BigQuery: orphaned record detection and quarantine
    CREATE OR REPLACE PROCEDURE detect_and_quarantine_orphans()
    BEGIN
      -- Find claims with no matching policy
      CREATE OR REPLACE TABLE `project.quarantine.orphaned_claims` AS
      SELECT
        c.*,
        'MISSING_POLICY' AS orphan_reason,
        CURRENT_TIMESTAMP() AS quarantined_at
      FROM `project.silver.claims` c
      LEFT JOIN `project.silver.policies` p ON c.policy_id = p.policy_id
      WHERE p.policy_id IS NULL;
    
      -- Log count for monitoring
      INSERT INTO `project.monitoring.pipeline_audit` (check_name, count, run_time)
      SELECT
        'orphaned_claims',
        COUNT(*),
        CURRENT_TIMESTAMP()
      FROM `project.quarantine.orphaned_claims`
      WHERE DATE(quarantined_at) = CURRENT_DATE();
    
      -- Remove orphans from Silver (they go to quarantine, not Gold)
      DELETE FROM `project.silver.claims`
      WHERE claim_id IN (
        SELECT claim_id FROM `project.quarantine.orphaned_claims`
        WHERE DATE(quarantined_at) = CURRENT_DATE()
      );
    
      SELECT CONCAT('Quarantined ', COUNT(*), ' orphaned claims.') AS result
      FROM `project.quarantine.orphaned_claims`
      WHERE DATE(quarantined_at) = CURRENT_DATE();
    END;
    
    
    
    

**Result:** Orphaned records were automatically detected, quarantined, and excluded from Gold layer aggregations. Actuarial report accuracy improved. Quarantined records were reviewed weekly by the data team and used to trace issues back to source systems — resulting in fixes that eliminated the root cause within 2 months.

* * *

**Q49. How did you automate query performance monitoring in BigQuery?**

**Situation:** At J&J, some analyst-written queries were scanning entire unpartitioned tables, causing slot contention and slowing down other users. There was no visibility into which queries were the most expensive until the monthly billing report arrived.

**Task:** Build real-time query cost monitoring so expensive queries were identified and addressed immediately.

**Action:**
    
    
    -- BigQuery: expensive query monitoring view
    CREATE OR REPLACE VIEW `project.monitoring.expensive_queries` AS
    SELECT
      job_id,
      user_email,
      creation_time,
      ROUND(total_bytes_processed / POW(1024, 3), 2) AS gb_scanned,
      ROUND(total_slot_ms / 1000.0, 1)               AS slot_seconds,
      ROUND(total_bytes_billed / POW(1024, 3) * 0.005, 4) AS est_cost_usd,
      -- Estimate: $5/TB
      REGEXP_EXTRACT(query, r'FROM `([^`]+)`')         AS primary_table,
      SUBSTR(query, 1, 500)                            AS query_preview,
      CASE
        WHEN total_bytes_processed > 1e11 THEN '🔴 CRITICAL (>100GB)'
        WHEN total_bytes_processed > 1e10 THEN '⚠️ HIGH (>10GB)'
        ELSE '✅ OK'
      END AS cost_flag
    FROM `region-eu.INFORMATION_SCHEMA.JOBS`
    WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
      AND job_type = 'QUERY'
      AND state    = 'DONE'
    ORDER BY total_bytes_processed DESC
    LIMIT 50;
    
    
    
    
    
    
    # Alert on critically expensive queries
    def check_expensive_queries():
        df = client.query("""
            SELECT * FROM `project.monitoring.expensive_queries`
            WHERE cost_flag LIKE '🔴%'
            AND DATE(creation_time) = CURRENT_DATE()
        """).to_dataframe()
    
        if not df.empty:
            for _, row in df.iterrows():
                alerter.send(Alert(
                    title   = f"Expensive Query Alert: {row['user_email']}",
                    message = f"Query scanned {row['gb_scanned']}GB (~${row['est_cost_usd']} USD)\n"
                              f"Table: {row['primary_table']}\n"
                              f"Preview: {row['query_preview']}",
                    severity = AlertSeverity.WARNING
                ))
    
    
    
    

**Result:** Top 5 most expensive query patterns identified within the first week. Two analysts were running `SELECT *` on a 500GB unpartitioned table daily. After adding partition filters, their queries went from 500GB scans to 2GB scans — a 99.6% reduction. Monthly BigQuery costs reduced by 55% in the first quarter.

* * *

**Q50. How did you automate the backup and recovery of critical data assets?**

**Situation:** At AXA Insurance, the Microsoft Fabric Silver layer had been accidentally overwritten during a schema migration test that ran against the wrong environment. Recovery required manually reprocessing 3 weeks of source files — taking 18 hours.

**Task:** Implement an automated backup and point-in-time recovery capability so any future incident could be recovered in minutes, not hours.

**Action:**
    
    
    from delta import DeltaTable
    from pyspark.sql import SparkSession
    from datetime import datetime, timedelta
    
    spark = SparkSession.builder.appName("BackupManager").getOrCreate()
    
    DELTA_PATH = "abfss://silver@axa.dfs.core.windows.net/claims_clean/"
    
    def create_snapshot(delta_path, snapshot_label):
        """Tag the current Delta version as a named snapshot."""
        dt = DeltaTable.forPath(spark, delta_path)
        history = dt.history(1)
        current_version = history.select("version").collect()[0][0]
    
        # Write snapshot metadata
        snapshot_meta = spark.createDataFrame([{
            "label":   snapshot_label,
            "version": current_version,
            "created": datetime.utcnow().isoformat(),
            "path":    delta_path
        }])
        snapshot_meta.write.mode("append").format("delta").save(
            "abfss://backup@axa.dfs.core.windows.net/snapshot_registry/"
        )
        print(f"Snapshot '{snapshot_label}' registered at version {current_version}")
    
    def restore_to_snapshot(delta_path, snapshot_label):
        """Restore Delta table to a named snapshot version."""
        registry = spark.read.format("delta").load(
            "abfss://backup@axa.dfs.core.windows.net/snapshot_registry/"
        )
        version = registry.filter(f"label = '{snapshot_label}'") \
                          .select("version").collect()[0][0]
    
        df_restored = spark.read.format("delta") \
            .option("versionAsOf", version) \
            .load(delta_path)
    
        df_restored.write.format("delta").mode("overwrite").save(delta_path)
        print(f"✅ Restored '{snapshot_label}' (version {version}) to {delta_path}")
    
    # Schedule daily snapshots
    create_snapshot(DELTA_PATH, f"daily_{datetime.today().strftime('%Y%m%d')}")
    
    # Recovery (when needed)
    # restore_to_snapshot(DELTA_PATH, "daily_20260417")
    
    
    
    

**Result:** Automated daily snapshots enabled point-in-time recovery in under 10 minutes vs the previous 18-hour reprocessing. The accidental overwrite scenario became a non-issue. Delta's time travel was used twice in 12 months to recover from errors — both times with zero data loss and minimal disruption.

* * *

## 📂 Section 5: Process, Scheduling & Strategy (Q51–Q65)

* * *

**Q51. How do you decide what to automate and what not to?**

**Situation:** At every engagement — J&J, AXA, Google — I was faced with more potential automation opportunities than available time. Automating the wrong things wastes engineering effort on low-value processes.

**Task:** Apply a structured framework for prioritising automation investment.

**Action:** I use a simple **ROI matrix** with four dimensions:

Dimension| Question  
---|---  
**Frequency**|  How often does this task run? Daily > monthly  
**Time cost**|  How long does it take manually? Hours > minutes  
**Error risk**|  How often does human error cause problems?  
**Stability**|  Is the process stable enough to automate?  
      
    
    HIGH VALUE to automate:
      ✅ Daily pipeline runs (frequency ×, time ×, error risk ×)
      ✅ Weekly reports distributed to 10+ people
      ✅ Data quality checks that must run every load
    
    LOW VALUE to automate:
      ❌ Ad-hoc one-time analyses
      ❌ Processes that change every month
      ❌ Tasks that take 5 mins and run quarterly
    
    
    
    

At Google Street View: weekly bug report compilation = daily (no), 3 hours, error-prone, stable → **automate**. Ad-hoc CEO request for a custom metric → **don't automate**.

**Result:** By applying this framework consistently, I avoided over-engineering and focused automation effort on the highest-ROI processes. The 3 automations at Google Street View (pipeline reporting, defect tracking, SSD monitoring) collectively recovered 5+ hours/week of analyst time.

* * *

**Q52. How did you introduce automation to a team that was resistant to change?**

**Situation:** At Wipro/Cisco, the data team had been using the same manual ETL process for 3 years. When I proposed automating the SQL Server data warehouse loading via SSIS, two senior analysts were concerned: "If it breaks, who fixes it?" and "What if it does something wrong silently?"

**Task:** Get buy-in for the automation without dismissing legitimate concerns.

**Action:**

  * Acknowledged their concerns were valid — a broken automated process is worse than a slow manual one if nobody knows it failed.
  * Proposed a parallel run: the automation ran alongside the manual process for 4 weeks. Both outputs were compared daily. This proved reliability without risk.
  * Built in visible failure notifications — if the automation failed, it sent an email within 5 minutes (more visible than a silent manual skip).
  * Involved the resistant analysts in designing the monitoring layer — they defined what alerts they needed, making them co-owners rather than spectators.
  * Ran a retrospective at week 4: showed the parallel run comparison (100% match) and quantified the time saved (3 hours/week between them).



**Result:** Both analysts became advocates for the automation by week 6. The SSIS pipeline was deployed to production. The pattern of "parallel run + transparent monitoring" became my standard approach for introducing automation to established teams.

* * *

**Q53. How do you ensure automated pipelines remain maintainable over time?**

**Situation:** At J&J, I inherited automation scripts written 3 years earlier that nobody on the current team fully understood. Modifying them was risky — a small change could break multiple downstream processes.

**Task:** Make the existing automation maintainable and establish standards that prevented the same problem recurring.

**Action:**

  1. **Refactored spaghetti scripts** into modular functions — each function had one responsibility, a clear name, and a docstring.
  2. **Introduced configuration files** — all environment-specific values (table names, thresholds, schedules) moved to YAML config files, not hardcoded in scripts.
  3. **Added inline documentation** explaining _why_ decisions were made, not just _what_ the code does.
  4. **Version-controlled everything** in Git — with meaningful commit messages ("Fix: handle null policy_id before MERGE" not "update script").
  5. **Wrote a runbook** for each pipeline: what it does, inputs/outputs, how to restart if it fails, who to contact.
  6. **Enforced via code review** — no PR merged without documentation and tests.



**Result:** The next engineer to modify the pipeline (3 months later) reported they understood the codebase in 2 days vs the 3 weeks the original had taken. Runbooks were used twice for incident recovery — both times resolved without escalation.

* * *

**Q54. Describe a time automation you built failed in production. What did you do?**

**Situation:** At Travelers Insurance, an automated Snowflake Task I built for the Silver layer refresh silently failed due to a timeout caused by a 3× volume spike. The fraud team discovered stale data at 8am — 6 hours after the failure.

**Task:** Recover quickly, minimise business impact, and build a better system.

**Action:**

  1. **Immediate response** — diagnosed via `TASK_HISTORY`, manually triggered a catch-up run on a larger warehouse, notified the fraud team with an ETA.
  2. **Business mitigation** — identified the 6-hour window of claims that needed manual review while the automated system caught up.
  3. **Root cause** — 3-day source system backlog flushed all at once, exceeding the task timeout.
  4. **Fix** — increased timeout, added dynamic warehouse sizing for high-volume scenarios, and added a volume anomaly alert that paused the task for human review if the incoming volume was >2× the 7-day average.
  5. **Retrospective** — shared the incident write-up with the team: what happened, what changed, and what the new system looks like.



**Result:** The improved system detected and recovered from 4 near-miss incidents in the following 6 months — all within 10 minutes, none reaching the business. The incident response template I created was used by the broader data team for future pipeline incidents.

* * *

**Q55. How did you automate the scheduling of complex multi-step data workflows?**

**Situation:** At J&J, the BigQuery pipeline had 12 interdependent steps — some could run in parallel, others had strict dependencies (e.g., Gold layer couldn't build until both Silver claims AND Silver policies were ready).

**Task:** Automate the orchestration of a complex DAG (Directed Acyclic Graph) of tasks with correct dependency management.

**Action:**
    
    
    # Using a simple dependency scheduler in Python
    # (In production, this would be Cloud Composer / Airflow)
    import threading
    from concurrent.futures import ThreadPoolExecutor, as_completed
    
    PIPELINE_DAG = {
        "ingest_claims":       [],
        "ingest_policies":     [],
        "ingest_handlers":     [],
        "clean_claims":        ["ingest_claims"],
        "clean_policies":      ["ingest_policies"],
        "build_silver_claims": ["clean_claims"],
        "build_silver_policies": ["clean_policies"],
        "build_gold_facts":    ["build_silver_claims", "build_silver_policies", "ingest_handlers"],
        "refresh_dashboards":  ["build_gold_facts"]
    }
    
    completed = set()
    lock = threading.Lock()
    
    def can_run(task, dag, completed):
        return all(dep in completed for dep in dag[task])
    
    def run_task(task_name):
        print(f"▶ Starting: {task_name}")
        # ... actual task logic here ...
        import time; time.sleep(1)  # simulate work
        with lock:
            completed.add(task_name)
        print(f"✅ Completed: {task_name}")
    
    def run_dag(dag):
        remaining = set(dag.keys())
        with ThreadPoolExecutor(max_workers=4) as executor:
            while remaining:
                ready = {t for t in remaining if can_run(t, dag, completed)}
                if not ready:
                    break
                futures = {executor.submit(run_task, t): t for t in ready}
                remaining -= ready
                for f in as_completed(futures):
                    f.result()  # propagate exceptions
    
    run_dag(PIPELINE_DAG)
    
    
    
    

**Result:** Parallel-eligible tasks (3 ingestion tasks) ran simultaneously, reducing total pipeline runtime by ~40% vs sequential execution. Dependency violations became impossible — Gold could never build before Silver was ready. The pattern was later migrated to Cloud Composer (Airflow) for full production scheduling.

* * *

## 📂 Section 6: Behavioural — Automation Strategy & Impact (Q56–Q65)

* * *

**Q56. What is the most impactful automation you've ever built? Why?**

**Situation:** Across 11+ years, the automation with the greatest combined impact was the Python + AppScript defect reporting pipeline at Google Street View.

**Task:** Reflect on why this particular automation stood out.

**Action/Reflection:** The defect pipeline automation stood out for three reasons:

  1. **Immediate time saving** — 3 hours/week eliminated for the engineering team.
  2. **Quality improvement** — bug-to-resolution time reduced by 18 hours because defects appeared in the tracking system within 6 hours of creation rather than at the next weekly sync.
  3. **Compound effect** — faster bug visibility enabled faster fixing, which improved the pipeline quality metrics that fed into the executive dashboards, which improved R&D decision-making. The automation didn't just save time — it made the whole system work better.



**Result:** What made it genuinely impactful was that it sat at a critical bottleneck — defect reporting was the slowest link in the quality improvement loop. Automating that one step unlocked improvements throughout the chain. I learned that the most valuable automations are those that unblock other work, not just ones that eliminate a single task.

* * *

**Q57. How do you measure the ROI of an automation project?**

**Situation:** At J&J, after I proposed automating the compliance report generation (a 5-hour monthly manual process), I was asked by the project manager to justify the engineering time required to build it.

**Task:** Quantify the business case for the automation investment.

**Action:**
    
    
    ROI Calculation Framework I used:
    
    COSTS:
      Build time:        12 hours engineering (one-time)
      Maintenance:       1 hour/month (estimated)
    
    BENEFITS (monthly):
      Manual time saved: 5 hours × 3 analysts = 15 analyst hours/month
      Error reduction:   2 rework incidents/month × 2 hours = 4 hours saved
      Total benefit:     19 analyst hours/month
    
    PAYBACK PERIOD:
      Build cost (12h) ÷ monthly benefit (19h) = 0.6 months
      → Payback in under 1 month
    
    12-MONTH ROI:
      Benefit: 19h × 12 = 228 analyst hours saved
      Cost:    12h build + (1h × 12) = 24h
      Net gain: 204 hours
      ROI: 204/24 = 8.5× return on engineering investment
    
    
    
    

Presented this one-pager to the PM. Approved immediately.

**Result:** Automation built and deployed within 2 weeks. 228 analyst hours saved in the first year. ROI exceeded the projection — error incidents dropped to zero (not 50% as conservatively estimated), adding further value. This ROI framework was adopted for all subsequent automation proposals in the team.

* * *

**Q58. How have you used automation to improve data team productivity?**

**Situation:** At the start of the J&J engagement, the analytics team of 12 analysts spent an estimated 30–40% of their time on manual, repetitive data tasks: running the same queries, copying data between systems, formatting reports.

**Task:** Use automation to shift team time from repetitive tasks to higher-value analytical work.

**Action:** Over 12 months, I built and deployed a portfolio of automations targeting the highest-time-cost manual tasks:

Automation| Time Saved/Month| Owner Impact  
---|---|---  
Automated compliance KPI report| 15 analyst-hours| 3 analysts freed  
Scheduled BigQuery reconciliation| 8 analyst-hours| 2 analysts freed  
Self-serve data dictionary| 20 analyst-hours| All 12 analysts  
Pipeline failure alerting| 12 analyst-hours| Engineering team  
STTM validation automation| 10 analyst-hours| 2 analysts freed  
**Total**| **65 analyst-hours/month**|   
  
**Result:** The team's ratio of "manual task time" to "analytical thinking time" shifted from ~40/60 to ~10/90. Three analysts redirected their freed time to building new analytical models that had previously been backlogged for 6+ months. One of those models directly informed the $2M+ cost-saving recommendation.

* * *

**Q59. Describe a time you had to automate a process with no existing documentation.**

**Situation:** At AXA Insurance, the legacy on-premises claims loading process had no documentation — it existed as tribal knowledge held by one retiring senior DBA. The process needed to be both understood and automated before the DBA left.

**Task:** Reverse-engineer an undocumented legacy process and automate its replacement within 6 weeks.

**Action:**

  * **Observation phase (week 1–2):** Shadowed the DBA during 3 loading cycles, documenting every step in real time. Used questions like "why do you do that step?" and "what happens if you skip it?" to surface implicit business rules.
  * **Process mapping (week 2):** Built a process flow diagram in Confluence capturing every decision point and exception case.
  * **Gap analysis (week 3):** Identified 4 undocumented exception handlers — specific claim types that needed special routing logic not captured in any system.
  * **Automation build (weeks 3–6):** Built the PySpark pipeline in Microsoft Fabric, incorporating all 4 exception handlers, with DQ checks validating outputs against the legacy system in parallel.
  * **Parallel validation (week 5–6):** Ran both systems simultaneously, achieving 100% output match before the DBA departed.



**Result:** The entire legacy loading process was automated and documented before the DBA left. Zero knowledge loss. The new pipeline ran in 4 minutes vs 3 hours for the manual process. The documentation I produced became the foundation for the full AXA migration runbook.

* * *

**Q60. How do you handle automation that needs to be updated frequently as business rules change?**

**Situation:** At J&J, the claims routing business rules changed 4 times in 12 months as the actuarial team refined their underwriting models. Each change required an analyst to locate the relevant Python function, understand the logic, and modify it correctly — a 2-hour exercise each time with risk of error.

**Task:** Design automation that was easy to update as business rules evolved, without requiring code changes.

**Action:**

  * Externalised all business rules from code into a **rules configuration table** in BigQuery that non-technical business owners could update via a Google Sheet.
  * The pipeline read rules from the config table at runtime — no code deployment needed for rule changes.


    
    
    from google.cloud import bigquery
    import pandas as pd
    
    def load_routing_rules():
        """Load current business rules from BigQuery config table."""
        client = bigquery.Client()
        rules = client.query("""
            SELECT claim_type, amount_threshold, routing_path, effective_date
            FROM `project.config.claim_routing_rules`
            WHERE effective_date <= CURRENT_DATE()
              AND (expiry_date IS NULL OR expiry_date > CURRENT_DATE())
            ORDER BY effective_date DESC
        """).to_dataframe()
        return rules
    
    def apply_routing(df_claims, rules):
        """Apply rules from config — no hardcoding."""
        df = df_claims.copy()
        df["routing_path"] = "INTERNAL_DEFAULT"  # fallback
    
        for _, rule in rules.iterrows():
            mask = (
                (df["claim_type"]   == rule["claim_type"]) &
                (df["loss_amount"]  <= rule["amount_threshold"])
            )
            df.loc[mask, "routing_path"] = rule["routing_path"]
    
        return df
    
    rules   = load_routing_rules()
    df_out  = apply_routing(df_claims, rules)
    
    
    
    

**Result:** Business rule changes went from a 2-hour code change + deployment cycle to a 5-minute Google Sheet update that took effect on the next pipeline run. The actuarial team updated rules themselves for the first time — no data engineering involvement needed. Change frequency increased (more experimentation) while engineering burden decreased.

* * *

**Q61. What are the risks of over-automation and how do you mitigate them?**

**Situation:** At AXA Insurance, an overly automated notification system began sending 50+ alert emails per day — including many low-severity "informational" alerts — causing alert fatigue. The team started ignoring emails, and a critical failure was missed.

**Task:** Redesign the alerting to be appropriately automated — not under-alerting or over-alerting.

**Action:**

Identified three main risks of over-automation:

  1. **Alert fatigue** — too many alerts → all alerts get ignored. _Mitigation:_ Tiered severity (CRITICAL/WARNING/INFO), with only CRITICAL going to email, others to a daily digest.
  2. **Hidden dependencies** — fully automated chains where a failure in step 3 of 10 stops everything silently. _Mitigation:_ Each step logs its status to an audit table; a separate monitoring job checks for stalled steps.
  3. **Automation of the wrong thing** — automating a flawed process scales the flaw. _Mitigation:_ Always map and validate the manual process before automating. "Automate a good process, not a bad one faster."
  4. **No human checkpoint** — for high-stakes actions (e.g., deleting records, sending executive reports), require human confirmation rather than full automation.



**Result:** After redesigning the AXA alerting system: email volume dropped from 50+ to 3–5 per day (all genuine CRITICAL issues). Alert response rate increased to 100%. The critical failure miss became impossible because all CRITICAL alerts now required acknowledgement within 30 minutes or auto-escalated.

* * *

**Q62. How did you automate testing before deploying pipeline changes to production?**

**Situation:** At J&J, pipeline deployments to production were done directly after developer testing in dev — no staging environment, no regression tests. Two production incidents in 3 months were traced to untested edge cases.

**Task:** Implement a formal pre-production testing gate that automated validation before any change went live.

**Action:** Built a 3-stage automated testing framework integrated into the Jenkins CI/CD pipeline:

**Stage 1 — Unit tests (runs on every commit):**
    
    
    def test_null_handling():
        df = pd.DataFrame({"loss_amount": [100, None, -50, 200]})
        df_clean = clean_claims(df)
        assert df_clean["loss_amount"].isnull().sum() == 0, "Nulls not handled"
        assert (df_clean["loss_amount"] >= 0).all(), "Negatives not capped"
        print("✅ Unit test passed: null handling")
    
    
    
    

**Stage 2 — Integration tests (runs on PR to main):**

  * Loaded a synthetic dataset with known properties into a test BigQuery project.
  * Ran the full pipeline end-to-end against the synthetic data.
  * Verified output counts, sums, and specific transformed values matched expected results.



**Stage 3 — Parallel validation (runs on staging):**

  * Deployed to staging and ran the pipeline against yesterday's production data.
  * Compared staging output to production baseline using the A/B comparison script (Q24).
  * Any field with >1% unexplained change flagged as a potential regression.



**Result:** Post-implementation: zero production incidents caused by untested code changes in the following 12 months. Developer confidence increased — engineers were willing to refactor more aggressively because regression tests caught mistakes automatically.

* * *

**Q63. How did you automate the onboarding of new data sources into an existing pipeline?**

**Situation:** At J&J, adding a new source system to the BigQuery model previously required a senior engineer to manually: write schema mapping, build ingestion scripts, add DQ checks, update the STTM, and create a Looker Studio data source — a 2-week effort per source.

**Task:** Build a templatised onboarding framework that reduced new source onboarding to a configuration exercise.

**Action:**

  * Created a **source onboarding config schema** (YAML):


    
    
    # new_source_config.yaml
    source_name: "TRAVELERS_CLAIMS_V2"
    connection_type: "SNOWFLAKE"
    connection_secret: "snowflake-travelers-prod"
    source_query: "SELECT * FROM TRAVELERS.CLAIMS WHERE LOAD_DATE = '{run_date}'"
    target_table: "project.bronze.travelers_claims_v2"
    schedule: "0 3 * * *"   # 3am daily
    
    schema:
      - name: claim_id     type: STRING  nullable: false  is_key: true
      - name: policy_id    type: STRING  nullable: false
      - name: loss_amount  type: FLOAT64 nullable: true
      - name: claim_date   type: DATE    nullable: false
      - name: status       type: STRING  nullable: true
    
    dq_rules:
      - field: claim_id    check: NOT_NULL
      - field: loss_amount check: NON_NEGATIVE
      - field: claim_date  check: NOT_FUTURE
    
    
    
    

  * Built a **bootstrap script** that read the YAML and:
    1. Created the BigQuery Bronze table with the specified schema
    2. Generated the ingestion script from a template
    3. Set up the DQ check functions automatically
    4. Registered the source in the pipeline monitoring table
    5. Created a Cloud Scheduler job for the defined schedule



**Result:** New source onboarding time reduced from 2 weeks to 2 days — 1 day to write and validate the config, 1 day to run the parallel validation. Added 3 new sources in 4 months using the framework vs the 2 sources added in the previous 12 months under the manual process.

* * *

**Q64. How do you document automated pipelines so others can maintain them?**

**Situation:** At Travelers Insurance, I was the primary builder of the Snowflake automation. When I was due to rotate to another engagement, the team needed to be able to run and maintain the pipelines independently.

**Task:** Create documentation that would enable any senior analyst to operate and troubleshoot the pipelines without my involvement.

**Action:** Created a **Pipeline Runbook** for each automated workflow, covering:
    
    
    # Claims Silver Refresh — Runbook
    
    ## Overview
    Automated Snowflake Task chain that ingests claims from Bronze,
    validates, transforms, and loads into Silver every 15 minutes.
    
    ## Architecture
    [Claims Bronze] → Stream → [silver_claims_refresh Task] → [Claims Silver]
    
    ## Schedule
    Every 15 minutes via Snowflake Task (SCHEDULE = '15 MINUTE')
    
    ## How to Check Status
    SELECT * FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
      SCHEDULED_TIME_RANGE_START => DATEADD('hour', -2, CURRENT_TIMESTAMP()),
      TASK_NAME => 'SILVER_CLAIMS_REFRESH'
    )) ORDER BY SCHEDULED_TIME DESC LIMIT 10;
    
    ## Common Failures & Fixes
    | Error | Cause | Fix |
    |---|---|---|
    | Statement timeout | Volume spike | Re-run with LARGE warehouse |
    | Stream stale | Task suspended | ALTER TASK ... RESUME |
    | MERGE conflict | Concurrent write | Wait 5 mins, re-run |
    
    ## Escalation
    If unresolvable after 30 mins: contact sddssd.s@gmail.com
    
    
    
    

Additionally, all code was in Git with README files, and a 2-hour handover walkthrough was recorded.

**Result:** The team operated the pipelines independently for 4 months after my rotation. Zero escalations to me. When a new team member joined 3 months later, the runbooks were their primary onboarding resource — they reported being self-sufficient within 3 days.

* * *

**Q65. What is your philosophy on automation in data engineering?**

**Situation:** After 11+ years building automation across Insurance, Geospatial, Finance, and Telecom, I've developed a clear point of view on where automation adds value and where it doesn't.

**Task:** Articulate a coherent automation philosophy grounded in real experience.

**Action/Response:**

My philosophy has four principles:

**1\. Automate the boring, not the complex.** Repetitive, high-frequency, low-judgment tasks are the best automation targets. Ad-hoc analysis, stakeholder conversations, and root-cause investigations require human judgment — automation can support them but shouldn't replace them.

**2\. Automation without monitoring is a liability.** An unmonitored automated pipeline is worse than a manual process — at least a human notices when the manual process fails. Every automation I build has failure detection and alerting built in from day one.

**3\. Simplicity compounds.** A simple automation that's easy to maintain delivers more long-term value than a complex one that nobody understands. At J&J, the externalised routing rules configuration (Q60) was a simple idea but delivered years of value because anyone could update it.

**4\. Automate for the next person, not just yourself.** I've inherited undocumented automation. I know what it costs. Every automation I build is documented, versioned, and tested — because I'll eventually move on, and the person who comes after me shouldn't spend weeks reverse-engineering my work.

**Result:** These principles have shaped how I approach every automation decision. The outcome has been pipelines that are reliable, maintainable, and genuinely trusted by the teams that depend on them — which is ultimately the only measure of a successful automation.

* * *

## 📂 Section 7: Advanced Automation Scenarios (Q66–Q80)

* * *

**Q66. How did you automate multi-environment pipeline promotion (dev → staging → prod)?**

**Situation:** At J&J, the same pipeline code was maintained in 3 separate copies across dev, staging, and prod environments — changes had to be manually re-applied in each, leading to environment drift and inconsistency.

**Task:** Automate the promotion of pipeline code across environments via CI/CD.

**Action:**
    
    
    # .github/workflows/pipeline_deploy.yml (adapted for Bitbucket/Jenkins)
    stages:
      - validate
      - deploy_dev
      - test_staging
      - deploy_prod
    
    validate:
      script:
        - python -m pytest tests/unit/ -v
        - python validate_sql.py --dir sql/ --project jnj-dev
    
    deploy_dev:
      script:
        - python deploy_pipeline.py --env dev --project jnj-dev
      only: [feature/*]
    
    test_staging:
      script:
        - python deploy_pipeline.py --env staging --project jnj-staging
        - python run_integration_tests.py --env staging
        - python compare_outputs.py --baseline prod --current staging
      only: [main]
    
    deploy_prod:
      script:
        - python deploy_pipeline.py --env prod --project jnj-prod
      when: manual   # requires human approval before production push
      only: [main]
    
    
    
    

Environment configuration injected at deploy time via Secret Manager — no environment-specific code in the repository.

**Result:** Environment drift eliminated. Every production deployment went through validated staging first. The manual approval gate for production gave engineers confidence without slowing down the pipeline. Deployment frequency increased from monthly to weekly as the risk of deployments decreased.

* * *

**Q67. How did you automate handling of API rate limits in data ingestion pipelines?**

**Situation:** At Google Street View, the internal fleet management API had a rate limit of 100 requests/minute. The ingestion pipeline was making 500+ calls per run, causing 429 errors and partial data loads.

**Task:** Build a rate-limit-aware ingestion client that maximised throughput without exceeding API limits.

**Action:**
    
    
    import time
    import requests
    from threading import Semaphore
    from concurrent.futures import ThreadPoolExecutor
    
    class RateLimitedAPIClient:
        def __init__(self, base_url, token, requests_per_minute=90):
            self.base_url = base_url
            self.headers  = {"Authorization": f"Bearer {token}"}
            self.semaphore = Semaphore(requests_per_minute)
            self.interval  = 60 / requests_per_minute  # seconds between calls
    
        def get(self, endpoint, params=None, max_retries=3):
            for attempt in range(max_retries):
                self.semaphore.acquire()
                try:
                    resp = requests.get(
                        f"{self.base_url}/{endpoint}",
                        headers=self.headers, params=params, timeout=15
                    )
                    if resp.status_code == 429:
                        retry_after = int(resp.headers.get("Retry-After", 60))
                        print(f"Rate limited. Waiting {retry_after}s...")
                        time.sleep(retry_after)
                        continue
                    resp.raise_for_status()
                    return resp.json()
                finally:
                    time.sleep(self.interval)
                    self.semaphore.release()
            raise RuntimeError(f"Failed after {max_retries} retries")
    
    client = RateLimitedAPIClient("https://fleet-api.internal", API_TOKEN, requests_per_minute=90)
    
    # Fetch 500 vehicles using thread pool (controlled concurrency)
    vehicle_ids = range(1, 501)
    with ThreadPoolExecutor(max_workers=5) as executor:
        results = list(executor.map(
            lambda vid: client.get(f"vehicles/{vid}"), vehicle_ids
        ))
    
    
    
    

**Result:** 429 errors eliminated entirely. Full 500-vehicle dataset loaded within each run. The rate-limiting client was made reusable and applied to 3 other API integrations in the Street View analytics stack. Data ingestion completeness reached 100% from ~85% previously.

* * *

**Q68. How did you automate historical data backfill when a new pipeline was introduced?**

**Situation:** At Travelers Insurance, the new CDC pipeline replaced a batch load that had run for 2 years. All historical records needed to be backfilled into the new Snowflake structure before the new pipeline could go live.

**Task:** Automate a one-time historical backfill of 2 years of claims data without impacting the live system.

**Action:**
    
    
    import snowflake.connector
    from datetime import date, timedelta
    
    conn = snowflake.connector.connect(
        user=USER, password=PASSWORD, account=ACCOUNT,
        warehouse="LARGE_WH", database="TRAVELERS_DW"
    )
    
    def backfill_date_range(start_date, end_date, batch_days=7):
        """Backfill historical data in weekly batches to manage memory and cost."""
        cursor  = conn.cursor()
        current = start_date
    
        while current < end_date:
            batch_end = min(current + timedelta(days=batch_days), end_date)
    
            cursor.execute(f"""
                INSERT INTO SILVER.CLAIMS (
                  SELECT * FROM BRONZE.CLAIMS_HISTORICAL
                  WHERE claim_date BETWEEN '{current}' AND '{batch_end}'
                    AND claim_id NOT IN (SELECT claim_id FROM SILVER.CLAIMS)
                )
            """)
            rows = cursor.fetchone()[0]
            print(f"Backfilled {rows} rows: {current} → {batch_end}")
    
            # Log progress
            cursor.execute(f"""
                INSERT INTO MONITORING.BACKFILL_LOG
                VALUES ('{current}', '{batch_end}', {rows}, CURRENT_TIMESTAMP())
            """)
            current = batch_end + timedelta(days=1)
    
        print("Backfill complete.")
    
    backfill_date_range(
        start_date=date(2024, 1, 1),
        end_date=date(2026, 3, 31)
    )
    
    
    
    

**Result:** 2 years of historical data backfilled in 4 hours using automated weekly batches. The live pipeline went live immediately after with a complete historical foundation. The fraud team could run 2-year trend analyses from day one. Zero manual data loading required.

* * *

**Q69. How did you automate the generation of synthetic test data?**

**Situation:** At J&J, integration tests for the BigQuery pipeline needed realistic claims data — but using real production data in a test environment was a GDPR violation. Creating realistic synthetic data manually was time-consuming.

**Task:** Automate the generation of statistically realistic synthetic claims data for testing.

**Action:**
    
    
    import pandas as pd
    import numpy as np
    from faker import Faker
    from datetime import datetime, timedelta
    import random
    
    fake = Faker("en-IE")  # Irish locale for realistic names/addresses
    
    def generate_synthetic_claims(n=10000, seed=42):
        np.random.seed(seed)
        random.seed(seed)
    
        claim_types = ["MOTOR", "PROPERTY", "LIABILITY", "HEALTH"]
        statuses    = ["OPEN", "CLOSED", "FRAUD", "PENDING"]
        regions     = ["Dublin", "Cork", "Galway", "Limerick", "Waterford"]
    
        return pd.DataFrame({
            "claim_id":          [f"CLM-{i:07d}" for i in range(1, n+1)],
            "policy_id":         [f"POL-{random.randint(1, n//3):07d}" for _ in range(n)],
            "policy_holder_name": [fake.name() for _ in range(n)],
            "claim_type":        np.random.choice(claim_types, n, p=[0.4, 0.3, 0.2, 0.1]),
            "loss_amount":       np.random.lognormal(mean=8, sigma=1.5, size=n).round(2),
            "claim_date":        [
                (datetime(2024,1,1) + timedelta(days=random.randint(0, 730))).strftime("%Y-%m-%d")
                for _ in range(n)
            ],
            "status":            np.random.choice(statuses, n, p=[0.3, 0.5, 0.05, 0.15]),
            "region":            np.random.choice(regions, n),
            "handler_id":        [f"H{random.randint(1, 50):03d}" for _ in range(n)],
        })
    
    df_synthetic = generate_synthetic_claims(n=10000)
    df_synthetic.to_parquet("test_data/synthetic_claims_10k.parquet", index=False)
    print(f"Generated {len(df_synthetic):,} synthetic claim records.")
    print(df_synthetic.describe())
    
    
    
    

**Result:** Realistic synthetic test dataset generated in under 30 seconds. Integration tests could run against meaningful data without any GDPR risk. Test coverage increased from 40% to 95% of transformation scenarios because edge cases could be deliberately injected into the synthetic data.

* * *

**Q70. How did you automate cost optimisation in cloud data platforms?**

**Situation:** At J&J, the monthly BigQuery bill was increasing 15% month-over-month as the analytics team grew and ran increasingly expensive queries. There was no visibility into what was driving costs until the monthly invoice.

**Task:** Build an automated cost governance system that maintained quality analytics while controlling spend.

**Action:** Built a 3-layer cost control system:

**Layer 1 — Query cost estimation before execution:**
    
    
    from google.cloud import bigquery
    
    def estimate_query_cost(query, project_id="jnj-prod"):
        client = bigquery.Client()
        job_config = bigquery.QueryJobConfig(dry_run=True, use_query_cache=False)
        job = client.query(query, job_config=job_config)
        gb_scanned = job.total_bytes_processed / (1024**3)
        cost_usd   = gb_scanned * 0.005  # $5/TB
        print(f"Estimated: {gb_scanned:.2f} GB scanned (~${cost_usd:.4f})")
        if cost_usd > 1.0:
            confirm = input(f"⚠️ Query costs ~${cost_usd:.2f}. Proceed? (y/n): ")
            if confirm.lower() != "y":
                raise SystemExit("Query cancelled by user.")
        return job
    
    
    
    

**Layer 2 — Automated slot quota per team:** Set BigQuery **reservation slots** per workgroup — actuarial team got 200 slots, BI team got 100 — preventing any one team from monopolising shared capacity.

**Layer 3 — Weekly cost report per user (automated, from Q49 monitoring view).**

**Result:** Monthly BigQuery costs reduced by 55% within 3 months. The per-user cost report created healthy awareness — analysts started adding partition filters without being asked. No queries were blocked (autonomy preserved), but cost visibility changed behaviour.

* * *

**Q71. How did you automate disaster recovery testing for data pipelines?**

**Situation:** At AXA Insurance, the disaster recovery plan existed as a document but had never been tested. When a pipeline incident occurred, the team discovered that the recovery steps were outdated and incomplete — adding hours to the recovery time.

**Task:** Automate regular DR testing so the recovery plan was always accurate and the team stayed practiced.

**Action:** Built a monthly automated DR drill:
    
    
    import subprocess
    from datetime import datetime
    
    def run_dr_drill(pipeline_name, delta_path):
        """Simulate a pipeline failure and test recovery."""
        print(f"\n{'='*50}")
        print(f"DR DRILL: {pipeline_name} — {datetime.now()}")
        print('='*50)
    
        # Step 1: Record current state
        pre_count = get_row_count(delta_path)
        print(f"Pre-drill row count: {pre_count:,}")
    
        # Step 2: Simulate failure (overwrite with empty frame)
        print("Simulating pipeline failure (overwrite with empty)...")
        spark.createDataFrame([], schema=get_schema(delta_path)) \
             .write.format("delta").mode("overwrite").save(delta_path)
    
        # Step 3: Execute recovery procedure
        start = datetime.now()
        restore_to_snapshot(delta_path, get_latest_snapshot_label(delta_path))
        recovery_time = (datetime.now() - start).seconds
    
        # Step 4: Validate recovery
        post_count = get_row_count(delta_path)
        success = pre_count == post_count
    
        print(f"\nDR Drill Result:")
        print(f"  Recovery time:     {recovery_time}s")
        print(f"  Rows before drill: {pre_count:,}")
        print(f"  Rows after restore: {post_count:,}")
        print(f"  Status: {'✅ SUCCESS' if success else '❌ FAILED'}")
    
        log_dr_result(pipeline_name, recovery_time, success)
    
    run_dr_drill("claims_silver_refresh", DELTA_PATH)
    
    
    
    

**Result:** DR drill ran automatically on the 1st of each month. Recovery procedure was validated as working. Mean recovery time measured and tracked — improved from 18 hours (first drill) to under 10 minutes (month 4, after automated snapshot implementation). The team was confident in the recovery process for the first time.

* * *

**Q72. How did you automate resource scaling based on pipeline workload?**

**Situation:** At Travelers Insurance, the Snowflake virtual warehouse was sized statically — `SMALL` for routine loads, but month-end actuarial queries required `LARGE` to run in reasonable time. Manually resizing before and after peak windows was forgotten half the time.

**Task:** Automate warehouse scaling based on the type of workload and time of day.

**Action:**
    
    
    -- Scheduled warehouse scaling via Snowflake Tasks
    
    -- Scale UP before month-end actuarial window (last day of month, 6pm)
    CREATE OR REPLACE TASK scale_up_month_end
      WAREHOUSE = XSMALL_WH
      SCHEDULE  = 'USING CRON 0 18 28-31 * * UTC'
    AS
      ALTER WAREHOUSE ANALYTICS_WH SET WAREHOUSE_SIZE = 'LARGE';
    
    -- Scale DOWN after month-end window (1st of month, 10am)
    CREATE OR REPLACE TASK scale_down_post_month_end
      WAREHOUSE = XSMALL_WH
      SCHEDULE  = 'USING CRON 0 10 1 * * UTC'
    AS
      ALTER WAREHOUSE ANALYTICS_WH SET WAREHOUSE_SIZE = 'SMALL';
    
    -- Auto-suspend when idle (always on)
    ALTER WAREHOUSE ANALYTICS_WH SET
      AUTO_SUSPEND = 120    -- suspend after 2 min idle
      AUTO_RESUME  = TRUE;  -- resume instantly on query
    
    ALTER TASK scale_up_month_end   RESUME;
    ALTER TASK scale_down_post_month_end RESUME;
    
    
    
    

**Result:** Month-end actuarial queries ran on `LARGE` automatically — response times improved from 20 minutes to under 2 minutes. Non-peak periods ran on `SMALL` with auto-suspend, reducing idle compute costs by ~40%. Zero manual resizing required.

* * *

**Q73. How did you automate cross-team data sharing and access management?**

**Situation:** At J&J, the actuarial team in Dublin needed access to claims data held in a BigQuery project managed by the US team. Previously, sharing required email chains, manual IAM changes, and often weeks of delays.

**Task:** Build an automated, auditable data sharing mechanism that gave the right teams access to the right data without manual IAM management.

**Action:** Used **BigQuery Analytics Hub** for controlled cross-project data sharing:
    
    
    from google.cloud import analyticshub_v1
    
    def create_data_exchange(exchange_name, project_id, location="EU"):
        client = analyticshub_v1.AnalyticsHubServiceClient()
        parent = f"projects/{project_id}/locations/{location}"
    
        exchange = analyticshub_v1.DataExchange(
            display_name = exchange_name,
            description  = "JnJ Claims Data — Actuarial Access",
            sharing_environment_config = analyticshub_v1.SharingEnvironmentConfig(
                default_exchange_config = {}
            )
        )
        return client.create_data_exchange(parent=parent, data_exchange=exchange)
    
    def add_subscriber(exchange_name, subscriber_project, subscriber_email):
        """Grant a team access to the shared dataset."""
        # Add IAM binding to allow subscriber to link the dataset
        policy = get_iam_policy(exchange_name)
        policy.bindings.append({
            "role":    "roles/analyticshub.subscriber",
            "members": [f"user:{subscriber_email}"]
        })
        set_iam_policy(exchange_name, policy)
        print(f"✅ {subscriber_email} granted subscriber access to {exchange_name}")
    
    # Automated by HR/onboarding system: when a user joins actuarial team, this fires
    add_subscriber("jnj-claims-exchange", "jnj-actuarial-eu", "new.actuary@jnj.com")
    
    
    
    

All access grants were logged to an audit table and reviewed monthly.

**Result:** Cross-team data sharing time reduced from weeks to hours. Actuarial teams in 3 regions could access shared claims data without IT involvement. All access was auditable and revocable centrally. Zero oversharing incidents — each team only saw the datasets relevant to their role.

* * *

**Q74. How did you automate the decommissioning of legacy data assets?**

**Situation:** At J&J, after the BigQuery consolidation was complete, 12 legacy tables remained in BigQuery that were no longer used — but nobody was confident enough to delete them because there was no visibility into whether they were still being queried.

**Task:** Build an automated usage monitoring system that safely identified and decommissioned unused assets.

**Action:**
    
    
    -- BigQuery: table usage monitoring
    CREATE OR REPLACE VIEW monitoring.table_usage_90d AS
    SELECT
      destination_table.dataset_id AS dataset,
      destination_table.table_id   AS table_name,
      COUNT(DISTINCT job_id)        AS query_count_90d,
      MAX(creation_time)            AS last_queried,
      DATE_DIFF(CURRENT_DATE(), DATE(MAX(creation_time)), DAY) AS days_since_last_query
    FROM `region-eu.INFORMATION_SCHEMA.JOBS`
    WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
      AND job_type = 'QUERY'
      AND state    = 'DONE'
    GROUP BY 1, 2;
    
    -- Flag candidates for decommission
    SELECT
      dataset, table_name, days_since_last_query,
      CASE
        WHEN days_since_last_query > 60 THEN '🔴 Decommission Candidate'
        WHEN days_since_last_query > 30 THEN '⚠️ Review'
        ELSE '✅ Active'
      END AS recommendation
    FROM monitoring.table_usage_90d
    ORDER BY days_since_last_query DESC;
    
    
    
    

Automated process:

  1. Monthly report generated and sent to data owners with the usage data.
  2. After 90 days of zero queries, tables moved to an archive dataset (not deleted).
  3. After a further 30 days in archive with no queries, automated deletion.



**Result:** 8 of 12 legacy tables were safely decommissioned over 4 months, reducing BigQuery storage costs by 25%. The 4 retained tables turned out to still be used by 2 external reports nobody had documented. The automated process caught this before deletion — those tables were properly documented and retained.

* * *

**Q75. How have you used automation to improve data observability?**

**Situation:** At Google Street View, the engineering team had no unified view of pipeline health, data freshness, or anomaly counts across the 8 pipelines in the analytics stack. Each team had their own ad-hoc monitoring — or none at all.

**Task:** Build a unified observability layer that gave a single view of the health of the entire data platform.

**Action:** Built a **Data Observability Dashboard** in Looker Studio backed by BigQuery:
    
    
    -- Unified observability metrics table (auto-populated by monitoring tasks)
    CREATE OR REPLACE TABLE monitoring.observability_metrics (
      metric_time     TIMESTAMP DEFAULT CURRENT_TIMESTAMP(),
      pipeline_name   STRING,
      metric_type     STRING,   -- 'FRESHNESS' | 'ROW_COUNT' | 'NULL_RATE' | 'LATENCY'
      metric_value    FLOAT64,
      threshold       FLOAT64,
      status          STRING,   -- 'OK' | 'WARNING' | 'CRITICAL'
      details         STRING
    );
    
    -- Example: automated freshness metric insert (runs every 15 mins per pipeline)
    INSERT INTO monitoring.observability_metrics
      (pipeline_name, metric_type, metric_value, threshold, status, details)
    SELECT
      'claims_silver'                                        AS pipeline_name,
      'FRESHNESS'                                            AS metric_type,
      TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), MAX(updated_at), MINUTE) AS metric_value,
      30                                                     AS threshold,
      CASE WHEN TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), MAX(updated_at), MINUTE) > 30
           THEN 'CRITICAL' ELSE 'OK' END                    AS status,
      CONCAT('Last refresh: ', CAST(MAX(updated_at) AS STRING)) AS details
    FROM `project.silver.claims`;
    
    
    
    

Looker Studio dashboard showed:

  * **Pipeline health RAG status** for all 8 pipelines
  * **Freshness trends** (last 24 hours)
  * **Row count trends** (anomaly detection)
  * **Active alerts** with timestamps



**Result:** The observability dashboard became the "source of truth" for pipeline health. On-call engineers checked it first for any data complaint. Mean time to detect issues reduced from hours to under 15 minutes across all pipelines. Leadership viewed it as evidence of a mature data operations capability.

* * *

## 📂 Section 8: Final 25 — Mixed Rapid-Fire Scenarios (Q76–Q100)

* * *

**Q76. How would you automate detection of PII in an unknown dataset?**

**S:** New data source at AXA with unclear field contents. **T:** Identify PII fields before granting analyst access. **A:** Built a regex + ML-based PII scanner:
    
    
    import re
    PII_PATTERNS = {
        "EMAIL":    r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}",
        "PHONE":    r"(\+353|0)\d{9}",
        "EIRCODE":  r"[A-Z]\d{2}\s?[A-Z0-9]{4}",
        "PPS":      r"\d{7}[A-Z]{1,2}"
    }
    def scan_for_pii(df):
        results = {}
        for col in df.select_dtypes(include="object").columns:
            sample = df[col].dropna().astype(str).head(100)
            for pii_type, pattern in PII_PATTERNS.items():
                hits = sample.str.contains(pattern, regex=True).sum()
                if hits > 0:
                    results[col] = pii_type
                    break
        return results
    
    
    
    

**R:** Identified 4 untagged PII columns in 3 minutes. Fields masked before analyst access granted. GDPR compliance maintained.

* * *

**Q77. How did you automate SSD lifecycle alerts at Google Street View?**

**S:** Collection vehicles had SSDs failing mid-run with no advance warning. **T:** Predictive alerting before failure. **A:** Cloud Function pulled SMART telemetry hourly into BigQuery. Calculated projected days-to-failure based on utilisation rate. Alert fired for SSDs with <30 days remaining. **R:** Reactive failures eliminated. Replacement scheduling became proactive. Collection uptime improved significantly.

* * *

**Q78. How did you automate Snowflake stream consumption monitoring?**

**S:** Snowflake Streams that weren't consumed within a certain period became stale — causing data loss. **T:** Alert when a stream was at risk of going stale. **A:**
    
    
    CREATE OR REPLACE TASK monitor_stream_staleness
      SCHEDULE = '1 HOUR'
    AS
    SELECT SYSTEM$SEND_EMAIL('data-ops@company.com',
      'Stream Staleness Warning',
      'A Snowflake Stream has not been consumed in 6+ hours. Risk of data loss.')
    WHERE (
      SELECT DATEDIFF('hour', MIN(SYSTEM$STREAM_GET_TABLE_TIMESTAMP('CLAIMS_STREAM')),
             CURRENT_TIMESTAMP())
    ) > 6;
    
    
    
    

**R:** Stream staleness incidents prevented. Zero data loss from stale streams in the following 12 months.

* * *

**Q79. How did you automate data contract enforcement between teams?**

**S:** At J&J, the analytics team depended on the source system team's data format. Schema changes broke pipelines without warning. **T:** Formalise and automate data contract validation. **A:** Defined explicit data contracts in JSON Schema. Validation ran automatically on every source system delivery:
    
    
    import jsonschema, json
    schema = json.load(open("contracts/claims_contract.json"))
    for record in incoming_records:
        jsonschema.validate(record, schema)  # raises on violation
    
    
    
    

**R:** Schema violations caught at ingestion before any transformation ran. Source team had clear, machine-readable contracts. Contract violations reduced from 6/month to 1/month within a quarter.

* * *

**Q80. How did you use automation to speed up BigQuery query development?**

**S:** At J&J, analysts spent significant time running expensive queries during development — accidentally scanning full tables while testing. **T:** Reduce development query costs without restricting access. **A:** Introduced a development dataset with a 10% sample of all production tables, automatically refreshed weekly:
    
    
    CREATE OR REPLACE TABLE dev.claims_sample AS
    SELECT * FROM prod.claims TABLESAMPLE SYSTEM (10 PERCENT);
    
    
    
    

Added a lint check that warned if a query referenced prod tables without a WHERE clause filtering on the partition column. **R:** Development query costs reduced by 70%. Query development speed increased as analysts iterated on cheap sample data before promoting to production queries.

* * *

**Q81. How did you automate Excel-to-BigQuery data ingestion for business users?**

**S:** At J&J, actuarial analysts submitted monthly adjustment files as Excel spreadsheets that were manually uploaded to BigQuery by an engineer. **T:** Allow business users to self-serve the upload without engineering involvement. **A:** Built a Google Forms → AppScript → BigQuery pipeline: users filled in a structured form, AppScript validated the submission and loaded it to a staging BigQuery table, a Cloud Function ran DQ checks and promoted to production if valid. **R:** Engineer involvement for routine uploads eliminated. Actuarial team self-sufficient for monthly adjustments. Upload-to-availability time reduced from 2 days to 30 minutes.

* * *

**Q82. How did you automate the detection of business rule violations in real time?**

**S:** At Travelers, certain claim types had regulatory caps on loss amounts. Violations were discovered during monthly audits — too late to correct before regulatory reporting. **T:** Detect violations at ingestion time, not audit time. **A:** Added regulatory rule checks to the Silver layer DQ gate:
    
    
    REGULATORY_CAPS = {"MOTOR": 500000, "LIABILITY": 1000000, "PROPERTY": 250000}
    violations = df[df.apply(
        lambda r: r["loss_amount"] > REGULATORY_CAPS.get(r["claim_type"], float("inf")), axis=1
    )]
    if not violations.empty:
        quarantine(violations)
        alert_compliance_team(violations)
    
    
    
    

**R:** Regulatory cap violations caught at ingestion. Compliance team alerted within 15 minutes. Zero violations reached regulatory reporting in the following 12 months.

* * *

**Q83. How did you automate data warehouse performance benchmarking?**

**S:** After migrating J&J from SQL Server to BigQuery, the CTO asked for a formal performance comparison to justify the investment. **T:** Automate a repeatable benchmark suite across both systems. **A:** Built a benchmarking script that ran 20 standard queries (point lookups, aggregations, joins, window functions) against both systems and recorded execution times:
    
    
    def benchmark_query(name, query, conn):
        start = time.time()
        pd.read_sql(query, conn)
        return {"query": name, "elapsed_s": round(time.time() - start, 2)}
    
    
    
    

Results written to BigQuery. Looker Studio showed side-by-side comparison per query category. **R:** BigQuery outperformed SQL Server on 18/20 query types. Average 7× faster on aggregation-heavy actuarial queries. CTO approved the full migration based on the benchmark report.

* * *

**Q84. How did you automate the identification of unused dashboard elements?**

**S:** At Google Street View, the Looker Studio dashboards had grown to 40+ charts over 3 years. Many were never viewed but cluttered the interface and made maintenance harder. **T:** Identify which dashboard elements were actually used. **A:** Used Looker Studio's activity logs (exported to BigQuery via the Looker Studio API) to track view events per chart element. Charts with zero views in 90 days were flagged for review. **R:** 14 unused charts identified and removed. Dashboard load time improved. Users reported the simplified layout was easier to navigate.

* * *

**Q85. How did you automate testing of SQL views after schema changes?**

**S:** At J&J, when a source table schema changed, SQL views built on top of it silently returned empty results or wrong data because column names had changed. **T:** Automatically test all dependent views after any schema change. **A:** Built a dependency map of all views → source tables. When a source table changed, the CI/CD pipeline automatically executed a "smoke test" query on all dependent views:
    
    
    def smoke_test_view(view_name):
        result = client.query(f"SELECT COUNT(*) AS n FROM `{view_name}` LIMIT 1").to_dataframe()
        assert result["n"].iloc[0] > 0, f"View {view_name} returned 0 rows — check dependencies"
        print(f"✅ {view_name}: {result['n'].iloc[0]:,} rows")
    
    
    
    

**R:** Schema-change-induced view failures detected automatically in CI before reaching production. 2 silent view failures caught in the first month of implementation.

* * *

**Q86. How did you automate monitoring of Power BI report usage?**

**S:** At AXA, leadership questioned whether the Power BI reports the data team spent significant time maintaining were actually being used. **T:** Provide usage analytics to prioritise maintenance effort. **A:** Used the Power BI Activity Log API to pull report view events into BigQuery daily:
    
    
    import requests
    activity = requests.get(
        "https://api.powerbi.com/v1.0/myorg/admin/activityevents",
        headers={"Authorization": f"Bearer {token}"},
        params={"startDateTime": start, "endDateTime": end}
    ).json()
    
    
    
    

Built a usage dashboard showing views per report per week, unique users, and last-viewed date. **R:** 3 reports identified with zero views in 90 days — retired. 2 high-traffic reports identified for priority maintenance investment. Report maintenance time allocated more effectively.

* * *

**Q87. How did you automate CDC for a system without native change tracking?**

**S:** At J&J, one legacy source system had no CDC capability — it could only produce full extracts. Re-loading 50M rows nightly was too expensive. **T:** Implement CDC manually for a system without built-in change tracking. **A:** Used a checksum-based approach:
    
    
    -- Add MD5 hash of all columns to detect changes
    SELECT
      claim_id,
      MD5(CONCAT(status, CAST(loss_amount AS STRING), handler_id)) AS row_hash
    FROM claims_bronze
    
    
    
    

Compared current hashes to previous load's hashes — only rows with changed hashes were re-processed. New rows (no previous hash) were inserted. **R:** Processing reduced from 50M rows to an average of 200K rows per night — a 99.6% reduction in rows processed. Load time dropped from 4 hours to 8 minutes.

* * *

**Q88. How did you automate data enrichment in a pipeline?**

**S:** At J&J, the claims data contained postcode fields but no geographic region information — needed for regional actuarial analysis. **T:** Automatically enrich claims records with region data based on postcode. **A:**
    
    
    import requests
    def get_region_from_eircode(eircode):
        resp = requests.get(f"https://api.eircode.ie/v1/lookup?code={eircode}")
        return resp.json().get("county", "UNKNOWN") if resp.ok else "UNKNOWN"
    
    # Vectorised for performance (batch API calls)
    df["region"] = df["postcode"].map(
        lambda code: get_region_from_eircode(code) if pd.notna(code) else "UNKNOWN"
    )
    
    
    
    

Results cached in a BigQuery lookup table to avoid redundant API calls for the same postcodes. **R:** 95% of claims records enriched with region data automatically. Regional actuarial analysis became possible for the first time. API call volume minimised by caching — lookup table hit rate reached 92%.

* * *

**Q89. How did you automate Snowflake RBAC management for a growing team?**

**S:** At Travelers, the analytics team grew from 5 to 18 users in 6 months. Managing role assignments manually in Snowflake became error-prone — users sometimes had too much access, sometimes too little. **T:** Automate RBAC based on a source-of-truth role registry. **A:** Maintained a Google Sheet (accessible to managers) defining user → role mappings. A Python script synced this to Snowflake nightly:
    
    
    for user, role in role_registry.items():
        cursor.execute(f"GRANT ROLE {role} TO USER {user}")
        cursor.execute(f"REVOKE ALL ROLES FROM USER {user} EXCEPT {role}")
    
    
    
    

**R:** Access mismatches eliminated. Managers could self-serve role changes via the Google Sheet. Security audit found zero over-privileged accounts — the first clean audit in 2 years.

* * *

**Q90. How did you automate end-of-month data close processes?**

**S:** At Travelers Insurance, month-end close required a specific sequence: freeze new incoming data, finalise aggregations, lock the Gold layer, generate actuarial reports, then re-open for next month. Done manually, it took a full day and was error-prone if steps were missed. **T:** Automate the month-end close sequence with locks and validation gates. **A:** Built a stored procedure in Snowflake that executed the full sequence with built-in validation at each step — aborting and alerting if any step failed. Used a `PIPELINE_STATE` table to track close status and prevent the sequence from running twice. **R:** Month-end close time reduced from 8 hours (manual) to 45 minutes (automated). Zero missed steps. Actuarial reports ready 7 hours earlier each month.

* * *

**Q91. How did you automate data lineage tracking without a dedicated lineage tool?**

**S:** At Cisco/Wipro, there was no budget for a lineage tool. Stakeholders still needed to understand where data came from. **T:** Build a lightweight lineage tracking system from available tools. **A:** Added a `source_system` and `pipeline_step` column to every table in the SQL Server warehouse. Every ETL step wrote its name and source to these columns on insert. Built a recursive SQL query that traced any record back to its origin:
    
    
    WITH lineage AS (
      SELECT table_name, source_system, pipeline_step, 1 AS depth FROM staging
      UNION ALL
      SELECT t.table_name, t.source_system, t.pipeline_step, l.depth + 1
      FROM tables t JOIN lineage l ON t.source_table = l.table_name
    )
    SELECT * FROM lineage ORDER BY depth;
    
    
    
    

**R:** Full lineage for any record traceable in seconds. Audit requests that previously took 2 days were answered in minutes. The lightweight approach was used for 2 years before the team invested in a dedicated lineage tool.

* * *

**Q92. How did you automate the validation of business metrics against known benchmarks?**

**S:** At J&J, new pipeline outputs were validated manually against "known good" benchmarks — historical averages that a senior analyst kept in a spreadsheet. **T:** Automate benchmark comparison so validation ran on every pipeline execution. **A:** Stored benchmarks in a BigQuery table with expected ranges (min, max, typical):
    
    
    SELECT
      m.metric_name,
      m.current_value,
      b.typical_value,
      b.min_acceptable,
      b.max_acceptable,
      CASE WHEN m.current_value BETWEEN b.min_acceptable AND b.max_acceptable
           THEN '✅ PASS' ELSE '❌ OUT OF RANGE' END AS validation_status
    FROM current_metrics m
    JOIN metric_benchmarks b USING (metric_name);
    
    
    
    

**R:** Benchmark validation ran in seconds after every Gold layer build. Out-of-range metrics flagged before any report was generated. Caught 2 calculation errors in the first month that would have produced incorrect actuarial figures.

* * *

**Q93. How did you automate the cleanup of temporary and intermediate pipeline tables?**

**S:** At J&J, staging tables created during pipeline runs accumulated indefinitely — consuming BigQuery storage and making the environment cluttered. **T:** Automatically clean up temporary assets after pipeline completion. **A:**
    
    
    from google.cloud import bigquery
    from datetime import datetime, timedelta
    
    client = bigquery.Client()
    
    def cleanup_temp_tables(dataset_id, prefix="tmp_", older_than_hours=24):
        tables = client.list_tables(dataset_id)
        deleted = 0
        for table in tables:
            if table.table_id.startswith(prefix):
                full_table = client.get_table(f"{dataset_id}.{table.table_id}")
                age = datetime.utcnow().replace(tzinfo=None) - \
                      full_table.created.replace(tzinfo=None)
                if age > timedelta(hours=older_than_hours):
                    client.delete_table(full_table)
                    print(f"Deleted: {table.table_id} (age: {age})")
                    deleted += 1
        print(f"Cleanup complete: {deleted} tables removed.")
    
    cleanup_temp_tables("project.staging", prefix="tmp_", older_than_hours=24)
    
    
    
    

**R:** Storage costs reduced by 18% in the first month. Dataset clutter eliminated — analysts could navigate the environment without wading through hundreds of stale temporary tables. Ran nightly via Cloud Scheduler.

* * *

**Q94. How did you automate data dictionary updates when schemas changed?**

**S:** At J&J, when a new column was added to a BigQuery table, the data dictionary was only updated if someone remembered to do it — which was often weeks later, causing confusion among analysts. **T:** Automatically detect new or changed columns and prompt for documentation. **A:** A Cloud Function triggered by BigQuery schema change events compared the current schema to the documented schema in the dictionary:
    
    
    def detect_undocumented_fields(table_id, dictionary_table):
        current_fields = {f.name for f in client.get_table(table_id).schema}
        documented = {r["field_name"] for r in
                      client.query(f"SELECT field_name FROM `{dictionary_table}`"
                                   f"WHERE table_name = '{table_id}'").result()}
        new_fields = current_fields - documented
        if new_fields:
            send_alert(f"Undocumented fields in {table_id}: {new_fields}")
    
    
    
    

**R:** New fields flagged for documentation within 24 hours of creation. Dictionary coverage maintained at 95%+ automatically. Analyst confusion about undocumented fields effectively eliminated.

* * *

**Q95. How would you automate the monitoring of a machine learning model's data inputs?**

**S:** Hypothetical but grounded in Travelers fraud detection work. If an ML model used claims features as inputs, drift in those features would degrade model performance without any code change. **T:** Monitor input feature distributions to detect model-relevant drift. **A:** Tracked summary statistics (mean, std, null rate, value distribution) for each model input feature on each daily run. Compared to a rolling 30-day baseline using KS-test (see Q43). Flagged if any feature drifted beyond threshold. **R:** Would catch upstream data changes that could silently degrade model performance — allowing the data team to alert the model owner before fraud detection accuracy degraded rather than after.

* * *

**Q96. How did you automate notification of data pipeline SLA breaches to business owners?**

**S:** At AXA, the actuarial team expected refreshed data by 7am daily. When pipelines were late, they discovered it themselves — by which point valuable working time had been lost. **T:** Proactively notify business owners the moment an SLA was at risk of breach. **A:** Built a pre-SLA warning that fired 30 minutes before the SLA deadline if the pipeline wasn't on track:
    
    
    # At 6:30am: predict if 7am SLA will be met based on current pipeline progress
    def predict_sla_compliance(pipeline_name, sla_time, current_time, expected_duration_mins):
        time_remaining = (sla_time - current_time).seconds / 60
        if expected_duration_mins > time_remaining:
            send_alert(f"⚠️ SLA at risk: {pipeline_name} may not complete by {sla_time}")
    
    
    
    

**R:** Business owners received pre-emptive warnings, allowing them to prepare alternatives before the SLA breach occurred. The actuarial team reported significantly reduced stress around month-end data availability.

* * *

**Q97. How did you automate the prioritisation of data quality issue resolution?**

**S:** At J&J, the DQ monitoring system generated a backlog of data quality issues across 12 tables. With no prioritisation, the team worked on issues in random order — sometimes spending time on low-impact problems while high-impact ones waited. **T:** Automatically prioritise DQ issues by business impact. **A:** Built a scoring model for DQ issues based on: table importance (Gold > Silver > Bronze), field criticality (key field > non-key), null rate (higher = more urgent), downstream consumers (more dependents = more urgent). Issues scored and ranked automatically in a Looker Studio DQ backlog dashboard. **R:** Engineering time focused on highest-impact issues first. Time-to-resolution for critical DQ issues reduced by 60%. Actuarial report accuracy improved as the highest-impact fields were prioritised.

* * *

**Q98. How did you automate the handling of schema evolution in Delta Lake?**

**S:** At AXA, as the claims data model evolved, new columns were occasionally added to source files. Without schema evolution handling, the PySpark pipeline failed when it encountered unexpected columns. **T:** Automate graceful schema evolution without manual intervention. **A:**
    
    
    # Enable schema merging in Delta writes
    df_new.write \
        .format("delta") \
        .mode("append") \
        .option("mergeSchema", "true") \    # auto-add new columns
        .save(DELTA_PATH)
    
    # For breaking changes (type changes), use schema evolution with validation
    from delta.tables import DeltaTable
    DeltaTable.forPath(spark, DELTA_PATH).upgradeTableProtocol(1, 2)
    
    
    
    

Additionally, set up a schema change notification: any new column logged and alerted to the team for documentation. **R:** Pipeline failures due to schema changes eliminated. New columns were automatically incorporated and flagged for documentation. Zero manual intervention needed for additive schema changes.

* * *

**Q99. How did you automate the coordination of parallel data migration streams?**

**S:** At AXA, migrating 20M+ records required 5 parallel migration streams (one per source system) to complete within the 2-week window. Coordinating these manually — knowing when each was done, handling failures, maintaining the overall progress view — was complex. **T:** Automate coordination of 5 parallel migration streams with central status tracking. **A:** Built a migration orchestrator:
    
    
    from concurrent.futures import ProcessPoolExecutor, as_completed
    import json
    
    MIGRATION_SOURCES = ["claims", "policies", "handlers", "products", "adjustments"]
    
    def migrate_source(source_name):
        """Run migration for a single source — returns status dict."""
        try:
            rows = run_batch_migration(source_name)
            return {"source": source_name, "status": "SUCCESS", "rows": rows}
        except Exception as e:
            return {"source": source_name, "status": "FAILED", "error": str(e)}
    
    with ProcessPoolExecutor(max_workers=5) as executor:
        futures = {executor.submit(migrate_source, src): src for src in MIGRATION_SOURCES}
        for future in as_completed(futures):
            result = future.result()
            log_migration_status(result)
            print(f"[{result['status']}] {result['source']}: {result.get('rows', 'N/A')} rows")
    
    
    
    

**R:** All 5 migration streams ran in parallel and completed in 4 hours vs the estimated 20 hours sequential. Central status table gave real-time progress visibility. Failed streams were automatically retried. Migration completed 2 days ahead of the deadline.

* * *

**Q100. What is one automation you wish you had built sooner — and why?**

**Situation:** Reflecting across 11+ years of data engineering and analytics.

**Task:** Identify the highest-regret automation gap and what I learned from it.

**Action/Reflection:**

The automation I wish I had built sooner at every engagement is a **data contract validation system** (Q79).

In almost every project, the most painful incidents weren't caused by bugs in my code — they were caused by upstream source systems changing their outputs without warning:

  * A date format that switched from `DD/MM/YYYY` to `MM/DD/YYYY`
  * A field that changed from nullable to NOT NULL
  * A new status code that wasn't in the lookup table



These changes propagated silently through pipelines for days before anyone noticed. The recovery was always more expensive than the prevention would have been.

A data contract — a machine-readable schema and business rule agreement between the data producer and consumer, validated automatically on every delivery — would have caught every single one of these in minutes.

I now build data contract validation as a **first deliverable** on every new engagement, before any transformation logic. The contract validation run is the first step in every pipeline.

**Result:** At J&J (where I introduced it mid-project), contract violations reduced from 6/month to 1/month within a quarter. At AXA (where I built it from day one), there were zero silent schema-change incidents across 18 months. The lesson: automation that catches errors at the boundary between systems is worth more than automation that cleans up errors after they've propagated.

* * *

_Best of luck in your interviews,_ hhhh _. Your automation track record — 100+ hours/month eliminated, $2M+ in savings identified, 85% reconciliation issue reduction — is exactly the kind of evidence these questions are designed to elicit._
