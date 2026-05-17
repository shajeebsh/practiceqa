---
title: "Test Lead iq"
date: 2014-07-14 10:09:46 
author: shajeebhameed
layout: page
slug: test-lead-iq
status: publish
---

* Source: https://www.youtube.com/watch?v=Ms4UuViEvqI

\----------------------------------------------------------------------------------  
  * Testing process for the new project(chandichaayan helped)
  * Initial steps will be to identify Scope, Identify risk involved and Identify the goal.
  * Reveiw and analyze the project requirement
  * Give an estimation for each requirement. The estimation will be based on each TC(test case) and each TC will be assigned approximate 2.5hrs.
  * Then prepare test strategy document which contains project scope, description, testing scope(functional and non-functional testing)



#### Functional Testing

  1. Smoke testing
  2. Functional testing
  3. Regression testing
  4. Defect retesting

Non - Functional Testing 
  1. System Performance testing (load, stress, spike, endurance)
  2. Non Functional requirements are not specified for Billing. Need to add and plan for executing in case if there is any NFR testing in scope.

**Entry Criteria** The Entry Criteria for the System Test will be on the completion of the following:  Release notes stating the functionality available in the current build should be available  Formal communication stating that the environment is ready to be tested.  Unit Test Log should be inspected and code coverage should be atleast 90%  Smoke Test should be pass (Confirm if a sanity check is required)  System Test scenarios / Test cases should be approved  System Test Case prioritization should be complete  Integration testing should have been completed  Controlled Environment for testing to be made available to the system testing team before the scheduled start testing  The test environment should be ready. Formal communication should be received by testing team (from the test environment owner)  Test data should be available. **Exit Criteria** The Exit Criteria for the System Test will be as follows:  Execution of all test cases and completion of testing.  The status of all valid defects reported during the System Test is “Closed” or “Deferred”. (No Level 1 or Level 2 defect should be open) If the exit criterion is not met, still the test phase can take an exit with stakeholder approval. System testing exit criteria should be met as the entry criteria for UAT, in order to ensure that system testing and UAT does not happen parallel. System test environment would be separate from UAT environment (UAT would be done on the production environment) 

## Test Strategy & Approach

**Type of Testing** | **Applicable For (Module / system)** | **Owner** | **Remarks**  
---|---|---|---  
Unit Testing | All Modules | Development team | None  
System Testing | Whole System through following UI: 

  1. PaaS Admin
  2. Process Modeler
  3. Process Work bench
  4. Cloud manager

| System Testing team | None  
Performance Testing – System level | Whole System | GTO Team | None  
Performance Testing – Module Level | All Modules | Dev team / Testing Team | Ownership yet to be finalized between dev and testing team. Training to be obtained to use VSTS Load Test.  
Defect Retesting | Against defects fixed | Testing Team | None  
Regression Testing | On new build, around the defects fixed | Testing Team | Manual testing.(Applicable if part of the functionality was changed).Extend of regression testing done would depend on time and resource availability.  
**Test Coverage**   **Type of Testing** | **% Coverage** | **Rationale for the same**  
---|---|---  
Unit Testing | 100 |   
System Testing | 100 |   
Performance Testing – System level | 100 |   
Performance Testing – Module Level | 100 |   
Defect Retesting | 100 |   
Regression Testing | 30-100 | Time availability might place a hindrance to have better regression test coverage.  
User Acceptance Testing | 100 |   
Automation | NA |   
Internationalization | NA |   
Browser Compatibility | NA |   
Customization testing | NA |   
SOX Compliance Testing | NA |   
Security Testing | NA |   
Portability Testing | NA |   
Test data, Test tool \-------------------------------------------------------------------- 

  * What tasks do you do in the begining of the project? 1\. review ans analyze the project requirements- Analyse the requirement--delgate tasks 2\. Plan and organize the KT to the testters and self 3\. Collect the queries related to the requirementws and get them resolved by the business person 4.Organize and lead the kick off meeting


  1. Can you walk through your test planning process?

Scope the requiement test Design the requied test strategy in lin with the scope and oranization standards Identify the requred tools Estimate the test effort and team(size, skills, attitude and schedule) Create the test schedule(tasks, dependencies and assigned testers) Identify the training requiment for testers Determine and procure the test environment (hardware and sofware and network) Identify test metrics Create software test plan, get it reviewed and approved /signed off 9:19
