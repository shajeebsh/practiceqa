---
title: "iCIMS QA with Star"
date: 2026-03-31 18:20:09 
author: shajeebhameed
layout: page
slug: icims-qa-with-star__trashed
status: trash
---

Here's the complete interview Q&A for the iCIMS Sr. Engineer, Quality role — with full STAR format applied to every behavioural question.

* * *

# 🎯 iCIMS Sr. Engineer, Quality — Complete Interview Q&A with STAR Framework

* * *

## 📋 SECTION 1: TELL ME ABOUT YOURSELF

**Q: Walk me through your background and why you're a fit for this role.**

**A:**

> "I'm a Senior QA/SDET Engineer with [X] years of experience spanning test automation, framework design, and quality leadership. I started my career in software engineering which gave me a strong foundation in scalable system design before specialising in quality engineering.

> Over the past [X] years I've worked as an SDET, building automation frameworks from scratch using Selenium and Playwright, and leading quality across multiple agile teams. I've driven CI/CD integration, performance testing, and security test automation, and more recently I've been working with AI integrations — specifically testing RAG-based systems and agentic AI features.

> On the leadership side, I've served as a technical lead for quality teams, mentoring engineers, running code reviews, and being the voice of quality in sprint planning and architecture discussions.

> iCIMS is particularly exciting to me because of its position as a Talent Cloud platform with a strategic AI hub here in Dublin. The combination of complex enterprise SaaS, AI integration testing, and the cross-functional leadership aspects of this role is exactly where I want to grow."

* * *

## 📋 SECTION 2: IN-SPRINT TESTING

* * *

**Q1: How do you design and implement a test plan as part of an in-sprint testing team?**

**A:**

> "My in-sprint test planning starts at the very beginning of the sprint, not at the end. During sprint planning I review user stories and acceptance criteria with the product owner to understand what 'done' means. I identify test scenarios across happy paths, edge cases, negative cases, and non-functional considerations.

> I categorise tests by type — which ones need automation, which need manual exploratory testing, and which need performance or security coverage. I document these in a structured test plan linked to the story in Jira.

> As development progresses, I test features as they become available rather than waiting for the end of the sprint, giving engineers timely feedback. By sprint end I have both manual test sign-off and automated tests committed to the repo before release."

**STAR Example:**

  * **Situation:** My team was struggling with last-minute quality issues causing sprint carry-over.
  * **Task:** I needed to redesign how we approached in-sprint testing to catch issues earlier.
  * **Action:** I introduced a shift-left approach — joining refinement sessions to flag testability gaps, writing test cases in parallel with development, and testing features within 24 hours of them being marked ready for QA.
  * **Result:** Sprint carry-over due to quality issues dropped by 60% over three sprints, and the team's confidence in release quality significantly improved.



* * *

**Q2: How do you make a Go/No-Go release decision?**

**A:**

> "A Go/No-Go decision is a structured assessment, not a gut feeling. I look at: automated test pass rate (I target above 95%), severity distribution of open defects (any P1 or P2 is an automatic No-Go), coverage of acceptance criteria, exploratory testing results, and performance benchmark comparisons.

> I also assess rollback risk — if something goes wrong post-deployment, how quickly and cleanly can we revert? I document my decision with evidence and communicate it clearly to the product owner and engineering manager."

**STAR Example:**

  * **Situation:** We had a release scheduled for Friday with a known intermittent defect that engineering had marked as low risk.
  * **Task:** I needed to make the Go/No-Go call with incomplete information and pressure to ship.
  * **Action:** I ran additional exploratory tests specifically targeting the defect's known trigger conditions. I found it was reproducible in 3 out of 10 attempts in a specific browser/locale combination that matched a significant portion of our customer base. I escalated with a documented risk analysis.
  * **Result:** The release was held, the defect was fixed over the weekend, and we shipped Monday. The product owner later confirmed the fix prevented what would have been a P1 incident for a major customer.



* * *

**Q3: How do you approach post-deployment checkout?**

**A:**

> "Post-deployment checkout is a targeted smoke test focused on the specific features deployed, not a full regression run. I maintain a deployment checkout script that covers critical user journeys affected by the release. I run this immediately after deployment to production, validate key integrations are responding, and confirm monitoring alerts are not firing unexpectedly.

> I also verify environment-specific configuration — things that work in staging can fail in production due to environment differences. I complete the checkout within 30 minutes and give a clear green or escalation signal to the team."

* * *

