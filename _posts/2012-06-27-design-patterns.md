---
title: "Utils"
date: 2012-06-27 03:50:40 
author: shajeebhameed
layout: page
slug: design-patterns
status: publish
---

## <http://www.dotnetfunda.com/articles/article130.aspx>  
  
## Software Architecture Interview Questions Part 1 - Design Pattern Interview Questions

[(B) What are design patterns?](<http://www.dotnetfunda.com/articles/article130.aspx#Whataredesignpatterns>) [(A) Can you explain factory pattern?](<http://www.dotnetfunda.com/articles/article130.aspx#Canyouexplainfactorypattern>) [(I) Can you explain abstract factory pattern?](<http://www.dotnetfunda.com/articles/article130.aspx#Canyouexplainabstractfactorypattern>) [(I)Can you explain builder pattern?](<http://www.dotnetfunda.com/articles/article130.aspx#Canyouexplainbuilderpattern>) [(I) Can you explain prototype pattern?](<http://www.dotnetfunda.com/articles/article130.aspx#Canyouexplainprototypepattern>) [(A) Can you explain shallow copy and deep copy in prototype patterns?](<http://www.dotnetfunda.com/articles/article130.aspx#Canyouexplainshallowcopyanddeepcopyinprototypepatterns>) [(B) Can you explain singleton pattern?](<http://www.dotnetfunda.com/articles/article130.aspx#Canyouexplainsingletonpattern>) [(A) Can you explain command patterns?](<http://www.dotnetfunda.com/articles/article130.aspx#Canyouexplaincommandpatterns>) [Other Interview question PDF's](<http://www.dotnetfunda.com/articles/article130.aspx#Other_Interview_question_PDFs_>)

### Introduction

Hi friends , Please do not think you get an architecture position by reading interview questions. But yes there should be some kind of reference which will help you quickly revise what are the definition. Just by reading these answers you get to a position where you are aware of the fundamentals. But if you have not really worked you will surely fail with scenario based questions. So use this as a quick revision rather than a short cut. 

In case your are completely new to design patterns or you really do not want to read this complete article do see our free [design pattern Training and interview questions / answers ](<http://www.questpond.com/DP/Design-Pattern-Training-and-Interview-Questions-and-Answers.html>)videos.

My Other Design Pattern Articles 

  1. Design pattern Interview Questions Part 2 :- [Interpreter pattern, iterator pattern, mediator pattern, memento pattern and observer pattern](<http://www.dotnetfunda.com/articles/article135.aspx>)
  2. Design pattern Interview Questions Part 3 :- [state pattern, strategy pattern, visitor pattern, adapter pattern and fly weight pattern](<http://www.dotnetfunda.com/articles/article137.aspx>)
  3. Design Patterb Interview Questions Part 4 :- [bridge pattern, composite pattern, decorator pattern, Façade pattern, chain of responsibility(COR), proxy pattern and template pattern](<http://www.dotnetfunda.com/articles/article139.aspx>)

Happy job hunting...... 

### (B) What are design patterns?

Design patterns are documented tried and tested solutions for recurring problems in a given context. So basically you have a problem context and the proposed solution for the same. Design patterns existed in some or other form right from the inception stage of software development. Let’s say if you want to implement a sorting algorithm the first thing comes to mind is bubble sort. So the problem is sorting and solution is bubble sort. Same holds true for design patterns. 

### (I) Which are the three main categories of design patterns?

There are three basic classifications of patterns Creational, Structural, and Behavioral patterns. **Creational Patterns** • **Abstract Factory** :- Creates an instance of several families of classes • **Builder** : - Separates object construction from its representation • **Factory Method** :- Creates an instance of several derived classes • **Prototype** :- A fully initialized instance to be copied or cloned • **Singleton** :- A class in which only a single instance can exist **_Note: - The best way to remember Creational pattern is by remembering ABFPS (Abraham Became First President of States). Structural Patterns_** • **Adapter** :-Match interfaces of different classes . • **Bridge** :-Separates an object’s abstraction from its implementation. • **Composite** :-A tree structure of simple and composite objects. • **Decorator** :-Add responsibilities to objects dynamically. • **Façade** :-A single class that represents an entire subsystem. • **Flyweight** :-A fine-grained instance used for efficient sharing. • **Proxy** :-An object representing another object. _**Note : To remember structural pattern best is (ABCDFFP) Behavioral Patterns**_ • **Mediator** :-Defines simplified communication between classes. • **Memento** :-Capture and restore an object's internal state. • **Interpreter** :- A way to include language elements in a program. • **Iterator** :-Sequentially access the elements of a collection. • **Chain of Resp** : - A way of passing a request between a chain of objects. • **Command** :-Encapsulate a command request as an object. • **State** :-Alter an object's behavior when its state changes. • **Strategy** :-Encapsulates an algorithm inside a class. • **Observer** : - A way of notifying change to a number of classes. • **Template Method** :-Defer the exact steps of an algorithm to a subclass. • **Visitor** :-Defines a new operation to a class without change. _**Note: - Just remember Music....... 2 MICS On TV (MMIICCSSOTV).**_ _**Note :- In the further section we will be covering all the above design patterns in a more detail manner.**_

### (A) Can you explain factory pattern?

  * Factory pattern is one of the types of creational patterns. You can make out from the name factory itself it’s meant to construct and create something. In software architecture world factory pattern is meant to centralize creation of objects. Below is a code snippet of a client which has different types of invoices. These invoices are created depending on the invoice type specified by the client. There are two issues with the code below:-

  * First we have lots of ‘new’ keyword scattered in the client. In other ways the client is loaded with lot of object creational activities which can make the client logic very complicated. Second issue is that the client needs to be aware of all types of invoices. So if we are adding one more invoice class type called as ‘InvoiceWithFooter’ we need to reference the new class in the client and recompile the client also.




**Figure: - Different types of invoice**

Taking these issues as our base we will now look in to how factory pattern can help us solve the same. Below figure ‘Factory Pattern’ shows two concrete classes ‘ClsInvoiceWithHeader’ and ‘ClsInvoiceWithOutHeader’.

The **first issue** was that these classes are in direct contact with client which leads to lot of ‘new’ keyword scattered in the client code. This is removed by introducing a new class ‘ClsFactoryInvoice’ which does all the creation of objects.

The **second issue** was that the client code is aware of both the concrete classes i.e. ‘ClsInvoiceWithHeader’ and ‘ClsInvoiceWithOutHeader’. This leads to recompiling of the client code when we add new invoice types. For instance if we add ‘ClsInvoiceWithFooter’ client code needs to be changed and recompiled accordingly. To remove this issue we have introduced a common interface ‘IInvoice’. Both the concrete classes ‘ClsInvoiceWithHeader’ and ‘ClsInvoiceWithOutHeader’ inherit and implement the ‘IInvoice’ interface. The client references only the ‘IInvoice’ interface which results in zero connection between client and the concrete classes ( ‘ClsInvoiceWithHeader’ and ‘ClsInvoiceWithOutHeader’). So now if we add new concrete invoice class we do not need to change any thing at the client side. In one line the creation of objects is taken care by ‘ClsFactoryInvoice’ and the client disconnection from the concrete classes is taken care by ‘IInvoice’ interface. 

**Figure: - Factory pattern**

Below are the code snippets of how actually factory pattern can be implemented in C#. In order to avoid recompiling the client we have introduced the invoice interface ‘IInvoice’. Both the concrete classes ‘ClsInvoiceWithOutHeaders’ and ‘ClsInvoiceWithHeader’ inherit and implement the ‘IInvoice’ interface.

**Figure :- Interface and concrete classes**

We have also introduced an extra class ‘ClsFactoryInvoice’ with a function ‘getInvoice()’ which will generate objects of both the invoices depending on ‘intInvoiceType’ value. In short we have centralized the logic of object creation in the ‘ClsFactoryInvoice’. The client calls the ‘getInvoice’ function to generate the invoice classes. One of the most important points to be noted is that client only refers to ‘IInvoice’ type and the factory class ‘ClsFactoryInvoice’ also gives the same type of reference. This helps the client to be complete detached from the concrete classes, so now when we add new classes and invoice types we do not need to recompile the client.

**Figure: - Factory class which generates objects**

**_Note :- The above example is given in C# . Even if you are from some other technology you can still map the concept accordingly. You can get source code from the CD in ‘FactoryPattern’ folder._**

### (I) Can you explain abstract factory pattern?

Abstract factory expands on the basic factory pattern. Abstract factory helps us to unite similar factory pattern classes in to one unified interface. So basically all the common factory patterns now inherit from a common abstract factory class which unifies them in a common class. All other things related to factory pattern remain same as discussed in the previous question. A factory class helps us to centralize the creation of classes and types. Abstract factory helps us to bring uniformity between related factory patterns which leads more simplified interface for the client. 

**Figure: - Abstract factory unifies related factory patterns**

Now that we know the basic lets try to understand the details of how abstract factory patterns are actually implemented. As said previously we have the factory pattern classes (factory1 and factory2) tied up to a common abstract factory (AbstractFactory Interface) via inheritance. Factory classes stand on the top of concrete classes which are again derived from common interface. For instance in figure ‘Implementation of abstract factory’ both the concrete classes ‘product1’ and ‘product2’ inherits from one interface i.e. ‘common’. The client who wants to use the concrete class will only interact with the abstract factory and the common interface from which the concrete classes inherit.

**Figure: - Implementation of abstract factory**

Now let’s have a look at how we can practically implement abstract factory in actual code. We have scenario where we have UI creational activities for textboxes and buttons through their own centralized factory classes ‘ClsFactoryButton’ and ‘ClsFactoryText’. Both these classes inherit from common interface ‘InterfaceRender’. Both the factories ‘ClsFactoryButton’ and ‘ClsFactoryText’ inherits from the common factory ‘ClsAbstractFactory’. Figure ‘Example for AbstractFactory’ shows how these classes are arranged and the client code for the same. One of the important points to be noted about the client code is that it does not interact with the concrete classes. For object creation it uses the abstract factory ( ClsAbstractFactory ) and for calling the concrete class implementation it calls the methods via the interface ‘InterfaceRender’. So the ‘ClsAbstractFactory’ class provides a common interface for both factories ‘ClsFactoryButton’ and ‘ClsFactoryText’.

**Figure: - Example for abstract factory**

_**Note: - We have provided a code sample in C# in the ‘AbstractFactory’ folder. People who are from different technology can compare easily the implementation in their own language.**_

We will just run through the sample code for abstract factory. Below code snippet ‘Abstract factory and factory code snippet’ shows how the factory pattern classes inherit from abstract factory.

**Figure: - Abstract factory and factory code snippet**

Figure ‘Common Interface for concrete classes’ how the concrete classes inherits from a common interface ‘InterFaceRender’ which enforces the method ‘render’ in all the concrete classes. 

**Figure: - Common interface for concrete classes**

****The final thing is the client code which uses the interface ‘InterfaceRender’ and abstract factory ‘ClsAbstractFactory’ to call and create the objects. One of the important points about the code is that it is completely isolated from the concrete classes. Due to this any changes in concrete classes like adding and removing concrete classes does not need client level changes.

**Figure: - Client, interface and abstract factory**

### (I)Can you explain builder pattern?

Builder falls under the type of creational pattern category. Builder pattern helps us to separate the construction of a complex object from its representation so that the same construction process can create different representations. Builder pattern is useful when the construction of the object is very complex. The main objective is to separate the construction of objects and their representations. If we are able to separate the construction and representation, we can then get many representations from the same construction.

**Figure: - Builder concept**

To understand what we mean by construction and representation lets take the example of the below ‘Tea preparation’ sequence. You can see from the figure ‘Tea preparation’ from the same preparation steps we can get three representation of tea’s (i.e. Tea with out sugar, tea with sugar / milk and tea with out milk).

### 

**Figure: - Tea preparation**

Now let’s take a real time example in software world to see how builder can separate the complex creation and its representation. Consider we have application where we need the same report to be displayed in either ‘PDF’ or ‘EXCEL’ format. Figure ‘Request a report’ shows the series of steps to achieve the same. Depending on report type a new report is created, report type is set, headers and footers of the report are set and finally we get the report for display.

### 

**Figure: - Request a report**

Now let’s take a different view of the problem as shown in figure ‘Different View’. The same flow defined in ‘Request a report’ is now analyzed in representations and common construction. The construction process is same for both the types of reports but they result in different representations.

### 

**Figure: - Different View**

We will take the same report problem and try to solve the same using builder patterns. There are three main parts when you want to implement builder patterns. • **Builder** : - Builder is responsible for defining the construction process for individual parts. Builder has those individual processes to initialize and configure the product. • **Director** : - Director takes those individual processes from the builder and defines the sequence to build the product. • **Product** : - Product is the final object which is produced from the builder and director coordination. First let’s have a look at the builder class hierarchy. We have a abstract class called as ‘ReportBuilder’ from which custom builders like ‘ReportPDF’ builder and ‘ReportEXCEL’ builder will be built.

### 

**Figure: - Builder class hierarchy**

Figure ‘Builder classes in actual code’ shows the methods of the classes. To generate report we need to first Create a new report, set the report type (to EXCEL or PDF) , set report headers , set the report footers and finally get the report. We have defined two custom builders one for ‘PDF’ (ReportPDF) and other for ‘EXCEL’ (ReportExcel). These two custom builders define there own process according to the report type.

### 

**Figure: - Builder classes in actual code**

Now let’s understand how director will work. Class ‘clsDirector’ takes the builder and calls the individual method process in a sequential manner. So director is like a driver who takes all the individual processes and calls them in sequential manner to generate the final product, which is the report in this case. Figure ‘Director in action’ shows how the method ‘MakeReport’ calls the individual process to generate the report product by PDF or EXCEL.

**Figure: - Director in action**

The third component in the builder is the product which is nothing but the report class in this case.

### 

**Figure: - The report class**

Now let’s take a top view of the builder project. Figure ‘Client,builder,director and product’ shows how they work to achieve the builder pattern. Client creates the object of the director class and passes the appropriate builder to initialize the product. Depending on the builder the product is initialized/created and finally sent to the client.

**Figure: - Client, builder, director and product**

The output is something like this. We can see two report types displayed with their headers according to the builder.

### 

**Figure: - Final output of builder**

_**Note :- In CD we have provided the above code in C# in ‘BuilderPattern’ folder.**_

### (I) Can you explain prototype pattern?

Prototype pattern falls in the section of creational pattern. It gives us a way to create new objects from the existing instance of the object. In one sentence we clone the existing object with its data. By cloning any changes to the cloned object does not affect the original object value. If you are thinking by just setting objects we can get a clone then you have mistaken it. By setting one object to other object we set the reference of object BYREF. So changing the new object also changed the original object. To understand the BYREF fundamental more clearly consider the figure ‘BYREF’ below. Following is the sequence of the below code:- • In the first step we have created the first object i.e. obj1 from class1. • In the second step we have created the second object i.e. obj2 from class1. • In the third step we set the values of the old object i.e. obj1 to ‘old value’. • In the fourth step we set the obj1 to obj2. • In the fifth step we change the obj2 value. • Now we display both the values and we have found that both the objects have the new value.

### 

**Figure :- BYREf**

The conclusion of the above example is that objects when set to other objects are set BYREF. So changing new object values also changes the old object value. There are many instances when we want the new copy object changes should not affect the old object. The answer to this is prototype patterns. Lets look how we can achieve the same using C#. In the below figure ‘Prototype in action’ we have the customer class ‘ClsCustomer’ which needs to be cloned. This can be achieved in C# my using the ‘MemberWiseClone’ method. In JAVA we have the ‘Clone’ method to achieve the same. In the same code we have also shown the client code. We have created two objects of the customer class ‘obj1’ and ‘obj2’. Any changes to ‘obj2’ will not affect ‘obj1’ as it’s a complete cloned copy.

**Figure: - Prototype in action**

_**Note :- You can get the above sample in the CD in ‘Prototype’ folder. In C# we use the ‘MemberWiseClone’ function while in JAVA we have the ‘Clone’ function to achieve the same.**_

### (A) Can you explain shallow copy and deep copy in prototype patterns?

There are two types of cloning for prototype patterns. One is the shallow cloning which you have just read in the first question. In shallow copy only that object is cloned, any objects containing in that object is not cloned. For instance consider the figure ‘Deep cloning in action’ we have a customer class and we have an address class aggregated inside the customer class. ‘MemberWiseClone’ will only clone the customer class ‘ClsCustomer’ but not the ‘ClsAddress’ class. So we added the ‘MemberWiseClone’ function in the address class also. Now when we call the ‘getClone’ function we call the parent cloning function and also the child cloning function, which leads to cloning of the complete object. When the parent objects are cloned with their containing objects it’s called as deep cloning and when only the parent is clones its termed as shallow cloning.

### 

**Figure: - Deep cloning in action**

### (B) Can you explain singleton pattern?
    
    
    Punch :-  Create a single instance of object and provides access to this single instance via a central point.

There are situations in project where we want only one instance of the object to be created and shared between the clients. For instance let’s say we have the below two classes, currency and country. These classes load master data which will be referred again and again in project. We would like to share a single instance of the class to get performance benefit by not hitting the database again and again. 
    
    
    public class Currency
        {
            List<string /> oCurrencies = new List<string />();
            public Currency()
            {
                oCurrencies.Add("INR");
                oCurrencies.Add("USD");
                oCurrencies.Add("NEP");
                oCurrencies.Add("GBP");
    
            }
            public IEnumerable<string /> getCurrencies()
            {
                return (IEnumerable<string />)oCurrencies;
            }
        }
    
    public class Country
        {
            List<string /> oCountries = new List<string />();
            public Country()
            {
                oCountries.Add("India");
                oCountries.Add("Nepal");
                oCountries.Add("USA");
                oCountries.Add("UK");
    
            }
            public IEnumerable<string /> getCounties()
            {
                return (IEnumerable<string />) oCountries;
            }
        }

Implementing singleton pattern is a 4 step process. **Step 1: - Create a sealed class with a private constructor** The private constructor is important so that clients cannot create object of the class directly. If you remember the punch, the main intention of this pattern is to create a single instance of the object which can be shared globally, so we do not want to give client the control to create instances directly. 
    
    
    public sealed class GlobalSingleton
        {
    private GlobalSingleton() { }
    
    	………..
    	……….
    	}

**Step 2: - Create aggregated objects of the classes (for this example it is currency and country) for which we want single instance.**
    
    
    public sealed class GlobalSingleton
        {
            // object which needs to be shared globally
            public Currency Currencies = new Currency();
            public Country Countries = new Country();

**Step 3: - Create a static read-only object of t he class itself and expose the same via static property as shown below.**
    
    
    public sealed class GlobalSingleton
        {
    	….
    	…
    // use static variable to create a single instance of the object
    static readonly GlobalSingleton INSTANCE = new GlobalSingleton();
    
    public static GlobalSingleton Instance
            {
                get
                {
    
                    return INSTANCE;
                }
            }       
        }

**Step 4: - You can now consume the singleton object using the below code from the client.**
    
    
    GlobalSingleton oSingle = GlobalSingleton.Instance;
    Country o = osingl1.Country;

Below goes the complete code for singleton pattern which we discussed above in pieces. 
    
    
    public sealed class GlobalSingleton
        {
            // object which needs to be shared globally
            public Currency Currencies = new Currency();
            public Country Countries = new Country();
    
            // use static variable to create a single instance of the object
            static readonly GlobalSingleton INSTANCE = new GlobalSingleton();
    
            /// This is a private constructor, meaning no outsides have access.
            private GlobalSingleton() 
            { }
    
    public static GlobalSingleton Instance
            {
                get
                {
    
                    return INSTANCE;
                }
            }       
        }

### (A) Can you explain command patterns?

Command pattern allows a request to exist as an object. Ok let’s understand what it means. Consider the figure ‘Menu and Commands’ we have different actions depending on which menu is clicked. So depending on which menu is clicked we have passed a string which will have the action text in the action string. Depending on the action string we will execute the action. The bad thing about the code is it has lot of ‘IF’ condition which makes the coding more cryptic.

**Figure: - Menu and Commands**

Command pattern moves the above action in to objects. These objects when executed actually execute the command. As said previously every command is an object. We first prepare individual classes for every action i.e. exit, open, file and print. Al l the above actions are wrapped in to classes like Exit action is wrapped in ‘clsExecuteExit’ , open action is wrapped in ‘clsExecuteOpen’, print action is wrapped in ‘clsExecutePrint’ and so on. All these classes are inherited from a common interface ‘IExecute’.

**Figure: - Objects and Command**

Using all the action classes we can now make the invoker. The main work of invoker is to map the action with the classes which have the action. So we have added all the actions in one collection i.e. the arraylist. We have exposed a method ‘getCommand’ which takes a string and gives back the abstract object ‘IExecute’. The client code is now neat and clean. All the ‘IF’ conditions are now moved to the ‘clsInvoker’ class.

### 

**Figure: - Invoker and the clean client**
