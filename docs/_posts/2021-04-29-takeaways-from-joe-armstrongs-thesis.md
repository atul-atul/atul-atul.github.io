---
title: "Takeaways from Joe Armstrong’s thesis"
toc: true
date: "2021-04-29"
categories: 
  - "notes"
tags: 
  - "actor-model"
  - "beam"
  - "distributed-systems"
  - "elixir"
  - "erlang"
  - "error-handling"
  - "fault-tolerance"
  - "functional-programming"
  - "generic-server"
  - "joe-armstrong"
  - "joe-armstrong-thesis"
  - "otp"
  - "system-design"
  - "technical-papers"
---

I recently finished reading [Joe Armstrong's thesis](https://erlang.org/download/armstrong_thesis_2003.pdf). **Making reliable distributed systems in the presence of software errors** (2003). To quote: "The central problem addressed by this thesis is the problem of constructing reliable systems from programs which may themselves contain errors. Constructing such systems imposes a number of requirements on any programming language that is to be used for the construction. I discuss these language requirements, and show how they are satisfied by Erlang."

\*\*\*\*\*\*\*

This is a fairly involved topic and the post is quite long. I have also written a summary which also is long but somewhat shorter than this post. If you prefer the shorter version it is available [here](https://abjtechblog.wordpress.com/2021/11/15/summary-joe-armstrongs-thesis-erlang/).

\*\*\*\*\*\*\*

A few impressions:

I learned a lot of things from this. I have read a book and programmed in elixir before (textbook problems) and also I have gone through the functional programming using Erlang course on futurelearn/ University of Kent's Erlang Masterclass ([https://www.youtube.com/playlist?list=PLlML6SMLMRgCaVx42utIleC2aerD504qj](https://www.youtube.com/playlist?list=PLlML6SMLMRgCaVx42utIleC2aerD504qj)). The course had a few sessions by Joe himself and while others videos were good, Joe's videos were great in terms of new-to-me concepts. So Erlang concepts were not strange to me- at least some of them. But still I gathered a few things from the thesis. I had found this thesis listed on Alan Kay's reading list (quora). So I read it.

The first thing that is immediately apparent is the simplicity of the model or approach. Threads, locks, synchronization from OOP world are so difficult to master.

1. **Introduction**\- Not much to note
2. **The Architecture Model**

_An architecture is the set of significant decisions about the organization of a software system, the selection of the structural elements and their interfaces by which the system is composed, together with their behaviour as specified in the collaborations among those elements, the composition of these structural and behavioural elements into progressively larger subsystems, and the architectural style that guides this organization- these elements and their interfaces, their collaborations, and their composition._ - Booch, Rumbaugh, and Jacobson

To make a fault-tolerant software system which behaves reasonably in the presence of software errors we proceed as follows:

1. We organize the software into a hierarchy of tasks... Tasks are ordered by complexity. The top level task is the most complex, when all the goals in the top level task can be achieved then the system should function perfectly. Lower level tasks should still allow the system to function in an acceptable manner, though it may offer a reduced level of service. The goals of a lower level task should be easier to achieve than the goals of a higher level task in the system.
2. Perform the top level task.
3. If an error is detected when trying to achieve a goal, we make an attempt to correct the error. If we cannot correct the error we immediately abort the current task and start performing a simpler task.

Programming a _hierarchy of tasks_ needs a strong encapsulation method. We need strong encapsulation for error isolation. Two processes operating on the same machine must be as independent as if they ran on physically separated machines

The essential problem that must be solved in making a fault-tolerant software system is therefore that of fault-isolation. To provide fault isolation we use the traditional operating system notion of a process. Different programmers write different applications which are run in different processes; errors in one application should not have a negative influence on the other applications running in the system.

Since all processes use the same CPU, and memory, processes which try to hog the CPU or which try to use excessive memory can negatively affect other processes in the system.

In our system processes, and concurrency, are part of the programming language and are not provided by the host operating system. Our applications are structured using large numbers of communicating parallel processes. We take this approach because it provides an architectural infrastructure, potential efficiency, fault isolation.

**Concurrency Oriented Programming (COP)**

The word concurrency refers to sets of events which happen simultaneously. The real world is concurrent. In the real world sequential activities are a rarity.

**Characteristics of a COPL**

COPLs are characterised by the following six properties:

1. COPLs must support processes. A process can be thought of as a self-contained virtual machine.
2. Several processes must be strongly isolated. A fault in one process should not adversely effect another process, unless such interaction is explicitly programmed.
3. Each process must be identified by a unique unforgeable process identifier (Pid).
4. There should be no shared state between processes. Processes interact by sending messages. If you know the Pid of a process then you can send a message to the process.
5. Message passing is assumed to be unreliable with no guarantee of delivery.
6. It should be possible for one process to detect failure and reason for failure in another process.

**Isolation** has several consequences:

1. Processes have “share nothing” semantics.
2. Message passing is the only way to pass data between processes.
3. Message passing is asynchronous.
4. Since nothing is shared, everything necessary to perform a distributed computation must be copied...The only way to know if a message has been correctly sent is to send a confirmation message back.

We will assume that processes know their own names, and that processes which create other processes know the names of the processes which they have created.

Knowing the name of a process is the key element of security. Since names are unforgeable the system is secure only if we can limit the knowledge of the names of the processes to trusted processes.

**Message passing**

Message passing obeys the following rules:

1. Message passing is atomic- a message is either delivered in its entirety or not at all.
2. Message passing between a pair of processes is ordered- meaning that if a sequence of messages is sent and received between any pair of processes then the messages will be received in the same order they were sent.
3. Messages should not contain pointers to data structures contained within processes- they should only contain constants and/or Pids.

3\. **Erlang**

The Erlang view of the world can be summarized in the following statements:

1. Everything is a process.
2. Processes are strongly isolated.
3. Process creation and destruction is a lightweight operation.
4. Message passing is the only way for processes to interact.
5. Processes have unique names.
6. If you know the name of a process you can send it a message.
7. Processes share no resources.
8. Error handling is non-local.
9. Processes do what they are supposed to do or fail.

**Concurrent programming**

When a message arrives at a process it is put into a mailbox belonging to that process. The next time the process evaluates a receive statement the system will look in the mailbox and try to match the first item in the mailbox with the set of patterns contained in the current receive statement. If no message matches then the received message is moved to a temporary “save” queue and the process suspends and waits for the next message. If the message matches and if any guard test following the matching pattern also matches then the sequence of statements following the match are evaluated. At the same time, any saved messages are put back into the input mailbox of the process.

```
register(Name, Pid)
```

creates a global process, and associates the atom Name with the process identifier Pid. Thereafter, messages sent by evaluating _Name ! Msg_ are sent to the process Pid.

**Process links and monitors**

When one process in the system dies we would like other processes in the system to be informed; recall that we need this in order to be able to program fault-tolerant systems. There are two ways of doing this. We can use either a process link or a process monitor.

Process links are used to group together sets of processes in such a way that if an error occurs in any one of the processes then all the processes in the group get killed.

Process monitors allow individual processes to monitor other processes in the system.

**Process links**

When a process fails the reason for failure is broadcast to all processes which belong in the so-called “link set” of the failing process. A process A can add the process B to its link set by evaluating the Built In Function (BIF) _link(B)_. Links are symmetric, in the sense that if A is linked to B then B will also be linked to A.

Signals are things which are sent between processes when a process terminates. The signal is a tuple of the form _{'EXIT', P, Why}_ where _P_ is the Pid of the process which has terminated and _Why_ is a term describing the reason for termination.

There is one exception to the rule that a system process will convert all signals into messages; evaluating _exit(P, kill)_ sends an un-stoppable exit to the process P which will be terminated. This use of _exit/2_ is needed to kill processes which refuse to honour requests to voluntarily terminate.

Process links are useful for setting up groups of processes, which will all die if anything goes wrong in any of the processes. Usually we just link together all the processes which belong to an application, and let one of the processes assume a “supervisor” role. The supervisor process is set to trap exits. If anything goes wrong then the entire group will die except the supervisor process which can receive messages which inform it about the failures of the other processes in the group.

**Process Monitors**

Process links are useful for entire groups of processes, but not for monitoring pairs of processes in an asymmetric sense. In a typical client-server model, the relationship between the client and the servers is asymmetric as regards error handling. Suppose that a server is involved in a number of long-lived sessions with a number of different clients; if the server crashes, then we want to kill all the clients, but if an individual client crashes we do not wish to kill the server.

**Dynamic code change**

The Erlang system allows for two versions of code for every module. If the code for a particular module has been loaded then all new processes that call any of this code will be dynamically linked with the latest version of the module. If the code for a particular module is subsequently changed then processes which execute code in this module can choose either to continue executing the old code for the module, or to use the new code. The choice is determined by how the code is called.

If the code is called with a fully-qualified name, that is a module name followed by a function name, then the latest version of the module will always be called, otherwise the current version will be called. For example, suppose we write a server loop as follows:

```
-module(m).
...
loop(Data, F) -> 
  receive
    {From, Q} ->
        {Reply, Data1} = F(Q, Data), 
        m:loop(data1, F)   
  end.
```

Note that it is the programmer’s responsibility to ensure that the new code to be called is compatible with the old code. It is also highly advisable that all code replacement calls are tail-calls, since in a tail-call there is no old code to return to, and thus after a tail-call all old code for a module can safely be deleted.

4\. **Programming Techniques**

The programming techniques are concerned with: Abstracting out concurrency, Maintaining the Erlang view of the world, The Erlang view of errors, Intentional programming.

**Abstracting out concurrency**

The generic component should hide details of concurrency and mechanisms for fault-tolerance from the plugins. The plugins should be written using only sequential code with well-defined types.

Structuring a system into generic component and plugins is a common programming technique- In Erlang world, the generic component can provide a rich environment in which to execute the plugin. The plugin code itself can have errors, and the code in the plug-in can be dynamically modified, the entire component can be moved in a network, and all this occurs without any explicit programming in the plugin code.

Abstracting out concurrency is one of the most powerful means available for structuring large software systems. Despite the ease with which concurrent programs can be written in Erlang it is still desirable to restrict code which explicitly handles concurrency to as few modules as possible.

The most common abstraction used by applications written in Erlang is the client-server abstraction. In virtually all systems that are programmed in Erlang, use of the client-server abstraction far outweighs the use of any other abstraction.

The client server model is generally used for resource management operations. We assume that several different clients want to share a common resource and that the server is responsible for managing the resource.

Server failures, should be detected not by client software, but by special supervisor processes which are responsible for correcting the errors introduced by the failure of a server.

Benefits of abstracting out concurrency (for example in client-server):

1. The code in the server can be re-used to build many different client- server applications.
2. The application code is much simpler than the server code.
3. To write the server code the programmer must understand all the details of Erlang’s concurrency model. This involves name registration, process spawning, untrappable exits to a process, and sending and receiving messages. To trap an exception the programmer must understand the notion of an exception and be familiar with Erlang’s exception handling mechanisms.
4. To write the application, the programmer only has to understand how to write a simple sequential program- they need to know nothing about concurrency or error handling.
5. We can imagine using the _same_ application code in a succession of progressively more sophisticated servers. I have shown three such servers but we can imagine adding more and more functions to the server while keeping the server/application interface unchanged.
6. The different servers (server1, server2 etc) imbue the application with different non-functional characteristics. The functional characteristics for all servers are the same (that it, given correctly typed arguments all programs will eventually produce identical results); but the non-functional characteristics are different.
7. The code which implements the non-functional parts of the system is limited to the server (by non-function we mean things like how the system behaves in the presence of errors, how long time function evaluation takes, etc) and is hidden from the application programmer.
8. The details of how the remote procedure call is implemented are hidden inside the server module. This means that the implementation could be changed at a later stage without changing the client code, should this become necessary.

**Maintaining the Erlang view of the world**

The Erlang view of the world is that everything is a process and that processes can only interact by exchanging messages.

When we interface Erlang programs to external software it is often convenient to write an interface program which maintains the illusion that “everything is a process.”

**Error handling philosophy**

1. Let some other process do the error recovery.
2. If you can’t do what you want to do, die.
3. Let it crash.
4. Do not program defensively.

If the process Pid1 fails and if the processes Pid1 and Pid2 are linked together and if process Pid2 is set to trap errors then a message of the form _{’EXIT’, Pid1, Why}_ will be delivered to Pid2 if Pid1 fails. 'Why' describes the reason for failure.

Note also that if the computer on which Pid1 executes dies, then an exit message _{’EXIT’, Pid1, machine\_died}_ will be delivered to Pid2. This message appears to have come from Pid1, but in fact comes from the run-time system of the node where Pid2 was executing.

The reason for coercing the hardware error to make it look like a software error is that we don’t want to have two different methods for dealing with errors, one for software errors and the other for hardware errors. For reasons of conceptual integrity we want one uniform mechanism.

Remote handling of error has several advantages:

1. The error-handling code and the code which has the error execute within different threads of control.
2. The code which solves the problem is not cluttered up with the code which handles the exception.
3. The method works in a distributed system and so porting code from a single-node system to a distributed system needs little change to the error-handling code.
4. Systems can be built and tested on a single node system, but deployed on a multi-node distributed system without massive changes to the code.

**Workers and supervisors**

To make the distinction between processes which perform work, and processes which handle errors clearer we oden talk about worker and supervisor processes:

One process, the worker process, does the job. Another process, the supervisor process observes the worker. If an error occurs in the worker, the supervisor takes actions to correct the error. The nice thing about this approach is that:

1. There is a clean separation of issues. The processes that are supposed to do things (the workers) do not have to worry about error handling.
2. We can have special processes which are only concerned with error handling.
3. We can run the workers and supervisors on different physical machines.
4. It often turns out that the error correcting code is generic, that is, generally applicable to many applications, whereas the worker code is more often application specific.

Point three is crucial- we can run worker and supervisor processes on different physical machines, and thus make a system which tolerates hardware errors where entire processes fail.

**Let it crash**

in the event of an error, then the program should just crash. But what is an error? For programming purpose we can say that:

1. exceptions occur when the run-time system does not know what to do.
2. errors occur when the programmer doesn’t know what to do.

The defensive code detracts from the pure case and confuses the reader- the diagnostic is often no better than the diagnostic which the compiler supplies automatically.

5\. Programming Fault-tolerant Systems

**Programming fault-tolerance**

To make a system fault-tolerant we organize the software into a _**hierarchy of tasks**_ that must be performed. The highest level task is to run the application according to some specification. If this task cannot be performed then the system will try to perform some simpler task. If the simpler task cannot be performed then the system will try to perform an even simpler task and so on. If the lowest level task in the system cannot be performed then the system will fail.

**Supervision hierarchies**

The basic idea is:

1. Try to perform a task.
2. If you cannot perform the task, then try to perform a simpler task.

To each task we associate a _supervisor process_—the supervisor will assign a _worker_ to try and achieve the goals implied by the task. If the worker process fails with a non-normal exit then the supervisor will assume that the task has failed and will initiate some error recovery procedure. The error recovery procedure might be to restart the worker or failing this try to do something simpler.

Supervisors and workers are arranged into hierarchical trees, according to the following:

1. _Supervision trees_ are trees of _Supervisors_.
2. _Supervisors_ monitor _Workers_ and _Supervisors_.
3. _Workers_ are instances of _Behaviours_.
4. _Behaviours_ are parameterized by _Well-behaved functions_.
5. _Well-behaved functions_ raise exceptions when errors occur.

Where:

- Supervision trees are hierarchical trees of supervisors. Each node in the tree is responsible for monitoring errors in its child nodes.
- Supervisors are processes which monitor other processes in the system. The things that are monitored are either supervisors or workers. Supervisors must be able to detect exceptions generated by the things they are monitoring, and be able to start, stop and restart the things they are monitoring.
- Workers are processes which perform tasks. If a worker process terminates with a non-normal exit (see page 74) the supervisor will assume that an error has occurred and will take action to recover from the error. Workers in our model are not arbitrary processes, but are instances of one of a small number of generic processes (called _behaviours_).
- Behaviours are generic processes whose operation is entirely characterized by a small number of callback functions. These functions must be instances of well-behaved functions.

Each supervisor has a SSRS (Start Stop and Restart Spec) for each of its children and obeys the following rules:

1. If my parent stops me then I should stop all my children.
2. If any of my children dies then I must try to restart that child.

**And/or supervision hierarchies**

The rules for a supervisor in an AND/OR tree are as follows:

- If my parent stops me then I should stop all my children.
- If any child dies and I am an AND supervisor stop all my children and restart all my children.
- If any child dies and I am an OR supervisor restart the child that died. AND supervision is used for dependent, or co-ordinated processes. In the AND tree the successful operation of the system depends upon the successful operation of all the children—thus if any child dies all the children should be stopped and then restarted.

**What is an error?**

It is the programmer who decides if an exception corresponds to an error- in our system the programmer must explicitly say which functions in the system are expected to never generate exceptions.

For our purposes we will define an error as a deviation between the observed behaviour of a system and the desired behaviour of a system.

The programmer must ensure that if the system behaves in a way that deviates from the specification, some kind of error recovery procedure is initiated, and that some record of this fact is recorded in a permanent error log, so that it can be corrected later.

**Well-behaved functions**

A well-behaved function (WBF) is a function which should not normally generate an exception. If an exception is generated by the WBF then the exception will be interpreted as an error.

If an exception occurs while trying to evaluate the WBF the WBF should try to correct the condition which caused the exception. If an error occurs within a WBF which cannot be corrected the programmer should terminate the function with an explicit exit statement.

Well-behaved functions are written according to the following rules:

1. The program should be isomorphic to the specification.
2. If the specification doesn’t say what to do, raise an exception.
3. If the generated exceptions do not contain enough information to be able to isolate the error, then add additional helpful information to the exception.
4. Turn non-functional requirements into assertions (invariants) that can be checked at run-time. If the assertion is broken then raise an exception.

6\. Building an Application

Applications which use the OTP software are built from a number of “behaviours.” Behaviours are abstractions of common programming patterns, which can be used as the building blocks for implementing systems in Erlang. The behaviours discussed in the remainder of this chapter are as follows:

1. gen\_server — used to build servers which are used in client- server models.
2. gen\_event — used for building event handlers. Event handlers are things like error loggers, etc. An event handler is something which responds to a stream of events, it does not necessarily reply to the processes which send events to the handler.
3. gen\_fsm — this is used for implementing finite state machines. supervisor — this is used for implementing supervision trees.
4. application — this is used as a container for packaging completed applications.

The chapter discusses how to implement each of these, terminology associated, etc.

**Discussion**

1. In the OTP system the generic modules which implemented the behaviours themselves were written by expert Erlang programmers. These modules are based on several years of experience and represent “best practice” in writing code for the particular problem.
2. Systems built using the OTP behaviours have a very regular structure. For example, all client-servers and supervision trees have an identical structure. The use of the behaviour forces a common structure in the solution of the problem. The applications programmer has to provide that part of the code which defines the semantics of their particular problem. All the infrastructure is provided automatically by the behaviour.
3. It is relatively easy for a new programmer joining an existing team to understand behaviour-based solutions to problems. Once they have gained familiarity with the behaviours they can easily recognize situations where a particular behaviour should be used.
4. Most of the “tricky” systems programming can be hidden within the implementation of the behaviours (which are actually much more complicated than described here). It you look back to the client- server and event handler behaviours you will see that all the code to do with concurrency, message passing etc is isolated in the “generic” part of the behaviour. The “problem specific” code only has pure sequential functions with well-defined types. This is a highly desirable state of affairs-concurrent programs which are “difficult” are isolated to small well-defined parts of the system. The vast majority of the code in the system can be written using sequential programs having well-defined types.

7\. OTP

8\. Case Studies  
Here Joe discusses whether the Erlang ecosystem was helpful and successful in designing fault tolerant systems.

“Clean” code for our purposes is assumed to be side-effect free code, such code is much easier to under- stand than “dirty” code, ie code having side-effects.

The “emergent style” of programming seems therefore, to favour a style of programming where a small number of modules have a large number of side-effects, together with a larger number of modules having very few side-effects.

The Erlang programming rules favour this style of programming, the intention is to get the more experienced programmers to write and test the code that has lots of side-effects.

The client-server abstraction (gen\_server) is so useful that 63% of all generic objects in the system (AXD301) are instances of client-servers.

9\. APIs and Protocols

10\. Conclusions

**Program Development Using Erlang Programming Rules and Conventions**

**Structure and Erlang Terminology**

Erlang systems are divided into **modules**. Modules are composed of **functions** and **attributes**. Functions are either only visible inside a module or they are **exported** i.e. they can also be called by other functions in other modules. Attributes begin with “-” and are placed in the beginning of a module.

The work in a system designed using Erlang is done by **processes**. A process is a job which can use functions in many modules. Processes com- municate with each other by **sending messages**. Processes **receive** messages which are sent to them, a process can decide which messages it is prepared to receive. Other messages are queued until the receiving process is prepared to receive them.

A process can supervise the existence of another process by setting up a **link** to it. When a process terminates, it automatically sends **exit signals** to the process to which it is linked. The default behaviour of a process receiving an exit signal is to terminate and to propagate the signal to its linked processes. A process can change this default behaviour by **trapping exits**, this causes all exit signals sent to a process to be turned into messages.

A **pure function** is a function that returns the same value given the same arguments regardless of the context of the call of the function. This is what we normally expect from a mathematical function. A function that is not pure is said to have **side e****c****ects**.

Side effects typically occur if a function a) sends a message b) receives a message c) calls exit d) calls any BIF which changes a process’s environment or mode of operation (e.g. get1, put2, erase1, process flag2 etc).