## 📋 SECTION 3: AUTOMATION FRAMEWORK

* * *

**Q4: Describe an automation framework you've built or significantly improved.**

**STAR Answer:**

  * **Situation:** I joined a team where the existing Selenium framework had 65% flakiness, took 3.5 hours to run, and engineers had stopped trusting it. Tests were being skipped rather than fixed.
  * **Task:** My mandate was to make the automation suite reliable enough to be used as a real CI gate within one quarter.
  * **Action:** I audited the full suite. The main issues were hardcoded sleeps, shared mutable test data causing race conditions between parallel tests, and no retry or quarantine mechanism. I refactored to use explicit waits throughout, introduced a test data factory that created isolated data per test, implemented parallel execution with 8 agents, added Allure reporting, and created a quarantine tag system for genuinely flaky tests so they didn't block CI while being fixed.
  * **Result:** Flakiness dropped from 65% to under 4% in 8 weeks. Runtime went from 3.5 hours to 38 minutes. Engineer confidence returned — the suite went from being ignored to being treated as a genuine quality gate.



* * *

**Q5: How do you ensure automation scripts meet reliability, performance, and maintenance requirements?**

**A:**

> "I treat automation code like production code — it goes through code review, has coding standards, and is measured against quality metrics.

> For reliability: I enforce explicit waits over hard sleeps, test isolation (no shared state), and I track flakiness rates per test in CI. Any test above 5% flakiness gets quarantined and investigated.

> For performance: I measure suite execution time as a pipeline metric. If the suite grows beyond acceptable CI time, I introduce parallelism or split into tiers — fast unit/API tests on every commit, slower E2E nightly.

> For maintenance: I use Page Object Model for UI tests and Service Object pattern for API tests, separating locators and business logic from test assertions. When the UI changes, only one place needs updating."

* * *

**Q6: How do you approach code reviews for other engineers' test code?**

**A:**

> "In code reviews I look for several things beyond 'does it work'. I check for test independence — does this test rely on state from another test? I look for meaningful assertions — is the test actually verifying the right thing, or just that no exception was thrown? I check for hardcoded values that should be configuration, and overly complex test logic that makes the test hard to understand.

> I also look for missing negative and edge case coverage — engineers often test only the happy path. I give written feedback that explains the 'why' not just the 'what', so it's a learning moment, not just a correction."

**STAR Example:**

  * **Situation:** A junior engineer submitted a PR with 15 new test cases that all passed CI but had a fundamental structural problem — they all depended on running in a specific order.
  * **Task:** I needed to give feedback that was constructive and educational, not just rejecting the PR.
  * **Action:** I wrote a detailed code review comment explaining test independence, provided a refactored example for one test showing the pattern, and offered a 30-minute pairing session to walk through the rest together.
  * **Result:** The engineer refactored all 15 tests correctly, and in subsequent PRs demonstrated they had internalised the principle without needing further coaching on it.



* * *

## 📋 SECTION 4: WEB TECHNOLOGIES & API TESTING

* * *

**Q7: Describe your experience with RESTful service testing. How do you approach it comprehensively?**

**A:**

> "I test REST APIs at multiple layers. At the contract layer I validate that request and response schemas match the OpenAPI specification using JSON Schema validation on every response. At the functional layer I cover all HTTP verbs, status codes, auth boundaries, error handling, and pagination. At the integration layer I test real service interactions in a deployed environment.

> For tools I use RestAssured in Java/Groovy stacks or Playwright's API testing capability for JavaScript projects. I also validate negative cases systematically — invalid tokens, malformed payloads, missing required fields, boundary values, and concurrent requests."

**STAR Example:**

  * **Situation:** Our API test suite only covered happy path scenarios and we were consistently getting production bugs on error handling edge cases.
  * **Task:** I needed to redesign the API test strategy to catch these failures before production.
  * **Action:** I introduced a systematic negative test matrix — for each endpoint I mapped every validation rule, auth requirement, and error condition to a test case. I also added JSON Schema validation on every response so structural regressions were caught automatically.
  * **Result:** Error-handling defects found in production dropped by 80% over the following two releases. The schema validation alone caught three breaking API changes before they reached staging.



* * *

**Q8: What web technologies are you proficient in and how does that help your testing?**

**A:**

> "My proficiency across HTML, CSS, JavaScript, AJAX, XML, JSON, and RESTful services directly improves my testing in several ways.

