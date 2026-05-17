---
title: "API Testing"
date: 2026-03-30 16:24:22 
author: shajeebhameed
layout: page
slug: api-testing
status: publish
---

Here's a comprehensive deep-dive on API testing questions and answers tailored to the iCIMS role:

* * *

## 🔌 API Testing — Complete Question & Answer Bank

* * *

### **SECTION 1: REST Fundamentals**

* * *

**Q1: What is REST and what are its core architectural constraints?**

**A:** REST (Representational State Transfer) is an architectural style, not a protocol. Its six constraints are:

  * **Statelessness** — each request must contain all information needed; the server holds no session state between requests.
  * **Client-Server separation** — UI and data storage concerns are separated, improving portability and scalability.
  * **Uniform Interface** — resources are identified by URIs, manipulated through representations, and self-descriptive messages are used.
  * **Cacheability** — responses must define themselves as cacheable or non-cacheable.
  * **Layered System** — clients don't need to know if they're talking to the actual server or an intermediary (load balancer, cache).
  * **Code on Demand (optional)** — servers can extend client functionality by transferring executable code.



In testing, understanding these constraints helps you design better tests — for example, testing that the server truly is stateless by deliberately omitting auth tokens between calls.

* * *

**Q2: What are the HTTP verbs and what does each one do?**

**A:**

Verb| Purpose| Idempotent?| Safe?  
---|---|---|---  
**GET**|  Retrieve a resource| ✅ Yes| ✅ Yes  
**POST**|  Create a new resource| ❌ No| ❌ No  
**PUT**|  Replace a resource entirely| ✅ Yes| ❌ No  
**PATCH**|  Partially update a resource| ❌ No*| ❌ No  
**DELETE**|  Remove a resource| ✅ Yes| ❌ No  
**HEAD**|  Same as GET but no body returned| ✅ Yes| ✅ Yes  
**OPTIONS**|  Returns allowed methods for a resource| ✅ Yes| ✅ Yes  
  
*PATCH can be made idempotent depending on implementation.

**Key testing points:**

  * GET should never mutate data — test that it doesn't.
  * PUT vs PATCH: PUT replaces the whole object (missing fields go null), PATCH only updates specified fields. Test both behaviors.
  * Calling DELETE twice should return 404 on the second call (idempotent in effect, not necessarily in response).



* * *

**Q3: What is idempotency and why does it matter for API testing?**

**A:** An operation is idempotent if making the same request multiple times produces the same result as making it once. GET, PUT, and DELETE are idempotent; POST generally is not.

**Why it matters in testing:**

  * Retry logic in distributed systems depends on idempotency — if a network failure occurs and a client retries a PUT, it shouldn't create duplicates.
  * Test idempotency explicitly: call PUT or DELETE twice with the same payload and assert the resource state is identical after both calls.
  * POST creating two records when called twice is expected behavior — but worth documenting as a test case to ensure retry logic at the client level handles it correctly.



* * *

**Q4: Explain the difference between PUT and PATCH with a test example.**

**A:**

Given a user resource:
    
    
    { "id": 1, "name": "Alice", "email": "alice@example.com", "role": "admin" }
    
    

**PUT request** with body `{ "name": "Alice Updated" }`: → Replaces the entire resource. Result: `{ "id": 1, "name": "Alice Updated", "email": null, "role": null }`

**PATCH request** with body `{ "name": "Alice Updated" }`: → Partially updates. Result: `{ "id": 1, "name": "Alice Updated", "email": "alice@example.com", "role": "admin" }`

**Test cases to write:**

  * PUT with a partial body — assert missing fields are nulled out or return a 400 validation error.
  * PATCH with a single field — assert only that field changed.
  * PATCH with an unknown field — assert it returns 400 or is ignored (depends on API contract).



* * *

### **SECTION 2: HTTP Status Codes**

* * *

**Q5: What are the key HTTP status code ranges and what does each mean?**

**A:**