**SW Engineering Principles**

1. **Export as few functions as possible from a module**
2. **Try to reduce intermodule dependencies**
3. **Put commonly used code into libraries**. The best library functions have no side effects. Libraries with functions with side effects limit the reusability.
4. **Isolate “tricky” or “dirty” code into separate modules**
5. **Don’t make assumptions about what the caller will do with the results of a function**
6. **Abstract out common patterns of code or behaviour**
7. **Top-down** Write your program using the top-down fashion, not bottom-up (starting with details). Top-down is a nice way of successively approaching details of the implementation, ending up with defining primitive functions. The code will be independent of representation since the representation is not known when the higher levels of code are designed.
8. **Don’t optimize code** at the first stage. First make it right, then (if necessary) make it fast (while keeping it right).
9. **Use the principle of “least astonishment”**
10. **Try to eliminate side effects**
11. **Don’t allow private data structure to “leak” out of a module**
12. **Make code as deterministic as possible**
13. **Do not program “defensively”** A defensive program is one where the programmer does not “trust” the input data to the part of the system they are programming. In general one should not test input data to functions for correctness. Most of the code in the system should be written with the assumption that the input data to the function in question is correct. Only a small part of the code should actually perform any checking of the data. This is usually done when data “enters” the system for the first time, so once data has been checked as it enters the system it should thereafter be assumed correct.
14. **Isolate hardware interfaces with a device driver**
15. **Do and undo things in the same function**. (For example- close file in the same function that is used for opening, reading its content)

