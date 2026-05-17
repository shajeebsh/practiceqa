---
title: "webMethods"
date: 2014-04-03 08:07:12 
author: shajeebhameed
layout: page
slug: webmethods
status: publish
---

* ============================================= 
    * http://documentation.softwareag.com/webmethods/wmsuites/wmsuite7/Developer/7-1-1_Developer_Users_Guide.pdf
  * http://techcommunity.softwareag.com/ecosystem/communities/public/webmethods/products/adapters/
  * (mymail/1234567890) 
    * http://www.idnxchange.com/blog/usages-of-designer-in-webmethods-82.html
    * **source** : http://www.developersbook.com/webMethods/interview-questions/webMethods-interview-questions-faqs.php
============================================ 1.What is EAI ?

EAI or Enterprise Applications Integration can be defined as data that can be integrated from disparate applications regardless of the platform, allowing the sharing of business processes amongst multiple organizations.

  2.What are the Major categories of EAI?

Integration can be at different application layers:

    * **Data Level Integration:**
      1. Batch data transfer, OR
      2. On-line propagation of data updates
    * **API Level Integration:**
      1. Data is accessed through published API services
    * **Service Method Level Integration:**
      1. Common services shared by different applications
    * **User Interface Level Integration:** The controller reacts to the user input. It creates and sets the model. 
      1. Common user interface (e.g. web based) for unified access to multiple applications.
 

3.What are the Advantages of EAI?

Advantages of EAI solutions are:

    * Streamlines business processes and helps raise organizational efficiency.
    * Real time information access among systems.
    * Maintains information integrity across multiple systems.
    * Speedier transactions at reduced costs.
    * If one of the applications misbehaves and requires to be shut down for maintenance, then with EAI, we can easily “decouple” it from rest of the systems. Which avoids having to bring down other systems.
 

4.What are the disadvantages of EAI?

The main disadvatages of using EAI systems:

    * **Constant change:** The very nature of EAI is dynamic and requires dynamic project managers to manage their implementation.
    * **Lack of EAI experts :** EAI requires knowledge of many issues and technical aspects.
    * **EAI is a tool paradigm:** EAI is not a tool, but rather a system and should be implemented as such.
    * **Building interfaces is an art :** Engineering the solution is not sufficient. Solutions need to be negotiated with user departments to reach a common consensus on the final outcome. A lack of consensus on interface designs leads to excessive effort to map between various systems data requirements.
    * **Loss of detail :** Information that seemed unimportant at an earlier stage may become crucial later.
    * **Accountability :** Since so many departments have many conflicting requirements, there should be clear accountability for the system's final structure.
 

5.What are the main companies which provide EAI tools / software?

    * TIBCO
    * webMethods
    * Vitria
    * iPlanet
    * MQSeries (IBM)
    * iPlanet
    * BizTalk (Microsoft)
    * WebLogic (BEA)
 

6.What is webMethods?

A company that provides integration tools. The key products include Integration Server, Enterprise Server, Business Integrator, Workflow and Mainframe Integration Server. webMethods is a company, not a product.

7.What are the modules of webMethods Product Suite?

    * Integration and B2B
    * Service Oriented Architecture
    * Business Process Management
    * Business Activity Monitoring
 

8.What are the tools of webMethods Integration ?

    * webMethods Adapters
    * webMethods Developer
    * webMethods Integration Server
    * webMethods Integration Platform
    * webMethods Broker
    * webMethods Monitor
    * webMethods Optimize for Infrastructure
    * webMethods Trading Networks
    * webMethods EDI Module
    * webMethods EDIINT
    * webMethods eStandards Modules
    * webMethods PIM
 

9.What Is Developer?

webMethods Developer is a graphical development tool that you use to build, edit, and test integration logic. It provides an integrated development environment in which to develop the logic and supporting objects that carry out the work of an integration solution. It also provides tools for testing and debugging the solutions you create.

 

