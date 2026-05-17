---
title: "Snowflake - DA/DM"
date: 2026-03-23 10:36:00 
author: shajeebhameed
layout: page
slug: snowflake-da-dm
status: publish
---

# Sr. Data Architect / Data Modeller

Based on the role requirements (Snowflake, dbt, data modeling, end-to-end architecture, AWS, governance), here are all the possible interview questions with answers tailored to your experience.

* * *

## 📋 Question Categories

Category| Focus  
---|---  
**Snowflake Architecture**|  Core platform knowledge  
**dbt (Data Build Tool)**|  Transformation workflows  
**Data Modeling**|  Kimball, Inmon, Data Vault  
**End-to-End Architecture**|  Design scenarios  
**AWS Analytics Stack**|  Cloud platform knowledge  
**Data Governance**|  Quality, lineage, compliance  
**Behavioral & Leadership**| Consulting, stakeholder management  
**Scenario-Based**|  Problem-solving  
  
* * *

# CATEGORY 1: SNOWFLAKE ARCHITECTURE

## Q1. Walk me through the Bronze-Silver-Gold architecture you implemented in Snowflake at Travelers.

**What they're assessing:** Your hands-on Snowflake experience and understanding of medallion architecture.

**Sample Answer:**

> "At Travelers Insurance, I implemented a **Bronze-Silver-Gold medallion architecture** for claims data processing:
> 
> **Bronze Layer (Raw):**
> 
>   * Ingested raw Medicare/Medicaid claims data from on-premise sources using **Snowpipe** for streaming and **COPY INTO** for batch
>   * Stored data exactly as received with full audit trail—no transformations applied
>   * Partitioned by ingestion date for efficient pruning
> 

> 
> **Silver Layer (Cleansed & Integrated):**
> 
>   * Created **Snowflake tasks** scheduled every 15 minutes for streaming, nightly for batch
>   * Applied data quality rules using SQL and JavaScript stored procedures
>   * Implemented **SCD Type II** using **streams and tasks** to track changes to dimensions (providers, policyholders)
>   * Joined claims data with reference tables to create an enterprise-wide integrated view following **Inmon-style normalization**
> 

> 
> **Gold Layer (Business-Ready):**
> 
>   * Built **dimensional models (star schemas)** for specific business processes:
>   * FactClaims (transactional claims)
>   * DimPolicyholder (with SCD II)
>   * DimProvider, DimProcedure, DimDate
>   * Used **clustering keys** on claim_date and policyholder_id for query performance
>   * Created **secure views** for different user roles (actuaries vs. claims adjusters)
> 

> 
> **Key outcomes:** Sub-second query performance for complex actuarial queries, reduced data latency from 24 hours to under 15 minutes for fraud detection, and established a foundation for predictive analytics."

* * *

## Q2. How do you optimize Snowflake query performance? Give specific examples.

**What they're assessing:** Your practical performance tuning knowledge.

**Sample Answer:**

> "I take a multi-layered approach to Snowflake performance optimization:
> 
> **1\. Warehouse Sizing & Configuration:**
> 
>   * At Travelers, we deployed **workload-specific virtual warehouses** :
>   * X-small for ELT processes (cost-effective, batch processing)
>   * Large for complex actuarial queries (sub-second performance)
>   * Used **auto-suspend** (1-5 minutes depending on workload) to control costs
>   * Enabled **multi-cluster warehouses** for high-concurrency workloads
> 

> 
> **2\. Table Design:**
> 
>   * **Clustering keys:** For the FactClaims table (billions of rows), we clustered on `claim_date` and `policyholder_id`—this reduced partition scans by 80%
>   * **Search Optimization Service (SOS):** Enabled for tables with frequent selective point lookups
>   * **Materialized views:** Created for frequently aggregated metrics to avoid recomputation
> 

> 
> **3\. Query Optimization:**
> 
>   * Used **query profiling** to identify bottlenecks—looked for large table scans, inefficient joins, spilling to local storage
>   * Replaced inefficient correlated subqueries with **JOINs** or **window functions**
>   * Leveraged **result caching** —repeated dashboard queries returned in milliseconds
> 

> 
> **4\. Data Modeling:**
> 
>   * Designed **star schemas** in Gold layer—denormalized for query performance
>   * Used **separate tables** for large dimensions to keep fact table lean
> 

> 
> **5\. Example:** One actuarial query was taking 8 minutes. After analyzing the query profile, I found it was scanning all partitions instead of pruning by date. I added a **date filter** and created a **clustering key** on claim_date—query time dropped to 3 seconds."

