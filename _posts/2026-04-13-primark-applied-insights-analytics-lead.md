---
title: "Primark -Analytics Lead"
date: 2026-04-13 15:51:41 
author: shajeebhameed
layout: page
slug: primark-applied-insights-analytics-lead
status: publish
---

## Primark – Applied Insights & Analytics Lead

### In-Depth Interview Preparation: Questions & Answers

### Tailored for Shajeeb S | Senior Data Analytics Specialist

* * *

> **How to use this guide:** Each section maps to a core competency from the Primark job spec. Read the question, understand what the interviewer is probing for (noted in _italics_), then deliver your answer using the STAR format where applicable — Situation, Task, Action, Result.

* * *

### SECTION 1: Leadership & Team Building

* * *

### Q1. This is a brand-new team you'll be building from scratch. How would you approach designing and hiring a high-performing applied insights team at Primark?

_Probing: Strategic thinking, team architecture, hiring philosophy, understanding of analytics maturity._

**Answer:**

I'd start by conducting a capability audit — understanding what analytical work currently exists, where it lives, and which business areas are most underserved. Before hiring anyone, I'd map the demand landscape: what decisions are being made manually that data should be informing?

From there, I'd design a team structure with a few core archetypes: BI developers who can build and maintain robust reporting infrastructure, analytics translators who can bridge commercial and technical conversations, and a workflow/data engineer to handle pipeline automation and data quality. In the early days, I'd prioritise versatile "T-shaped" analysts — broad generalists with depth in one area — because in a new team you can't afford specialists who can't flex.

On hiring, I look for intellectual curiosity above all else. Technical skills are trainable; the habit of asking "why does this number look like this?" is not. I'd also establish a structured onboarding that immerses new analysts in the business context — understanding Primark's trading calendar, seasonality, and the decisions store operations and merchandising make daily.

At Cognizant, when I consolidated four legacy data sources into a single BigQuery model serving 150+ analysts, one of the biggest lessons was that the technical architecture is only half the battle — building trust with those analysts and ensuring they understood and adopted the new framework required deliberate relationship-building. I'd bring that same intentionality to growing this team at Primark.

* * *

### Q2. How do you mentor and develop junior analysts? Give a specific example.

_Probing: People leadership, coaching style, culture of continuous learning._

**Answer:**

My approach to mentoring is to build confidence through visibility. Junior analysts often know more than they think they do — they just need someone to create space for them to demonstrate it.

Practically, I do three things: first, I involve them early in problem framing, not just execution. When they understand why a piece of analysis matters commercially, the quality of their work improves significantly. Second, I conduct structured code and dashboard reviews — not just fixing what's wrong, but explaining the reasoning behind data modelling choices, visual design decisions, and how we communicate uncertainty to non-technical stakeholders. Third, I give them ownership of at least one deliverable that they present directly to the business, with me in the room for support rather than control.

At AXA, I mentored analysts on the Power BI framework we were building, including teaching them how to use Microsoft Purview for data lineage tracking — something most of them hadn't used before. One analyst I worked with went from producing static Excel reports to owning a live Direct Lake dashboard consumed by the actuarial team. That shift in their confidence and capability was one of the most satisfying outcomes of that project.

* * *

### Q3. How do you handle underperformance in a data team? What steps do you take?

_Probing: Management courage, fairness, process orientation._

**Answer:**

I address it early, directly, and privately. Underperformance left unaddressed demoralises the rest of the team, who notice far sooner than managers typically do.

My process: First, I have a candid one-to-one conversation to understand root cause. In my experience, underperformance usually falls into one of three categories — skills gap, context gap (they don't understand what's expected or why), or motivation gap. Each has a different solution. I won't treat a skills gap as a motivation problem, because that's both unfair and ineffective.

I set clear, measurable improvement targets with a defined timeframe, provide the resources or coaching needed, and check in frequently. I document the conversations and outcomes. If performance doesn't improve despite genuine support, I involve HR and follow a structured process — but that's a last resort, not a first response.

