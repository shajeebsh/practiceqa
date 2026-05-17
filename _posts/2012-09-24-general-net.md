---
title: "General .Net"
date: 2012-09-24 09:56:51 
author: shajeebhameed
layout: page
slug: general-net
status: publish
---

**Recommendations for Web User Controls vs. Web Custom Controls**

If none of the existing ASP.NET server controls meet the specific requirements of your applications, you can create either a Web user control or a Web custom control that encapsulates the functionality you need. The main difference between the two controls lies in ease of creation vs. ease of use at design time. 

Web user controls are easy to make, but they can be less convenient to use in advanced scenarios. You develop Web user controls almost exactly the same way that you develop Web Forms pages. Like Web Forms, user controls can be created in the visual designer, they can be written with code separated from the HTML, and they can handle execution events. However, because Web user controls are compiled dynamically at run time they cannot be added to the Toolbox, and they are represented by a simple placeholder glyph when added to a page. This makes Web user controls harder to use if you are accustomed to full Visual Studio .NET design-time support, including the Properties window and Design view previews. Also, the only way to share the user control between applications is to put a separate copy in each application, which takes more maintenance if you make changes to the control.

Web custom controls are compiled code, which makes them easier to use but more difficult to create; Web custom controls must be authored in code. Once you have created the control, however, you can add it to the Toolbox and display it in a visual designer with full Properties window support and all the other design-time features of ASP.NET server controls. In addition, you can install a single copy of the Web custom control in the global assembly cache and share it between applications, which makes maintenance easier. For more information see [global assembly cache](<http://msdn.microsoft.com/en-us/library/aa720110\(VS.71\).aspx>).

If your control has a lot of static layout, a user control might make sense. If your control is mostly dynamically generated — for instance rows of a data-bound table, nodes of a tree view, or tabs of a tab control — a custom control would be a better choice.

The main differences between the two types are outlined in this table:

**Web user controls** |  **Web custom controls**  
---|---  
Easier to create |  Harder to create  
Limited support for consumers who use a visual design tool |  Full visual design tool support for consumers  
A separate copy of the control is required in each application |  Only a single copy of the control is required, in the global assembly cache  
Cannot be added to the Toolbox in Visual Studio |  Can be added to the Toolbox in Visual Studio  
Good for static layout |  Good for dynamic layout  
  
**Factors** |  **Truncate** |  **Delete**  
---|---|---  
Definition |  Truncate Table is a statement that quickly deletes all records in a table by de-allocating the data pages used by the table. |  Delete Table statement deletes one row at a time by logging each row in the transaction log as well as maintaining a Log Sequence Number (LSN) information  
Logging |  The only logging performed is de-allocation |  Each row that is deleted is logged  
Restored |  Data cannot be restored |  Data Can be restored  
Speed |  Faster |  Slower  
Selective Deletion |  Selective deletion is not possible |  Selective Deletion is possible  
Identity to SEED |  Truncate resets the IDENTITY back to the SEED and de-allocated pages are returned to the system to be used in other areas |  Does not reset the IDENTITY back to SEED. When you delete large number of rows using delete statement, the table may hang on to empty pages requiring manual release using DBCC SHRINKDATABASE (db_name)  
Replication |  Cannot be used for tables involved in replication |  Can be used  
Foreign Key References |  Cannot be used when a Foreign Key references to the table being truncated |  Can be used  
Triggers |  Truncate Statements don’t not Fire Triggers |  Delete statement fires triggers  
  
**Factors** |  **Stored procedures** |  **Functions**  
---|---|---  
Return Value |  May return a scalar value or a table or nothing at all |  Must always return a value (a scalar value or a table)  
Call/Execution |  Can only be Executed, It cannot be called within a SQL Query |  It can be called within a SQL Query.  
Usage |  Used whenever DML is required |  Used for read only calls  
Compilation |  Complied only once |  Compiles each time when a function is called  
Contain DML |  Can have DML inside Stored Procedure |  Can’t use DML inside a function that is called in a SELECT statement.  
**Factors** |  **Server.Transfer** |  **Response.Redirect**  
---|---|---  
Round Trips |  Does not involve an additional round trip and hence is faster |  It involves an additional round trip to the server  
Transfer |  Can work on a site that is residing in the same web server. Cannot use to send a user to an external site |  Can send users to an external site  
URL |  Maintains the original url in the browser |  Does not maintain the original url in the browser  
PreserveForm |  If the PreserveForm parameter is set to true, then the original query string and form variables will be available in the transferred page also |  PreserveForm parameter is not available  
  
**1.1 What is .NET?**

.NET is a general-purpose software development platform, similar to Java. At its core is a virtual machine that turns intermediate language (IL) into machine code. High-level language compilers for C#, VB.NET and C++ are provided to turn source code into IL. C# is a new programming language, very similar to Java. An extensive class library is included, featuring all the functionality one might expect from a contempory development platform - windows GUI development (Windows Forms), database access (ADO.NET), web development (ASP.NET), web services, XML etc.

See also Microsoft's [definition](<http://msdn.microsoft.com/netframework/gettingstarted/default.aspx>).

**1.2 When was .NET announced?**

Bill Gates delivered a keynote at Forum 2000, held June 22, 2000, outlining the .NET 'vision'. The July 2000 PDC had a number of sessions on .NET technology, and delegates were given CDs containing a pre-release version of the .NET framework/SDK and Visual Studio.NET.

**1.3 What versions of .NET are there?**

The final versions of the 1.0 SDK and runtime were made publicly available around 6pm PST on 15-Jan-2002. At the same time, the final version of Visual Studio.NET was made available to MSDN subscribers.

.NET 1.1 was released in April 2003, and was mostly bug fixes for 1.0.

.NET 2.0 was released to MSDN subscribers in late October 2005, and was officially launched in early November.

**1.4 What operating systems does the .NET Framework run on?**

The runtime supports Windows Server 2003, Windows XP, Windows 2000, NT4 SP6a and Windows ME/98. Windows 95 is not supported. Some parts of the framework do not work on all platforms - for example, ASP.NET is only supported on XP and Windows 2000/2003. Windows 98/ME cannot be used for development.

IIS is not supported on Windows XP Home Edition, and so cannot be used to host ASP.NET. However, the [ASP.NET Web Matrix](<http://www.asp.net/>) web server does run on XP Home.

The [.NET Compact Framework](<http://msdn.microsoft.com/mobility/netcf/>) is a version of the .NET Framework for mobile devices, running Windows CE or Windows Mobile.

The [Mono](<http://www.mono-project.com/>) project has a version of the .NET Framework that runs on Linux.

**1.5 What tools can I use to develop .NET applications?**

There are a number of tools, described here in ascending order of cost:

  * The [.NET Framework SDK](<http://msdn.microsoft.com/netframework/>) is free and includes command-line compilers for C++, C#, and VB.NET and various other utilities to aid development. 

  * [SharpDevelop](<http://www.icsharpcode.net/OpenSource/SD/Default.aspx>) is a free IDE for C# and VB.NET. 

  * [Microsoft Visual Studio Express](<http://msdn.microsoft.com/vstudio/express/>) editions are cut-down versions of Visual Studio, for hobbyist or novice developers.There are different versions for C#, VB, web development etc. Originally the plan was to charge $49, but MS has decided to offer them as free downloads instead, at least until November 2006. 

  * [Microsoft Visual Studio Standard 2005](<http://www.amazon.com/exec/obidos/ASIN/B000BT8TRQ/ref=nosim/andymcmullsho-20>) is around $300, or $200 for the [upgrade](<http://www.amazon.com/exec/obidos/ASIN/B000BT8TS0/ref=nosim/andymcmullsho-20>). 

  * [Microsoft VIsual Studio Professional 2005](<http://www.amazon.com/exec/obidos/ASIN/B000BTA4LU/ref=nosim/andymcmullsho-20>) is around $800, or $550 for the [upgrade](<http://www.amazon.com/exec/obidos/ASIN/B000BT8TRG/ref=nosim/andymcmullsho-20>). 

  * At the top end of the price range are the [Microsoft Visual Studio Team Edition for Software Developers 2005 with MSDN Premium](<http://www.amazon.com/exec/obidos/ASIN/B000BAWKPM/ref=nosim/andymcmullsho-20>) and [Team Suite](<http://www.amazon.com/exec/obidos/ASIN/B000BARBAQ/ref=nosim/andymcmullsho-20>) editions. 




You can see the differences between the various Visual Studio versions [here](<http://lab.msdn.microsoft.com/vs2005/productinfo/productline/default.aspx>). 

**1.6 Why did they call it .NET?**

I don't know what they were thinking. They certainly weren't thinking of people using search tools. It's meaningless marketing nonsense.

**2\. Terminology**

**2.1 What is the CLI? Is it the same as the CLR?**

The CLI (Common Language Infrastructure) is the definiton of the fundamentals of the .NET framework - the Common Type System (CTS), metadata, the Virtual Execution Environment (VES) and its use of intermediate language (IL), and the support of multiple programming languages via the Common Language Specification (CLS). The CLI is documented through ECMA - see [http://msdn.microsoft.com/net/ecma/](<http://msdn.microsoft.com/net/ecma/>) for more details.

The CLR (Common Language Runtime) is Microsoft's primary _implementation_ of the CLI. Microsoft also have a shared source implementation known as [ROTOR](<http://msdn.microsoft.com/net/sscli/>), for educational purposes, as well as the [.NET Compact Framework](<http://msdn.microsoft.com/mobility/netcf/>) for mobile devices. Non-Microsoft CLI implementations include [Mono](<http://www.mono-project.com/>) and [DotGNU Portable.NET](<http://www.dotgnu.org/>).

**2.2 What is the CTS, and how does it relate to the CLS?**

CTS = Common Type System. This is the full range of types that the .NET runtime understands. Not all .NET languages support all the types in the CTS. 

CLS = Common Language Specification. This is a subset of the CTS which _all_ .NET languages are expected to support. The idea is that any program which uses CLS-compliant types can interoperate with any .NET program written in any language. This interop is very fine-grained - for example a VB.NET class can inherit from a C# class.

**2.3 What is IL?**

IL = Intermediate Language. Also known as MSIL (Microsoft Intermediate Language) or CIL (Common Intermediate Language). All .NET source code (of any language) is compiled to IL during development. The IL is then converted to machine code at the point where the software is installed, or (more commonly) at run-time by a Just-In-Time (JIT) compiler.

**2.4 What is C#?**

C# is a new language designed by Microsoft to work with the .NET framework. In their "Introduction to C#" whitepaper, Microsoft describe C# as follows:

"C# is a simple, modern, object oriented, and type-safe programming language derived from C and C++. C# (pronounced “C sharp”) is firmly planted in the C and C++ family tree of languages, and will immediately be familiar to C and C++ programmers. C# aims to combine the high productivity of Visual Basic and the raw power of C++."

Substitute 'Java' for 'C#' in the quote above, and you'll see that the statement still works pretty well :-).

If you are a C++ programmer, you might like to check out my [C# FAQ](<http://www.andymcm.com/csharpfaq.htm>).

**2.5 What does 'managed' mean in the .NET context?**

The term 'managed' is the cause of much confusion. It is used in various places within .NET, meaning slightly different things.

Managed _code_ : The .NET framework provides several core run-time services to the programs that run within it - for example exception handling and security. For these services to work, the code must provide a minimum level of information to the runtime. Such code is called managed code. 

Managed _data_ : This is data that is allocated and freed by the .NET runtime's garbage collector.

Managed _classes_ : This is usually referred to in the context of Managed Extensions (ME) for C++. When using ME C++, a class can be marked with the __gc keyword. As the name suggests, this means that the memory for instances of the class is managed by the garbage collector, but it also means more than that. The class becomes a fully paid-up member of the .NET community with the benefits and restrictions that brings. An example of a benefit is proper interop with classes written in other languages - for example, a managed C++ class can inherit from a VB class. An example of a restriction is that a managed class can only inherit from one base class.

**2.6 What is reflection?**

All .NET compilers produce metadata about the types defined in the modules they produce. This metadata is packaged along with the module (modules in turn are packaged together in assemblies), and can be accessed by a mechanism called **reflection**. The System.Reflection namespace contains classes that can be used to interrogate the types for a module/assembly.

Using reflection to access .NET metadata is very similar to using ITypeLib/ITypeInfo to access type library data in COM, and it is used for similar purposes - e.g. determining data type sizes for marshaling data across context/process/machine boundaries.

Reflection can also be used to dynamically invoke methods (see System.Type.InvokeMember), or even create types dynamically at run-time (see System.Reflection.Emit.TypeBuilder). 

**3\. Assemblies**

**3.1 What is an assembly?**

An assembly is sometimes described as a logical .EXE or .DLL, and can be an _application_ (with a main entry point) or a _library_. An assembly consists of one or more files (dlls, exes, html files etc), and represents a group of resources, type definitions, and implementations of those types. An assembly may also contain references to other assemblies. These resources, types and references are described in a block of data called a _manifest_. The manifest is part of the assembly, thus making the assembly self-describing.

An important aspect of assemblies is that they are part of the identity of a type. The identity of a type is the assembly that houses it combined with the type name. This means, for example, that if assembly A exports a type called T, and assembly B exports a type called T, the .NET runtime sees these as two completely different types. Furthermore, don't get confused between assemblies and namespaces - namespaces are merely a hierarchical way of organising type names. To the runtime, type names are type names, regardless of whether namespaces are used to organise the names. It's the assembly plus the typename (regardless of whether the type name belongs to a namespace) that uniquely indentifies a type to the runtime.

Assemblies are also important in .NET with respect to security - many of the security restrictions are enforced at the assembly boundary.

Finally, assemblies are the unit of versioning in .NET - more on this below. 

**3.2 How can I produce an assembly?**

The simplest way to produce an assembly is directly from a .NET compiler. For example, the following C# program:

public class CTest

{

public CTest() { System.Console.WriteLine( "Hello from CTest" ); }

}

can be compiled into a library assembly (dll) like this:

csc /t:library ctest.cs

You can then view the contents of the assembly by running the "IL Disassembler" tool that comes with the .NET SDK.

Alternatively you can compile your source into modules, and then combine the modules into an assembly using the assembly linker (al.exe). For the C# compiler, the /target:module switch is used to generate a module instead of an assembly.

**3.3 What is the difference between a private assembly and a shared assembly?**

  * **Location and visibility** : A private assembly is normally used by a single application, and is stored in the application's directory, or a sub-directory beneath. A shared assembly is normally stored in the global assembly cache, which is a repository of assemblies maintained by the .NET runtime. Shared assemblies are usually libraries of code which many applications will find useful, e.g. the .NET framework classes. 

  * **Versioning** : The runtime enforces versioning constraints only on shared assemblies, not on private assemblies.




**3.4 How do assemblies find each other?**

By searching directory paths. There are several factors which can affect the path (such as the AppDomain host, and application configuration files), but for private assemblies the search path is normally the application's directory and its sub-directories. For shared assemblies, the search path is normally same as the private assembly path plus the shared assembly cache.

**3.5 How does assembly versioning work?**

Each assembly has a version number called the compatibility version. Also each reference to an assembly (from another assembly) includes both the name and version of the referenced assembly.

The version number has four numeric parts (e.g. 5.5.2.33). Assemblies with either of the first two parts different are normally viewed as incompatible. If the first two parts are the same, but the third is different, the assemblies are deemed as 'maybe compatible'. If only the fourth part is different, the assemblies are deemed compatible. However, this is just the default guideline - it is the version policy that decides to what extent these rules are enforced. The version policy can be specified via the application configuration file.

**Remember** : versioning is only applied to shared assemblies, not private assemblies.

**3.6 How can I develop an application that automatically updates itself from the web?**

For .NET 1.x, use the [Updater Application Block](<http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dnbda/html/updater.asp>). For .NET 2.x, use [ClickOnce](<http://msdn.microsoft.com/longhorn/understanding/pillars/fundamentals/default.aspx?pull=/library/en-us/dnwinforms/html/clickonce.asp>).

**4\. Application Domains**

**4.1 What is an application domain?**

An AppDomain can be thought of as a lightweight process. Multiple AppDomains can exist inside a Win32 process. The primary purpose of the AppDomain is to isolate applications from each other, and so it is particularly useful in hosting scenarios such as ASP.NET. An AppDomain can be destroyed by the host without affecting other AppDomains in the process.

Win32 processes provide isolation by having distinct memory address spaces. This is effective, but expensive. The .NET runtime enforces AppDomain isolation by keeping control over the use of memory - all memory in the AppDomain is managed by the .NET runtime, so the runtime can ensure that AppDomains do not access each other's memory.

One non-obvious use of AppDomains is for unloading types. Currently the only way to unload a .NET type is to destroy the AppDomain it is loaded into. This is particularly useful if you create and destroy types on-the-fly via reflection.

Microsoft have an [AppDomain FAQ](<http://www.gotdotnet.com/team/clr/AppdomainFAQ.aspx>).

**4.2 How does an AppDomain get created?**

AppDomains are usually created by hosts. Examples of hosts are the Windows Shell, ASP.NET and IE. When you run a .NET application from the command-line, the host is the Shell. The Shell creates a new AppDomain for every application.

AppDomains can also be explicitly created by .NET applications. Here is a C# sample which creates an AppDomain, creates an instance of an object inside it, and then executes one of the object's methods:

using System;

using System.Runtime.Remoting;

using System.Reflection;

public class CAppDomainInfo : MarshalByRefObject

{

public string GetName() { return AppDomain.CurrentDomain.FriendlyName; }

}

public class App

{

public static int Main()

{

AppDomain ad = AppDomain.CreateDomain( "Andy's new domain" );

CAppDomainInfo adInfo = (CAppDomainInfo)ad.CreateInstanceAndUnwrap( 

Assembly.GetCallingAssembly().GetName().Name, "CAppDomainInfo" ); 

Console.WriteLine( "Created AppDomain name = " + adInfo.GetName() );

return 0;

}

}

**4.3 Can I write my own .NET host?**

Yes. For an example of how to do this, take a look at the source for the [dm.net moniker](<http://staff.develop.com/jasonw/clr/readme.htm>) developed by Jason Whittington and Don Box. There is also a code sample in the .NET SDK called CorHost.

**5\. Garbage Collection**

**5.1 What is garbage collection?**

Garbage collection is a heap-management strategy where a run-time component takes responsibility for managing the lifetime of the memory used by objects. This concept is not new to .NET - Java and many other languages/runtimes have used garbage collection for some time. 

**5.2 Is it true that objects don't always get destroyed immediately when the last reference goes away?**

Yes. The garbage collector offers no guarantees about the time when an object will be destroyed and its memory reclaimed.

There was an [interesting thread](<http://discuss.develop.com/archives/wa.exe?A2=ind0007&L=DOTNET&P=R24819>) on the DOTNET list, started by Chris Sells, about the implications of non-deterministic destruction of objects in C#. In October 2000, Microsoft's Brian Harry posted a [lengthy analysis](<http://discuss.develop.com/archives/wa.exe?A2=ind0010A&L=DOTNET&P=R28572>) of the problem. Chris Sells' [response](<http://discuss.develop.com/archives/wa.exe?A2=ind0010C&L=DOTNET&P=R983>) to Brian's posting is here.

**5.3 Why doesn't the .NET runtime offer deterministic destruction?**

Because of the garbage collection algorithm. The .NET garbage collector works by periodically running through a list of all the objects that are currently being referenced by an application. All the objects that it doesn't find during this search are ready to be destroyed and the memory reclaimed. The implication of this algorithm is that the runtime doesn't get notified immediately when the final reference on an object goes away - it only finds out during the next 'sweep' of the heap.

Futhermore, this type of algorithm works best by performing the garbage collection sweep as rarely as possible. Normally heap exhaustion is the trigger for a collection sweep.

**5.4 Is the lack of deterministic destruction in .NET a problem?**

It's certainly an issue that affects component design. If you have objects that maintain expensive or scarce resources (e.g. database locks), you need to provide some way to tell the object to release the resource when it is done. Microsoft recommend that you provide a method called Dispose() for this purpose. However, this causes problems for distributed objects - in a distributed system who calls the Dispose() method? Some form of reference-counting or ownership-management mechanism is needed to handle distributed objects - unfortunately the runtime offers no help with this.

**5.5 Should I implement Finalize on my class? Should I implement IDisposable?**

This issue is a little more complex than it first appears. There are really two categories of class that require deterministic destruction - the first category manipulate unmanaged types directly, whereas the second category manipulate _managed_ types that require deterministic destruction. An example of the first category is a class with an IntPtr member representing an OS file handle. An example of the second category is a class with a System.IO.FileStream member.

For the first category, it makes sense to implement IDisposable _and_ override Finalize. This allows the object user to 'do the right thing' by calling Dispose, but also provides a fallback of freeing the unmanaged resource in the Finalizer, should the calling code fail in its duty. However this logic does not apply to the second category of class, with only managed resources. In this case implementing Finalize is pointless, as managed member objects cannot be accessed in the Finalizer. This is because there is no guarantee about the ordering of Finalizer execution. So only the Dispose method should be implemented. (If you think about it, it doesn't really make sense to call Dispose on member objects from a Finalizer anyway, as the member object's Finalizer will do the required cleanup.)

For classes that need to implement IDisposable _and_ override Finalize, see Microsoft's [documented pattern](<http://msdn.microsoft.com/library/default.asp?url=/library/en-us/cpgenref/html/cpconFinalizeDispose.asp>). 

Note that some developers argue that implementing a Finalizer is always a bad idea, as it hides a bug in your code (i.e. the lack of a Dispose call). A less radical approach is to implement Finalize but include a Debug.Assert at the start, thus signalling the problem in developer builds but allowing the cleanup to occur in release builds.

**5.6 Do I have any control over the garbage collection algorithm?**

A little. For example the System.GC class exposes a Collect method, which forces the garbage collector to collect all unreferenced objects immediately.

Also there is a **gcConcurrent** setting that can be specified via the application configuration file. This specifies whether or not the garbage collector performs some of its collection activities on a separate thread. The setting only applies on multi-processor machines, and defaults to true.

**5.7 How can I find out what the garbage collector is doing?**

Lots of interesting statistics are exported from the .NET runtime via the '.NET CLR xxx' performance counters. Use Performance Monitor to view them.

**5.8 What is the lapsed listener problem?**

The lapsed listener problem is one of the primary causes of leaks in .NET applications. It occurs when a subscriber (or 'listener') signs up for a publisher's event, but fails to unsubscribe. The failure to unsubscribe means that the publisher maintains a reference to the subscriber as long as the publisher is alive. For some publishers, this may be the duration of the application.

This situation causes two problems. The obvious problem is the leakage of the subscriber object. The other problem is the performance degredation due to the publisher sending redundant notifications to 'zombie' subscribers. 

There are at least a couple of solutions to the problem. The simplest is to make sure the subscriber is unsubscribed from the publisher, typically by adding an Unsubscribe() method to the subscriber. Another solution, documented [here](<http://www.windojitsu.com/blog/weakevent.html>) by Shawn Van Ness, is to change the publisher to use weak references in its subscriber list.

**5.9 When do I need to use GC.KeepAlive?**

It's very unintuitive, but the runtime can decide that an object is garbage much sooner than you expect. More specifically, an object can become garbage while a method is executing on the object, which is contrary to most developers' expectations. Chris Brumme [explains](<http://blogs.msdn.com/cbrumme/archive/2003/04/19/51365.aspx>) the issue on his blog. I've taken Chris's code and expanded it into a full app that you can play with if you want to prove to yourself that this is a real problem:

using System;

using System.Runtime.InteropServices;

class Win32

{

[DllImport("kernel32.dll")] 

public static extern IntPtr CreateEvent( IntPtr lpEventAttributes, 

bool bManualReset,bool bInitialState, string lpName);

[DllImport("kernel32.dll", SetLastError=true)] 

public static extern bool CloseHandle(IntPtr hObject);

[DllImport("kernel32.dll")] 

public static extern bool SetEvent(IntPtr hEvent);

}

class EventUser

{

public EventUser() 

{ 

hEvent = Win32.CreateEvent( IntPtr.Zero, false, false, null ); 

}

~EventUser() 

{ 

Win32.CloseHandle( hEvent ); 

Console.WriteLine("EventUser finalized");

}

public void UseEvent() 

{ 

UseEventInStatic( this.hEvent ); 

}

static void UseEventInStatic( IntPtr hEvent )

{

//GC.Collect();

bool bSuccess = Win32.SetEvent( hEvent );

Console.WriteLine( "SetEvent " + (bSuccess ? "succeeded" : "FAILED!") );

}

IntPtr hEvent;

}

class App

{

static void Main(string[] args)

{

EventUser eventUser = new EventUser();

eventUser.UseEvent();

}

}

If you run this code, it'll probably work fine, and you'll get the following output:

SetEvent succeeded

EventDemo finalized

However, if you uncomment the GC.Collect() call in the UseEventInStatic() method, you'll get this output:

EventDemo finalized

SetEvent FAILED!

(Note that you need to use a release build to reproduce this problem.)

So what's happening here? Well, at the point where UseEvent() calls UseEventInStatic(), a copy is taken of the hEvent field, and there are no further references to the EventUser object anywhere in the code. So as far as the runtime is concerned, the EventUser object is garbage and can be collected. Normally of course the collection won't happen immediately, so you'll get away with it, but sooner or later a collection will occur at the wrong time, and your app will fail. 

A solution to this problem is to add a call to GC.KeepAlive(this) to the end of the UseEvent method, as [Chris explains](<http://blogs.msdn.com/cbrumme/archive/2003/04/19/51365.aspx>).

**6\. Serialization**

**6.1 What is serialization?**

Serialization is the process of converting an object into a stream of bytes. Deserialization is the opposite process, i.e. creating an object from a stream of bytes. Serialization/Deserialization is mostly used to transport objects (e.g. during remoting), or to persist objects (e.g. to a file or database).

**6.2 Does the .NET Framework have in-built support for serialization?**

There are two separate mechanisms provided by the .NET class library - XmlSerializer and SoapFormatter/BinaryFormatter. Microsoft uses XmlSerializer for Web Services, and SoapFormatter/BinaryFormatter for remoting. Both are available for use in your own code.

**6.3 I want to serialize instances of my class. Should I use XmlSerializer, SoapFormatter or BinaryFormatter?**

It depends. XmlSerializer has severe limitations such as the requirement that the target class has a parameterless constructor, and only public read/write properties and fields can be serialized. However, on the plus side, XmlSerializer has good support for customising the XML document that is produced or consumed. XmlSerializer's features mean that it is most suitable for cross-platform work, or for constructing objects from existing XML documents.

SoapFormatter and BinaryFormatter have fewer limitations than XmlSerializer. They can serialize private fields, for example. However they both require that the target class be marked with the [Serializable] attribute, so like XmlSerializer the class needs to be written with serialization in mind. Also there are some quirks to watch out for - for example on deserialization the constructor of the new object is not invoked.

The choice between SoapFormatter and BinaryFormatter depends on the application. BinaryFormatter makes sense where both serialization and deserialization will be performed on the .NET platform and where performance is important. SoapFormatter generally makes more sense in all other cases, for ease of debugging if nothing else.

**6.4 Can I customise the serialization process?**

Yes. XmlSerializer supports a range of attributes that can be used to configure serialization for a particular class. For example, a field or property can be marked with the [XmlIgnore] attribute to exclude it from serialization. Another example is the [XmlElement] attribute, which can be used to specify the XML element name to be used for a particular property or field.

Serialization via SoapFormatter/BinaryFormatter can also be controlled to some extent by attributes. For example, the [NonSerialized] attribute is the equivalent of XmlSerializer's [XmlIgnore] attribute. Ultimate control of the serialization process can be acheived by implementing the the ISerializable interface on the class whose instances are to be serialized.

**6.5 Why is XmlSerializer so slow?**

There is a once-per-process-per-type overhead with XmlSerializer. So the first time you serialize or deserialize an object of a given type in an application, there is a significant delay. This normally doesn't matter, but it may mean, for example, that XmlSerializer is a poor choice for loading configuration settings during startup of a GUI application.

**6.6 Why do I get errors when I try to serialize a Hashtable?**

XmlSerializer will refuse to serialize instances of any class that implements IDictionary, e.g. Hashtable. SoapFormatter and BinaryFormatter do not have this restriction.

**6.7 XmlSerializer is throwing a generic "There was an error reflecting MyClass" error. How do I find out what the problem is?**

Look at the InnerException property of the exception that is thrown to get a more specific error message.

**6.8 Why am I getting an InvalidOperationException when I serialize an ArrayList?**

XmlSerializer needs to know in advance what type of objects it will find in an ArrayList. To specify the type, use the XmlArrayItem attibute like this:

public class Person

{

public string Name;

public int Age;

}

public class Population

{

[XmlArrayItem(typeof(Person))] public ArrayList People;

}

**7\. Attributes**

**7.1 What are attributes?**

There are at least two types of .NET attribute. The first type I will refer to as a _metadata_ attribute - it allows some data to be attached to a class or method. This data becomes part of the metadata for the class, and (like other class metadata) can be accessed via reflection. An example of a metadata attribute is [serializable], which can be attached to a class and means that instances of the class can be serialized.

[serializable] public class CTest {}

The other type of attribute is a _context_ attribute. Context attributes use a similar syntax to metadata attributes but they are fundamentally different. Context attributes provide an interception mechanism whereby instance activation and method calls can be pre- and/or post-processed. If you have encountered Keith Brown's universal delegator you'll be familiar with this idea. 

**7.2 Can I create my own metadata attributes?**

Yes. Simply derive a class from System.Attribute and mark it with the AttributeUsage attribute. For example:

[AttributeUsage(AttributeTargets.Class)]

public class InspiredByAttribute : System.Attribute 

{ 

public string InspiredBy;

public InspiredByAttribute( string inspiredBy )

{

InspiredBy = inspiredBy;

}

}

[InspiredBy("Andy Mc's brilliant .NET FAQ")]

class CTest

{

}

class CApp

{

public static void Main()

{ 

object[] atts = typeof(CTest).GetCustomAttributes(true);

foreach( object att in atts )

if( att is InspiredByAttribute )

Console.WriteLine( "Class CTest was inspired by {0}", ((InspiredByAttribute)att).InspiredBy );

}

}

**7.3 Can I create my own context attibutes?**

Yes. Take a look at Peter Drayton's [Tracehook.NET](<http://www.razorsoft.net/TraceHook.htm>).

**8\. Code Access Security**

**8.1 What is Code Access Security (CAS)?**

CAS is the part of the .NET security model that determines whether or not code is allowed to run, and what resources it can use when it is running. For example, it is CAS that will prevent a .NET web applet from formatting your hard disk.

**8.2 How does CAS work?**

The CAS security policy revolves around two key concepts - code groups and permissions. Each .NET assembly is a member of a particular code group, and each code group is granted the permissions specified in a **named permission set**.

For example, using the default security policy, a control downloaded from a web site belongs to the 'Zone - Internet' code group, which adheres to the permissions defined by the 'Internet' named permission set. (Naturally the 'Internet' named permission set represents a very restrictive range of permissions.) 

**8.3 Who defines the CAS code groups?**

Microsoft defines some default ones, but you can modify these and even create your own. To see the code groups defined on your system, run 'caspol -lg' from the command-line. On my system it looks like this:

Level = Machine

Code Groups:

1\. All code: Nothing

1.1. Zone - MyComputer: FullTrust

1.1.1. Honor SkipVerification requests: SkipVerification

1.2. Zone - Intranet: LocalIntranet

1.3. Zone - Internet: Internet

1.4. Zone - Untrusted: Nothing

1.5. Zone - Trusted: Internet

1.6. StrongName -

0024000004800000940000000602000000240000525341310004000003

000000CFCB3291AA715FE99D40D49040336F9056D7886FED46775BC7BB5430BA4444FEF8348EBD06

F962F39776AE4DC3B7B04A7FE6F49F25F740423EBF2C0B89698D8D08AC48D69CED0FC8F83B465E08

07AC11EC1DCC7D054E807A43336DDE408A5393A48556123272CEEEE72F1660B71927D38561AABF5C

AC1DF1734633C602F8F2D5: Everything

Note the hierarchy of code groups - the top of the hierarchy is the most general ('All code'), which is then sub-divided into several groups, each of which in turn can be sub-divided. Also note that (somewhat counter-intuitively) a sub-group can be associated with a more permissive permission set than its parent.

**8.4 How do I define my own code group?**

Use caspol. For example, suppose you trust code from www.mydomain.com and you want it have full access to your system, but you want to keep the default restrictions for all other internet sites. To achieve this, you would add a new code group as a sub-group of the 'Zone - Internet' group, like this:

caspol -ag 1.3 -site www.mydomain.com FullTrust 

Now if you run caspol -lg you will see that the new group has been added as group 1.3.1:

...

1.3. Zone - Internet: Internet

1.3.1. Site - www.mydomain.com: FullTrust

...

Note that the numeric label (1.3.1) is just a caspol invention to make the code groups easy to manipulate from the command-line. The underlying runtime never sees it. 

**8.5 How do I change the permission set for a code group?**

Use caspol. If you are the machine administrator, you can operate at the 'machine' level - which means not only that the changes you make become the default for the machine, but also that users cannot change the permissions to be more permissive. If you are a normal (non-admin) user you can still modify the permissions, but only to make them more restrictive. For example, to allow intranet code to do what it likes you might do this:

caspol -cg 1.2 FullTrust

Note that because this is more permissive than the default policy (on a standard system), you should only do this at the machine level - doing it at the user level will have no effect.

**8.6 Can I create my own permission set?**

Yes. Use caspol -ap, specifying an XML file containing the permissions in the permission set. To save you some time, [here](<http://www.andymcm.com/samplepermset.xml>) is a sample file corresponding to the 'Everything' permission set - just edit to suit your needs. When you have edited the sample, add it to the range of available permission sets like this:

caspol -ap samplepermset.xml

Then, to apply the permission set to a code group, do something like this:

caspol -cg 1.3 SamplePermSet

(By default, 1.3 is the 'Internet' code group)

**8.7 I'm having some trouble with CAS. How can I troubleshoot the problem?**

Caspol has a couple of options that might help. First, you can ask caspol to tell you what code group an assembly belongs to, using caspol -rsg. Similarly, you can ask what permissions are being applied to a particular assembly using caspol -rsp.

**8.8 I can't be bothered with CAS. Can I turn it off?**

Yes, as long as you are an administrator. Just run:

caspol -s off

**9\. Intermediate Language (IL)**

**9.1 Can I look at the IL for an assembly?**

Yes. MS supply a tool called Ildasm that can be used to view the metadata and IL for an assembly. 

**9.2 Can source code be reverse-engineered from IL?**

Yes, it is often relatively straightforward to regenerate high-level source from IL. Lutz Roeder's [Reflector](<http://www.aisto.com/roeder/dotnet/>) does a very good job of turning IL into C# or VB.NET.

**9.3 How can I stop my code being reverse-engineered from IL?**

You can buy an IL obfuscation tool. These tools work by 'optimising' the IL in such a way that reverse-engineering becomes much more difficult.

Of course if you are writing web services then reverse-engineering is not a problem as clients do not have access to your IL.

**9.4 Can I write IL programs directly?**

Yes. [Peter Drayton](<http://www.razorsoft.net/>) posted this simple example to the DOTNET mailing list:

.assembly MyAssembly {}

.class MyApp {

.method static void Main() {

.entrypoint

ldstr "Hello, IL!"

call void System.Console::WriteLine(class System.Object)

ret

}

}

Just put this into a file called hello.il, and then run ilasm hello.il. An exe assembly will be generated.

**9.5 Can I do things in IL that I can't do in C#?**

Yes. A couple of simple examples are that you can throw exceptions that are not derived from System.Exception, and you can have non-zero-based arrays.

**10\. Implications for COM**

**10.1 Does .NET replace COM?**

This subject causes a lot of controversy, as you'll see if you read the mailing list archives. Take a look at the following two threads:

[http://discuss.develop.com/archives/wa.exe?A2=ind0007&L=DOTNET&D=0&P=68241](<http://discuss.develop.com/archives/wa.exe?A2=ind0007&L=DOTNET&D=0&P=68241>) [http://discuss.develop.com/archives/wa.exe?A2=ind0007&L=DOTNET&P=R60761](<http://discuss.develop.com/archives/wa.exe?A2=ind0007&L=DOTNET&P=R60761>)

The bottom line is that .NET has its own mechanisms for type interaction, and they don't use COM. No IUnknown, no IDL, no typelibs, no registry-based activation. This is mostly good, as a lot of COM was ugly. Generally speaking, .NET allows you to package and use components in a similar way to COM, but makes the whole thing a bit easier.

**10.2 Is DCOM dead?**

Pretty much, for .NET developers. The .NET Framework has a new remoting model which is not based on DCOM. DCOM was pretty much dead anyway, once firewalls became widespread and Microsoft got [SOAP](<http://www.w3.org/TR/soap/>) fever. Of course DCOM will still be used in interop scenarios. 

**10.3 Is COM+ dead?**

Not immediately. The approach for .NET 1.0 was to provide access to the existing COM+ services (through an interop layer) rather than replace the services with native .NET ones. Various tools and attributes were provided to make this as painless as possible. Over time it is expected that interop will become more seamless - this may mean that some services become a core part of the CLR, and/or it may mean that some services will be rewritten as managed code which runs on top of the CLR.

For more on this topic, search for postings by Joe Long in the archives - Joe is the MS group manager for COM+. Start with this message:

[http://discuss.develop.com/archives/wa.exe?A2=ind0007&L=DOTNET&P=R68370](<http://discuss.develop.com/archives/wa.exe?A2=ind0007&L=DOTNET&P=R68370>)

**10.4 Can I use COM components from .NET programs?**

Yes. COM components are accessed from the .NET runtime via a Runtime Callable Wrapper (RCW). This wrapper turns the COM interfaces exposed by the COM component into .NET-compatible interfaces. For oleautomation interfaces, the RCW can be generated automatically from a type library. For non-oleautomation interfaces, it may be necessary to develop a custom RCW which manually maps the types exposed by the COM interface to .NET-compatible types.

Here's a simple example for those familiar with ATL. First, create an ATL component which implements the following IDL:

import "oaidl.idl"; 

import "ocidl.idl";

[

object,

uuid(EA013F93-487A-4403-86EC-FD9FEE5E6206),

helpstring("ICppName Interface"), 

pointer_default(unique), 

oleautomation

] 

interface ICppName : IUnknown

{ 

[helpstring("method SetName")] HRESULT SetName([in] BSTR name); 

[helpstring("method GetName")] HRESULT GetName([out,retval] BSTR *pName ); 

}; 

[ 

uuid(F5E4C61D-D93A-4295-A4B4-2453D4A4484D), 

version(1.0),

helpstring("cppcomserver 1.0 Type Library")

] 

library CPPCOMSERVERLib 

{

importlib("stdole32.tlb");

importlib("stdole2.tlb"); 

[

uuid(600CE6D9-5ED7-4B4D-BB49-E8D5D5096F70), 

helpstring("CppName Class") 

]

coclass CppName

{ 

[default] interface ICppName; 

};

};

When you've built the component, you should get a typelibrary. Run the TLBIMP utility on the typelibary, like this:

tlbimp cppcomserver.tlb

If successful, you will get a message like this:

Typelib imported successfully to CPPCOMSERVERLib.dll

You now need a .NET client - let's use C#. Create a .cs file containing the following code:

using System;

using CPPCOMSERVERLib; 

public class MainApp 

{ 

static public void Main() 

{ 

CppName cppname = new CppName();

cppname.SetName( "bob" ); 

Console.WriteLine( "Name is " + cppname.GetName() ); 

}

}

Compile the C# code like this:

csc /r:cppcomserverlib.dll csharpcomclient.cs

Note that the compiler is being told to reference the DLL we previously generated from the typelibrary using TLBIMP. You should now be able to run csharpcomclient.exe, and get the following output on the console:

Name is bob

**10.5 Can I use .NET components from COM programs?**

Yes. .NET components are accessed from COM via a COM Callable Wrapper (CCW). This is similar to a RCW (see previous question), but works in the opposite direction. Again, if the wrapper cannot be automatically generated by the .NET development tools, or if the automatic behaviour is not desirable, a custom CCW can be developed. Also, for COM to 'see' the .NET component, the .NET component must be registered in the registry.

Here's a simple example. Create a C# file called testcomserver.cs and put the following in it:

using System; 

using System.Runtime.InteropServices;

namespace AndyMc 

{ 

[ClassInterface(ClassInterfaceType.AutoDual)]

public class CSharpCOMServer

{ 

public CSharpCOMServer() {} 

public void SetName( string name ) { m_name = name; } 

public string GetName() { return m_name; } 

private string m_name; 

} 

}

Then compile the .cs file as follows:

csc /target:library testcomserver.cs

You should get a dll, which you register like this:

regasm testcomserver.dll /tlb:testcomserver.tlb /codebase

Now you need to create a client to test your .NET COM component. VBScript will do - put the following in a file called comclient.vbs:

Dim dotNetObj 

Set dotNetObj = CreateObject("AndyMc.CSharpCOMServer") 

dotNetObj.SetName ("bob") 

MsgBox "Name is " & dotNetObj.GetName()

and run the script like this:

wscript comclient.vbs

And hey presto you should get a message box displayed with the text "Name is bob".

An alternative to the approach above it to use the [dm.net moniker](<http://staff.develop.com/jasonw/clr/readme.htm>) developed by Jason Whittington and Don Box.

**10.6 Is ATL redundant in the .NET world?**

Yes. ATL will continue to be valuable for writing COM components for some time, but it has no place in the .NET world.

**11\. Threads**

**11.1 How do I spawn a thread?**

Create an instance of a System.Threading.Thread object, passing it an instance of a ThreadStart delegate that will be executed on the new thread. For example:

class MyThread

{

public MyThread( string initData )

{

m_data = initData;

m_thread = new Thread( new ThreadStart(ThreadMain) ); 

m_thread.Start(); 

}

// ThreadMain() is executed on the new thread.

private void ThreadMain()

{ 

Console.WriteLine( m_data );

}

public void WaitUntilFinished()

{

m_thread.Join();

} 

private Thread m_thread;

private string m_data;

}

In this case creating an instance of the MyThread class is sufficient to spawn the thread and execute the MyThread.ThreadMain() method:

MyThread t = new MyThread( "Hello, world." );

t.WaitUntilFinished();

**11.2 How do I stop a thread?**

There are several options. First, you can use your own communication mechanism to tell the ThreadStart method to finish. Alternatively the Thread class has in-built support for instructing the thread to stop. The two principle methods are Thread.Interrupt() and Thread.Abort(). The former will cause a ThreadInterruptedException to be thrown on the thread when it next goes into a WaitJoinSleep state. In other words, Thread.Interrupt is a polite way of asking the thread to stop when it is no longer doing any useful work. In contrast, Thread.Abort() throws a ThreadAbortException regardless of what the thread is doing. Furthermore, the ThreadAbortException cannot normally be caught (though the ThreadStart's finally method will be executed). Thread.Abort() is a heavy-handed mechanism which should not normally be required.

**11.3 How do I use the thread pool?**

By passing an instance of a WaitCallback delegate to the ThreadPool.QueueUserWorkItem() method

class CApp

{

static void Main()

{

string s = "Hello, World";

ThreadPool.QueueUserWorkItem( new WaitCallback( DoWork ), s );

Thread.Sleep( 1000 ); // Give time for work item to be executed

}

// DoWork is executed on a thread from the thread pool.

static void DoWork( object state )

{

Console.WriteLine( state );

}

}

**11.4 How do I know when my thread pool work item has completed?**

There is no way to query the thread pool for this information. You must put code into the WaitCallback method to signal that it has completed. Events are useful for this.

**11.5 How do I prevent concurrent access to my data?**

Each object has a concurrency lock (critical section) associated with it. The System.Threading.Monitor.Enter/Exit methods are used to acquire and release this lock. For example, instances of the following class only allow one thread at a time to enter method f():

class C

{

public void f()

{

try

{

Monitor.Enter(this);

...

}

finally

{

Monitor.Exit(this);

}

}

}

C# has a 'lock' keyword which provides a convenient shorthand for the code above:

class C

{

public void f()

{

lock(this)

{

...

}

}

}

Note that calling Monitor.Enter(myObject) does NOT mean that all access to myObject is serialized. It means that the synchronisation lock associated with myObject has been acquired, and no other thread can acquire that lock until Monitor.Exit(o) is called. In other words, this class is functionally equivalent to the classes above:

class C

{

public void f()

{

lock( m_object )

{

...

}

}

private m_object = new object();

}

Actually, it could be argued that this version of the code is superior, as the lock is totally encapsulated within the class, and not accessible to the user of the object. 

**11.6 Should I use ReaderWriterLock instead of Monitor.Enter/Exit?**

Maybe, but be careful. ReaderWriterLock is used to allow multiple threads to read from a data source, while still granting exclusive access to a single writer thread. This makes sense for data access that is mostly read-only, but there are some caveats. First, ReaderWriterLock is relatively poor performing compared to Monitor.Enter/Exit, which offsets some of the benefits. Second, you need to be very sure that the data structures you are accessing fully support multithreaded read access. Finally, there is apparently a bug in the v1.1 ReaderWriterLock that can cause starvation for writers when there are a large number of readers. 

Ian Griffiths has some interesting discussion on ReaderWriterLock [here](<http://www.interact-sw.co.uk/iangblog/2004/04/26/rwlockvsmonitor>) and [here](<http://www.interact-sw.co.uk/iangblog/2004/05/12/rwlock>).

**12\. Tracing**

**12.1 Is there built-in support for tracing/logging?**

Yes, in the System.Diagnostics namespace. There are two main classes that deal with tracing - Debug and Trace. They both work in a similar way - the difference is that tracing from the Debug class only works in builds that have the DEBUG symbol defined, whereas tracing from the Trace class only works in builds that have the TRACE symbol defined. Typically this means that you should use System.Diagnostics.Trace.WriteLine for tracing that you want to work in debug and release builds, and System.Diagnostics.Debug.WriteLine for tracing that you want to work only in debug builds.

**12.2 Can I redirect tracing to a file?**

Yes. The Debug and Trace classes both have a Listeners property, which is a collection of sinks that receive the tracing that you send via Debug.WriteLine and Trace.WriteLine respectively. By default the Listeners collection contains a single sink, which is an instance of the DefaultTraceListener class. This sends output to the Win32 OutputDebugString() function and also the System.Diagnostics.Debugger.Log() method. This is useful when debugging, but if you're trying to trace a problem at a customer site, redirecting the output to a file is more appropriate. Fortunately, the TextWriterTraceListener class is provided for this purpose.

Here's how to use the TextWriterTraceListener class to redirect Trace output to a file:

Trace.Listeners.Clear();

FileStream fs = new FileStream( @"c:\log.txt", FileMode.Create, FileAccess.Write );

Trace.Listeners.Add( new TextWriterTraceListener( fs ) );

Trace.WriteLine( @"This will be writen to c:\log.txt!" );

Trace.Flush();

Note the use of Trace.Listeners.Clear() to remove the default listener. If you don't do this, the output will go to the file _and_ OutputDebugString(). Typically this is not what you want, because OutputDebugString() imposes a big performance hit.

**12.3 Can I customise the trace output?**

Yes. You can write your own TraceListener-derived class, and direct all output through it. Here's a simple example, which derives from TextWriterTraceListener (and therefore has in-built support for writing to files, as shown above) and adds timing information and the thread ID for each trace line:

class MyListener : TextWriterTraceListener

{

public MyListener( Stream s ) : base(s)

{

}

public override void WriteLine( string s )

{

Writer.WriteLine( "{0:D8} [{1:D4}] {2}", 

Environment.TickCount - m_startTickCount, 

AppDomain.GetCurrentThreadId(),

s );

}

protected int m_startTickCount = Environment.TickCount;

}

(Note that this implementation is not complete - the TraceListener.Write method is not overridden for example.)

The beauty of this approach is that when an instance of MyListener is added to the Trace.Listeners collection, all calls to Trace.WriteLine() go through MyListener, including calls made by referenced assemblies that know nothing about the MyListener class.

**12.4 Are there any third party logging components available?**

[Log4net](<http://logging.apache.org/log4net/>) is a port of the established log4j Java logging component.

**13\. Miscellaneous**

**13.1 How does .NET remoting work?**

.NET remoting involves sending messages along channels. Two of the standard channels are HTTP and TCP. TCP is intended for LANs only - HTTP can be used for LANs or WANs (internet).

Support is provided for multiple message serializarion formats. Examples are SOAP (XML-based) and binary. By default, the HTTP channel uses SOAP (via the .NET runtime Serialization SOAP Formatter), and the TCP channel uses binary (via the .NET runtime Serialization Binary Formatter). But either channel can use either serialization format.

There are a number of styles of remote access:

  * **SingleCall**. Each incoming request from a client is serviced by a new object. The object is thrown away when the request has finished. 

  * **Singleton**. All incoming requests from clients are processed by a single server object. 

  * **Client-activated object**. This is the old stateful (D)COM model whereby the client receives a reference to the remote object and holds that reference (thus keeping the remote object alive) until it is finished with it.




Distributed garbage collection of objects is managed by a system called 'leased based lifetime'. Each object has a lease time, and when that time expires the object is disconnected from the .NET runtime remoting infrastructure. Objects have a default renew time - the lease is renewed when a successful call is made from the client to the object. The client can also explicitly renew the lease.

If you're interested in using XML-RPC as an alternative to SOAP, take a look at Charles Cook's [XML-RPC.Net](<http://www.cookcomputing.com/xmlrpc/xmlrpc.shtml>).

**13.2 How can I get at the Win32 API from a .NET program?**

Use P/Invoke. This uses similar technology to COM Interop, but is used to access static DLL entry points instead of COM objects. Here is an example of C# calling the Win32 MessageBox function:

using System; 

using System.Runtime.InteropServices; 

class MainApp 

{ 

[DllImport("user32.dll", EntryPoint="MessageBox", SetLastError=true, CharSet=CharSet.Auto)] 

public static extern int MessageBox(int hWnd, String strMessage, String strCaption, uint uiType);

public static void Main() 

{

MessageBox( 0, "Hello, this is PInvoke in operation!", ".NET", 0 ); 

}

} 

Pinvoke.net is a great resource for off-the-shelf P/Invoke signatures.

**13.3 How do I write to the application configuration file at runtime?**

You don't. See [http://www.interact-sw.co.uk/iangblog/2004/11/25/savingconfig](<http://www.interact-sw.co.uk/iangblog/2004/11/25/savingconfig>).

**13.4 What is the difference between an event and a delegate?**

An event is just a wrapper for a multicast delegate. Adding a public event to a class is almost the same as adding a public multicast delegate field. In both cases, subscriber objects can register for notifications, and in both cases the publisher object can send notifications to the subscribers. However, a public multicast delegate has the undesirable property that external objects can _invoke_ the delegate, something we'd normally want to restrict to the publisher. Hence events - an event adds public methods to the containing class to add and remove receivers, but does not make the invocation mechanism public. 

See this [post](<http://blog.monstuff.com/archives/000040.html>) by Julien Couvreur for more discussion.

**13.5 What size is a .NET object?**

Each instance of a reference type has two fields maintained by the runtime - a method table pointer and a sync block. These are 4 bytes each on a 32-bit system, making a total of 8 bytes per object overhead. Obviously the instance data for the type must be added to this to get the overall size of the object. So, for example, instances of the following class are 12 bytes each:

class MyInt

{

...

private int x;

}

However, note that with the current implementation of the CLR there seems to be a minimum object size of 12 bytes, even for classes with no data (e.g. System.Object).

Values types have no equivalent overhead.

**14\. .NET 2.0**

**14.1 What are the new features of .NET 2.0?**

Generics, anonymous methods, partial classes, iterators, property visibility (separate visibility for get and set) and static classes. See [http://msdn.microsoft.com/msdnmag/issues/04/05/C20/default.aspx](<http://msdn.microsoft.com/msdnmag/issues/04/05/C20/default.aspx>) for more information about these features.

**14.2 What are the new 2.0 features useful for?**

Generics are useful for writing efficient type-independent code, particularly where the types might include value types. The obvious application is container classes, and the .NET 2.0 class library includes a suite of generic container classes in the System.Collections.Generic namespace. Here's a simple example of a generic container class being used:

List<int> myList = new List<int>();

myList.Add( 10 );

Anonymous methods reduce the amount of code you have to write when using delegates, and are therefore especially useful for GUI programming. Here's an example

AppDomain.CurrentDomain.ProcessExit += delegate { Console.WriteLine("Process ending ..."); };

Partial classes is a useful feature for separating machine-generated code from hand-written code in the same class, and will therefore be heavily used by development tools such as Visual Studio. 

Iterators reduce the amount of code you need to write to implement IEnumerable/IEnumerator. Here's some sample code:

static void Main()

{

RandomEnumerator re = new RandomEnumerator( 5 );

foreach( double r in re )

Console.WriteLine( r );

Console.Read();

}

class RandomEnumerator : IEnumerable<double>

{

public RandomEnumerator(int size) { m_size = size; }

public IEnumerator<double> GetEnumerator()

{

Random rand = new Random();

for( int i=0; i < m_size; i++ )

yield return rand.NextDouble();

}

int m_size = 0;

}

The use of 'yield return' is rather strange at first sight. It effectively synthethises an implementation of IEnumerator, something we had to do manually in .NET 1.x.

**14.3 What's the problem with .NET generics?**

.NET generics work great for container classes. But what about other uses? Well, it turns out that .NET generics have a major limitation - they require the type parameter to be _constrained_. For example, you cannot do this:

static class Disposer<T>

{

public static void Dispose(T obj) { obj.Dispose(); }

}

The C# compiler will refuse to compile this code, as the type T has not been constrained, and therefore only supports the methods of System.Object. Dispose is not a method on System.Object, so the compilation fails. To fix this code, we need to add a **where** clause, to reassure the compiler that our type T does indeed have a Dispose method

static class Disposer<T> where T : IDisposable

{

public static void Dispose(T obj) { obj.Dispose(); }

}

The problem is that the requirement for explicit contraints is very limiting. We can use constraints to say that T implements a particular _interface_ , but we can't dilute that to simply say that T implements a particular _method_. Contrast this with C++ templates (for example), where no constraint at all is required - it is _assumed_ (and verified at compile time) that if the code invokes the Dispose() method on a type, then the type will support the method. 

In fact, after writing generic code with interface constraints, we quickly see that we haven't gained much over non-generic interface-based programming. For example, we can easily rewrite the Disposer class without generics:

static class Disposer

{

public static void Dispose( IDisposable obj ) { obj.Dispose(); }

}

For more on this topic, start by reading the following articles:

Bruce Eckel: [http://www.mindview.net/WebLog/log-0050](<http://www.mindview.net/WebLog/log-0050>) Ian Griffiths: [http://www.interact-sw.co.uk/iangblog/2004/03/14/generics](<http://www.interact-sw.co.uk/iangblog/2004/03/14/generics>) Charles Cook: [http://www.cookcomputing.com/blog/archives/000425.html](<http://www.cookcomputing.com/blog/archives/000425.html>)

**14.4 What's new in the .NET 2.0 class library?**

Here is a selection of new features in the .NET 2.0 class library:

  * Generic collections in the System.Collections.Generic namespace. 

  * The **System.Nullable <T>** type. (Note that C# has special syntax for this type, e.g. int? is equivalent to Nullable<int>) 

  * The **GZipStream** and **DeflateStream** classes in the System.IO.Compression namespace. 

  * The **Semaphore** class in the System.Threading namespace. 

  * Wrappers for DPAPI in the form of the **ProtectedData** and **ProtectedMemory** classes in the System.Security.Cryptography namespace. 

  * The IPC remoting channel in the System.Runtime.Remoting.Channels.Ipc namespace, for optimised intra-machine communication.




and many, many more. See [http://msdn2.microsoft.com/en-us/library/t357fb32(en-US,VS.80).aspx](<http://msdn2.microsoft.com/en-us/library/t357fb32\(en-US,VS.80\).aspx>) for a comprehensive list of changes.
