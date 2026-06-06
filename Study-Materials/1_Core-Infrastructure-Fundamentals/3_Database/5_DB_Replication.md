# Replication — System Design Interview Notes

# 1. Introduction

Replication is one of the MOST IMPORTANT concepts in distributed systems and system design interviews.

Almost every large-scale system uses replication:

* databases
* caches
* Kafka
* distributed file systems
* microservices
* cloud storage systems

Interviewers ask replication because it tests whether you understand:

* high availability
* fault tolerance
* scalability
* consistency tradeoffs
* distributed systems behavior

The MOST important thing to understand:

> Replication means keeping copies of data on multiple machines.

But the REAL engineering question is:

> WHY are we creating multiple copies?

---

# 2. Why Replication Exists

Suppose system has only ONE database server.

Problems:

* server crash → system down
* hardware failure → data unavailable
* high read traffic → bottleneck
* region outage → complete outage

Single server becomes:

```text id="ur7dco"
Single Point of Failure (SPOF)
```

Replication solves this.

---

# 3. Core Goals of Replication

Replication mainly exists for:

* High Availability
* Fault Tolerance
* Read Scalability
* Disaster Recovery
* Geographic Distribution

---

# 4. High Availability

Suppose primary DB crashes.

Without replication:

```text id="10v4q6"
Application unavailable
```

With replication:

* replica can take over
* system continues functioning

This improves:

```text id="nkkthv"
Availability
```

---

# Real World Example

Banking app with:

* Primary DB in Mumbai
* Replica DB in Bangalore

If Mumbai server crashes:

* Bangalore replica promoted

Users can still access system.

---

# 5. Fault Tolerance

Hardware failures are NORMAL in production.

Machines fail because of:

* disk crashes
* power failures
* memory corruption
* network issues

Replication ensures:

> data survives failures.

---

# Real Engineering Insight

At scale:

```text id="s8hr92"
Failure is expected
```

Good systems are designed assuming machines WILL fail.

---

# 6. Read Scalability

Suppose millions of users reading same data.

Single DB becomes bottleneck.

Replication allows:

* distributing read traffic
* improving throughput

---

# Real Example — Instagram

Millions of users viewing:

* profiles
* posts
* comments

Reads dominate writes.

Instead of:

```text id="pcd3zy"
1 DB handling all reads
```

system uses:

* multiple replicas

Read traffic distributed across replicas.

---

# 7. Basic Replication Architecture

Most common setup:

```text id="9n1vgn"
Primary Database
      ↓
Replica 1
Replica 2
Replica 3
```

Primary handles:

* writes

Replicas handle:

* reads

This is:

```text id="9vx3z9"
Primary-Replica Replication
```

Also called:

* Master-Slave
* Leader-Follower

---

# 8. How Primary-Replica Replication Works

Suppose user updates profile:

```sql id="1h95bw"
UPDATE users
SET name='Akashdeep'
WHERE id=1;
```

Write goes to:

```text id="x1t5tb"
Primary DB
```

Primary:

* updates local data
* propagates changes to replicas

Replicas copy same update.

---

# 9. Replication Lag

VERY important interview topic.

Replication is usually NOT instantaneous.

There may be delay:

```text id="ak40ea"
Primary updated
Replica not updated yet
```

This delay is:

```text id="80icvk"
Replication Lag
```

---

# Real World Example

Instagram like count:

Primary:

```text id="k0l81t"
101 likes
```

Replica:

```text id="l4fjlwm"
100 likes
```

for few milliseconds/seconds.

This is stale read caused by replication lag.

---

# 10. Synchronous Replication

# How It Works

Primary waits for replicas to confirm write BEFORE responding success.

Flow:

```text id="1es4t7"
Client Write
    ↓
Primary
    ↓
Replica ACK
    ↓
Success Response
```

---

# Advantages

* strong consistency
* replicas always up-to-date
* safer failover

---

# Disadvantages

* slower writes
* higher latency
* reduced availability during failures

---

# Real Use Cases

Used when:

* correctness critical

Examples:

* banking
* financial systems
* inventory systems

---

# 11. Asynchronous Replication

# How It Works

Primary responds immediately after local write.

Replication happens later asynchronously.

Flow:

```text id="1jlwm7"
Client Write
    ↓
Primary
    ↓
Success Response
    ↓
Replica updated later
```

---

# Advantages

* very fast writes
* lower latency
* better throughput

---

# Disadvantages

* replication lag
* stale reads possible
* possible data loss during failover

---

# Real Use Cases

Used in:

* social media
* analytics
* recommendation systems

Where slight inconsistency acceptable.

---

# 12. Important Tradeoff

Synchronous replication:

```text id="jlwm55"
Better consistency
Worse performance
```

Asynchronous replication:

```text id="4jlwm3"
Better performance
Weaker consistency
```

Classic distributed systems tradeoff.

---

# 13. Read Replicas

Very common interview topic.

Read replicas are:

> replicas dedicated to serving read traffic.

---

# Example

```text id="xjlwm6"
Primary → Writes
Replica1 → Reads
Replica2 → Reads
Replica3 → Reads
```

Application routes:

* writes → primary
* reads → replicas

---

# Real Production Example

Suppose:

```text id="njlwm8"
95% traffic = reads
```

Read replicas massively improve scalability.

Used heavily in:

* MySQL
* PostgreSQL
* cloud databases

---

# 14. Problem — Stale Reads

Suppose:

1. User updates profile
2. Immediately reads profile
3. Read goes to stale replica

User sees old data.