The goal is always to retain talent where possible. Recruiting is expensive, and a good analyst who understands Primark's commercial context is a significant asset.

* * *

## SECTION 2: Analytics Strategy & Portfolio Management

* * *

### Q4. How would you prioritise an analytics portfolio when you have more demand than capacity?

_Probing: Strategic prioritisation, business value assessment, stakeholder management._

**Answer:**

Prioritisation is one of the most important — and most under-appreciated — skills in analytics leadership. Without a structured approach it degrades into whoever shouts loudest getting served first, which destroys team morale and business trust.

My framework has three components:

**Business value:** What is the commercial impact of the insight? Is it a decision that affects revenue, margin, store operations, or customer experience? What is the cost of not having it?

**Feasibility:** What data do we already have? What would need to be built? Is this a quick win or a six-month platform dependency?

**Strategic alignment:** Does this advance a priority on the company's roadmap, or is it tactical noise?

I use a simple 2x2 of business value vs. delivery effort to visualise the portfolio. High value, low effort = do now. High value, high effort = plan and resource properly. Low value, low effort = batch and automate. Low value, high effort = challenge the request.

Critically, I communicate this framework to stakeholders so they understand why priorities are set the way they are. Transparency here prevents resentment. At J&J, I managed a portfolio of analytical requests across finance, sales, and operations — applying this framework helped us redirect resource toward the analysis that ultimately identified $2M in cost savings, rather than getting buried in ad hoc reporting.

* * *

### Q5. What does a good KPI framework look like, and how do you design one?

_Probing: Analytical rigour, commercial understanding, ability to connect data to decisions._

**Answer:**

A good KPI framework is opinionated — it tells a story about what the business believes drives performance, not just what can be measured.

My design process:

  1. **Start with decisions, not metrics.** I ask business stakeholders: "What decision would you make differently if you had better data?" That surfaces the metrics that matter, rather than the metrics that are merely available.
  2. **Build a hierarchy.** There should be a small number (3–5) of outcome KPIs at the top — things like sales per square foot, stock turn rate, or customer conversion. Below those sit 8–12 driver metrics that explain movement in the outcomes. Below those are diagnostic indicators that help analysts investigate anomalies.
  3. **Define clearly.** Every KPI needs an unambiguous definition, an owner, a target, a data source, and a refresh cadence. Ambiguity in definitions is the single biggest cause of conflicting numbers across dashboards — which destroys trust in analytics.
  4. **Build in a benefits-realisation measure.** How will we know if the insight we delivered changed a decision or improved an outcome? This closes the loop and demonstrates the value of the analytics function.



At Google Street View I built a KPI framework for image quality and pipeline performance — tracking latency, defect rates, and resolution times. Having those metrics standardised meant that when I identified a 25% bottleneck in the ingestion pipeline, I could quantify the impact in business terms, not just technical ones.

* * *

### Q6. How do you balance quick wins with longer-term strategic capability building?

_Probing: Planning maturity, stakeholder management, avoiding short-termism._

**Answer:**

This tension is real and constant in analytics leadership. Lean too far toward quick wins and you end up with a dashboard factory with no scalable foundation. Lean too far toward long-term capability and you lose credibility with the business before the platform is ready.

My rule of thumb is 60/30/10: roughly 60% of capacity on delivering business value now (reports, dashboards, analyses that inform current decisions), 30% on building the data infrastructure and automation that makes future delivery faster and better, and 10% on exploratory work — proofs of concept, new data sources, emerging techniques.

I communicate this split explicitly to stakeholders. When I'm allocating time to platform work, I explain why in commercial terms: "We're spending six weeks building a reusable data model for sales performance because once it's done, every future request in this area will take days instead of weeks." That framing gets buy-in.

At AXA, the migration from legacy reporting to Microsoft Fabric and Power BI was exactly this kind of long-term investment. It required us to absorb short-term delivery pressure while building something that would dramatically accelerate everything downstream — and it did. The executive dashboards we built in Direct Lake mode delivered sub-second query performance for teams that had previously waited hours for refreshes.