Range| Category| Common Examples  
---|---|---  
**1xx**|  Informational| 100 Continue, 101 Switching Protocols  
**2xx**|  Success| 200 OK, 201 Created, 204 No Content  
**3xx**|  Redirection| 301 Moved Permanently, 304 Not Modified  
**4xx**|  Client Error| 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable Entity, 429 Too Many Requests  
**5xx**|  Server Error| 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout  
  
* * *

**Q6: What's the difference between 400, 401, 403, and 404? Give test scenarios for each.**

**A:**

**400 Bad Request** — The request is malformed or fails validation.

  * Test: Send a POST with a missing required field → expect 400 with a descriptive error message.
  * Test: Send invalid data types (string where integer expected) → expect 400.



**401 Unauthorized** — Authentication is missing or invalid.

  * Test: Call a protected endpoint with no token → expect 401.
  * Test: Call with an expired JWT token → expect 401.
  * Test: Call with a malformed token → expect 401.



**403 Forbidden** — Authenticated but not authorized.

  * Test: Log in as a read-only user and call a DELETE endpoint → expect 403.
  * Test: Attempt to access another user's resource → expect 403 (not 404, to avoid leaking resource existence in some designs).



**404 Not Found** — Resource doesn't exist.

  * Test: GET `/users/99999` where that ID doesn't exist → expect 404.
  * Test: After a DELETE, GET the same resource → expect 404.



* * *

**Q7: When would you expect a 409 Conflict vs a 422 Unprocessable Entity?**

**A:**

**409 Conflict** — The request conflicts with the current state of the resource.

  * Example: Attempting to create a user with an email that already exists → 409.
  * Example: Trying to update a record that has been modified since you last fetched it (optimistic locking) → 409.



**422 Unprocessable Entity** — The request is syntactically correct but semantically invalid.

  * Example: A date range where the end date is before the start date.
  * Example: A field value that is a valid string but not within allowed enum values.



**Test angle:** Many APIs use these inconsistently. Part of your job as an SDET is to test that the API uses the correct status code for each scenario and document deviations as defects.

* * *

**Q8: What should you test around 5xx errors?**

**A:**

  * **500 Internal Server Error** — Should not expose stack traces or internal details in the response body. Test that error responses are sanitized.
  * **503 Service Unavailable** — Test that your client handles retry-after headers correctly.
  * **Chaos/fault injection testing** — Deliberately take a downstream dependency offline and assert the API returns a graceful 503/504, not an unhandled 500.
  * **Error response schema** — Even error responses should have a consistent schema. Test that 5xx responses still return JSON with a consistent `error` or `message` field, not raw HTML.



* * *

### **SECTION 3: Authentication & Authorization**

* * *

**Q9: How do you test JWT-based authentication comprehensively?**

**A:**

**Positive tests:**

  * Valid token returns 200 with correct data.
  * Token with correct scope/claims accesses the right resources.



**Negative tests:**

  * No token → 401.
  * Expired token → 401 (check the `exp` claim).
  * Malformed token (truncated, random string) → 401.
  * Token with invalid signature (tampered payload) → 401.
  * Token from a different environment (e.g., staging token on prod) → 401.



**Authorization (RBAC) tests:**

  * Admin token can access admin endpoints.
  * Standard user token on admin endpoints → 403.
  * User A's token cannot access User B's private resources → 403.



**Security tests:**

  * Algorithm confusion attack: send a token with `"alg": "none"` → must be rejected.
  * Decode the JWT payload and verify no sensitive data (passwords, PII) is stored in the claims.



* * *

**Q10: How do you handle API keys and OAuth2 in your test framework?**

**A:** I never hardcode credentials in test code. API keys and secrets are stored in environment variables or a secrets manager (Vault, AWS Secrets Manager) and injected at runtime. In CI/CD, they're stored as encrypted pipeline secrets.

For OAuth2 flows in automation:

  * I use the **client credentials flow** for service-to-service tests — it's the cleanest for automation as it doesn't require user interaction.
  * For user-facing flows, I either mock the OAuth provider in lower environments or use a dedicated test IdP with test user credentials.
  * I implement a token cache in the framework so tests don't re-authenticate on every single request, reducing test time and rate limit hits.



* * *

### **SECTION 4: API Test Design**

