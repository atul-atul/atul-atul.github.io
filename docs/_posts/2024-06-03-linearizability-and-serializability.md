---
title: "Linearizability And Serializability"
date: "2024-06-07"
last_modified_at: 2024-06-07T00:00:01-00:00
tags: 
  - "distributed-systems"
  - "technical-reading"
  - reading
---
We see these two concepts often when there are concurrent reads and writes on shared objects and have some expectations from a well-behaved system. The system does not have to be a database or a distributed system. But let's talk here with those in the background.

#### Linearizability
Linearizability deals with the results of single operations on single objects. Of course, it is about multiple such operations happening on the shared object. But it talks about results of single operations. It is like consistency. Simply, it means that operations happen and complete one after another and can be treated as having finished instantaneously. So the results of a series of operations on an object can be viewed as series of results. And because of instantaneous nature, result of an operation is available for consumption before the subsequent operation begins. Put differently, the sequence of operations can be reasoned about (this operation happened before that; these operations happened after that, etc.) 

From [wikipedia entry](https://en.wikipedia.org/wiki/Linearizability): "In a concurrent system, processes can access a shared object at the same time. Because multiple processes are accessing a single object, a situation may arise in which while one process is accessing the object, another process changes its contents. Making a system linearizable is one solution to this problem. In a linearizable system, although operations overlap on a shared object, each operation appears to take place instantaneously. Linearizability is a strong correctness condition, which constrains what outputs are possible when an object is accessed by multiple processes concurrently. It is a safety property which ensures that operations do not complete unexpectedly or unpredictably. If a system is linearizable it allows a programmer to reason about the system."

A series of operations is linearizable if the operations on an object are totally ordered. The oerations can be written as sequence of operations taking place one after another in real time (physical/ wall clock times). And the results of operations are also in the order. And if there are no writes/ updates this should not matter. But if there are reads and writes then the read operations see preceding writes in the order.

Is the following history linearizable then?
![Is this linearizable history?](/images/linearizability.png "Is this linearizable history?")

I took this from [a lecture video](https://www.youtube.com/watch?v=pbmyrNjzdDk). There are three processes. \|---- means the operation started and ----\| means the operation completed. The operations like WX1 write a value of 1 into object X. And operations like RX3 read the value of X as 3. 

The history can be viewed as linearizable if the server chooses to execute operation WX3 before WX2. As these two are concurrent, the server may choose the order in which to execute. But because RX3 on a process happened before RX2, it means that if the system claims to be linearizable, then write of value 3 into X must have happened before write of value 2. So if the system claims to be lineaziable, we can reason about the operations as if they occurred in the following sequence: WX1, WX3, RX3, WX2, RX2. If no process read value of 1, how can we say that WX1 occurred first? Because it was sent by process P1 before WX2 and server finished that operation. 

--------------------

#### Serializability
Serializabilityon the other hand deals with transactions and which take place over one or more objects. A transactions is a group of operations such that the results of all operations within a transaction form the result of the transaction. So while linearizability deals with a history of operations, serializability is about history of transactions. And while linearizability deals with single shared object, serializability is about multiple shared objects. Seiralizability is more about isolation of one transaction from another. And the serial order in which operations and transactions were submitted can be different from the order in which transactions were run.

Quoting from [a page](http://www.bailis.org/blog/linearizability-versus-serializability/): "Unlike linearizability, serializability does not— by itself— impose any real-time constraints on the ordering of transactions. Serializability is also not composable. Serializability does not imply any kind of deterministic order- it simply requires that some equivalent serial execution exists."

Or from [wiki entry](https://en.wikipedia.org/wiki/Database_transaction_schedule#Duration_and_order_of_actions): 
"Usually, for the purpose of reasoning about concurrency control in databases, an operation is modelled as atomic, occurring at a point in time, without duration. Real executed operations always have some duration.

Operations of transactions in a schedule can interleave (i.e., transactions can be executed concurrently), but time orders between operations in each transaction must remain unchanged. The schedule is in partial order when the operations of transactions in a schedule interleave (i.e., when the schedule is conflict-serializable but not serial). The schedule is in total order when the operations of transactions in a schedule do not interleave (i.e., when the schedule is serial)."

So serializability is a difficult guarantee because in the worst case scenario, any two transactions can be interleaving and then total order needs to be ensured. Instead databases allow certain other [transaction isolation levels](https://learn.microsoft.com/en-us/sql/odbc/reference/develop-app/transaction-isolation-levels) like Read committed, repeatable read, etc.

--------------------
Proper linearizability and serializability can be difficult to achieve. So some co-ordination (ex. Paxos, ZooKeeper, etc.) and trade-offs (ex. transaction isolation levels) may be required.

--------------------
Further reading (TO DO): 
[Linearizability: A Correctness Condition for Concurrent Objects](https://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)

[Spanner](https://github.com/papers-we-love/papers-we-love/blob/main/datastores/spanner-google's-globally-distributed-database.pdf)
