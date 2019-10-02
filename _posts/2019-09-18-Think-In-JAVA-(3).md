---
layout: post
title:  "Think in Java"
categories: Java
tags:  Java
excerpt: "Some interesting words from \"Think in Java\" "
---

* content
{:toc}


Finally, I finished reading this book except for the last chapter talking about Java GUI. I've spent many hours on the 900+ pages. Apparently, I am not good at fast reading.

I know Java and use Java but I still think this book really a good one. It not only help me go over the basic knowledge but also get me more familiar with some Java features, OO programming and even design patterns which I've planned to read about (maybe next year).

The author Bruce Eckel is an interesting man. He has some interesting words in the book.

Here I put some below:


- In any relationship it’s important to have boundaries that are respected by all parties involved.

- So you should be prepared for some compromises here and there.

- Over time, you’ll become better at recognizing situations.

- Any abstraction should be motivated by a real need.

- The basic philosophy of Java is that "badly formed code will not be run."

- An exceptional condition is a problem that prevents the continuation of the current method or scope.

- Don’t catch an exception unless you know what to do with it.

- Examination of small programs leads to the conclusion that requiring exception specifications could both enhance developer productivity and enhance code quality, but experience with large software projects suggests a different result—decreased productivity and little or no increase in code quality.

- A good programming language is one that helps programmers write good programs. No programming language will prevent its users from writing bad programs

- Neal Gafter (one of the lead developers for Java SE5) points out that he was lazy when rewriting the Java libraries, and that we should not do what he did. Neal also points out that he could not fix some of the Java library code without breaking the existing
interface. So even if certain idioms appear in the Java library sources, that’s not necessarily the right way to do it. When you look at library code, you cannot assume that it’s an example
that you should follow in your own code.

- A primary goal of programming design is to "separate things that change from things that stay the same".

- One of the problems with the Java I/O library is that it requires you to write quite a bit of code in order to perform these common operations—there are no basic helper functions to do them for you. What’s worse, the decorators make it rather hard to remember how to open files.

- There has been a big improvement in Java SE5: They’ve finally added the kind of output formatting that virtually every other language has always supported.

- However, there’s nothing that prevents you from having table produce a function object. For certain types of problems, the concept of "table-driven code" can be very powerful.


- As I have been saying throughout this book, elegance is important, and clarity may make the difference between a successful solution and one that fails because others cannot understand it.

- It’s hard enough to write tests without adding any new hurdles, so
@Unit tries to make it trivial. This way, you’re more likely to actually write the tests.


- Providers of APIs and frameworks will start providing annotations as part of their toolkits. As you can imagine by seeing the @Unit
system, it is very likely that annotations will cause significant changes in our Java programming experience.
> e.g. lombok package

- You can think of a single-threaded program as one lonely entity moving around through your problem space and doing one thing at a time. Because there's only one entity, you never have to think about the problem of two entities trying to use the same resource at the same time, problems like two people trying to park in the same space, walk through a door at the same time, or even talk at the same time. With multithreading, things aren't lonely anymore, but you now have the possibility of two or more threads trying to use the same limited resource at once.

