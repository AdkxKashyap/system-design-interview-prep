# Advanced Replication Concepts — System Design Interview Notes

# 1. Introduction

Advanced replication concepts are extremely important for:

* SDE2 interviews
* senior backend interviews
* distributed systems design

These topics test whether you understand:

* distributed databases
* replication internals
* consistency tradeoffs
* failover handling
* global systems design
* fault tolerance

The most important understanding:

```text id="6s7z0a"
Replication is fundamentally about balancing:
Consistency + Availability + Latency + Fault Tolerance
```

---

# 2. Leader-Follower Replication

This is the MOST common replication architecture.

Examples:

* MySQL replication
* PostgreSQL replication
* MongoDB replica sets
* Kafka partition leaders

---

# Core Idea

One node becomes:

```text id="n1k2o3"
Leader (Primary)
```

Other nodes become:

```text id="0n9v8b"
Followers (Replicas)
```

---

# Architecture

```text id="x1m7wa"
                 Writes
Client ───────────► Leader
                       │
                       │ Replication
                       ▼
               ┌─────────────┐
               │ Follower 1  │
               └─────────────┘
                       │
                       ▼
               ┌─────────────┐
               │ Follower 2  │
               └─────────────┘
```

---

# How It Works

Suppose user updates profile:

```sql id="r4t9za"
UPDATE users
SET bio = 'System Design'
WHERE id = 1;
```

Request goes ONLY to:

```text id="j2c6kx"
Leader
```

Leader:

1. writes data locally
2. updates WAL/replication log
3. propagates changes to followers

Followers replay same operations.

---

# Why Single Leader Important

Without leader:
multiple nodes may accept conflicting writes.

Example:

Replica A:

```text id="m8f0lr"
username = Akash
```

Replica B:

```text id="s5x4yt"
username = Akashdeep
```

Conflict resolution becomes difficult.

Single leader simplifies consistency.

---

# Real World Analogy

Think of:

```text id="r3h7vp"
School Principal
```

Only principal officially approves changes.

Teachers:

* maintain copies
* distribute information
* answer questions

Similar to:

```text id="u4n1jw"
Leader-Follower Replication
```

---

# Why Followers Exist

Followers mainly provide:

* read scaling
* failover
* backups
* analytics queries

---

# Real Example — Instagram

Millions of users reading:

* profiles
* reels
* comments

Reads distributed across followers.

Writes centralized through leader.

---

# 3. Replication Log / WAL (Very Important)

Leader maintains:

```text id="n6y4ca"
WAL (Write Ahead Log)
```

Every write recorded sequentially.

Followers continuously:

* pull log entries
* replay changes locally

---

# Replication Flow

```text id="k1v9pd"
Write Request
     ↓
 Leader DB
     ↓
 Write to WAL
     ↓
 Send WAL entries to followers
     ↓
 Followers replay changes
```

---

# 4. Synchronous Replication

# Core Idea

Leader waits for followers BEFORE confirming success.

---

# Flow

```text id="q2m8er"
Client Write
    ↓
Leader writes locally
    ↓
Followers ACK
    ↓
Success returned
```

---

# Advantages

* strong consistency
* replicas always updated
* safer failover

---

# Disadvantages

* slower writes
* higher latency
* lower availability during failures

---

# Real Use Cases

Used when:

* correctness critical

Examples:

* banking
* financial systems
* inventory systems

---

# 5. Asynchronous Replication

# Core Idea

Leader responds immediately after local write.

Followers updated later.

---

# Flow

```text id="z5t0ql"
Client Write
    ↓
Leader writes locally
    ↓
Success returned immediately
    ↓
Followers updated later
```

---

# Advantages

* fast writes
* lower latency
* higher throughput

---

# Disadvantages

* replication lag
* stale reads
* possible data loss during failover

---

# Real Use Cases

Used in:

* social media
* analytics
* recommendation systems
* feeds

where slight inconsistency acceptable.

---

# 6. Replication Lag

Replication is NOT always instantaneous.

Example:

Leader:

```text id="c1d8oa"
Likes = 101
```

Follower:

```text id="y4e7ib"
Likes = 100
```

because replication not caught up yet.

This delay:

```text id="b9p2wh"
Replication Lag
```

---

# Real Production Problem

User:

1. updates profile
2. refreshes immediately
3. read goes to stale follower

User sees old data.

This is:

```text id="m5u3jr"
Read-after-write inconsistency
```

---

# 7. Failover

Suppose leader crashes.

System promotes follower:

```text id="t8v0cs"
Follower → New Leader
```

This process:

```text id="h4w9zk"
Failover
```

---

# Important Challenges

Need to ensure:

* only ONE leader exists
* no data corruption
* replicas synchronized properly

---

# 8. Split Brain Problem

VERY important distributed systems problem.

---

# What is Split Brain?

Suppose network partition occurs.

Two nodes BOTH think:

```text id="w7y2nm"
"I am leader"
```

Both accept writes.

Data diverges.

This is:

```text id="d0x8pv"
Split Brain
```

---

# Why Dangerous?

Can cause:

* conflicting writes
* corrupted data
* permanent inconsistency

---

# Example

Node A:

```text id="u6b1kf"
balance = ₹5000
```

Node B:

```text id="j9q4al"
balance = ₹3000
```

after partition heals:

* conflict resolution difficult

---

# 9. Split Brain Solutions (Brief Notes)

## Leader Election

