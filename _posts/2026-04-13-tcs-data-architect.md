---
title: "TCS Data Architect"
date: 2026-04-13 16:36:50 
author: shajeebhameed
layout: page
slug: tcs-data-architect
status: publish
---

# Interview Preparation Guide

## Role: Data Architect — TCS (Dublin, Hybrid)

### Candidate: Shajeeb S

* * *

> **How to use this guide:** Each question includes _why they're asking it_ , a _structured answer_ , and _key emphasis points_. Practise delivering answers conversationally — don't memorise verbatim. Aim for 2–3 minutes per answer unless the question is clearly a short one.

* * *

## SECTION 1: Data Modelling Deep Dives

* * *

### Q1. Walk me through your approach to designing a dimensional data model from scratch. How do you decide between Inmon and Kimball?

**Why they're asking:** This is the central technical skill for this role. They want to know you understand both methodologies philosophically and can choose the right one based on context — not just name-drop them.

**Your Answer:**

"My approach always starts with understanding the business requirements and the analytical use cases — not the technology. I treat data modelling as a translation exercise between business intent and data structure.

When evaluating between Inmon and Kimball, I look at a few key dimensions:

**Inmon (3NF, enterprise data warehouse):** I favour this when the organisation needs a single authoritative version of truth across many subject areas, or when data is heavily reused across different analytical domains. At J&J Global Services, for example, I consolidated 4 legacy data sources into a normalised relational model in BigQuery — the foundation was Inmon-style to ensure referential integrity and governance across 150+ users globally.

**Kimball (dimensional, star schema):** I favour this when the priority is query performance, BI accessibility, and speed of analytical delivery. At AXA Insurance, I applied Kimball dimensional modelling for claims, policy, and underwriting domains — with clearly defined Fact tables and Dimension tables using SCD I and SCD II patterns. Actuaries and claims adjusters needed fast, intuitive access to historical trends, and a star schema delivered exactly that.

In practice, my preferred modern pattern is a hybrid: a normalised Data Vault or 3NF layer in the Bronze/Silver zones of a Medallion architecture, with dimensional star schemas surfaced in the Gold layer for consumption. This gives you the governance and auditability of Inmon with the query performance of Kimball.

The key is that the model must serve the business — I always validate my logical model against business user stories before moving to physical design."

**Key emphasis:** Be specific about AXA (Kimball/dimensional) and J&J (normalised/Inmon). Mention SCD patterns proactively. Show you can contextualise the choice rather than applying one methodology rigidly.

**Likely follow-up:** "Can you describe a specific SCD II implementation?"

* * *

### Q2. Describe your experience implementing SCD I, SCD II, and SCD III patterns. Give a concrete example.

**Why they're asking:** This is explicitly listed in the job spec — they will almost certainly ask it. They want to confirm you've actually implemented these, not just defined them.

**Your Answer:**

"SCD — Slowly Changing Dimensions — manage how historical data is preserved or overwritten when dimension values change. I've implemented all three types in production environments, but SCD I and SCD II are the workhorses.

**SCD I (overwrite):** Used when history is not important. At AXA, simple reference attributes like current contact addresses on the Customer dimension were managed as SCD I — we simply updated the record in place. Simple, low maintenance.

**SCD II (add new row):** The most common pattern for analytical use cases where history matters. I implemented SCD II for the Policy dimension at AXA — when a policyholder changed their coverage tier or premium band, we inserted a new row with a new surrogate key and updated the `EffectiveDate`, `ExpiryDate`, and `IsCurrent` flag on the prior record. This allowed actuaries to analyse claims against the policy terms that were active _at the time of the claim_ — not just current terms. I engineered these SCD II pipelines using Azure Dataflows Gen2 and Data Pipelines in Microsoft Fabric, with incremental loading patterns to avoid full reloads.

