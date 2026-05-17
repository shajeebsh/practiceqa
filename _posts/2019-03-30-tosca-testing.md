---
title: "TOSCA Testing"
date: 2019-03-30 16:46:43 
author: shajeebhameed
layout: page
slug: tosca-testing
status: publish
---

[**https://www.wisdomjobs.com/e-university/tosca-interview-questions.html**](<https://www.wisdomjobs.com/e-university/tosca-interview-questions.html>)

[**https://www.javainuse.com/misc/tosca-testing-tool-interview-questions**](<https://www.javainuse.com/misc/tosca-testing-tool-interview-questions>)

[**https://www.qatechies.com/tosca-interview-questions-1/**](<https://www.qatechies.com/tosca-interview-questions-1/>)

<https://www.qatechies.com/tosca-interview-questions-2/>

<https://www.qatechies.com/tosca-interview-questions-3/>

<https://www.qatechies.com/tosca-interview-questions-4/>

[**https://www.qatechies.com/tosca-tbox-architecture-and-related-interview-questions/**](<https://www.qatechies.com/tosca-tbox-architecture-and-related-interview-questions/>)

**__**

Tosca is an integrated software testing solution that employs a unique Model-based Test Automation and Test Case Design approach, encompassing risk-based testing, test data management and provisioning, service virtualization, and more.Using a unique Model-based Test Automation and Test Case Design approach that encompasses risk-based testing, service virtualization, and more, Tosca Testsuite consistently delivers a 10x performance improvement to enterprise testing globally.

Table constraint

Reusable testcase

Run testcase n number of times

TDM benefits

Identifying Object by CSS

Caption, Identification, Property and Result

<https://support.tricentis.com/community/article.do?number=KB0014376>

**Recovery Scenarios**

**** Retry Leave – TestCase, TestStep, TestStepValue

Setting >> Recovery >> OnDialog,Verification, Exception, Retries

**TestSheet to db**

**Buffer vs xBuffer –**

**Buffer -** <Buffername>=.<Property> usage {B[<Buffername>]}

**xBuffer -**{XB[<Buffername>]}

**Folder structure**

**Template, TestSheet, TestCaseDesign, Attributes**

**TDM**

**Identification – anchor and image**

**TQL – Subparts and superparts –**

  1. **Subparts – searches all the child elements in TOSCA where as superparts searches for parent elements. Eg – “- >SUBPARTS” – Sub Elements, “+>SUBPARTS”**
  2. **“- >SUPERPARTS” returns the super elements as a result.**



****

**_ActionModes_**

ActionModes for input operations:

  * Input
  * Insert



ActionModes for reading operations:

  * Buffer
  * Constraint
  * Verify
  * WaitOn
  * Select



****

****

****

**Exploratory Testing**

**Jenkins**

**Reusability**

**TCP** is a Test Configuration Parameter

**Business parameter** which is used to pass dynamic values in test step blocks.

Business parameter is created by Right-click on a Reusable TestStep Block and select Create Business Parameter Container from the context menu.

**What Is Action Mode For****Dynamic Buffer****?**

**Answer :**

Action mode for Xbuffer should be: Verify

**Syntax for****XBuffer****:** {XB[<Buffername>]}

**Difference Between Buffer And xBuffer?**

**Answer :**

Buffers are temporarily saved to the section Buffer of the Settings dialog under Settings->Engine.

To buffer a value in the Settings dialog, a Buffer name and a Buffer value are required. Set the ActionMode to Buffer.

The XBuffer allows you to read dynamic values of a string and to buffer them using the ActionMode Verify.

**__**

**_Best Practices_**

  1. Pgt Mgt - Folder Structure – Maintainance, Project by mission critical
  2. Promoted
  3. InWork
  4. InReview
  5. IntoProduction
  6. Common repor
  7. Single
  8. Creating user groups
  9. Permission
  10. Proper naming conventions – Folder, Testscrips – Improved readablitiy
  11. USE TEST DATA MANAGEMENT (TDM) OR TEST DATA SERVICE (TDS)
  12. INCLUDE AND USE A “STATUS” COLUMN IN YOUR TDM OR TDS
  13. KEEP TDS DATA UP TO DATE DURING TESTCASE EXECUTION



[HTTPS://SUPPORT.TRICENTIS.COM/COMMUNITY/ARTICLE.DO?NUMBER=KB0014231](<https://support.tricentis.com/community/article.do?number=KB0014231>)

How data driven testingis achievedinTOSCA Testsuite?

Data driventestinginTOSCA Testsuite canbe achievedbyusing testsheet inTestCaseDesign Section.There we can create an attributes(Parameterswhichneedtocoverfordata driventesting) & Create variousinstances(possibilities)foreachattribute.Finallygenerate instances.Create a template fromyourtestcase,attachtestdata sheettotemplate &theninstantiate the template. For details, clickhere

  1. What is template?how to create template? –> In general,template isnothingbutsomethinginstandardformat.InTOSCA TestSuite,we can converttestcase to template sothatit can be usedfor variousdatacombinations.Template canuse testdata from testdata sheet. For details, clickhere
  2. what is meanby instantiating template? –> Instantiatingtemplate meansconvertingone templatetestcase intomultiple testcasesbasedon the testdata whichis suppliedtotemplate.Thisishow we canachieve datadriventestingusing TOSCA TestSuite. For details, clickhere



**__**

**__**

**__**

**__**

**_Point to Learn_**

**__**

**Steering Test objects**

**Action Mode – DoNothing, Input, Verify, Buffer, Waiton, Output**

**__**

**_Jenkins_**

C:\ProgramFiles\Tricentis\Tosca TestSuite\ToscaCI\Client\ToscaCIClient.exe –m local

**__**

**Model Based Test Technique**

**Model Based And Risk Based Testing**

  1. **_Model based test Technique_** _:_ One major feature where this tool is gaining leverage over other automation tools is due to its model based technique where a model of AUT (application under test) is created instead of scripts for test automation. All the technical details about AUT, test script logic and test data are separately saved and merged together at the time of test case execution. The central model gets updated the moment any change is encountered in element of the application. This central model helps in removing ambiguity and a larger effort as all the test scripts that are responsible to test the updated element need not be recreated manually to reflect the change.
  2. **_Risk based test technique:_** As the name rightly suggests, this one is used to assess risk with respect to test cases and help in identifying the right set of test scripts and the risk impacted by them. Various black box test techniques such as Equivalence Partitioning, Decision Box, boundary testing and combinatorial methodology such as linear expansion, etc. are used to minimize the test script count while ensuring to increase the functional risk coverage. Once the test execution is completed, the reports related to technical, functional, business and compliance are collated and risks are measured based upon risk coverage.



****

**Identify Objects In Tosca**

**Identify By Anchor**

**Library Parameter -** is nothing but business parameter which is used to pass dynamic values in test step

**Test Configuration Parameter vs Business parameter**

AidPack – Module used for stirng manipulation

**Executing Test Cases In Scratch Book And Executionlist**

Project root Elements – ExecutionList, Modules, Reporting, Requirements, Testcase Design, Testcases.

**__**

**_Engines_**

The Tosca TBox framework is the basis for steering XEngines and allows all XEngines to be uniquely steered. This applies to both GUI and non-GUI.

Tosca XScan 3.0 - <https://documentation.tricentis.com/en/1100/content/tosca_commander/creating_xmodules.htm>

Tosca AnyUI Engine 3.0

Tosca API Engine 3.0

Tosca Database Engine 3.0

Tosca DotNet Engine 3.0-Tosca DotNet Engine 3.0 allows WinForms applications to be tested that were built with Microsoft™ .NET Framework 4.6.

Tosca JavaFX Engine 3.0

Tosca JSON Engine 3.0

Tosca Mobile+ Engine

Tosca Mobile Engine 3.0-The Tosca Mobile Engine 3.0 is used to test mobile native and mobile web applications on mobile devices, smartphones and tablets, as well as simulators and emulators.

With the Tosca Mobile Engine 3.0, you can automate your native iOS and Android applications and execute TestCases in the browsers Google Chrome or Apple Safari without the need for the Tosca Mobile Browser.

Tosca SAP Engine 3.0

Tosca SAP Web Extension 3.0

Tosca XBrowser Engine 3.0 - Tosca XBrowser Engine 3.0 can be used to test web applications.

<https://documentation.tricentis.com/en/1100/content/engines_3.0/xbrowser/xbrowser.htm>

<https://documentation.tricentis.com/en/1100/content/tbox/tbox_intro.htm>

**__**

**_Tosca_**

  * Navigation Pane
  * Working Pane



o Details

o Properties

o Control Flow Diagram

  * Creation and Reusability of Modules



o Element containing technical steering information of an application.

o Module Attributes

  * Representations of individual controls of an applicatoin, contained in modules
  * Save modules in a recommended **Folder**



o **5** Methods of Control Identification

  * Identify by Properties - Using own properties
  * Parent’s properties – For eg select UL control for selecting all the LI children
  * Identify by Anchor
  * Identify by Image
  * Identify by Index



_Create a Module_

  1. Scan application page using Xscan
  2. Select necessary controls
  3. Uniquely identify each control
  4. Rename the module
  5. Save.



Right click >> Scan application >> Browser – It will lists all the browsers

Double click the intended browser – Select the section in the web page in which need to san

o Tosca will all the business controls(textbox, button etc) from scan.

o Slide left or right to show up controls

o Also mouse over the control in the web page in which we need to add to Tosca

  * Rename the module to meaning full name



_Naming Conventions and TestCase Structures_

Test case structure

  1. Precondition
  2. Workflow
  3. Order Product
  4. Start checkout
  5. Checkout process
  6. Verify Discount
  7. Confirm Order
  8. Postcondition



Create a TestCase and name it

Create and name TestStep Folders to contain TestSteps

Create a Test Configuration Parameter

**_Questions_**

**Q:** What is the difference between Test Configuration Parameter and Business Parameter ?

**Ans:** Test Configuration Parameter is set on Test case level or higher, Test case folder or even superior Test case

Folder. Value is reused and the it can be used several different places. It is set for multi-use.

**Recovery Scenario In Tosca Testsuite**

Setting -> TBox -> Recovery

  1. Verification-Failure: verification does not provide the expected results (e.g. an invoice total differs from the TestCase definition).
  2. Dialog-Failure: the application wants to steer a control which does either not exist or is not in an operational state. Application errors like crashes or jams (i.e. if the application is not responding) also belong to this category.
  3. User Abort: Abort of the test execution by the user.
  4. On Exception Failure



Business Parameter is only used in Test Step level. It is set for single use.

_Test configuration_

Got to “Test configuration” tab >> Right click test case >> Create Test Configuration Parameter

Checklist/Folder for Each Section

  1. Approved
  2. Ready for approval
  3. InWork
  4. Rejected



Example

  1. MyProject_MP
  2. 1_MP_Requirements
  3. Agile Env
  4. 01_Initial Implementation
  5. 02_Backlog
  6. 2_MP_TestCaseDesign
  7. Big Machines
  8. Merchant Service Cloud



iii. TDM Definition

  1. Test Data preparation
  2. 3_MP_TestCases
  3. Recovery Scenarios
  4. 1_Approved_TestCases



iii. 2_Ready for approval_TestCases

  1. 3_InWork_TestCases
  2. 11_OutOfScope_TestCases
  3. 12_Duplicates(During modifications)



vii. Overview

  1. 4_MP_ExecutionLists
  2. _Archive
  3. Overview



iii. Rerun Execution

  1. Tester execution
  2. Approved TestCases
  3. 5_MP_Modules
  4. 1_Approved
  5. 2_Ready for approval



iii. 3_Inwork

  1. 4_Rejected
  2. \-----------------
  3. Overview



Introduction to Requirements

We can deeply design requirement structure.​

  1. Requirement Folder
  2. Requirement Set


  * Regression Test View
  * Release Test View
  * Agile - 
    * ​Regression Test View
    * Sprint Test View​
  * Tabs in Requirement View - Used for Risk based assessment approach testing(Higher weight mean higher risk) 
    * Frequency Class
    * Damage Class
    * Weight - 2^DC * 2^FC
    * Relative Weight(%)
    * Contribution(%)
    * Coverage Specified(%)
    * Execution State(%)



Dynamic Expressions and Execution Lists

Syntax:-

  * {​RND[No of Digits]}
  * {​RND[LowerLimit][Upper Limit]}
  * {RANDOMTEXT[No of characters]}
  * {RNDDECIMAL[Digits before decimal][Digits after decimal]}



Right click expression >> Select "Translate Value" for getting output without running the whole scripts.

Dynamic Date Expression

  * {​EXPRESSION[Basedate][Offset][Format]} 
    * EXPRESSION - DATE, TIME, DATETIME, MONTHLAST, MONTHFIRST
    * Basedate - According to TOSCA date format - 25.12.16
    * Offset - +2y, -4M, +2d
    * Format - dd/MM/yyyy



** _Buffer_**

Used to store values and pass it to another test

Settings >> Engine

**_Dynamic Expression_**

_Test Execution_

How to set a Buffer ?

  * Enter a property (ie InnerText) and a buffer name as the value of TestStep attribute
  * Set ActionMode to Buffer



​​​

How to use Set Buffer? 

  * Enter {B[name of saved variable]} as the value of TestStep Attribute
  * Set ActionMode to input of verify

Note: Stored values can be viewed in menu item: Project >> Setting >> Engine >> Buffer 

How to use Dynamic Comparison ?

  * Select a TestStepValue and enter the string you want to verify.
  * Replace the dynamic port of the string using {XB[Buffername]}
  * Set ActionMode to Verify



**_Introduction to Exploratory Testing_**

  1. What is exploratory testing?
  2. Creating exploratory testing sessions
  3. Capturing interactions of exploratory scenarios 
     1. Use Video or Interation option to capture sessions
     2. Send mail to the testers for getting Exploratory test along with Link. By clicking the link the tester can download agent, and do the exploratory testing and send back the results to the host.
  4. The Exploratory testing agent​



**_Loops and Conditions_**

  1. What are loops and conditions?
  2. How to create loops and conditions?
  3. Repetitions on folder level
  4. When to loops and conditions

    **_Standard Modules_** Top 10 stadard modules 

  1. OpenUrl
  2. TBox Wait – Some TC requires certain time to wait. This allows us perform comparison.
  3. TBox window operation
  4. TBox Set Buffer
  5. TBox Evaluation Tool
  6. TBox Read/Create file
  7. TBox Delete Buffer
  8. SAP Logon
  9. SAP Login
  10. TBox Send Keys
