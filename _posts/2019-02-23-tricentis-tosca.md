---
title: "Tricentis TOSCA"
date: 2019-02-23 15:47:20 
author: shajeebhameed
layout: page
slug: tricentis-tosca
status: publish
---

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

__

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

Folder. Value is reused and the it can be used several different places. It is set for multi use.

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
