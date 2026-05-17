---
title: "Coded UI - c# tricks"
date: 2015-08-25 10:48:43 
author: shajeebhameed
layout: page
slug: coded-ui-c-tricks
status: publish
---

1. **using**

When the `SomeType` class implements [`IDisposable`](<http://msdn.microsoft.com/en-us/library/system.idisposable.aspx>). Equalent to try finally block. 

wo uses: 

  * Using _directives_ , e.g. 
        
        using System;
        using System.IO;
        using WinForms = global::System.Windows.Forms;
        using WinButton = WinForms::Button;

These are used to import namespaces (or create aliases for namespaces or types). These go at the top of the file, before any declarations.
  * Using _statements_ e.g. 
        
        using (Stream input = File.OpenRead(filename))
        {
            ...
        }

This can only be used with types that implement `IDisposable`, and is syntactic sugar for a try/finally block which calls `Dispose` in the finally block. This is used to simplify resource management.



  1. **Virtual, Override and New keyword**



### http://www.codeproject.com/Articles/816448/Virtual-vs-Override-vs-New-Keyword-in-Csharp

### Virtual Keyword

`Virtual `keyword is used for generating a virtual path for its derived classes on implementing method overriding.`Virtual `keyword is used within a set with `override `keyword. It is used as: 

Hide Copy Code
    
    
    // Base Class
        class A
        {
            public virtual void show()
            {
                Console.WriteLine("Hello: Base Class!");
                Console.ReadLine();
            }
        }

### Override Keyword

`Override `keyword is used in the derived class of the base class in order to override the base class method.`Override `keyword is used with `virtual `keyword, as: 

Hide Copy Code
    
    
    // Base Class
        class A
        {
            public virtual void show()
            {
                Console.WriteLine("Hello: Base Class!");
                Console.ReadLine();
            }
        }
    
    // Derived Class
    
        class B : A
        {
            public override void show()
            {
                Console.WriteLine("Hello: Derived Class!");
                Console.ReadLine();
            }
        }

### New Keyword

`New `keyword is also used in polymorphism concept, but in the case of method overloading So what does overloading means, in simple words we can say procedure of hiding your base class through your derived class. It is implemented as: 

Hide Copy Code
    
    
    class A
        {
            public void show()
            {
                Console.WriteLine("Hello: Base Class!");
                Console.ReadLine();
            }
        }
    
        class B : A
        {
            public new void show()
            {
                Console.WriteLine("Hello: Derived Class!");
                Console.ReadLine();
            }
        }

  1. **polimorphism – compile time and run time**
  2. **boxing and unboxing**
  3. **hashtable and dictionary**

#### Hashtable and Dictionary are collection of data structures to hold data as key-value pairs. Dictionary is generic type, hash table is not a generic type. The Hashtable is a weakly typed data structure, so you can add keys and values of any Object Type to the Hashtable. The Dictionary class is a strongly types < T Key, T Value > and you must specify the data types for both the key and value.

#### Dictionary:

     * It returns error if we try to find a key which does not exist.
     * It is faster than a Hashtable because there is no boxing and unboxing.
     * Only public static members are thread safe.
     * Dictionary is a generic type which means we can use it with any data type.

#### Hashtable:

     * It returns null if we try to find a key which does not exist.
     * It is slower than dictionary because it requires boxing and unboxing.
     * All the members in a Hashtable are thread safe,
     * Hashtable is not a generic type
  4. Hashtable numbers = new Hashtable(); numbers.Add(1, "one"); numbers.Add(2, "two"); numbers.Add(3, "three"); numbers.Add(4, "four"); numbers.Add(5, "five"); foreach (DictionaryEntry num in numbers) { MessageBox.Show(num.Key + " - " + num.Value); } Dictionary<int, string> dictionary = new Dictionary<int, string >(); dictionary.Add(1,"one"); dictionary.Add(2,"two"); dictionary.Add(3,"three"); dictionary.Add(4,"four"); dictionary.Add(5, "five"); foreach (KeyValuePair<int,string> pair in dictionary) { MessageBox.Show(pair.Key + " - " + pair.Value); }


  1. **ArrayList – only stores object or ref types.** http://www.codeproject.com/Tips/855546/What-is-the-Difference-between-ArrayList-and-List
  2. First, one should know what is upcasting? Upcasting is converting derived type into base type. In .NET, all data-types are derived from `Object`. So we can typecast any type to `Object `type. For example, if `Customer` is class, then we can create object of `Customer` like this: 

Hide Copy Code
         
         Object cust = new Customer()
         

Here `new Customer()` will create object on heap and its address we are putting in reference variable of type`Object`. ![](http://www.codeproject.com/KB/dotnet/855546/upcasting.png)
  3. ## ArrayList

Hide Copy Code
         
         ArrayList marks = new ArrayList();
         marks.Add(50);
         marks.Add(70.5);
         marks.Add("Sixty");

In the above code snippet, we are creating object of `ArrayList `and adding different type of data in it. But actually `ArrayList `is a collection of `**Object**`type, and when we add any `**item**`to `ArrayList`, it first **converts** it to object type (upcasting) and then adds it to collection object. **Interesting Fact** : As `ArrayList `can only create collection for `Object `type, it is said to be non-generic class. It might be confusing as it seems that we can add any `datatype `value like `int`, `float`, `string `to `ArrayList`collection so in that sense it should be called as generic class. But in fact, it internally converts all these `datatypes`in `object `type and then adds to collection. We can visualise it as: ![](http://www.codeproject.com/KB/dotnet/855546/ArrayList.png)

## List

Hide Copy Code
         
         List<int> marks = new List<int>();
         
         marks.Add(50);
         marks.Add(70);
         marks.Add(60);

In the above snippet, we can observe that while creating object of `list `class, we have mentioned `datatype `of collection we want to create. We need to pass `datatype `while creating object as `List `class doesn’t hard code it internally. So the above declaration will create marks as collection of **integers** ; and **not** collection of **objects** as in case of `ArrayList`.
  4. **abstract and interface**
  5. There are some similarities and differences between an interface and an abstract class that I have arranged in a table for easier comparison:  Feature | Interface | Abstract class  
---|---|---  
Multiple inheritance | A class may inherit several interfaces. | A class may inherit only one abstract class.  
Default implementation | An interface cannot provide any code, just the signature. | An abstract class can provide complete, default code and/or just the details that have to be overridden.  
Access Modfiers | An interface cannot have access modifiers for the subs, functions, properties etc everything is assumed as public | An abstract class can contain access modifiers for the subs, functions, properties  
Core VS Peripheral | Interfaces are used to define the peripheral abilities of a class. In other words both Human and Vehicle can inherit from a IMovable interface. | An abstract class defines the core identity of a class and there it is used for objects of the same type.  
Homogeneity | If various implementations only share method signatures then it is better to use Interfaces. | If various implementations are of the same kind and use common behaviour or status then abstract class is better to use.  
Speed | Requires more time to find the actual method in the corresponding classes. | Fast  
Adding functionality (Versioning) | If we add a new method to an Interface then we have to track down all the implementations of the interface and define implementation for the new method. | If we add a new method to an abstract class then we have the option of providing default implementation and therefore all the existing code might work properly.  
Fields and Constants | No fields can be defined in interfaces | An abstract class can have fields and constrants defined  
  6. **design pattern**
  7. **find element using selenium**
  8. **will finally executes all time? yes, even if try returns** **===========================**
  9. **dependency injection – when App2 is not ready how to automate**
  10. **interdependent module.**
  11. **Soap**
  12. **can web api delete operation have post data**
  13. **switch and if differences**

IF: it works under the Boolean literal whereas SWITCH: it works under the integer/character literal **switch**

  * `switch` is usually more compact than lots of nested `if else` and therefore, more readable
  * If you omit the `break` between two switch cases, you can fall through to the next case in many C-like languages. With `if else` you'd need a `goto` (which is not very nice to your readers ... if the language supports `goto` at all).
  * In most languages, `switch` only accepts primitive types as key and constants as cases. This means it can be optimized by the compiler using a jump table which is very fast.
  * It is not really clear how to format `switch` correctly. Semantically, the cases are jump targets (like labels for `goto`) which should be flush left. Things get worse when you have curly braces: 
        
        case XXX: {
        } break;
        

Or should the braces go into lines of their own? Should the closing brace go behind the `break`? How unreadable would that be? etc.
  * In many languages, `switch` only accepts only some data types.

**if-else**

  * `if` allows complex expressions in the condition while switch wants a constant
  * You can't accidentally forget the `break` between `if`s but you can forget the `else` (especially during cut'n'paste)
  * it accepts all data types.


  1. **waterfall and agile diff**