* * *

## Q3. Explain Change Data Capture (CDC) in Snowflake. How did you implement it at Travelers?

**What they're assessing:** Your understanding of real-time data ingestion patterns.

**Sample Answer:**

> "CDC in Snowflake enables real-time or near-real-time data ingestion from source systems. At Travelers, we implemented CDC to reduce data latency from 24 hours to under 15 minutes:
> 
> **Implementation Approach:**
> 
> **1\. Source Integration:**
> 
>   * Connected to on-premise claims databases using **Kafka** and **Snowpipe**
>   * Used **Debezium** for change data capture from source transaction logs
>   * Streamed changes to Snowflake staging tables
> 

> 
> **2\. Snowflake Components:**
> 
>   * **Streams:** Created streams on staging tables to track INSERT/UPDATE/DELETE operations
>   * **Tasks:** Scheduled tasks every 15 minutes to process streamed changes
>   * **Stored Procedures:** JavaScript procedures to apply changes with proper SCD handling
> 

> 
> **3\. Processing Logic:**
>     
>     
>     -- Stream captures changes
>     CREATE STREAM claims_stream ON TABLE staging.raw_claims;
>     
>     -- Task processes stream
>     CREATE TASK process_claims_stream
>       WAREHOUSE = compute_wh
>       SCHEDULE = '15 MINUTE'
>     AS
>       CALL apply_claim_changes();
> 
> **4\. SCD Type II Implementation:**
> 
>   * For UPDATE: Close current record (end_date = CURRENT_TIMESTAMP) and insert new record
>   * For INSERT: Add new record with effective_date = CURRENT_TIMESTAMP
>   * For DELETE: Mark as is_current = FALSE rather than physical delete
> 

> 
> **Challenges & Solutions:**
> 
>   * **Late-arriving data:** Implemented merge logic that checks for existing records before inserting
>   * **Performance:** Partitioned streams by source system and claim_date
>   * **Data consistency:** Used Snowflake's transaction support—if dimension update fails, fact processing rolls back
> 

> 
> **Outcome:** Fraud detection team got near-real-time visibility into claims patterns."

* * *

## Q4. What are Snowflake's key features for data governance and security?

**What they're assessing:** Your understanding of Snowflake's governance capabilities.

**Sample Answer:**

> "At Travelers, we implemented comprehensive governance using Snowflake's native features:
> 
> **1\. Access Control:**
> 
>   * **RBAC (Role-Based Access Control):** Created hierarchical roles—`DATA_ENGINEER`, `ACTUARY`, `CLAIMS_ADJUSTER`, `ANALYST`
>   * **Column-level security:** Used **dynamic data masking** for sensitive fields like SSN and medical codes
>   * **Row-level security:** Implemented **row access policies** —actuaries could see all claims, adjusters only their assigned cases
> 

> 
> **2\. Data Protection:**
> 
>   * **Time Travel:** Retained historical data for 7 days for accidental deletion recovery—saved us multiple times
>   * **Fail-safe:** Used for long-term recovery (7 days after Time Travel)
>   * **Transparent data encryption:** Enabled by default for data at rest and in transit
> 

> 
> **3\. Data Sharing & Collaboration:**
> 
>   * **Secure Data Sharing:** Shared aggregated claims data with external partners without copying data
>   * **Data Exchange:** Used for internal sharing between teams
> 

> 
> **4\. Audit & Compliance:**
> 
>   * **Query history:** Monitored all access patterns via `ACCOUNT_USAGE` views
>   * **Access history:** Tracked which roles accessed which data
>   * **Compliance:** Ensured HIPAA and Medicare/Medicaid requirements were met
> 

> 
> **5\. Data Quality:**
> 
>   * Implemented **not-null constraints** and **unique constraints** where applicable
>   * Used **stored procedures** for automated data quality checks
> 

> 
> **Example:** When an auditor requested access logs for a specific dataset, we queried `ACCOUNT_USAGE.ACCESS_HISTORY` to provide a complete audit trail."

* * *

# CATEGORY 2: DBT (DATA BUILD TOOL)

## Q5. What is dbt and how does it fit into a modern data stack?

**What they're assessing:** Your understanding of dbt's role in transformation.

**Sample Answer:**

