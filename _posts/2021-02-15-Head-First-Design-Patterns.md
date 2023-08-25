---
layout: post
title:  "Head First Design Patterns"
categories: technology
tags: design patterns
excerpt: "Basic object-oriented programming design patterns"
---

* content
{:toc}

## Head First

### Learning principles

Material:

1. Make it visual: graphics and words.

2. Personalized conversational style

3. Think deeply

4. Keep attention

5. Touch emotions

Actions:

1. Slow down to think, ask yourself questions

2. Write notes

3. Don't skip core conetent

4. No new things after learning

5. Drink water

6. Talk about it loudly

7. Take a break

8. Fee something

9. Apply the learning to problems

## Strategy Pattern

> Case: Duck classes, different ducks may share or not share some behaviors
Only inheritance: (-) not all subclasses have the same behaviers
Only interfaces: (-) behavior codes are not reused
Stable: Duck class -> keep it as a class to delegate behaviors
Changing: behaviors -> pull out, make it an interface with different implementations

✓ **Design Principle: Identify the aspects of your application that vary and separate them from what stays the same.** Most patterns and principles address issues of change in software. Most patterns allow some part of a system to vary independently of all other parts. We often try to take what varies in a system and encapsulate it.

✓ **Design Principle: Program to an interface or supertype, not an implementation.**

✓ **Design Principle: Favor composition over inheritance.**

➜ *The **Strategy** Pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.*

* Good OO designs are reusable, extensible, and maintainable.
* Patterns show you how to build systems with good OO design qualities.Patterns are proven object-oriented experience.
* Patterns don’t give you code, they give you general solutions to design problems. You apply them to your specific application.
* Patterns aren’t invented, they are discovered. Most patterns and principles address issues of change in software.
* Most patterns allow some part of a system to vary independently of all other parts.
* We often try to take what varies in a system and encapsulate it.
* Patterns provide a shared language that can maximize the value of your communication with other developers.

## Observer Pattern

We’re going to look at all kinds of interesting aspects of Observer, like its one-to-many relationships and loose coupling.

> Case: Weather updates are displayed in screens, there could be many screens

➜ *The **Observer** Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically.*

✓ **Design Principle: Strive for loosely coupled designs between objects that interact.**

* The Observer Pattern defines a one-to-many relationship between objects.
* Subjects update Observers using a common interface.
* Observers of any concrete type can participate in the pattern as long as they implement the Observer interface.
* Observers are loosely coupled in that the Subject knows nothing about them, other than that they implement the Observer interface.
* You can push or pull data from the Subject when using the pattern (pull is considered more “correct”).
* Swing makes heavy use (listeners) of the Observer Pattern, as do many GUI frameworks.

## Decorator Pattern

> Case: Beverage classes, beverages can have many toppings

✓ **Design Principle: Classes should be open for extension, but closed for modification.**

➜ *The **Decorator** Pattern attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.*

As long as you only write code against the abstract component type, the use of decorators will remain transparent to your code.
Java I/O also points out one of the downsides of the Decorator Pattern: designs using this pattern often result in a large number of small classes that can be overwhelming to a developer trying to use the Decorator-based API.

* Inheritance is one form of extension, but not necessarily the best way to achieve flexibility in our designs.
* In our designs we should allow behavior to be extended without the need to modify existing code.
* Composition and delegation can often be used to add new behaviors at runtime.
* The Decorator Pattern provides an alternative to subclassing for extending behavior.
* The Decorator Pattern involves a set of decorator classes that are used to wrap concrete components.
* Decorator classes mirror the type of the components they decorate. (In fact, they are the same type as the components they decorate, either through inheritance or interface implementation.)
* Decorators change the behavior of their components by adding new functionality before and/or after (or even in place of) method calls to the component.
* You can wrap a component with any number of decorators.
* Decorators are typically transparent to the client of the component—that is, unless the client is relying on the component’s concrete type.
* Decorators can result in many small objects in our design, and overuse can be complex.

## Factory Pattern

