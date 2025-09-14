---
title: "Bulkhead in Software Systems"
toc: true
date: "2022-02-16"
last_modified_at: 2022-02-16T00:00:01-00:00
tags:
  - "technical-papers"
  - "technical-reading" 
  - reading
---
**From the book Release It! Second Edition**

In a ship, *bulkheads* are partitions that, when sealed, divide the ship into separate, watertight compartments. With hatches closed, a bulkhead prevents water from moving from one section to another. In this way, a single penetration of the hull does not irrevocably sink the ship. The bulkhead enforces a principle of damage containment.

By partitioning your systems, you can keep a failure in one part of the system from destroying everything. Physical redundancy is the most common form of bulkheads. (Physical servers- virtue of hardware and software separation, redundant virtual partitions)

The goal is to identify the natural boundaries that let you partition the system in a way that is both technically feasible and financially beneficial. The boundaries of this partitioning may be aligned with the callers, with functionality, or with the topology of the system.

With cloud-based systems and software-defined load balancers, bulkheads do not need to be permanent. With a bit of automation, a cluster of VMs can be carved out and the load balancer can direct traffic from a particular consumer to that cluster. This is similar to A/B testing, but as a protective measure rather than an experiment. Dynamic partitions can be made and destroyed as traffic patterns change.

Bulkheads are effective at maintaining service, or partial service, even in the face of failures. They are especially useful in service-oriented (and microservices) architectures, where the loss of a single service could have repercussions throughout the enterprise.