**SCD III (add new column):** Used sparingly — typically when you need only one prior value tracked alongside the current. At J&J, for a Salesperson dimension, we tracked both `CurrentRegion` and `PreviousRegion` as SCD III — simple to query but limited in historical depth.

My recommendation to clients is always: use SCD II for analytically sensitive attributes, SCD I for low-importance reference data, and SCD III only when you know for certain that exactly one prior state is sufficient."

**Key emphasis:** Give the AXA SCD II example in detail — it's your strongest. Show you understand the business _reason_ for each type, not just the mechanics.

* * *

### Q3. Explain the difference between a Conceptual, Logical, and Physical data model. How do you produce each one?

**Why they're asking:** This is listed explicitly in the essential skills. They want to confirm you understand all three layers and can produce and communicate them to different stakeholders.

**Your Answer:**

"These three layers represent increasing levels of technical detail — and each serves a different audience.

**Conceptual Data Model:** This is the highest-level view — entities and their relationships, no attributes, no data types. It's a communication tool for business stakeholders. At AXA, my conceptual model had entities like Policy, Claim, Customer, and Underwriting Event — connected by high-level relationships. It's the shared language between the data team and business.

**Logical Data Model:** This introduces all attributes, primary keys, foreign keys, and relationships — but remains database-agnostic. At AXA, my logical model for the Claims domain detailed every attribute of the Claim Fact and its related dimensions (Policy, Provider, Date, Geography) including cardinality and nullability. I validated this against business user stories before any physical design.

**Physical Data Model:** This is the implementation-ready design — data types, indexes, partitioning strategies, storage formats. At AXA on Microsoft Fabric Lakehouse, my physical model specified Delta table formats, partition keys (by ClaimDate), clustering columns for query optimisation, and surrogate key generation strategies. On Azure Synapse, I chose distribution strategies (hash vs replicated) based on table size and join patterns.

The key discipline I follow: _never jump to physical design without a validated logical model_. I've seen projects fail because physical tables were built directly from source system schemas without a proper logical model — and the analytical needs were never properly captured.

I document all three levels and maintain them as living artefacts — that STTM documentation practice I've applied across multiple engagements ensures the models stay aligned with the business as it evolves."

* * *

### Q4. What is your experience with Data Lakehouse and Delta Lake architectures?

**Why they're asking:** Both are explicitly listed in the job spec. They want practical, hands-on experience — not theoretical knowledge.

**Your Answer:**

"This is an area I've worked in deeply, particularly at AXA Insurance where I architected a full Lakehouse on Microsoft Fabric.

A **Lakehouse** combines the scalability of a Data Lake (schema-on-read, raw storage, low cost) with the reliability and governance of a Data Warehouse (ACID transactions, schema enforcement, versioning). The architectural pattern I apply is Medallion: Bronze for raw ingestion, Silver for cleansed and conformed data, Gold for business-ready dimensional models.

**Delta Lake** is the storage layer that makes this work. It adds ACID transactions, time travel, schema evolution, and change data feed capabilities on top of Parquet files. At AXA, I used Delta tables across the Lakehouse to:

  * Enable time travel for point-in-time recovery on claims data
  * Implement SCD II patterns using Delta's MERGE operation — matching incoming records against the existing dimension and branching into insert vs update logic
  * Support incremental loading via Change Data Feed without full table scans
  * Allow schema evolution as new data sources were onboarded without breaking downstream pipelines



The practical advantage I highlighted to the AXA team was that Delta Lake allowed us to retire a costly separate staging database — the Bronze layer _became_ the staging environment with full auditability.

I also used OneLake Shortcuts to reference external Azure Data Lake sources without data duplication — maintaining a single source of truth across the estate."

* * *

## SECTION 2: Azure & ETL Architecture

* * *

### Q5. How would you define and document data architecture principles in Azure for a new client engagement?

**Why they're asking:** This is almost verbatim from the job spec ("Defining data architecture principles in Azure"). They want to know you can own this work, not just execute tasks.

**Your Answer:**