> Case: Pizza preparation, there are different styles of pizzas and condiments in different stores

A factory method handles object creation and encapsulates it in a subclass. This decouples the client code in the superclass from the object creation code in the subclass.

➜ *The **Factory Method** Pattern defines an interface for creating an object, but lets subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.*

✓ **Design Principle: Depend upon abstractions. Do not depond upon concrete classes.**
Dependency Inversion Principle: It suggests that our high-level components should not depend on our low-level components; rather, they should both depend on abstractions.

➜ *The **Abstract Factory** Pattern provides an interface for creating families of related or dependent objects without specifying their concrete classes.*

* All factories encapsulate object creation.
* Simple Factory, while not a bona fide design pattern, is a simple way to decouple your clients from concrete classes.
* Factory Method relies on inheritance: object creation is delegated to subclasses, which implement the factory method to create objects.
* Abstract Factory relies on object composition: object creation is implemented in methods exposed in the factory interface.
* All factory patterns promote loose coupling by reducing the dependency of your application on concrete classes.
* The intent of Factory Method is to allow a class to defer instantiation to its subclasses.
* The intent of Abstract Factory is to create families of related objects without having to depend on their concrete classes.
* The Dependency Inversion Principle guides us to avoid dependencies on concrete types and to strive for abstractions.
* Factories are a powerful technique for coding to abstractions, not concrete classes.

## Singleton Pattern

➜ *The **Singleton** Pattern ensures a class has only one instance, and provides a global point of access to it.*

> Case: Chocolate boiler

Multithreading:

1. Synchronizing getInstance()

2. Using eagerly created instance

3. Sychronizing the class in a double-checking lock and use volatile reference

* The Singleton Pattern also provides a global access point to that instance. And it is directly connected with other objects.
* Java’s implementation of the Singleton Pattern makes use of a private constructor, a static method combined with a static variable. Thus, it cannot be extended.
* Be careful if you are using multiple class loaders; this could defeat the Singleton implementation and result in multiple instances.
* You can use Java’s enums to simplify your Singleton implementation.

## Command Pattern

> Case: design a remote control API

➜ *The **The Command** Pattern encapsulates a request as an object, thereby letting you parameterize other objects with different requests, queue or log requests, and support undoable operations.*

A null object is useful when you don’t have a meaningful object to return, and yet you want to remove the responsibility for handling null from the client.

With lambda expressions, instead of instantiating the concrete command objects, you can use function objects in their place. But you can only do this if your Command interface has one abstract method.
http://tutorials.jenkov.com/java/lambda-expressions.html

* The Command Pattern decouples an object making a request from the one that knows how to perform it.
* A Command object is at the center of this decoupling and encapsulates a receiver with an action (or set of actions).
* An invoker makes a request of a Command object by calling its execute() method, which invokes those actions on the receiver.
* Invokers can be parameterized with Commands, even dynamically at runtime.
* Commands may support undo by implementing an undo() method that restores the object to its previous state before the execute() method was last called.
* MacroCommands are a simple extension of the Command Pattern that allow multiple commands to be invoked. Likewise, MacroCommands can easily support undo().
* In practice, it’s not uncommon for “smart” Command objects to implement the request themselves rather than delegating to a receiver.
* Commands may also be used to implement logging and transactional systems. The semantics of some applications require that we log all actions and be able to recover after a crash by reinvoking those actions. The Command Pattern can support these semantics with the addition of two methods: store() and load().

## Adpater and Facade Pattern

> Case: Turkey to Duck behavior

➜ *The **Adapter Pattern** converts the interface of a class into another interface the clients expect. Adapter lets classes work together that couldn’t otherwise because of incompatible interfaces.*

➜ *The **Facade Pattern** provides a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.*

✓ **Design Principle: Principle of Least Knowledge: talk only to your immediate friends.**
The principle provides some guidelines: take any object, and from any method in that object, invoke only methods that belong to the object itself, objects passed in as a parameter to the method, any object the method creates or instantiates, any components of the object.

