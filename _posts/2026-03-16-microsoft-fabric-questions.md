---
title: "Microsoft Fabric Questions"
date: 2026-03-16 17:45:52 
author: shajeebhameed
layout: page
slug: microsoft-fabric-questions
status: publish
---

#### Q1. What is Microsoft Fabric and how does it differ from traditional analytics platforms?  
  
**What they're assessing:** Your fundamental understanding of Fabric as a unified platform (SaaS) versus assembling components (PaaS).

**How to structure your answer:**

  1. **Definition:** Start with a clear, high-level definition.
  2. **Key Differentiator:** Explain the "unified" aspect (OneLake, shared compute) and how it simplifies the traditional PaaS model.
  3. **Connect to your experience:** Briefly mention how this aligns with the convergence you've seen in modern platforms like Snowflake.



> **Sample Answer:**  
>  "Microsoft Fabric is a unified, software-as-a-service (SaaS) analytics platform that brings together data engineering, data warehousing, data science, real-time analytics, and business intelligence into a single, cohesive environment. Its core is **OneLake** , a single, logical data lake that underpins all experiences .
> 
> The key difference from traditional platforms, like assembling a solution with separate Azure Data Lake, Azure Synapse, and Azure Data Factory, is the **deep integration**. In Fabric, the Lakehouse, Data Warehouse, and Power BI all automatically share data on OneLake without needing to move or copy it. This drastically reduces complexity and eliminates the infrastructure management overhead of a traditional PaaS setup. This unified approach aligns perfectly with the lakehouse architectures I've implemented, such as at AXA, where we built a Bronze-Silver-Gold medallion architecture but within a single platform for simplicity and performance."

#### Q2. Explain the role of OneLake and the concept of Shortcuts. How would you troubleshoot a broken Shortcut?

**What they're assessing:** Your knowledge of Fabric's core storage layer and your troubleshooting methodology .

**How to structure your answer:**

  1. **Define OneLake:** Describe it as the single, logical data lake for the entire organization.
  2. **Explain Shortcuts:** Explain that they are virtual pointers to data stored elsewhere (ADLS, AWS S3, or another Fabric workspace) without physically copying the data.
  3. **Troubleshooting:** Provide a step-by-step diagnostic approach.



> **Sample Answer:**  
>  "**OneLake** is Fabric's unified storage foundation. It provides a single-pane-of-glass for all your data, whether it's in a Lakehouse, Warehouse, or even external sources.
> 
> **Shortcuts** are a key feature that allows us to virtualize data from external sources or other Fabric workspaces into OneLake. This promotes reusability and prevents data duplication, which is critical for maintaining a single source of truth—a principle I strictly adhered to during the Affinion migration.
> 
> To troubleshoot a **broken Shortcut** , I'd follow a systematic process:
> 
>   1. **Verify the Source:** First, I'd check if the underlying source data still exists at the original path and hasn't been moved or deleted. This involves checking the source storage account (e.g., Azure Data Lake) or the source Fabric workspace.
>   2. **Check Permissions:** The most common issue is a permissions failure. I would verify that the Fabric service or the user account has the necessary **Access Control Lists (ACLs)** and read permissions on the source location .
>   3. **Refresh Shortcut Metadata:** In the Fabric portal, I would attempt to refresh the shortcut's metadata to see if it's a transient syncing issue.
>   4. **Recreate if Necessary:** If the source URI has permanently changed, the shortcut would need to be deleted and recreated pointing to the new location ."
> 


#### Q3. Walk me through the medallion architecture (Bronze, Silver, Gold) in a Microsoft Fabric Lakehouse.

**What they're assessing:** Your ability to translate a common architectural pattern into Fabric-specific components. This is a direct hit based on your CV.

**How to structure your answer:**

  1. **Link to Your Experience:** Start by stating you implemented this at AXA.
  2. **Define Layers:** Clearly explain the purpose of each layer (Bronze: raw, Silver: cleansed/validated, Gold: business-ready aggregations).
  3. **Map to Fabric:** Explain how you would build these layers using Fabric tools.



> **Sample Answer:**  
>  "Absolutely. This is a pattern I successfully implemented for AXA Insurance to modernize their claims processing. In a Microsoft Fabric Lakehouse, the medallion architecture is a perfect fit for organizing data.
> 
>   * **Bronze Layer:** This is the raw data ingestion layer. I'd use **Dataflows Gen2** or **Pipelines** to land raw claims data directly into a **Lakehouse** table in Delta Parquet format . This serves as our immutable source of truth for auditability, a key requirement for their compliance needs.
>   * **Silver Layer:** This is the cleansing and validation layer. Using **PySpark notebooks** within Fabric, I would clean the data, remove duplicates, apply business rules, and join with master data. The output is written as a new set of Delta tables in the same or a different Lakehouse, representing a trusted, integrated view of claims.
>   * **Gold Layer:** This is the business aggregation layer. Here, I'd create aggregated tables and fact/dimension models to support specific use cases like actuarial analysis or fraud detection. These tables can be stored in a **Warehouse** or as Lakehouse tables with a **SQL Endpoint**. Finally, **Power BI** would connect to these Gold layer tables using **Direct Lake mode** for high-performance, near-real-time reporting . This entire flow within a single platform is what makes Fabric so powerful."
> 


#### Q4. What is Direct Lake mode in Power BI, and what are its performance considerations?

**What they're assessing:** Your understanding of this key Fabric feature that bridges the data lake and BI .

**How to structure your answer:**

  1. **Define Direct Lake:** Explain it as a mode where Power BI reads the Delta tables directly from OneLake, bypassing the need for an import or a query engine.
  2. **Benefits:** Mention the key advantages: high performance like Import mode, with the data freshness closer to DirectQuery.
  3. **Performance Considerations:** Discuss when it works best and potential pitfalls.



> **Sample Answer:**  
>  "**Direct Lake** is a revolutionary query mode in Power BI for Fabric. It allows Power BI to read the Delta Parquet files directly from OneLake, without having to first import the data into the Power BI model's memory or execute live queries against a SQL endpoint. This provides the best of both worlds: the high query performance of Import mode and the ability to work with large, fresh datasets.
> 
> However, for optimal performance, there are key considerations. If the Delta tables are not optimized—for example, if they have **too many small files** —query performance can degrade . To mitigate this, we must ensure Spark operations perform **OPTIMIZE** and **VACUUM** regularly to compact small files. Also, while Direct Lake is fast, very complex models might still benefit from some in-memory processing. In my AXA project, after setting up the Gold layer, we ensured proper table maintenance (using `OPTIMIZE` on Delta tables) to guarantee the sub-second query performance needed for their analysts."

#### Q5. Compare and contrast using Dataflows Gen2 vs. PySpark Notebooks for data transformation in Fabric.

**What they're assessing:** Your ability to choose the right tool for the job, demonstrating engineering judgment .

**How to structure your answer:**

  1. **State the Purpose of Each:** Explain the strengths of each tool.
  2. **Provide a Decision Framework:** Give clear examples of when you'd pick one over the other.
  3. **Connect to your (updated) CV:** Mention your experience with both.



