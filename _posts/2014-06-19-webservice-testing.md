---
title: "WebService Testing"
date: 2014-06-19 08:52:59 
author: shajeebhameed
layout: page
slug: webservice-testing
status: publish
---

##### **List of nonfunctional characteristics**

##  http://www.ibm.com/developerworks/library/ws-soa-nonfunctional/

http://www.soapui.org/security-testing/overview-of-security-scans.html 

##### Table 1. List of nonfunctional characteristics

Type of test | Test definition  
---|---  
**Accessibility** | Verify the ability to access the functionality of the application.  
**Audit and control** | Verify how easily it's possible to check the historic workflow and audit trail.  
**Availability** | Verify that the application has a high uptime as stipulated in the service level agreement (SLA).  
**Compatibility** | Verify if the application fits in a pre-existing (older) environment.  
**Documentation** | Verify that the user guides provide the right instructions.  
**Installation** | Verify if the application works on the defined middleware stack.  
**Interoperability** | Verify if application works after changing a vital component in the environment.  
**Load/Volume** | Verify if the application processes the required number of transactions in a given time.  
**Maintainability** | Verify how easy it is to maintain the application after it goes into production.  
**Performance** | Verify if criteria like response time, throughput, concurrency and so on meet requirements.  
**Reliability** | Verify that the application works if stressed in production-like environment.  
**Scalability** | Verify if the application can meet the growing needs of a business.  
**Security** | Verify if the application has enough security to protect information theft.  
**Serviceability** | Verify if the application can be debugged without making any impact on the business.  
**Usability** | Verify that the application is usable from an end-user perspective.  
  ======================================== 

  1. http://www.ibm.com/developerworks/library/ws-tip-strstest/index.html
  2. http://www.qaielearning.com/KnowledgePapers/Introduction-to-Testing-Webservices.pdf
  3. http://www.soapui.org/REST-Testing/getting-started.html
  4. Soap UI Training - https://www.youtube.com/watch?v=WbLaipYhl1s

======================================== Webservice testing strategies Unit testing Webservice is similar to any other traditional application, so unit testing is a must. Unit test cases must be written before the application is developed. As and when the application is built, test cases are applied on the code. Hence the functionality is verified as and when we develop the webservice. Basic testing The main aim of this testing is to test whether the webservice is accessible and can be invoked properly. Main focus in this phase should be to carry out the following procedures. Get the WSDL file and test whether it is well-formed and in compliance with the WSDL specifications published by W3C Using this WSDL file generate the client side stubs that handle the interaction with the webservice. Test the webservice functionality that is whether the webservice responds to the requests submitted to it correctly. This involves coding a sample invoker to the client stubs. Invoke the sample invoker by passing it the parameters required by the webservice. Check the response of the webservice from a functionality point of view. The sample invoker calls the client stubs which further call the webservice. The stub constructs the SOAP message from the parameters passed to it and passes this message to the service. This message can be monitored by a Sniffer program like TCP Monitor. 

If there are any security checks, like username and password we need to

test their effectiveness. The intent of this step should be to break in the

system and gain unauthorized access.  

## Designing stress applications

