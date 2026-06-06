# CAP Theorem — System Design Interview Notes

# 1. Introduction

CAP theorem is one of the MOST IMPORTANT distributed systems concepts.

Interviewers use CAP theorem to test whether you understand:

* distributed databases
* network failures
* consistency tradeoffs
* availability tradeoffs
* real-world distributed system behavior

Most candidates memorize:

```text
Consistency
Availability
Partition Tolerance
```

and say:

> "Choose 2 out of 3"

This is incomplete and often misleading.

The REAL understanding is:

> During a network partition, distributed systems must choose between consistency and availability.

---

# 2. Why CAP Theorem Exists

CAP theorem only matters in:

```text
Distributed Systems
```

Example:

* multi-region databases
* replicated databases
* distributed storage systems

The core problem:

> networks are unreliable.

Servers can become disconnected because of:

* network failures
* packet loss
* region outages
* router failures

This creates:

```text
Network Partitions
```

---

# 3. What is a Network Partition?

A partition means:

> nodes in distributed system cannot communicate reliably.

Example:

```text
Mumbai Server ↔ Singapore Server
```

connection breaks.

Both servers still alive,
but cannot synchronize data properly.

This is:

```text
Partition
```

---

# 4. Why Partition Tolerance is Mandatory

In real distributed systems:

* networks WILL fail
* packets WILL drop
* regions WILL become isolated

Therefore:

```text
Partition Tolerance (P)
```

is NOT optional.

Real systems MUST tolerate partitions.

---

# 5. What Does Consistency Mean in CAP?

CAP consistency means:

> all replicas see the SAME data at the SAME time.

Example:

User updates name:

```text
Akash → Akashdeep
```

Immediately after update:
every replica must return:

```text
Akashdeep
```

No stale data allowed.

---

# 6. What Does Availability Mean?

Availability means:

> every request receives SOME response.

Even during failures.

Response may:

* be stale
* be outdated

but system still responds.

---

# 7. The REAL CAP Tradeoff

Suppose partition occurs:

```text
Server A ↔ Server B disconnected
```

Now user writes data to:

```text
Server A
```

Question:

> What should Server B do?

Two choices exist:

* preserve consistency
* preserve availability

Cannot fully guarantee both during partition.

---

# 8. CP Systems (Consistency + Partition Tolerance)

CP systems prioritize:

* correctness
* consistency
* strong guarantees

during partitions.

---

# How CP Works

If replicas cannot synchronize:

* some requests rejected
* system may become partially unavailable

Reason:

> serving stale data would violate consistency.

---

# Real World Example — Banking

Suppose account balance updated in Mumbai.

Singapore replica disconnected.

Would you allow Singapore to serve stale balance?

Usually:

```text
NO
```

Better to reject request temporarily than show incorrect balance.

---

# CP Systems Examples

* HBase
* ZooKeeper
* etcd
* MongoDB (strong consistency mode)

---

# Use Cases

Use CP when:

* correctness critical
* invariants must never break

Examples:

* banking
* inventory systems
* payment systems
* distributed locks

---

# 9. AP Systems (Availability + Partition Tolerance)

AP systems prioritize:

* uptime
* responsiveness
* availability

during partitions.

---

# How AP Works

Even if replicas disconnected:

* system continues serving requests
* stale data may be returned

---

# Real World Example — Social Media

Suppose Instagram like count delayed:

```text
100 → 101
```

Some replicas may temporarily still show:

```text
100
```

Usually acceptable.

Users prefer:

* service available
  over
* perfect consistency

---

# AP Systems Examples

* Cassandra
* DynamoDB
* CouchDB

---

# Use Cases

Use AP when:

* availability more important
* eventual consistency acceptable

Examples:

* social feeds
* analytics
* recommendation systems
* chat systems

---

# 10. Why CA Systems Don’t Really Exist

CA means:

* Consistency
* Availability
* NO partition tolerance

But in distributed systems:

```text
Partitions are unavoidable
```

Therefore practical distributed systems must handle:

```text
Partition Tolerance
```

So real systems usually become:

* CP
  OR
* AP

during failures.

---

# 11. Eventual Consistency

Very important AP concept.

Eventual consistency means:

> replicas eventually converge to same value.

Not immediately,
but eventually.

---

# Example

Like count updated:

```text
100 → 101
```

Some replicas temporarily show:

```text
100
```

After synchronization:
all replicas show:

```text
101
```

---

# 12. Why Large Internet Systems Prefer AP

Because:

* global outages unacceptable
* uptime critical
* users tolerate small inconsistencies

Examples:

* Instagram
* Facebook
* Twitter
* YouTube feeds

---

# 13. Why Financial Systems Prefer CP

Because:

* stale data dangerous
* correctness critical
* invariants must never break

Examples:

* banking
* trading systems
* payment ledgers
* inventory systems

---

# 14. Important Interview Insight

CAP tradeoff matters specifically:

> during network partitions.

Without failures,
systems can often provide:

* consistency
* availability

simultaneously.

---

# 15. Quorum Concept (Important)

Suppose:

```text
3 replicas
```

Write succeeds only if:

```text
2 replicas acknowledge
```

Read succeeds from:

```text
2 replicas
```

This balances:

* consistency
* availability

Used heavily in:

* Cassandra
* DynamoDB

---

# 16. CAP in Microservices

Microservices often choose:

* eventual consistency
* asynchronous communication

because distributed ACID transactions are expensive.

Example:

* Order Service
* Payment Service
* Inventory Service

may temporarily disagree during failures.

This is accepted tradeoff.

---

# 17. Real Interview Thought Process

## Banking System

Think:

* correctness critical
* stale data unacceptable

Likely:

```text
CP
```

---

## Social Media Feed

Think:

* uptime critical
* stale likes acceptable

Likely:

```text
AP
```

---

# 18. Common Interview Misconception

Wrong understanding:

```text
Pick any 2 of 3
```

Correct understanding:

> Partition tolerance is mandatory in distributed systems, so the real tradeoff is Consistency vs Availability during failures.

---

# 19. Final Mental Model

## Consistency

```text
All replicas see same data immediately
```

---

## Availability

```text
Every request receives response
```

---

## Partition Tolerance

```text
System survives network failures
```

---

# 20. Final Revision Summary

| Type | Prioritizes  | Sacrifices            | Example Use Cases   |
| ---- | ------------ | --------------------- | ------------------- |
| CP   | Correctness  | Availability          | Banking, inventory  |
| AP   | Availability | Immediate consistency | Social media, feeds |

---

# 21. Senior-Level Insight

CAP theorem fundamentally exists because:

> distributed systems must make correctness vs availability decisions when network communication becomes unreliable.