This is:

```text id="5jlwm0"
Read-after-write inconsistency
```

---

# Solutions

* read from primary temporarily
* sticky sessions
* quorum reads
* bounded staleness

Very common interview discussion.

---

# 15. Multi-Primary Replication

Instead of single primary:

```text id="6jlwm2"
Multiple nodes accept writes
```

Example:

```text id="6jlwm1"
Region A → writes
Region B → writes
```

---

# Advantages

* better write availability
* lower regional latency
* no single write bottleneck

---

# Problem — Write Conflicts

Suppose:

Region A:

```text id="jlwm64"
username = Akash
```

Region B simultaneously:

```text id="vjlwm2"
username = Akashdeep
```

Conflict occurs.

---

# Conflict Resolution Needed

System must decide:

* which write wins
* how to merge changes

Very difficult problem.

---

# Common Conflict Strategies

* Last Write Wins (LWW)
* Vector clocks
* CRDTs
* Application-level merge

---

# 16. Leaderless Replication

Used in systems like:

* DynamoDB
* Cassandra

No single leader exists.

Any node can accept writes.

---

# How It Works

Write sent to:

* multiple replicas simultaneously

Consistency achieved using:

* quorum reads
* quorum writes

---

# Advantages

* no single point of failure
* high availability
* better partition tolerance

---

# Disadvantages

* conflict resolution complex
* eventual consistency common
* harder operationally

---

# 17. Quorum Reads/Writes

VERY important distributed systems concept.

Suppose:

```text id="hjlwm7"
3 replicas
```

Write succeeds if:

```text id="7jlwm0"
2 replicas acknowledge
```

Read succeeds from:

```text id="qjlwm1"
2 replicas
```

This improves:

* consistency
* fault tolerance

---

# Important Formula

If:

```text id="mjlwm8"
R + W > N
```

where:

* R = read quorum
* W = write quorum
* N = total replicas

then strong consistency possible.

Very famous interview concept.

---

# 18. Geo Replication

Replication across regions/countries.

Example:

```text id="jlwm29"
US Replica
Europe Replica
India Replica
```

---

# Why Needed?

* lower latency
* disaster recovery
* regional fault tolerance

---

# Problem

Cross-region replication slower because:

* network latency high
* distributed coordination expensive

---

# 19. Failover

Suppose primary crashes.

System promotes replica:

```text id="jlwm83"
Replica → New Primary
```

This process:

```text id="3jlwm4"
Failover
```

Can be:

* automatic
* manual

---

# Important Challenge

Need to ensure:

* no split brain
* no data corruption

---

# 20. Split Brain Problem

Very important interview topic.

Suppose network partition occurs.

Two nodes BOTH think:

```text id="9jlwm9"
"I am primary"
```

Now:

* both accept writes
* data diverges

This is:

```text id="8jlwm1"
Split Brain
```

Dangerous distributed systems problem.

---

# Solutions

* leader election
* consensus protocols
* quorum systems

---

# 21. Replication vs Backup

Very common interview question.

---

# Replication

Purpose:

```text id="jlwm70"
High availability + scalability
```

Data copied continuously.

---

# Backup

Purpose:

```text id="jlwm91"
Disaster recovery
```

Point-in-time snapshot.

---

# Important Difference

Replication copies:

* corruption too

Backup helps recover historical state.

---

# 22. Real Production Examples

## MySQL/PostgreSQL

Usually:

* primary-replica replication

---

## Cassandra/DynamoDB

Usually:

* leaderless replication
* eventual consistency

---

## Kafka

Replicates partitions across brokers for durability.

---

## MongoDB

Uses replica sets:

* primary
* secondary replicas

---

# 23. Most Important Interview Thought Process

Suppose interviewer says:

> "Design Twitter timeline DB."

Think:

* reads massive?
* global users?
* can stale data tolerated?

Likely:

* asynchronous replication
* read replicas

---

Suppose interviewer says:

> "Design payment ledger."

Think:

* correctness critical
* stale reads dangerous

Likely:

* synchronous replication
* stronger consistency

---

# 24. Common Interview Questions

# Q1. Why Use Replication?

* high availability
* fault tolerance
* read scalability

---

# Q2. Why Not Replicate Synchronously Everywhere?

Because:

* latency increases
* availability decreases
* coordination expensive

---

# Q3. What is Replication Lag?

Delay between:

* primary update
* replica synchronization

---

# Q4. Why Are Stale Reads Dangerous?

Users may see:

* outdated balances
* stale inventory
* inconsistent profiles

---

# Q5. What is Split Brain?

Two nodes simultaneously acting as primary.

Can corrupt data.

---

# 25. Final Mental Model

Replication fundamentally exists to provide:

```text id="0jlwm5"
Availability + Fault Tolerance + Scalability
```

But replication introduces:

* consistency complexity
* synchronization delays
* conflict resolution problems

---

# 26. Final Revision Summary

| Replication Type | Main Idea                       | Tradeoff                          |
| ---------------- | ------------------------------- | --------------------------------- |
| Synchronous      | Replicas updated before success | Better consistency, slower writes |
| Asynchronous     | Replicas updated later          | Faster writes, stale reads        |
| Primary-Replica  | Single write leader             | Simpler consistency               |
| Multi-Primary    | Multiple write nodes            | Conflict resolution complexity    |
| Leaderless       | No central leader               | Eventual consistency              |

---

# 27. Senior-Level Insight

Replication is fundamentally about:

> trading consistency complexity for higher availability, fault tolerance, and scalability in distributed systems.