> "**dbt (Data Build Tool)** is a transformation workflow tool that enables data analysts and engineers to write modular SQL transformations with software engineering best practices. It fits into the **T (Transform)** layer of ELT architecture.
> 
> **Key Concepts:**
> 
>   * **Models:** SQL SELECT statements that define transformations
>   * **Materializations:** How models are built (table, view, incremental, ephemeral)
>   * **Sources:** References to raw data with freshness checks
>   * **Tests:** Data quality assertions (unique, not null, relationships, custom)
>   * **Documentation:** Auto-generated data catalog from markdown files
>   * **Macros:** Reusable SQL logic (DRY principle)
>   * **Lineage:** Automatically generated DAG showing model dependencies
> 

> 
> **How It Fits:**
> 
> Layer| Tool  
> ---|---  
> Extract| Fivetran, Airbyte, Stitch  
> Load| Snowflake, BigQuery, Redshift  
> **Transform**| **dbt (models, tests, docs)**  
> Serve| Looker, Power BI, Tableau  
>   
> **Why dbt Matters:**
> 
>   * Brings **version control** (Git) to SQL transformations
>   * Enables **code reusability** through macros and Jinja
>   * Provides **testing** as a first-class citizen
>   * Creates **documentation** automatically
> 

> 
> **My Experience:** While my transformation experience has been in Snowflake SQL, Snowpark, and PySpark, I've studied dbt fundamentals and understand its value. Given my strong SQL background, I'm confident I can ramp up quickly."

* * *

## Q6. How would you structure dbt models for a Bronze-Silver-Gold architecture?

**What they're assessing:** Your understanding of how dbt maps to medallion architecture.

**Sample Answer:**

> "Here's how I would structure dbt models for a Bronze-Silver-Gold architecture:
> 
> **Folder Structure:**
>     
>     
>     models/
>     ├── bronze/
>     │   ├── src_claims.yml (source definitions)
>     │   ├── stg_claims.sql (raw ingestion)
>     │   └── ...
>     ├── silver/
>     │   ├── int_claims_cleaned.sql (intermediate transformations)
>     │   ├── int_claims_enriched.sql
>     │   ├── dim_provider.sql (dimension table)
>     │   ├── dim_policyholder.sql
>     │   └── ...
>     ├── gold/
>     │   ├── fct_claims.sql (fact table)
>     │   ├── fct_daily_claims.sql (aggregated fact)
>     │   └── ...
>     └── marts/
>         └── claims/
>             ├── fct_claims_by_region.sql
>             └── ...
> 
> **Materialization Strategy:**
> 
> Layer| Materialization| Reason  
> ---|---|---  
> Bronze| View or Ephemeral| Lightweight, avoid duplication  
> Silver| Table (incremental)| Large volumes, processed incrementally  
> Gold| Table (incremental)| Large volumes, frequent updates  
> Marts| Table (full refresh)| Smaller, business-facing aggregates  
>   
> **Incremental Logic Example:**
>     
>     
>     -- Silver layer incremental model
>     {{
>       config(
>         materialized='incremental',
>         unique_key='claim_id',
>         incremental_strategy='merge'
>       )
>     }}
>     
>     SELECT *
>     FROM {{ ref('stg_claims') }}
>     WHERE claim_date >= '2020-01-01'
>     
>     {% if is_incremental() %}
>       AND claim_date > (SELECT MAX(claim_date) FROM {{ this }})
>     {% endif %}
> 
> **Testing:**
>     
>     
>     # silver/schema.yml
>     models:
>       - name: dim_provider
>         columns:
>           - name: provider_id
>             tests:
>               - unique
>               - not_null
>           - name: tax_id
>             tests:
>               - not_null
>       - name: fct_claims
>         columns:
>           - name: claim_amount
>             tests:
>               - not_null
>               - accepted_values:
>                   values: [0, 1000000]

* * *

## Q7. What are dbt tests and how do you use them for data quality?

**What they're assessing:** Your understanding of data quality in dbt.

**Sample Answer:**

