---
title: "Idempotent operations"
toc: true
date: "2023-10-21"
tags: 
  - "distributed-systems"
  - "idempotence"
  - "technical-reading"
  - reading
---

Here is some content about [idempotent operations](https://en.wikipedia.org/wiki/Idempotence) from the book Foundations of Scalable Systems by Ian Gorton

> The general approach to building idempotent operations is as follows:
> 
> 1. Clients include a unique idempotency key in all requests that mutate state. The key identifies a single operation from the specific client or event source. It is usually a composite of a user identifier, such as the session key, and a unique value such as a local timestamp, UUID, or a sequence number.
> 
> 3. When the server receives a request, it checks to see if it has previously seen the idempotency key value by reading from a database that is uniquely designed for implementing idempotence. If the key is not in the database, this is a new request. The server therefore performs the business logic to update the application state. It also stores the idempotency key in a database to indicate that the operation has been successfully applied.
> 
> 5. If the idempotency key is in the database, this indicates that this request is a retry from the client and hence should not be processed. In this case the server returns a valid response for the operation so that (hopefully) the client won’t retry again.
> 
> The database used to store idempotency keys can be implemented in, for example:
> 
> 1. A separate database table or collection in the transactional database used for the application data
> 
> 3. A dedicated database that provides very low latency lookups, such as a simple key-value store
> 
> Unlike application data, idempotency keys don’t have to be retained forever. Once a client receives an acknowledgment of a success for an individual operation, the idempotency key can be discarded. The simplest way to achieve this is to automatically remove idempotency keys from the store after a specific time period, such as 60 minutes or 24 hours, depending on application needs and request volumes.
> 
> In addition, an idempotent API implementation must ensure that the application state is modified and the idempotency key is stored. Both must occur for success. If the application state is modified and, due to some failure, the idempotent key is not stored, then a retry will cause the operation to be applied twice. If the idempotency key is stored but for some reason the application state is not modified, then the operation has not been applied. If a retry arrives, it will be filtered out as duplicate as the idempotency key already exists, and the update will be lost. 
> 
> The implication here is that the updates to the application state and idempotency key store must both occur, or neither must occur
