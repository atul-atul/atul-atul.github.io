---
title: "Zookeeper Paper"
last_modified_at: 2024-05-12T00:00:01-00:00
categories: 
  - "books"
tags: 
  - "distributed-systems"
  - "Zookeeper-paper"
  - "technical-papers"
  - "technical-reading"
  - "technology-books"
---
Here is my understanding about ZooKeeper from reading [the ZooKeeper paper](https://github.com/papers-we-love/papers-we-love/blob/main/distributed_systems/zookeeper-wait-free-coordination-for-internet-scale-systems.pdf). Some other references: [MIT's lecture video](https://www.youtube.com/watch?v=pbmyrNjzdDk) and [course notes](http://nil.csail.mit.edu/6.824/2021/notes/l-zookeeper.txt).

The ideas relevant in this description include: distributed applications, group membership, group messaging, broadcast, quorum, leader election, distributed locks, consistency, performance, hierarchical file system, FIFO, linearizability, write-ahead-log, etc. (Some of these topics are so ubiquitous in distributed systems that it'd be worthwhile for me to write about these in short for quick reference.)

They identify Zookeper (ZK) as a co-ordination service. What does it co-ordinate? Processes of distributed applications. 

A typical workload of a ZK application is dominated by read operations and it becomes desirable to scale read throughput. 

Clients submit asynchronous (wait-free) requests to ZK. All requests (read or write) *from a client* are FIFO and all write requests are linearizable. 

To achieve this, write/ update requests must be submitted to leader but ZK allows clients to submit read requests to local servers (followers). This yields greater performance when read:write request ratio is higher. And also adding followers increases throughput. However, if the read:write ratio is small (more write requests) then the performance suffers and adding more followers does not help as much. Allowing read requests to bypass the leader (by directly querying a follower) results in weak consistency guarantees. A workaround to ensure strong consistency is to for the client to submit a *sync* request before read. But then performance is impacted. So it's recommeded only when the client must read consistent data.

Per client FIFO request gurantee is achieved via a (what the paper calls) pipeline architecture (implemented using write-ahead-log, zxid, etc.). This FIFO gurantee means that a client doen't have to wait for previous request/ operation to be completed before submitting another request. Hence asynch. Also, because of the per client FIFO guarantee, the property of read-your-own-write is fulfilled for any given client. 

In ZK, servers process read operations locally, and are not totally ordered. 

To guarantee that update operations satisfy linearizability, ZK implements a leader-based atomic broadcast protocol, called Zab. The writes in a quorum are commited via a simple majority. Say if the cluster has n machines then at least (1+(n/2)) should be ok to commit the update operation. Having simple majority while commiting updates may lead to weakly consistent states. Say in a cluster of 10 servers, only 6 might have the updated value. A subsequent read request for the same data may be sent to one of the remaining four servers (as it doesn't have to go via master) and the client may read stale data. Thus weak/ relaxed consistency.

**Znodes**
ZK resembles a hierarchical file system. Individual nodes (znodes) map to abstractions of the client application, typically corresponding to config/ meta-data used for coordination purposes. A znode has a version. Clients check version before updating znode. 
![ZK znodes](/images/zk_znodes.png "ZK znodes")

A znode can be of type *Regular* or *Ephimeral* each with or without *Sequential* flag. Nodes created with the sequential flag set have the value of a monotonically increasing counter appended to its name (like p_1, p_2 in the figure). If n is the new znode and p is the parent znode, then the sequence value of n is never smaller than the value in the name of any other sequential znode ever created (previously) under p. So in the figure if we create a new sequential (regular or ephimeral) child znode under /app1 and name the new node as background_task then the new znode will be identified as /app1/background_task_4 as the sequence counter had reached upto 3 (for p_3) under /app1. Another node created under p_3 with name xyz will be accessible as /app1/p_3/xyz_5.

**Watches**
ZooKeeper implements watches to allow clients to receive timely notifications of changes without requiring polling. When a client issues a read operation with a watch flag set, the operation completes as normal except that the server promises to notify the client when the information returned has changed. Watches are one-time triggers associated with a session; they are unregistered once triggered or the session closes. Watches indicate that a change has happened, but do not provide the change.

**Sessions**
A client connects to ZooKeeper and initiates a session. Sessions have an associated timeout. Sessions enable a client to move transparently from one server to another within a ZooKeeper ensemble, and hence persist across ZooKeeper servers. This is achieved via zxid, versions, and watches.

**Client API**
While ZK is more of a service (co-ordination as a service) than a library, it provides a client library which makes it easy for developers to submit request, manage connections, etc. ZK service provides APIs and developers can implement locking, caching, etc. at the client end using these APIs.

All methods have both a synchronous and an async version available through the API. The ZooKeeper client guarantees that the corresponding callbacks for each operation are invoked in order.
```
  create(path, data, flags)
    exclusive(atomic)- only first create indicates success. 
  delete(path, version)
    if znode.version = version, then delete
  exists(path, watch)
    watch=true means also send notification if path is later created/ deleted
  getData(path, watch)
  setData(path, data, version)
    if znode.version = version, then update
  getChildren(path, watch)
  sync()
    sync then read ensures that any writes before sync are flushed and visible to same client's read
    client could instead submit a write (as writes are guaranteed to be Linearizable)
```

**Ordering guarantees**
A new leader can designate a path as the *ready* znode; other processes will only use the configuration when that znode exists. The new leader makes the configuration change by first deleting ready file, updating the various configuration znodes, and creating a new ready file. Because of the ordering guarantees, if a process sees the ready znode, it must also see all the configuration changes made by the new leader. If the new leader dies before the ready znode is created, the other processes know that the configuration has not been finalized and do not use it.

What happens if a process sees that *ready* exists before the new leader starts to make a change and then the process starts reading the configuration while the change initiated by the leader is in progress. This problem is solved by the ordering guarantee for the notifications: if a client is watching for a change, the client will see the notification event (that leader has deleted ready file and initiated some changes) before it sees the new state of the system after the change is made.

  * *Linearizable writes*
    clients (one or more) send writes to the leader
    the leader chooses an order, numbered by "zxid"
    sends to replicas, which all execute in zxid order
  * *FIFO client order*
    each client specifies an order for its operations (reads AND writes) by sending requests one after another
    writes:
      writes appear in the write order in client-specified order
      this is the business about the "ready" file in section 2.3
    reads:
      each read executes at a particular point in the write order (when the ready file exists)
      a client's successive reads execute at non-decreasing points in the order
      a client's read executes after all previous writes by that client
        a server may block a client's read to wait for previous write, or sync()

  Read order rules help reasoning.
    e.g. if read sees "ready" file, subsequent reads see previous writes.
         (Section 2.3)
         Write order:      Read order:
         delete("ready")
         write f1
         write f2
         create("ready")
                           exists("ready")
                           read f1
                           read f2
         even if client switches servers!

    e.g. watch triggered by a write is guaranteed to be delivered before reads from subsequent writes.
         Write order:      Read order:
                           exists("ready", watch=true)
                           read f1
         delete("ready")
         write f1
         write f2
                           read f2


A few consequences/ constraints:
  Leader must preserve client write order across leader failure.
  Replicas must enforce "a client's reads never go backwards in zxid order"
    despite replica failure.
  Client must track highest zxid it has read
    to help ensure next read doesn't go backwards
    even if sent to a different replica

**Locks**
If many clients set watch, there's a herd effect when the previously held lock gets released- many clients will want to acquire lock at once. Psuedocode for simple locking to avoid the herd effect could look like:
Lock
1 n = create(l + “/lock-”, EPHEMERAL|SEQUENTIAL) 
2 C = getChildren(l, false)
3 if n is lowest znode in C, exit (the client acquires the lock)
4 p = znode in C ordered just before n
5 if exists(p, true) wait for watch event 
6 goto 2
Unlock
1 delete(n)

The use of the SEQUENTIAL flag in line 1 of Lock orders the client’s attempt to acquire the lock with respect to all other attempts. If the client’s znode has the lowest sequence number at line 3, the client holds the lock. Otherwise, the client waits for deletion of the znode that either has the lock or will receive the lock before this client’s znode. By only watching the znode that precedes the client’s znode, we avoid the herd effect by only waking up one client process when a lock is released or a lock request is abandoned. What happens if node p (preceding client znode n) gets deleted? A notification is triggered and the client checks if it can now acquire a lock. The check helps in scenarios where the node p may not have had a lock but may have been removed because of missing heartbeat/ network/ hardware failure, etc.

**Read-Write Locks**
Write Lock
1 n = create(l + “/write-”, EPHEMERAL|SEQUENTIAL) 
2 C = getChildren(l, false)
3 if n is lowest znode in C, exit
4 p = znode in C ordered just before n
5 if exists(p, true) wait for event 
6 goto 2
Read Lock
1 n = create(l + “/read-”, EPHEMERAL|SEQUENTIAL)
2 C = getChildren(l, false)
3 if no write znodes lower than n in C, exit
4 p = write znode in C ordered just before n
5 if exists(p, true) wait for event
6 goto 3


**ZK service**
Components:
![ZK service components](/images/zk_service_components.png "ZK service components")

Because state changes depend on the application of previous state changes, Zab provides stronger order guarantees than regular atomic broadcast. More specifically, Zab guarantees that changes broadcast by a leader are delivered in the order they were sent and all changes from previous leaders are delivered to an established leader before it broadcasts its own changes.

Read requests are handled locally at each server. Each read request is processed and tagged with a zxid that corresponds to the last transaction seen by the server. This zxid defines the partial order of the read requests with respect to the write requests. Processing reads locally (an in-memory operation on the local server), yeilds excellent performance with read-dominant workloads.

There are two reasons for write requests taking longer than read requests. First, write requests must go through atomic broadcast, which requires some extra processing and adds latency to requests. The other reason for longer processing of write requests is that servers must ensure that transactions are logged to non-volatile store before sending acknowledgments back to the leader.

General takeaways:
Finished reading this paper quite fast. Also, this is the first time I watched videos related to the paper before reading the paper. Maybe that's a reason behind finishing the paper fast. Also, I hadn't read any paper for a while even though I wanted to. And that was getting on my nerves. And being familiar with some concepts beforehand helped like broadcast, linearizability. I know they say that anybody can program even if they don't have computer science education background. But I am sure if I had an educational background in CS, it would have helped me a lot.
