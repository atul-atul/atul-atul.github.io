---
title: "LSM Trees, SSTables and MemTables"
toc: true
date: 2026-09-08
last_modified_at: 2026-09-08T00:00:01-00:00
tags: 
  - "distributed-systems"
  - Tech
---

We will discuss LSM Trees.
But first a few basic components:

![Writing To A LSM Tree](/images/writing_to_a_bigtable.png "Writing To A LSM Tree")

## Redo-Log/ Write-Ahead-Log/ Commit-Log 
Any data/ command operation coming into our system is first persisted in a durable way. We write that data into an append only log file (and its backup(s) via a background thread/kernel copy because after all the hard-disk itself can crash). This is generally known as write-ahead-log, redo-log, commit-log, etc. This generally serves as entry point. (I mean apart from cross cutting concerns like rate-limiting, access-control, etc.) This is where data first enters into our system. If any subsequent processing goes wrong we have single point from where we can restart.

## Memtable 
Google's BigTable paper coined this term, I think. And it may be good to say it right away that it's not a table and not even a hash-table. It is a structure (for example, a red-black tree- TODO revisit trees and balancing) that holds data in memory in a sorted order of keys. After it grows to some pre-set limit, it is flushed from memory into a more durable SSTable file. Memtable data will be lost in case of a crash and we may need to begin again with re-do log. Why hold it in memory then? For faster access of course as in case of [BigTable](/reading-googles-bigtable-paper/)

## SSTable
An SSTable (Sorted String Table) is an immutable, ordered data file on disk. It's sorted based on key. It's generally formed after a MemTable is flushed from memory to the file system. We can also get a new SSTable file when merging (and compacting) existing SSTables. Of course flushing MemTable is simple as the MemTable is also sorted. And when merging two SSTables, it is simple merge operation like that in merge sort. SSTables may be compacted (thus reclaiming the space) if we (and this is not necessarily always the case) decide that we don't need old data for same key- for example, in an update operation.

## LSM tree
A Log-Structured-Merge-tree uses above components.
For example, at level 0 in the diagram below (image from wikipedia), is MemTables in memory. At level 1 SSTables, at level 2 merged and compacted SSTables, etc. 

![LSM Tree](/images/LSM_Tree.png "LSM Tree")

## System Performance
Performs well when writing data into the system. No random I/O, writes are appends, sequential and efficient. Also, MemTable flush writes into SSTables are also efficient for same reasons. 

While reading the data is first searched in MemTable based on key; if not found there it is searched in the newest SSTable file, then a SSTable older than that, etc. Of course, in the worst case (data not found in the system) reading may not be a very efficient operation. (TODO read/ write about bloom filters, etc.). Also, if the rate of incoming data is far more (thrashing?) than the rate at which SSTables are merged and compacted, space utilization can also suffer.

LSM-trees are better suited for write-heavy applications. 

They can suffer from Write-Amplification: a value is first written to the log for durability, then again when the memtable is written to disk, and again every time the key-value pair is part of a compaction. But if the values are significantly larger than the keys, this overhead can be reduced by storing values separately from keys and performing compaction only on SSTables containing keys and references to values.

Are LSM trees good for range queries? If the query is based on the key. 