* Facades and adapters may wrap multiple classes, but a facade’s intent is to simplify, while an adapter’s is to convert the interface to something different.
  * When you need to use an existing class and its interface is not the one you need, use an adapter.
  * When you need to simplify and unify a large interface or complex set of interfaces, use a facade.
* An adapter changes an interface into one a client expects.
* A facade not only simplifies an interface, it decouples a client from a subsystem of components.
* Implementing an adapter may require little work or a great deal of work depending on the size and complexity of the target interface.
* Implementing a facade requires that we compose the facade with its subsystem and use delegation to perform the work of the facade.
* There are two forms of the Adapter Pattern: object and class adapters. Class adapters require multiple inheritance.
  * The only difference is that with a class adapter we subclass the Target and the Adaptee, while with an object adapter we use composition to pass requests to an Adaptee.
* You can implement more than one facade for a subsystem.
* An adapter wraps an object to change its interface, a decorator wraps an object to add new behaviors and responsibilities, and a facade “wraps” a set of objects to simplify.

## Template Method Pattern

> Case: Coffee and Tea

➜ *The **Facade Pattern** defines the skeleton of an algorithm in a method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm’s structure.*

✓ **Design Principle: Hollywood: Don't call us, we'll call you.**

* A template method defines the steps of an algorithm, deferring to subclasses for the implementation of those steps.
* The Template Method Pattern gives us an important technique for code reuse.
* The template method’s abstract class may define concrete methods, abstract methods, and hooks.
  * Abstract methods are implemented by subclasses.
  * Hooks are methods that do nothing or default behavior in the abstract class, but may be overridden in the subclass.
  * granularity vs flexibility
* To prevent subclasses from changing the algorithm in the template method, declare the template method as final.
* The Hollywood Principle guides us to put decision making in high-level modules that can decide how and when to call low-level modules.
* You’ll see lots of uses of the Template Method Pattern in real-world code, but (as with any pattern) don’t expect it all to be designed “by the book.”
* The Strategy and Template Method Patterns both encapsulate algorithms, the first by composition and the other by inheritance.
* Factory Method is a specialization of Template Method.

## Iterator and Composite Patterns

> Case: Breakfast and Dinner Menus

➜ *The **Iterator Pattern** provides a way to access the elements of an aggregate object sequentially without exposing its underlying representation.*

✓ **Design Principle: A class should have only one reason to change.**
Every responsibility of a class is an area of potential change. More than one responsibility means more than one area of change.

Cohesion is a term you’ll hear used as a measure of how closely a class or a module supports a single purpose or responsibility.
We say that a module or class has high cohesion when it is designed around a set of related functions, and we say it has low cohesion when it is designed around a set of unrelated functions.
Cohesion is a more general concept than the Single Responsibility Principle, but the two are closely related. Classes that adhere to the principle tend to have high cohesion and are more maintainable than classes that take on multiple responsibilities and have low cohesion.

➜ *The **Composite Pattern** allows you to compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.*

* An Iterator allows access to an aggregate’s elements without exposing its internal structure.
* An Iterator takes the job of iterating over an aggregate and encapsulates it in another object.
* When using an Iterator, we relieve the aggregate of the responsibility of supporting operations for traversing its data.
* An Iterator provides a common interface for traversing the items of an aggregate, allowing you to use polymorphism when writing code that makes use of the items of the aggregate.
* The Iterable interface provides a means of getting an iterator and enables Java’s enchanced for loop.
* We should strive to assign only one responsibility to each class.
* The Composite Pattern allows clients to treat composites and individual objects uniformly.
* A Component is any object in a Composite structure. Components may be other composites or leaves.
* There are many design tradeoffs in implementing Composite. You need to balance **transparency and safety** with your needs.

## State Pattern

> Case: Gumball machine

➜ *The **State Pattern** allows an object to alter its behavior when its internal state changes. The object will appear to change its class.*