> Understanding JavaScript means I can write robust Playwright tests, debug async timing issues, and understand why a UI element behaves inconsistently. Understanding AJAX means I know when to wait for network responses versus DOM changes — critical for avoiding flaky waits.

> Understanding JSON and RESTful conventions means I can spot when an API is returning a wrong status code, a malformed response, or using incorrect HTTP semantics. I can read network traffic in DevTools and translate what I see directly into test assertions.

> This technical depth also makes me a more credible reviewer in architecture and code discussions — I can participate meaningfully, not just flag testability concerns from the outside."

* * *

## 📋 SECTION 5: CI/CD & AGILE

* * *

**Q9: How have you embedded quality into a CI/CD pipeline?**

**A:**

> "Quality in CI/CD is about making the right checks happen automatically at the right stage. My typical pipeline structure is: on every commit, fast unit and API tests run in under 5 minutes. On PR merge, the full integration suite runs as a quality gate — a failing suite blocks the merge. On deployment to staging, E2E regression and smoke tests run. On production deployment, a targeted checkout script runs immediately.

> I also integrate security scanning (SonarQube for SAST), dependency vulnerability checks, and test coverage thresholds as pipeline gates. I publish test results to dashboards so quality metrics are visible to the whole team, not buried in CI logs."

**STAR Example:**

  * **Situation:** Our team was doing manual QA sign-off before every deployment, which was creating a bottleneck — deployments were being queued and engineers were losing hours waiting.
  * **Task:** I needed to automate enough of the quality gate to remove the manual bottleneck without sacrificing safety.
  * **Action:** I built a tiered pipeline — a fast API smoke test suite (3 minutes) ran on every commit, and a fuller E2E suite (25 minutes) ran on PR merge. I also introduced automated deployment checkout so post-deploy validation happened without manual intervention.
  * **Result:** Deployment frequency increased from twice a week to daily. Manual QA time was reduced by 70%, and the team redirected that time toward exploratory testing and new feature coverage.



* * *

**Q10: How do you work in an Agile/Lean environment as a QA engineer?**

**A:**

> "In Agile I'm an active sprint participant, not a gatekeeper at the end. I attend story refinement to flag testability concerns before development starts. I write acceptance criteria alongside the product owner. I test features within the sprint as they're completed rather than doing a big-bang test at the end.

> In a Lean context I apply WIP limits to my own test work — I don't start testing five stories simultaneously. I focus on flow and use cycle time as my personal metric: how long does it take from 'story ready for test' to 'story tested and done'? Reducing that cycle time is how I contribute to delivery speed without sacrificing quality."

* * *

## 📋 SECTION 6: PERFORMANCE TESTING

* * *

**Q11: Describe your experience with performance testing. What tools have you used?**

**A:**

> "I've used JMeter, k6, and Gatling across different projects. My approach to performance testing follows four test types: load testing at expected concurrency to verify SLA compliance, stress testing beyond expected load to find breaking points, spike testing for sudden burst scenarios, and soak testing over hours to catch memory leaks.

> I report on p50, p95, and p99 response times rather than averages, because averages hide tail latency. I correlate test results with infrastructure metrics — CPU, memory, DB connection pool — to identify root causes rather than just symptoms."

**STAR Example:**

  * **Situation:** Our application was performing well in functional testing but customers were reporting slowness during peak hiring periods — typically Monday mornings when companies post jobs for the week.
  * **Task:** I needed to reproduce the performance issue in a controlled test environment and identify the root cause.
  * **Action:** I built a JMeter load profile that modelled the Monday morning traffic spike — a sudden burst to 5x normal concurrency over 10 minutes. Running this against staging immediately reproduced the issue. Correlating with infrastructure metrics I found the database connection pool was exhausting at 3x load, creating a queue that cascaded into timeouts.
  * **Result:** The team increased the connection pool size and added query caching for the most expensive read operations. Re-testing confirmed p95 response times dropped from 8 seconds to 420ms at peak load. The fix shipped before the next peak period.



* * *

## 📋 SECTION 7: SECURITY TESTING

* * *

**Q12: How do you evaluate security defects using automated tools?**

**A:**

> "I integrate security testing at multiple pipeline stages. For static analysis I use SonarQube or Checkmarx in the build pipeline to catch code-level vulnerabilities — SQL injection risks, hardcoded secrets, insecure dependencies. For dynamic testing I use OWASP ZAP against running test environments to scan API endpoints and web interfaces. For dependency scanning I use OWASP Dependency Check or Snyk to flag vulnerable third-party libraries.