> "dbt tests are assertions about your data that run as part of your pipeline. They're critical for maintaining data quality.
> 
> **Types of Tests:**
> 
> **1\. Generic Tests (Built-in):**
>     
>     
>     models:
>       - name: customers
>         columns:
>           - name: customer_id
>             tests:
>               - unique
>               - not_null
>           - name: status
>             tests:
>               - accepted_values:
>                   values: ['active', 'pending', 'closed']
>           - name: email
>             tests:
>               - not_null
> 
> **2\. Relationships (Referential Integrity):**
>     
>     
>     models:
>       - name: orders
>         columns:
>           - name: customer_id
>             tests:
>               - relationships:
>                   to: ref('customers')
>                   field: customer_id
> 
> **3\. Custom Tests (Singular):**
>     
>     
>     -- tests/assert_positive_claim_amount.sql
>     SELECT claim_id, claim_amount
>     FROM {{ ref('fct_claims') }}
>     WHERE claim_amount < 0
> 
> **4\. Custom Generic Tests:**
>     
>     
>     -- macros/test_no_negative_amounts.sql
>     {% test no_negative_amounts(model, column_name) %}
>         SELECT *
>         FROM {{ model }}
>         WHERE {{ column_name }} < 0
>     {% endtest %}
> 
> **My Experience:**  
>  At Refinitiv, I built a **validation framework with 50+ automated checks** achieving 99.9% data consistency. In dbt, I would implement similar checks as tests, ensuring:
> 
>   * No nulls in primary keys
>   * Foreign keys exist in dimension tables
>   * Amounts within acceptable ranges
>   * No future dates in timestamp columns
> 

> 
> **Best Practices:**
> 
>   * **Test early:** Run tests after silver layer before gold
>   * **Fail fast:** Stop pipeline if critical tests fail
>   * **Document:** Test descriptions help new team members
>   * **Store results:** Track test outcomes over time"
> 


* * *

# CATEGORY 3: DATA MODELING (KIMBALL, INMON, DATA VAULT)

## Q8. Compare Kimball and Inmon methodologies. When would you use each?

**What they're assessing:** Your understanding of data warehousing approaches.

**Sample Answer:**

> "Kimball and Inmon represent two different philosophies in data warehousing:
> 
> **Inmon (Bill Inmon - Corporate Information Factory):**
> 
>   * **Top-down approach:** Build a centralized, normalized (3NF) enterprise data warehouse first
>   * **Process:** Source → Enterprise Data Warehouse (3NF) → Data Marts
>   * **Strengths:** Single source of truth, strong data governance, ideal for enterprise-wide reporting
>   * **Weaknesses:** Slower time-to-value, more complex, requires more upfront design
>   * **Best for:** Large enterprises with strong data governance needs, regulatory reporting
> 

> 
> **Kimball (Ralph Kimball - Dimensional Modeling):**
> 
>   * **Bottom-up approach:** Build dimensional data marts (star schemas) directly from sources
>   * **Process:** Source → Data Marts (star schemas) → Integrated via conformed dimensions
>   * **Strengths:** Faster implementation, business-user friendly, easier to understand
>   * **Weaknesses:** Potential data redundancy, integration challenges across marts
>   * **Best for:** Departments needing quick insights, business intelligence, ad-hoc analysis
> 

> 
> **My Hybrid Approach (at Travelers):**
> 
>   * **Bronze/Silver:** Followed Inmon philosophy—normalized (3NF) enterprise layer for data integrity
>   * **Gold:** Followed Kimball philosophy—dimensional models for business consumption
> 

> 
> **When to Use Each:**
> 
> Scenario| Choose  
> ---|---  
> Enterprise-wide reporting, strong governance| Inmon  
> Quick time-to-value, departmental BI| Kimball  
> Both (most real-world)| **Hybrid**  
>   
> **Example:** At Travelers, we needed both: actuarial teams needed complex, integrated data (Inmon), while business analysts needed simple, fast queries on claims (Kimball). Hybrid was the answer."

* * *

## Q9. Explain Slowly Changing Dimensions (SCD) Types. Give examples of each.

**What they're assessing:** Your dimensional modeling knowledge.

**Sample Answer:**

> "SCD handles changes to dimension attributes over time. I've implemented Types I and II extensively.
> 
> **SCD Type I (Overwrite):**
> 
>   * Old value is replaced with new value
>   * No history kept
>   * **Example:** Customer contact phone number—only need current number
>   * **Implementation:**
> 

>     
>     
>     UPDATE dim_customer
>     SET phone = '555-1234'
>     WHERE customer_id = 123;
> 
> **SCD Type II (Add New Row):**
> 
>   * New row added with new values, plus effective dates and current flag
>   * Full history preserved
>   * **Example:** Customer address change—need to know which address was valid for past orders
>   * **Implementation:**
> 

>     
>     
>     -- Close current record
>     UPDATE dim_customer
>     SET end_date = '2024-01-01', is_current = FALSE
>     WHERE customer_id = 123 AND is_current = TRUE;
>     
>     -- Insert new record
>     INSERT INTO dim_customer (customer_id, address, effective_date, end_date, is_current)
>     VALUES (123, 'New Address', '2024-01-01', NULL, TRUE);
> 
> **SCD Type III (Add New Column):**
> 
>   * Add column to store previous value
>   * Limited history (only one change)
>   * **Example:** Previous loyalty tier
>   * **Implementation:**
> 

