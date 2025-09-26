---
title: "Summary: Joe Armstrong's Thesis: Erlang"
toc: true
date: "2021-11-15"
last_modified_at: 2025-05-19T00:00:01-00:00
tags: 
  - "actor-model"
  - "distributed-systems"
  - "erlang"
  - "fault-tolerance"
  - "joe-armstrong"
  - "joe-armstrongs-thesis"
  - "technical-papers"
  - "technical-reading"
  - Tech
---

If my [earlier post about Joe Armstrong's Thesis](https://atul-atul.github.io/notes/reading/takeaways-from-joe-armstrongs-thesis/) was a long read, here is an attempt at a shorter version of my notes of [Joe Armstrong's thesis](https://erlang.org/download/armstrong_thesis_2003.pdf) on **Making reliable distributed systems in the presence of software errors**. This post is still a little long read. But in my experience, papers talk a lot of conceptual things and are not easy to condense.

The essential problem in making a fault-tolerant software system is that of fault-isolation. To provide fault isolation we use the notion of a process. Different programmers write different applications which are run in different processes. And errors in one application should not have a negative influence on the other applications running in the system.

The real world is concurrent. In the real world sequential activities are a rarity.

#### Characteristics of a COPL (Concurrency Oriented Programming Language)

1. COPLs must support processes. A process can be thought of as a self-contained virtual machine.
2. Several processes operating on the same machine must be strongly isolated. A fault in one process should not adversely effect another process, unless such interaction is explicitly programmed.
3. Each process must be identified by a unique unforgeable process identifier (Pid).
4. There should be no shared state between processes. Processes interact by sending messages. If you know the Pid of a process then you can send a message to the process.
5. Message passing is assumed to be unreliable with no guarantee of delivery.
6. It should be possible for one process to detect failure in another process and reason for the failure.

#### Isolation

Two processes operating on the same machine must be as independent as if they ran on physically separated machines.

#### Message Passing

1. Message passing is atomic. A message is either delivered in its entirety or not at all.
2. Message passing between a pair of processes is ordered- meaning that if a sequence of messages is sent and received between any pair of processes then the messages will be received in the same order they were sent. Also, if process A sends a message to B then B can only receive this message at some point in time after A has sent the message. This is known as casual ordering.
3. Messages should not contain pointers to data structures contained within processes- they should only contain constants and/or Pids. (No pass by reference.)

#### The Erlang view of the world

1. Everything is a process.
2. Processes are strongly isolated.
3. Process creation and destruction is a lightweight operation.
4. Message passing is the only way for processes to interact.
5. Processes have unique names.
6. If you know the name of a process you can send it a message.
7. Processes share no resources.
8. Error handling is non-local.
9. Processes do what they are supposed to do or fail.

#### Concurrency in Erlang

When a message arrives at a process it is put into a mailbox belonging to that process. The next time the process will try to match the first item in the mailbox with the set of acceptable patterns. If no message matches then the received message is moved to a temporary “save” queue and the process suspends and waits for the next message. If the message matches, then the sequence of statements following the match are evaluated. At the same time, any saved messages are put back into the input mailbox of the process.

#### Abstracting out concurrency

The generic component should hide details of concurrency and mechanisms for fault-tolerance from the plugins. The plugins should be written using only sequential code with well-defined types.

Erlang systems are structured into generic component (server) and plugins (clients). The generic component can provide a rich environment in which to execute the plugin.

The plugin code itself can have errors, and the code in the plug-in can be dynamically modified, the entire component can be moved in a network, and all this occurs without any explicit programming in the plugin code.

Abstracting out concurrency is one of the most powerful means available for structuring large software systems. Despite the ease with which concurrent programs can be written in Erlang it is still desirable to restrict code which explicitly handles concurrency to as few modules as possible.

The most common abstraction used by applications written in Erlang is the client-server abstraction. We assume that several different clients want to share a common resource and that the server is responsible for managing the resource.

Benefits of abstracting out concurrency (for example in client-server):

1. The code in the server can be re-used to build many different client- server applications.
2. The application code is much simpler than the server code.
3. To write the server code the programmer must understand all the details of Erlang’s concurrency model. This involves name registration, process spawning, untrappable exits to a process, and sending and receiving messages. To trap an exception the programmer must understand the notion of an exception and be familiar with Erlang’s exception handling mechanisms.
4. To write the application, the programmer only has to understand how to write a simple sequential program- they need to know nothing about concurrency or error handling.
5. The different servers (server1, server2 etc) imbue the application with different non-functional characteristics. The functional characteristics for all servers are the same (that it, given correctly typed arguments all programs will eventually produce identical results); but the non-functional characteristics are different.
6. The code which implements the non-functional parts of the system is limited to the server (by non-function we mean things like how the system behaves in the presence of errors, how long time function evaluation takes, etc.) and is hidden from the application programmer.
7. The details of how the remote procedure call is implemented are hidden inside the server module. This means that the implementation could be changed at a later stage without changing the client code, should this become necessary.

#### Error handling philosophy

1. Let some other process do the error recovery.
2. If you can’t do what you want to do, let it crash.
3. Do not program defensively.

Remote handling of error has several advantages:

1. The error-handling code and the code which has the error execute within different threads of control.
2. The application code is not cluttered up with the code which handles the exception.
3. The method works in a distributed system and so porting code from a single-node system to a distributed system needs little change to the error-handling code.
4. Systems can be built and tested on a single node system, but deployed on a multi-node distributed system without massive changes to the code.

We build our applications in two parts: a part that solves the application problem and a part that corrects errors if they have occurred.

1. The application code is written with as little defensive code as possible; we assume that all arguments to functions are correct and the programs will execute without errors.
2. The part that corrects errors is often generic, so the same error-correcting code can be used for many different applications. For example, in database transactions if something goes wrong in the middle of a transaction, we simply abort the transaction and let the system restore the database to the state it was in before the error occurred.

One process, the **worker process**, does the job. Another process, the **supervisor process** observes the worker. If an error occurs in the worker, the supervisor takes actions to correct the error. The nice thing about this approach is that:

1. There is a clean separation of issues. The worker processes don't worry about error handling.
2. We can have special processes which are only concerned with error handling.
3. We can run the workers and supervisors on different physical machines.
4. It often turns out that the error correcting code is generic, that is, generally applicable to many applications, whereas the worker code is more often application specific.

When one process in the system dies we would like other processes in the system to be informed. There are two ways of doing this. We can use either a process link or a process monitor.

**Process links** are used to group together sets of processes in such a way that if an error occurs in any one of the processes then all the processes in the group get killed.

**Process monitors** allow individual processes to monitor other processes in the system.

Monitors are used when you want asymmetry in error handling; links are used when you want symmetric error handling. Monitors are typically used by servers to monitor the behavior of clients.

To make a system fault-tolerant we organize the software into a **hierarchy of tasks**. The highest level task is to run the application according to some specification. If this task cannot be performed then the system will try to perform some simpler task. If the simpler task cannot be performed then the system will try to perform an even simpler task and so on. If the lowest level task cannot be performed then the system will fail.

#### Supervision hierarchies

The basic idea is:

1. Try to perform a task.
2. If you cannot perform the task, then try to perform a simpler task.

To each task we associate a _supervisor process_—the supervisor will assign a _worker_ to try and achieve the goals implied by the task. If the worker process fails with a non-normal exit then the supervisor will assume that the task has failed and will initiate some error recovery procedure. The error recovery procedure might be to restart the worker or failing this try to do something simpler.

Supervisors and workers are arranged into hierarchical trees, according to the following:

1. _Supervision trees_ are trees of _Supervisors_.
2. _Supervisors_ monitor _Workers_ and _Supervisors_.
3. _Workers_ are instances of _Behaviors_.
4. _Behaviors_ are parameterized by _Well-behaved functions_.
5. _Well-behaved functions_ raise exceptions when errors occur.
