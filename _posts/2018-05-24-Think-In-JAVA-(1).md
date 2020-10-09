---
layout: post
title:  "Think in Java -- OOP"
categories: tech
tags:  Java OOP
excerpt: "Reading notes for \"Think in Java\""
---

* content
{:toc}



Among all articles talking about OOP that I've read, this book is the best one. Bruce Eckel, the author, uses only 15 pages with clear and elegant words to give an introduction of the key concepts in OOP.

## Object-Oriented-Programming

### OOP

In fact, all programming languages provide abstractions. That is a point that I've never think about.

A problem is always divided into two space, the problem space and the solution space. The latter is the place where the problem exists, such as a business; the latter is the place where you're implementing that solution, such as a computer.
The programmer needs to build a business model in the solution space, build a machine model in the solution space and establish the association between the two.

Many approaches try to provide tools for programmers in the solution space but object-oriented approach in the problem space. We refer to the elements in the problem space and their representations in the solution space as “objects.” Thus, OOP allows you to describe the problem in terms of the problem, rather than in terms of the computer.

Alan Kay, a famous OOP pioneer, summarized five basic characteristics of Smalltalk, the first successful object-oriented language:

    1. Everything is an object
    2. A program is a bunch of objects telling each other what to do by sending messages.
    3. Each object has its own memory made up of other objects.
    4. Every object has a type.
    5. All objects of a particular type can receive the same messages.

Grady Booch, who developed Unified Modeling Language (UML), offers an even more succinct description of an object:

    An object has state, behavior and identity.


### Class and interface

A class describes a set of objects that have identical characteristics (data elements) and behaviors (functionality).

You can create objects or instances of a class and sending messages or requests to them. You send a message and the object figures out what to do with it. The requests you can make of an object are defined by its interface, and the type is what determines the interface.

We can break up the playing field into class creators (those who create new data types) and client programmers (the class consumers who use the data types in their applications). Class creators need to design the interface and implement how a class do with messages, while client programmers only focus on using classes.

To set different access for different users, Java uses three explicit keywords to set the boundaries in a class: public, private, and protected.

    1. public: the following element is available to everyone
    2. private: no one can access that element except inside methods of that class. Private is a brick wall between you and the client programmer. A private member is invisible from outside.
    3. protected: acts like private, with the exception that an inheriting class has access to protected members, but not private members
    4. “default” access: when not using aforementioned specifiers. This is usually called package access because classes can access the members of other classes in the same package (library component), but outside of the package those same members appear to be private.

Thinking of an object as a service provider has an additional benefit: It helps to improve the cohesiveness of the object.
> High cohesion is a fundamental quality of software design: It means that the various aspects of a software component (such as an object, although this could also apply to a method or a library of objects) “fit together” well.

In a good object-oriented design, each object does one thing well, but doesn't try to do too much. In my opinion, functions should follow the same style -- a good function does one thing well but not too much. Linux kernel codes are good examples.


### Reusing and inheritance

A class does more than describe the constraints on a set of objects; it also has a relationship with other classes. A new class can be composed from existing classes, which is called composition (if the composition happens dynamically, it's usually called aggregation). Inheritance expresses this similarity between types by using the concept of base classes and derived classes.

With objects, the type(class) hierarchy is the primary model, so you go directly from the description of the system in the real world to the description of the system in code. Casting the solution in the same terms as the problem is very useful because you don't need a lot of intermediate models to get from a description of the problem to a description of the solution.

* Is-a vs. is-like-a relationships

    There is a debate here: Should inheritance override only base class methods (and not add new methods that aren't in the base class)? Should base class and derived class have exactly the same interface?

    Some people prefer pure substitution which is often referred to as the substitution principle. Bill Harlan has a good paper [here](http://www.billharlan.com/papers/Avoid_extending_classes.html) talking about avoiding extending classes.

    I think it is not easy to avoid extending classes. However, we should think more when we want to add brand new methods to the derived class. We should look closely for the possibility that our base class might also need these additional methods. The process of discovery and iteration of your design happens regularly in object-oriented programming.


### Polymorphism

A class can be cast downcast into children(derived) classes and thus is polymorphic. It can have different actions with a same interface. In Java, dynamic binding is the default behavior and you don't need to remember to add any extra keywords in order to get polymorphism. Then, casting to a base type is moving up the inheritance diagram: “upcasting.”

> There is an important difference between an OOP compiler and a non-OOP compiler. A non-OOP compiler use early binding which means the compiler generates a call to a specific function name, and the runtime system resolves this call to the absolute address of the code to be executed. In comparison, object-oriented languages use the concept of late binding. When you send a message to an object, the code being called isn't determined until run time. The compiler does ensure that the method exists and performs type checking on the arguments and return value, but it doesn't know the exact code to execute.

With polymorphism, the program is extensible.


### Object class and containers

There is an ultimate base class -- Object. It turns out that the benefits of the singly rooted hierarchy are many.

All objects in a singly rooted hierarchy can be guaranteed to have certain functionality. You know you can perform certain basic operations on every object in your system. All objects can easily be created on the heap, and argument passing is greatly simplified. A singly rooted hierarchy makes it much easier to implement a garbage collector, which is one of the fundamental improvements of Java over C++.

Besides directly manipulate objects, we also need arrangement of objects. Fortunately, a good OOP language comes with a set of containers as part of the package. Those containers can all hold Objects.

A container that holds Objects can hold anything. But when you added an object reference into the container it was upcast to Object, thus losing its character. When fetching it back, you got an Object reference, and not a reference to the type that you put in. Then you should cast down the hierarchy to a more specific type.

A solution for that is to use a parameterized type mechanism. A parameterized type is a class that the compiler can automatically customize to work with particular types.
One of the big changes in Java SE5 is the addition of parameterized types, called generics in Java.


### Lifetime

You can placing the objects on the stack (these are sometimes called automatic or scoped variables) or in the static storage area. This places a priority on the speed of storage allocation and release, and this control can be very valuable in some situations. Another approach is to create objects dynamically in a pool of memory called the heap. In this approach, you don't need to know the amount and size of objects until run time.

In Java, every time you want to create an object, you simply use the `new` operator to build a dynamic instance of that object.
And you don't need to worry about destroying the object within Java, since the garbage collector is designed to take care of the problem of releasing the memory.


### Exception

A thrown exception is unlike an error value that’s returned from a method or a flag that’s set by a method in order to indicate an error condition—these can be ignored. An exception cannot be ignored, so it’s guaranteed to be dealt with at some point. Finally, exceptions provide a way to reliably recover from a bad situation. Instead of just exiting the program, you are often able to set things right and restore execution, which produces much more robust programs.


### Concurrency

Separately running pieces are called threads, and the general concept is called concurrency.
Java support concurrency at the language level so the programmer doesn’t need to worry about whether there are many processors or just one. With threads, the program can be logically divided into tasks.