>     
>     
>     ALTER TABLE dim_customer ADD COLUMN previous_tier VARCHAR;
>     
>     UPDATE dim_customer
>     SET previous_tier = current_tier, current_tier = 'Gold'
>     WHERE customer_id = 123;
> 
> **My Experience:**
> 
>   * At **Travelers** , we used **SCD Type II** for policyholder address history and claims adjuster assignments—critical for audit and historical analysis
>   * At **JNJ** , we used **SCD Type I** for employee contact details where only current information mattered
> 

> 
> **When to Use Which:**
> 
>   * **Type I:** Correcting errors, non-critical attributes, operational data
>   * **Type II:** Regulatory requirements, historical analysis, audit trails
>   * **Type III:** Limited history needs, space-constrained environments"
> 


* * *

## Q10. What is Data Vault modeling? Have you used it?

**What they're assessing:** Your awareness of alternative modeling approaches.

**Sample Answer:**

> "**Data Vault** is a hybrid data modeling approach designed for enterprise data warehousing. It consists of three components:
> 
> **1\. Hubs:** Unique business keys (e.g., customer_id, product_id)
> 
>   * Contains only business keys, no descriptive data
> 

> 
> **2\. Links:** Relationships between hubs (e.g., customer purchase product)
> 
>   * Captures associations
> 

> 
> **3\. Satellites:** Descriptive attributes of hubs or links with historical tracking
> 
>   * Contains context and metadata
> 

> 
> **Benefits:**
> 
>   * **Scalability:** Parallel loading
>   * **Auditability:** Complete history preserved
>   * **Agility:** Easy to add new data sources
> 

> 
> **Challenges:**
> 
>   * More complex querying than star schemas
>   * Requires more joins for business reporting
>   * Higher learning curve
> 

> 
> **My Experience:**  
>  I have **foundational understanding** of Data Vault methodology through study and research. In practice, my primary experience is with **Kimball and Inmon**. For example, at Travelers, we used:
> 
>   * **Inmon-style** normalized Silver layer for enterprise integration
>   * **Kimball-style** star schemas for Gold layer business reporting
> 

> 
> **When Data Vault Makes Sense:**
> 
>   * Large enterprises with multiple source systems
>   * Need for complete auditability
>   * Frequent source system changes
> 

> 
> **My Adaptability:** Given my strong dimensional modeling foundation and understanding of Data Vault concepts, I'm confident I can quickly apply Data Vault methodology if required."

* * *

# CATEGORY 4: END-TO-END ARCHITECTURE

## Q11. Design an end-to-end data architecture for a retail company migrating from on-premise to cloud.

**What they're assessing:** Your ability to design complete solutions.

**Sample Answer:**

> "Based on my Affinion migration experience (20M+ records to Azure), here's my approach:
> 
> **1\. Assessment Phase (4-6 weeks):**
> 
>   * Profile source data—identify orphan records, data quality issues (as we did at Affinion)
>   * Map dependencies: ETL jobs, reports, applications
>   * Understand business requirements: SLAs, latency needs, new capabilities
> 

> 
> **2\. Target Architecture Design:**
>     
>     
>     ┌─────────────────────────────────────────────────────────────────┐
>     │                     Source Systems                              │
>     │  Oracle (Legacy ERP) │ SQL Server (POS) │ Salesforce (CRM)     │
>     └──────────────────────┴──────────────────┴──────────────────────┘
>                                  │
>                                  ▼
>     ┌─────────────────────────────────────────────────────────────────┐
>     │                     Ingestion Layer                             │
>     │  Azure Data Factory (Batch) │ Event Hubs (Streaming)           │
>     └─────────────────────────────────────────────────────────────────┘
>                                  │
>                                  ▼
>     ┌─────────────────────────────────────────────────────────────────┐
>     │                     Bronze Layer (Raw)                          │
>     │  Azure Data Lake Storage (Parquet)                             │
>     │  Partition: source_system/year/month/day                       │
>     └─────────────────────────────────────────────────────────────────┘
>                                  │
>                                  ▼
>     ┌─────────────────────────────────────────────────────────────────┐
>     │                     Silver Layer (Cleansed)                     │
>     │  PySpark/Databricks or dbt                                      │
>     │  - Deduplication, validation                                    │
>     │  - SCD Type II implementation                                   │
>     │  - 3NF for enterprise integration                               │
>     └─────────────────────────────────────────────────────────────────┘
>                                  │
>                                  ▼
>     ┌─────────────────────────────────────────────────────────────────┐
>     │                     Gold Layer (Business)                       │
>     │  Azure Synapse or Snowflake                                     │
>     │  - Dimensional models (star schemas)                           │
>     │  - Aggregated tables for performance                           │
>     └─────────────────────────────────────────────────────────────────┘
>                                  │
>                  ┌───────────────┼───────────────┐
>                  ▼               ▼               ▼
>             ┌─────────┐   ┌─────────┐   ┌─────────┐
>             │ Power BI│   │ Tableau │   │ Data API│
>             │Reports  │   │Dashboards│   │ (Apps)  │
>             └─────────┘   └─────────┘   └─────────┘
> 
> **3\. Migration Strategy:**
> 
>   * **Phased approach:** Customer → Product → Transactions → Sales
>   * **Parallel runs:** Run both old and new systems during validation
>   * **Reconciliation framework:** Automated scripts comparing source and target
> 