> Beyond tooling, I maintain a manual security test checklist covering the OWASP Top 10 — auth bypass, IDOR, mass assignment, sensitive data exposure, and injection. I treat security defects with the same severity classification system as functional defects, and P1 security findings are automatic release blockers."

**STAR Example:**

  * **Situation:** Our team had no formal security testing process — security was tested ad-hoc before major releases only.
  * **Task:** As QA lead I was asked to introduce automated security testing without significantly increasing pipeline time.
  * **Action:** I integrated SonarQube into the CI build (added 2 minutes to pipeline time), configured OWASP Dependency Check to run nightly rather than on every commit to keep CI fast, and set up OWASP ZAP in a weekly scheduled scan against staging. I also wrote a security test checklist based on OWASP Top 10 for manual review on new features.
  * **Result:** In the first month, Dependency Check flagged a critical CVE in a third-party library that the team immediately patched. SonarQube caught two hardcoded API keys in a new feature branch before they merged. The security posture improved significantly with minimal pipeline overhead.



* * *

## 📋 SECTION 8: AI & RAG TESTING

* * *

**Q13: Describe your experience testing AI integrations with agentic AI or RAG models.**

**A:**

> "Testing AI systems requires a fundamentally different approach from traditional software because outputs are non-deterministic — the same input doesn't always produce the same output, so exact-match assertions don't work.

> For RAG systems I test at two layers. For retrieval quality I use metrics like context recall (did the right documents get retrieved?) and context precision (was the retrieved content relevant and not noisy?). For generation quality I use faithfulness scoring (is the answer grounded in the retrieved documents, or is the model hallucinating?) and answer relevance (does the response actually answer the question?).

> To handle non-determinism I use semantic similarity scoring rather than exact string matching, LLM-as-judge evaluation using a rubric, and statistical consistency testing — running the same question multiple times and measuring the pass rate.

> I maintain a golden dataset of curated question-answer pairs with expected source documents, which acts as a regression suite that I run after every prompt change or retrieval configuration update."

**STAR Example:**

  * **Situation:** Our product team shipped a new AI-powered candidate matching feature using a RAG pipeline. There was no test coverage for it — the team wasn't sure how to test something with variable outputs.
  * **Task:** I needed to design a test strategy from scratch for an AI system that no one on the team had tested before.
  * **Action:** I researched RAGAS and similar evaluation frameworks. I built a golden dataset of 75 test cases with known expected matches, covering both strong matches and edge cases (candidates with partial skill matches, candidates with identical profiles but different locations). I implemented faithfulness scoring using a secondary LLM call as judge, and set up the evaluation to run as part of the CI pipeline on every model config change. I also added adversarial tests — deliberately crafted queries designed to trigger hallucination.
  * **Result:** The evaluation framework caught two significant regressions before they reached production — one where a prompt change caused the model to start ignoring location constraints, and another where a retrieval config change caused it to surface irrelevant candidates with high confidence scores. The team credited the test framework with preventing two customer-facing incidents.



* * *

**Q14: How do you test for hallucination in AI systems?**

**A:**

> "Hallucination testing has two layers — automated and adversarial.

> For automated detection, I use faithfulness scoring: I decompose the generated answer into individual claims and verify each claim can be traced to a retrieved source document. Any claim without grounding is a potential hallucination. Tools like RAGAS do this automatically.

> For adversarial testing, I deliberately ask questions where the answer is not in the knowledge base, to verify the system says 'I don't know' rather than inventing an answer. I also test questions where the knowledge base contains related but not exactly matching information, to see if the model extrapolates incorrectly.

> The most dangerous failure mode I test for specifically is confident hallucination — where the model gives a wrong answer with high apparent certainty. I catch this by comparing the model's confidence signal against the faithfulness score and flagging divergence."

* * *

## 📋 SECTION 9: LEADERSHIP & VOICE OF QUALITY

* * *

**Q15: How have you acted as the "voice of the customer" within a sprint team?**

**A:**

> "I bring the customer's perspective into sprint conversations by staying connected to real customer feedback. I regularly review support tickets, incident reports, and customer-reported bugs to find patterns that inform where we need better test coverage."

