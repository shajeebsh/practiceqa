---
title: "Microsoft Fabric"
date: 2026-03-25 19:17:09 
author: shajeebhameed
layout: page
slug: data-analysis-msf
status: publish
---

# Interview Preparation: AIB Senior Data Analyst Role

* * *

## 📋 Interview Structure (Typical for AIB)

Stage| Focus| Duration  
---|---|---  
**Initial Screen**|  HR/Recruiter – eligibility, motivation, salary| 15-20 mins  
**Technical Interview**|  SQL, Python, Fabric, data analysis| 45-60 mins  
**Stakeholder/Case Study**|  Business problem solving, presentation| 45-60 mins  
**Final / Leadership**|  Cultural fit, career aspirations| 30 mins  
  
* * *

# CATEGORY 1: MOTIVATION & FIT

## Q1. Why are you interested in this Senior Data Analyst role at AIB?

**Sample Answer:**

> "I've spent the last 11 years working in data analytics across different industries, but I've always been drawn to roles where data is used to make informed, responsible decisions. AIB's focus on building robust, well-governed data foundations really resonates with me—especially given the regulated environment you operate in.
> 
> Most recently, I worked with AXA Insurance on a Microsoft Fabric migration, where I was responsible for analysing data quality, building monitoring frameworks, and creating dashboards that gave their teams real-time visibility into claims. That experience—combining technical work with business impact—is exactly what I'm looking for in my next role.
> 
> I also appreciate that this role sits at the intersection of data, platforms, and governance. It's not just about building reports—it's about ensuring the underlying data is trustworthy, well-governed, and scalable. That's where I've spent most of my career, and I'd love to bring that experience to AIB."

* * *

## Q2. What do you know about AIB and the BI Infrastructure team?

**What they're assessing:** Your research and genuine interest.

**Sample Answer:**

> "AIB has been on a significant digital transformation journey over the past few years, with a strong focus on modernising data platforms and enabling better decision-making across the business. The BI Infrastructure team, as I understand it, is responsible for the underlying platforms that support enterprise reporting and analytics—things like data warehouses, semantic models, and governance frameworks.
> 
> What stands out to me is the emphasis on building scalable, well-governed foundations. That's exactly the kind of work I've been doing—whether it's consolidating legacy data sources at J&J, building lakehouse architectures at Travelers, or implementing data quality frameworks at AXA. I'm excited about the opportunity to contribute to that work."

* * *

# CATEGORY 2: TECHNICAL – SQL & PYTHON

## Q3. Walk me through a complex SQL query you've written recently.

**Sample Answer:**

> "At J&J, I was working with claims data that spanned multiple tables—policyholders, claims, payments, and providers. I needed to calculate the total paid amount per policyholder for the last 12 months, but only for claims that were approved and where the provider was in-network.
> 
> The query looked something like this:
>     
>     
>     SELECT 
>         ph.policyholder_id,
>         ph.name,
>         SUM(c.claim_amount) AS total_paid,
>         COUNT(c.claim_id) AS claim_count
>     FROM policyholders ph
>     INNER JOIN claims c ON ph.policyholder_id = c.policyholder_id
>     INNER JOIN providers p ON c.provider_id = p.provider_id
>     WHERE c.status = 'approved'
>       AND p.network_status = 'in-network'
>       AND c.claim_date >= DATEADD(month, -12, GETDATE())
>     GROUP BY ph.policyholder_id, ph.name
>     HAVING SUM(c.claim_amount) > 10000
>     ORDER BY total_paid DESC;
> 
> The challenge was making sure it performed well—the claims table had millions of rows. I ended up adding indexes on `policyholder_id`, `claim_date`, and `status`, and used execution plans to confirm the joins were efficient. It ended up running in under 2 seconds, which was critical for the dashboard the team was building."

* * *

## Q4. How do you approach performance tuning in SQL?

**Sample Answer:**