> 
> **4\. Governance & Security:**
> 
>   * **RBAC:** Data engineers, analysts, business users
>   * **Dynamic masking:** PII protection
>   * **Purview:** Data lineage, catalog, classification
> 

> 
> **5\. CI/CD:**
> 
>   * Terraform for infrastructure
>   * Azure DevOps for pipeline deployments
>   * Git for version control
> 

> 
> **Key Success Factors:**
> 
>   * Profile source data before migration (lesson from Affinion)
>   * Automate validation (50+ checks at Refinitiv)
>   * Involve business stakeholders early"
> 


* * *

## Q12. How do you handle schema evolution in a data pipeline?

**What they're assessing:** Your ability to handle changing data structures.

**Sample Answer:**

> "Schema evolution is inevitable. Here's my approach:
> 
> **1\. Prevention (Design):**
> 
>   * Store raw data in **flexible formats** (Parquet, JSON) that accommodate schema changes
>   * At **Travelers** , Bronze layer stored raw claims exactly as received—schema changes didn't break ingestion
> 

> 
> **2\. Detection (Monitoring):**
> 
>   * Automated **schema validation** at the start of each pipeline run
>   * Alert if unexpected fields appear or required fields are missing
>   * At **Refinitiv** , we had 50+ automated checks that would flag schema drift
> 

> 
> **3\. Handling (Strategy):**
> 
> Change Type| Strategy  
> ---|---  
> New optional column| Auto-add to schema, process with default null  
> New required column| Manual review, update downstream models  
> Column renamed| Map old to new, document change  
> Column removed| Mark as deprecated, phase out usage  
> Data type change| Cast/conversion logic, validation  
>   
> **4\. Implementation Example (PySpark):**
>     
>     
>     # Read with schema evolution
>     df = spark.read.option("mergeSchema", "true").parquet("bronze/")
>     
>     # Or use schema validation
>     expected_schema = StructType([...])
>     if df.schema != expected_schema:
>         logging.warning(f"Schema drift detected: {df.schema}")
>         # Handle based on drift type
> 
> **5\. dbt Approach:**
> 
>   * Use **`incremental_strategy='append'`** to add new rows
>   * **`--full-refresh`** when schema changes require rebuild
>   * Document schema changes in dbt docs
> 

> 
> **6\. Communication:**
> 
>   * Notify downstream consumers (analysts, data scientists) of changes
>   * Maintain **data contract** between producers and consumers
> 

> 
> **My Experience:**  
>  At **JNJ** , when consolidating 4 legacy systems, each had different schemas. We created a **unified STTM mapping** that handled the evolution from source to target, documenting 200+ attributes with transformation rules."

* * *

# CATEGORY 5: AWS ANALYTICS STACK

## Q13. Your experience is primarily Azure. How would you adapt to AWS?

**What they're assessing:** Your ability to transfer cloud knowledge to AWS.

**Sample Answer:**

> "While my primary cloud experience is Azure and GCP, I understand that cloud architecture principles are transferable. Here's how I map my experience to AWS:
> 
> **Service Mapping:**
> 
> My Experience (Azure/GCP)| AWS Equivalent  
> ---|---  
> Azure Data Lake Storage| Amazon S3  
> Azure Data Factory| AWS Glue / Data Pipeline  
> Azure Synapse Analytics| Amazon Redshift  
> Azure Databricks| AWS EMR / Glue Studio  
> Azure Functions| AWS Lambda  
> Power BI| Amazon QuickSight  
> Microsoft Purview| AWS Glue Data Catalog + Lake Formation  
> BigQuery| Amazon Redshift Spectrum  
> Snowflake| Snowflake on AWS (cross-cloud)  
>   
> **How I Would Approach AWS:**
> 
> **1\. Leverage Core Principles:**
> 
>   * **Separation of storage and compute:** S3 (storage) + EMR/Athena (compute)
>   * **Medallion architecture:** Bronze/Silver/Gold in S3 with Parquet/Delta
>   * **Infrastructure as code:** Terraform supports AWS equally
> 