**Error Handling**

1. **Separate error handling and normal case code** Don’t clutter code for the “normal case” with code designed to handle exceptions. As far as possible you should only program the normal case. If the code for the normal case fails, the process should report the error and crash as soon as possible. Don’t try to fix up the error and continue. The error should be handled in a different process. Clean separation of error recovery code and normal case code should greatly simplify the overall system design.
2. **Identify the error kernel** One of the basic elements of system design is identifying which part of the system has to be correct and which part of the system does not have to be correct.

**Processes, Servers and Messages**

1. **Implement a process in one module**
2. **Use processes for structuring the system**
3. **Registered processes** Registered processes should be registered with the same name as the module. This makes it easy to find the code for a process. Only register processes that should live a long time.
4. **Assign exactly one parallel process to each true concur- rent activity in the system** “Use one parallel process to model each truly concurrent activity in the real world.”
5. **Each process should only have one “role”** (Supervisor, worker, etc.)
6. **Use generic functions for servers and protocol handlers wherever possible**
7. **Tag messages** All messages should be tagged. This makes the order in the receive statement less important and the implementation of new messages easier.
8. **Flush unknown messages**
9. **Write tail-recursive servers**
10. **Interface functions** Use functions for interfaces whenever possible, avoid sending messages directly. Encapsulate message passing into interface functions.

UBF