> "I take a systematic approach:
> 
> **1\. Understand the query** – First, I look at the execution plan to see where the bottlenecks are. Is it a table scan? An expensive join? Missing indexes?
> 
> **2\. Check indexing** – I verify that columns used in WHERE, JOIN, and ORDER BY clauses are indexed appropriately. For large tables, I also consider covering indexes to avoid key lookups.
> 
> **3\. Optimise joins** – I make sure joins are on indexed columns and that the join order is optimal. For example, filtering the smaller table first can reduce the rows being joined.
> 
> **4\. Reduce data early** – I push filters as early as possible in the query. Subqueries or CTEs can help isolate data before joining.
> 
> **5\. Review statistics** – Sometimes the issue is stale statistics, which can lead to poor execution plans. I check when stats were last updated and refresh if needed.
> 
> **6\. Consider schema changes** – If the query is still slow, I look at whether partitioning, indexing strategies, or even denormalisation might help.
> 
> At Refinitiv, we had a query that was taking 45 seconds. After analysing the execution plan, I noticed a missing index on a timestamp column used in a WHERE clause. Adding that index dropped execution time to under 2 seconds."

* * *

## Q5. Tell me about a time you used Python to solve a data problem.

**Sample Answer:**

> "At Google Street View, we had a manual reporting process that was taking up over 100 hours a month. The research and compliance teams were pulling data from multiple sources—BigQuery, APIs, and spreadsheets—and manually combining them to produce weekly reports.
> 
> I built a Python script using Pandas to automate the whole process. It would:
> 
>   * Extract data from BigQuery using the Google Cloud client library
>   * Pull defect data from the Bug Tracking API
>   * Combine and transform the data using Pandas (joins, aggregations, calculated fields)
>   * Generate the report in Excel format
>   * Send it via email using an internal API
> 

> 
> The script cut the manual effort down to zero—the reports just ran on a schedule. That saved over 100 hours a month and also reduced errors from manual copying.
> 
> The key was making it reliable. I added error handling, logging, and alerts if any step failed. That way, if something broke, the team knew immediately."

* * *

# CATEGORY 3: TECHNICAL – MICROSOFT FABRIC

## Q6. Tell me about your experience with Microsoft Fabric.

**Sample Answer:**

> "I worked with Microsoft Fabric recently on a migration project for AXA Insurance. We were moving their claims data from an on-premise system to the cloud, and Fabric was the platform we used.
> 
> My role was focused on data analysis and quality. I was responsible for:
> 
>   * Analysing the legacy claims data to identify quality issues and patterns that would impact the migration
>   * Building data quality monitoring frameworks using Fabric notebooks (PySpark) to catch anomalies in real-time
>   * Creating Power BI dashboards that used Direct Lake mode, which gave the actuarial and fraud teams sub-second query performance
>   * Implementing governance tracking with Microsoft Purview so we had full lineage visibility for audit purposes
> 

> 
> The outcome was significant: we reduced data latency from 24 hours to under 15 minutes, and cut manual review work by 40%.
> 
> What I appreciated about Fabric is how integrated everything is. The same platform handled the ingestion, transformation, and reporting—which meant less moving parts to manage."

* * *

## Q7. What is Direct Lake mode in Power BI and why is it useful?

**Sample Answer:**

> "Direct Lake is a new mode in Power BI that's specific to Fabric. It sits between Import and DirectQuery.
> 
> With Import mode, data is loaded into memory—it's fast, but you have to refresh it. With DirectQuery, queries go back to the source—data is live, but can be slower.
> 
> Direct Lake reads the Delta tables directly from OneLake, without needing to import or run a query. You get the performance of Import mode with the data freshness of DirectQuery. It's a game-changer for large datasets where you need real-time data but can't afford slow queries.
> 
> At AXA, we used Direct Lake for the dashboards the fraud team used. They were querying billions of rows of claims data, but with Direct Lake, they were getting responses in sub-seconds. It completely changed how they worked."

* * *

## Q8. How have you used Microsoft Purview for data governance?

**Sample Answer:**

> "At AXA, we used Purview to establish data lineage and governance for the claims data. One of the key requirements was being able to trace any report back to the source data—especially important for audit and compliance.
> 
> We set up Purview to automatically scan the Fabric environment and capture lineage as data moved from ingestion through to Power BI. That meant if an analyst saw a number in a dashboard that didn't look right, we could trace it back to the original source and see exactly how it was transformed.
> 
> We also used sensitivity labels to classify data. Anything containing personal health information was automatically tagged as 'Confidential', and access was restricted accordingly.
> 
> It gave the governance team confidence that we had visibility into what data was where, and how it was being used."

* * *

# CATEGORY 4: DATA GOVERNANCE & QUALITY