Test systems that attempt to stress a Web service need to be designed to exercise code in particular ways. These styles go beyond functional verification to see if a Web service not only does what it is supposed to, but also continues to perform as it should when certain stressful conditions are applied. There are four basic conditions which stress tests must apply to a Web service. Many established stress systems apply these conditions. Effective stress testing systems apply these key conditions: 

  1. **Repetition:** Probably the most obvious and easy to understand stress condition is test repetition. In other words, test repetition means performing a particular operation or function over and over, such as repeatedly calling a Web service. A functional verification test can be designed to see if an operation either works or does not work. A stress test will determine if an operation works and continues to work every time it is carried out. This is essential in concluding whether a product is fit to be used in a production situation. A customer will typically use a product repeatedly, and therefore stress testing should find the code defects before a customer finds them. Many naive stress systems implement only this condition, but simply extending a functional verification test to be repeated many times does not constitute an effective stress test. When used in combination with the following principles, repetition can uncover obscure code defects.
  2. **Concurrency:** Concurrency is the act of performing several operations simultaneously. In other words, performing several tests at the same time, for example calling a number of Web services on the same server simultaneously. This principle may not apply to all products (such as stateless services), but the majority of software has some element of concurrent or multi-threaded behavior, which can only be tested by executing multiple instances of the code. A functional or unit test will rarely incorporate any concurrent design. A stress system must go beyond the functional tests to exercise multiple code paths at the same time. How this is done depends on the specific product. For example, a Web service stress test would need to simulate multiple clients at once. A Web service (or any multi-threaded code) will typically access some shared data amongst the thread instances. The complication that is added by this extra dimension of programming often means that code has many defects attributed to the concurrency. Since introducing concurrency means that code in a thread might be interrupted with code from other threads, defects are uncovered that are only found when a set of instructions are executed in a particular order (such as with a particular timing condition). By combining with the principle of repetition, you can cover many code paths _and_ timing conditions.
  3. **Magnitude:** Another condition that stress systems should apply to their products concerns the amount of load they apply in any single operation. A stress test can repeatedly carry out an operation, but that operation should also strain the product by itself. For example, if you have a Web service that allows a client to enter a message, you could incorporate high usage into a single operation by simulating a client that enters a huge message. In other words, you increase the magnitude of the operation. This magnitude is always application specific, but can be identified by looking for values in the product that can be measured and modified by a user - for example, size of data, length of delay, amount of funds transferred, speed of input, variety of input, etc. On its own, a single strenuous operation might not find a code defect (or might only find a functional defect), but when combined with the other stress principles, you increase the chances of finding a problem.
  4. **Random Variation:** Finally, no stress system would be complete without an element of randomness. If you randomly use the countless number of variables that the previous stress principles introduce, you are able to cover many different code paths each time a test is run. The following are just a few examples of how you can vary a test during its lifetime. With repetition you could vary the time between repetitions, the number of repetitions before you restart or reconnect to service, or the order of Web services that are repeated. For concurrency you can vary the Web services that are carried out together, the number of Web services running at any one time, or whether to run a number of different services or a number of the same instance. Magnitude is perhaps the easiest to modify -- each time a test is repeated you can modify the variables that are present in the application (for example, sending assorted sizes of message or values of numerical input). Since it is difficult to consistently recreate a stress defect if the test is entirely random, some systems use random variation based on a constant random seed. In this way the defect has a higher chance of being recreated with the same seed.

