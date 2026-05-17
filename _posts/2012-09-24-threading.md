---
title: "Threading"
date: 2012-09-24 10:02:10 
author: shajeebhameed
layout: page
slug: threading
status: publish
---

**Introduction** Asynchronous processing and background processing was always a must for serious programs serving complex user needs. The Windows NT platform offers a great way to accomplish this, but the implementation was sometimes tedious and always labor intensive. It is the reason why I first studied multithreading options offered by the .NET framework. This article shows three different ways of creating (and using) threads, without communication and synchronization between them. Before you start writing multithreaded programs, bear in mind some guidelines (from MSDN � Threading design guidelines):

  * Avoid providing static methods that mutate static state 
  * Design for server environment 
  * Instances do not need to be thread safe 
  * Static states must be thread safe

**Simple threading** Let�s first try the simplest way to create a new thread. The starting point for such a thread is void method with no parameters. Thread creation is done in two steps: 

  1. Create a delegate object, initialized with our method 
  2. Use this object as an initialization parameter when creating a new Thread object.

When our Thread object is created, call the Start method and background processing will commence.  Collapse[ Copy Code](<http://www.codeproject.com/KB/dotnet/multithread.aspx>) publicclass MainClass{ publicvoid threadMethod(){ ... } publicstaticvoid Main(){ ... ThreadStart entry = new ThreadStart(threadMethod ) ; Thread thread1 = new Thread( entry ) ; thread1.Start() ; ... } } This sample hides the fact that using C# we do not have a global function, and usually you will start the thread on static class method.  Collapse[ Copy Code](<http://www.codeproject.com/KB/dotnet/multithread.aspx>) publicclass Tester{ publicstaticvoid Test(){ ... } } publicclass MainClass{ publicstaticvoid Main() ... ThreadStart entry = new ThreadStart( Tester.Test ) ; Thread thread1 = new Thread( entry ) ; thread1.Start() ; ... } } Now we came to the first beautiful part of the .NET framework. We can start threads on class instances and create real living objects.  Collapse[ Copy Code](<http://www.codeproject.com/KB/dotnet/multithread.aspx>) publicclass Tester{ publicvoid Test(){ ... } } publicclass MainClass{ publicstaticvoid Main() ... Tester testObject = new Tester() ; ThreadStart entry = new ThreadStart( testObject.Test ) ; Thread thread1 = new Thread( entry ) ; thread1.Start() ; ... } } **Timer threads** A common use of threads is for all kinds of periodical updates. Under Win32 we have two different ways: window timers and time limited waiting for events. .NET offers three different ways: 

  * Windows timers with the System.WinForms.Timer class 
  * Periodical delegate calling with System.Threading.Timer class (works on W2K only) 
  * Exact timing with the System.Timers.Timer class

For inexact timing we use the window timer. Events raised from the window timer go through the message pump (together with all mouse events and UI update messages) so they are never exact. The simplest way for creating a WinForms.Timer is by adding a Timer control onto a form and creating an event handler using the control's properties. We use the Interval property for setting the number of milliseconds between timer ticks and the Start method to start ticking and Stop to stop ticking. Be careful with stopping, because stopped timers are disabled and are subject to garbage collection. That means that stopped timers can not be started again. The System.Threading.Timer class is a new waiting thread in the thread pool that periodically calls supplied delegates. Currently it works on Windows 2000 only. Documentation for this class is not finished yet (beta 1). To use it you must perform several steps: 

  1. Create state object which will carry information to the delegate 
  2. Create TimerCallback delegate with a method to be called. You can use static or instance methods. 
  3. Create a Timer object with time to wait before first call and periods between successive calls (as names of this two parameters suggests, first parameter should be lifetime of this timer object, but it just don�t work that way) 
  4. Change the Timer object settings with the Change method (same remark on parameters apply) 
  5. Kill the Timer object with the Dispose method

If we put this in code we get:  Collapse[ Copy Code](<http://www.codeproject.com/KB/dotnet/multithread.aspx>) using System; using System.Threading;   _// class for storing current state_ publicclass StateObj{ ... }   _// class that will work on timer request_ publicclass TimerClass{ publicvoid TimerKick( object state ){ StateObj param = (StateObj)state ; _// do some work with param_ } }   _// usage block_ { ... _// prepare state object_ StateObj state = new StateObj() ; _// prepare testing object_ _�_ _not necessary when cb is static_ TimerClass testObj = new TimerClass() ; _// prepare callback delegate_ _�_ _careful on static methods_ TimerCallback tcb = new TimerCallback( obj.TimerKick ) ; _// make timer_ long waitTime = 2000 ; _// wait before first tick in ms_ long periodTime = 500 ; _// timer period_ Timer kicker = new Timer( tcb, state, waitTime, periodTime ) ; _// do some work_ ... kicker.Change( waitTime, periodTime ) ; _// do some more work_ ... kicker.Dispose() ; ... } For _waitTime_ and _periodTime_ you can use TimeSpan types if it makes more sense. In the supplied sample you can see that the same state object is used on both timers. When you run the sample the state values printed are not consecutive. I left it like this to show how important synchronization is in a multithread environment. The final timing options come from the System.Timers.Timer class. It represents server-based timer ticks for maximum accuracy. Ticks are generated outside of our process and can be used for watch-dog control. In the sameSystem.Timers namespace you can find the Schedule class which gives you the ability to schedule timer events fired at longer time intervals. System.Timers.Timer class is the most complete solution for all time fired events. It gives you the most precise control and timing and is surprisingly simple to use. 

  1. Create the Timer object. You can a use constructor with interval setting. 
  2. Add your event handler (delegate) to the Tick event 
  3. Set the Interval property to the desired number of milliseconds (default value is 100 ms) 
  4. Set the AutoReset property to false if you want the event to be raised only once (default is true � repetitive raising) 
  5. Start the ticking with a call to the Start() method, or by setting Enabled property to true. 
  6. Stop the ticking with call to the Stop() method or by setting the Enabled property to false.

Collapse[ Copy Code](<http://www.codeproject.com/KB/dotnet/multithread.aspx>) using System.Timers ;   void TickHandler( object sender, EventArgs e ){ _// do some work_ }   _// usage block_ { ... _// create timer_ Timer kicker = new Timer() ; kicker.Interval = 1000 ; kicker.AutoReset = false ;   _// add handler_ kicker.Tick += new EventHandler( TickHandler ) ;   _// start timer_ kicker.Start() ;   _// change interval_ kicker.Interval = 2000 ;   _// stop timer_ kicker.Stop() ;   _// you can start and stop timer againg_ kicker.Start() ; kicker.Stop() ; ... } I should mention a few things about using the Timers.Timer class: 

  * In VS Beta 1 you must add a reference to System.Timers namespace by hand. 
  * Whenever you use the Timers namespace together with System.WinForms or System.Threading, you should reference the Timer classes with the full name to avoid ambiguity. 
  * Be careful when using Timers.Timer objects. You may find them a lot faster than the old windows timers approach (think why). Do not forget synchronize data access.

**Thread pooling** The idea for making a pool of threads on the .NET framework level comes from the fact that most threads in multithreaded programs spend most of the time waiting for something to happen. It means that thread entry functions contain endless loops which calls real working functions. By using the ThreadPool type object preparing working functions is simpler and for bonus we get better resource usage. There are two important facts relating to ThreadPool object. 

  * There is only one ThreadPool type object per process 
  * There is only one working thread per thread pool object

The most useful use of a ThreadPool object is to add a new thread with a triggering event to the thread pool. i.e.. "when this event happens do this". For using ThreadPool this way you must perform following steps: 

  1. Create event 
  2. Create a delegate of type WaitOrTimerCallback
  3. Create an object which will carry status information to the delegate. 
  4. Add all to thread pool 
  5. Set event

In C#:  Collapse[ Copy Code](<http://www.codeproject.com/KB/dotnet/multithread.aspx>) _// status information object_ publicclass StatusObject{ _// some information_ }   _// thread entry function_ publicvoid someFunc( object obj, bool signaled ){ _// do some clever work_ }   _// usage block_ { ... _// create needed objects_ AutoResetEvent myEvent = new AutoResetEvent( false ) ; WaitOrTimerCallback myThreadMethod = new WairOrTimerCallback( someFunc ) ; StatusObject statusObject = new StatusObject() ;   _// decide how thread will perform_ int timeout = 10000 ; _// timeout in ms_ bool repetable = true ; _// timer will be reset after event fired or timeout_   _// add to thread pool_ ThreadPool.RegisterWaitForSingleObject( myEvent, myThreadMethod, statusObject, timeout, repetable ) ; ... _// raise event and start thread_ myEvent.Set() ; ... } A less common use of a thread pool will be (or at least should be, be aware of misuse) adding threads to be executed when the processor is free. You could think of this kind of usage as "OK, I have this to do, so do it whenever you have time". Very democratic way of handling background processing which can stop your program quickly. Remember that inside the thread pool you have only one thread working (per processor). Using thread pool this way is even simpler:

  1. Create a delegate of type WaitCallback
  2. Create an object for status information, if you need it 
  3. Add to thread pool

Collapse[ Copy Code](<http://www.codeproject.com/KB/dotnet/multithread.aspx>) _// status information object_ publicclass StatusObject{ _// some information_ }   _// thread entry function_ publicvoid someFunc( object obj, bool signaled ){ _// do some clever work_ }   _// usage block_ { ... _// create needed objects_ WaitCallback myThreadMethod = new WairOrTimerCallback( someFunc ) ; StatusObject statusObject = new StatusObject() ;   _// add to thread pool_ ThreadPool.QueueUserWorkItem( myThreadMethod, statusObject ) ; ... } Some notes on thread pool: 

  * Don�t use a thread pool for threads that perform long calculations. You only have one thread per processor actually working. 
  * System.Thread.Timer type objects are one thread inside thread pool 
  * You have only one thread pool per process

**Conclusion** This article shows the way I used to find out secrets of .NET multithreading. When you try using this, be careful on thread synchronization. This is also subject of my next article, where I will show all different ways of synchronization that .NET offers. For more information read articles in MSDN library: 

  * .NET framework design guidelines 
  * Threading design guidelines 
  * Asynchronous execution