* The State Pattern allows an object to have many different behaviors that are based on its internal state.
* Unlike a procedural state machine, the State Pattern represents each state as a full-blown class.
* The Context gets its behavior by delegating to the current state object it is composed with.
* By encapsulating each state into a class, we localize any changes that will need to be made.
* The State and Strategy Patterns have the same class diagram, but they differ in intent.
  * The Strategy Pattern typically configures Context classes with a behavior or algorithm.
  * The State Pattern allows a Context to change its behavior as the state of the Context changes.
* State transitions can be controlled by the State classes or by the Context classes.
* Using the State Pattern will typically result in a greater number of classes in your design.
* State classes may be shared among Context instances.

## Proxy Pattern

> Case: Gumballmachine Monitor

### Java RMI

1. Make a Remote Interface
    1. Extend java.rmi.Remote

          ```java
          public interface MyRemote extends Remote
          ```

    2. Declare that all methods throw RemoteException

        ```java
        import java.rmi.*;
        public interface MyRemote extends Remote {
          public String sayHello() throws RemoteException;
        }
        ```

    3. Be sure arguments and return values are primitives or Serializable

2. Make a Remote Implementation
    1. Implement the Remote interface

        ```java
        public class MyRemoteImpl extend UnicastRemoteObject implements MyRemote {
          public String sayHello() {
            return "Server says: hi!";
          }
        }
        ```

    2. Extend UnicastRemoteObject

        ```java
        public class MyRemoteImpl extend UnicastRemoteObject implements MyRemote {
          private static final long serialVersionUID = 1L;
        }
        ```

    3. Write a no-arg constructor that declares RemoteException

        ```java
        public MyRemoteImpl() throws RemoteException {}
        ```

    4. Register the service with the RMI registry

        ```java
        try {
          MyRemote service = new MyRemoteImpl();
          Naming.rebind("RemoteHello", service);
          // client will use Naming.lookup("rmi://127.0.0.1/RemoteHello") to get a stub object of the remote service.

        } catch (Exception ex) {...}
        ```

3. Start the RMI registry
    1. Bring up a terminal and start the rmiregistry.

4. Start the remote service
    1. Bring up another terminal and start your service

### Pattern

➜ *The **Proxy Pattern** provides a surrogate or placeholder for another object to control access to it.*

Use the Proxy Pattern to create a representative object that controls access to another object, which may be remote (remote proxy), expensive to create (virtual proxy), or in need of securing (protection proxy).

### Java API’s dynamic proxy

1. Create InvocationHandlers: implement the behavior of the proxy. Java will take care of creating the actual proxy class and object.

    ```java
    public class UserInvocationHandler implements InvocationHandler {
      Person person;

      public UserInvocationHandler(Person person) {
        this.person = person;
      }

      public Object invoke(Object proxy, Method method, Object[] args) throws IllegalAccessException {


        try {
          if (method.getName().startsWith("yyy")) {
            return method.invoke(this.person, args);
          } else ...
        } catch (InvocationTargetException e) {...}
      }
      return null;
    }
    ```

2. Write the code that creates the dynamic proxies.

    ```java
    // takes the real object and return a proxy one
    Person getUserProxy(Person person) {
      return (Person) Proxy.newProxyInstance(
        person.getClass().getClassLoader(),
        person.getClass().getInterfaces(),
        new UserInvocationHandler(person));
      ) 
      // this class is created at runtime
      // this Proxy class has a static method isProxyClass()
    }
    ```

3. Wrap the real object with the appropriate proxy.

    ```java
    Person proxyPerson = getUserProxy(realPerson);
    ```

### Other proxies

* Firewall Proxy: access control
* Smart Reference Proxy: additional actions whenever a subject is reference
* Caching Proxy: temporary storage
* Synchronization Proxy: safe for multiple threads
* Complexity Hiding Proxy, Facade Proxy: control access instead of alternative interface
* Copy-On-Write Proxy: deferring the copying of an object until it is required

## Coumpound Pattern

Patterns are often used together and combined within the same design solution.
A compound pattern combines two or more patterns into a solution that solves a recurring or general problem.