* * *

## SECTION 3: Technical – BI & Visualisation

* * *

### Q7. Walk us through your approach to designing an executive dashboard. What makes one great vs. mediocre?

_Probing: BI craft, communication of data, understanding of senior stakeholder needs._

**Answer:**

A great executive dashboard answers the question the executive has before they realise they have it. A mediocre one presents everything available and makes the executive do the work.

My design principles:

**Lead with the conclusion.** The most important number or trend should be in the top-left and visible without scrolling. Executives should be able to read the headline story in under 10 seconds.

**Use visual hierarchy deliberately.** Primary KPIs large and prominent; supporting context smaller and secondary. If everything is the same size, nothing is important.

**Make anomalies surface themselves.** Conditional formatting, variance indicators, and trend arrows mean the executive's eye is immediately drawn to what needs attention — they shouldn't have to hunt for it.

**Minimise interactions for the standard view.** Filters and drill-downs should exist, but the default view should answer 80% of questions without any clicking.

**One truth.** Every number on the dashboard should trace back to a single governed data source. Nothing destroys executive trust faster than a KPI that differs between the dashboard and the spreadsheet.

At AXA, the Power BI dashboards I built for the actuarial and fraud detection teams were consumed daily by senior leaders who needed to make real-time reserving decisions. I spent as much time on information architecture and layout as I did on the data model, because a correct number poorly presented leads to bad decisions just as much as a wrong number does.

* * *

### Q8. The job spec mentions transitioning from Alteryx to Tableau/Power BI reporting frameworks. What does that transition involve and how would you manage it?

_Probing: Change management, technical depth in workflow automation, risk awareness._

**Answer:**

This kind of platform transition is as much a change management exercise as a technical one. The technical migration is usually the more straightforward part.

**Technical approach:** I'd start by cataloguing the full Alteryx workflow inventory — classifying each workflow by complexity, business criticality, and run frequency. I'd identify which workflows are pure data transformation (candidates for SQL or Python-based pipelines), which produce reports (candidates for Power BI or Tableau with a clean data model behind them), and which have complex branching logic that needs careful translation.

I'd build a migration factory: a repeatable pattern for converting common workflow archetypes, starting with the simplest and building complexity from there. Running Alteryx and the new solution in parallel during transition is non-negotiable — until output reconciliation is confirmed, the old system stays live.

**Change management:** I'd involve the analysts who use the Alteryx workflows early. They know the quirks, the undocumented logic, and the edge cases that will bite you if you ignore them. Making them co-owners of the migration rather than passive recipients of change dramatically increases adoption success.

**Risk management:** I'd maintain a risk register for business-critical workflows with explicit rollback procedures. No migration of a report used in trading or store operations goes live without sign-off from the business owner.

I led an analogous transition at AXA — moving from a legacy claims reporting system to Microsoft Fabric and Power BI. The principles were identical: catalogue, classify, run in parallel, validate, and bring the users with you.

* * *

### Q9. When would you choose Tableau over Power BI, and vice versa? What are the strengths of each?

_Probing: Technical breadth, tool-agnosticism, ability to recommend the right tool._

**Answer:**

Both are excellent tools and the right choice depends on the use case, the existing stack, and the audience.

**Power BI strengths:** Deep integration with the Microsoft ecosystem — if the organisation runs on Azure, Fabric, or SQL Server, Power BI is a natural fit with significantly reduced friction. The Direct Lake mode in Microsoft Fabric delivers exceptional performance for large datasets. The licencing model (included in many Microsoft 365 packages) often makes it more cost-effective at enterprise scale. DAX is powerful for complex calculations once learned.

**Tableau strengths:** Superior visual flexibility — Tableau's chart types and customisation options are more extensive, and it's generally considered the stronger tool for exploratory analysis and storytelling. It's also more tool-agnostic, connecting cleanly to a wider range of data sources without being tied to a single cloud vendor. Tableau Prep is excellent for visual data preparation. For organisations without a Microsoft-first stack, Tableau often wins.