> 
> **2\. Start with Foundational Services:**
> 
>   * **S3:** Data lake storage with partitioning, lifecycle policies
>   * **Glue:** ETL/catalog similar to Data Factory
>   * **Redshift:** Data warehouse similar to Synapse
>   * **EMR:** Managed Spark similar to Databricks
> 

> 
> **3\. AWS-Specific Features I'd Learn:**
> 
>   * **Athena:** Serverless query on S3
>   * **Kinesis:** Real-time streaming (like Event Hubs)
>   * **Lake Formation:** Governance and security
> 

> 
> **My Learning Plan:**
> 
>   * I've already started AWS hands-on practice
>   * I'll complete AWS Data Analytics Specialty certification
>   * I can transfer my Azure patterns to AWS with minimal learning curve
> 

> 
> **Key Point:** Good architecture is cloud-agnostic. My experience in medallion architecture, data modeling, and governance applies regardless of platform."

* * *

## Q14. How would you use AWS services to build a real-time analytics pipeline?

**What they're assessing:** Your AWS architecture design skills.

**Sample Answer:**

> "Here's how I'd design a real-time analytics pipeline using AWS services:
> 
> **Architecture:**
>     
>     
>     Source (Transactional DB) → DMS → Kinesis Data Streams
>                                        ↓
>                            Kinesis Data Analytics (Flink)
>                                        ↓
>                                        ├→ S3 (Raw Bronze)
>                                        ├→ DynamoDB (Real-time dashboard)
>                                        └→ Lambda (Alerts)
>                                        ↓
>                                  Glue (ETL to Silver/Gold)
>                                        ↓
>                                  Redshift / S3
>                                        ↓
>                                  QuickSight / Power BI
> 
> **Component Breakdown:**
> 
> **1\. Data Ingestion:**
> 
>   * **AWS DMS (Database Migration Service):** Change data capture from source databases
>   * **Kinesis Data Streams:** High-throughput streaming data ingestion
> 

> 
> **2\. Real-time Processing:**
> 
>   * **Kinesis Data Analytics:** Apache Flink for stream processing—windowed aggregations, anomaly detection
>   * **Lambda:** Serverless functions for alerting (e.g., fraud detection)
> 

> 
> **3\. Storage:**
> 
>   * **S3:** Raw data (Bronze), partitioned by date
>   * **DynamoDB:** Low-latency storage for real-time dashboard data
> 

> 
> **4\. Batch Processing:**
> 
>   * **AWS Glue:** ETL jobs for Silver/Gold transformations
>   * **EMR:** For complex PySpark transformations
> 

> 
> **5\. Serving:**
> 
>   * **Redshift:** Data warehouse for historical analysis
>   * **Redshift Spectrum:** Query S3 data directly
>   * **QuickSight:** BI dashboards
> 

> 
> **Example Use Case (Fraud Detection):**
> 
>   1. Transactions stream into **Kinesis**
>   2. **Kinesis Analytics** runs sliding window to detect suspicious patterns
>   3. **Lambda** triggers alert if fraud detected
>   4. Raw data lands in **S3** (Bronze)
>   5. **Glue** transforms to Silver (cleansed) and Gold (aggregated)
>   6. **Redshift** serves historical analysis
>   7. **QuickSight** shows fraud trends dashboard
> 

> 
> **My Approach:**  
>  While my hands-on experience is Azure, this architecture is cloud-agnostic. I've built similar pipelines using Azure Event Hubs, Stream Analytics, and Data Lake."

* * *

# CATEGORY 6: DATA GOVERNANCE

## Q15. How do you implement data governance in a modern data platform?

**What they're assessing:** Your end-to-end governance knowledge.

**Sample Answer:**

> "I take a layered approach to data governance:
> 
> **Layer 1: People & Processes**
> 
>   * **Define roles:** Data Owners, Data Stewards, Data Consumers
>   * **Establish policies:** Data classification, retention, access
>   * **Create processes:** Data quality monitoring, issue resolution
> 

