---
title: "Reading Google's Bigtable Paper"
toc: true
date: "2023-07-18"
last_modified_at: 2024-06-05T00:00:01-00:00
tags: 
  - "bigtable"
  - "distributed-systems"
  - "googles-bigtable-paper"
  - "technical-papers"
  - "technical-reading"
---

Below are my notes from my reading of [Google's bigtable paper](https://github.com/papers-we-love/papers-we-love/blob/main/datastores/bigtable-a-distributed-storage-system-for-structured-data.pdf).

Bigtable is a sparse, distributed, persistent multi-dimensional sorted map. The map is indexed by a row key, column key, and a timestamp; each value in the map is an uninterpreted array of bytes.

Users typically access a row. Row operations are atomic. Columns (like languages) can be grouped in column families (typically same type of data). Column families are typically few in number (hundreds) but a table can have many columns. Access control and both disk and memory accounting are performed at the column-family level. Each cell in a Bigtable can contain multiple versions of the same data; these versions are indexed by timestamp.

![A Bigtable](/images/a_bigtable.png "A Bitgtable storing webpages and links to those pages")

**Building Blocks:** GFS, Chubby, SSTable, Memtable (These topics are good for separate blog posts)

**Implementation Components:** a client library, a master, and many tablet servers.

As familiar terms help, let's note that a tablet in BigTable is like a shard or partition.

The master is responsible for assigning tablets to tablet servers, detecting the addition and expiration of tablet servers, balancing tablet-server load, and garbage collection of files in GFS. In addition, it handles schema changes such as table and column family creations.

Each tablet server manages a set of tablets (ten to a thousand tablets per tablet server). The tablet server handles read and write requests to the tablets that it has loaded, and also splits tablets that have grown too large.

Clients communicate directly with tablet servers for reads and writes; most clients never communicate with the master. As a result, the master is lightly loaded in practice.

A Bigtable cluster stores a number of tables. Each table consists of a set of tablets, and each tablet contains all data associated with a row range. Initially, each table consists of just one tablet. As a table grows, it is automatically split into multiple tablets.

**Tablet Location**: (B+ - tree)
![Tablet Location Heirarchy](/images/bigtable_tablet_location_heirarchy.png "Tablet Location Heirarchy")

The first level is a file stored in Chubby that contains the location of the _root tablet_. The _root tablet_ contains the location of all tablets in a special METADATA table. Each METADATA tablet contains the location of a set of user tablets. The _root tablet_ is just the first tablet in the METADATA table, but is treated specially- it is never split- to ensure that the tablet location hierarchy has no more than three levels.

The client library caches tablet locations- it reads the metadata for more than one tablet whenever it reads the METADATA table. If the client does not know the location of a tablet, or if it discovers that cached location information is incorrect, then it recursively moves up the tablet location hierarchy

**Tablet Assignment**:

Each tablet is assigned to one tablet server at a time. The master keeps track of the set of live tablet servers, and the current assignment of tablets to tablet servers, including which tablets are unassigned. When a tablet server starts, it creates, and acquires an exclusive lock on, a uniquely-named file in a specific Chubby directory. The master monitors this directory (the _servers directory_) to discover tablet servers. A tablet server stops serving its tablets if it loses its exclusive lock but can continue to reacquire an exclusive lock as long as the file exists.

To detect when a tablet server is no longer serving its tablets, the master periodically asks each tablet server for the status of its lock. If a tablet server reports that it has lost its lock, or if the master was unable to reach a server during its last several attempts, the master attempts to acquire an exclusive lock on the server’s file. If the master is able to acquire the lock, then Chubby is live and the tablet server is either dead or having trouble reaching Chubby, so the master ensures that the tablet server can never serve again by deleting its server file. Once a table server’s file has been deleted (the tablet server kills itself), the master can move all the tablets that were previously assigned to that server into the set of unassigned tablets.

The master executes the following steps at startup.

1. The master grabs a unique _master_ lock in Chubby, which prevents concurrent master instantiations.

2. The master scans the servers directory in Chubby to find the live servers.

3. The master communicates with every live tablet server to discover what tablets are already assigned to each server.

4. The master scans the METADATA table to learn the set of tablets. Whenever this scan encounters a tablet that is not already assigned, the master adds the tablet to the set of unassigned tablets, which makes the tablet eligible for tablet assignment.

Read and Write: Memtable is in memory; commit logs and SSTable files are in files system.

![Writing To A Bigtable](/images/writing_to_a_bigtable.png "Writing To A Bigtable")

Write: 1. Write commit log, write into memtable (group commit), flush into SSTable.

Read: Read SSTable(s), Merge memtable data that is not flushed yet. As SSTable and memtable are String Sorted, the merged view is efficiently formed.

SSTables are immutable.

The only mutable data structure that is accessed by both reads and writes is the memtable. To reduce contention during reads of the memtable, each memtable row is copy-on-write and allows reads and writes to proceed in parallel.

**Compactions:**

Minor compaction: Freeze memtable upon reaching a size threshold, create a new memtable. Convert the frozen one into SSTable.

Merging Compaction (Done periodically): Merge a few SSTables and memtable into a new SSTable. Delete the source tables.

Major Compaction (Done periodically): Merge all SSTables into a single one.

**Refinements:** Locality Groups, Compression(Bentley McIlroy, fast compression), Caching(Scan cache, block cache), Bloom Filters, Commit log implementation- a single commit log per tablet server (multiple tablets)- sorting the commit log entries in order of the keys <table, row name, log sequence number>.

**Some thoughts upon reading:**

Wow, what level of thinking is that.

This was published in 2006. Distributed systems was not a buzzword yet. A lot of this thought process is reflected elsewhere now.

A lot of thinking has gone into what can go wrong and how to mitigate that. (I should re-read Nygard's Release It)

Understand in somewhat more depth: B+ trees, Bloom Filters, Chubby, GFS, Zookeeper, Cassandra. So much to do and so little time.