"This is something I've done in practice, not just in theory. At AXA Insurance, I was the lead Data Architect responsible for establishing the architecture discipline from scratch for a large-scale Azure migration.

My approach has four phases:

**1\. Discovery & Assessment:** I start with business requirements — what analytical use cases need to be served, what data sources exist, what the current pain points are. I also assess the existing technical estate: on-premise databases, integration patterns, data volumes, and latency requirements.

**2\. Reference Architecture Design:** I define a reference data architecture that is aligned with Microsoft's Azure Well-Architected Framework and the client's specific needs. Typically this includes: the Medallion layering strategy, the ETL/ELT toolchain (Azure Data Factory for orchestration, Dataflows for transformation), storage design (Azure Data Lake Gen2, Delta Lake), the serving layer (Azure Synapse or Microsoft Fabric Warehouse), and the governance stack (Microsoft Purview).

**3\. Principles Documentation:** I document the guiding principles — things like 'data is immutable in the Bronze layer', 'all dimension tables implement SCD II by default', 'no business logic in the ingestion layer', 'all pipelines must be idempotent'. These become the standards the team works to.

**4\. Advocacy & Governance:** I don't just write documents — I run architecture review sessions, review data models before they go to physical design, and update the reference architecture as the project evolves.

At AXA, this resulted in a documented architecture that the whole team of 8 worked from, with clear patterns for ETL design, modelling standards, and governance — and it significantly reduced rework and inconsistency across the engagement."

**Key emphasis:** Show you've _owned_ this — not just contributed to it. Use the AXA example specifically.

* * *

### Q6. Walk me through how you design an ETL flow for a Dimension and Fact table. What are the key considerations?

**Why they're asking:** A core technical question. They want depth on pipeline design, not just tool names.

**Your Answer:**

"ETL design for dimensional data structures follows a clear sequence that I've refined across many engagements.

**For a Dimension table:** The pipeline sequence I follow is: Extract → Cleanse/Conform → Surrogate Key Assignment → SCD Logic → Load.

The critical stage is SCD logic. For SCD II, I use a MERGE pattern:

  * Hash the business key and tracked attributes to detect changes
  * If no match on business key → INSERT as new record with IsCurrent = TRUE
  * If match found and attributes changed → UPDATE old record (IsCurrent = FALSE, ExpiryDate = today) and INSERT new record
  * If match found and no change → pass through



I always separate surrogate key generation from the source natural keys — this decouples the warehouse from upstream system changes.

**For a Fact table:** The sequence is: Extract → Lookup Dimension Surrogate Keys → Apply Business Rules → Validate → Load.

The most important step is the dimension lookup — replacing natural keys with surrogate keys from each dimension. I implement referential integrity checks at this stage: any Fact record that cannot be resolved to a valid Dimension surrogate key is routed to a reject/quarantine table with an error code, not silently dropped.

**Key pipeline considerations I always address:**

  * **Idempotency:** The pipeline must produce the same result if re-run — essential for error recovery
  * **Incremental loading:** I use watermarks, CDC, or Change Data Feed to avoid full reloads wherever possible
  * **Error handling & logging:** Every pipeline logs row counts at each stage and alerts on anomalies
  * **Late-arriving data:** I design Fact pipelines to handle late-arriving dimension records



At AXA, implementing these patterns in Azure Dataflows Gen2 reduced data latency from 24 hours to under 15 minutes."

* * *

### Q7. What is your experience migrating on-premises solutions to Azure? What challenges did you face?

**Why they're asking:** Explicitly in the job spec. TCS's client likely has on-prem legacy systems they need to migrate.

**Your Answer:**

"I've led two significant on-premises to Azure migrations.

The largest was at Affinion/Cognizant — migrating 20M+ customer records from legacy on-premise SQL Server systems to Azure Data Lake and Azure Synapse Analytics. The key challenges and how I addressed them:

