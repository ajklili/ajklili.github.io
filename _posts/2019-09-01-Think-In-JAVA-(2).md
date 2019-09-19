---
layout: post
title:  "Think in Java -- Multithreading"
categories: Java
tags:  Java multithread
excerpt: "Reading notes for \"Think in Java\" multithreading part"
---

* content
{:toc}


## Threads
Using multithreading, each of these independent tasks (also called subtasks) is driven by a thread of execution.

A thread is a single sequential flow of control within a process.
A single process can have multiple concurrently executing tasks.


### CPU
Since an underlying mechanism divides up the CPU time for you. One of the great things about threading is that you are abstracted away
from this layer, you does not need to know whether it is running on a single CPU or many.
Thus, using threads is a way to create transparently scalable programs.


### task
A thread drives a task, so you need a way to describe that task. This is provided by the `Runnable` interface. To define a task, simply implement `Runnable` and write a `run()` method to make the task do your bidding.

```java
public class SomeRunnable implements Runnable {
    public SomeRunnable() {}
    public void run() {
        while(countDown-- > 0) {
            System.out.print(status());
            Thread.yield();
        }
    }
}

public class MainThread {
    public static void main(String[] args) {
        SomeRunnable launch = new SomeRunnable();
        launch.run();
    }
}
```


### thread
The traditional way to turn a `Runnable` object into a working task is to hand it to a `Thread` constructor.

Calling a `Thread` object's `start()` will perform the necessary initialization for the thread and then call that `Runnable`'s `run()` method to start the task in the new thread.

```java
public class BasicThreadClass {
    public static void main(String[] args) {
        Thread t = new Thread(new SomeRunnable());
        t.start();
    }
}
```

Each `Thread` "registers" itself so there is actually a reference to it someplace, and the garbage collector can't clean it up until the task exits its run() and dies.


### executor
Executors provide a layer of indirection between a client and the execution of a task; instead of a client executing a task directly, an intermediate object executes the task. Executors allow you to manage the execution of asynchronous tasks without having to explicitly manage the life cycle of threads. `Executors` are the preferred method for starting tasks.

`CachedThreadPool` creates one thread per task

```java
public class ThreadPoolClass {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        // ExecutorService exec = Executors.newFixedThreadPool(5);
        // ExecutorService exec = Executors.newSingleThreadExecutor();
        for(int i = 0; i < 5; i++)
            exec.execute(new LiftOff());
        exec.shutdown();
    }
}
```

The call to `shutdown()` prevents new tasks from being submitted to that Executor. The current thread (in this case, the one driving main()) will continue to run all tasks submitted before shutdown() was called. The program will exit as soon as all the tasks in the Executor finish.


### return value
`Callable`, introduced in Java SE5, is a generic with a type parameter representing the return value from the method `call()` (instead of `run()`), and must be invoked using an `ExecutorService` `submit()` method.

```java
public class TaskWithResult implements Callable<String> {
    public TaskWithResult() {
    }
    public String call() {
        return "result";
    }
}

public class CallableDemo {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        ArrayList<Future<String>> results = new ArrayList<Future<String>>();
        for(int i = 0; i < 10; i++)
            results.add(exec.submit(new TaskWithResult(i)));
        for(Future<String> fs : results)
        try {
            // get() blocks until completion:
            System.out.println(fs.get());
        } catch(InterruptedException e) {
            System.out.println(e);
            return;
            } catch(ExecutionException e) {
                System.out.println(e);
                } finally {
                    exec.shutdown();
                }
            }
        }
```


### sleeping
By calling `sleep()` , you can cease/block the execution of that task for a given time.

```java
try {
    while(countDown-- > 0) {
        System.out.print(status());
        // Old-style:
        // Thread.sleep(100);
        // Java SE5/6-style:
        TimeUnit.MILLISECONDS.sleep(100);
    }
} catch(InterruptedException e) {
    System.err.println("Interrupted");
}
```