**My recommendation framework:** If Primark is already heavily invested in Microsoft Azure and Fabric, Power BI is the logical backbone for operational reporting and executive dashboards. Tableau may be better suited for analytical workbenches, ad hoc exploration, or stakeholder-facing presentations where visual polish is paramount.

At Google, I used Looker Studio for the Google ecosystem. At AXA, Power BI with Direct Lake was the right call given the Microsoft Fabric environment. At Travelers, I used both Tableau and Power BI on the same project for different audience types — that flexibility is valuable.

* * *

### Q10. How do you ensure data quality and consistency across multiple dashboards and reports?

_Probing: Governance mindset, technical rigour, scalability thinking._

**Answer:**

Data quality in BI is 80% architecture and 20% process. If you build quality into the data model, you don't have to rely on individual analysts doing the right thing every time.

**Architecture:** A centralised semantic layer is essential. Whether that's a Power BI dataset, a Tableau data source, or a governed dbt model in the data warehouse — every report should pull from the same certified, tested layer, not directly from raw tables. This eliminates the "why does your number differ from mine?" conversation entirely.

**Automated monitoring:** I build data quality checks at the pipeline level — row count validations, null checks on critical fields, range checks on key metrics, and reconciliation tests against known totals. At AXA, I implemented this using Fabric notebooks with PySpark, cutting manual review effort by 40%.

**Governance framework:** Every critical metric needs a business owner and a clear definition published in a data catalogue. I use tools like Microsoft Purview or equivalent to provide data lineage — so when a number looks wrong, any analyst can trace it back to its source in minutes rather than hours.

**Certification process:** Not every dashboard should be treated as equal. I distinguish between certified/production dashboards (governed, tested, maintained) and exploratory/sandbox reports (clearly labelled as such). This prevents stakeholders from making decisions based on unvalidated analysis.

* * *

## SECTION 4: Retail Commercial Acumen

* * *

### Q11. Primark is a fast-fashion retailer. How would you approach building analytics capability for trading and merchandising teams?

_Probing: Retail domain knowledge, ability to translate commercial context into analytics solutions._

**Answer:**

Retail trading and merchandising analytics has a very specific rhythm, and understanding that rhythm is the prerequisite for building anything useful.

The weekly trading cycle drives everything. Merchants need to understand how current week sales are tracking versus last year and versus plan — ideally by Wednesday so they have time to react. That means the data pipeline and reporting cadence needs to be aligned to the trading week, not to a convenient technical schedule.

For trading, I'd focus on: like-for-like sales performance by category, store cluster, and geography; sell-through rates by SKU to identify fast and slow movers early; availability metrics to flag when stock constraints are limiting sales.

For merchandising, the priority is usually range performance analytics — which options are working, which should be dropped or reordered — plus intake vs. demand forecasting and markdown optimisation. These analyses require clean, consistent SKU-level data that connects sales, stock, and intake, which is often where the data quality work is most important.

I'd partner closely with the buying and merchandising teams from day one — sitting in on trading meetings before building anything, so I understand the questions they're actually asking and the decisions they're actually making. The risk with analytics teams that aren't embedded in the business is that they build impressive things that answer no one's real question.

My experience at Affinion Voyager migrating 20M+ retail customer records gave me direct exposure to retail data complexity — the messy reality of loyalty systems, omnichannel identifiers, and seasonal data spikes.

* * *

### Q12. How would you embed data-driven decision-making into teams that are currently relying on gut instinct and spreadsheets?

_Probing: Change management, commercial empathy, stakeholder influence._

**Answer:**

The first thing I never do is tell people their current way of working is wrong. That triggers defensiveness and kills adoption before it starts. Experienced merchants and operators often have genuine pattern recognition built from years of experience — the goal is to augment that, not replace it.

My approach has three phases:

**Phase 1 — Listen and mirror.** In the first few weeks, I sit with the teams and understand how they currently work. I ask: "What do you look at first on a Monday morning? What would make you more confident in that decision?" Then I take their existing spreadsheets, clean them up, automate their refresh, and give them back something better than what they had. This builds trust quickly because I'm solving their problem, not my agenda.