**Data Quality at Source:** Legacy systems accumulate decades of inconsistencies. I implemented a thorough data profiling phase using STTM methodology — documenting every source attribute, its population rate, data type mismatches, and transformation rules before writing a single pipeline. This prevented downstream surprises.

**Schema Mapping & Transformation Complexity:** Source schemas rarely map cleanly to target dimensional models. I designed a staging layer in Azure where raw data landed first, and transformation logic was applied in a separate step — keeping ingestion and transformation concerns separated.

**Reconciliation & Validation:** With 20M+ records, you need an automated way to verify the migration is correct. I built reconciliation scripts comparing row counts, aggregate sums, and key business metrics between source and Azure target at every stage of the pipeline.

**Cutover Planning:** Zero-downtime migration required careful orchestration — we ran parallel loads and validated the Azure environment before switching business users over.

More recently at AXA, I led the migration from a legacy insurance data platform to Microsoft Fabric — a different complexity because it was an active production system, not a one-time bulk load. There, the incremental CDC-based approach was essential to keep the Azure environment continuously synchronised while we validated and cut over progressively.

The consistent lesson: **data architecture decisions made during migration last for years**. Getting the modelling and governance right at the outset matters far more than hitting an arbitrary go-live date."

* * *

## SECTION 3: Process, Governance & Stakeholder Management

* * *

### Q8. The job spec mentions "establishing processes and procedures for a Data Architecture discipline at the customer." How would you approach this from day one?

**Why they're asking:** This is a leadership/consulting question. TCS wants a Data Architect who can build capability at the client, not just deliver code.

**Your Answer:**

"This is genuinely one of the most important — and often most underestimated — parts of a Data Architect role. Technical delivery without a functioning architecture discipline creates technical debt that compounds quickly.

My approach on day one is to **observe before prescribing**. I spend the first two weeks understanding the current state: what modelling practices exist (even informally), how data design decisions are made, what the team's skill level is, and what the business's maturity is around data.

Then I establish the discipline in layers:

**Layer 1 — Standards & Reference Architecture:** Document the agreed data architecture principles, modelling standards (which methodology, SCD approaches, naming conventions), and ETL design patterns. Make these accessible and version-controlled.

**Layer 2 — Architecture Review Process:** Introduce lightweight architecture review checkpoints — not bureaucratic gates, but structured conversations where new data models or pipeline designs are reviewed against standards before development starts. This prevents rework.

**Layer 3 — STTM and Documentation Culture:** I mandate Source-to-Target Mapping documentation for every data pipeline. At J&J, I produced STTM for 200+ business attributes. This documentation becomes invaluable for onboarding, auditing, and governance.

**Layer 4 — Knowledge Transfer:** I run working sessions with the team to build capability. The architecture discipline shouldn't be dependent on one person — it should be embedded in the team.

At AXA, establishing these processes early meant that when we onboarded new data domains, the team could apply the patterns independently rather than waiting for me to design every pipeline."

* * *

### Q9. How do you approach working with business stakeholders to gather and understand data requirements?

**Why they're asking:** The job spec mentions "understanding business requirements relating to business data sources." They want to know you can bridge technical and business worlds.

**Your Answer:**

"Data architecture that isn't grounded in business requirements is an academic exercise. I treat requirements gathering as a core part of my role, not something the business analyst does in isolation.

My approach:

**Start with the analytical use case, not the source data.** I ask: what decisions does this data need to support? At AXA, before modelling the Claims domain, I sat with actuaries and claims adjusters to understand exactly what questions they needed to answer — which policy terms were active at claim time, how fraud patterns changed over time, how utilisation compared across regions. Those questions drove my dimensional model design.

**Translate between languages.** Business stakeholders don't speak in entities and attributes — they speak in reports, KPIs, and questions. My job is to translate their language into data model requirements, then present the model back to them in business terms for validation. I use ER diagrams with business-friendly labels, not technical notation, for these reviews.