* * *

**Q11: What are the layers of API testing and what does each cover?**

**A:**

Layer| What It Tests| Tools  
---|---|---  
**Unit**|  Individual functions/methods| JUnit, Jest  
**Contract**|  Schema and interface agreements between services| Pact, JSON Schema  
**Integration**|  Real interactions between services in a test environment| RestAssured, Supertest  
**E2E**|  Full user journey spanning multiple APIs| Playwright, RestAssured  
**Performance**|  Throughput, latency under load| k6, JMeter, Gatling  
**Security**|  Auth, injection, exposure| OWASP ZAP, Burp Suite  
  
A mature API testing strategy covers all layers, with the bulk of coverage at the integration level (following the testing pyramid principle).

* * *

**Q12: How do you approach testing API pagination?**

**A:**

  * Test the first page: verify correct number of items returned, correct `page` and `pageSize` values in metadata.
  * Test the last page: verify it returns fewer items than `pageSize` and that `hasNextPage` or `nextCursor` is absent or null.
  * Test boundary: `pageSize=1` — verify only one item returned.
  * Test invalid params: `page=-1`, `pageSize=0`, `pageSize=9999` (beyond max) — expect 400 with clear error.
  * Test sort consistency: same items should not appear on multiple pages when paginating through the full dataset.
  * Test with filters applied: pagination should respect filters — page 2 of a filtered result set.



* * *

**Q13: How do you test API versioning?**

**A:**

  * **v1 still works** — after introducing v2, regression test that v1 endpoints return the same responses as before.
  * **Breaking changes are in the new version only** — a field removed in v2 should still be present in v1.
  * **Deprecation headers** — check that deprecated endpoints return a `Deprecation` or `Sunset` header.
  * **Version in URL vs header** — if the API supports both, test both routing mechanisms.
  * **Default version behavior** — what happens with no version specified? Document and test it.



* * *

**Q14: How do you test error response bodies, not just status codes?**

**A:** Status codes alone aren't enough. I validate the full error response schema:
    
    
    {
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "email is required",
        "field": "email",
        "requestId": "abc-123"
      }
    }
    
    

**What I check:**

  * Error message is human-readable and actionable — not "null pointer exception."
  * Machine-readable error code is present for client-side handling.
  * No internal stack traces or sensitive system info leaked.
  * `requestId` or correlation ID is present for traceability.
  * Consistent schema across all error types — inconsistent error formats are a defect.



* * *

### **SECTION 5: Tools & Frameworks**

* * *

**Q15: How do you structure an API test automation framework?**

**A:** My framework typically has these layers:
    
    
    /framework
      /clients          ← HTTP client wrappers (auth, base URL, retries)
      /endpoints        ← One class per API resource (UserAPI, JobAPI)
      /models           ← Request/response POJOs or data classes
      /data             ← Test data factories and builders
      /tests            ← Actual test classes
      /config           ← Environment configs (dev, staging, prod)
      /utils            ← Schema validators, assertion helpers
      /reports          ← Allure or ExtentReports integration
    
    

**Key design principles:**

  * Tests read like specifications — `userApi.createUser(payload).assertStatus(201)`.
  * No hardcoded URLs or credentials.
  * Each test is fully independent — no shared state.
  * Schema validation on every response, not just happy path tests.



* * *

**Q16: Compare RestAssured, Postman/Newman, and k6 — when do you use each?**

**A:**

**RestAssured (Java):**

  * Best for: Integration test suites that need to live in the same Java codebase as the application.
  * Strengths: Fluent DSL, BDD syntax, deep JUnit/TestNG integration, excellent for complex assertion chains.
  * Weakness: Verbose for simple tests; requires Java knowledge.



**Postman/Newman:**

  * Best for: Quick exploratory testing, sharing test collections with non-engineers, contract documentation.
  * Strengths: Visual interface, easy for developers to write quick smoke tests, Newman runs in CI.
  * Weakness: Collections can become unmaintainable at scale; limited code reuse; not ideal for complex test logic.



