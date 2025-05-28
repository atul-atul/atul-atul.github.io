---
title: "Reading Paper Scaling Memcache at Facebook"
toc: true
date: "2024-12-08"
last_modified_at: 2024-12-09T00:00:01-00:00
categories: 
  - notes
  - reading
tags: 
  - "distributed-systems"
  - "Memcache-paper"
  - "technical-papers"
  - "technical-reading"
---
I recently read the paper [Scaling Memcache At Facebook](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf). Some of my notes below:

---
First let's familiarize with some terms, ideas which are used in the paper/ my notes on it. This section may come across as teach-me-as-if-I-am-five-year-old description. Sorry for that.
A caching solution generally holds data in a hash-table like key-value structure- mostly in memory- to provide- more or less- O(1) look up. It saves trips to and thus reduces load on the data store. Thus improving performance**. The data store can be a database, a service, etc. Data can become stale (inconsistent with underlying source-of-truth store) and updates/ deletes/ new additions of data need to be managed. Some caching systems may take up persistence of data (data written first into the cache and later flushed to data-store in batches AKA write-back). Evicting data from cache/ cache invalidation can be challenging (There are two hard things in computer science: cache invalidation, naming things, and off-by-one errors.- Jeff Atwood of StackOverflow, codinghorror blog.). Look up terms like LRU, LFU, TTL, cache-aside, write-through/back/around, versioning, cache-stampede/ thundering-herd, etc.

**memcached**- Free & open source, high-performance, distributed memory object caching system. I pronounce it with d in the end (like in etcd) but you can pronounce it as the past tense of a verb. It was first designed for use in LiveJournal and is now a popular caching solution.

**memcache**- Notice the missing d in the end. Facebook's (FB) distributed caching system which uses memcached. When fb designed this system memcached was not a distributed solution. This paper discusses this system. They added capabilities to memcached and some of them are now open sourced.