**Phase 2 — Show, don't tell.** I find one decision where data clearly improved the outcome and make sure it's visible and attributed. "We reacted to the slow sell-through on these styles two weeks earlier than we would have otherwise — that saved X in markdown." Early wins that are clearly connected to the analytics work create internal advocates.

**Phase 3 — Embed.** Once trust is established, I work to get data into the rhythm of regular meetings — not as a separate "data review" but as a natural part of trading reviews, planning sessions, and post-season analyses. The goal is that decisions are never made without referencing the data, not because it's mandated, but because people find it genuinely useful.

* * *

### Q13. How would you approach analytics for store operations at a retailer like Primark?

_Probing: Operational analytics knowledge, ability to identify value in non-obvious areas._

**Answer:**

Store operations is often an underserved area for analytics compared to trading and buying, which creates significant opportunity. The decisions being made at store level — around staffing, scheduling, space allocation, replenishment, and shrink — are numerous and have real P&L impact.

Key analytics areas I'd prioritise:

**Labour scheduling optimisation.** Understanding footfall patterns by hour, day, and season and aligning staffing levels accordingly is one of the highest-ROI applications of retail analytics. The data is usually available (transaction logs, footfall counters) but rarely connected.

**Stock availability and replenishment.** Tracking when products are on the shop floor versus in the stockroom, and how quickly replenishment happens, directly impacts sales. Missed sales due to poor availability are often invisible in standard reporting.

**Shrink and loss prevention.** Combining sales data, stock intake, and inventory counts to identify stores or categories with anomalous shrink rates.

**Store cluster performance.** Not all stores are the same — grouping stores by performance characteristics and trading patterns allows benchmarking and targeted interventions rather than one-size-fits-all decisions.

I'd engage regional managers and area managers early to understand what operational questions keep them up at night. The insight is rarely in the data alone — it's in the conversation between the data and the person who knows what the data should be telling them.

* * *

## SECTION 5: Stakeholder Management & Influence

* * *

### Q14. Tell me about a time you had to challenge a senior leader with data they didn't want to hear.

_Probing: Courage, influencing skills, ability to hold firm under pressure._

**Answer:**

At J&J Global Services, I was conducting a deep-dive analysis of operational processes across the Salesforce integration programme. The data clearly indicated that a significant portion of the manual reconciliation work — which the team was proud of and had significant time invested in — was creating $2M+ in avoidable costs and process inefficiencies. The recommendation was effectively to automate and restructure a process that senior stakeholders had built and championed.

I knew the finding would be uncomfortable, so I was deliberate about how I presented it. I led with the opportunity rather than the problem: "The analysis has identified $2M in recoverable value through process redesign — here's the evidence and the recommendation." I made sure the data was unambiguous — three different analytical approaches all pointed to the same conclusion — and I brought supporting documentation so the finding couldn't be dismissed as a methodology issue.

The initial reaction was sceptical. I was asked to re-examine my assumptions. I stood by the analysis, offered to walk through the methodology in detail, and suggested we validate against a sample. The sample confirmed the finding. The result was that the process was redesigned along the lines I recommended.

The lesson I took from that: data courage is only effective if it's paired with data rigour. You can only challenge senior leaders if the work is bulletproof.

* * *

### Q15. How do you manage stakeholders who have conflicting priorities for your team's time?

_Probing: Political navigation, prioritisation, communication skills._

**Answer:**

Conflicting stakeholder priorities are inevitable when an analytics team serves multiple business areas. Without a clear process, the team ends up torn in multiple directions and delivering mediocre results everywhere.

My approach: I establish a governance forum — typically a monthly analytics steering group — where the key business owners prioritise the analytics roadmap together, rather than each lobbying me individually. When stakeholders have to prioritise in front of each other, the conversations become much more productive. They often resolve conflicts themselves because they understand each other's context better.