**k6:**

  * Best for: Performance and load testing of APIs.
  * Strengths: JavaScript-based scripts, excellent metrics output, designed for CI integration, scales well.
  * Weakness: Not designed for functional test assertions.



**My approach at iCIMS:** RestAssured or a similar code-first framework for the core integration test suite in CI, Postman collections for developer-facing documentation and smoke tests, and k6 for performance regression tests.

* * *

**Q17: How do you implement schema validation in your API tests?**

**A:** I use JSON Schema validation on every API response, not just the happy path. The schema defines required fields, data types, formats, and allowed values.
    
    
    // RestAssured example
    given()
      .auth().oauth2(token)
    .when()
      .get("/api/v1/jobs/123")
    .then()
      .statusCode(200)
      .body(matchesJsonSchemaInClasspath("schemas/job-response.json"));
    
    

**What schema validation catches that assertion-based tests miss:**

  * New fields added to response that break downstream consumers.
  * Type changes (a field changing from integer to string).
  * Nullable fields that should never be null.
  * Fields going missing in edge case responses.



I store schemas in version control alongside test code and update them intentionally — schema changes trigger a review.

* * *

### **SECTION 6: Advanced API Testing**

* * *

**Q18: What is contract testing and how does Pact work?**

**A:** Contract testing verifies that a consumer and provider agree on the interface between them, without requiring both to be running simultaneously.

**How Pact works:**

  1. The **consumer** writes a test that defines what request it will send and what response it expects → this generates a "pact file" (JSON contract).
  2. The **provider** runs the pact file against its actual implementation to verify it can fulfill the contract.
  3. Both sides publish results to a **Pact Broker**.



**Why it's better than integration testing alone:**

  * Catches breaking API changes before deployment, not after.
  * No shared test environment needed.
  * Each team owns their side of the contract.



**At iCIMS** , with multiple services interacting, contract testing between microservices would prevent the classic "it worked in isolation but broke in integration" problem.

* * *

**Q19: How do you test API rate limiting?**

**A:**

  * Send requests up to the rate limit threshold and verify 200 responses.
  * Send one request over the limit and verify 429 Too Many Requests.
  * Verify the response includes `Retry-After` or `X-RateLimit-Remaining` headers.
  * Wait for the rate limit window to reset and verify requests succeed again.
  * Test different limits for different user tiers (free vs. premium) if applicable.



**In automation** , I use a controlled loop with a counter and timestamps, being careful not to run rate limit tests in parallel with other tests since they can interfere.

* * *

**Q20: How do you test API responses for security vulnerabilities?**

**A:**

**Injection:**

  * Send SQL injection strings in query params and request bodies → expect 400, never a 500 or unexpected data returned.
  * Send XSS payloads in string fields → response should return them escaped, not executable.



**Data exposure:**

  * Verify sensitive fields (passwords, SSNs, full card numbers) are never returned in responses — even in error messages.
  * Check that internal IDs or implementation details aren't exposed.



**IDOR (Insecure Direct Object Reference):**

  * As User A, try to GET/PUT/DELETE resources belonging to User B using their IDs → must return 403 or 404.
  * Enumerate sequential IDs to check if other users' data is accessible.



**Mass assignment:**

  * Send extra fields in POST/PUT body (e.g., `"role": "admin"`) → verify they are ignored and don't elevate privileges.



* * *

**Q21: How do you handle testing in a microservices architecture where services depend on each other?**

**A:** I use a combination of strategies:

  * **Mocking/Stubbing** — Use WireMock or MockServer to stub downstream services in unit and integration tests. This isolates the service under test and removes flakiness from network dependencies.
  * **Contract tests** — As described above with Pact, to verify the stubs reflect the real service behavior.
  * **Component tests** — Test a single service with all its real internal components but with external dependencies stubbed.
  * **End-to-end tests** — A smaller, targeted suite that runs against a full deployed environment, covering the most critical cross-service journeys only.



The mistake I've seen is teams relying entirely on E2E tests for coverage — they become slow, flaky, and hard to maintain. The bulk of coverage should be at the component and contract level.

* * *

**Q22: How do you test asynchronous APIs (webhooks, event-driven, message queues)?**