**STAR Example:**

  * **Situation:** Our team was consistently hitting sprint velocity targets but customer satisfaction scores were declining due to recurring UX friction issues.
  * **Task:** I needed to identify why our tests were passing but customers were unhappy, and bridge that gap.
  * **Action:** I spent a week reviewing the last quarter of support tickets and categorised them by theme. I found that 40% of tickets related to a specific workflow that our tests covered functionally but never tested from a real user's perspective — we were testing that buttons worked, not that the workflow made sense. I brought this analysis to the team, reframed three existing stories to include usability acceptance criteria, and introduced persona-based exploratory testing sessions.
  * **Result:** Support tickets in that workflow category dropped by 55% over the following two months. The product owner started inviting me to customer feedback reviews as a standing item.



* * *

**Q16: How do you promote quality as a culture across teams that don't report to you?**

**A:**

> "Culture change through influence rather than authority requires three things: making quality visible, making good practices easy, and building relationships."

**STAR Example:**

  * **Situation:** At a previous company, quality was seen as the QA team's responsibility — developers would throw code over the fence and expect QA to find all the bugs.
  * **Task:** As QA lead I was asked to shift this culture without any direct authority over engineering teams.
  * **Action:** I took three approaches. First, I built a quality dashboard that showed defect escape rates, test coverage, and flakiness trends per team — making quality data visible created natural accountability. Second, I created shared test utilities and templates that made writing good tests the easy default for developers. Third, I ran monthly 'quality clinic' sessions — informal drop-ins where engineers could ask QA questions and learn test design techniques. I focused on making these genuinely useful rather than preachy.
  * **Result:** Within two quarters, developer-contributed test coverage increased by 35%. The defect escape rate across engineering dropped by 40%. More importantly, engineers started raising quality concerns in sprint planning themselves — the culture had genuinely shifted.



* * *

**Q17: Describe your experience leading quality teams technically.**

**STAR Answer:**

  * **Situation:** I inherited a QA team of four engineers with varying skill levels, no shared standards, and three different automation approaches across the same product suite.
  * **Task:** I needed to consolidate the approach, upskill the team, and establish technical leadership without demotivating experienced engineers.
  * **Action:** I started with a technical audit — mapping each engineer's strengths and gaps. I ran weekly technical sessions on framework design, API testing, and CI integration. I established a shared framework standard, but involved the team in the decisions so they had ownership of the outcome. I introduced code review for all test code, pairing junior and senior engineers to accelerate knowledge transfer. I also created a technical roadmap for the team showing where we were going and why.
  * **Result:** Within six months the team had consolidated onto a single framework, test coverage increased by 45%, and two engineers who had been junior received positive performance reviews noting their technical growth. One was promoted to a mid-level role.



* * *

## 📋 SECTION 10: DEFECT MANAGEMENT

* * *

**Q18: How do you create and document defects effectively?**

**A:**

> "A well-written defect report is a communication tool, not just a bug ticket. I follow a consistent structure: clear title describing the actual behaviour, environment and version details, precise steps to reproduce, expected behaviour, actual behaviour, severity and priority assessment, and supporting evidence — screenshots, logs, network traces.

> I write defect titles from the user's perspective — 'User cannot submit application form when CV attachment exceeds 5MB' rather than 'Upload button broken'. This helps product owners triage impact and helps developers understand the user context immediately."

**STAR Example:**

  * **Situation:** Our team had a defect backlog that product owners found impossible to prioritise — descriptions were vague, steps were missing, and severity was inconsistently applied.
  * **Task:** I needed to improve defect quality across the team without creating excessive overhead.
  * **Action:** I created a defect writing template in Jira with mandatory fields and example entries. I ran a 30-minute team session showing side-by-side comparisons of poor and well-written defects. I also introduced a peer review step — before a defect was assigned to engineering, another QA engineer confirmed it was reproducible and the report was complete.
  * **Result:** Developer-to-QA defect bounce-backs dropped by 70% — defects that came back because they couldn't be reproduced or lacked information nearly disappeared. Engineering satisfaction with QA improved noticeably in the next team retrospective.



* * *

## 📋 SECTION 11: CERTIFICATIONS & CONTINUOUS LEARNING

* * *

**Q19: What certifications do you hold or are you pursuing, and how do they apply to this role?**

**A:**

> "I currently hold [your certification] which covers [relevant area]. The iCIMS role specifically mentions ISTQB Advanced certifications and the newer CT-GenAI and CT-AI certifications, which I find particularly relevant given the AI integration testing aspects of this role.

> The ISTQB CT-GenAI certification covers testing AI systems, prompt testing, evaluation strategies for generative AI, and the unique challenges of non-deterministic outputs — which maps directly to the RAG and agentic AI testing requirements in this spec.