### priority
The priority of a thread conveys the importance of a thread to the scheduler.

```java
public class SomeRunnable implements Runnable {
    public SomeRunnable() {}
    public void run() {
        Thread.currentThread().setPriority(Thread.MIN_PRIORITY);
        while(countDown-- > 0) {
            System.out.print(status());
            Thread.yield();
        }
    }
}
```

The only portable approach is to stick to `MAX_PRIORITY`, `NORM_PRIORITY`, and `MIN_PRIORITY` when you're adjusting priority levels.


### yielding
When you call `yield()`, you are suggesting that other threads of the same priority might be run.

The call to the static method `Thread.yield()` inside `run()` is a suggestion to the thread scheduler (the part of the Java threading mechanism that moves the CPU from one thread to the next) that says, "I've done the important parts of my cycle and this would be a good time to switch to another task for a while."

It's completely optional.


### daemon
A "daemon" thread is intended to provide a general service in the background as long as the program is running, but is not part of the essence of the program. Thus, when all of the **non-daemon** threads complete, the program is terminated, killing all daemon threads in the process.

```java
public class c implements Runnable {
    public SomeRunnable() {}
    public void run() {
        ...
    }
}

public class MainThread {
    public static void main(String[] args) {
        for(int i = 0; i < 10; i++) {
            Thread daemon = new Thread(new SomeRunnable());
            daemon.setDaemon(true); // Must call before start()
            daemon.start();
        }
    }
}
```


It is possible to customize the attributes (daemon, priority, name) of threads created by `Executors` by writing a custom `ThreadFactory`. You can now pass a new DaemonThreadFactory as an argument to
`Executors.newCachedThreadPool()`.

```java
public class DaemonThreadFactory implements ThreadFactory {
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setDaemon(true);
        return t;
    }
}

public class DaemonFromFactory implements Runnable {
    public void run() {
        ...
    }

    public static void main(String[] args) throws Exception {
        ExecutorService exec = Executors.newCachedThreadPool(new DaemonThreadFactory());
        for(int i = 0; i < 10; i++)
            exec.execute(new DaemonFromFactory());
    }
}
```

You can find out if a thread is a daemon by calling `isDaemon()`. If a thread is a daemon, then any threads it creates will automatically be daemons.

Daemon threads will terminate their `run()` methods without
executing finally clauses.


### joining

One thread may call `join()` on another thread to wait for the second thread to complete before proceeding. If a thread calls `t.join()` on another thread `t`, then the calling thread is
suspended until the target thread t finishes (when `t.isAlive()` is `false`).


### exception
Because of the nature of threads, you can't catch an exception that has escaped from a thread.

Exceptions won't propagate across threads back to main(), you must locally handle any exceptions that arise within a task.

`Thread.UncaughtExceptionHandler` is a new interface in Java SE5; it allows you to attach an exception handler to each `Thread` object. `Thread.UncaughtExceptionHandler.uncaughtException()` is automatically called when that thread is about to die from an uncaught exception.



## Sharing resource

### mutex
To solve the problem of thread collision, virtually all concurrency schemes serialize access to shared resources. This means that only one task at a time is allowed to access the shared resource.

This is ordinarily accomplished by putting a clause around a piece of code that only allows one task at a time to pass through that piece of code. Because this clause produces mutual exclusion, a common name for such a mechanism is mutex.

### synchronized method
To prevent collisions over resources, Java has built-in support in the form of the `synchronized` keyword. When a task wishes to execute a piece of code guarded by the `synchronized` keyword, it checks to see if the lock is available, then acquires it, executes the code, and releases it.

You can prevent collisions by declaring methods synchronized. For fields, you should make the data elements of a class private and access that memory only through methods.

All objects automatically contain a single lock (also referred to as a *monitor*) shared by all the synchronized methods of the particular object. When you call any synchronized method, that object is locked and no other synchronized method of that object can be called until the first one finishes and releases the lock.