---
One of the authors of the paper discusses some of the techniques/ rationale in [this talk](https://youtu.be/m4_7W4XzRgk).

---
For convenience and to avoid confusion, some clarification on terms used. They use the term client not to indicate the client app of cached-data but the memcached client app/ library to embed into/ call from the facebook application. FB web or mobile app interacts with memcached servers via this client side app. This client-library kind of app is called mcrouter.
The term 'frontend clusters' is used a lot. If I am not mistaken it means the deployments of user facing facebook site. As the term frontend can become overloaded with things like webapp, backend for mobile app, UI, javascript, etc. let's keep in mind that it means deployments of user facing facebook site.***

---
FB users consume an order of magnitude more content than they create. This behavior results in a workload dominated by fetching data. So caching such content helps. Second, the read operations fetch data from a variety of sources such as MySQL databases, HDFS installations, and backend services. These are authoritative sources. Memcache is not the authoritative source of the data. This heterogeneity of data sources requires a flexible caching strategy. Also, they are willing to expose slightly stale data in exchange for insulating a backend storage service from excessive load.

![demand-filled look-aside cache](/images/cache-aside-diagram.png "Cache-aside or look-aside. Demand filled.") 

Their use of cache is demand-filled look-aside cache (cache-aside). The content is saved to data store but is cached only when it's fetched/ read. On updates, they delete cached data instead of updating it because deletes are idempotent. 

The data that is cached isn't just end-user created data. Some derived data such as data created by ML algorithms is also held in cache.

The scale and reach of FB, wide fanout (say celebrity created content), read heavy use, latency expectations, etc. dictate that FB site is deployed in multiple regions and clusters within those regions. This places demand on data sources. So data sources are also deployed in these clusters and memcached servers as well. But this also means they need data replication and consistency. 

![FB_MC_Arch](/images/FB_MC_arch.png "FB memcache deployment")

---
The goals of caching at FB: reduce the latency of fetching cached data or the load imposed due to a cache miss.

#### Reduce Latency

The memcached servers are co-located in a cluster with webservers and storage. This reduces load on databases and other services and also reduces network lags. Cached items are distributed across the memcached servers through consistent hashing. Thus web servers have to routinely communicate with many memcached servers to satisfy a user request. As a result, all web servers communicate with every memcached server in a short period of time. This all-to-all communication pattern can cause incast congestion or allow a single server to become the bottleneck for many web servers. Data replication often alleviates the single-server bottleneck but leads to significant memory inefficiencies in the common case. They solved this using an indirection by introducing memcache client which runs on each web server. This client serves a range of functions, including serialization, compression, request routing, error handling, and request batching. The memcache client logic is provided as two components: a library that can be embedded into applications or as a standalone proxy named mcrouter. For convenience, I will call both of these (library and proxy) as mcrouter. This proxy presents a memcached server interface and routes the requests/replies to/from other servers. These mcrouters maintain a map of all available servers, which is updated through an auxiliary configuration system (I believe this aux config service is zookeeper).

To minimize the number of network round trips necessary to respond to page requests, FB construct a directed acyclic graph (DAG) representing the dependencies between data. A web server uses this DAG to maximize the number of items that can be fetched concurrently. They also use techniques such as sliding window to throttle the demand. The window size is dependent on observed metrics because smaller window means more number of request in progress and higher window size can cause incast congestion.

They use UDP for get requests to reduce latency and overhead. Since UDP is connectionless, each thread in the web server is allowed to directly communicate with memcached servers directly, bypassing mcrouter. For reliability, they route set and delete operations over TCP via mcrouter.

They treat get errors as cache misses, but web servers skip inserting entries into memcached after querying for data to avoid putting additional load on a possibly overloaded network or server. This number tends to be small in their experience.

---
#### Reduce Load on data stores or services

They use leases to address two problems: stale sets and thundering herds. (If you are not aware, look up thundering herds and also cache stampede.) 

A stale set occurs when a web server sets a value in memcache that does not reflect the latest value that should be cached. This can occur when concurrent updates to memcache get reordered. To refresh your memory, two operations are concurrent when each of them is unaware of the other taking place. And re-ordering of operations is interesting and a common problem because of network delays, etc. 

Stale sets are more relevant particularly for cache-aside solutions. Suppose the entry for Mary Jane's marital status is missing in the cache while it's stored as single in the master database. One fine day, Peter Parker looks at her profile and her status. His read from data store request would subsequently want to set the status as single in the demand-filled cache. But that fine day being the first day of spring and all, Mary Jane gets married to Harry. (In a Spiderman story she shouldn't, but this being memcached story suppose she does.) Now Harry and MJ would update their marital statuses to married. Suppose this data store update and subsequent read-your-own-write operation inserts her status as married into the cache. But due concurrent operation reordering Peter's read operation overwrites that status to single. It can cause bad blood. This kind of scenario is stale set. [Read more about it here](https://atul-atul.github.io/notes/linearizability-and-serializability/).

![Stale Set](/images/stale_set.png "Spiderman Spiderman :)")

A thundering herd happens when a specific key undergoes heavy read and write activity. As the write activity repeatedly invalidates the recently set values, many reads default to the more costly path.

A memcached instance gives a lease to a mcrouter to set data back into the cache when that client experiences a cache miss. The lease is a token bound to the specific key the client originally requested. The client provides the lease token when setting the value in the cache. With the lease token, memcached can verify and determine whether the data should be stored and thus arbitrate concurrent writes. Verification can fail if memcached has invalidated the lease token due to receiving a delete request for that item. Leases prevent stale sets in a manner similar to how load-link/store-conditional operates.

To address thundering herds each memcached server regulates the rate at which it returns tokens. By default, a token only once every 10 seconds per key. Requests for a key's value within 10 seconds of a token being issued results in a special notification telling the mcrouter to wait a short amount of time. Typically, the mcrouter with the lease will have successfully set the data within a few milliseconds. Thus, when waiting clients retry the request, the data is often present in cache.

At FB, there are areas where stale values (not stale sets) in cache are ok. The above Spiderman scenario was relevant when data was not present in cache. Because FB delete cached data from cache on data store updates, stale values are more relevant in value being updated in data store but not yet deleted from cache. A get request can return a lease token or data that is marked as stale. Applications that can continue to make forward progress with stale data do not need to wait for the latest value to be fetched from the databases.

To further reduce load on different backend stores, services which can differ in access patterns, memory footprints, etc. FB created multiple cache pools. They partition a cluster's memcached servers into separate pools. One pool (named wildcard) is the default. For keys not suitable for this wildcard pool, separate pools are provisioned. For example, a small pool for keys that are accessed frequently but for which a cache miss is inexpensive, another large pool for infrequently accessed keys for which cache misses are expensive, app pool(a pool devoted for a specific application- high churn, high miss rate), a replicated pool for frequently accessed data, and a regional pool for rarely accessed information, etc.

Some pools keys are replicated to improve the latency and efficiency of memcached servers. The idea is to replicate a category of keys within a pool when (1) the application routinely fetches many keys simultaneously, (2) the entire data set fits in one or two memcached servers and (3) the request rate is much higher than what a single server can manage. Replication in this instance is better than further dividing the key space. Consider a memcached server holding 100 items and capable of responding to 500k requests per second. Each request asks for 100 keys. The difference in memcached overhead for retrieving 100 keys per request instead of 1 key is small. To scale the system to process 1M requests/sec, splitting the the key space by introducing a new server may work. But clients now need to split each request for 100 keys into two parallel requests for ∼50 keys. Consequently, both servers still have to process 1M requests per second. However, if the entire keyspace is replicated instead of splitting it, a client's request for 100 keys can be sent to any replica. This reduces the load per server to 500k requests per second. Each client chooses replicas based on its own IP address. This approach requires delivering invalidations to all replicas to maintain consistency.

---
Handling Failures: Two scales of failures: (1) a widespread outage that affects many servers within the cluster- In this case requests are routed to different cluster or regions. (2) a small number of hosts are inaccessible due to a network or server failure For this scenario, FB dedicate small set of machines, named Gutter, to take over the responsibilities of a few failed servers. Gutter accounts for approximately 1% of the memcached servers in a cluster.
When a memcached client receives no response to its get request, the client assumes the server has failed and issues the request again to a special Gutter pool. If this second request misses, the client will insert the appropriate key-value pair into the Gutter machine after querying the database. Entries in Gutter expire quickly to obviate Gutter invalidations. Gutter limits the load on backend services at the cost of slightly stale data. By using Gutter to store these results, a substantial fraction of these failed get requests are converted into hits in the gutter pool thereby reducing load on the backing store.

---
#### Replication within a regiion
(refer to the image above) FB split web and memcached servers into multiple frontend clusters. These clusters, along with a storage cluster that contain the databases, define a region. This region architecture also allows for smaller failure domains and a tractable network configuration. Thus FB trade replication of data for more independent failure domains, tractable network configuration, and a reduction of incast congestion.

A web server that modifies data also sends invalidations to its own cluster. SQL statements that modify authoritative state are amended to include memcache keys that need to be invalidated once the transaction commits. Invalidation daemons (named mcsqueal) run on every database. Each daemon inspects the SQL statements that its database commits, extracts any deletes, and broadcasts these deletes to the memcache deployment in every frontend cluster in that region. Invalidation daemons batch deletes into fewer packets and send them to mcrouter instances in each frontend cluster. These mcrouters then unpack individual deletes from each batch and route those invalidations to the right memcached serves. Invalidation via mcsqueal is better than via web servers. First, batching is possible in mcsqueal and second, use of mcrouters avoids misrouting of deletes due to a configuration error.

A system called Cold Cluster Warmup mitigates poor hit rates when cache is cold by allowing clients in the "cold cluster" (i.e. the frontend cluster that has an empty cache) to retrieve data from the "warm cluster" (i.e. a cluster that has caches with normal hit rates) rather than the persistent storage. Memcached deletes support nonzero hold-off times that reject add operations for the specified hold-off time. By default, all deletes to the cold cluster are issued with a two second hold-off. When a miss is detected in the cold cluster, the client re-requests the key from the warm cluster and adds it into the cold cluster. The failure of the add (as the key is not present even in the warm cluster) indicates that newer data is available on the database and thus the client will refetch the value from the databases.

---
#### Consistency across regions
Each region consists of a storage cluster and several frontend clusters. One region holds the master databases and the other regions contain read-only replicas. FB use on MySQL's replication mechanism to keep replica databases up-to-date with their masters. When scaling across multiple regions, maintaining consistency between data in memcache and the persistent storage becomes the primary technical challenge. These challenges stem from a single problem: replica databases may lag behind the master database.
The paper describes what works for FB. Memcache system at FB represents just one point in the wide spectrum of consistency and performance trade-offs. They provide best-effort eventual consistency but place an emphasis on performance and availability.

Writes from a master region: use mcsqueal described earlier.
Writes from a non-master region: Consider a user who updates his data from a non-master region when replication lag is excessively large. The user's next request could result in confusion if his recent change is missing. A cache refill from a replica's database should only be allowed after the replication stream has caught up. Without this, subsequent requests could result in the replica's stale data being fetched and cached. FB use a remote marker mechanism to minimize the probability of reading stale data. The presence of the marker indicates that data in the local replica database are potentially stale and the query should be redirected to the master region. When a web server (non-master region) wishes to update data that affects a key k, that server (1) sets a remote marker rk in the region, (2) performs the write to the master embedding k and rk to be invalidated in the SQL statement, and (3) deletes k in the local cluster. On a subsequent request for k, a web server will be unable to find the cached data, check whether rk exists, and direct its query to the master or local region depending on the presence of rk. In this situation, FB are ok with added latency (introduced due to a cache miss) so as to avoid reading stale data. The presence of a remote marker helps distinguish whether a non-master database holds stale data or not.

---
#### Performance Optimizations
The first major optimizations were to: (1) allow automatic expansion of the hash table to avoid look-up times drifting to O(n), (2) make the server multi-threaded using a global lock to protect multiple data structures, and (3) giving each thread its own UDP port to reduce contention when sending replies and later spreading interrupt processing overhead. The first two optimizations were contributed back to the open source community. Employing fine-grained locking triples the peak get rate. UDP implementation outperforms TCP implementation by 13% for single gets and 8% for 10-key multigets(more data into each request than single gets).***

---
Memcached employs a slab allocator to manage memory. The allocator organizes memory into slab classes, each of which contains pre-allocated, uniformly sized chunks of memory. Memcached stores items in the smallest possible slab class that can fit the item's metadata, key, and value. Slab classes start at 64 bytes and exponentially increase in size by a factor of 1.07 up to 1 MB. FB implemented an adaptive allocator that periodically re-balances slab assignments to match the current workload. It identifies slab classes as needing more memory if they are currently evicting items and if the next item to be evicted was used at least 20% more recently than the average of the least recently used items in other slab classes. If such a class is found, then the slab holding the least recently used item is freed and transferred to the needy class.

---
While memcached supports expiration times, entries may live in memory well after they have expired. Memcached lazily evicts such entries by checking expiration times when serving a get request for that item or when they reach the end of the LRU. Although efficient for the common case, this scheme allows shortlived keys that see a single burst of activity to waste memory until they reach the end of the LRU.
FB introduced a hybrid scheme that relies on lazy eviction for most keys and proactively evicts shortlived keys when they expire. To do this they place short-lived items into a circular buffer of linked lists (indexed by seconds until expiration)– called the Transient Item Cache– based on the expiration time of the item. Every second, all of the items in the bucket at the head of the buffer are evicted and the head advances by one. 

FB modified memcached to store its cached values and main data structures in System V shared memory regions so that the data can remain live across a software upgrade and thereby minimize disruption.

In conclusion section the paper says: Many of the trade-offs discussed are not fundamental, but are rooted in the realities of balancing engineering resources while evolving a live system under continuous product development. While building, maintaining, and evolving our system we have learned the following lessons. (1) Separating cache and persistent storage systems allows us to independently scale them. (2) Features that improve monitoring, debugging and operational efficiency are as important as performance. (3) Managing stateful components is operationally more complex than stateless ones. As a result keeping logic in a stateless client helps iterate on features and minimize disruption. (4) The system must support gradual rollout and rollback of new features even if it leads to temporary heterogeneity of feature sets. (5) Simplicity is vital.

---
** Performance is one of those hygiene factors, isn't it? You can [read more about hygiene factors](https://en.wikipedia.org/wiki/Two-factor_theory) on wikipedia. Basically, having a clean table in cafetaria during lunch hour does not make you happy but seeing it unclean will make you unhappy. 

Here is some related content I copied from DDIA book:

  It seems intuitively obvious that a fast service is better for users than a slow service. However, it is surprisingly difficult to get hold of reliable data to quantify the effect that latency has on user behavior.

  Some often-cited statistics are unreliable. In 2006 Google reported that a slowdown in search results from 400 ms to 900 ms was associated with a 20% drop in traffic and revenue. However, another Google study from 2009 reported that a 400 ms increase in latency resulted in only 0.6% fewer searches per day, and in the same year Bing found that a two-second increase in load time reduced ad revenue by 4.3%. Newer data from these companies appears not to be publicly available.

  A more recent Akamai study claims that a 100 ms increase in response time reduced the conversion rate of e-commerce sites by up to 7%; however, on closer inspection, the same study reveals that very fast page load times are also correlated with lower conversion rates! This seemingly paradoxical result is explained by the fact that the pages that load fastest are often those that have no useful content (e.g., 404 error pages). However, since the study makes no effort to separate the effects of page content from the effects of load time, its results are probably not meaningful.

  A study by Yahoo compares click-through rates on fast-loading versus slow-loading search results, controlling for quality of search results. It finds 20–30% more clicks on fast searches when the difference between fast and slow responses is 1.25 seconds or more.

---
*** By and large, from technical and social perspective, FB created a lot of value. I deleted my FB account long, well actually, long long back. I haven't been a socially extroverted person ever. But a factor was my experience with orkut. I did not want to be on another such network. Of course, FB makes a lot of things easy. And not being on it has some compromises associated with it which any herd animal has to make by not being a part of the a herd. And I don't know if people use FB much these days. Do they even change their DPs? Going back to [my post on platforms](https://atul-atul.github.io/notes/platforms/) or quoting from Steve Yegge's platform rant, "Facebook -- that is, the stock service they offer with walls and friends and such -- is the killer app for the Facebook Platform.  And it is a very serious mistake to conclude that the Facebook App could have been anywhere near as successful without the Facebook Platform." It's the platform which enabled things like ads, viral content, suggestions, third party integrations, games, marketplace, smear campaigns, propoganda, etc. If you haven't, go read Steve Yegge's platform rant right now. It is more valuable than the paper.

---
I finished reading the paper yesterday. Reading it took a few sittings over the last two weeks. And even with underlines, annotations, etc. writing this post from the paper took a lot of time today. Plus there are things like nagging internal voice telling you to take a shortcut and watch a related memcache talk, read some post to help you understand. But we do some things precisely because they need the attention thus delaying gratification in the process. Otherwise, I would end up doom-scrolling on social networks, wouldn't I? So long as we can be happy in long term with the choices we make and paths we take... Of course, I felt bad when I realized I missed watching [Ellyse Perry ODI century](https://www.espncricinfo.com/series/australia-women-vs-india-women-2024-25-1426497/australia-women-vs-india-women-2nd-odi-1426621/full-scorecard). And I did watch the movie Aliens. So there is that.