> Beyond formal certifications, I stay current through [specific activities — conferences, reading, communities]. The AI testing space in particular is moving so fast that continuous learning outside of formal certification is essential."

* * *

**Q20: Why iCIMS specifically, and why this role?**

**A:**

> "iCIMS sits at an interesting intersection — it's an enterprise SaaS platform at scale, it's building AI capabilities with a dedicated Dublin hub, and the quality engineering role here is genuinely senior and strategic, not just execution.

> The combination of in-sprint testing leadership, framework ownership, cross-functional quality culture work, and AI integration testing is exactly the breadth I'm looking for. Most QA roles are either too narrow — pure automation — or too abstract — pure process. This one spans the full spectrum.

> The Dublin AI hub angle is also compelling. Being part of a team specifically focused on AI development in Ireland means I'd be working on cutting-edge problems in AI quality engineering at the same time that the industry is figuring out what good looks like. That's a rare opportunity."

* * *

## 📋 SECTION 12: SITUATIONAL / CURVEBALL QUESTIONS

* * *

**Q21: Tell me about a time you disagreed with a product decision that you felt created quality risk.**

**STAR Answer:**

  * **Situation:** A product owner wanted to skip regression testing for a hotfix deployment to meet a customer deadline.
  * **Task:** I needed to express my concern professionally while respecting the business urgency.
  * **Action:** Rather than simply objecting, I quantified the risk. I identified the three areas most likely to be affected by the hotfix based on the code change, ran a targeted 45-minute regression on just those areas, and presented the results with a clear risk statement: "These three areas are clean. These two areas I haven't covered — here's the known risk if we ship without testing them." I gave the product owner the information to make an informed decision rather than making it for them.
  * **Result:** The product owner chose to ship, accepted the documented risk, and the deployment was clean. More importantly, my approach built trust — the team saw I could balance quality standards with business reality rather than being a blocker.



* * *

**Q22: Describe a time your automation caught a critical bug before it reached production.**

**STAR Answer:**

  * **Situation:** We were mid-sprint on a release that included a refactor of our candidate application submission flow.
  * **Task:** My E2E automation suite included tests for this flow as part of nightly regression.
  * **Action:** The nightly run flagged a failure in the application submission test — specifically, a candidate profile with a non-standard character in their name was causing a silent 500 error in the backend that the UI was masking with a generic success message. Without the automation, this would have looked like a successful submission to the user while the data was silently lost.
  * **Result:** The defect was caught 48 hours before the release was scheduled to deploy. Engineering fixed it the same day. Post-mortem analysis confirmed this would have affected approximately 8% of candidate profiles based on the name pattern. It would have been a critical P1 incident affecting data integrity.



* * *

**Q23: How do you handle pressure from stakeholders to reduce testing time to ship faster?**

**STAR Answer:**

  * **Situation:** Engineering and product were pushing to cut the regression cycle from 3 days to 1 day without reducing scope.
  * **Task:** I needed to respond to the business pressure with a solution rather than a flat refusal.
  * **Action:** Instead of defending the 3-day timeline, I analysed where the time was actually going. I found that 60% of the manual regression time was spent on areas that had strong automation coverage and hadn't had a defect in 6 months. I proposed a risk-based approach: automate the stable areas fully and focus manual time on the high-risk, recently changed areas. I also invested two weeks in improving parallel execution to cut automated suite time in half.
  * **Result:** We reduced total regression time to 1.5 days without reducing meaningful coverage. Stakeholders got closer to their target, the team got a sustainable process, and defect escape rate didn't increase. The solution was a better testing strategy, not just doing less testing.



* * *

## 📋 QUESTIONS TO ASK iCIMS

Always finish with strong questions. These show senior-level thinking:

  1. _"How is the Dublin AI hub structured — is the quality team embedded in product squads or is there a centralised QA function?"_
  2. _"What does the current automation framework look like, and what are the biggest pain points you're hoping this hire will address?"_
  3. _"How mature is the AI testing practice here — are you building it from scratch or expanding an existing approach?"_
  4. _"What does the 'voice of quality' look like in practice — how much influence does QA have in sprint planning and architecture decisions?"_
  5. _"What does success look like in the first 90 days for this role?"_



* * *

Would you like me to now run a live mock interview for either the iCIMS or Cubic³ role — where I ask questions one at a time and give you real-time feedback on your answers?