> **Sample Answer:**  
>  "Both are essential tools in a Fabric engineer's toolkit, and I have experience leveraging both.
> 
>   * **Dataflows Gen2** are a low-code, Power Query-based ETL tool. They are ideal for **faster development cycles, simpler data transformations, and data ingestion** , especially for business analysts or data engineers who prefer a graphical interface. I'd use them for ingesting data from sources like SharePoint or Salesforce and performing initial light cleansing.
>   * **PySpark Notebooks** provide a code-first experience for complex, large-scale data engineering. They are best suited for **advanced transformations, implementing complex business logic, and performing large-scale aggregations** . They offer much more flexibility and control.
> 

> 
> My decision framework is based on complexity and scale. For the **AXA project** , we used Dataflows Gen2 for initial ingestion of reference data, but the core claims processing—involving complex joins, window functions, and business rules—was all done in **PySpark notebooks** to ensure we could scale to millions of claims efficiently and have full control over the transformation logic."

* * *

### Category 2: PySpark & Data Engineering

This section validates the skills you've added to your CV .

#### Q6. Explain the concept of lazy evaluation in PySpark and how it impacts performance.

**What they're assessing:** Your fundamental understanding of Spark's distributed processing model.

**How to structure your answer:**

  1. **Define Lazy Evaluation:** Explain that transformations are not executed immediately but build a DAG (Directed Acyclic Graph).
  2. **Explain the Trigger:** Clarify that execution only happens when an "action" (like `.show()`, `.count()`, `.write()`) is called.
  3. **Performance Benefit:** Explain how this allows Spark's Catalyst Optimizer to create the most efficient execution plan.



> **Sample Answer:**  
>  "**Lazy evaluation** is a core optimization strategy in PySpark. When I apply transformations like `.filter()` or `.select()` to a DataFrame, Spark doesn't execute them right away. Instead, it builds a **DAG** , or a logical execution plan, of all the operations I intend to perform.
> 
> Execution only begins when an **action** , such as `.count()` or `.write()`, is called. This is a huge performance benefit because the **Catalyst Optimizer** can then analyze the entire DAG to create the most efficient physical execution plan. For example, it might push down filters or reorder joins to minimize data shuffling across the cluster. Without lazy evaluation, each transformation would materialize intermediate results, causing massive I/O and slowdowns, especially when processing the large claims datasets we handled at AXA."

#### Q7. How do you handle data skew in a PySpark job? Give an example of how you've diagnosed and solved it.

**What they're assessing:** Your ability to troubleshoot real-world performance problems.

