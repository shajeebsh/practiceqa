---
title: "CubicAutoTest"
date: 2026-03-30 18:13:23 
author: shajeebhameed
layout: page
slug: cubicautotest__trashed
status: trash
---

> Claude [link](<https://claude.ai/share/fab7851b-a2e6-4e84-8a0d-04a19e5b8f9a>)

I'll train you in a mock interview format, covering all key areas of this Cubic³ role. Let me structure this as a proper interview training program.

* * *

### 🎯 Cubic³ — Senior Test Automation Engineer

### Complete Interview Training Program

* * *

#### 📋 PART 1: ROLE ANALYSIS — Know What They're Really Looking For

Before answering a single question, understand the **three pillars** of this role:

Pillar| What It Means  
---|---  
**Technical Leadership**|  You're the SME — teams look to you for framework decisions, tooling choices, best practices  
**Hands-On Engineering**|  You still write code — frameworks, automation suites, CI integration  
**Enablement & Mentoring**| You multiply the team — training, dashboards, standards, upskilling others  
  
Cubic³ is clearly building a **centre of excellence for test automation** — your answers must reflect someone who can both do the work AND raise the bar across the engineering group.

* * *

#### 📋 PART 2: YOUR OPENING — "Tell Me About Yourself"

This is always the first question. Here's a **formula** and a **template** :

**Formula:** Current Role → Key Achievements → Technical Depth → Leadership/Mentoring → Why Cubic³

**Template Answer:**

> "I'm a Senior Test Automation Engineer with X years of experience, specialising in building and scaling automation frameworks across API, web, and non-functional testing. In my current role at [Company], I [key achievement — e.g., designed a Playwright-based E2E framework that reduced regression time from 4 hours to 25 minutes]. I've worked extensively with .NET/C# and Python, and have strong experience integrating automation into CI/CD pipelines. More recently I've been exploring how AI can be leveraged in automation strategies — from intelligent test generation to flakiness detection. Beyond the technical side, I've mentored junior automation engineers and contributed to quality culture across cross-functional teams. I'm drawn to Cubic³ because of the scale and complexity of what you're building — particularly the IoT and telecoms domain, and the opportunity to shape automation strategy at a senior level."

* * *

#### 📋 PART 3: QUESTION BANK WITH FULL ANSWERS

* * *

#### 🔷 SECTION A: Framework Design & Architecture

* * *

**Q1: Walk me through how you would design a test automation framework from scratch for a new project.**

**A:** I approach it in five phases:

**1\. Discovery:** I meet with engineering leads, architects, and product owners to understand the tech stack, release cadence, team skill levels, and what "quality" means for this product. I ask: what are the riskiest areas? What's the current pain point in testing?

**2\. Framework Selection:** Based on the stack, I select tools that align with the team's existing languages. For a .NET/C# shop, I'd lean towards RestSharp for API testing, Playwright or Selenium for UI, and SpecFlow if BDD is valued. I align choices with the Architecture Team's Technology Radar — I don't introduce tools that fall outside approved standards without a formal proposal.

**3\. Architecture Design:** I design for the testing pyramid — heavy unit/integration coverage, leaner E2E. Core design patterns I use: Page Object Model for UI, Service Object pattern for API, Builder pattern for test data, and a shared config/environment abstraction layer.

**4\. CI Integration:** The framework must run headlessly in CI from day one — not as an afterthought. I wire it into the pipeline (GitHub Actions, Azure DevOps, Jenkins) with parallel execution, test result publishing, and failure alerting.

**5\. Documentation & Onboarding:** I write a getting-started guide so any engineer can run the suite in under 10 minutes. Frameworks that aren't adopted are worthless.

* * *

**Q2: How do you evaluate whether to extend an existing automation framework vs. rebuild it?**

**A:** I use a structured assessment:

  * **Flakiness rate** — above 15-20% is a serious signal.
  * **Maintenance burden** — if the team spends more time fixing tests than writing new ones, that's a red flag.
  * **Scalability** — can it run in parallel? Can it scale with the product?
  * **Tech debt** — is the underlying tooling deprecated or unmaintained?
  * **Adoption** — are engineers actively using and contributing to it?



If two or more of these are failing, I make the case for a rebuild. But I do it incrementally — run old and new frameworks in parallel, migrate test by test, and retire the old framework once the new one reaches parity. A big-bang rewrite carries its own risk.

* * *

**Q3: What automation design patterns do you use and when?**

**A:**

Pattern| When I Use It  
---|---  
**Page Object Model (POM)**|  UI test suites — separates locators from test logic  
**Screenplay Pattern**|  Complex UI journeys with multiple actors/roles  
**Service Object / API Client**|  API test layers — one class per resource  
**Builder / Factory**|  Test data generation — fluent, readable data setup  
**Strategy Pattern**|  Swappable components — e.g., different auth strategies per environment  
**Facade Pattern**|  Simplifying complex external dependencies for tests  
  
At the senior level, I also review other engineers' code for anti-patterns — like tests that depend on each other, hardcoded test data, or logic buried in helper methods that makes tests unreadable.

* * *

#### 🔷 SECTION B: API & Microservice Testing

* * *

**Q4: How do you approach API test automation for a microservices architecture?**

**A:** In microservices, I test at three levels:

**Component level** — test each service in isolation using mocked dependencies (WireMock, MockServer). This gives fast, reliable feedback.

**Contract level** — use Pact or similar consumer-driven contract testing to verify that service interfaces honour their agreements without requiring both services to be running. Critical in a microservices environment where teams deploy independently.

**Integration level** — a targeted suite that tests real service interactions in a deployed test environment. Smaller in scope than the component suite but verifies the actual wiring works.

For tools, I use **RestSharp** (C#) or **RestAssured** (Java) for the programmatic test layer. I validate not just status codes but full response schemas using JSON Schema validation on every response.

* * *

**Q5: How have you used mocking frameworks in your automation work?**

**A:** Mocking is essential for reliable, fast automation. I use mocking at several levels:

  * **WireMock** — for stubbing HTTP dependencies in integration tests. I define stubs as JSON mappings checked into version control alongside tests, so stubs are versioned and reviewed.
  * **In-language mocking** — Moq in C#, Mockito in Java — for unit-level test isolation.
  * **Record and replay** — some tools let you record real API traffic and replay it in tests, which is useful for complex third-party integrations.



The discipline I enforce is that mocks must reflect reality. Stale mocks that don't match the real service are worse than no mocks at all — they give false confidence. I pair mock usage with contract tests to keep them honest.

* * *

**Q6: How do you test TLS/SSL on server infrastructure?**

**A:** This is relevant to Cubic³'s telecoms/IoT context. My approach:

  * Verify the certificate is valid, not expired, and issued by a trusted CA using tools like `openssl s_client` or `testssl.sh`.
  * Verify the correct TLS version is enforced (TLS 1.2 minimum; TLS 1.3 preferred) and older versions (TLS 1.0, 1.1, SSLv3) are rejected.
  * Test cipher suite configuration — only strong ciphers should be accepted.
  * In automated tests, configure the HTTP client to enforce certificate validation and test that self-signed or expired certs are rejected in production-mode config.
  * Test mutual TLS (mTLS) where required — client certificate is presented and validated.



* * *

#### 🔷 SECTION C: CI/CD & Continuous Testing

* * *

**Q7: Describe your experience building automation into CI/CD pipelines.**

**A:** I've integrated automation into pipelines using Azure DevOps, GitHub Actions, and Jenkins. My standard approach:

  * **Unit and API tests** run on every commit — fast feedback loop, must complete in under 5 minutes.
  * **Integration tests** run on PR merge to the main branch — slightly longer, but still gated before deployment.
  * **E2E regression suite** runs nightly or on deployment to a staging environment.
  * **Performance tests** run on a scheduled basis or when triggered by significant changes.



Pipeline configuration as code — I write YAML pipeline definitions checked into the same repo as the tests. No manual pipeline configuration.

I also implement **test result publishing** — integrating with tools like Allure, Azure Test Plans, or TestRail via API so results are visible in dashboards, not just in raw logs.

* * *

**Q8: How do you ensure your automation suite doesn't become a bottleneck in CI?**

**A:** Several strategies:

  * **Parallelism** — distribute tests across multiple agents or containers. A suite of 1,000 tests that takes 2 hours sequentially can run in 15 minutes across 10 agents.
  * **Test tagging and selective execution** — tag tests by type, feature, or risk level. On a hot-fix branch, only run smoke and critical path tests.
  * **Fail fast** — configure pipelines to stop on first P1 failure rather than running the full suite when a critical path is already broken.
  * **Quarantine flaky tests** — flaky tests that aren't quarantined erode confidence and slow everyone down with re-runs.
  * **Caching** — cache dependencies (NuGet packages, node_modules) so pipeline startup time is minimal.



* * *

#### 🔷 SECTION D: AI in Test Automation

* * *

**Q9: How have you used or would you use AI in test automation strategies?**

**A:** This is directly called out in the Cubic³ job spec, so give a strong answer:

**Current/Practical uses I've implemented or explored:**

  * **AI-assisted test generation** — tools like GitHub Copilot, Diffblue (for Java), or custom LLM prompting to generate boilerplate test cases from OpenAPI specs or user stories. This speeds up initial test creation significantly.
  * **Visual AI testing** — tools like Applitools use AI to detect visual regressions, far more resilient to minor UI changes than pixel-perfect comparisons.
  * **Flakiness detection and root cause analysis** — ML models trained on test history can identify which tests are likely to be flaky before they waste CI time.
  * **Self-healing locators** — tools like Healenium use AI to automatically update broken UI locators when the DOM changes, reducing maintenance overhead.
  * **Log and failure analysis** — LLM-based tools can summarise long test failure logs and suggest root causes, saving engineers time in triage.



**Strategic direction I'd propose at Cubic³:** Build an AI layer into the quality dashboard that flags anomalies in test results, predicts high-risk areas based on code change patterns, and recommends test coverage gaps.

* * *

**Q10: What are the risks of using AI in test automation and how do you mitigate them?**

**A:**

Risk| Mitigation  
---|---  
AI-generated tests may be low quality or miss edge cases| Human review of all AI-generated tests before merging  
Self-healing locators may mask real UI bugs| Log all healed locators and review them in retrospectives  
Over-reliance on AI tools creates skill atrophy| Use AI to accelerate, not replace — engineers must understand what tests do  
AI tools introduce new dependencies and costs| Evaluate against Technology Radar; get architecture sign-off  
Non-determinism in AI-powered assertions| Define acceptable variance ranges; use structured outputs  
  
* * *

#### 🔷 SECTION E: Non-Functional Testing

* * *

**Q11: Walk me through how you design a performance test strategy.**

**A:** I design performance testing around four test types:

**Load testing** — expected concurrent users, verify SLA compliance (e.g., p95 response time < 500ms).

**Stress testing** — push beyond expected load to find the breaking point and understand failure modes.

**Spike testing** — sudden traffic burst (relevant for Cubic³'s telecoms context where network events can cause traffic spikes).

**Soak/Endurance testing** — sustained load over hours to catch memory leaks and connection pool exhaustion.

**Tools:** JMeter for script-based load testing with broad protocol support; k6 for code-first, CI-friendly performance tests; Gatling for high-concurrency Scala-based tests.

**Metrics I report:** p50/p95/p99 response time, throughput (RPS), error rate, Apdex score. I correlate test timelines with infrastructure metrics (CPU, memory, DB connections) to identify root causes.

* * *

**Q12: How do you approach security testing in automation?**

**A:** I integrate security testing at multiple pipeline stages:

  * **SAST (Static)** — SonarQube or Checkmarx in the build pipeline, catching code-level vulnerabilities before deployment.
  * **DAST (Dynamic)** — OWASP ZAP in the test environment, running automated scans against running APIs.
  * **Dependency scanning** — OWASP Dependency Check or Snyk to flag vulnerable third-party libraries.
  * **Manual security test cases** — OWASP Top 10 coverage as part of the API test suite: injection, broken auth, IDOR, mass assignment, sensitive data exposure.



For Cubic³'s IoT/M2M context specifically, I'd also test device authentication, firmware update integrity, and communication channel security.

* * *

#### 🔷 SECTION F: IoT, Telecoms & Networking

* * *

**Q13: What specific challenges does IoT/M2M testing present compared to standard web application testing?**

**A:** IoT introduces several unique testing challenges:

  * **Device diversity** — testing across different hardware profiles, firmware versions, and network conditions is far more complex than browser compatibility.
  * **Network simulation** — M2M systems must handle intermittent connectivity, packet loss, and low-bandwidth scenarios. I use network emulation tools (tc/netem on Linux, Clumsy on Windows) to simulate these conditions.
  * **Protocol variety** — beyond HTTP/REST, IoT systems use MQTT, CoAP, AMQP. Test tooling must support these protocols.
  * **State management** — devices have persistent state (firmware version, configuration). Tests must manage device state carefully to be repeatable.
  * **Scale** — testing at scale means simulating thousands of concurrent devices, not just hundreds of concurrent users.
  * **Security surface** — physical access, firmware tampering, and device identity are attack vectors that don't exist in web apps.



* * *

**Q14: What experience do you have with telecoms/IP networking testing?**

**A:** _(Tailor to your actual experience — here's a framework if you have limited exposure:)_

> "My direct telecoms experience includes [X]. I'm familiar with IP networking fundamentals — TCP/UDP, DNS, routing, NAT, VLANs — and have tested REST APIs that sit on top of network infrastructure. I'm experienced with testing TLS/SSL configurations and have worked with tools like Wireshark for packet-level debugging during test failure investigation. I'm actively building on this — I've been studying [MQTT/CoAP/SIP] as part of preparing for roles in this domain, and I'm confident that my strong automation engineering fundamentals transfer quickly to new protocol domains."

* * *

#### 🔷 SECTION G: Leadership, Mentoring & Communication

* * *

**Q15: How do you approach mentoring junior and mid-level automation engineers?**

**A:** I structure mentoring around three tracks:

**Technical track** — pair programming on framework tasks, code reviews with detailed written feedback (not just "fix this" but "here's why this pattern is better"), and recommending specific learning resources.

**Problem-solving track** — when a junior engineer hits a blocker, I resist giving the answer immediately. I ask guiding questions — "what have you tried?", "what does the error tell you?", "where would you look next?" — to build their debugging instinct.

**Visibility track** — I actively create opportunities for less experienced engineers to present their work, lead a test design session, or write a knowledge base article. Growth requires visibility, not just skill.

I also maintain an open-door policy — I make it clear that no question is too basic. A team where juniors are afraid to ask questions has a knowledge gap hiding in plain sight.

* * *

**Q16: How do you drive quality culture across teams that don't report to you?**

**A:** Culture change through influence requires a three-part approach:

**Make quality visible** — the quality dashboard Cubic³ mentions is key here. When teams can see their test coverage, flakiness rates, and defect escape rates in one place, quality becomes a shared concern, not just a QA concern. Numbers create accountability without confrontation.

**Make good practices easy** — if the right way is also the easy way, adoption follows. I invest in shared libraries, templates, and starter kits so engineers can write good tests without being experts. Friction is the enemy of adoption.

**Build relationships** — I schedule regular 1:1s with engineering leads across teams. I learn their pain points and tailor quality solutions to their specific context. I also celebrate wins publicly — when a team catches a critical bug via automation, that story gets told.

* * *

**Q17: Describe how you'd present a new automation strategy proposal to engineering leadership.**

**A:** I structure it as a business case, not a technical document:

  1. **Current state** — here's what we have today, here's what it's costing us (defect escape rate, manual testing hours, release delays).
  2. **Proposed state** — here's what I'm proposing, here's how it works at a high level.
  3. **ROI** — here's the expected reduction in manual effort, faster release cycles, earlier defect detection.
  4. **Risks and mitigations** — I address concerns proactively rather than waiting to be challenged.
  5. **Phased plan** — a big-bang implementation proposal gets rejected. A phased plan with quick wins in phase 1 gets approved.



I keep slides visual and minimal — one key point per slide. I prepare for the questions leadership always asks: "How long will this take?", "What does it cost?", "What happens if it doesn't work?"

* * *

#### 🔷 SECTION H: Tools — Deep Dives

* * *

**Q18: Compare Selenium and Playwright for a senior-level UI automation decision.**

**A:**

Factor| Selenium| Playwright  
---|---|---  
**Architecture**|  WebDriver protocol — external driver process| Direct browser communication — faster, more reliable  
**Auto-waiting**|  Manual waits required| Built-in smart auto-waiting  
**Parallel execution**|  Requires Grid setup| Native parallelism out of the box  
**Browser support**|  Chrome, Firefox, Safari, Edge, IE| Chrome, Firefox, Safari, Edge (no IE)  
**Language support**|  Java, C#, Python, Ruby, JS — broad| JS/TS, Python, Java, C#  
**Ecosystem maturity**|  15+ years, massive community| 4+ years, rapidly growing  
**Mobile**|  Appium extends it| Experimental mobile support  
**Shadow DOM / iframes**|  Painful to work with| Much better native support  
  
**My recommendation for Cubic³:** Playwright for new web automation work. For C# specifically, Playwright has a well-maintained .NET binding. The auto-waiting alone eliminates the majority of flakiness I've seen in Selenium suites.

* * *

**Q19: How do you use TestRail effectively in a large engineering organisation?**

**A:** TestRail works best when it's the single source of truth for test coverage. Key practices:

  * **Link test cases to requirements** — every test case references a Jira story or requirement. This gives you traceability from requirement to test to result.
  * **Automated result publishing** — integrate your test framework with TestRail's API so automated test results are published automatically after each run. No manual result entry.
  * **Milestones for releases** — use TestRail milestones to track test completion per release. Leadership can see at any moment what percentage of test cases have passed for an upcoming release.
  * **Test plans for regression** — maintain a curated regression plan that's updated each sprint, not a static document that drifts.
  * **Metrics** — use TestRail's reports to track defect density per feature area, which informs where to invest more test coverage.



* * *

**Q20: How do you work within a Kanban/lean engineering model?**

**A:** Kanban suits test automation work well because automation tasks don't always align neatly with sprint boundaries. Key practices:

  * **WIP limits** — I honour WIP limits strictly. Having three automation frameworks in progress simultaneously means none gets done well.
  * **Flow metrics over velocity** — I track cycle time (how long a task takes from start to done) and use it to identify bottlenecks, rather than focusing on story points.
  * **Continuous deployment mindset** — automation work is never "done" — it's always maintained and improved. Kanban's continuous flow model fits this better than sprint commitments.
  * **Blocker visibility** — I make blockers immediately visible on the board and escalate quickly rather than letting tasks sit in limbo.



* * *

#### 📋 PART 4: QUESTIONS TO ASK THEM

Always prepare 4–5 strong questions. These signal senior-level thinking:

  1. _"What does the current test automation landscape look like at Cubic³ — what frameworks exist and what are the biggest pain points you're looking to solve?"_
  2. _"How does the Test Systems Team interact with individual engineering squads — is it an embedded model or a centre of excellence model?"_
  3. _"How mature is the Technology Radar process, and how much influence does the QA team have in adding or retiring tools?"_
  4. _"What does success look like for this role in the first 6 and 12 months?"_
  5. _"How is AI currently being used or explored across the engineering group?"_



* * *

#### 📋 PART 5: MOCK INTERVIEW RAPID-FIRE PREP

Here are quick-answer prompts — answer these in under 90 seconds each:

  * _"What's the most complex automation framework you've built?"_
  * _"Tell me about a time your automation caught a critical production bug."_
  * _"How do you handle a situation where developers push back on adding test automation?"_
  * _"What's your approach to test data management?"_
  * _"Describe a time you failed in a QA initiative and what you learned."_
  * _"How would you upskill a developer who has zero automation experience?"_
  * _"What's your view on 100% test automation coverage — is it a good goal?"_



* * *

Would you like me to now run a **live mock interview** where I ask you questions one at a time and give you feedback on your answers? Or would you prefer I deep-dive on any specific section — such as the **IoT/telecoms angle** , the **C#/.NET automation specifics** , or the **AI in testing strategy**?

> 30-minute interview with the QA Engineering Manager(Claude [link](<https://claude.ai/share/c1a93ac1-4a84-42bf-98e4-dfcbd661dc19>)) 

Congratulations Shajeeb! That's a great step forward. A 30-minute interview with the QA Engineering Manager is typically a first-stage screening — focused on fit, experience overview, and a few technical probes rather than deep coding. Let me build you a full preparation plan for Friday.---

#### 🎯 What Samantha Is Screening For

As a QA Engineering Manager, Samantha will be assessing three things in 30 minutes:

**Can this person lead technically?** Cubic³ wants a senior SME, not just a hands-on engineer. She needs to see that you set direction, make framework decisions, and influence teams — not just execute tasks assigned to you.

**Does the experience translate to Cubic³'s domain?** Cubic³ works in telecoms and IoT — complex, distributed systems. Your experience across Fintech (LSEG), Life Sciences (J&J), and Geospatial (Google) shows you handle complexity. Draw that connection explicitly.

**Is AI testing real or buzzword?** The job spec called out AI in automation strategy. Your RAG/RAGAS experience is a genuine differentiator. Samantha will probe whether it's real depth or CV padding.

* * *

#### 🗣️ Your Opening — "Tell Me About Yourself"

This is the single most important answer of the interview. You have roughly 90 seconds. Here's a tight, punchy version tailored to the Cubic³ role:

> "I'm a Senior QA Automation Engineer and SDET with 11 years of experience, based here in Dublin. My background spans building and scaling test automation frameworks across some complex enterprise environments — Google Street View, Johnson & Johnson's global services platform, LSEG's FX trading system, and Cisco. I work primarily in Playwright and Python for web and API automation, and Robot Framework for keyword-driven enterprise workflows.

> What's probably most relevant to Cubic³ is the combination of two things. First, I've recently been working on AI quality engineering — specifically testing a RAG-based LLM platform, implementing the RAGAS evaluation framework, and designing adversarial prompt suites for hallucination detection. That's a capability most automation engineers don't have yet. Second, I've led quality strategy across teams up to 30 engineers, so I understand the SME and mentoring side of a senior role, not just the hands-on engineering.

> I'm particularly drawn to Cubic³ because of the technical complexity of what you're building in the telecoms and IoT space — it's a domain where quality engineering really matters, and where an AI-informed automation strategy can make a significant difference."

* * *

#### 💪 Your Three Power Stories

In 30 minutes you won't cover everything. Prepare these three STAR stories cold — they cover the most likely probes:

**Story 1 — Framework at scale (Google, 27 months)**

> Playwright/Python E2E framework, 300+ geospatial workflows, 99.8% accuracy, eliminated 100+ manual hours/month, integrated with Bug Tracking API, CI/CD gates in Google Cloud Workflow. This is your technical depth story.

**Story 2 — AI/RAG testing (Confidential Enterprise Client)**

> Designed end-to-end RAG pipeline quality strategy, RAGAS implementation, hallucination detection, adversarial prompt suites, CI/CD quality gates in Python. This is your differentiator story — nobody else in the room will have this.

**Story 3 — Team leadership (J &J, team of 24)**

> Automation-first in-sprint strategy, Robot Framework with CumulusCI, mobile automation with Appium, multilingual email automation, X-Ray/JIRA integration across 24-person team. This is your leadership and collaboration story.

* * *

#### ❓ Three Strong Questions to Ask Samantha

These signal strategic thinking and genuine curiosity — not "when do I start?"

  1. _"What does the current test automation landscape look like at Cubic³ — and what are the biggest pain points you're hoping to address with this hire?"_
  2. _"How is AI currently being explored or used within the engineering and QA teams? I've been working extensively on AI quality engineering and I'm curious where Cubic³ is on that journey."_
  3. _"What does success look like in the first 90 days for someone coming into this role?"_



* * *

#### ⚡ Friday Checklist

  * [ ] Re-read the Cubic³ job spec the morning of the interview
  * [ ] Practice your "tell me about yourself" out loud — time yourself at 90 seconds
  * [ ] Have your three power stories ready with specific numbers
  * [ ] Prepare your three questions for Samantha
  * [ ] Test your video/audio if it's remote — confirm the format with Kevin if unclear
  * [ ] Have a copy of your CV visible during the call
  * [ ] Be ready 5 minutes early



* * *

Click any question in the card above to get your fully tailored answer for that specific question. Would you like me to run a full **mock 30-minute interview** right now — where I play Samantha and ask you questions one at a time with feedback after each answer?

1\. Tell me about yourself 

I'm a Senior QA Automation Engineer and SDET with over 12 years of experience building and scaling test automation frameworks across web, mobile, API, and cloud platforms — for global clients like Google, Johnson & Johnson, LSEG, Cisco, and Thomson Reuters.

My career has spanned some genuinely diverse and complex domains — geospatial data at Google Street View, FX trading at Refinitiv, global HR services at J&J, and more recently AI/LLM quality engineering, where I've been validating RAG pipelines and testing LLM response quality using the RAGAS evaluation framework.

On the technical side, I'm strongest in Playwright, Robot Framework, Selenium, Python, and CI/CD pipelines — Jenkins, GitHub Actions, and Google Cloud Workflow. But what I'm probably most proud of is the strategic side: I've led teams of up to 30 engineers, built automation strategies from scratch, and consistently driven measurable outcomes — things like eliminating 100-plus manual hours a month, cutting bug resolution time by 18 hours, and achieving 99.8% data transformation accuracy.

I'm based in Dublin, fully eligible to work in Ireland, and I'm at a point in my career where I'm looking for a role like this one at Cubic3 — where I can bring that breadth of experience into a high-impact Senior Automation Engineer position and help shape quality culture from the ground up.

2. Walk me through the most complex automation framework you have built 

SITUATION  
At Google Street View, I was brought in to build automation for a large-scale geospatial data pipeline — covering over 300 distinct workflows from raw data ingestion all the way through transformation and publishing to Google Maps layers. This was complex at every level: the data volumes were enormous, the workflows were tightly interdependent, and there was no existing automation — everything was manual. The team was six engineers and the timelines were aggressive.

TASK  
My mandate was to architect an end-to-end test automation framework from scratch that could validate data integrity across the full pipeline, integrate into CI/CD to block bad deployments, and generate reliable reporting for executive stakeholders — all while the team kept delivering sprint work in parallel.

ACTION  
I designed a Playwright/Python framework as the core engine, building modular test components that could validate each stage of the pipeline independently and then as a chain. I integrated directly with BigQuery to run automated ETL validation — asserting transformation accuracy at each layer. I also built a direct integration with Google's Bug Tracking API so defects raised by the framework were automatically logged with context, cutting out the manual triage loop entirely. On the CI/CD side, I implemented quality gates in Google Cloud Workflow that would automatically block a deployment if pipeline test thresholds weren't met. And I automated Looker Studio dashboard validation so that the executive-level reporting layer was also covered — not just the back-end data.

RESULT  
The framework achieved 99.8% data transformation accuracy across all Google Maps layers. It eliminated over 100 manual hours per month. The Bug Tracking API integration alone cut average bug-to-resolution time by 18 hours. Separately, through data analysis enabled by the framework, we identified a critical ingestion bottleneck that drove a 25% pipeline latency reduction the following quarter. At J&J, a comparable but different complexity challenge — I built a Robot Framework suite with a custom REST API keyword library covering 24 engineers and multiple Salesforce modules including HR, Payroll, and Procurement — but the Google Street View framework is the one I'd point to as the most architecturally complex.

3. How have you implemented AI in your test automation strategy 

This is probably the area I'm most excited to talk about, because my experience here goes beyond just using AI-assisted tools — I've actually been building quality engineering strategy for AI systems themselves.

At Google, I was responsible for the end-to-end QA strategy for a RAG-based AI platform — Retrieval-Augmented Generation. The challenge with testing AI is that you can't simply assert "output equals expected output" the way you would with deterministic software. The outputs are probabilistic, context-dependent, and can degrade in subtle ways that aren't obvious until you measure them quantitatively.

So I implemented the RAGAS evaluation framework — which gave us quantitative metrics across four key dimensions: retrieval accuracy, context relevance, faithfulness, and answer correctness. These became our quality gates — if a deployment failed to meet defined thresholds on any of those dimensions, it was blocked from going to production. That was a significant shift: moving LLM quality from a subjective, human-review process to an automated, metrics-driven pipeline.

Beyond RAGAS, I built adversarial prompt test suites specifically designed to probe the boundaries of the RAG system — prompt injection attacks, jailbreak attempts, edge-case retrieval scenarios where the model might hallucinate by confusing context or drifting on facts. Hallucination detection was a key workstream — I designed test cases that deliberately targeted knowledge boundary violations.

The other important piece was working with ML engineers to translate non-deterministic AI behaviours into testable acceptance criteria. That required a different kind of thinking — defining what "good enough" looks like when you can't guarantee exact outputs.

So to directly answer: my AI automation experience isn't about using Copilot to generate test scripts — it's about building the quality infrastructure that makes AI systems themselves production-safe.

4. How do you drive quality culture and mentor other engineers 

Quality culture, to me, starts with making quality everyone's problem — not just the QA team's. The most effective thing I've done consistently across my career is to position automation and quality thinking as a product delivery accelerator, not a gate at the end.

The clearest example is the In-Sprint automation strategy I implemented at Johnson & Johnson. The team was 24 engineers across a global Salesforce implementation — HR, Payroll, Procurement, Live Chat. Rather than having QA work in a separate lane, I designed an automation-first workflow where test coverage was being built in the same sprint as the feature, across Predev, UAT, Preprod, and Production environments. That changed how developers thought about their own deliverables — they knew automation was coming in-sprint, so they designed with testability in mind from the start.

On mentoring specifically — with a team that size, you can't individually coach everyone all the time. What I found most effective was building shared tooling and standards that made good practice the path of least resistance. At J&J, I built JIRA/X-Ray integration tooling that made test traceability and sprint reporting easy for the whole team — so good documentation and coverage tracking became natural, not an overhead. At Thomson Reuters, I led RPA best practices across a team of 30 and introduced an auto-rerun algorithm for flaky tests — that was a teaching moment too, because it forced a conversation about test reliability standards.

For more junior engineers, I tend to get them involved in architecture decisions early — even if just as observers — because understanding why the framework is designed a certain way is what makes people grow beyond just writing test scripts. I also use code review as a coaching tool rather than just a gating mechanism.

The measure I use for whether quality culture is working is simple: are developers raising quality concerns themselves, before QA does? When that starts happening, you know the culture has shifted.

5. How do you integrate automation into CI/CD pipelines

CI/CD integration has been a constant thread across all of my projects, and I've worked across quite a range of tooling — Jenkins, GitHub Actions, Google Cloud Workflow, and Azure DevOps — so I can speak to the differences between them.

My core philosophy is that automation in CI/CD should be a deployment blocker, not an afterthought report. The pipeline should fail fast and fail loudly when quality thresholds aren't met.

The most technically interesting CI/CD work I did was at Google. I implemented quality gates in Google Cloud Workflow that would automatically block a deployment if the pipeline test suite didn't pass. This wasn't just a pass/fail — I had to define specific quality thresholds tied to the RAGAS metrics for the LLM platform, and separately tied to data transformation accuracy for the geospatial pipeline. The automated test suite for the LLM work was integrated so that any deployment failing response consistency or retrieval precision thresholds was hard-blocked before it could reach production.

At Refinitiv, I designed the AutoTestService for automated deployment of the FX Trading GUI and its test server infrastructure — integrating directly with Jenkins to ensure every build triggered a full regression sweep. I also built a ReRun utility for automatic re-execution of failed TestRail cases, which meant the CI pipeline was more resilient to transient failures rather than raising false alarms.

At J&J, the Jenkins integration covered Robot Framework test execution across four environments — Predev, UAT, Preprod, Production — so every stage of the release pipeline had automated coverage.

The pattern I always follow is: fast feedback loops first — unit and API tests run on every commit, UI tests on merge to main, full regression on release candidates. And I always make sure test reporting is integrated — ReportPortal at Thomson Reuters, X-Ray at J&J — so the pipeline results are visible and actionable, not just green/red signals.

6. What is your experience with API and microservice test automation

API testing has been central to my work for the last several years, and I've approached it from a few different angles depending on the project's stack and team needs.

At Johnson & Johnson, I built a custom REST API keyword library on top of Robot Framework specifically for the Salesforce Global Services platform. This wasn't off-the-shelf Postman testing — it was a purpose-built library that allowed the team to write readable, business-language test cases against Salesforce REST APIs for Case Management, HR, Payroll, and Procurement workflows. By building it as a keyword library, I made it accessible to engineers who weren't deep Python developers, which accelerated test coverage across a 24-person team. I also implemented multilingual email automation for case lifecycle notifications — which itself required API-level testing of notification triggers and payload validation.

At Google, API automation was integrated into the pipeline validation framework — I was asserting data contracts at each stage of the geospatial ingestion pipeline, using BigQuery and AppScript alongside REST API calls. The integration with the Bug Tracking API was also REST-based — I built the Playwright framework to automatically raise structured defect tickets via API when test failures were detected.

For tooling specifically: I've used REST Assured for Java-based projects, the Robot Framework REST Library for keyword-driven API suites, Postman and Newman for exploratory and regression collections in CI, and SoapUI for legacy SOAP-based services at earlier clients.

My approach to microservice testing specifically is to test at the contract level first — validate that each service's API contract is stable before running integration tests — and to isolate service dependencies using mocking where needed so tests aren't brittle against infrastructure variability. I find that combining contract-level API tests with targeted end-to-end smoke tests gives the best coverage-to-maintenance ratio for microservice architectures.

All 6 answers are above — click any question to expand it, and use the copy button to grab each answer cleanly.

A few notes on how these were calibrated:

**Q1 (Tell me about yourself)** is kept to roughly 90 seconds spoken — it opens with scope, moves through domain breadth, lands on measurable impact, and closes with a forward-looking hook toward Cubic3. Add a specific line about why Cubic3 if you know their product domain.

**Q2 (Complex framework)** uses full STAR structure with Google Street View as the primary example — it's the strongest story given the 300+ workflows, BigQuery ETL validation, and the hard metrics (99.8%, 100+ hours, 18-hour reduction). J&J gets a natural mention in the Result section to show range.

**Q3 (AI in automation)** deliberately reframes the question — instead of "AI tools that help me test", it positions Shajeeb as someone who tests AI systems themselves. This is a strong differentiator and likely unlike anything else Samantha will hear.

**Q4 (Quality culture)** leads with the In-Sprint philosophy from J&J, uses the tooling approach as a mentoring mechanism, and ends with a litmus test that shows strategic maturity.

**Q5 (CI/CD)** covers all three platforms from the CV with concrete examples, and articulates a clear philosophy around hard-blocking vs. reporting.

**Q6 (API/microservice)** ends with a contract-testing approach to microservices — a forward-looking answer that goes beyond tool names.

> What is a test strategy?

This is a broad but important question — and one that could come up at Cubic3. Let me give you both a strong conceptual answer and a tailored version Shajeeb can deliver in the interview.

* * *

## What is a test strategy?

A test strategy is a **high-level document and set of decisions** that defines _how_ quality will be achieved across a project or product. It sits above a test plan — it answers the "why and how" rather than the "what and when."

It covers:

  * What types of testing will be done and why
  * What will and won't be automated
  * How testing fits into the delivery lifecycle (CI/CD, sprints, releases)
  * Tools, frameworks, environments, and data
  * Risk-based prioritisation — where do failures hurt most?
  * Entry/exit criteria and quality gates
  * Roles, responsibilities, and coverage ownership



* * *

#### How you should answer this in the Cubic3 interview

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-15.png)

The widget above covers both the conceptual framework and a copy-ready interview answer. Click each phase to expand it, and use the star section at the bottom to grab Shajeeb's full spoken answer.

A few things worth noting about how this question typically lands in interviews:

It can be asked in two very different ways. "What is a test strategy?" is a knowledge question — the interviewer wants to see if you can articulate the difference between a strategy (the why and how) and a test plan (the what and when). "How would you design one?" is a behavioural question — they want to hear your process and see evidence you've actually done it.

The strongest answer does both: starts with a crisp definition, then immediately grounds it in real delivery experience. The interview answer in the widget does exactly that — it moves through all six pillars using Shajeeb's actual projects (Google, J&J, Refinitiv) so it's concrete and credible rather than theoretical.

The key differentiator Samantha is likely listening for is whether Shajeeb positions quality as a blocker at the end of delivery, or as something embedded from the start — the shift-left mindset. The answer makes that clear throughout.