For day-to-day conflicts, I maintain a transparent backlog — visible to all stakeholders — so everyone can see what's in progress, what's queued, and where their request sits. Transparency is the best antidote to the perception that someone is being favoured.

When genuine conflicts arise that can't be resolved at working level, I escalate with a clear options analysis: "If we prioritise X, Y will move to Q3. If we split resource, both take longer. Which outcome does the business prefer?" I make the trade-offs explicit and put the decision with the people who own the business outcomes.

* * *

### Q16. How do you communicate complex analytical findings to non-technical senior leaders?

_Probing: Data storytelling, communication skills, executive presence._

**Answer:**

The fundamental principle is: lead with the so-what, not the how. Senior leaders don't need to understand the methodology — they need to understand what it means for their decision.

My communication framework for executive audiences:

**One headline.** Every analytical finding should be summarisable in one sentence. If I can't state it in one sentence, I haven't finished the analysis yet. "Sales performance in the Southeast region is underperforming by 12% vs plan, driven primarily by two product categories — here's the recommendation."

**Evidence, not explanation.** I show enough of the data to establish credibility, but I don't walk through every step of the analysis. Two or three charts maximum. If they want the methodology, it exists and I can provide it — but I don't front-load it.

**Explicit recommendation.** I always end with a clear recommended action and the expected impact. Analytical findings without a recommendation are information, not insight. The job isn't done until I've told them what to do with it.

**Acknowledge uncertainty.** Where the data has limitations, I say so. Senior leaders who discover a caveat you didn't mention will never fully trust your analysis again.

At Google, I presented real-time R&D metrics to senior leadership across global teams via Looker Studio dashboards. The design philosophy was identical — make the headline visible in seconds, make the anomalies surfaceable without hunting, and be prepared to go deep if challenged.

* * *

## SECTION 6: Data Governance & Quality

* * *

### Q17. How do you approach data governance in a large retail organisation?

_Probing: Governance maturity, understanding of enterprise data complexity, practical vs. theoretical._

**Answer:**

Data governance in retail is particularly challenging because data is diverse (transactions, inventory, customer, supplier, logistics), high volume, and time-sensitive. A governance framework that works in a bank won't work in a retailer without significant adaptation.

My practical approach:

**Establish data ownership.** Every critical data domain — product, store, transaction, customer — needs a designated business owner who is accountable for its quality and definition. This is a business responsibility, not a technical one. The data team supports and enables; it doesn't own the business definitions.

**Prioritise by impact.** Governance effort should be proportional to business risk. Sales transaction data and inventory data need the tightest controls because decisions made on that data have immediate P&L consequences. Internal operational data can tolerate more flexibility.

**Build quality into the pipeline, not onto it.** Quality checks should fire at ingestion — not at dashboard level where the bad data has already propagated. I use automated row-count, null, range, and referential integrity checks at every pipeline stage.

**Metadata and lineage.** Every dataset should be catalogued with source, owner, refresh cadence, and known quality issues. When a number looks wrong, any analyst should be able to trace it in minutes. At AXA, I implemented this with Microsoft Purview — full lineage from source system to executive dashboard.

**Practical governance, not theoretical.** A 200-page governance policy no one reads does nothing. Simple, enforced standards — one source of truth per metric, every dashboard certified or labelled as exploratory — achieve more.

* * *

### Q18. A key business metric suddenly changes dramatically overnight. How do you diagnose and respond?

_Probing: Data quality instincts, analytical process under pressure, communication._

**Answer:**

This scenario is a test of both technical skill and composure. The worst response is to either immediately escalate a false alarm or to sit on it trying to figure it out while the business is making decisions on bad data.

My diagnostic process:

**Step 1 — Is it real or a data issue?** Before anything else, I check pipeline logs, run volumes, and data freshness indicators. In my experience, 70% of sudden metric changes are data issues — a failed ETL job, a duplicated load, a source system change. At AXA, I built monitoring that flagged exactly this type of anomaly automatically, which is why I'd build that capability at Primark from day one.

**Step 2 — Scope the impact.** Which dashboards and reports are affected? Who has already seen the data? Who is making decisions based on it?