**Every method that accesses a critical shared resource must be synchronized or it won't work right.**

Apply Brian's Rule of Synchronization:
> If you are writing a variable that might next be read by another thread, or reading a variable that might have last been written by another thread, you must use synchronization, and further, both the reader and the writer must synchronize using the same monitor lock.


### synchronized code block
Sometimes, you only want to prevent multiple thread access to part of the code inside a method instead of the entire method. The section of code you want to isolate this way is called a *critical section* and is created using the `synchronized` keyword. Here, `synchronized` is used to specify the object whose lock is being used to synchronize the enclosed code.


### explicit lock
The explicit Lock object also gives you finer-grained control over locking and unlocking than does the built-in synchronized lock. This is useful for implementing specialized synchronization structures, such as hand-overhand locking (also called lock coupling), used for traversing the nodes of a linked list—the traversal code must capture the lock of the next node before it releases the current node's lock.

```java
class X {
    private final ReentrantLock lock = new ReentrantLock();
    // ...

    public void m() {
        lock.lock();  // block until condition holds
        try {
            // ... method body
        } finally {
            lock.unlock()
        }
    }
}
```


### atomicity and volatility
*Atomicity* applies to "simple operations" on primitive types except for longs and doubles.

Changes made by one task, even if they're atomic in the sense of not being interruptible, might not be *visible* to other tasks (the changes might be temporarily stored in a local processor cache, for example), so different tasks will have a different view of the application's state.

The `volatile` keyword also ensures visibility across the application. If you declare a field to be `volatile`, this means that as soon as a write occurs for that field, all reads will see the change. This is true even if local caches are involved—volatile fields are immediately written through to main memory, and reads occur from main memory.

Volatile doesn't work when the value of a field depends on its previous value (such as incrementing a counter), nor does it work on fields whose values are constrained by the values of other fields.

**It's important to understand that atomicity and volatility are distinct concepts.**



## Interrupting

### states
A thread can be in any one of four states:
1. New: A thread remains in this state only momentarily, as it is being created. It allocates any necessary system resources and performs initialization. At this point it becomes eligible to receive CPU time. The scheduler will then transition this thread to the runnable or blocked state.

2. Runnable: This means that a thread can be run when the time-slicing mechanism has CPU cycles available for the thread. Thus, the thread might or might not be running at any moment, but there's nothing to prevent it from being run if the scheduler can arrange it. That is, it's not dead or blocked.

3. Blocked: The thread can be run, but something prevents it. While a thread is in the blocked state, the scheduler will simply skip it and not give it any CPU time. Until a thread reenters the runnable state, it won't perform any operations.

        sleep()
        wait()
        blocking IO
        synchronized method/block

4. Dead: A thread in the dead or terminated state is no longer schedulable and will not receive any CPU time. Its task is completed, and it is no longer runnable. One way for a task to die is by returning from its run() method, but a task's thread can also be interrupted, as you'll see shortly.


### shutdown
```java
public class OrnamentalGarden {
    public static void main(String[] args) throws Exception {
        ExecutorService exec = Executors.newCachedThreadPool();
        for(int i = 0; i < 5; i++)
            exec.execute(new Entrance(i));
        // Run for a while, then stop and collect the data:
        TimeUnit.SECONDS.sleep(3);
        Entrance.cancel();
        exec.shutdown();
        if(!exec.awaitTermination(250, TimeUnit.MILLISECONDS))
            print("Some tasks were not terminated!");
        print("Total: " + Entrance.getTotalCount());
        print("Sum of Entrances: " + Entrance.sumEntrances());
    }
}
```


### interruption

    Executor:shutdownNow() or Future:cancel()
    -> interrupt()
    -> throw InterruptedException


>SleepBlock is an example of interruptible blocking, whereas IOBlocked and SynchronizedBlocked are uninterruptible blocking.