Only one node elected as leader.

Common algorithms:

* Raft
* Paxos

---

## Quorum Systems

Require majority agreement before becoming leader.

Example:

```text id="p2f7ws"
2 out of 3 replicas required
```

Prevents multiple leaders.

---

## Fencing Tokens

Each leader gets unique increasing token/version.

Old leaders rejected automatically.

Used heavily in distributed locking systems.

---

## STONITH (Shoot The Other Node In The Head)

Aggressively shuts down conflicting node.

Common in:

* clustered systems
* enterprise HA setups

---

# 10. Multi-Region Replication

# Why Needed

Suppose ALL traffic goes to:

```text id="s8m1dz"
US database
```

Indian users experience:

* high latency
* slow writes
* poor UX

---

# Why Latency Happens

Because:

* speed of light limitations
* intercontinental network delays
* cross-region routing overhead

India ↔ US roundtrip may take:

```text id="q7y5nb"
200–300ms
```

Too slow for modern apps.

---

# 11. Multi-Region Replication Architecture

Replicas distributed globally.

---

# Example

```text id="n3k8xa"
         US Leader
            │
   ┌────────┼────────┐
   ▼        ▼        ▼

India    Europe    Singapore
Replica  Replica    Replica
```

---

# Benefits

## Lower Latency

Indian users read from:

```text id="m6w0pf"
India replica
```

instead of US.

Much faster.

---

## Disaster Recovery

If US region fails:

* India/Europe replicas survive

Huge availability improvement.

---

## Geographic Fault Tolerance

Regional outages no longer catastrophic.

---

# 12. Cross-Region Replication Challenges

Suppose:
US user updates profile.

Question:

> how fast should India replica reflect change?

Immediately?

Eventually?

This creates:

* consistency challenges
* replication lag
* coordination overhead

---

# Why Strong Global Consistency Is Hard

Suppose synchronous replication across:

* US
* India

Every write must wait globally.

Result:

```text id="f2l4vy"
300ms+ write latency
```

Poor user experience.

---

# Real Production Tradeoff

Most global systems choose:

* eventual consistency
* asynchronous cross-region replication

because:

* latency more important
* uptime more important

---

# Real Example — Social Media

If like count delayed:

```text id="o9x6kd"
1–2 seconds
```

usually acceptable.

Users prefer:

* fast system
* available system

over perfect consistency.

---

# 13. Quorum Reads/Writes

Very important distributed systems concept.

Used heavily in:

* Cassandra
* DynamoDB
* Riak

---

# Core Idea

Suppose data replicated to:

```text id="r5u1ew"
3 nodes
```

Question:

> how many replicas must participate in reads/writes?

This determines:

* consistency
* availability
* latency

---

# 14. Write Quorum

Suppose:

```text id="g3v9lm"
W = 2
```

Meaning:

> write succeeds only if 2 replicas acknowledge.

---

# Example

Replicas:

```text id="v1b7pt"
A B C
```

Write sent to all.

If:

```text id="q6z2fr"
A + B acknowledge
```

write succeeds.

Even if:

```text id="s4e0dn"
C failed
```

system still available.

---

# 15. Read Quorum

Suppose:

```text id="a8j3xo"
R = 2
```

Read queries 2 replicas.

System compares versions and returns latest value.

---

# 16. Most Important Formula

If:

```text id="h0m9cu"
R + W > N
```

where:

* R = read quorum
* W = write quorum
* N = total replicas

then:

> strong consistency possible.

---

# Example

Suppose:

```text id="x4t8ks"
N = 3
W = 2
R = 2
```

Then:

```text id="b7y5lf"
2 + 2 > 3
```

So reads and writes overlap on at least one replica.

Latest write guaranteed visible.

---

# 17. Why Quorums Powerful

Quorums allow balancing:

* consistency
* availability
* latency

instead of requiring:

```text id="d1u6vr"
ALL replicas always
```

Very flexible production approach.

---

# 18. Quorum Tradeoffs

## Higher Quorum

* better consistency
* higher latency
* lower availability

---

## Lower Quorum

* faster
* more available
* weaker consistency

Classic distributed systems tradeoff.

---

# 19. Real Production Example — DynamoDB

Applications can configure:

* strong consistency
* eventual consistency

using quorum settings.

Very powerful production feature.

---

# 20. Final Mental Models

# Leader-Follower Replication

```text id="v9q0ax"
Single node handles writes
Followers replicate changes
```

Good for:

* simpler consistency
* read scaling

---

# Multi-Region Replication

```text id="p8d2mk"
Replicas distributed globally
```

Good for:

* low latency
* disaster recovery
* global availability

---

# Quorum Reads/Writes

```text id="r2w4jc"
Require subset of replicas for operations
```

Good for:

* balancing consistency and availability

---

# 21. Final Revision Summary

| Concept                  | Main Goal           | Main Tradeoff          |
| ------------------------ | ------------------- | ---------------------- |
| Leader-Follower          | Simpler consistency | Leader bottleneck      |
| Synchronous Replication  | Strong consistency  | Higher latency         |
| Asynchronous Replication | Faster writes       | Replication lag        |
| Multi-Region Replication | Low global latency  | Eventual consistency   |
| Quorum Reads/Writes      | Tunable consistency | Operational complexity |

---

# 22. Senior-Level Insight

Advanced replication is fundamentally about:

> balancing consistency, latency, availability, and fault tolerance across geographically distributed systems.