**A:** Async APIs require a different approach since there's no immediate response to assert on.

**For webhooks:**

  * Stand up a test listener endpoint (or use a tool like Webhook.site in dev).
  * Trigger the event via the API.
  * Poll the listener for the expected webhook payload within a timeout.
  * Assert the payload schema, event type, and data correctness.



**For message queues (Kafka, SQS):**

  * Consume from the test topic/queue after triggering the action.
  * Assert message schema and content.
  * Test dead letter queue behavior by sending malformed messages.
  * Test ordering guarantees if the API contract requires it.



**Key principle:** Always assert eventual consistency with a reasonable timeout and clear failure messaging when the timeout is hit — not just an indefinite wait.

* * *

**Q23: How do you approach API performance testing specifically?**

**A:**

**Baseline test:** Single user, measure p50/p95/p99 latency and establish a baseline.

**Load test:** Ramp up to expected concurrent users. Verify latency stays within SLA and error rate stays below threshold (typically <1%).

**Stress test:** Push beyond expected load to find the breaking point. Identify which resource saturates first — CPU, DB connections, memory.

**Spike test:** Sudden burst of traffic (simulating a viral moment or batch job). Measures recovery time.

**Soak test:** Sustained load over hours. Catches memory leaks and connection pool exhaustion.

**Key metrics I track:**

  * Response time (p50, p95, p99) — not just average, which hides tail latency.
  * Throughput (requests per second).
  * Error rate.
  * Apdex score.
  * Infrastructure metrics correlated with test timeline (CPU, memory, DB query time).



* * *

### **SECTION 7: iCIMS-Specific Scenarios**

* * *

**Q24: iCIMS is a talent acquisition platform. What specific API testing considerations would you have for an ATS system?**

**A:**

  * **Candidate data privacy** — APIs handling PII (name, email, CV content) must be tested for proper data masking, retention policies, and GDPR compliance. Test that candidate data isn't leaked across employer accounts.
  * **Multi-tenancy isolation** — Employer A must never see Employer B's candidates or jobs. I'd extensively test tenant boundary enforcement at the API level.
  * **High-volume scenarios** — Job applications can spike significantly (a popular company posting a role). Performance test the application submission API under burst load.
  * **Integration ecosystem** — iCIMS integrates with many HRIS and background check providers. Test that webhook deliveries to partner systems are reliable and retried on failure.
  * **Workflow state machines** — Candidate pipeline stages (applied → screened → interviewed → offered) are state machine logic. Test all valid and invalid state transitions via the API.



* * *

**Q25: How would you test an AI-powered candidate matching API?**

**A:** This is directly relevant to iCIMS's AI capabilities. I'd approach it at multiple levels:

  * **Functional correctness** — Does a candidate with all matching skills rank higher than one with partial skills? Use carefully constructed test data with known expected rankings.
  * **Bias testing** — Verify that protected characteristics (age, gender, ethnicity inferred from names) don't influence match scores. Run identical profiles with only names changed.
  * **Consistency testing** — The same candidate and job should produce the same score (or within an acceptable variance) across multiple calls.
  * **Boundary testing** — What happens with a job description with no skills listed? A candidate with an empty profile? An extremely long job description?
  * **Latency** — AI inference can be slow. Test that the API meets its SLA and degrades gracefully under load, potentially returning cached or approximate results rather than timing out.



* * *

## 💡 Quick Reference Cheat Sheet

**Verbs at a glance:**

  * **GET** = Read (safe, idempotent)
  * **POST** = Create (neither)
  * **PUT** = Full replace (idempotent)
  * **PATCH** = Partial update (not inherently idempotent)
  * **DELETE** = Remove (idempotent)
  * **OPTIONS** = What can I do here? (safe, idempotent)
  * **HEAD** = GET without body (safe, idempotent)



**Status codes to know cold:**

  * 200 OK, 201 Created, 204 No Content
  * 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found
  * 409 Conflict, 422 Unprocessable, 429 Too Many Requests
  * 500 Server Error, 503 Unavailable, 504 Gateway Timeout



* * *