### Patterns work together

* A goose came along and wanted to act like a Quackable too. So we used the *Adapter Pattern* to adapt the goose to a Quackable. Now, you can call quack() on a goose wrapped in the adapter.
* The Quackologists decided they wanted to count quacks. So we used the *Decorator Pattern* to add a QuackCounter decorator that keeps track of the number of times quack() is called, and then delegates the quack to the Quackable it’s wrapping.
* The Quackologists were worried they’d forget to add the QuackCounter decorator. So we used the *Abstract Factory Pattern* to create ducks for them. Now, whenever they want a duck, they ask the factory for one, and it hands back a decorated duck. (And don’t forget, they can also use another duck factory if they want an undecorated duck!)
* We had management problems keeping track of all those ducks and geese and quackables. So we used the *Composite Pattern* to group Quackables into Flocks. The pattern also allows the Quackologist to create subFlocks to manage duck families. We used the *Iterator Pattern* in our implementation by using java.util’s iterator in ArrayList.
* The Quackologists also wanted to be notified when any Quackable quacked. So we used the *Observer Pattern* to let the Quackologists register as Quackable Observers. Now they’re notified every time any Quackable quacks. We used iterator again in this implementation. The Quackologists can even use the Observer Pattern with their composites.

### Model-View-Controller

**Model**: The model holds all the data, state, and application logic. The model is oblivious to the view and controller, although it provides an interface to manipulate and retrieve its state and it can send notifications of state changes to observers.
**View**: Gives you a presentation of the model. The view usually gets the state and data it needs to display directly from the model.
**Controller**: Takes user input and figures out what it means to the model.

1. The user does something to the view. The view tells the controller what the user did.
2. The controller takes the actions and interprets them that how the model should be manipulated based on that action.
3. The controller can also tell the view to change as a result.
4. When something changes in the model, the model notifies the view about the changed state.
5. The view gets the state directly from the model and displays it. It can also ask the model for state as the result of controller requesting some change in the view.
6. The controller may also be notified of changes and then interact with the view.

Patterns:

* The model uses *Observer* to keep the views and controllers updated on the latest state changes.
* The view and the controller, on the other hand, implement the *Strategy* Pattern. The controller is the strategy of the view, and it can be easily exchanged with another controller if you want different behavior.
* The view itself also uses a pattern internally to manage the windows, buttons, and other components of the display: the *Composite* Pattern.
* The *Adapter* Pattern can be used to adapt a new model to an existing view and controller.

MVC has been adapted to the web. There are many web MVC frameworks with various adaptations of the MVC pattern to fit the client/server application structure.

## Guidance on Patterns

➜ *A **Pattern** is a solution to a problem in a context.* The context

The context is the situation in which the pattern applies. This should be a recurring situation.
The problem refers to the goal you are trying to achieve in this context, but it also refers to any constraints that occur in the context.
The solution is what you’re after: a general design that anyone can apply that resolves the goal and set of constraints.

To be a pattern writer:

1. Do your homework
2. Take tiem to reflect, evaluate
3. Get your ideas down on paper in a way others can understand
4. Have others try your patterns; then refine and refine some more
5. Don't forget the Rule of Three

Categories:

* Creational: involve object instantiation and all provide a way to decouple a client from the objects it needs to instantiate
  * Singleton
  * Builder
  * Factory Method
  * Abstract Factory
  * Prototype
* Behavioral: concerned with how classes and objects interact and distribute responsibility
  * Template Method
  * Visitor
  * Mediator
  * Iterator
  * Interpreter
  * Command
  * Observer
  * Memento
  * State
  * Strategy
  * Chain of Responsibility
* Sturctural: let you compose classes or objects into larger structures
  * Decorator
  * Composite
  * Proxy
  * Flyweight
  * Bridge
  * Adapter
  * Facade

Experience and practice:
Let patterns emerge naturally as your design progresses.

