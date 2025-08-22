---
title: "Cassandra Paper"
date: "2024-06-22"
last_modified_at: 2024-06-22T00:00:01-00:00
categories:
  - technology
  - reading
tags: 
  - reading
  - "distributed-systems"
  - "Cassandra-paper"
  - "technical-papers"
  - "technical-reading"
---
Below are some notes from my reading of [Facebook's Cassandra paper](https://www.cs.cornell.edu/projects/ladis2009/papers/lakshman-ladis2009.pdf).

Cassandra was designed at Facebook to fulfill the storage needs of the Inbox Search use case. The current version of [Apache project](https://cassandra.apache.org/_/index.html) is 5.0, and as the ASF website says is used by thousands of companies. TODO: watch [Walmart Cassandra talk](https://aidevcass23.sched.com/event/1Gy6p/embracing-multi-cloud-cassandra-at-walmart-andrew-weaver-patrick-lee-walmart) and go through [documentation](https://cassandra.apache.org/doc/5.0/index.html)

Some terms translated to familiar terms to avoid confusion:
Column family- corresponds to a table, I think.

Cassandra stores high volume structured data having high write throughput. Queries have low latency. The data is non-relational, although transaction capabilities are targeted (TODO read if transactions are supported in newer versions). Horizontal scaling with number of users was also a requirement in the original inbox search use case.

The design builds on typical distributed systems concepts and patterns: write ahead/ commit log, compaction, quorum (sloppy and strict), cluster membership, consistent hashing, partitioning, replication, ZooKeeper, etc.

**API** supports following operations:
```
1. insert(table,key,rowMutation) 
2. get(table,key,columnName)
3. delete(table,key,columnName)
```

#### Data Model 
Instead of modeling the data first and then writing queries (as in RDBMS), with Cassandra we model the queries considering most common query paths the application will use and let the data be organized around them. For example, data model for user inbox search use case could consider a typical query path like a user searching their inbox for messages from one of their friends which contain certain words. 

Typically applications use a dedicated Cassandra cluster and manage them as part of their service. Although the system supports the notion of multiple tables all deployments have only one table in their schema. A table in Cassandra is a distributed multi dimensional map indexed by a string key. The value is an object which is highly structured. Every operation under a single row key is atomic per replica no matter how many columns are being read or written into. Columns are grouped together into sets called column families. Cassandra exposes two kinds of columns families, Simple and Super column families. Super column families can be visualized as a column family within a column family. Applications can specify the sort order of columns within a Super Column or Simple Column family. Any column within a column family is accessed using the convention column family : column and any column within a column family that is of type super is accessed using the convention column family : super column : column


#### Partitioning and Load Balancing 
Cassandra partitions data across the cluster using consistent hashing but uses an order preserving hash function to do so. The basic consistent hashing algorithm presents some challenges. First, the random position assignment of each node on the ring leads to non-uniform data and load distribution. Second, the basic algorithm is oblivious to the heterogeneity in the performance of nodes. Typically there exist two ways to address this issue: One is for nodes to get assigned to multiple positions (vnodes) in the circle, and the second is to analyze load information on the ring and have lightly loaded nodes move on the ring to alleviate heavily loaded nodes. Cassandra opts for the latter as it makes the design and implementation very tractable and helps to make very deterministic choices about load balancing.

#### Replication
Cassandra is configured such that each row is replicated across multiple data centers. Each data item is replicated at N hosts, where N is the replication factor configured per cluster deployment. Each data item identified by a key is assigned to a node by hashing the data item’s key to yield its position on the ring, and then walking the hash-ring clockwise to find the first node with a position larger than the item’s position. This node is deemed the coordinator for this key. (Thus, each node becomes responsible for the region in the ring between it and its predecessor node on the ring.) The coordinator is in charge of the replication of the data items that fall within its range. Cassandra provides various replication policies such as "Rack Unaware", "Rack Aware" (within a data center) and "Datacenter Aware". Replicas are chosen based on the replication policy chosen by the application. Cassandra system elects a leader amongst its nodes using Zookeeper. All nodes on joining the cluster contact the leader who tells them for what ranges they are replicas for and the leader makes a concerted effort to maintain the invariant that no node is responsible for more than N-1 ranges in the ring. Nodes that are responsible for a given range form the "preference list" for the range. The preference list of a key is constructed such that the storage nodes are spread across multiple data centers.

Cassandra provides durability guarantees by relaxing the quorum requirements (they must be using version vectors/ vector clocks for reconciliation, I think).

#### Cluster membership and node failure detection
Rather than a simple heartbeat timeout, Cassandra uses sliding window approach. (Accrual Failure Detection) Every node in the system maintains a sliding window of inter-arrival times of gossip messages from other nodes in the cluster and decides if a node is no longer available. TODO read more about this and Scuttlebutt anti-entropy mechanism.

#### Bootstrapping
Every message contains the cluster name of each Cassandra instance. If a manual error in configuration led to a node trying to join a wrong Cassandra instance it can thwarted based on the cluster name. When a node starts for the first time, it is assigned an UUID token for its position in the ring. For fault tolerance, the mapping is persisted to disk locally and also in Zookeeper. The UUID token information is then gossiped around the cluster. This enables any node to route a request for a key to the correct node in the cluster. In the bootstrap case, when a node needs to join a cluster, it reads its configuration file which contains a list of a few contact points within the cluster. This can also come from a configuration service like Zookeeper. Here is a peculiar approach which I have not seen in many papers: A node outage rarely signifies a permanent departure and therefore should not result in re-balancing of the partition assignment or repair of the unreachable replicas. Cassandra uses an explicit mechanism to initiate the addition and removal of nodes from a Cassandra instance. An administrator (manual) uses a command line tool or a browser to connect to a Cassandra node and issue a membership change to join or leave the cluster. (This is a peculiar approach to mention in paper. But, as I remember it, in the book Release It Michael Nygard mentions a Reddit outage that was caused by automation scripts trying to scale cluster and suggests having manual intervention in such cases.)

When a new node is added into the system (manually), it gets assigned a token such that it can alleviate a heavily loaded node. This results in the new node splitting a range that some other node was previously responsible for. The node giving up the data uses kernel- kernel copy techniques to stream the data over to the new node.

#### Persistence
Typical write operation involves a write into a write ahead log and then an update into an in-memory data structure. When the in-memory data structure crosses a certain threshold, calculated based on data size and number of objects, it dumps itself to disk (data file). All writes are sequential to disk and also generate an index for efficient lookup based on row key. These indices are also persisted along with the data file. Over time many such data files are merged into one file. This process is very similar to the compaction process that happens in the Bigtable system. (If not familiar, [take a quick look at](https://atul-atul.github.io/reading-googles-bigtable-paper/) minor, major compactions of memtable, SSTables, etc.). The Cassandra system indexes all data based on primary key. The data file on disk is broken down into a sequence of blocks. Each block contains at most 128 keys and is demarcated by a block index. The block index captures the relative offset of a key within the block and the size of its data. When an in-memory data structure is dumped to disk a block index is generated and their offsets written out to disk as indices. This index is also maintained in memory for fast access.

#### Reading data
A typical read operation first queries the in-memory data structure before looking into the files on disk. The files are looked at in the order of newest to oldest. When a disk lookup occurs we could be looking up a key in multiple files on disk. In order to prevent lookups into files that do not contain the key, a bloom filter, summarizing the keys in the file, is also stored in each data file and also kept in memory. A key in a column family could have many columns. Some special indexing is required to retrieve columns which are further away from the key. In order to prevent scanning of every column on disk Cassandra maintains column indices which allow to jump to the right chunk on disk for column retrieval.

#### Request Handling
Uses state machine. The state machine morphs through the following states:
1. identify the node(s) that own the data for the key 
2. route the requests to the nodes and wait on the responses to arrive 
3. if the replies do not arrive within a configured timeout value fail the request and return to the client
4. figure out the latest response based on timestamp
5. schedule a repair of the data at any replica if they do not have the latest piece of data.

There are some **trade-offs:**
1. Configuring write ahead log to fast-sync mode (where writes to the log file are buffered) may end up losing data if the node crashes (buffered but not yet persisted data).
2. Requests could be sync or async. Sync mode uses strict quorum before returning response (consistent data or failure). Async writes use sloppy quorum.
3. Some other like block/ file sizes, log rolling strategies, sorting, caching, etc.
Most of these are config level decisions and can differ from application to application.

#### General takeaways
It's a small paper and I finished reading it quite quickly. A lot of concepts are seen elsewhere in distributed systems landscape. Being familiar with some concepts helped. Looking at the latest code was a bit tedious though I did not spend much time there (TODO). The code could have been documented better I think.