## Q9. What's your approach to data quality in a project?

**Sample Answer:**

> "I take a proactive approach to data quality—I don't wait for issues to surface.
> 
> **1\. Discovery phase:** Before any project starts, I profile the data. At Affinion, we found orphan records and inconsistent formats before we even started writing migration code. That upfront work saved us months of headaches.
> 
> **2\. Define quality rules:** I work with business stakeholders to define what 'good' data looks like. At J&J, I documented 200+ business attributes with their transformation rules and validation logic.
> 
> **3\. Build monitoring:** I implement automated checks that run as part of the pipeline. At Refinitiv, we had 50+ checks that ran continuously, flagging any anomalies in real-time. That helped us achieve 99.9% data consistency.
> 
> **4\. Alert and escalate:** If a quality check fails, I set up alerts so the right people know immediately. For critical issues, we had automated escalation to the operations team.
> 
> **5\. Root cause analysis:** When issues do happen, I document what went wrong and put measures in place to prevent recurrence."

* * *

## Q10. How do you ensure data is compliant with regulations like GDPR?

**Sample Answer:**

> "I've worked in regulated environments like healthcare and finance, so I'm very conscious of compliance requirements.
> 
> My approach has a few layers:
> 
> **First, data classification.** I work with the business to identify what data is sensitive—PII, health information, financial data. At AXA, we used Purview to automatically tag sensitive data.
> 
> **Second, access controls.** Not everyone needs access to everything. At Travelers, we implemented RBAC so that claims adjusters could only see their own cases, while actuaries had broader access.
> 
> **Third, audit trails.** I ensure there's full lineage from source to report. If an auditor asks where a number came from, I want to be able to show them.
> 
> **Fourth, data minimisation.** I'm careful not to bring in data that isn't needed. At J&J, we documented exactly which attributes were required for each use case.
> 
> **Fifth, retention.** I work with the business to define how long data should be kept, and build that into the pipeline—so data is automatically archived or deleted when it reaches the end of its retention period."

* * *

# CATEGORY 5: STAKEHOLDER ENGAGEMENT & CONSULTING

## Q11. Tell me about a time you had to explain complex technical findings to a non-technical stakeholder.

**Sample Answer:**

> "At J&J, I was doing a deep-dive analysis of their employee grievance data. I found that 40% of cases were basic password resets and HR queries that could be handled by a self-service portal—which meant significant cost savings if we automated them.
> 
> The challenge was presenting this to the executive team, who weren't technical. Instead of showing them SQL queries or data models, I built a simple Power BI dashboard that showed:
> 
>   * The volume of cases by type
>   * The cost per case
>   * The potential savings from automation
> 

> 
> I walked them through it visually, focusing on the business impact rather than the technical details. I also made sure to address their concerns upfront—they were worried about the implementation cost, so I included a breakdown of the investment needed and the payback period.
> 
> They ended up approving the project. The key was keeping it simple and tying everything back to business outcomes."

* * *

## Q12. How do you handle a situation where a stakeholder asks for something that isn't feasible?

**Sample Answer:**

> "I've run into this a few times. The key is not to just say 'no'—that creates frustration. Instead, I try to understand what they're actually trying to achieve and offer alternatives.
> 
> For example, at Refinitiv, a stakeholder wanted real-time data on every single trade, updated every second. Technically, it was possible, but the cost and complexity would have been huge.
> 
> Instead of saying 'no,' I asked what business decision they needed to make with that data. It turned out they needed to monitor for market anomalies, but they could do that with 5-minute aggregates—not second-by-second data.
> 
> So we built a solution that gave them the insights they needed, but at a fraction of the cost. They were happy because they got what they needed, and I was happy because it was a more sustainable solution.
> 
> I think the lesson is: listen first, understand the real need, and then propose alternatives that meet that need in a way that's technically sound."

* * *

# CATEGORY 6: SCENARIO-BASED

## Q13. You notice a critical dashboard is showing incorrect numbers. Walk me through how you'd handle it.

**Sample Answer:**