* The goal should be implicity rather than how to apply a pattern.
* Patterns themselves are not making things done. To use patterns, you still need to think through the consequences for the rest of your design.
* Knowing when a pattern applies is where your experience and knowledge come in. Once you’re sure a simple solution will not meet your needs, you should consider the problem along with the set of constraints under which the solution will need to operate — these will help you match your problem to a pattern. Just make sure you are adding patterns to deal with practical change that is likely to happen, not hypothetical change that may happen.
* Refactoring is the process of making changes to your code to improve the way it is organized. The goal is to improve its structure, not change its behavior. This is a great time to reexamine your design to see if it might be better structured with patterns.
* Take out what you don’t really need. Don’t be afraid to remove a pattern from your design when a simpler solution without the pattern would be better.
* If you have a practical need to support change in a design today, go ahead and employ a pattern to handle that change. However, if the reason is only hypothetical, don’t add the pattern; it will only add complexity to your system.

## Other Patterns

### Bridge

> Problem: how to create an object-oriented design that allows you to vary the implementation and the abstraction?

Use the Bridge Pattern to vary not only your implementations, but also your abstractions.
The Bridge Pattern allows you to vary the implementation and the abstraction by placing the two in separate class hierarchies.

The `Abstraction` (abstract class) will hold a handler/bridge to an `Implementor` (interface). The former will have `RefinedAbstraction` classes to control variation in one aspect, the latter will have `ConcreteImplementor` to control variation in another aspect. Then these two are combined via the bridge (usually the Implementor is passed into the class constructor of the Abstraction).

Pros/Benefits:

* Decouples an implementation so that it is not bound permanently to an interface.
* Abstraction and implementation can be extended independently.
* Changes to the concrete abstraction classes don’t affect the client.

Cons/Drawbacks:

* Useful any time you need to vary an interface and an implementation in different ways.
* Increases complexity.

Uses:

* Useful in graphics and windowing systems that need to run over multiple platforms.

### Builder

> Problem: how to provide a way to create the complex structure without mixing it with the steps for creating it?

Use the Builder Pattern to encapsulate the construction of a product and allow it to be constructed in steps.

We encapsulate the creation of the `Product` object in a `Builder` (it can be an interface with different `ConcreteBuilder` classes), and have our client ask the builder to construct the structure for it using `Director`(has detailed logic and provides `void construct(Builder builder)` method and `getProduct()` method, this part can be done by the client as well).

Pros:

* Encapsulates the way a complex object is constructed.
* Allows objects to be constructed in a multistep and varying process (as opposed to one-step factories).
* Hides the internal representation of the product from the client.
* Product implementations can be swapped in and out because the client only sees an abstract interface.

Cons:

* Constructing objects requires more domain knowledge of the client than when using a Factory.

Uses:

* Often used for building composite structures.

### Chain of Responsibility

Use the Chain of Responsibility Pattern when you want to give more than one object a chance to handle a request.

With the Chain of Responsibility Pattern, you create a chain of objects to examine requests. Each object in turn examines a request and either handles it or passes it on to the next object in the chain.

Pros:

* Decouples the sender of the request and its receivers.
* Simplifies your object because it doesn’t have to know the chain’s structure and keep direct references to its members.
* Allows you to add or remove responsibilities dynamically by changing the members or order of the chain.

Cons:

* Execution of the request isn’t guaranteed; it may fall off the end of the chain if no object handles it (this can be an advantage or a disadvantage).
* Can be hard to observe and debug at runtime.

Uses:

* Commonly used in Windows systems to handle events like mouse clicks and keyboard events.

### Flyweight

Use the Flyweight Pattern when one instance of a class can be used to provide many virtual instances.

Pros:

* Reduces the number of object instances at runtime, saving memory.
* Centralizes state for many “virtual” objects into a single location.

Cons:

* A drawback of the Flyweight Pattern is that once you’ve implemented it, single, logical instances of the class will not be able to behave independently from the other instances.

Uses:

* The Flyweight is used when a class has many instances, and they can all be controlled identically.

### Interpreter

