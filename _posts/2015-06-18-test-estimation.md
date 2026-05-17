---
title: "Test estimation"
date: 2015-06-18 10:45:06 
author: shajeebhameed
layout: page
slug: test-estimation
status: publish
---

# 

# http://www.softwaretestinghelp.com/test-estimation-of-selenium-automation-project-selenium-tutorial-32/

# 7 Factors Affecting Test Estimation of Selenium Automation Project – Selenium Tutorial #32

Posted In | [Selenium Tutorials](<http://www.softwaretestinghelp.com/category/selenium-tutorials/>) | Last Updated: "February 12, 2015"

In last couple of Selenium tutorials we learned about [automation testing using Cucumber and Selenium tool](<http://www.softwaretestinghelp.com/cucumber-bdd-tool-selenium-tutorial-30/> "Cucumber and Selenium tools"). We also discussed about[integration of selenium WebDriver with Cucumber](<http://www.softwaretestinghelp.com/selenium-webdriver-cucumber-selenium-tutorial-31/> "Selenium Cucumber integration"). In this tutorial we will discuss about different **factors affecting effort estimation of Selenium automation**. Planning and estimation are two most important aspect of a software development life cycle. I personally feel that in software industry, there are **no bullet proof methods** of doing anything. Since every project is exclusive and have different sets of complexity and environmental factors, implementing the estimation and planning strategy should be a collaborative effort of the individual teams with proper interventions of seniors and management support. [![Selenium automation](http://cdn2.softwaretestinghelp.com/wp-content/qa/uploads/2014/11/Selenium-automation.jpg)](<http://cdn2.softwaretestinghelp.com/wp-content/qa/uploads/2014/11/Selenium-automation.jpg>) _Before you begin with estimating any project, it is imperial to understand each and every phase that your project will be going through, so that you can give a correct and a justified estimation._ Estimation can not only be done for the manual testing process, but in this era of automation, estimation techniques are applied to test automation as well. Now Selenium gaining a momentum and popularity in the market, I am trying to write about some factors which should be taken into consideration while estimating a Selenium project. Let’s Start!! I am assuming that we are starting the Automation initiative from the scratch and that we have no ready-made framework available. 

### **Factors affecting estimation of selenium automation**

The various factors which effect and which you should consider for estimation of “Selenium” specific project are explained below: 

### **#1 Scope of the project**

Scope typically means identifying the correct test cases for automation. Apply “Divide & rule” strategy to accomplish it. Break your application in small chunks or modules and analyze each of them to come up with the appropriate test cases for automation. **The steps involved are:**

  1. Identify the various factors which will form the basis of identifying the candidate test cases.
  2. Break the application into smaller modules
  3. Analyze each module to identify the candidate test cases
  4. Calculate ROI

For more details of how to identify the correct test case, please see my previous paper: [Selection of correct test cases for Automation](<http://www.softwaretestinghelp.com/manual-to-automation-testing-process-challenges/> "http://www.softwaretestinghelp.com/manual-to-automation-testing-process-challenges/")

### **#2 Complexity of the application**

Steps involved here are: 

  1. Determine the Size the application based on the number of test cases that needs to be automated.
  2. Size complexity through Fibonacci series.
  3. Identify the verification point and check point of each test case

Here we have to establish the definition of big / medium and small sized application. This definition differs from an individual / group perspective. How you classify your application, depends can also be dependent upon the number of test cases. **For example:** If your application has 300 – 500 test cases to automate, you can consider it as small sized application. If the test cases are over 1500, it can be classified as complex. This factor can be different for different application. For some, 1500 test cases to automate can be considered as small / medium scaled. So once you have identified the exact number of test cases, scale it to small / medium or large. Your strategy towards estimating the effort will hugely dependent on these criteria. You have to also consider the different check points and verification points for your test case. A test case can have more than 1 check point but will have only 1 verification point. In case you have more than 1 verification point, it is recommended to bifurcate into separate test cases. This will also ease your maintenance and enhancement of your test suite. 

### **#3 Use of supporting tools / technologies**

**Steps involved here are:**

  1. Identify the framework and automation needs
  2. Based on the needs, analyze and identify the tools to be used.
  3. Identify the dependencies / implications of using the tool.

Selenium alone is not sufficient to build a framework or complete the automation. Selenium (Web driver) will only script the test case, but there are other tasks as well, like reporting the result, tracking the logs, taking screen shots etc. To achieve these you need separate tools that will be integrated with your framework. So it is important here to identify these supporting entities which will best suite your requirement and will help to get a positive ROI 

### **#4 Implementing the Framework**

Here comes the tricky part J the steps involved are!! 

  1. Identify the input (pattern in which data is fed in to script) and output (reports / test results) of your automation suite.
  2. Design your input files. This may range from a simple text file to complex excel file. It is basically the file which will have your test data.
  3. Design the folder structure based on your input parameters and
  4. Implement the reporting feature ( either in some excel file or using any tool like ReportNG)
  5. Determine / implement logger in your frame work
  6. Implement the build tool in your framework
  7. Implement the unit test framework (Junit or TestNG)

There are many other requirements apart from just scripting in test automation with Selenium, like reading the data from a file, reporting / tracking the test results, tracking logs , trigger the scripts based on the input conditions and environment etc. So we need a structure that will take care of all these scripts. This structure is nothing but your Framework. Web applications are complex by nature because it involves lots of supporting tools and technology to implement. In a similar way, implementing framework in Selenium is also tricky (I will not say complex) as it involves other tools to integrate. Since we know Selenium is NOT a tool but actually a collection / group of jar files, it is configured and not “Installed”, Selenium itself is not strong enough to build a complex framework. It requires a list of third party tools for building a framework. We have to remember here that there is nothing “Ready-made” in Selenium. For everything, we have to code, so provisions in estimation should be there for googling the errors and troubleshooting. Here we have to understand that that Framework building is the most important aspect of your Automation effort. If your framework is rock solid, maintenance and enhancement becomes easier specially in the era of Agile, if your framework is good, you can integrate your tests in all the sprints easily. I won’t be wrong if I say that this particular factor of designing the Framework should be the most important aspect of estimation. If needed (like in complex application) this factor should be again broken down into a separate WBS and estimation should be done. 

### **#5 Learning & Training**

Learning Selenium is a bit different than learning any other automation tool. It basically involves learning a programming language than just a scripting language (though script language helps while building a framework , like you want to write a script that would invoke your automated scripts after doing the environment setting changes). In case we are combining WebDriver with java, I would say that if one is well versed with core java, they are in a very good shape to start with selenium automation. Along with learning java, provisions should be there to learn other technologies like ANT / Maven( for building), TestNG/jUnit ( unit test framework), Log4J( for Logging), reporting ( for reporting) etc. this list may grow based on the level of framework. The more this list grows, the more time it would take. If the management has decided to go with selenium, these learning activities can be done parallel with the planning activity. Because there is no limit to learn these technologies, it is suggested to have a definite plan (syllabus) ready for the team so that they can initiated their learning process in a definite direction and everybody is in the same page. Practically speaking, we testers do not have a very much keen in learning a full-fledged programming language and we feel this is developers piece of cake. But now we have to change this mentality and should consider learning the programming language to be equally important as learning new testing process. This will not only increase tester’s knowledge about the language and automation but also will give a chance to understand how the application works internally which may increase their scope to find new bugs. 

### **#6 Environment setup**

Environment set up deals with (not limited to):- 

  * Setting up the code in the test environment
  * Setting up code in production environment
  * Writing scripts for triggering the automated tests.
  * Developing the logic for reporting
  * Establishing the frequency of running the scripts and developing logic for its implementation
  * Creating text / excel files for entering the test data and test cases
  * Creating property files for tracking the environments and credentials



### **#7 Coding / scripting and review**

Before you actually start writing your tests, there are 2 prerequisites: 

\------------ 

  1. Candidate test cases should be handy
  2. Framework is ready

Identify different actions that your web application does. It can be simple actions like navigation, clicking, entering text; or a complex action like connect to database, handle flash or Ajax. Take one test case at a time, and identify what all action that particular test case does and estimate hours accordingly for per test case. The sum of all the hours for the entire test suite will give you the exact number. Provision should be there for Review as well. The reviews are simple the code review which can be done either by peer or a developer. Pair programming is the best option which is quick, but if it is not possible, based on the available resources or organizations review strategy, hours should be allocated for it. **More details about each factor affecting estimation:** **Factor #1: Scope** **Meaning** : Identifying the candidate test cases for automation through the ROI **Steps Involved:**

  1. Identify the various factors which will form the basis of identifying the candidate test cases.
  2. Break the application into smaller modules
  3. Analyze each module to identify the candidate test cases
  4. Calculate the ROI

**Deliverable:** List of test cases that needs to automated. **Remarks:** It is important to freeze your scope once you go ahead with other steps of estimation. **Factor #2: Complexity** **Meaning:** Establish the definition of big / medium and small sized application. **Steps Involved:**

  1. Size the application based on the number of test cases that needs to be automated.
  2. Size complexity through Fibonacci series.
  3. Identify the verification point and check point of each test case.

**Deliverable:** Size of the application – Small, medium or Big. Number of test cases and their corresponding checkpoint and verification point. **Remarks** : Recommended – A test case can have multiple check point but only 1 verification point. If a test case has more than 1 verification point, it should be bifurcated into a separate test case. **Factor #3: Supporting tools** **Meaning:** Selenium itself is not strong enough to build a complex framework. It requires a list of third party tools for building a framework. **Steps Involved:**

  1. Finalized IDE
  2. Finalized unit test tool
  3. Finalized logger
  4. Finalized reporting tool
  5. Finalized build tool

**Deliverable:** List of tools needed to create the framework. **Remarks:** Examples: 

  * Eclipse / RAD – as IDE
  * Ant / Maven – As build tool
  * jUnit / TestNG – as unit test framework
  * Log4j – as Logger
  * ReportiNG – as reporting tool
  * Text files – for tracking the environments / credentials
  * Excel files – for tracking the test data
  * Perl / Python – for setting up environment and triggering the test scripts.

**Factor #4: Implementing Framework** **Meaning:** Creation of structure **Steps Involved:**

  1. Design your input files.
  2. Design the folder structure
  3. Determine / implement logger in your frame work
  4. Implement the build tool in your framework
  5. Implement the unit test framework

**Deliverable:**

  * Framework and folder structure created in the IDE.
  * Excel sheets containing your input data
  * Property files containing environment related data and credentials.

**Remarks:** This is the most crucial step. It is advisable to include some buffer time while estimating because some time trouble shooting take more time than expected. **Factor #5: Environment set up** **Meaning:** Deals with code set up and downloading / preparing for the code deployment **Steps Involved:**

  1. Prepare the input file and reporting
  2. Create the triggering script

**Deliverable:** Environment ready **Remarks:** We should try to build our framework in such a way that with least hassle, our code is deployed in to the said environment / box. I should not be wrong if I say that with minimal entries into our text files (which have the url and credentials) our code should be ready to run and ROCK!**** **Factor #6: Learning & training** **Meaning:** Learning a programming language and other supporting technologies **Steps Involved:** Prepare a plan as per your automation needs and share it with the team and encourage them to learn and proceed as per the syllabus. **Deliverable:** Training Plan and its tracker which will track the progress of the team. **Remarks:** Emphasis should be on building logics rather learning syntax. **Factor #7: Coding / scripting and Review** **Meaning:** Writing the actual test scripts and reviewing them **Steps Involved:**

  1. Test cases and framework is ready.
  2. Take / divide the test cases and convert it into automated scripts and track your progress

**Deliverable:** Automated test scripts **Remarks:** Whole team should participate in writing the test scripts using the implemented framework. So while estimating, efforts from the whole team should be taken into consideration. 

### **Conclusion** :

Having said about all these points, do not forget to include Management overhead and some buffer time in your final Selenium automation estimation. The best and the proven way to do any estimation is to follow the WBS (Work Break down Structure) mechanism. This is straight forward and serves the purpose of implementing the automation estimation needs. The factors mentioned above are the ones based on my experience, but there can be other entities as well which might affect the strategy. The thumb rule here is **“Identify certain criteria, divide your modules or test case on those criteria; and scale it”.** Based on your scaled figure – you can come to an accurate estimation.
