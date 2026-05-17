---
title: "PySpark Quality Monitoring"
date: 2026-04-12 16:36:49 
author: shajeebhameed
layout: page
slug: pyspark-quality-monitoring
status: publish
---

This is a solid topic to know deeply given your AXA and Travelers experience — here's a practical, end-to-end guide.

## What a data quality monitoring framework does in PySpark

It runs a defined set of checks against a dataset, scores each check as pass/fail, logs the results, and optionally alerts or halts the pipeline on critical failures. The framework sits inside your ETL pipeline, typically after ingestion and after transformation.

* * *

## Core building blocks

### 1\. Load your data
    
    
    from pyspark.sql import SparkSession
    from pyspark.sql import functions as F
    
    spark = SparkSession.builder.appName("DQ_Framework").getOrCreate()
    
    df = spark.read.format("delta").load("/mnt/bronze/claims")
    
    

* * *

### 2\. Define a check runner

The pattern is: each check is a function that returns a result dict. You collect all results into a log table.
    
    
    from datetime import datetime
    
    def run_check(check_name, severity, passed, actual_value, threshold=None):
        return {
            "check_name":   check_name,
            "severity":     severity,          # critical / major / minor
            "passed":       passed,
            "actual_value": str(actual_value),
            "threshold":    str(threshold),
            "run_timestamp": datetime.utcnow().isoformat(),
            "table_name":   "claims"
        }
    
    

* * *

### 3\. Completeness checks (null rates)
    
    
    total = df.count()
    
    def null_rate_check(col_name, threshold, severity):
        null_count = df.filter(F.col(col_name).isNull()).count()
        rate = null_count / total
        return run_check(
            check_name=f"null_rate_{col_name}",
            severity=severity,
            passed=rate <= threshold,
            actual_value=round(rate, 4),
            threshold=threshold
        )
    
    results = []
    results.append(null_rate_check("policy_id",     threshold=0.0,  severity="critical"))
    results.append(null_rate_check("claim_date",    threshold=0.0,  severity="critical"))
    results.append(null_rate_check("cause_of_loss", threshold=0.05, severity="major"))
    results.append(null_rate_check("adjuster_id",   threshold=0.10, severity="minor"))
    
    

* * *

### 4\. Uniqueness / duplicate checks
    
    
    def duplicate_check(col_name, severity):
        total_rows = df.count()
        distinct_rows = df.select(col_name).distinct().count()
        has_dupes = distinct_rows < total_rows
        return run_check(
            check_name=f"duplicate_{col_name}",
            severity=severity,
            passed=not has_dupes,
            actual_value=total_rows - distinct_rows
        )
    
    results.append(duplicate_check("claim_id", severity="critical"))
    
    

* * *

### 5\. Referential integrity checks
    
    
    policies_df = spark.read.format("delta").load("/mnt/silver/policies")
    
    def referential_integrity_check(left_df, right_df, left_key, right_key, severity):
        orphans = (
            left_df.select(left_key)
            .join(right_df.select(right_key), left_df[left_key] == right_df[right_key], "left_anti")
            .count()
        )
        return run_check(
            check_name=f"ref_integrity_{left_key}",
            severity=severity,
            passed=orphans == 0,
            actual_value=orphans
        )
    
    results.append(referential_integrity_check(df, policies_df, "policy_id", "policy_id", "critical"))
    
    

* * *

### 6\. Business rule / temporal checks
    
    
    def temporal_check(check_name, condition_expr, severity):
        violations = df.filter(condition_expr).count()
        return run_check(
            check_name=check_name,
            severity=severity,
            passed=violations == 0,
            actual_value=violations
        )
    
    # Claim date cannot precede policy inception
    results.append(temporal_check(
        "claim_before_inception",
        F.col("claim_date") < F.col("policy_inception_date"),
        severity="critical"
    ))
    
    # Reserve must be positive for open claims
    results.append(temporal_check(
        "negative_reserve_open_claim",
        (F.col("claim_status") == "OPEN") & (F.col("reserve_amount") <= 0),
        severity="critical"
    ))
    
    # Settlement date after claim date
    results.append(temporal_check(
        "settlement_before_claim",
        F.col("settlement_date") < F.col("claim_date"),
        severity="major"
    ))
    
    

* * *

### 7\. Statistical / range checks
    
    
    def range_check(col_name, min_val, max_val, severity):
        violations = df.filter(
            (F.col(col_name) < min_val) | (F.col(col_name) > max_val)
        ).count()
        return run_check(
            check_name=f"range_{col_name}",
            severity=severity,
            passed=violations == 0,
            actual_value=violations,
            threshold=f"{min_val} - {max_val}"
        )
    
    results.append(range_check("claim_amount", min_val=0, max_val=10_000_000, severity="major"))
    
    

* * *

### 8\. Write results to a quality log table
    
    
    from pyspark.sql.types import StructType, StructField, StringType, BooleanType
    
    schema = StructType([
        StructField("check_name",    StringType()),
        StructField("severity",      StringType()),
        StructField("passed",        BooleanType()),
        StructField("actual_value",  StringType()),
        StructField("threshold",     StringType()),
        StructField("run_timestamp", StringType()),
        StructField("table_name",    StringType()),
    ])
    
    results_df = spark.createDataFrame(results, schema)
    
    results_df.write.format("delta") \
        .mode("append") \
        .save("/mnt/silver/dq_results")
    
    

This log table is what feeds your Power BI monitoring dashboard — you query it for pass rates, trend over time, and open critical failures.

* * *

### 9\. Pipeline gate — halt on critical failures
    
    
    critical_failures = [r for r in results if r["severity"] == "critical" and not r["passed"]]
    
    if critical_failures:
        failed_checks = [r["check_name"] for r in critical_failures]
        raise Exception(f"Pipeline halted — critical DQ failures: {failed_checks}")
    
    

This is the pattern that prevents bad data from propagating into the Silver or Gold layer.

* * *

## How to structure it at scale

Rather than hardcoding every check, define them as config so non-engineers can add rules:
    
    
    checks_config = [
        {"type": "null_rate",  "column": "policy_id",   "threshold": 0.0,  "severity": "critical"},
        {"type": "null_rate",  "column": "claim_date",  "threshold": 0.0,  "severity": "critical"},
        {"type": "duplicate",  "column": "claim_id",    "severity": "critical"},
        {"type": "range",      "column": "claim_amount","min": 0, "max": 10_000_000, "severity": "major"},
    ]
    
    for check in checks_config:
        if check["type"] == "null_rate":
            results.append(null_rate_check(check["column"], check["threshold"], check["severity"]))
        elif check["type"] == "duplicate":
            results.append(duplicate_check(check["column"], check["severity"]))
        elif check["type"] == "range":
            results.append(range_check(check["column"], check["min"], check["max"], check["severity"]))
    
    

Store the config in a JSON or Delta table and load it at runtime — this is how you make the framework maintainable by data stewards without touching code.

* * *

## How this maps to your AXA experience

When you describe this in interview, the framing is: at AXA you built this using **Fabric notebooks (PySpark)** , the checks ran after Bronze ingestion and before Silver promotion, critical failures triggered an alert before the actuarial dashboards refreshed, and the results fed a Power BI monitoring view for the data steward team. The 40% reduction in manual review came from replacing ad hoc spot-checks with this automated, logged framework.

That's a complete, production-grade answer to the question.