> A heavy-handed but sometimes effective solution to this problem is to close the underlying resource on which the task is blocked

> Blocked `nio` channels automatically respond to interrupts.


### wait
`wait(pause)` suspends the task while waiting for the world to change, and only when a `notify()` or `notifyAll()` occurs—suggesting that something of interest may have happened—does the task wake up and check for changes. Thus, `wait()` provides a way to synchronize activities between tasks.

1. The object lock is released during the `wait()`.
2. You can also come out of the `wait()` due to a `notify()` or `notifyAll()`, in addition to letting the clock run out.

These methods are part of the base class `Object` and not part of `Thread`.
In fact, the only place you can call `wait()`,
`notify()`, or `notifyAll()` is within a synchronized method or block:
```java
synchronized(x) {
    x.notifyAll();
}
```

It's safer to call `notifyAll()`, which wakes up all the tasks waiting on that lock.

In order for the task to wake up from a `wait()`, it must first reacquire the lock that it released when it entered the `wait()`. The task will not wake up until that lock becomes available.

You must surround a `wait()` with a while loop that checks the condition(s) of interest. This is important.

```java
synchronized(sharedMonitor) {
    while(someCondition)
        sharedMonitor.wait();
}
```


### signal
The basic class that uses a mutex and allows task suspension is the `Condition`, and you can suspend a task by calling `await()` on a
`Condition`.

When external state changes take place that might mean that a task should continue processing, you notify the task by calling `signal()`, to wake up one task, or `signalAll()`, to wake up all tasks that have suspended themselves on that Condition object
(as with `notifyAll()`, `signalAll()` is the safer approach).


### use `BlockingQueue` or `Pipe`

All Known Implementing Classes:

    ArrayBlockingQueue
    DelayQueue
    LinkedBlockingDeque
    LinkedBlockingQueue
    LinkedTransferQueue
    PriorityBlockingQueue
    SynchronousQueue


## Deadlock
The dining philosophers problem is interesting precisely because it demonstrates that a program can appear to run correctly but actually be able to deadlock.

To repair the problem, you must understand that deadlock can occur if four conditions are simultaneously met:

1. Mutual exclusion. At least one resource used by the tasks must not be shareable. In this case, a Chopstick can be used by only one Philosopher at a time.
2. At least one task must be holding a resource and waiting to acquire a resource currently held by another task. That is, for deadlock to occur, a Philosopher must be holding one Chopstick and waiting for another one.
3. A resource cannot be preemptively taken away from a task. Tasks only release resources as a normal event. Our Philosophers are polite and they don't grab Chopsticks from other Philosophers.
4. A circular wait can happen, whereby a task waits on a resource held by another task, which in turn is waiting on a resource held by another task, and so on, until one of the tasks is waiting on a resource held by the first task, thus gridlocking everything. In DeadlockingDiningPhilosophers.java, the circular wait happens because each Philosopher tries to get the right Chopstick first and then the left.



## Summary

The goal of this chapter was to give you the foundations of concurrent programming with Java threads, so that you understand that:

1. You can run multiple independent tasks.
2. You must consider all the possible problems when these tasks shut down.
3. Tasks can interfere with each other over shared resources. The mutex (lock) is the basic tool used to prevent these collisions.
4. Tasks can deadlock if they are not carefully designed.


The main drawbacks to multithreading are:

1. Slowdown occurs while threads are waiting for shared resources.
2. Additional CPU overhead is required to manage threads.
3. Unrewarded complexity arises from poor design decisions.
4. Opportunities are created for pathologies such as starving, racing, deadlock, and livelock (multiple threads working individual tasks that the ensemble can't finish).
5. Inconsistencies occur across platforms. For instance, while developing some of the examples for this book, I discovered race conditions that quickly appeared on some computers but that wouldn't appear on others. If you develop a program on the latter, you might get badly surprised when you distribute it.