**Document agreed definitions.** At J&J, one of the most valuable things I produced was the STTM documentation — not just transformation rules, but agreed business definitions for each attribute. What exactly is a 'policy effective date'? Is it the date the policy was issued or when cover commenced? These definitions prevent costly downstream disagreements.

**Iterate.** I don't do big-bang requirements gathering. I validate the conceptual model early, the logical model before physical design, and the first data loads before the full pipeline is built. Catching misalignments early is dramatically cheaper than fixing a production data warehouse."

* * *

### Q10. How do you manage Agile delivery in a data engineering context? What does your typical sprint structure look like?

**Why they're asking:** Significant Agile experience is listed as essential. They want to see you've genuinely worked in Agile, not just attended standups.

**Your Answer:**

"I've worked in Agile delivery throughout my career at both Cognizant and Wipro, and I'm a genuine advocate for it in data engineering — though it requires adaptation from pure software development Agile.

**My typical sprint structure in data projects:**

Stories are written around analytical outcomes, not technical tasks. Rather than 'build Claims fact table', the story is 'As an actuary, I need to analyse claim costs by policy type and region over time' — so the acceptance criteria are business-testable.

In a two-week sprint:

  * **Day 1–2:** Sprint planning, refining data requirements for the sprint's stories, agreeing STTM for new data elements
  * **Day 3–8:** Development — ETL pipeline builds, model implementation, unit testing against agreed business rules
  * **Day 9–10:** Integration testing with the upstream source systems and downstream consumers, data quality validation, sprint review with business stakeholders



**Architecture governance in Agile:** I introduced a lightweight 'Architecture Checkpoint' in sprint planning — a 30-minute review of any new data models or pipeline designs against our reference architecture standards before they go into development. This prevents technical debt from accumulating sprint by sprint.

**Backlog grooming for data:** Source system changes, data quality issues, and new business requirements all go through the backlog. I maintain an Architecture Debt register as part of the backlog so that refactoring work is visible and prioritised alongside feature delivery.

The key adaptation I've found essential in data Agile: _definition of done must include data quality validation_ , not just code completion."

* * *

## SECTION 4: Role-Specific & Situational Questions

* * *

### Q11. Why are you interested in this specific role at TCS, and what do you know about TCS as an organisation?

**Why they're asking:** Standard culture/fit question but important — they want motivated candidates who've done research, not just job applicants.

**Your Answer:**

"A few things drew me specifically to this role.

First, the technical fit is very strong. The role is centred on Azure data architecture — defining principles, designing ETL flows, implementing dimensional models, migrating on-premises to Azure — and these are exactly the areas I've spent the last several years specialising in. The AXA and Affinion migrations in particular directly mirror what this role requires.

Second, the Utilities sector is genuinely interesting to me. Data architecture in regulated, infrastructure-heavy industries like Utilities has specific challenges — real-time data from operational systems, complex regulatory reporting, long historical data retention requirements — and I find that technical complexity engaging.

Third, TCS's scale and the way it operates in the market. As a global IT services leader working with some of the largest enterprises, the exposure to complex, high-stakes data programmes is significant. The learning environment and breadth of projects is something I actively want in my next role.

I'm also drawn to the hybrid Dublin model — I'm Dublin-based, so the 2-day minimum on-site arrangement works well for me."

**Tip:** Research TCS's recent announcements, AI strategy, or any Utilities sector case studies and weave one specific reference in. Shows genuine interest.

* * *

### Q12. Describe a situation where a data architecture decision you made had significant business impact — positive or negative.

**Why they're asking:** Behavioural/STAR question to assess judgement, accountability, and business awareness.

**Your Answer (STAR format):**

"At AXA Insurance, early in the engagement, there was pressure from the project timeline to take a shortcut in the dimensional model design — specifically, to implement SCD I for the Policy dimension rather than SCD II, because SCD II was more complex to build and would take an extra sprint.

**Situation:** The business stakeholders hadn't explicitly asked for policy history — the initial requirements focused on current-state reporting.