**Step 3 — Communicate immediately.** If the data is bad, I tell the business before they discover it themselves. "We have detected an anomaly in [metric] — we are investigating and will confirm by [time]. Please do not use [specific report] for decisions until further notice." Transparency under pressure builds trust faster than perfect performance.

**Step 4 — Root cause and fix.** Fix the data, re-run the affected period, and validate the corrected output against expectations.

**Step 5 — Post-mortem.** How did this happen? What monitoring would have caught it earlier? What process change prevents recurrence? I document this and add the monitoring check to the standard pipeline.

* * *

## SECTION 7: Situational & Behavioural

* * *

### Q19. Why Primark specifically? Why this role?

_Probing: Genuine motivation, company knowledge, fit._

**Answer:**

Primark is an interesting analytics challenge for a specific reason: the scale and speed of the business creates enormous data complexity, but Primark's value proposition — fashion at accessible prices — means the commercial decisions being made every week are genuinely consequential. There's no luxury margin to absorb bad merchandising calls. That commercial sharpness makes the analytics work meaningful.

The fact that this is a new team being built from scratch is what makes this role compelling to me rather than just another analytics leadership position. I've spent most of my career building things — new pipelines, new data models, new reporting frameworks — and there's a specific satisfaction in building a team with the culture and capability standards embedded from the beginning, rather than inheriting a structure and trying to change it.

I've worked across retail-adjacent environments and in high-volume transactional data contexts — the migration of 20M+ retail customer records at Affinion, the commercial analysis work at J&J — and I'm keen to apply that experience in a business where the data directly drives the commercial decisions I'm close to.

The opportunity to work at a company growing to 440+ stores globally, where analytics capability is clearly being invested in at a strategic level, is exactly the kind of platform I want to be building on.

* * *

### Q20. What would your first 90 days look like in this role?

_Probing: Structured thinking, pragmatism, understanding of how to earn trust in a new organisation._

**Answer:**

**Days 1–30: Listen and learn.**

I don't build anything in the first month. I meet with every key business stakeholder — trading, merchandising, supply chain, store operations, finance — and ask two questions: "What data do you currently use to make decisions?" and "What decision do you wish you could make better with data?" I map the existing analytics landscape: what tools exist, what data is available, where the quality issues are, and where the biggest unmet demand is.

I also spend time with the data team — or those adjacent to it — understanding the current state of the data infrastructure, the tech stack, and any legacy technical debt.

**Days 31–60: Diagnose and plan.**

I synthesise everything I've learned into a clear picture of the current analytics maturity and the highest-priority opportunities. I draft a 12-month analytics roadmap with a prioritised backlog, a proposed team structure, and a capabilities assessment. I identify two or three quick wins — analysis or dashboards that can be delivered in weeks and will build credibility with the business.

I also develop the governance foundations: metric definitions, data quality standards, and the BI architecture principles that will govern everything the team builds.

**Days 61–90: Deliver and hire.**

I deliver the first quick wins. I begin the hiring process for the team, using the capability map I've built to spec the roles needed. I run the analytics roadmap past key stakeholders for alignment and secure sponsorship for the longer-term platform investments.

The goal at 90 days is not to have transformed anything — it's to have built the trust, the plan, and the foundations that make the transformation possible.

* * *

### Q21. What is your biggest professional failure and what did you learn from it?

_Probing: Self-awareness, learning orientation, honesty._

**Answer:**

Early in my career at Cognizant, I built what I believed was an excellent data migration solution for a client — technically robust, well-tested, and delivered on time. What I underestimated was the change management component. The analysts who were supposed to use the new system were comfortable with the legacy process, hadn't been involved in the design, and found reasons to work around the new solution rather than adopt it. Six months after delivery, adoption was much lower than expected and the business value wasn't being realised.

The technical work was good. The project management failed because I had treated the human side of change as someone else's problem.

