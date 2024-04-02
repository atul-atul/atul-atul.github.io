---
title: "Two Phase Commit"
date: "2023-10-23"
categories: ["notes"]
tags: ["distributed-systems", "technical-reading" "two-phase-commit"]
---

An old topic. Some content/ notes from Foundations of Scalable Systems by Ian Gorton. Posting here for quick reference.

When a client starts a transaction a coordinator (can be database internal or an external service) allocates a globally unique transaction identifier (_tid_) and returns this to the client. The _tid_ identifies the transaction context which records the database partitions, or participants, that take part in the transaction and the state of their communications. The context is persisted by the coordinator, so that it durably maintains the state of the transaction.

The client then executes the operations defined by the transaction, passing the _tid_ to each participant that performs the database operations. Each participant acquires locks on mutated objects and executes the operations locally. It also durably associates the _tid_ with the updates in a local transaction log. These database updates are not completed at this stage—this only occurs if the transaction commits.

Once all the operations in the transaction are completed successfully, the client tries to commit the transaction. This is when the 2PC algorithm commences on the coordinator, which drives two rounds of votes with the participants:

1. **Prepare phase** The coordinator sends a message to all participants to tell them to prepare to commit the transaction. When a participant successfully prepares, it guarantees that it can commit the transaction and make it durable. After this, it can no longer unilaterally decide to abort the transaction. If a participant cannot prepare, that is, if it cannot guarantee to commit the transaction, it must abort. Each participant then informs the coordinator about its decision to commit or abort by returning a message that contains its decision.

2. **Resolve phase** When all the participants have replied to the _prepare_ phase, the coordinator examines the results. If all the participants can commit, the whole transaction can commit, and the coordinator sends a commit message to each participant. If any participant has decided that it must abort the transaction, or doesn’t reply to the coordinator within a specified time period, the coordinator sends an abort message to each participant.

![](images/two_phase_commit.png)

2PC has two main **failure modes**. These are participant failure and coordinator failure. As usual, failures can be caused by systems crashing, or being partitioned from the rest of the application. From the perspective of 2PC, the crashes and partitions are indistinguishable:

1. **Participant failure** When a participant crashes before the _prepare_ phase completes, the transaction is aborted by the coordinator. This is a straightforward failure scenario. It’s also possible for a participant to reply to the _prepare_ message and then fail. In either case, when the participant restarts, it needs to communicate with the coordinator to discover transaction outcomes. The coordinator can use its transaction log to look up the outcomes and inform the recovered participant accordingly. The participant then completes the transaction locally. Essentially then, participant failure doesn’t threaten consistency, as the correct transaction outcome is reached.

3. **Coordinator failure** Should the coordinator fail after sending the _prepare_ message, participants have a dilemma. Participants that have voted to commit must block until the coordinator informs them of the transaction outcome. If the coordinator crashes before or during sending out the commit messages, participants cannot proceed, as the coordinator has failed and will not send the transaction outcome until it recovers. There is no simple resolution to this problem. A participant cannot autonomously decide to commit as it does not know how other participants voted. If one participant has voted to roll back, and others to commit, this would violate transaction semantics. The only practical resolution is for participants to wait until the coordinator recovers and examines its transaction log.6 The log enables the coordinator to resolve all incomplete transactions. If it has logged a commit entry for an incomplete transaction, it will inform the participants to commit. Otherwise, it will roll back the transaction. The downside is that participants must block while the coordinator recovers.

In summary, the weakness of 2PC is that it is not tolerant of coordinator failure. One possible way to fix this, as with all single point of failure problems, is to replicate the coordinator and transaction state across participants. If the coordinator fails, a participant can be promoted to coordinator and complete the transaction. Taking this path leads to a solution that requires a distributed consensus algorithm (like paxos, raft, etc.).