10.What Is an Element?

An element is an item that exists in the Navigation panel in webMethods Developer.Elements include folders, services, specifications, IS document types, triggers, and ISschemas. In the Navigation panel, servers and packages are not considered to be elements.

11.What Is an Element?

An element is an item that exists in the Navigation panel in webMethods Developer.Elements include folders, services, specifications, IS document types, triggers, and ISschemas. In the Navigation panel, servers and packages are not considered to be elements.

 

12.What Is a Startup Service?

A startup service is one that Integration Server automatically executes when it loads a package into memory.

 

13.What Is a Flow Service?

A flow service is a service that is written in the webMethods flow language. This simple yet powerful language lets you encapsulate a sequence of services within a single service and manage the flow of data among them.

 

14.What Is the Pipeline?

The pipeline is the general term used to refer to the data structure in which input and output values are maintained for a flow service. It allows services in the flow to share data.The pipeline starts with the input to the flow service and collects inputs and outputs from subsequent services in the flow. When a service in the flow executes, it has access to all data in the pipeline at that point.

 

15.How to invoke a service from a browser ?

Use a URL in the form: http://servername:port/invoke/folder.subFolder.subsubFolder/serviceName (the package name is not part of the URL in any way)

 

16.What happens when the pub.flow:tracePipeline service is invoked?

The Integration Server logs the name-value pairs in the pipeline at that time

 

17.When creating a BRANCH flow element, what is the purpose of the "scope" field on the properties tab? 

To restrict pipeline access to only the data in this document

 

18.What is the primary function of the built-in pub.flow:savePipeline service?

Save the current pipeline to a named memory location on the Integration Server

 

19.When you create and save the FLOW "my.pack:myFlow" in the "MyPack" package, where will you find the code?

In the "MyPack\ns\my\pack\myFlow\flow.xml" file

 

20.What is the Branch operation?

Branch operation conditionally executes an operation based on the value of a variable at run time

 

21.What is the default behavior, if a Flow EXIT does not specify a "from"?

$loop will be assumed, and a com.wm.lang.flow.FlowException will be thrown if the EXIT is not in a LOOP

 

22.An Integration Server package may have one or more startup services. When does a startup service execute?

Whenever the package is loaded or re-loaded

 

23.By default, the webMethods Integration Server has an HTTP listener assigned to which port?

5555

 

24.How can the webMethods Integration Server logging date format be changed?

By editing the watt.server.dateStampFmt parameter in the server.cnf file

 

25.When coding IS Services, how can a variable of type Document Type be represented in Java?

Variable of type Document Type be represented as "IData "

 

26.For a REPEAT operation to execute as long as the specified repeat condition remains true, the count parameter needs to be set to: 

The count parameter needs to be set to "-1 " .

 

27.When creating Flow services, what is the purpose of a SEQUENCE operation?

The purpose of Sequence operation is to group a subset of Flow operations so that they are treated as a unit.

 

28.If the webMethods Integration Server is started with from the server root directory with this command, "bin\server.bat -debug 9 -log none", what does this tell the server to do? 

Start in level 9 debug mode and write all server log information to the screen.

 

29.The Integration Server requires access to the Java classes for each JDBC driver that it will use. Typically, where must such Java classes be placed? 

webMethods6\IntegrationServer\lib\jars

 

30.The Flow Services are physically stored on the webMethods Integration Server in the form of:

Flow Services are physically stored on the webMethods Integration Server as "XML" files.

 

31.What is the default behavior if a Flow EXIT does not specify a "from"?

The EXIT will throw an java.lang.NullPointerException.

 

32.After a default installation, in order to use the pub.file:getFile service, what needs to be done?

pub.file:getFile does not require any modifications to the Integration Server.

 

33.What happens when the pub.flow:tracePipeline service is invoked?

The Integration Server logs the name-value pairs in the pipeline at that time.