That experience fundamentally changed how I approach analytics delivery. Now I involve end users in design from the beginning — their feedback surfaces edge cases, yes, but more importantly, their involvement creates ownership. I also build adoption metrics into every delivery: usage rates, active user counts, and time-to-first-insight. If adoption is low, the project isn't finished, regardless of what the technical delivery checklist says.

Every major platform I've built since — the AXA dashboards, the BigQuery model for J&J — has had explicit stakeholder engagement embedded in the delivery plan, not bolted on at the end.

* * *

### Q22. Where do you see analytics capability evolving in retail over the next three to five years?

_Probing: Forward-thinking, awareness of trends, strategic vision._

**Answer:**

Three shifts I think will define the next period:

**From descriptive to prescriptive.** Most retail analytics today is still largely descriptive — it tells you what happened. The maturity shift is toward prescriptive analytics: systems that don't just surface a slow sell-through rate but recommend the optimal markdown timing and depth, or that don't just flag a staffing variance but suggest the reallocation. This requires cleaner data foundations and stronger commercial logic embedded in the models, which is why getting the governance and architecture right now is so important.

**AI-augmented analysis.** Large language models are going to change how analysts interact with data. Natural language querying of data models, automated narrative generation for reporting, and AI-assisted anomaly investigation will all become standard. This doesn't replace analysts — it removes the low-value work and frees them for higher-level interpretation and decision support. Teams that build good data foundations now will be best placed to exploit these capabilities.

**Real-time operational intelligence.** Retailers are moving toward in-store operational decisions supported by near-real-time data — replenishment triggers, live availability monitoring, dynamic labour allocation. This requires different architecture (streaming pipelines, edge analytics) than traditional batch reporting. At AXA, I reduced data latency from 24 hours to under 15 minutes using CDC pipelines — the same architectural thinking applies to retail operational data.

For Primark specifically, given the scale and the value proposition, I'd be particularly focused on availability analytics and markdown optimisation — both areas where better data can protect margin while maintaining the accessibility of the brand.

* * *

## SECTION 8: Questions to Ask the Interviewer

_Asking great questions signals strategic thinking and genuine interest. Pick 3–4 from the list below._

* * *

  1. **"What is the current state of analytics maturity at Primark, and what are the biggest gaps this team is being created to address?"**
  2. **"How would you describe the relationship between the analytics function and the trading/merchandising teams today? Is the appetite for data-driven decision-making already established, or is part of the brief to build that culture?"**
  3. **"What does success look like for this role at 6 months, 12 months, and 3 years?"**
  4. **"What is the current technology stack, and are there any constraints or decisions already made about the future direction — particularly around Alteryx, Tableau, and Power BI?"**
  5. **"How many people do you envisage in this team at maturity, and what does the hiring timeline look like?"**
  6. **"What are the biggest commercial challenges Primark is navigating right now where analytics could play a role?"**
  7. **"How does this role connect to Primark's broader data strategy — is there a group-level data platform or data science function this team would work alongside?"**
  8. **"What are the governance and data quality challenges in the current analytics estate that this role would need to address?"**



* * *

## QUICK REFERENCE: Key Numbers to Drop in Interview

Achievement| Context  
---|---  
**100+ manual hours/month eliminated**|  Google Street View — workflow automation  
**$2M+ cost savings identified**|  J&J Global Services — deep-dive commercial analysis  
**24 hours → 15 minutes** data latency| AXA Insurance & Travelers — CDC pipeline implementation  
**40% reduction** in manual review effort| AXA Insurance — data quality monitoring framework  
**85% reduction** in reconciliation issues| LSEG/Refinitiv — real-time quality monitoring  
**20M+ customer records** migrated| Affinion Voyager — Azure cloud migration  
**150+ analysts and executives** served| J&J — consolidated BigQuery single source of truth  
**200+ business attributes** documented| J&J — STTM governance documentation  
**11+ years** cross-industry analytics experience| Full career  
**Stamp 4** visa — full work authorisation| Ireland  
  
* * *

_Prepared specifically for Shajeeb S | Primark Applied Insights & Analytics Lead | April 2026_