**How to structure your answer:**

  1. **Define Data Skew:** Explain that it happens when data isn't evenly distributed across partitions, causing some tasks to take much longer than others.
  2. **Diagnosis:** Explain how you would identify it (e.g., via the Spark UI's stage and task timeline).
  3. **Solutions:** Mention a few common strategies (salting, increasing partitions, broadcast joins).



> **Sample Answer:**  
>  "Data skew is a common challenge where some partitions have significantly more data than others, leading to a few long-running 'straggler' tasks that bottleneck the entire job. I diagnosed this once in a previous role by looking at the **Spark UI** and noticing that for a particular join stage, most tasks completed in seconds, but a few were taking minutes because the join key was highly skewed (e.g., a few customers with millions of transactions).
> 
> To solve it, I used a **salting** technique. I added a random number (the 'salt') to the skewed join key, breaking it into smaller, more manageable sub-keys. I then performed the join in two steps, ultimately aggregating the results. Another approach, if the skewed data came from a small table, would be to use a **broadcast join** to avoid shuffling the large skewed dataset altogether. For the massive datasets at Refinitiv, having these troubleshooting skills was critical to maintaining our SLAs."

#### Q8. What is the difference between `repartition()` and `coalesce()`? When would you use each?

**What they're assessing:** Your understanding of partition management and its performance implications .

**How to structure your answer:**

  1. **Define Both:** Explain what each function does.
  2. **Highlight the Key Difference:** `repartition()` shuffles data across the network; `coalesce()` minimizes shuffling by merging existing partitions.
  3. **Provide Use Cases:** Give clear scenarios for using each.



> **Sample Answer:**  
>  "Both are used to manage the number of partitions, but they work very differently.
> 
>   * **`repartition(num_partitions)`** is an expensive operation that performs a **full shuffle** of the data across the cluster to create a new, even distribution of partitions. I would use `repartition()` when I need to **increase** the number of partitions (e.g., to increase parallelism after heavy filtering) or to ensure an even distribution before a wide transformation like a join.
>   * **`coalesce(num_partitions)`** is an optimized version that only **reduces** the number of partitions. It tries to merge existing partitions into a smaller number, minimizing data shuffling as much as possible. I would use `coalesce()` after a filter operation that has left many partitions nearly empty, or just before writing out a dataset to create a reasonable number of output files, as it's much more efficient than `repartition()` in these scenarios ."
> 


* * *

### Category 3: Data Governance (Microsoft Purview)

This section addresses the "Ideally, you will also have" requirement for Purview .

#### Q9. How would you implement a data governance strategy for a Microsoft Fabric environment using Purview?

**What they're assessing:** Your ability to translate governance principles into a concrete plan with Purview.

**How to structure your answer:**

  1. **Start with the 'Why':** Emphasize the goals: compliance, security, and data discoverability.
  2. **Map Features to Goals:** Explain how different Purview features address these goals.
  3. **Connect to Your Experience:** Reference your past governance work.



> **Sample Answer:**  
>  "My approach to data governance, which I've applied at companies like JNJ and Travelers, is to build it in layers. For a Fabric environment, I would leverage Microsoft Purview as the central control plane.
> 
>   1. **Data Mapping & Classification:** I'd start by using Purview to **automatically scan** the data in OneLake (Lakehouses, Warehouses) to create an up-to-date data map and inventory . I'd then apply **sensitivity labels** to automatically classify sensitive data, such as PII in AXA's claims data, based on built-in or custom sensitive information types.
>   2. **Data Lineage:** Next, I'd ensure Purview is capturing **data lineage** from Fabric activities . This is crucial for impact analysis (e.g., "if I change this source table, what reports will break?") and for audit compliance, which was a major requirement at Travelers.
>   3. **Policy Enforcement:** Finally, I'd work with the data owners to define and enforce data access policies within Fabric, ensuring that the classifications in Purview are used to drive access control decisions, aligning with principles I established while creating data governance frameworks in the past."
> 


#### Q10. What are sensitivity labels and how do they contribute to data security and compliance?

**What they're assessing:** Your understanding of a core Purview concept .

**How to structure your answer:**

  1. **Define Sensitivity Labels:** Explain they are metadata tags that indicate the sensitivity of data (e.g., Public, Confidential, Highly Restricted).
  2. **Explain the Mechanism:** Describe how they can be applied manually or automatically based on content.
  3. **Explain the Benefits:** Discuss how they enable protection (e.g., encryption, access restrictions) and compliance reporting.



> **Sample Answer:**  
>  "**Sensitivity labels** from Microsoft Purview are tags that classify and protect an organization's data based on its business impact, confidentiality, or regulatory requirements . For example, we could have labels like 'Public,' 'Internal,' 'Confidential,' and 'Highly Restricted.'
> 
> In a Fabric environment, these labels can be applied automatically to data in OneLake based on the content, like detecting a credit card number or a national ID. Once applied, they contribute to security by enforcing protective actions, such as preventing 'Highly Restricted' data from being shared externally or requiring special permissions to access. This is fundamental for compliance with regulations like GDPR, as it allows an organization to prove it has controls in place to protect personal data . At JNJ, having a clear classification scheme for our data was the first step in meeting their strict compliance needs."

* * *

### Category 4: Consulting & Leadership (Senior Manager)

This section is critical for a senior role at EY. They need to see you as a trusted advisor, not just a technician .

#### Q11. Describe a time when you had to influence a client stakeholder who was resistant to your proposed data architecture.

**What they're assessing:** Your consulting and communication skills, specifically your ability to build trusted advisor relationships.

**How to structure your answer (STAR Method):**

  1. **Situation:** Set the context.
  2. **Task:** Describe the challenge with the stakeholder.
  3. **Action:** Detail the steps you took to listen, build a prototype, and demonstrate value.
  4. **Result:** Explain the positive outcome and the relationship you built.



> **Sample Answer (using the Travelers story we discussed earlier):**  
>  "**Situation:** At Travelers, while implementing the new Snowflake architecture, the lead actuary was highly resistant. He had been burned by failed data migrations before and didn't trust that we could maintain the integrity of his claims data, which is critical for loss reserving.  
> **Task:** My task was to earn his trust and get his buy-in, as his team's data was central to the project's success. A direct 'trust me' approach wouldn't work.  
> **Action:** I scheduled a one-on-one meeting just to listen to his concerns and learn about his past negative experiences. I acknowledged that his fears were valid. Then, instead of arguing, I built a prototype. I took one month of his most complex claims data, ran it through our Bronze-Silver-Gold pipeline, and created a detailed reconciliation report. I then walked him through the report, showing him exactly how our validation framework would catch any discrepancies before they impacted his work. I asked him to be part of the UAT process, validating the transformed data himself.  
> **Result:** He became one of our strongest advocates. He even discovered some data quality issues in the source systems that we could help him fix. This experience reinforced for me that transparency, listening, and delivering tangible proof are the most powerful tools to build a trusted advisor relationship."

#### Q12. How would you handle a situation where a client's business team sees your data governance policies as "unnecessary red tape"?

**What they're assessing:** Your ability to communicate value, build consensus, and adapt policies without compromising security .

**How to structure your answer:**

  1. **Acknowledge and Listen:** Start by understanding their perspective and frustration.
  2. **Reframe the Narrative:** Explain how governance enables them, rather than blocks them.
  3. **Use Real-World Examples:** Connect policies to tangible business outcomes (e.g., preventing costly errors, ensuring data is trusted for decisions).
  4. **Seek a Balance:** Show willingness to adjust the implementation to make it more palatable while maintaining the core security and compliance objectives.



> **Sample Answer:**  
>  "This is a very common challenge in governance roles. My first step would be to **listen to their concerns** —is it slowing them down? Is it confusing? Understanding the root of the resistance is key.
> 
> Then, I'd work to **reframe the narrative**. Instead of 'red tape,' I'd position governance as a way to ensure that the data they are using is **trustworthy and consistent** , which actually speeds up their analysis because they no longer have to clean or argue about data definitions. I'd use examples from past projects, like at JNJ where we found $2M in savings by having clean, governed data, to make it tangible.
> 
> Finally, I'd look for a **compromise**. Maybe the policy is sound, but the process for requesting access is too cumbersome. We could streamline that while keeping the underlying security controls. The goal is to be a partner in their success, not an obstacle. This approach of 'security _as an enabler_ ' is something I always strive for."

#### Q13. What do you think are the biggest challenges in a data migration project, and how would you mitigate them from a consulting perspective?

**What they're assessing:** Your project management and risk mitigation skills, leveraging your Affinion experience.

**How to structure your answer:**

  1. **Name the Challenge:** Identify a key risk (e.g., poor source data quality, scope creep, stakeholder alignment).
  2. **Explain the Mitigation:** Detail the proactive steps you take to manage that risk.
  3. **Connect to Experience:** Relate it directly to your Affinion project.



> **Sample Answer:**  
>  "Based on my experience leading the 20-million-record migration for Affinion, I believe the single biggest challenge is **underestimating the complexity and poor quality of legacy source data**.
> 
> To mitigate this from a consulting perspective, I always insist on a **'discovery-first' approach**. Before committing to a timeline or even a full architecture, we must conduct a thorough **data profiling and assessment phase**. At Affinion, we spent weeks just profiling the legacy Oracle data, which allowed us to identify orphan records and data quality issues _before_ writing a single line of migration code. This upfront investment saved us from countless headaches later and allowed us to give the client a realistic timeline and set accurate expectations—a key part of building trust. We also established a **rigorous validation framework** from day one, with reconciliation scripts that ran in parallel during the migration, giving the client real-time visibility into our progress and data fidelity [referencing your Affinion achievements]."

* * *

### Category 5: General & Behavioral

These foundational questions will often precede or follow the technical grilling .

#### Q14. Tell me about yourself. (The Elevator Pitch)

**What they're assessing:** Your ability to synthesize your career into a compelling, concise story that leads to this role.

**How to structure your answer:**

  1. **Present:** Briefly state your current role and core expertise.
  2. **Past:** Highlight key experiences and the value you delivered.
  3. **Future:** Connect it to why you are interested in this specific EY role.



> **Sample Answer:**  
>  "I'm a Data Architect with over 9 years of experience helping global companies turn their complex, disparate data into a strategic, business-ready asset. My expertise lies in leading end-to-end data transformations, from profiling legacy systems and designing modern lakehouse architectures to implementing robust governance frameworks.
> 
> I've had the privilege of doing this across several industries, most recently for AXA Insurance where I led the migration of their critical claims processing to a modern platform, and for Johnson & Johnson, where my analysis identified over $2M in potential cost savings. Throughout these engagements at Cognizant, I've acted as a trusted advisor, working closely with client leadership to ensure our technical solutions directly addressed their business challenges.
> 
> I'm excited about this Senior Manager role at EY because it perfectly aligns with my experience in leading complex data migrations and architecting solutions on Microsoft Fabric, a platform I see as the future of unified analytics. I'm drawn to EY's purpose-driven culture and the opportunity to mentor teams and shape data strategy for a wider range of clients."

#### Q15. Why do you want to join EY? / Why are you looking for a change?

**What they're assessing:** Your motivation, cultural fit, and career alignment .

**How to structure your answer:**

  1. **Be Positive:** Frame it as a pull towards EY's opportunities, not a push away from your current role.
  2. **Align with Your Goals:** Mention specific things about the role/company that appeal to your career aspirations (e.g., senior leadership, specific tech like Fabric, client variety).
  3. **Show You've Done Your Research:** Mention EY's purpose, their focus on AI & Data, etc.



> **Sample Answer:**  
>  "I'm at a point in my career where I'm looking to take the next step into a more strategic, senior leadership role, and the opportunity at EY is exactly what I've been hoping for. While I've grown tremendously at Cognizant, I'm incredibly excited by EY's bold investment in the AI and Data space and the chance to work with Microsoft Fabric at an enterprise scale from day one.
> 
> I'm drawn to EY's purpose of 'building a better working world' and the firm's reputation for developing deep, trusted advisor relationships with clients. I want to move beyond just delivering a project and have a hand in shaping a client's overall data strategy, which is a key part of this Senior Manager role. The chance to join a globally connected team of diverse experts and mentor the next generation of data architects is also a huge motivational factor for me."
> 
> * * *
> 
> Based on your request for more questions and the search results, I've compiled **additional advanced questions** targeting the specific areas you wanted to improve: **Consulting Experience** , **PySpark/Notebooks** , and **Governance (Purview)**. I've also included new categories based on real EY interview patterns .
> 
> * * *
> 
> ## 📋 NEW QUESTION CATEGORIES
> 
> Category| Focus Areas| Source  
> ---|---|---  
> **Advanced Fabric Troubleshooting**|  Real-world problem diagnosis|   
> **Consulting & Client Management**| Stakeholder influence, requirements gathering|   
> **PySpark Performance Deep-Dive**|  Optimization, skew handling, debugging|   
> **Purview & Data Governance Scenarios**| Compliance, lineage, sensitive data|   
> **Case Study / Design Problems**|  End-to-end architecture design|   
>   
> * * *
> 
> # Category 1: ADVANCED FABRIC TROUBLESHOOTING
> 
> ## Q16. A Pipeline in Fabric is failing intermittently when calling a Notebook. How would you diagnose and fix this?
> 
> **What they're assessing:** Your ability to troubleshoot real-world Fabric issues systematically .
> 
> **How to structure your answer:**
> 
>   1. **Check Error Output:** Start with the Pipeline activity error details
>   2. **Verify Resource Linkage:** Ensure the Notebook hasn't been moved or renamed
>   3. **Examine Notebook Itself:** Run the Notebook independently to isolate the issue
>   4. **Check Dependencies:** Verify data sources, mount points, and library imports
>   5. **Review Capacity:** Check if you're hitting capacity limits or throttling
> 

>
>> **Sample Answer:**  
>  "This happened to me on a client project, and I developed a systematic troubleshooting approach:
>> 
>> **First** , I'd examine the **Pipeline activity error output** in the Fabric Monitor hub to get the specific error message . This often reveals whether it's a permission issue, timeout, or code failure.
>> 
>> **Second** , I'd verify the **resource linkage** —if the Notebook was moved or renamed, the Pipeline reference breaks. I'd check the activity configuration to ensure it points to the correct workspace and item ID.
>> 
>> **Third** , I'd run the **Notebook independently** to confirm it works in isolation. If it fails there too, the issue is in the Notebook code itself.
>> 
>> **Fourth** , I'd check **data dependencies** —if the Notebook reads from OneLake or external sources, I'd verify those locations are accessible and haven't changed permissions .
>> 
>> **Finally** , I'd review **capacity metrics** in the Fabric portal. If we're hitting throttling limits, I might need to adjust the Pipeline schedule or request a capacity increase.
>> 
>> At AXA, we had a similar intermittent failure that turned out to be a timeout during peak hours. We fixed it by adding retry logic and optimizing the Notebook to process data more efficiently."
> 
> * * *
> 
> ## Q17. A user reports they cannot see data in a Power BI report connected to a Fabric Lakehouse via Direct Lake. What's your diagnostic process?
> 
> **What they're assessing:** Your understanding of the Direct Lake architecture and permission layers .
> 
> **How to structure your answer:**
> 
>   1. **Check Workspace Access:** Verify user has access to the workspace
>   2. **Check Lakehouse Permissions:** Verify Read permission on the Lakehouse
>   3. **Check SQL Endpoint Access:** Test direct SQL endpoint connection
>   4. **Check Semantic Model Permissions:** Verify RLS/OLS settings
>   5. **Check Data Itself:** Ensure tables aren't empty and Delta logs are intact
> 

>
>> **Sample Answer:**  
>  "This is a common issue with layered permissions in Fabric. I'd diagnose systematically:
>> 
>> **Layer 1: Workspace Access** — First, I'd confirm the user has at least Viewer role in the Fabric workspace containing the Lakehouse and the report .
>> 
>> **Layer 2: Lakehouse Permissions** — Even with workspace access, the user needs **Read permission on the Lakehouse item itself**. I'd check this in the Lakehouse permissions settings.
>> 
>> **Layer 3: SQL Endpoint** — I'd have the user try to query the Lakehouse's SQL endpoint directly using a tool like Azure Data Studio. If this fails, the issue is at the data layer, not Power BI.
>> 
>> **Layer 4: Semantic Model** — If the report uses a semantic model, I'd check for **Row-Level Security (RLS)** or **Object-Level Security (OLS)** that might be filtering data for that user .
>> 
>> **Layer 5: Data Integrity** — Finally, I'd verify the Delta tables themselves are healthy—checking the `_delta_log` folder for corruption and running `OPTIMIZE` if needed .
>> 
>> At AXA, we once spent hours troubleshooting only to discover the user hadn't been added to the workspace at all—always start with the simplest explanation!"
> 
> * * *
> 
> ## Q18. What are Common Causes of Slow Performance in Fabric Dataflows Gen2, and How Do You Optimize Them?
> 
> **What they're assessing:** Your practical optimization knowledge for low-code ETL .
> 
> **How to structure your answer:**
> 
>   1. **Identify Common Issues:** List typical bottlenecks (large data volumes, inefficient steps, external sources)
>   2. **Diagnostic Approach:** Explain how to identify the problem
>   3. **Optimization Techniques:** Provide specific solutions
> 

>
>> **Sample Answer:**  
>  "Based on my experience, Dataflows Gen2 performance issues typically stem from:
>> 
>> **1\. Pulling Too Much Data** — The most common mistake is loading entire tables instead of filtering at the source. I always **filter as early as possible** in the Power Query steps.
>> 
>> **2\. Inefficient Step Order** — Operations like joins and merges should happen **after filtering** , not before. I review the query folding indicators to ensure work is being pushed back to the source when possible.
>> 
>> **3\. Timeouts with External Sources** — When connecting to on-premises or external data, gateway issues or network latency can cause failures. I monitor refresh history and consider using staging queries.
>> 
>> **4\. Large Data Volumes** — For datasets beyond a few GB, Dataflows Gen2 may struggle. At that point, I'd recommend moving the transformation to **PySpark Notebooks** instead .
>> 
>> **Diagnostic Approach:** I always start with the **refresh history** in the Fabric portal, which shows step-by-step execution times. This quickly identifies which step is the bottleneck.
>> 
>> **Optimization Example:** In a recent project, a client's Dataflow was taking 2+ hours. We found they were joining two large tables in Power Query instead of in the source SQL. By rewriting the source query to perform the join before ingestion, we cut runtime to 15 minutes."
> 
> * * *
> 
> # Category 2: CONSULTING & CLIENT MANAGEMENT
> 
> ## Q19. How Do You Gather Requirements from Business Stakeholders at the Start of a Data Architecture Project?
> 
> **What they're assessing:** Your consulting methodology and ability to translate business needs into technical solutions .
> 
> **How to structure your answer:**
> 
>   1. **Start with Business Goals:** Understand what the business wants to achieve
>   2. **Identify Stakeholders:** Map who needs to be involved
>   3. **Use Structured Workshops:** Facilitate sessions to gather requirements
>   4. **Document and Validate:** Create artifacts and get sign-off
>   5. **Connect to Technical Design:** Show how requirements become architecture
> 

>
>> **Sample Answer:**  
>  "Requirements gathering is where successful projects are built. My approach is:
>> 
>> **1\. Understand Business Goals First** — Before talking about data, I ask: 'What business problem are we trying to solve?' At AXA, we started with 'We need to detect fraud faster'—that 15-minute latency target came directly from the business, not from me.
>> 
>> **2\. Identify All Stakeholders** — I map out who has a stake: business users, data owners, compliance, IT operations. Each group has different needs. At JNJ, we needed input from HR, Payroll, Procurement—all had different requirements .
>> 
>> **3\. Facilitate Structured Workshops** — I use a combination of:
>> 
>>   * **Current state mapping** (as-is processes)
>>   * **Future state visioning** (to-be scenarios)
>>   * **Data discovery sessions** (what data exists, where, how it's used)
>> 

>> 
>> **4\. Document in Business-Friendly Language** — I create artifacts like:
>> 
>>   * **Conceptual data models** (entities and relationships, no technical jargon)
>>   * **User journey maps** for data flows
>>   * **Prioritized requirements list** with MoSCoW (Must-have, Should-have, Could-have, Won't-have)
>> 

>> 
>> **5\. Validate and Get Sign-off** — I review everything with stakeholders before writing a line of code. This prevents the 'I thought you meant something else' problem later.
>> 
>> **6\. Trace to Technical Design** — Every technical decision should trace back to a business requirement. When I designed the AXA medallion architecture, I could point to specific requirements that drove each layer: Bronze for audit compliance, Silver for data quality, Gold for analyst speed ."
> 
> * * *
> 
> ## Q20. Describe a Time When You Had to Advocate for a Data Architecture Decision That Was Initially Unpopular with the Client.
> 
> **What they're assessing:** Your ability to influence and build trusted advisor relationships .
> 
> **How to structure your answer (STAR Method):**
> 
>   1. **Situation:** Set the context
>   2. **Task:** Describe the unpopular decision you needed to advocate for
>   3. **Action:** Explain how you built your case, listened to concerns, and persuaded stakeholders
>   4. **Result:** Share the positive outcome
> 

>
>> **Sample Answer (using a real example):**  
>  "**Situation:** At Travelers, I recommended implementing a Bronze-Silver-Gold medallion architecture in Snowflake. The actuarial team was resistant—they wanted to keep data in the source system format because they 'trusted it' and were burned by past migrations.
>> 
>> **Task:** I needed to earn their trust and get buy-in for an architecture I knew was essential for scalability and governance.
>> 
>> **Action:**
>> 
>>   1. **Listened First** — I scheduled one-on-one meetings to understand their concerns. They had legitimate fears based on negative past experiences.
>>   2. **Built a Prototype** — Instead of arguing, I took one month of their most complex claims data through the proposed pipeline and created a detailed reconciliation report showing 100% data fidelity.
>>   3. **Demonstrated Value** — I showed them additional data quality checks they didn't have before, and performance improvements (queries that took hours now took seconds).
>>   4. **Involved Them** — I asked them to validate the transformed data against their existing reports and incorporated their feedback into the validation framework.
>> 

>> 
>> **Result:** They became the project's strongest advocates. They even discovered data quality issues in the source systems they hadn't known about. This experience reinforced that **showing is more powerful than telling** when influencing clients ."
> 
> * * *
> 
> ## Q21. How Do You Handle a Situation Where a Client's Business Team Wants a Feature That You Know Will Create Technical Debt?
> 
> **What they're assessing:** Your ability to balance business needs with architectural integrity .
> 
> **How to structure your answer:**
> 
>   1. **Acknowledge the Business Need:** Show you understand why they want it
>   2. **Explain the Technical Impact:** Clearly articulate the long-term consequences
>   3. **Propose Alternatives:** Offer solutions that meet their needs with less debt
>   4. **Document Trade-offs:** If they still choose the path, document the decision
> 

>
>> **Sample Answer:**  
>  "This happens frequently in consulting. My approach is to be a **partner, not a gatekeeper** :
>> 
>> **1\. Understand the 'Why'** — First, I dig into what business problem they're trying to solve. Often, there's a legitimate urgent need driving the request.
>> 
>> **2\. Explain Consequences in Business Terms** — I don't say 'this will create technical debt.' Instead, I explain: 'This approach will make future changes slower and more expensive. Here's what that means for your roadmap.'
>> 
>> **3\. Propose Alternatives** — I try to find a middle ground. For example: 'We can build a quick solution now, but let's design it so we can replace it later without breaking everything.' Or 'What if we deliver 80% of the functionality now with a cleaner architecture?'
>> 
>> **4\. Document Everything** — If they still choose the debt-creating path, I document:
>> 
>>   * The business decision
>>   * The technical implications
>>   * A plan to remediate later
>> 

>> 
>> **Real Example:** At Refinitiv, the business wanted a real-time dashboard urgently. The clean architecture would take 3 months; a quick solution could be ready in 2 weeks. We agreed to build the quick solution but designed it with an abstraction layer so we could replace the backend later without rewriting the frontend. Six months later, we did exactly that ."
> 
> * * *
> 
> # Category 3: PYSPARK PERFORMANCE DEEP-DIVE
> 
> ## Q22. Explain Lazy Evaluation in PySpark and How It Impacts Performance. Provide an Example.
> 
> **What they're assessing:** Your fundamental understanding of Spark's core optimization mechanism .
> 
> **How to structure your answer:**
> 
>   1. **Define Lazy Evaluation:** Transformations build a DAG; execution happens only on actions
>   2. **Explain the Benefit:** Allows Catalyst Optimizer to create efficient execution plans
>   3. **Give an Example:** Show code demonstrating the concept
>   4. **Connect to Your Experience:** Reference how you've leveraged this
> 

>
>> **Sample Answer:**  
>  "**Lazy evaluation** is Spark's most important performance feature. It means transformations like `.filter()`, `.select()`, and `.join()` are not executed immediately. Instead, Spark builds a **DAG (Directed Acyclic Graph)** of transformations and only executes when an **action** like `.count()`, `.show()`, or `.write()` is called .
>> 
>> **Why this matters:** Lazy evaluation allows Spark's **Catalyst Optimizer** to analyze the entire query plan and optimize it. For example:
>>     
>>     
>>     # These transformations just build the plan
>>     df_filtered = df.filter(col("amount") > 1000)
>>     df_selected = df_filtered.select("customer_id", "amount")
>>     df_grouped = df_selected.groupBy("customer_id").avg("amount")
>>     
>>     # This action triggers execution with an optimized plan
>>     result = df_grouped.collect()
>> 
>> Without lazy evaluation, each transformation would materialize intermediate data, causing massive I/O. With lazy evaluation, Spark might push the filter down to the data source or reorder operations to minimize shuffles.
>> 
>> In my **AXA project** , we processed millions of claims daily. Understanding lazy evaluation helped us design transformations that allowed Spark to optimize the physical plan, reducing processing time by 40% ."
> 
> * * *
> 
> ## Q23. What Is Data Skew and How Do You Handle It in PySpark?
> 
> **What they're assessing:** Your ability to diagnose and fix real-world performance problems .
> 
> **How to structure your answer:**
> 
>   1. **Define Data Skew:** Uneven data distribution causing some tasks to run much longer than others
>   2. **Explain How to Identify:** Using Spark UI (stage timeline, task duration)
>   3. **Provide Solutions:** Salting, broadcast joins, increasing partitions
>   4. **Give a Real Example:** Reference your experience
> 

>
>> **Sample Answer:**  
>  "**Data skew** occurs when data is unevenly distributed across partitions, causing a few 'straggler' tasks to take much longer than others. This is a common performance killer in joins and groupBy operations .
>> 
>> **How to identify it:** In the **Spark UI** , I look at the stage timeline. If most tasks complete quickly but a few take much longer, that's data skew. I also check task duration metrics—if max duration is 10x the median, skew is likely.
>> 
>> **Solutions I've used:**
>> 
>> **1\. Salting for Joins:**
>>     
>>     
>>     # Add a random salt to the skewed key
>>     from pyspark.sql.functions import rand, lit, concat
>>     
>>     skewed_df = skewed_df.withColumn("salt", (rand() * 10).cast("int"))
>>     skewed_df = skewed_df.withColumn("salted_key", concat(col("join_key"), lit("_"), col("salt")))
>>     
>>     # Create salt copies in the small table
>>     small_df = small_df.crossJoin(spark.range(10).toDF("salt"))
>>     small_df = small_df.withColumn("salted_key", concat(col("join_key"), lit("_"), col("salt")))
>>     
>>     # Join on salted key
>>     result = skewed_df.join(small_df, "salted_key")
>> 
>> **2\. Broadcast Join for Small Tables:** If one table is small (<100MB), I force a broadcast join to avoid shuffling the large table entirely.
>> 
>> **3\. Increasing Partitions:** For groupBy skew, I sometimes increase the number of partitions using `.repartition()` to distribute the load.
>> 
>> **Real Example:** At Refinitiv, we had a join where a few trading symbols had millions of transactions while most had thousands. We used salting to distribute those heavy symbols across multiple partitions, reducing job time from 45 minutes to 12 minutes ."
> 
> * * *
> 
> ## Q24. What Is the Difference Between `repartition()` and `coalesce()`? When Would You Use Each?
> 
> **What they're assessing:** Your understanding of partition management and its performance implications .
> 
> **How to structure your answer:**
> 
>   1. **Define Both:** Explain what each function does
>   2. **Highlight the Key Difference:** `repartition()` shuffles data; `coalesce()` minimizes shuffling
>   3. **Provide Use Cases:** Give clear scenarios for each
>   4. **Performance Considerations:** Explain when to use which
> 

>
>> **Sample Answer:**  
>  "Both functions manage the number of partitions, but they work very differently:
>> 
>> **`repartition(num_partitions)`** performs a **full shuffle** of data across the cluster to create a new, even distribution. It can **increase or decrease** the number of partitions. I use `repartition()` when I need to:
>> 
>>   * Increase parallelism (e.g., after heavy filtering that left many partitions empty)
>>   * Ensure even distribution before a wide transformation like a join
>>   * Change the number of output files
>> 

>> 
>> **`coalesce(num_partitions)`** only **reduces** the number of partitions and tries to merge existing partitions **without a full shuffle**. It's more efficient but can only decrease partitions. I use `coalesce()` when:
>> 
>>   * I need to reduce partitions after filtering
>>   * I'm writing to a sink that performs better with fewer files
>>   * Performance is critical and I want to avoid a shuffle
>> 

>> 
>> **Example from my experience:**
>>     
>>     
>>     # After filtering, many partitions may be nearly empty
>>     filtered_df = large_df.filter(col("status") == "active")  # 10% of data remains
>>     
>>     # Better: coalesce to reduce partitions without shuffle
>>     optimized_df = filtered_df.coalesce(50)  # merges partitions locally
>>     
>>     # Avoid: repartition would shuffle all data unnecessarily
>>     # bad_df = filtered_df.repartition(50)  # full shuffle - slower!
>> 
>> At AXA, we used `coalesce()` extensively after filtering to optimize write performance to OneLake ."
> 
> * * *
> 
> ## Q25. How Do You Debug a Failing PySpark Job? Walk Me Through Your Process.
> 
> **What they're assessing:** Your troubleshooting methodology and practical experience .
> 
> **How to structure your answer:**
> 
>   1. **Check Error Logs:** Start with the error message and stack trace
>   2. **Use Spark UI:** Examine stages, tasks, and executor logs
>   3. **Isolate the Problem:** Run with sample data to reproduce
>   4. **Check Data Quality:** Look for nulls, unexpected values, schema mismatches
>   5. **Incrementally Test:** Run step-by-step to identify failing transformation
> 

>
>> **Sample Answer:**  
>  "Debugging PySpark jobs requires a systematic approach. Here's my process:
>> 
>> **1\. Read the Error Log** — The error message often tells you exactly what failed. Is it an OOM error? A null pointer? A schema mismatch?
>> 
>> **2\. Examine the Spark UI** — This is invaluable. I look at:
>> 
>>   * **Stage timeline** — which stage failed?
>>   * **Task metrics** — were there data skew issues?
>>   * **Executor logs** — detailed error messages from the failing executor
>> 

>> 
>> **3\. Isolate the Problem** — I try to reproduce the error with a smaller sample:
>>     
>>     
>>        # Take a sample that preserves the problematic pattern
>>        sample_df = df.sample(0.01, seed=42)
>>        # Run the pipeline on sample
>> 
>> **4\. Check Data Quality** — Often, the issue is unexpected data:
>>     
>>     
>>        # Check for nulls in key columns
>>        df.filter(col("join_key").isNull()).count()
>>     
>>        # Check data types
>>        df.printSchema()
>>     
>>        # Check for unexpected values
>>        df.groupBy("status").count().show()
>> 
>> **5\. Incremental Testing** — I comment out transformations and add them back one by one to identify exactly which step fails.
>> 
>> **6\. Use Logging** — I add logging at key points:
>>     
>>     
>>        import logging
>>        logging.info(f"After filtering: {df.count()} records")
>> 
>> **Real Example:** At Refinitiv, a job was failing with a cryptic error. Using the Spark UI, I traced it to a single task that was processing a partition with corrupt data. The fix was adding data validation before the failing transformation ."
> 
> * * *
> 
> # Category 4: PURVIEW & DATA GOVERNANCE SCENARIOS
> 
> ## Q26. What Is Microsoft Purview and How Does It Support Data Governance in a Microsoft Fabric Environment?
> 
> **What they're assessing:** Your understanding of Purview's role in the broader governance landscape .
> 
> **How to structure your answer:**
> 
>   1. **Define Purview:** Unified data governance service for on-premises, multi-cloud, and SaaS
>   2. **Explain Key Capabilities:** Data map, data catalog, data lineage, data classification
>   3. **Connect to Fabric:** How Purview integrates with Fabric for governance
>   4. **Reference Your Experience:** Mention your governance work
> 

>
>> **Sample Answer:**  
>  "**Microsoft Purview** is a unified data governance service that helps organizations discover, catalog, and govern their data assets across on-premises, multi-cloud, and SaaS environments .
>> 
>> **Key capabilities relevant to Fabric:**
>> 
>> **1\. Data Map** — Purview creates an automated, up-to-date map of your entire data estate, including Fabric items like Lakehouses, Warehouses, and Power BI datasets.
>> 
>> **2\. Data Catalog** — It creates a business-friendly catalog where users can search for and discover data assets. At JNJ, this kind of catalog would have helped analysts find trusted data faster.
>> 
>> **3\. Data Classification** — Purview can automatically scan data and apply **sensitivity labels** based on content (e.g., detecting PII in claims data) .
>> 
>> **4\. Data Lineage** — It captures lineage across the data lifecycle, showing how data moves from source to consumption. This is critical for impact analysis and compliance .
>> 
>> **In the AXA project** , we integrated Fabric with Purview to:
>> 
>>   * Automatically discover and classify sensitive claims data
>>   * Track lineage from raw ingestion through Bronze-Silver-Gold to Power BI reports
>>   * Ensure compliance with healthcare regulations
>> 

>> 
>> This gave the governance team visibility and control while allowing analysts to work freely with trusted data ."
> 
> * * *
> 
> ## Q27. Explain How Data Lineage Works in Microsoft Purview and Why It Matters for Compliance.
> 
> **What they're assessing:** Your understanding of lineage as a governance capability .
> 
> **How to structure your answer:**
> 
>   1. **Define Data Lineage:** The ability to track data from its origin through transformations to consumption
>   2. **Explain How Purview Captures Lineage:** Automated collection from data systems
>   3. **Explain Compliance Value:** Impact analysis, audit trails, regulatory requirements
>   4. **Give a Real Example:** Reference your experience
> 

>
>> **Sample Answer:**  
>  "**Data lineage** is the ability to track data from its source, through all transformations, to its final consumption in reports and models .
>> 
>> **How Purview captures lineage:**
>> 
>>   * It integrates with data processing systems like Fabric Pipelines, Dataflows, and Notebooks
>>   * When a Pipeline reads from a Lakehouse and writes to a Warehouse, Purview captures that relationship
>>   * It builds a **end-to-end lineage graph** showing dependencies
>> 

>> 
>> **Why it matters for compliance:**
>> 
>> **1\. Impact Analysis** — If a source system changes, lineage shows exactly which downstream reports and models will be affected. This prevents surprises.
>> 
>> **2\. Audit Trails** — Regulators often require proof of data provenance. For healthcare claims at AXA, we needed to show that reported numbers could be traced back to source transactions .
>> 
>> **3\. Root Cause Analysis** — When a report has incorrect numbers, lineage helps trace back to find where the error originated.
>> 
>> **4\. Data Quality** — Lineage helps identify which transformations might be introducing quality issues.
>> 
>> **Real Example:** At Travelers, we didn't have automated lineage initially. When an actuarial report had a discrepancy, it took days to trace manually. With Purview, that same investigation would take minutes. For my next project, I'd implement Purview lineage from day one ."
> 
> * * *
> 
> ## Q28. What Are Sensitivity Labels and How Do You Implement Them in Fabric Using Purview?
> 
> **What they're assessing:** Your understanding of data classification and protection .
> 
> **How to structure your answer:**
> 
>   1. **Define Sensitivity Labels:** Metadata tags indicating data sensitivity (Public, Confidential, Highly Restricted)
>   2. **Explain Automatic vs. Manual Labeling:** How labels can be applied
>   3. **Explain Protection Mechanisms:** How labels enforce security
>   4. **Connect to Fabric/Purview:** How the integration works
> 

>
>> **Sample Answer:**  
>  "**Sensitivity labels** are metadata tags that classify data based on its business impact, confidentiality, or regulatory requirements—for example, 'Public,' 'Internal,' 'Confidential,' 'Highly Restricted' .
>> 
>> **In Microsoft Purview, labels can be applied:**
>> 
>> **Automatically** — Purview can scan data and apply labels based on content. If it detects credit card numbers or national IDs, it can automatically apply a 'Confidential' or 'Highly Restricted' label.
>> 
>> **Manually** — Data owners can also apply labels manually in the Fabric portal or through APIs.
>> 
>> **In Fabric, these labels:**
>> 
>>   * Travel with the data across Lakehouses, Warehouses, and Power BI
>>   * Enforce protective actions (e.g., preventing 'Highly Restricted' data from being shared externally)
>>   * Are visible in Power BI, warning users about sensitive data
>> 

>> 
>> **Implementation in Fabric:**
>> 
>>   1. **Configure sensitivity labels** in Microsoft 365 Compliance Center
>>   2. **Enable Purview integration** in Fabric tenant settings
>>   3. **Set up automatic labeling rules** based on sensitive information types
>>   4. **Monitor label usage** in Purview insights dashboard
>> 

>> 
>> At **AXA** , we used sensitivity labels to ensure that claims data containing personal health information was properly protected. When analysts queried the data, labels would appear in Power BI, reminding them of handling requirements ."
> 
> * * *
> 
> ## Q29. How Would You Design a Data Governance Framework for a Client Using Microsoft Purview and Fabric?
> 
> **What they're assessing:** Your ability to architect a complete governance solution .
> 
> **How to structure your answer:**
> 
>   1. **Start with Governance Principles:** Define goals (compliance, security, discoverability)
>   2. **Map Capabilities to Tools:** Show how Purview and Fabric address each goal
>   3. **Define Roles and Responsibilities:** Who does what?
>   4. **Outline Implementation Phases:** Practical rollout approach
>   5. **Connect to Your Experience:** Reference past governance work
> 

>
>> **Sample Answer:**  
>  "I would design a governance framework in four layers:
>> 
>> **Layer 1: Foundation (People & Processes)**
>> 
>>   * **Define governance goals** with stakeholders: compliance requirements (GDPR, HIPAA), security needs, data quality targets
>>   * **Establish roles:** Data Owners, Data Stewards, Data Consumers
>>   * **Create policies:** data classification standards, retention rules, access control policies
>> 

>> 
>> **Layer 2: Data Discovery & Catalog (Purview Data Map)**
>> 
>>   * **Register data sources:** all Fabric items (Lakehouses, Warehouses) plus any external sources
>>   * **Set up automated scanning** to keep the data map current
>>   * **Create business glossary** with standard definitions (e.g., what is a 'customer'?)
>> 

>> 
>> **Layer 3: Classification & Protection (Purview & Fabric)**
>> 
>>   * **Define sensitivity labels** aligned with data policies
>>   * **Configure automatic labeling** rules to detect sensitive data
>>   * **Implement access controls** in Fabric based on labels (e.g., only certain users can access 'Highly Restricted' data)
>> 

>> 
>> **Layer 4: Monitoring & Compliance (Purview Insights)**
>> 
>>   * **Track data lineage** for critical reports to enable impact analysis
>>   * **Monitor label usage** and access patterns
>>   * **Generate compliance reports** for auditors
>> 

>> 
>> **Implementation Approach:**
>> 
>>   * **Phase 1 (Quick Wins):** Set up Purview scanning, implement sensitivity labels for obvious PII
>>   * **Phase 2 (Expand):** Add business glossary, train data stewards
>>   * **Phase 3 (Advanced):** Implement automated policy enforcement, lineage tracking
>> 

>> 
>> At **JNJ** , we built a similar framework manually. With Purview and Fabric, I could implement this in weeks instead of months ."
> 
> * * *
> 
> # Category 5: CASE STUDY / DESIGN PROBLEMS
> 
> ## Q30. Design a Real-Time Fraud Detection System for a Credit Card Company Using Microsoft Fabric.
> 
> **What they're assessing:** Your end-to-end architecture design skills .
> 
> **How to structure your answer:**
> 
>   1. **Understand Requirements:** What are the goals (latency, scale, accuracy)?
>   2. **High-Level Architecture:** Diagram in words
>   3. **Component Breakdown:** Explain each piece
>   4. **Data Flow:** Walk through how data moves
>   5. **Governance & Security:** How do you protect sensitive data?
> 

>
>> **Sample Answer:**  
>  "I would design this using Microsoft Fabric's real-time capabilities:
>> 
>> **Requirements:**
>> 
>>   * Sub-second to seconds latency for fraud detection
>>   * Ability to process millions of transactions daily
>>   * Machine learning model scoring in real-time
>>   * Historical data for model training
>>   * Strict security for sensitive payment data
>> 

>> 
>> **High-Level Architecture:**
>>     
>>     
>>     Transaction Stream → Eventstream → Real-Time Hub → KQL Database → Alert/Action
>>                                              ↓
>>                                         Shortcut to OneLake
>>                                              ↓
>>                                       Lakehouse (Bronze/Silver/Gold)
>>                                              ↓
>>                                         Power BI (Dashboards)
>> 
>> **Component Breakdown:**
>> 
>> **1\. Ingestion (Eventstream)** — Transactions stream in via Kafka or Event Hubs. Fabric Eventstream captures them with millisecond latency .
>> 
>> **2\. Real-Time Processing (KQL Database)** — The eventstream routes data to a Kusto Query Language (KQL) database for real-time querying. Fraud detection rules run here, flagging suspicious transactions immediately.
>> 
>> **3\. Persistent Storage (OneLake via Shortcut)** — A **shortcut** from the KQL database to OneLake makes the data available for historical analysis without duplication .
>> 
>> **4\. Batch Processing (Lakehouse with PySpark)** — The raw transaction data lands in Bronze layer. PySpark notebooks transform it to Silver (cleaned, enriched) and Gold (aggregated for ML training).
>> 
>> **5\. Machine Learning** — Historical data from Gold layer trains fraud detection models. These models can be deployed for real-time scoring via the KQL database.
>> 
>> **6\. Visualization (Power BI with Direct Lake)** — Dashboards show fraud trends, alert volumes, and model performance using Direct Lake for fast queries on large datasets.
>> 
>> **7\. Governance (Purview)** — All data is scanned and classified. Payment card data gets 'Highly Restricted' labels with access controls. Lineage tracks data from raw transaction to fraud report .
>> 
>> **Why Fabric:** This unified platform handles streaming, batch, and BI in one environment—reducing complexity and latency. The same pattern could be adapted from my AXA experience, but for fraud instead of claims ."
> 
> * * *
> 
> ## Q31. A Client Wants to Migrate Their On-Premise Data Warehouse to Microsoft Fabric. How Would You Approach This?
> 
> **What they're assessing:** Your migration methodology, leveraging your Affinion experience .
> 
> **How to structure your answer:**
> 
>   1. **Assessment Phase:** Understand current state
>   2. **Strategy Phase:** Choose migration approach
>   3. **Design Phase:** Design target architecture
>   4. **Execution Phase:** Phased migration with validation
>   5. **Governance Phase:** Establish new governance
> 

>
>> **Sample Answer:**  
>  "Based on my Affinion migration experience (20M+ records to Azure), here's my approach:
>> 
>> **Phase 1: Assessment (4-6 weeks)**
>> 
>>   * **Data profiling** — Analyze source data quality, identify orphan records, document issues (like we did at Affinion)
>>   * **Dependency mapping** — Understand ETL jobs, reports, and applications dependent on the warehouse
>>   * **Requirements gathering** — Work with business to understand needs: latency, SLAs, new capabilities they want
>> 

>> 
>> **Phase 2: Strategy & Architecture Design**
>> 
>>   * **Choose migration pattern:**
>>   * **Rehost** for simple tables (lift-and-shift to Fabric Warehouse)
>>   * **Refactor** for complex transformations (rebuild as Lakehouse with PySpark)
>>   * **Retire** for unused data
>>   * **Design target architecture** in Fabric:
>>   * Bronze layer for raw migrated data
>>   * Silver layer for cleansed, integrated data
>>   * Gold layer for business-facing models
>> 

>> 
>> **Phase 3: Development & Testing**
>> 
>>   * **Build migration pipelines** using Fabric Dataflows Gen2 and Pipelines
>>   * **Create validation framework** with reconciliation scripts (as I did at Affinion)
>>   * **Run parallel tests** comparing source and target results
>>   * **Performance test** to ensure SLAs are met
>> 

>> 
>> **Phase 4: Phased Migration**
>> 
>>   * **Migrate by data domain** (customer → product → transactions)
>>   * **Each phase includes:**
>>   * Full reconciliation
>>   * Business user validation
>>   * Rollback plan if issues arise
>> 

>> 
>> **Phase 5: Governance & Optimization**
>> 
>>   * **Set up Purview** for data catalog, lineage, and classification
>>   * **Establish monitoring** in Fabric
>>   * **Optimize** based on usage patterns
>> 

>> 
>> **Key Success Factors:**
>> 
>>   * **Don't trust source data** — always profile first (lesson from Affinion)
>>   * **Involve business early** — they need to validate results
>>   * **Automate validation** — reconciliation scripts catch issues immediately
>> 

>> 
>> This structured approach ensures 100% data fidelity while delivering the benefits of modern Fabric architecture ."
> 
> * * *
> 
> ## 📚 QUICK REFERENCE: Key Frameworks to Remember
> 
> Scenario| Framework to Use| Your Experience to Reference  
> ---|---|---  
> **Fabric Troubleshooting**|  Check: Resource → Code → Dependencies → Capacity| AXA project  
> **Client Influence**|  Listen → Prototype → Demonstrate → Involve| Travelers (actuary story)  
> **Data Skew**|  Identify → Diagnose (Spark UI) → Apply solution (salting/broadcast)| Refinitiv  
> **Purview Governance**|  Map → Classify → Protect → Monitor| JNJ governance work  
> **Migration**|  Assess → Design → Build → Validate → Migrate| Affinion (20M records)  
>   
> * * *
> 
> ## 🎯 Final Interview Tips for EY Senior Manager
> 
> Tip| Why  
> ---|---  
> **Tell stories, don't just list facts**|  Use STAR method (Situation, Task, Action, Result) for behavioral questions  
> **Connect everything to your projects**|  Every answer should reference AXA, JNJ, Refinitiv, Affinion, or Google  
> **Show consulting mindset**|  Talk about client relationships, business value, trade-offs  
> **Be honest about what you're learning**|  "I'm currently deepening my Fabric knowledge through hands-on practice" shows growth mindset  
> **Prepare questions for them**|  Ask about
