---
title: "n-Tier in ASP.NET MVC"
date: 2012-06-22 04:54:04 
author: shajeebhameed
layout: page
slug: n-tier-in-asp-net-mvc
status: publish
---

## <http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En>

## 

## Table of Contents

>   * [Introduction](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Introduction>)
>   * [Pre-Requisites](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Pre>)
>   * [What is ASP.NET MVC?](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#What>)
>   * [Why ASP.NET MVC?](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Why>)
>   * General Details 
>     * [Does ASP.NET MVC is the Only Option?](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Only>)
>     * [Building the DMS using ASP.NET MVC](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Building>)
>     * [Design considerations](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#desingcons>)
>     * [How you Architect a System?](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#How>)
>   * Main Design the DMS 
>     * [Building the Code Framework for our DMS](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Framework>)
>     * [Let’s Look at Our Data Access Layer Design](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#DAL>)
>     * [Let's Look at Our Business Logic Layer Design](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#BLL>)
>   * Additional Part of the DMS Design 
>     * [Validating Business Objects](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Validating>)
>     * [Let's Look at Our Presentation Layer (User Interface) Design](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#UI>)
>     * [Dependency Injection](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#DI>)
>     * [Scalability Options of the Design](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Scalability>)
>   * [Summary of the DMS Design](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Summary>)
>   * [Unit Testing](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Unit>)
>   * [Other Things to be Noted](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Other>)
>   * [Conclusion](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Conclusion>)
>   * [History](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#History>)
> 


## Introduction

The Model View Controller (MVC) is an architectural pattern used in software engineering. The pattern isolates "domain logic" (the application logic for the user) from input and presentation (GUI), permitting independent developments, testing and maintenance of each. Today, the Microsoft variant of MVC called ASP.NET MVC, which is now part of .NET framework 4.1, is gaining momentum among software designers. In addition to that the enthusiasm is also high among developers to support the community with materials that speed up application development with ASP.NET MVC framework. There are many source libraries on the internet, where such contributions can be found. These free off-the-shell libraries/ components have positively influence the modern software industry in many different ways. However, as it was with any other case, this has a few negatives too. Today, unfortunately, junior designers, who are trying to do their first system in MVC, are struggling to select the right set of off-the-shell components correctly. They do worthlessly complex designs by needlessly committing them-selves to use these off-the-shell components. One needs to understand that you don't have to use everything in every (or the first) system you design. You need to use only the suited ones. But finding the suited options, when many different options are made available, is easier said than done. The stated problem is a little more complicated than previously stated as the technology providers too have competitive products. Therefore to start off, let me gives couple of such examples, where you see different means made available to achieve the same goal. 

  1. Entity Framework, Linq to SQL, Subsonic, Hibernated or any other can be used as an ORM (Object Relational Mapping) tool.
  2. _'AutoMapper'_ ,  _'StructureMap'_ , or any other can be used to map one DTO (Data Transfer Object) to another.
  3. The  _'Validation Application Block'_ of Microsoft Enterprise Library,  _'System.ComponentModel'_ namespace of .NET Framework, Microsoft Workflow Engine (WF) of .NET Framework or any other can be used to validate business objects.
  4. Views of MVC, Web-From or generic ASP.NET controls or even plain HTML can be used to develop the User Interface (UI) layer.
  5. JavaScript, Ajax, J-Query can be used as your front end scripting language with or without JSON (Java Script Object Notation).
  6. MVC Architecture or any other, or even a combination of architectures can be used to design a system.

Among the items listed above, you can see there are similar or more applicable options. Additionally there are other options, which have to be selected base on your requirement. These options have to be selected considering the functional and non functional requirement of your system. Once well designed, the system truly adds value to your development process. A designer with experience, selects his options the right way first time, and ends up keeping significant advantages. If you are a junior system designer, then at least be careful not to pick an awe fully wrong set of options. Such selection can make your 'software design' carrier end even before you start it. They say "Perfection (in design) is achieved not when there is nothing more to add, but rather when there is nothing more to take away." One who reads through this article will realize that an architecture is built with Microsoft ASP.NET MVC Framework, Microsoft Entity Framework, Ajax, J-Query,  _'AutoMapper'_ , Enterprise Library (Unity Application Block),  _'MsTest'_ , Log4Net etc. This design may not be the most flawless of all. But I can assure you that this is not a terribly wrong one either. It will fall within the acceptable range, and once completed, you will be able to re-use this as a framework to develop your very own (small to medium sized) web applications. 

## Pre-requisites

In order to follow this article, you are expected to have at least 4 years or more of development experience with a sound understanding of OOP concepts and design patterns. Additionally, since our system is going to be designed with ASP.NET MVC framework, you need to have experience developing a few test applications with ASP.NET MVC framework, most preferably with Microsoft Entity Framework. If you think you have sufficient expertise, then you are best to further read through this article. If you still have not setup ASP.NET MVC, please have the items listed below installed before proceeding any further. You can also install ASP.NET MVC using [Microsoft Web Platform Installer](<http://www.microsoft.com/web/downloads/platform.aspx>) too. 

  * [Microsoft Visual Studio 2010 (RC, in my case)](<http://www.microsoft.com/visualstudio/en-us/download> "VS 2010") (or any other later version)
  * [ASP.NET MVC](<http://www.asp.net/mvc/> "ASP.NET MVC")
  * [Unity Application Block](<http://unity.codeplex.com/Wikipage>) (The completed version of Enterprise Library will auto install this)
  * [Ajax](<http://www.asp.net/ajaxlibrary/download.ashx>)
  * [J-Query](<http://docs.jquery.com/Downloading_jQuery>)
  * [Auto-Mapper](<http://automapper.codeplex.com/>)
  * Entity Frame Work – This comes with the VS 2010



## What is ASP.NET MVC?

As stated before, ASP.NET MVC is the Microsoft variant of MVC and it is a free, Microsoft framework for developing great web applications using the Model-View-Controller pattern. It provides total control over your HTML and URLs, enables rich Ajax integration, and facilitates test driven development. [Read more >>](<http://www.asp.net/mvc/>) I hope that the above description also answers the famous question that some developers have, 'Is MVC a pattern or framework?'. Let me reiterate here again, just to ensure that you understand it correctly. The MVC is a pattern, as well as a framework. The MVC Pattern is being used to develop Microsoft ASP.NET MVC framework. So that first to be introduced is the pattern and then the framework. 

## Why ASP.NET MVC?

![MVCSummary.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/MVCSummary1.jpg) ASP.NET is not something completely new, but it is something that Microsoft has built on top of the standard ASP.NET library. It can be taken as an extension to the .NET Framework. In other words, it is something greater than standard ASP.NET or Web Forms. When you are working with Web-Form the framework itself controls plenty of variables, where as with ASP.NET MVC it does not do that. Instead the framework allows you to control them. So with ASP.NET MVC, it is left to you to decide how you want to build your system. Though I said that we need to take care of everything with ASP.NET MVC, it is not really difficult. There are many resources available on the internet to make life easier for MVC developers. Unlike before, where they complained about having to write more codes to get something done in MVC, now you can do them quicker with the support of contributions of other developers. The ‘[MVC Contrib](<http://mvccontrib.codeplex.com/>)’ is one and was designed to add functionality and ease-of-use to Microsoft's ASP.NET MVC Framework. The  _'MVC Contrib'_ is useful for developers looking to develop and test web applications on top of the ASP.NET MVC framework. In addition to that there are many new features made available with the latest version of ASP.NET MVC framework. You can learn more about this by reading “What’s New in ASP.NET MVC 2” document on the [www.asp.net/mvc ](<http://www.codeproject.com/KB/aspnet/www.asp>)web-site. 

## Does ASP.NET MVC is the Only Option?

There are debates. Some designers say Web-Form is better than MVC. They say that there are many 3rd party controls already built for Web-Form. The MVC is extend-able, yet for all they say that the architecture needs you to write more codes. Additionally they also say that a well architected Web-Form application too can be just as extensible as MVC. In my view point, the decision has to be taken based on your requirements. When it is 50:50, the latest technology is the better choice. However we should not choose Web-Form simply because of that natural resistance we have for changing what we used to be. This habit of resisting the change is not something that you can practice in IT. The IT change rapidly forcing one to give up old things and move ahead with new ones. I think, the ASP.NET MVC going to have a great future. Microsoft has done it right this time with ASP.NET MVC. It is already proving to be the framework for building next generation of web applications. So why wait? If you adopt early, you will grow easier with it. If not then, you will experience the pain of becoming an out-dated developer with the next wave of technology updates. In the next part of the article, I will gradually start designing our Document Management System (DMS). That will demonstrate the application of Microsoft ASP.NET MVC framework to build a real world web application. However, before we move on to that, I need to stress-out few other important things about MVC here.. 

  * ASP.NET MVC is "*not" WebForms 4.0
  * There is a small learning curve with MVC, but there are *many* resources to assist you
  * MVC gives you more control, which is generally better
  * If you like Unit Testing, ASP.NET MVC will make you happy.



## Building the DMS using ASP.NET MVC

I have carefully picked a couple of options to build our DMS. This design is taking advantage of Microsoft ASP.NET MVC Framework, Microsoft Entity Framework, Ajax, J-Query,  _'AutoMapper'_ , Enterprise Library (Unity Application Block),_'MsTest'_ , Log4Net etc. I am planning to discuss some of the important selections that I have made in later part of this article, but mostly I expect you to learn those by looking at the source code itself. I will mainly focus on our goal, which is to teach you how to establish the right architecture to build a small to mid-size web application with ASP.NET MVC framework. In that effort, I am planning touch base with all important points. I will give just enough details for an experienced developer to understand them but not anymore as I need to control the length of this Article. As the first step of architecturing our DMS (or any) system, you need to thoroughly analyze the system. This includes of fully and deeply understands its requirements, time period, cost constrains and quality requirements etc. Let me give few points that helps you understand it below... 

  * The business problem that the proposed system is supposed to solve?
  * The functional and non functional requirements
  * Project quality requirements
  * How much money, the customer is willing to spend for this system?
  * The project time period, start and end of the project
  * What is the scale (How big the system is?)?
  * How flexible the system needs to be?
  * How extendable the system has to be?
  * How customizable the system needs to be?

Once the system requirements are fully understood, mainly covering the questions given above, you can start the first few steps of designing a system. This is where you define the structure of the software and find the right way in which that structure provides conceptual integrity for the system. This is the development work product that gives the highest return on investment in terms of quality, schedule and cost. 

## Design considerations

There are many aspects to consider in the design of a piece of software. The importance of each should reflect the goals the software is trying to achieve. Some of these aspects that I referred form [Wiki-Software Design](<http://en.wikipedia.org/wiki/Software_design>) are given below... 

  * **Compatibility** \- The software is able to operate with other products that are designed for interoperability with another product. For example, a piece of software may be backward-compatible with an older version of itself.
  * **Extensibility** \- New capabilities can be added to the software without major changes to the underlying architecture.
  * **Fault-tolerance** \- The software is resistant to and able to recover from component failure.
  * **Maintainability** \- The software can be restored to a specified condition within a specified period of time. For example, antivirus software may include the ability to periodically receive virus definition updates in order to maintain the software's effectiveness.
  * **Modularity** \- the resulting software comprises well defined, independent components. That leads to better maintainability. The components could be then implemented and tested in isolation before being integrated to form a desired software system. This allows division of work in a software development project.
  * **Packaging** \- Printed material such as the box and manuals should match the style designated for the target market and should enhance usability. All compatibility information should be visible on the outside of the package. All components required for use should be included in the package or specified as a requirement on the outside of the package.
  * **Reliability** \- The software is able to perform a required function under stated conditions for a specified period of time.
  * Re-usability - the modular components designed should capture the essence of the functionality expected out of them and no more or less. This single-minded purpose renders the components reusable wherever there are similar needs in other designs.
  * **Robustness** \- The software is able to operate under stress or tolerate unpredictable or invalid input. For example, it can be designed with a resilience to low memory conditions.
  * **Security** \- The software is able to withstand hostile acts and influences.
  * **Usability** \- The software user interface must be usable for its target user/audience. Default values for the parameters must be chosen so that they are a good choice for the majority of the users.

Imagine that our DMS is a complicated business problem, and then try to direct your design effort to reduce its complexity. This has to happen through abstraction and separation of concerns. This means, breaking the system in to its structural elements, architectural components, subsystems, sub-assemblies, parts or  _'chunks'_. This is not hard. Any experience designers can identify the components belonging to a system just by looking at its requirement document. However there are standard techniques that use to do this too. I also have written an Article on this topic, which you can find in [the code project](<http://www.codeproject.com/Articles/70061/www.codeproject.com>) with the title "[A Practical Approach to Computer Systems Design and Architecture](<http://www.codeproject.com/KB/architecture/System_DesignByNirosh.aspx>)". 

## How you Architecture a System?

An experienced Architect does not need to go through every single step in the book to get a reasonable design done for a small web application. Such Architects can use their experience to speed up the process. Since I have done similar web applications before and have understood my deliverable, I am going to take the faster approach to get the initial part of our DMS design done. That will hopefully assist me to shorten the length of this article. For those who do not have experience, let me briefly mention the general steps that involved in architecturing a software below... 

  1. Understand the initial customer requirement - Ask questions and do research to further elaborate the requirement
  2. Define the process flow of the system preferably in visual (diagram) form. I usually draw a process-flow diagram here. In my effort, I would try to define the manual version of the system first and then would try to convert that into the automated version while identifying the processes and their relations. This process-flow diagram that we draw here can be used as the medium to validate the captured requirements with the customer too.
  3. Identify the software development model that suite your requirements 
     * _When the requirements are fully captured and defined before the design start, you can use the _'Water-Fall'_ model. But when the requirements are undefined, a variant of  _'Spiral'_ can be used to deal with that._
     * _When requirements are not defined, the system gets defined while it is being designed. In such cases, you need to keep adequate spaces in respective modules, which later expansions are expected._
  4. Decide what architecture to be used. In my case, to design our Document Management System (DMS), I will be using a combination of ASP.NET MVC and [Multitier Architecture](<http://en.wikipedia.org/wiki/Multitier_architecture>) (Three Tier Variant).
  5. Analyze the system and identify its modules or sub systems.
  6. Pick one sub system at a time and further analyze it and identify all granular level requirements belonging to that part of the systems.
  7. Recognize the data entities and define the relationships among entities (Entity Relationship Diagram or ER Diagram). That can followed by identifying the business entities (Some business entities directly map with the classes of your system) and define the business process flow.
  8. Organized your entities. This is where you normalize your database, and decide what OOP concepts and design pattern to be used etc.
  9. Make your design consistent. Follow the same standards across all modules and layers. This includes streamlining the concepts (as an example, if you have used two different design patterns in two different modules to achieve the same goal, then pick the better approach and use that in both the places), and conventions used in the project.
  10. Tuning the design is the last part of the process. In order to do this, you need to have a meeting with the project team. In that meeting you need to present your design to your team and make them ask questions about it. Take this as an opportunity to honestly evaluate/ adjust your design.



## Building the Code Framework for our DMS

As I said, our Document Management System (DMS) architecture is done combining the three layer architecture and Model View Controller architecture. In order to speed up the process, let me directly define the system framework of our DMS. Please follow the instructions given below to create your system framework directly in Visual Studio 2010. 

  1. Open your  _'Visual Studio 2010'_ and create a new Project of type ASP.NET MVC (in my case it was MVC version 2.0)
  2. In the same solution, create a  _'Console Library'_ type Project for your Data Access Layer (DAL). This will create a separate assembly for your Data Access Layer. In addition to logically group your data access logic in to a separate assembly this separation has the following advantages 
     * Allows modifying the data access logic without affecting other layers.
     * Support scaling the system easily.
     * Support changing the database type without affecting the corresponding business functions.
     * .. etc.
  3. Create a  _'Console Library'_ type Project for core Business Operations. Just like it was with DAL, this will create a separate layer for Business Logics.
  4. Create another  _'Console Library'_ type Project for common operations. There are some business functions that are common to all the layers. These functions can be implemented in a separate assembly. That way you can reuse that with other system too. I will talk about this later in the article too.

Once everything is completed, your solution will be looking like this. ![ProjectList.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/ProjectList1.jpg) In the above diagram, the  _'MvcDemo'_ project is the MVC Web Project. That project is created using Visual Studio's default MVC project template. In it, I decided to keep  _'Views'_ and  _'Controllers'_ in the same web project, while  _'Models'_ were shifted out to another layer (you will read more about this change later). So in summary now you have a solution with one ASP.NET MVC Web Application project, and three empty console libraries together with a test project, which is automatically created for us by Visual Studio. _Note: I have used the main project name ('MvcDemo') as a prefix to name the console libraries, but that is something that you can opted to follow or not._ In our solution each sub system represents a project. Each project represents a meaningful portion of the main business problem. If you want to reuse one project in multiple other projects (Just like we will be doing it with _'MvcDemo.Common'_ project), you can do that by adding the project reference. 

## Let’s Look at Our Data Access Layer Design

The data access layer provides a centralized location for all calls into the database, and thus makes it easier to port the application to other database systems. There are many different options to get the Data Access Layer built quickly. In my effort to build the DAL, I would have chosen from many different model providers, for example: 

  1. NHibernate
  2. LINQ to SQL

Or else I would have chosen any other model provider, possibly: 
  1. SubSonic
  2. LLBLGen Pro
  3. LightSpeed
  4. ..or who knows

This means that we will be using an ORM (Object Relational Mapping) tool to generate our Data Access Layer. As you can see, there are many tools, but all having their advantages and disadvantages. In order to select the right tool, you need consider your project requirements, the skills of the development team, and if an organization has standardized on a specific tool etc. However, I am not going to use the tools listed above, instead decided to use [Microsoft Entity Framework](<http://en.wikipedia.org/wiki/ADO.NET_Entity_Framework>) (EF), which is something Microsoft has newly introduced. The ADO.NET Entity Framework (EF) is an object-relational mapping (ORM) framework for the .NET Framework. ADO.NET Entity Framework (EF) abstracts the relational (logical) schema of the data that is stored in a database and presents its **conceptual schema** to the application. However the conceptual model that EF created fail to meet my requirements. Therefore I had to find a work around to create the right conceptual model for this design. You will see few adjustments that I have made to achieve my requirement in later part of this Article. The selection of EF (Entity Framework) for my ASP.NET MVC front end is not a blindfolded decision. I do have few affirmations... 

  1. ASP.NET MVC Framework is Microsoft, so as ADO.NET Entity Framework. They tend to work well together than any other tool.
  2. Latest version of EF (Entity Framework) has many promising features, and has addressed many issue had in its initial version.
  3. In general Microsoft seems serious about EF. That made me thinks that EF would have a significant share in the future of ORM tools.

The community has raised concerns over what has been promised and is being delivered with EF. You can read this[ADO .NET Entity Framework Vote of No Confidence](<http://efvote.wufoo.com/forms/ado-net-entity-framework-vote-of-no-confidence/>) for more details about it. Microsoft and the Entity Framework team have received a tremendous amount of feedback from experts in entity-based applications and software architectures on the .NET platform. While Microsoft’s announcement of its intention to provide framework support for entity architectures was received with enthusiasm, the Entity Framework itself has consistently proved to be cause for significant concern. However the second version of Entity Framework (known, somewhat confusingly, as  _'Entity Framework v4'_ because it forms part of .NET 4.0) is available in Beta form as part of Visual Studio 2010, and has addressed many of the criticisms made of version 1. The Entity data model (which is also called the conceptual model of the database), which ADO.NET Entity Framework auto created for our DMS is given below. This has a 1:1 (one to one) mapping between the database tables and the so called conceptual models. These models or objects can be used to communicate with the respective physical data source or the tables of the database. I hope you can understand my conceptual model quite easily. ![EntityDesignerDiagram.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/EntityDesignerDiagram1.jpg) The Data Access Layer project, which is given below, has three folders namely Models, Infrastructure and Interfaces (You will later notice that this project template is being reused in other projects of this system too). Outside of those folders, you find few classes such as one base class and two concrete classes. These are the main operational classes of the data access layer (DAL) project. This project template made it possible to directly access all commonly used main operational classes without worrying about navigating in to directories. "The inherent complexity of a software system is related to the problem it is trying to solve. The actual complexity is related to the size and structure of the software system as actually built. The difference is a measure of the inability to match the solution to the problem." \-- Kevlin Henney, "For the sake of simplicity" (1999) Simplicity is the soul of efficiency. This does not mean that the design has to be unrealistically simple. It has to be just enough to achieve the requirements and not more or less. Additionally the consistency is also important. It can reduce the differences in between modules, thus would help to easily understand the design. Therefore the design has to be simple and consistent. While keeping these in mind, let's have a close look at our Data Access Layer (DAL) design. ![DalDesingView1.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/DalDesingView1.jpg) In the Data Access Layer (DAL) design, I thought of using a variant of repository pattern. The Repository pattern is commonly used by many enterprise designers. It is a straight forward design that gives you testable & reusable code modules. Furthermore it gives flexibility and separation of concerns between the conceptual model and business logics too. In my design there are two Repositories namely ‘DocumentRepository’ and ‘FileRepository’, which will mediates between the domain (also called Business Logic Layer) and data mapping layers (also called Data Access Layer) using domain objects (also called Business Objects or Models). In general it is recommended having one repository class for every business object of the system. Among the few folders that you see in that project, the folder named 'Interfaces' is important. So let's check inside the 'Interfaces' folder and see what interfaces that we have in it... 

  1. IRepository<T> \- The generic interface that abstractly defines the behavior of all the repositories of our system. This is the super repository. This is what extends to create the abstract repository with the name 'RepositoryBase<T>'.
  2. IDocumentRepository – This is a specialized repository, which defines the behavior specific to  _'Document'_.
  3. IFileRepository – Just like it was with 'IDocumentRepository', this defines the behavior of the  _'File'_ repository.
  4. IRepositoryContext - This defines the behavior of the context of our repositories.
  5. IPagination - This defined the definition for pagination related operations.
  6. _'IUnitOfWork'_ and  _'IUnitOfWorkFactory'_ were there but removed from the design later. Initially, I was planning to write some codes for dependency injection (DI) too. But later I found that Microsoft enterprise library has everything done for us. They have an application block called  _'UnityApplicaitonBlock'_ , which does what I was initiated with those two interfaces. Therefore I removed unity and inversion of control related implementations from the project. Now the source code you download now will not have those implementations.

I have used interfaces in the DAL design, and that might made you ask questions like 'why we need these interfaces?' and 'why not just have the concrete implementation alone?'. There are many reasons for having interfaces in a design. They allow implementation to be interchangeable (One common scenario is, when unit testing, you can replace the actual repository implementation with a fake one). Additionally, when carefully used, they help to better organize your design. The interface is the bare minimum needed to explain what a function or class will do. It's the lease amount of code you can write for a full declaration of functionality and usability. For this reason, it's (more) clearer for a user of your function or class to understand how the object works. The user shouldn't have to look through all of your implementations to understand what the object does. So, again defining interfaces is the more organized approach. In some instances, I have seen designers add interfaces for every class, which I thought is a too extreme design practice. In my design I have used interfaces not only to organize the design but also to interchange implementation too. Later in the article you will see how these interfaces are being used to assist 'unity application block' to inject dependencies too. You can see in the screen above that I have set of interfaces define for pagination too. The pagination (of business objects/ entities) is something that developers code in many different layers. Some code it in the UI (User Interface) layer, while others do it in BLL (Business Logic Layer) or DAL (Data Access Layer). I thought of implementing it inside DAL to avoid unwanted network round trips that would occurred otherwise in a distributed multi-tier deployment setup. Additionally when keep it inside DAL, I will have the option of using some of the built-in EF (Entity Framework) functions for pagination works. The figure below summarizes the overall DAL design with its main components and their relations. I think it is clear how the three interfaces and their concrete implementations are being defined. ![ClassDiagram.Dal1.png](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/ClassDiagram.Dal1.png) I think it is important for you to know how this design evolves. So let me talk about that here. Initially, I had the _'IRepository <T>'_ interface. That was used to implement the two specialized 'Document' and 'File' repositories. That time I noticed that the two was having many common operations. So I thought of abstractly define them with a abstract class. Thus, decided to add a new generic abstract class called 'RepositoryBase<T>'. Once all these were done, the final system had the generic  _'IRepository <T>'_ interface right at the top, then the abstract 'RepositoryBase<T>' class, and then two concrete implementations of 'File' and 'Document' repositories. I thought it is done, and had a final look before closing-up the design, but that made me realized that the design is still having issues. So let me talk about that in the next paragraph. The design I had made it impossible to interchange implementation of either of the specialized repository. As an example, if I have a test/ fake version of the 'DocumentRepository' implemented with the name 'TestDocumentRepository' and wanted to interchange that with the 'DocumentRepository', then it is not possible with my design. So I decided to do few more adjustment to the design by introducing two specialized interfaces called 'IDocumentRepository' and 'IFileRepository'. This made it possible to fully hide the special implementations, hence gain the required interchangeability. That final change concludes my DAL design. As I said before, I used unity application block to inject dependencies to this system. The code below shows how one can use the unity application block to interchange implementations of IDocumentRepository. The Unity Application Block supports writing this code in 'global.asax' file or to use the 'web.config' to set detail to resolve dependencies. Please expect more details on this later.. 

![](http://www.codeproject.com/images/minus.gif) Collapse | [Copy Code](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#>)
    
    
    //when using the actual DocumentRepository implementation
    .RegisterType<IDocumentRepository, DocumentRepository>()
    
    //when using the test version of the DocumenRepository implementation
    .RegisterType<IDocumentRepository, TestDocumentRepository>()

I also realized that with few adjustments, this very same design pattern can be used in business logic layer too. So you will see the same design approach is being re-used in  _'MvcDemo.Core'_ project too. Before we move on to the next section, let's have a look at the implementation of  _'RepositoryBase <T>'_ class too. In the base repository implementation, I was careful not to make it necessarily complex. As you can see, it only has the most needed methods. You may also see how the dependency that this class has with the ‘IRepositoryContext’ is handled by passing the dependent class through the constructor. You will later see how this technique is used to inject dependencies across all layers. You can find more about Unity DI (Dependency Injection) [here..](<http://unity.codeplex.com/>)

![](http://www.codeproject.com/images/minus.gif) Collapse | [Copy Code](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#>)
    
    
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Data.Objects;
    using System.Linq.Expressions;
    using MvcDemo.Dal.Interfaces;
    using MvcDemo.Dal.Infrastructure;
    using MvcDemo.Dal.EntityModels;
    
    namespace MvcDemo.Dal
    {
        public abstract class RepositoryBase<T> : IRepository<T>
            where T: class
        {
            public RepositoryBase()
                : this(new DmsRepositoryContext())
            {
            }
    
            public RepositoryBase(IRepositoryContext repositoryContext)
            {
                repositoryContext = repositoryContext ?? new DmsRepositoryContext();
                _objectSet = repositoryContext.GetObjectSet<T>();
            }
    
            private IObjectSet<T> _objectSet;
            public IObjectSet<T> ObjectSet
            {
                get
                {
                    return _objectSet;
                }
            }
    
            #region IRepository Members
    
            public void Add(T entity)
            {
                this.ObjectSet.AddObject(entity);
            }
    
            public void Delete(T entity)
            {
                this.ObjectSet.DeleteObject(entity);
            }
    
            public IList<T> GetAll()
            {
                return this.ObjectSet.ToList<T>();
            }
    
            public IList<T> GetAll(Expression<Func<T, bool>> whereCondition)
            {
                return this.ObjectSet.Where(whereCondition).ToList<T>();
            }
    
            public T GetSingle(Expression<Func<T, bool>> whereCondition)
            {
                return this.ObjectSet.Where(whereCondition).FirstOrDefault<T>();
            }
    
            public void Attach(T entity)
            {
                this.ObjectSet.Attach(entity);
            }
    
            public IQueryable<T> GetQueryable()
            {
                return this.ObjectSet.AsQueryable<T>();
            }
    
            public long Count()
            {
                return this.ObjectSet.LongCount<T>();
            }
    
            public long Count(Expression<Func<T, bool>> whereCondition)
            {
                return this.ObjectSet.Where(whereCondition).LongCount<T>();
            }
    
            #endregion
    
        }
    }

In the above code, the  _'ObjectSet'_ is a public property that has being exposed via  _'get'_. That will allow outside parties to know the currently active object set of the respective repository. However when you implement the repository with a specific type, the returned  _'ObjectSet'_ becomes predictable. This is a weakness of my design. As an example, when implementing ‘IFileRepository’ the  _'ObjectSet'_ will guarantee to return  _'IObjectSet <File>'_ but not anything else. So then one can argue saying that the property is not necessary to have publicly exposed. That is a very valid argument. As you can realize now, I cannot defend my design. One should be able to defend his/ her design. Therefore, I think it is acceptable to reduce the property's access level from  _'public'_ to  _'protected'._ That way only the extended classes will be able to access the active object set. As I said before, you have to question your design, the questioning validate the design. If you check the code line below, you might notice something unfamiliar there. The  _'??'_ operator is not commonly used in general coding, but happened to be very useful for me here. 

![](http://www.codeproject.com/images/minus.gif) Collapse | [Copy Code](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#>)
    
    
    repositoryContext = repositoryContext ?? new DmsRepositoryContext();

The  _'??'_ operator is called the null-coalescing operator and is used to define a default value for a null able value types as well as reference types. It returns the left-hand operand if it is not null; otherwise it returns the right operand. This code pattern is important when using an external module (Unity Application Block) to inject dependencies. 

## Let's Look at Our Business Logic Layer Design

The business logic layer is usually one of the layers in our multi-layer architecture. This part of the code contains the business logic for our application. This separation of business functions into a separate project would have the same advantages that we recognized in the Data Access Layer design. The screen below expands the  _'Business Logic Layer'_ part of the system. In that, the folder named  _'Models'_ has a set of  _'View'_ and  _'Edit'_ model classes. These classes represent the  _'Models'_ of the MVC pattern. They are the domain/ conceptual models of our DMS's BLL (Business Logic Layer). As an example, to display a document in the front end, you can have a view-model with the name  _'DocumentViewModel'_ , with properties to display on the UI in it. These model classes have their corresponding  _'Views'_ and  _'Controllers'_ too. In addition to that, when a ‘View’ (Web Page) is use to edit/ update an entity, you can use a  _'DocumentEditModel'_ to capture user inputs. That same model can use to store the definitions of its property validation requirement too (read '[Validating Business Objects](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#Validating>)' below for more details). You can further break this system to make it more flexible. But that comes with a cost. Further you break, difficult it will be to maintain your code. I thought this is just right for our requirement. Therefore based on your requirements you can decide how further you need to break the system. You can ignore the ‘Mapping’ related interfaces for now, but I will come back to that little later. ![CoreProjectViewNew1.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/CoreProjectViewNew1.jpg) As you can see in the screen above, the BLL design is somewhat similar to our Data Access Layer (DAL) design. Just like it was with DAL, I have a separate set of interfaces that define the framework for this layer. 

  * IService<T> \- Is the super interface that uses to implement the services of this application. I also had another interface called ‘IServiceBase’, but that make BLL inconsistent with the DAL design. Therefore I decided to remove that from the design later.
  * IDocumentService - This defines the  _'Document'_ specific services.
  * IFileService - This defines the  _'File'_ specific services.
  * IMapper, IMapperRegister - I was trying to make the mapping library a plug-gable one, so that I will have the option to plug any other mapping library later. However, you need to give me little more time to complete that part of the code.

As you see above, I have  _'DocumentService'_ , which implements  _'IDocumentService'_ , and  _'FileService'_ , which implements  _'IFileService'_. The  _'ServiceBase <T>'_, which is the counter part of the  _'RepositoryBase <T>'_ of DAL, is empty yet. However with my experience I know that we are going to get functions that are common to both Document and File Service later, that could abstractly implement/ define in the  _'ServiceBase <T>'_. I thought it is a good practice to have an abstract type base class defines whenever you do multiple implementations out of a generic interface. ![ClassDiagramServiceNew1.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/ClassDiagramServiceNew1.jpg)

## Let's Look at Our Presentation Layer (User Interface) Design

This is the topmost layer of our application. The presentation layer displays information for the user. It communicates with other layers to produce results to the web browser. In general the architecture we used has several differences that I wanted to point at. We removed the  _'Models'_ of the MVC from our web project and include that in the Business Logic Layer. We also removed the regular presentation layer of the three tier architecture and merge that with the MVC front end. This means that  _'Views'_ and  _'Controllers'_ of the MVC represent the Presentation Layer part of the three tiered system. As this is bit complex to explain, I have drawn the diagram below to further elaborate it. ![MvcMergedThreeLayer1.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/MvcMergedThreeLayer1.jpg) In the screen below you will see how our ASP.NET MVC web project structure looks like. The default directory structure of an ASP.NET MVC Application has 3 top-level directories: 

  * /Controllers
  * /Models
  * /Views

As you can probably guess, it recommend putting your Controller classes underneath the 'Controllers' directory, your view templates underneath your 'Views' directory and as you already knows we have decided to move the 'Models' out of this project. So in our project we will not use the 'Model' folder. In general ASP.NET MVC framework doesn't force you to always use this structure. Unless you have a good reason to use an alternative file layout, I'd recommend using this default layout. ![WebProjectView.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/WebProjectViewNew1.jpg)

## Dependency Injection

Dependency injection is a way of objects getting configured with their dependencies by an external entity. This may sound a bit abstract, so let me give you an example here. E.g.:- In our code you have a class called 'DmsController', which has a dependency with 'IDocumentService'. This means that when you create an object of type DmsController, you need to pass an implementation of _'IDocumentService_ ' as a parameter. In this case think that you have two implementation of 'IDocumentService', where one is called 'DocumentService' (actual) and the other is called 'TestDocumentService' (Unit Test). Then the Unity Application Block (using a pattern called Unity Pattern) allows you to dynamically configure the right implementation to resolve the dependency that ‘DmsController’ has with 'IDocumentService'. If that is still not clear, let me show the way that I am using this application block to inject dependencies in my code. In order to use Unity Application Block, you need to have a custom controller factory defined. In the screen above, you can see a class called  _'UnityControllerFactory'_. I used that class as my custom controller factory. If you look at the source code of that class, you will see that it inherits from the MVC  _'DefaultControllerFactory'_. This means that I can use that class to replace the default MVC Controller Factory. As per the requirement of Unity Application Block, this custom class is used to set the container that carries the detail to resolve interfaces. Let me show you how the ‘Application_Start’ method of ‘Global.asax’ looks like.. 

![](http://www.codeproject.com/images/minus.gif) Collapse | [Copy Code](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#>)
    
    
    protected void Application_Start()
    {
        AreaRegistration.RegisterAllAreas();
    
        RegisterRoutes(RouteTable.Routes);
    
        /*//Configure container with web.config
        UnityConfigurationSection section = (UnityConfigurationSection)ConfigurationManager.GetSection("unity");
        section.Configure(_container, "containerOne");*/
    
        _container = new UnityContainer()
            .RegisterType<IDocumentService, DocumentService>(new ContainerControlledLifetimeManager())
            .RegisterType<IFileService, FileService>()
            .RegisterType<IDocumentRepository, DocumentRepository>()
            .RegisterType<IFileRepository, FileRepository>()
            .RegisterType<IRepositoryContext, DmsRepositoryContext>()
            .RegisterType<IFormsAuthenticationService, FormsAuthenticationService>()
            .RegisterType<IMembershipService, AccountMembershipService>();
    
        //Set for Controller Factory
        IControllerFactory controllerFactory = new UnityControllerFactory(_container);
    
        ControllerBuilder.Current.SetControllerFactory(controllerFactory);
    }

As I said above, the  _'UnityContainer'_ is used to register the interfaces with its respective concrete implementations. This is just one time registration hence it is advised to do it inside the  _'Application_Start'_ method of the  _'Global.asax'_. The container is passed on to my custom made UnityControllerFactory. Finally, you have to use the built-in method of _‘ControllerBuilder’_ to set my controller factory as the current controller factory of this application. The rest is automatic. For more details on this topic, please visit the [Unity Dependency Injection IoC Screencast ..>>](<http://www.pnpguidance.net/Screencast/UnityDependencyInjectionIoCScreencast.aspx>) You can also see how my ‘UnityControllerFactory’ looks like … 

![](http://www.codeproject.com/images/minus.gif) Collapse | [Copy Code](<http://www.codeproject.com/Articles/70061/Architecture-Guide-ASP-NET-MVC-Framework-N-tier-En#>)
    
    
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using System.Web.Mvc;
    using Microsoft.Practices.Unity;
    using System.Web.Routing;
    using System.ComponentModel;
    
    namespace MvcDemo.Infrastracture.Mvc
    {
        public class UnityControllerFactory : DefaultControllerFactory
        {
            private readonly IUnityContainer _container;
    
            public UnityControllerFactory(IUnityContainer container)
            {
                _container = container;
            }
    
            protected override IController GetControllerInstance(RequestContext requestContext, Type controllerType)
            {
                if (controllerType != null)
                {
                    return _container.Resolve(controllerType) as IController;
                }
                return base.GetControllerInstance(requestContext, controllerType);
            }
        }
    }

The 'DefaultControllerFactory' represents the controller factory that is registered by default. This class provides a convenient base class for developers who want to make only minor changes to controller creation. As you can see, we have created our custom controller factory with the name ‘UnityControllerFactory‘ by extending the default controller factory. Reference.. 

  * [ASP.NET MVC Tip: Dependency Injection with Unity Application Block](<http://weblogs.asp.net/shijuvarghese/archive/2008/10/24/asp-net-mvc-tip-dependency-injection-with-unity-application-block.aspx>)
  * [DI using Unity Application Blocks](<http://www.dotnetspark.com/kb/267-di-using-unity-application-blocks.aspx>)



## Validating Business Objects

A business object used to store data. A smooth functioning of a business function depends on the validity of its associated business objects. Therefore it is essential validating business objects before them being used in business functions. As an example, let's take a business object named  _'User'_ , which has a property called  _'Password'_. Let's also assume that its length has to be at least 8 characters long. Then, before you storing the password, you need to do a length validation to make sure that the property meets its business requirement. In .NET there are several options available for object validations. 

  * Validation Application Block of Enterprise Library
  * WorkFlow Foundation, Rules Engine
  * .NET framework's built-in  _'Data Annotation Validator'_
  * ..etc.

The validation can be done at different layers of a system. Some validations can efficiently be done inside the UI (User Interface) layer. The UI layer is the faster and most efficient layer for user input validations. However there are other validations that you cannot do in UI layer due to various limitations. As an example the business logic layer is the most suitable place to validate a credit card detail. The  _'Data Annotation Validator'_ is what we will be using in our code. That allows validation in the UI layer as well as in the BLL. Additionally, as I noted above, it is part of .NET framework too. Please refer some online materials find more about  _'Data Annotation Validator'_. A quick Googling brought  _'[this](<http://msdn.microsoft.com/en-us/library/ee256141.aspx>)'_ up for me. 

## Scalability Options of the Design

Since we have separate projects for our  _'BLL'_ and  _'DAL'_ , we have the option of breaking those layers in to physically tiers too. As an example you can wrap the  _'MvcDemo.Core'_ (BLL) project with a ‘WebService’ to deploy it in a separate Application Server. You can use a ‘WebService’ proxy to communicate with the  _'BLL'_ , while keeping the _'MvcDemo'_ web site in the original server. The same technique can be used to further scale the system with  _'DAL'_ project too. ![ScalledSystem.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/ScalledSystem1.jpg) The  _'MvcDemo.Common'_ project is a special one that groups all common operations that are common to any project. As examples we do exception loggings, data type conversions, null handling, encryptions etc in our common project. You can use this project in all of your company projects. All common functions that you want to re-use can be added to this project. Over time, this common project will grow in its size and have more and more classes representing more and more sharable business functions. In addition to creating that common project, you can also extend the same concept to develop domain centric platforms too. A domain centric platform will have commonly used business functions that are specific to a particular domain. In order to build platform like that, you need to select a domain that your company is providing solutions for. As an example if your company is focusing on providing financial solutions, then you can create a platform for financial applications. This can be done by selecting and adding business functions to your platform at the completion of each financial project. This approach will gradually build a platform that you can use as the base framework to develop any financial application. 

## Summary of the DMS Design

![ArchiSummary.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/ArchiSummary1.jpg) As you can see in the diagram above, the system has three main sections. 

  * **MVC** \- This has the views and controllers. In my design, I have a controller called  _'DmsController'_. That controller supposes to control all DMS related operations. However it would be better if I had separate controllers for each business entity of my system. In that case since I have two business entities, I should have two controllers namely  _'DocumentController'_ and  _'FileController'_. A controller can associate with one or more models. As an example  _'DocumentController'_ can associate with  _'DocumentViewModel'_.  _'DocumentEditModel'_ ,_'DocumentDetailViewModel'_ ,  _'DocumentSummaryViewModel'_ etc. There are view models as well as Edit models. A view models use to display details in a view. The properties of a view models need to directly map to the properties that you display in the associated view. An edit models is different and use to capture user inputs. As an example when you upload a new document or edit properties of an existing document, you need to use a  _'DocumentEditModel'_. That has to be associated with _'DocumentEditView'_. Additionally, an Edit models need to have its respective validations defined against each property too.


  * **BLL** \- The Business Logic Layer mainly has models, and services. The Services associate with Mapping (in my case I used  _'AutoMapper'_ but there are many other feature rich libraries for this) Library to map Data Objects to Business Models and wise versa.


  * **DAL** \- The DAL mainly has repositories and repository context together with the Entity Framework entity models.



## Unit Testing

Unit testing is very important part of programming. It can be taken as the first mode of testing done on the code. That helps developers to find bugs early. The earlier the defects were detected, the easier they can be fixed. In this project, we will be using  _'Unity Application Block'_ together with  _'MsTest'_ to perform unit testing. The  _'Unity Application Block'_ allow us to develop highly loosely coupled ASP.NET MVC applications. I didn't yet complete the Unit Testing part in the source code, but I will explain you how that can be done. Look at the diagram below. This is the same diagram that I used above. You will see that I have completely removed the DAL part and have changed the  _'DocumentService'_ to  _'TestDocService'_. What I have done here is that I am trying to test the  _'DmsController'_ functions while keeping the  _'DocumentService'_ under my control. In order to do that, I developed a new  _'TestDocService'_ class by implementing the  _'IDocumentService'_ interface. Instead of using dependency that the  _'DocumentService'_ has with DAL, now I can hardcode everything inside  _'TestDocService'_. In other word, I can create a self sufficient service implementation with  _'TestDocSercvice'_. You can have few such test implementations to test various other scenarios too. As an example you can implement a  _'Test'_ version of _'DocumentService'_ to test 'Pagination' related behaviors. That can have a  _'GetDocumentList'_ method implementation that returns a  _'DocumentListViewModel '_ with few hundred of  _'DocumentViewModel'_ in it. This test implementation can be used to test the pagination part of the User Interface. ![TestingCodeInject.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/TestingCodeInject1.jpg) I found this great blog post that you can use to further read along this topic. 

  * [Walkthrough: Creating a Basic MVC Project with Unit Tests in Visual Studio](<http://msdn.microsoft.com/en-us/library/dd410597.aspx>)
  * [Dave Arlin - Bloggin'](<http://blog.davearlin.com/Trackback.aspx?guid=0c2deede-9c40-423d-b630-a550f2c6b548>)



## Other Things to be Noted

The Exception handling is one of the most import aspects of software development. It improves the quality of the software. However there are many techniques that developers use to handle exceptions. I thought it is important to consolidating exception-handling knowledge into a set of polished best practices and patterns. In my code I decided to catch/ log exceptions in 'Service' classes. You can log the exception and wrap it in a custom exception and throw it back to the UI layer. That way UI layer has to catch the exception again and redirect user to the respective error view. However I decided to cut that part off, instead I decided to return null when there is an exception. This frankly limits your options, when it comes to showing the right error message to the user. You can see the implementation of 'DocumentService' class to see how I have handled exceptions in my code. ![ExceptionHandling.jpg](http://www.codeproject.com/KB/aspnet/ASP_NET_MVC_WITH_EF/ExceptionHandling1.jpg) For further readings... 

  * [Exception Handling Best Practices in .NET](<http://www.codeproject.com/KB/architecture/exceptionbestpractices.aspx>)
  * [Best Practices for Handling Exceptions](<http://msdn.microsoft.com/en-us/library/seyhszts.aspx>)



## Conclusion

An effective design needs one to address each specific problem separately and uniquely than treating everything as if they have the same needs. When it comes to putting this into practice, it requires you to use your experience. I have this design, to take that as the basis for you to hopefully develop your own concepts in this area. At a later time you may need to revisit this write-up, perhaps to review and challenge your own concept.