A single stress test will typically combine all of the above, and run for as long a period of time as is allowed. The longer the test is allowed to execute, the more code paths are covered and the more defects can be found. Of course, once a defect is found it must be diagnosed and fixed. Since a code defect in a stress test could show itself after days of running, the system must ensure that all available debug information is generated when something goes wrong -- otherwise the same amount of time might have to be taken to recreate it. The following Security Scans are currently available (click on the name for a dedicated page) 
  * [SQL Injection](<http://www.soapui.org/security-testing/security-scans/sql-injection.html>) : tries to exploit bad database integration coding
  * [XPath Injection](<http://www.soapui.org/security-testing/security-scans/xpath-injection.html>) : tries to exploit bad XML processing inside your target service
  * [Boundary Scan](<http://www.soapui.org/security-testing/security-scans/boundary-scan.html>) : tries to exploit bad handling of values that are outside of defined ranges
  * [Invalid Types](<http://www.soapui.org/security-testing/security-scans/invalid-types.html>) : tries to exploit handling of invalid input data
  * [Malformed XML](<http://www.soapui.org/security-testing/security-scans/malformed-xml.html>) : tries to exploit bad handling of invalid XML on your server or in your service
  * [XML Bomb](<http://www.soapui.org/security-testing/security-scans/xml-bomb.html>) : tries to exploit bad handling of malicious XML request (be careful)
  * [Malicious Attachment](<http://www.soapui.org/security-testing/security-scans/malicious-attachment.html>) : tries to exploit bad handling of attached files
  * [Cross Site Scripting](<http://www.soapui.org/security-testing/security-scans/cross-site-scripting.html>) : tries to find cross-site scripting vulnerabilities
  * [Custom Script](<http://www.soapui.org/security-testing/security-scans/custom-scan.html>) : allows you to use a script for generating custom parameter fuzzing values



# 1\. Adding Security Scans

Add a Security Scan to a TestStep in your Security Tests either with the “Add SecurityScan” button or the corresponding TestStep right-click menu option in the Security Test window. You will first be prompted for which type of Security Scan to add (differs based on the underlying TestStep) and then open the corresponding Security Scan configuration window:   ![security-scan-configuration](http://smartbearsoftware.com/soapui/media/images/stories/security/security-scan-configuration.png) (You can open this for an existing Scan by double-clicking it in the Security Test window). This dialog has a similar layout for all Security Scans: the top of the dialog usually contains a table for defining which parameters in the request to use for test testing (see below). In the middle there is an area for Security Scan specific configuration components (not used in the above screenshot), and at the bottom there are a number of tabs for further configuration: 

  * Assertions : the assertions used to validate and check the response for any signs of a successful security exploit
  * Strategy : settings related to how multiple parameters should be permutated against each other (see below)
  * Advanced : settings specific for the Security Scan (if applicable)

The table of parameters, assertion tab and strategy tab are common for most Security Scans, let’s have a look at them in more detail. 

# 2\. Security Scan Parameters

Most Security Scans require you define which content of the underlying request you want to use as placeholders for the corresponding scan, for example for a SOAP request you might have a message as follows: ![login-request](http://smartbearsoftware.com/soapui/media/images/stories/security/login-request.png) When performing for example a SQL Injection scan with this request, you would want to send the malicious SQL statements in both the username and password fields, which would require you to define these two as parameters in the table. In the Pro version, this is easiest achieved by pressing the “Extract Parameters” button on the top of the table itself, which will search all available properties in the request and automatically add them to the table if they contain a value: ![extract-parameters-button](http://smartbearsoftware.com/soapui/media/images/stories/security/extract-parameters-button.png) Alternatively you can use the “Add Parameter” button which will open a dialog for specifying the Parameter manually: 

![add-parameter-dialog](http://smartbearsoftware.com/soapui/media/images/stories/security/add-parameter-dialog.png)

Here you need to specify the following: 

  * The underlying Test Property that contains the parameter value (for example Request for SOAP requests)
  * A unique label for the parameter
  * An optional XPath statement specifying where in the Test Property value to find the parameter 
    * This is for properties containing XML values, for example for REST or HTTP parameters, you would (probably) leave this field blank

For HTTP or REST requests, all defined request parameters are available as Test Properties, and if any of them contain XML you can further refine how that should be mutated as described with the XPath configuration above. 

# 3\. Security Scan Assertions

Assertions are used to assess if the responses for the Security Scan requests contain some kind of content that indicates if the target system has a corresponding vulnerability. The mechanism is the same as for standard Test Requests; use the table in the assertions tab to specify which assertion to use and their configuration:   ![security-assertions-tab](http://smartbearsoftware.com/soapui/media/images/stories/security/security-assertions-tab.png) Selecting the right assertions is not always trivial; how do you want your system to behave when under attack? Should it return an error or just ignore the attack? Maybe a SQL Injection attack would be “successful” if it managed to provoke the expected behavior for valid input (thus fooling the system) – an adequate assertion would in that case perhaps be to check that the response does not contain what would otherwise be expected. All the standard assertions are available, but also a number of new ones have been added specifically for this purpose (see below). 

## 3.1. Invalid HTTP Codes

Allows you to specify a comma-separated list of HTTP status codes that should not be returned by the target service ![invalid-http-status-codes-assertion](http://smartbearsoftware.com/soapui/media/images/stories/security/invalid-http-status-codes-assertion.png)

## 3.2. Valid HTTP Codes

Allows you to specify a comma-separated list of HTTP status codes that should be returned ![valid-http-status-codes-assertion](http://smartbearsoftware.com/soapui/media/images/stories/security/valid-http-status-codes-assertion.png)

## 3.3. System Information Exposure

Checks the response for content that reveals system information which could be used by hackers to further exploit any existing vulnerabilities, for example if the response gives away which database version that is being used (in an error message), hackers could use this information to try to exploit known security issues with that database. ![sensitive-information-exposure-assertion](http://smartbearsoftware.com/soapui/media/images/stories/security/sensitive-information-exposure-assertion.png) As you can see, the default configuration fo this assertion can be specified at both the global and project level, so if you want to add any custom tokens to be used in the search it can be done either specifically for the containing Security Scan or at a higher level. The default list provided with soapUI is visible in the Global Preferences under the “Global Sensitive Information Tokens” tab: ![global-sensitive-information-exposure-tokens](http://smartbearsoftware.com/soapui/media/images/stories/security/global-sensitive-information-exposure-tokens.png) The table has two columns: 

  * The token itself can either contain a plain string or a regular expression prefixed with a tilde sign (to separate it from a plain string)
  * A description that will be shown in the Security Log if the corresponding token is found



## 3.4. Cross Site Scripting Assertion

This assertion is available only for the Cross Site Scripting Scan and automatically added as well, it checks the response for the same injection strings that were sent with the parameter (read more) and also allows you to specify a script that populates a list of URLs to check separately for the send XSS tokens: ![cross-site-scripting-assertion](http://smartbearsoftware.com/soapui/media/images/stories/security/cross-site-scripting-assertion.png) The script alternative is useful if you for example are posting data into a system via a REST or SOAP interface where the immediate response is never viewed in a browser, but a separate page might make the posted data available to end users – this page should then be checked for an eventual XSS vulnerability. 

# 4\. Strategy

The strategy tab allows you to specify how multiple parameters are to be permutated and executed during the execution of the Security Scan: ![security-strategy-tab](http://smartbearsoftware.com/soapui/media/images/stories/security/security-strategy-tab.png) There are currently two modes of execution: 

  * “One by One” – mutates one parameter at a time; if you for example have three parameters defined and the Security Scan has 20 values to send for each, the number of total requests will be 60 – each request containing one mutation for only one parameter, the other parameters are left as in the original request
  * “All at once” – mutates all parameters at the same time; in the above example this would result in 20 requests in total.

The “Request Delay” setting allows you to specify a delay between multiple mutated requests so the Security Scan doesn’t overload your server, and the “Apply to Failed TestSteps” option allows you to control if the Security Scan should be applied even if the underlying functional TestStep fails its assertions – by default this is disabled as it might not make sense. 

# 5\. Execution

When a Security Scan is run as part of the containing Security Test, it sends the different mutation requests as configured, mutating the defined parameters for each request. The Security Log shows specifically which values were sent for each parameter and request, together with any assertion failures: ![security-log-with-parameter-values](http://smartbearsoftware.com/soapui/media/images/stories/security/security-log-with-parameter-values.png) Double-clicking an entry will open the standard message viewer allowing you to see the actual request sent and response received for further analysis, for example the failed Malformed XML request above shows the following response message: ![message-result-for-failed-assertion](http://smartbearsoftware.com/soapui/media/images/stories/security/message-result-for-failed-assertion.png) Here we can see the response contains a stacktrace giving away a number of details on what software the server using, which could be an entry point for exploiting known issues with that server setup.