> 
> **Layer 2: Discovery & Catalog (Microsoft Purview / AWS Glue Data Catalog)**
> 
>   * Automated scanning of data sources
>   * Business glossary with standard definitions
>   * Searchable catalog for analysts
> 

> 
> **Layer 3: Classification & Protection**
> 
>   * **Sensitivity labels:** Public, Internal, Confidential, Highly Restricted
>   * **Automatic classification:** Detect PII, PHI, financial data
>   * **Access controls:** RBAC, column-level security, dynamic masking
> 

> 
> **Layer 4: Lineage & Audit**
> 
>   * **End-to-end lineage:** Track data from source to report
>   * **Impact analysis:** Understand downstream effects of changes
>   * **Audit trails:** Complete history of data access and changes
> 

> 
> **Layer 5: Quality & Monitoring**
> 
>   * **Automated data quality checks** (like the 50+ checks at Refinitiv)
>   * **SLA monitoring:** Freshness, volume, latency
>   * **Alerting** for anomalies
> 

> 
> **My Experience:**
> 
>   * **JNJ:** Developed STTM documentation for 200+ business attributes, ensuring GDPR compliance
>   * **AXA:** Integrated Microsoft Purview for lineage, classification, and sensitivity labels
>   * **Travelers:** Established governance frameworks with RBAC and dynamic data masking
>   * **Refinitiv:** Built validation frameworks with 50+ automated checks achieving 99.9% consistency
> 

> 
> **Key Principle:** Governance should enable, not block. The goal is to give users confidence in data while ensuring compliance."

* * *

# CATEGORY 7: BEHAVIORAL & LEADERSHIP

## Q16. Tell me about a time you had to influence a stakeholder who disagreed with your architecture decision.

**What they're assessing:** Your communication and influence skills.

**Sample Answer (using the Travelers story):**

> "**Situation:** At Travelers, the lead actuary was resistant to the new Snowflake architecture. He had been burned by failed data migrations before and didn't trust we could maintain data integrity.
> 
> **Task:** I needed to earn his trust and get his buy-in for an architecture I knew was essential for scalability.
> 
> **Action:**
> 
>   1. **Listened first:** Scheduled a meeting to understand his concerns. He had legitimate fears based on past experiences.
>   2. **Built a prototype:** Took one month of his most complex claims data through the Bronze-Silver-Gold pipeline and created a detailed reconciliation report.
>   3. **Demonstrated value:** Showed him 100% data fidelity, additional quality checks, and performance improvements (hours → seconds).
>   4. **Involved him:** Asked him to validate the transformed data against his existing reports.
> 

> 
> **Result:** He became the project's strongest advocate. He even discovered data quality issues in source data he hadn't known about.
> 
> **Lesson:** Showing is more powerful than telling. Trust is built through transparency and tangible results."

* * *

## Q17. Describe a challenging data migration project and how you ensured success.

**What they're assessing:** Your project management and problem-solving skills.

**Sample Answer (using the Affinion story):**

> "**Project:** Affinion Voyager – 20+ million customer records from legacy Oracle to Azure
> 
> **Challenges:**
> 
>   * Legacy data had orphan records, inconsistent formats
>   * Business couldn't afford downtime during cutover
>   * Multiple source systems with no single source of truth
> 

> 
> **My Approach:**
> 
> **1\. Discovery Phase (2 weeks):**
> 
>   * Profiled all source data, identified orphan records and quality issues
>   * Presented remediation plan before migration began
> 

> 
> **2\. Phased Strategy:**
> 
>   * Migrated by domain: customer → accounts → transactions
>   * Each phase had parallel run comparing source and target
> 

> 
> **3\. Automation:**
> 
>   * Built C# scripts for ETL (reduced manual errors by 40%)
>   * Created reconciliation framework with automated checks
> 

> 
> **4\. Validation:**
> 
>   * Count reconciliation (row counts match)
>   * Sum reconciliation (critical fields match)
>   * Sample record deep-dives
>   * Business rule validation
> 

> 
> **5\. Communication:**
> 
>   * Power BI dashboard showing migration progress
>   * Daily stand-ups with stakeholders
> 

> 
> **Result:** 100% data fidelity, zero business disruption, single source of truth for 150+ analysts."

* * *

# CATEGORY 8: SCENARIO-BASED

## Q18. A data pipeline fails at 2 AM. Walk me through your response.

**What they're assessing:** Your operational readiness and problem-solving.

**Sample Answer:**

> "I follow a structured incident response process:
> 
> **1\. Detection (Automated):**
> 
>   * Monitoring alerts via email/Slack/P
>