**Task:** As Data Architect, I was responsible for the data model design and had to make the call.

**Action:** I proactively went back to the actuarial team and asked a specific question: 'Do you ever need to know what the policy terms were _at the time of the claim_ , not just what they are today?' The answer was an immediate 'yes — that's fundamental to our reserving calculations.' Without SCD II, historical claims analysis against historical policy terms would be impossible. I documented this requirement, updated the logical model to SCD II, and re-scoped the sprint to accommodate it. I presented the trade-off to the project manager clearly: one extra sprint now versus a potentially costly data model rewrite later.

**Result:** The SCD II Policy dimension became one of the most heavily used structures in the platform — actuaries used it constantly for loss reserving and trend analysis. Had we shipped SCD I, we would have been unable to answer fundamental business questions with the data, and a remediation would have been far more expensive than the extra sprint.

The lesson: when modelling decisions interact with analytical requirements, the architect needs to _ask the uncomfortable questions_ proactively, not wait for the business to specify every requirement."

* * *

### Q13. What do you see as the key challenges in establishing a data architecture practice at a client in the Utilities sector?

**Why they're asking:** Tests your sector awareness and consulting mindset. Shows you think beyond technology.

**Your Answer:**

"Without specific knowledge of TCS's Utilities client yet, I can speak to the challenges typical of that sector from what I know:

**Legacy system complexity:** Utilities often have decades-old operational systems — SCADA, AMI (smart meter infrastructure), ERP, and billing systems — that were never designed with analytics in mind. Source data quality and documentation are typically poor. My STTM and data profiling approach is essential here.

**High data volumes and real-time requirements:** Smart meter data, network sensor data, and operational events generate enormous volumes at high frequency. The ETL architecture needs to handle both batch and streaming patterns — and the data modelling needs to accommodate time-series structures alongside traditional dimensional models.

**Regulatory reporting:** Utilities are heavily regulated. Data lineage, audit trails, and the ability to reproduce historical calculations are non-negotiable. This makes Delta Lake's time travel capability and Purview's lineage tracking particularly valuable.

**Organisational change:** A data architecture discipline often doesn't exist formally at the client. Getting buy-in from IT teams, data owners, and business stakeholders requires clear communication, quick wins, and demonstrating value early.

**On-premises to Azure migration:** Many Utilities still run on-premises infrastructure. The migration path needs to preserve business continuity while modernising the platform — which is directly what I've done at AXA and Affinion.

I'd want to understand TCS's specific client's maturity level in my first meetings to tailor the architecture approach accordingly."

* * *

### Q14. What is your preferred approach to data governance and lineage in an Azure environment?

**Why they're asking:** Data governance is increasingly a non-negotiable in enterprise data architecture and is in the job spec implicitly through Azure Data Architecture responsibility.

**Your Answer:**

"My preferred toolset for Azure data governance is Microsoft Purview, which I've implemented in production at AXA Insurance.

**Lineage:** I configure Purview to automatically scan Azure Data Factory pipelines and Fabric/Synapse assets, which captures end-to-end lineage from source system through Bronze → Silver → Gold layers automatically. For custom transformations, I supplement with manual lineage entries. This means business stakeholders and auditors can trace any KPI on a dashboard back to its source system record — which is invaluable for both trust and compliance.

**Data Classification & Sensitivity:** I implement sensitivity labels — PII, financial data, regulated data — as part of the initial data architecture design, not as an afterthought. At AXA, this was essential for claims data containing health and personal financial information.

**Data Quality:** I don't treat this as purely a governance topic — I embed quality checks directly into the ETL pipelines. At AXA, I built PySpark-based quality frameworks in Fabric Notebooks that ran automatically after each pipeline execution, flagging anomalies and alerting the team before data reached the Gold layer.