> "First, I'd acknowledge it and communicate. I'd let the stakeholders know that I'm aware of an issue and investigating—no one likes to discover a problem themselves.
> 
> Next, I'd trace the data lineage to find where the error is coming from. Is it at the source? A transformation step? The semantic model? The dashboard itself?
> 
> Once I identify the root cause, I'd assess the impact. How many reports are affected? Is it a data issue that affects everything, or a specific calculation?
> 
> Then I'd fix it. If it's a data issue, I'd need to decide whether to reprocess or correct forward. If it's a dashboard issue, I'd update and redeploy.
> 
> After the fix, I'd validate that everything is correct and communicate back to stakeholders that it's resolved.
> 
> Finally, I'd do a root cause analysis to understand why it happened and put measures in place to prevent recurrence. Was it missing data quality checks? A gap in the pipeline? A manual step that should be automated?
> 
> At Refinitiv, we had a similar issue where a data quality check failed. We ended up adding a validation step that flagged similar issues automatically in the future."

* * *

## Q14. A business user asks for a new dashboard. How do you approach gathering requirements?

**Sample Answer:**

> "I don't start by asking what they want on the dashboard—that's a recipe for building something they don't actually use.
> 
> Instead, I ask:
> 
>   * What business decision are you trying to make?
>   * What question are you trying to answer?
>   * How will you use this information?
>   * How often do you need to see it?
> 

> 
> For example, at AXA, a stakeholder asked for a dashboard showing 'all claims data.' That's too broad. I dug in and found they actually needed to monitor claims that had been open for more than 30 days, so they could prioritise follow-up.
> 
> Once I understand the real need, I sketch out a mock-up—nothing fancy, just on paper or in Excel. I show it to them and ask: 'Does this give you what you need?'
> 
> Then I build a prototype and get feedback. I'd rather get feedback early than spend weeks building something that misses the mark.
> 
> Finally, I document the requirements and get sign-off before building the final version."

* * *

# CATEGORY 7: YOUR CV & ACHIEVEMENTS

## Q15. Tell me about a project you're most proud of.

**Sample Answer:**

> "I'm particularly proud of the AXA Insurance project. It was a large-scale migration of claims data to Microsoft Fabric, and my focus was on data quality and analytics.
> 
> When we started, the legacy data was in a mess—inconsistent formats, missing values, no clear lineage. The fraud team was working with data that was 24 hours old, which meant they were often reacting after the fact.
> 
> I built data quality monitoring frameworks that caught issues in real-time. We reduced manual review work by 40% because the system was flagging anomalies automatically instead of waiting for someone to notice.
> 
> I also created Power BI dashboards using Direct Lake mode, which gave the fraud team sub-second query performance. They could now query billions of rows of claims data and get answers instantly.
> 
> But what I'm most proud of is the impact. We reduced data latency from 24 hours to under 15 minutes, which meant the fraud team could detect suspicious activity much faster. And the governance framework we put in place gave the compliance team confidence that everything was traceable for audit.
> 
> It was a challenging project, but seeing the team's workflows transform because of the work we did was incredibly satisfying."

* * *

## Q16. How did you identify $2M+ in cost savings at J&J?

**Sample Answer:**

> "At J&J, I was working with their employee grievance system. They had a large volume of cases coming through to support teams—things like password resets, HR queries, payroll issues.
> 
> I did a deep-dive analysis of the case data. I looked at case volumes by type, resolution times, repeat cases, and cost per case. What I found was:
> 
>   * 40% of cases were simple password resets and basic HR queries
>   * Procurement cases had a 30% repeat rate—same employees submitting multiple cases on the same issue
>   * Payroll cases spiked on specific days, but agents were idle the rest of the month
> 

> 
> I built a Power BI dashboard to visualise these patterns and calculated the potential savings if we addressed each area. For example, if we automated the simple cases through a self-service portal, we could reduce agent time by 40%.
> 
> I presented the findings to leadership, and they ended up approving a transformation program. The total estimated savings came to just over $2 million annually.
> 
> The key was making it visual and tying everything to business outcomes—not just showing them data, but showing them what that data meant for their bottom line."

* * *

## 🎯 Final Tips

Tip| Why It Matters  
---|---  
**Use the STAR method**|  Situation, Task, Action, Result—keeps answers structured  
**Bring a portfolio**|  Power BI dashboards, SQL scripts, or project documentation—shows rather than tells  
**Prepare questions for them**|  Shows genuine interest and engagement  
**Be honest about gaps**|  If you don't know something, say so—but show you're willing to learn  
**Connect to the role**|  Every answer should tie back to how it applies to AIB  
  
* * *