33.How to use SEQUENCE as the Target of a BRANCH? 

Set evaluate label property of branch step to true. Then set the label property of sequence with the value on which it needs to be processed.

 

34.How to Restore a Session on a Server? 

Developer gets disconnected from the server if the server goes down or if there is a problem in the network. Donot close the developer.If you close the developer you wont be able to save the changes.Once the server come up or the network problem is resolved. you will be automatically connected to the server and then you can retsore your session.

 

35.How to open a session on a different server? 

Select "session" from the menu in toolbar and click open. key in the server IP and port on which you have to open the connection.The user name and password on that server.

 

36.How ACLs Affect locking? 

ACLS are used to give the authorization to the particular user groups.If u give ACL to administrator then the admin group users who are there in that group they can have the acess to that particular service.Otherwise we can not use it.This is called ACL locking.

 

37.How to Change the Order of Steps in a Flow Service? 

We can change the Order of steps in a Flow Service of Various other services which are called in sequence within Flow service using "Shift Up and Shift Down" buttons exists at top of Editor Panel. As well we can move any service or Map inside a SEQUENCE or BRANCH using "Shift Left and Shift Right" buttons

 

38.When and why should we use transformers and flow services? How are they different from each other? 

Mapping is the process of performing transformations to resolve data representation differences between services or document formats. By linking variables to each other on the Pipeline tab, you can accomplish name transformations and structural transformations. However, to perform value transformations you must execute some code or logic. Developer provides two ways for you to invoke services: You can insert INVOKE steps or you can insert transformers onto the Pipeline tab. Transformers are the services you use to accomplish value transformations on the Pipeline tab.

 

39.What are Structural transformations ? 

Splitting one field into several or merging fields, reordering portions of a message or renaming fields are know as structural transformations

 

40.How to Move Flow Steps? 

Open the webMethods developer in EditPrespective.Select the flow step u wanna move just drag it to the place u want to move.Otherwise use the arrow buttons on the editor pannel to move the selected flow steps.

 

41.How to remove a system lock from an element? 

System locks can be removed by making the server side files of the element as redable.right Click on the elemet in developer which is system locked.and choose the lock properties. It will display the server side files for the element.Make the files as readable and click the referesh button in the developer.You will find that the element is no more locked.

 

42.How to Find Elements in the Navigation Panel? 

Just right click on the element ehich u want to see then u click Locate in navigation option then u can see that element in the navigation panel

 

43.How to find dependents of a selected element on the server? 

Right click on the element for which you have to find the dependents in the navigational pannel.and click on the option find dependents.

 

44.How do I change the JVM used by Integration Server? 

To change to the JDK used by webMethods you will need to edit the IntegrationServer\bin\server.bat or IntegrationServer/bin/server.sh file used to start up Integration Server. Edit the file and change the following line to point to the JDK path
    
    SET JAVA_DIR=C:\opt\j2sdk1.4.2

   

45.How do I debug the Developer IDE itself ? 

Start the developer up in debug mode, similar to the Integration server:
    
    cd pathToWMInstall/Developer/bin
    developer.bat -debug 10
    

   

46.What is the difference between drop and delete pipeline variable? 

**Drop pipeline** is an explicit cleanup. It is a request for the pipeline to remove a variable from the available list of variables and make the object it refers to available for garbage collection by the Java Virtual Machine. **Delete** is purely a design time operation to remove the variable from the current view. It is only of use if you have created a variable that you didn't mean to create. If you delete a variable that was there because it was previously in the pipeline when you change the view in developer you will see the variable appear again.

 

47.How do I see the java code for my flow service ? 

Flow is not turned into java code. It resides on disk as XML representing the flow operations which is then parsed and turned into an in-memory java tree of the operations. Although the underlying code that implements the flow operations is java, it is stored on disk as XML.

48.How do I throw an exception when using a try-catch block ? 

