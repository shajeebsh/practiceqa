---
title: "Failed_rejected IQ"
date: 2026-04-14 09:59:42 
author: shajeebhameed
layout: page
slug: failed_rejected
status: publish
---

Q1: "Qualities of a QA automation engineer?"

What they're really asking: Are you self-aware about what makes a great QA engineer? Do you embody those qualities?

Opening

A great QA automation engineer is, at their core, someone who thinks like a detective and builds like an engineer. Over my 11 years across Google, J&J, and LSEG, I've come to believe the qualities fall into three categories — technical, mindset, and collaboration.

Technical qualities

  * Strong programming fundamentals — not just writing tests, but writing clean, maintainable code. At Google I built a Playwright/Python framework that 6 engineers maintained across 300+ workflows. That only works if the code is readable.
  * Analytical thinking — the ability to decompose a complex feature into boundary conditions, edge cases, and failure modes. Financial software like AccountsIQ has real-world consequences if a calculation is wrong.
  * Tool versatility — knowing when to use UI automation, API testing, or direct database validation, and choosing the right tool for the job rather than defaulting to one approach.



Q2: "Types of issues a QA automation engineer solves?"

What they're really asking: Do you understand the full breadth of your role beyond 'running tests'?

Think time: 30 secTarget length: ~2 min

Opening

The issues a QA automation engineer solves fall into four distinct categories — functional correctness, non-functional quality, process efficiency, and framework health. Let me walk through each with examples from my own experience.

Functional & correctness issues

  * Regression defects — features breaking silently when unrelated code changes. At LSEG, a FX trading framework I built caught a pricing calculation regression that would have caused incorrect trade settlements across multiple currencies.
  * Data integrity issues — at Google, I automated BigQuery ETL pipeline validation that verified 99.8% accuracy in geospatial data transformations. Silent data corruption is the hardest type of bug to detect manually.
  * Cross-environment inconsistencies — behaviour that works in Dev but fails in UAT or Prod due to configuration drift. Our CI quality gates at Google automatically blocked deployments that failed environment-specific checks.



Non-functional issues

  * Performance degradation — I identified and resolved pipeline bottlenecks at Google that delivered a 25% latency reduction. Performance tests catch slowdowns before users report them.
  * Security vulnerabilities — injection risks, broken authentication flows, exposed API endpoints that bypass authorisation. Especially critical in financial software where data isolation between entities is a compliance requirement.
  * Accessibility and cross-browser inconsistencies — SaaS products used globally must work across Chromium, Firefox, and Safari.



Q3: "Security guidelines in QA automation?"

What they're really asking: Are you aware that automation itself can be a security risk? This is a sophisticated question — most candidates miss this angle.

Think time: 30 secTarget length: ~90 sec

Opening

Security in QA automation operates at two levels — the security of the automation framework itself, and the security aspects that the automation validates in the product. Both matter, and I've applied guidelines across both in my work.

Securing the framework itself

  * Never hardcode credentials — usernames, passwords, API keys, and tokens must live in environment variables or a secrets manager like Azure Key Vault or GitHub Secrets, never in source code. This is fundamental and I enforce it in every team I work with.
  * Secrets scanning in CI — automated checks that block PRs if credentials are accidentally committed. I configure this in GitHub Actions pipelines as a mandatory quality gate.
  * Test data isolation — production data must never be used in test environments. I always use synthetic, anonymised datasets that cannot expose real customer information, which is especially critical in financial software under GDPR.
  * Least-privilege test accounts — automation service accounts should have only the permissions needed to run tests, not admin access. This limits the blast radius if credentials are ever compromised.
  * Secure storage of test artifacts — screenshots, videos, and reports captured during testing may contain sensitive financial data. These should be stored in access-controlled locations with retention policies, not public CI artifact buckets.



Security testing the product

  * Authentication & authorisation testing — verifying that a user with Entity A access cannot view or modify Entity B data. In a multi-entity accounting product like AccountsIQ, this is a compliance-critical test.
  * Input validation — testing for SQL injection, XSS, and malformed inputs in forms and API endpoints. At Thomson Reuters I included negative security scenarios in every regression suite.
  * Session management — verifying that sessions expire correctly, tokens are invalidated on logout, and concurrent session rules are enforced.



Q4: "Understanding of generative AI and where it's being used?"

What they're really asking: Are you forward-thinking? Do you understand AI practically, not just theoretically? Can you apply it responsibly in QA?

Think time: 30 secTarget length: ~2 min

Opening

Generative AI refers to AI systems that can produce new content — text, code, images, data — by learning patterns from large training datasets. The most prominent examples are large language models like GPT-4 and Claude, as well as image generators like DALL-E and Midjourney. But from a QA engineering perspective, the most relevant applications are in the software development and testing lifecycle.

Where it's being used broadly

  * Software development — tools like GitHub Copilot use generative AI to suggest and complete code in real time. Developers are seeing 20–40% productivity improvements in boilerplate-heavy tasks.
  * Customer service — AI chatbots now handle first-line support at scale. AccountsIQ's own product category — SaaS finance — is seeing AI embedded directly into financial reporting, anomaly detection, and forecasting workflows.
  * Content and document generation — legal contracts, financial reports, marketing copy. Enterprises are using AI to draft first versions that humans then review and refine.
  * Healthcare and life sciences — AI is being used for drug discovery, medical imaging analysis, and clinical trial design. During my time on J&J projects I saw early adoption of AI-driven quality workflows.



Specifically in QA automation

  * Test case generation — LLMs can generate test scenarios from user stories or OpenAPI specifications. I've started using this to rapidly cover edge cases I might not have thought of, while applying human review for any financial logic tests where the AI's output must be verified carefully.
  * Self-healing locators — AI-powered tools can detect when a UI selector has broken and suggest an updated one, reducing maintenance overhead in fast-releasing SaaS products.
  * Intelligent test data generation — generating realistic but synthetic financial datasets that cover boundary conditions, without using real customer data.
  * Root cause analysis — AI tools integrated with CI pipelines can analyse failure patterns across test runs and surface probable root causes, reducing mean time to diagnosis.