Use the Interpreter Pattern to build an interpreter for a language.

When you need to implement a simple language, the Interpreter Pattern defines a class-based representation for its grammar along with an interpreter to interpret its sentences. To represent the language, you use a class to represent each rule in the language. Here’s the duck language translated into classes. Notice the direct mapping to the grammar.

To interpret the language, call the interpret() method on each expression type. This method is passed a context—which contains the input stream of the program we’re parsing—and matches the input and evaluates it.

Pros:

* Representing each grammar rule in a class makes the language easy to implement.
* Because the grammar is represented by classes, you can easily change or extend the language.
* By adding methods to the class structure, you can add new behaviors beyond interpretation, like pretty printing and more sophisticated program validation.

Cons:

* This pattern can become cumbersome when the number of grammar rules is large. In these cases a parser/compiler generator may be more appropriate.

Uses:

* Use Interpreter when you need to implement a simple language.
* Appropriate when you have a simple grammar and simplicity is more important than efficiency.
* Used for scripting and programming languages.

### Mediator

Use the Mediator Pattern to centralize complex communications and control between related objects.

With a Mediator added to the system, all of the appliance objects can be greatly simplified:
They tell the Mediator when their state changes.
They respond to requests from the Mediator.

Before we added the Mediator, all of the appliance objects needed to know about each other; that is, they were all tightly coupled. With the Mediator in place, the appliance objects are all completely decoupled from each other.
The Mediator contains all of the control logic for the entire system.

Pros:

* Increases the reusability of the objects supported by the Mediator by decoupling them from the system.
* Simplifies maintenance of the system by centralizing control logic.
* Simplifies and reduces the variety of messages sent between objects in the system.

Cons:

* A drawback of the Mediator Pattern is that without proper design, the Mediator object itself can become overly complex.

Uses:

* The Mediator is commonly used to coordinate related GUI components.

### Memento

Use the Memento Pattern when you need to be able to return an object to one of its previous states; for instance, if your user requests an “undo.”

The Memento has two goals:
Saving the important state of a system’s key object
Maintaining the key object’s encapsulation

Keeping the Single Responsibility Principle in mind, it’s also a good idea to keep the state that you’re saving separate from the key object. This separate object that holds the state is known as the Memento object.

Pros:

* Keeping the saved state external from the key object helps to maintain cohesion.
* Keeps the key object’s data encapsulated.
* Provides easy-to-implement recovery capability.

Cons:

* A drawback to using Memento is that saving and restoring state can be time-consuming.

Uses:

* The Memento is used to save state.
* Serialization

### Prototype

Use the Prototype Pattern when creating an instance of a given class is either expensive or complicated.

The Prototype Pattern allows you to make new instances by copying existing instances. (In Java this typically means using the clone() method, or deserialization when you need deep copies.) A key aspect of this pattern is that the client code can make new instances without knowing which specific class is being instantiated.

Pros:

* Hides the complexities of making new instances from the client.
* Provides the option for the client to generate objects whose type is not known.
* In some circumstances, copying an object can be more efficient than creating a new object.

Cons:

* A drawback to using Prototype is that making a copy of an object can sometimes be complicated.

Uses:

* Prototype should be considered when a system must create new objects of many types in a complex class hierarchy.

### Visitor

Use the Visitor Pattern when you want to add capabilities to a composite of objects and encapsulation is not important.

The Visitor works hand in hand with a Traverser. The Traverser knows how to navigate to all of the objects in a Composite. The Traverser guides the Visitor through the Composite so that the Visitor can collect state as it goes. Once state has been gathered, the Client can have *the Visitor perform various operations on the state*. When new functionality is required, only the Visitor must be enhanced.

Pros:

* Allows you to add operations to a Composite structure without changing the structure itself.
* Adding new operations is relatively easy.
* The code for operations performed by the Visitor is centralized.

Cons:

* The Composite classes’ encapsulation is broken when the Visitor is used.
* Because the traversal function is involved, changes to the Composite structure are more difficult.