Set a flag in your catch block or leave a variable holding the error message in the pipeline. Outside the catch block put a branch on that variable or flag and if it is non-null then exit with failure or call the service that generates the exception.

 

49.How to get the current index of the List in a loop? 

There is a special variable on the pipeline called $iteration which will be incremented as the loop operator works up through the list.

 

49.How to limit a flow service executed only by one thread at a time? 

    * Create a java service
    * Create a private static object on the shared source (private static Object LOCK_OBJ = new Object(); )
    * Have the code below on the java service source: 
          
          IDataCursor idc = pipeline.getCursor();
          IDataCursor idcResult = null;
          
          try {
          // put this section into a critical section to ensure single-threaded execution
          
          synchronized(LOCK_OBJ)
                          {
                              Execute a flow service using Service.doInvoke
                          }
          idc.destroy();
          idcResult.destroy();
          
          } catch (Exception exc){ 
                          ServerAPI.logError(exc); 
                          idc.destroy();     
          throw new ServiceException(exc.toString());             }\\
          

IDataCursor idcResult = null; try { // put this section into a critical section to ensure single-threaded execution synchronized(LOCK_OBJ) { Execute a flow service using Service.doInvoke } idc.destroy(); idcResult.destroy(); } catch (Exception exc){ ServerAPI.logError(exc); idc.destroy(); throw new ServiceException(exc.toString()); }\\\  

50.How do I sort using the JDBC select adapter service? 

Although there is no tab to specify "order by" the same functionality is able to be specified in the "SELECT" tab. One of the columns in the is labelled "Sort Order" which will allow you to specify the column(s) you wish to sort by. To alter the order: simply alter the order of the columns selected.

 

51.How should I organise connection pools ? 

If you have adapter notifications and adapter services then you will need to have two separate connections. Otherwise you may get strange errors about transactions and the like. You should also avoid having connection pools shared across different functional areas, even if they are pointing to the same database. The reason for this is that tuning the size of the pool becomes quite difficult if you have multiple types of usage of a pool. You are also unable to easily change the database settings for one without impacting on the other. One approach that seems to work quite well is to have separate pools for each package (generally.. not a hard and fast rule though), as your packages should generally be divided up according to functional area too.

 

52.How to preserve existing pipeline before a restorePipeline step ? 

Set the "$merge" variable in restorePipeline or "merge" in restorePipelineFromFile to be true. This will ensure that everything in the pipeline before a restorePipeline step is preserved.

 

53.What is the primary function of the built-in pub.flow:savePipeline service? 

Save the current pipeline to a named memory location on the Integration Server.

 

54.What is the default behavior if a Flow EXIT does not specify a "from"? 

$loop will be assumed, and a com.wm.lang.flow.FlowException will be thrown if the EXIT is not in a LOOP

 

55.An Integration Server package may have one or more startup services. When does a startup service execute? 

Startup service will execute whenever the package is loaded or re-loaded

 

56.What is the primary purpose of a Web Service Connector? 

The purpose of Web Service Connector is to invoke a Web Service on a different web server Startup service will execute whenever the package is loaded or re-loaded

 

57.The Developer was used to create a Java Service named myService in a folder named myFolder in the Default package. What is the best way to hide the source code of the Java service? 

Configure an Access Control List which only allows members of authorized groups and assign it as the Read ACL for the service

 

58.When using the SEQUENCE Flow operation, if exit-on is set to SUCCESS, what condition will cause the entire SEQUENCE to fail? 

When all of the child operations fail .

 

59.The Integration Server requires access to the Java classes for each JDBC driver that it will use. Typically, where must such Java classes be placed? 

Java classes are be placed in _webMethods6\IntegrationServer\lib\jars_

 

60.TThe Package Management interface of the webMethods IS Administrator can be used to create package dependencies. Package dependencies can be used to:? 

Ensure that specific webMethods IS packages are loaded before the depending package loads