**Governance as Architecture Principle:** My key belief is that governance cannot be bolted on after the fact. The naming conventions, the SCD approach, the surrogate key strategy, the STTM documentation — these governance decisions are made at the modelling stage, not added later. I establish these as architecture principles at the start of each engagement."

* * *

## SECTION 5: Technical Deep Dives (Rapid Fire)

* * *

### Q15. What is the difference between a Data Lake, a Data Warehouse, a Data Mart, and a Data Lakehouse?

**Your Answer:**

"**Data Lake:** Schema-on-read storage for raw, unprocessed data in its native format. Extremely flexible and cheap, but without structure it becomes a 'data swamp' without governance. Azure Data Lake Storage Gen2 is my primary tool here.

**Data Warehouse:** Schema-on-write, structured, governed — optimised for analytical queries. Data is cleansed, modelled, and integrated before load. Azure Synapse Analytics or Microsoft Fabric Warehouse in my recent work.

**Data Mart:** A subset of the data warehouse, scoped to a specific business domain or team — e.g., a Finance data mart or a Claims data mart. Faster to build, easier for end users to consume, but risk of siloing if not federated properly.

**Data Lakehouse:** Combines the raw storage flexibility of a Data Lake with the reliability, ACID transactions, and schema enforcement of a Data Warehouse — enabled by storage formats like Delta Lake. This is my primary recommended architecture for modern Azure deployments. It eliminates the need for separate Lake and Warehouse infrastructure in many scenarios."

* * *

### Q16. What is normalisation and when would you choose to not normalise?

**Your Answer:**

"Normalisation is the process of structuring a relational database to reduce data redundancy and improve integrity — typically through 1NF, 2NF, 3NF, and BCNF. It ensures every attribute depends on the key, the whole key, and nothing but the key.

I apply full 3NF normalisation in operational databases and in the conformed/Silver layer of my Medallion architectures — where data integrity and update consistency are paramount.

However, in the analytical Gold layer (dimensional model / data warehouse), I **deliberately denormalise** using star or snowflake schemas. A Fact table with surrogate key lookups to flat Dimension tables is technically denormalised — but that redundancy is a deliberate trade-off for query performance and analytical simplicity. Actuaries at AXA don't want to join 15 tables to answer a question. A denormalised dimension with all relevant attributes pre-joined is far more performant and intuitive.

The rule I follow: **normalise for write operations; denormalise strategically for read/analytics.** "

* * *

### Q17. What is a surrogate key and why is it important in dimensional modelling?

**Your Answer:**

"A surrogate key is a system-generated, meaningless integer key assigned to each row in a dimension table — completely separate from the business's natural key (like a customer ID or policy number from the source system).

They're critical for three reasons:

  1. **SCD II support:** When a dimension record changes and we insert a new row (SCD II), the surrogate key uniquely identifies that specific version of the record. Fact tables point to the surrogate key — so historical facts remain correctly associated with the dimension state that was active at the time.
  2. **Source system independence:** If the source system renumbers its keys, or if data comes from multiple source systems with overlapping natural keys, the surrogate key insulates the data warehouse from those upstream changes.
  3. **Performance:** Integer surrogate keys are faster to join than composite or string natural keys.



At AXA, I implemented a surrogate key generation strategy using sequences in Fabric Lakehouse — ensuring every dimension table had a clean integer SK with no dependency on the source system's identifier strategy."

* * *

_End of Interview Preparation Guide_

* * *

**Final Tips:**

  * **Open strongly:** When asked "Tell me about yourself" — lead with the Azure migration and dimensional modelling experience. Don't start with your education.
  * **Use numbers:** Every impact statement should have a metric — 93% latency reduction, 20M records, $2M savings, 150+ users. Numbers make answers memorable.
  * **Ask good questions:** At the end, ask about the specific client, what the current data maturity looks like, and what the first 90 days are expected to look like. It shows consulting mindset.
  * **Hybrid commitment:** Proactively confirm you're Dublin-based and comfortable with the 2-day on-site requirement — it removes any ambiguity.
