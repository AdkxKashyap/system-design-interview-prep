# Sharding — System Design Interview Notes

# 1. Introduction

One of the MOST important scaling concepts in system design is understanding:

```text
Replication → solves READ scaling
Sharding → solves WRITE scaling
```

This is an extremely important interview insight.

Most systems initially scale using:

* bigger machines
* replication
* caching

But eventually:

> single database cannot handle write throughput anymore.

This is where sharding becomes necessary.

---

# 2. The Real Problem Sharding Solves

Suppose you built Instagram.

Initially:

```text
1 database server
```

works fine.

Then product grows:

* hundreds of millions of users
* billions of posts
* billions of likes/comments
* massive writes/sec

Single DB starts struggling:

* CPU overloaded
* storage huge
* indexes massive
* write latency increases
* locks increase
* replication lag grows

Eventually:

```text
One machine cannot handle workload
```

Vertical scaling hits physical limits.

Adding:

* more RAM
* bigger CPU
* faster SSD

cannot scale infinitely.

---

# 3. What is Sharding?

Sharding means:

> splitting data across MULTIPLE database servers.

Instead of:

```text
1 huge DB storing everything
```

we distribute data:

```text
Shard1 → Some users
Shard2 → Some users
Shard3 → Some users
```

Each shard stores:

```text
PART of total dataset
```

---

# 4. Beginner-Friendly Analogy

Imagine library.

Initially:

```text
1 bookshelf
```

stores all books.

Eventually:

* too many books
* shelf overloaded

Solution:

* Shelf A-F
* Shelf G-M
* Shelf N-Z

Storage distributed.

This is essentially:

```text
Sharding
```

---

# 5. Most Important Mental Model

## Replication

```text
Same data copied to many machines
```

Goal:

* availability
* read scaling

---

## Sharding

```text
Different data stored on different machines
```

Goal:

* write scaling
* storage scaling

---

# 6. Real Production Example

Suppose user table:

| user_id | name  |
| ------- | ----- |
| 1       | Akash |
| 2       | John  |
| 3       | Emma  |

Initially:
all users stored in one DB.

At scale:

```text
500 million users
```

System shards users:

```text
Shard1 → user_id 1–100M
Shard2 → user_id 100M–200M
Shard3 → user_id 200M–300M
```

Now:

* writes distributed
* storage distributed
* indexing distributed

Huge scalability improvement.

---

# 7. Shard Key (VERY Important)

Shard key determines:

> where data lives.

Without shard key:
system cannot know:

```text
Which shard stores data?
```

---

# Example

Suppose shard key:

```text
user_id
```

System may use:

```text
user_id % 3
```

to determine shard.

Example:

| user_id | shard  |
| ------- | ------ |
| 1       | Shard1 |
| 2       | Shard2 |
| 3       | Shard0 |

This is:

```text
Hash-Based Sharding
```

---

# 8. Why Choosing Good Shard Key is CRITICAL

Bad shard key creates:

* hotspots
* uneven traffic
* uneven storage

---

# Bad Example

Suppose Instagram shards by:

```text
country
```

Now:

```text
India shard
```

becomes overloaded.

Smaller countries underutilized.

This creates:

```text
Hotspotting / Data Skew
```

Very common interview discussion.

---

# 9. Good Shard Key Characteristics

A good shard key should:

* distribute traffic evenly
* distribute storage evenly
* avoid hotspots
* have high cardinality
* align with query patterns

Example:

```text
user_id
```

usually works well.

---

# 10. Common Sharding Strategies

For SDE2 interviews focus mainly on:

* Range-based sharding
* Hash-based sharding

---

# 11. Range-Based Sharding

Data split by ranges.

Example:

```text
Shard1 → user_id 1–1000
Shard2 → user_id 1001–2000
Shard3 → user_id 2001–3000
```

---

## Advantages

* simple
* range queries efficient

Example:

```sql
SELECT * FROM users
WHERE user_id BETWEEN 1000 AND 2000
```

Only one shard needed.

---

## Problem

New writes often hit:

```text
last shard
```

creating hotspot.

---

# 12. Hash-Based Sharding

Use:

```text
hash(user_id) % N
```

to distribute data.

---

## Advantages

* traffic evenly distributed
* avoids hotspots

---

## Problem

Range queries become difficult.

Query may need to hit:

```text
multiple shards
```

This is:

```text
Scatter-Gather Query
```

which is expensive.

---

# 13. Visual Understanding

## Without Sharding

```text
                ┌────────────┐
                │  Single DB │
                └────────────┘
                       ↑
             ALL reads/writes
```

Single bottleneck.

---

## With Sharding

```text
          ┌────────────┐
          │ Router/App │
          └─────┬──────┘
                │
      ┌─────────┼─────────┐
      ↓         ↓         ↓
 ┌────────┐ ┌────────┐ ┌────────┐
 │Shard 1 │ │Shard 2 │ │Shard 3 │
 └────────┘ └────────┘ └────────┘
```

Load distributed.

---

# 14. Routing Requests to Correct Shard

Application/router computes:

```text
hash(user_id)
```

Then determines:

```text
which shard stores data
```

This routing layer is extremely important.

---

# 15. Resharding (Very Important)

Suppose system grows again.

Need:

```text
Shard4
```

Now:

* existing data must move
* routing changes
* rebalancing required

This process:

```text
Resharding
```

---

# Why Resharding Difficult

Moving TBs/PBs of data:

* expensive
* operationally risky
* may cause downtime
* may increase latency

Real production challenge.

---

# 16. Cross-Shard Joins Become Hard

Suppose:

* users table on Shard1
* orders table on Shard3

Now joins become:

* distributed
* network-heavy
* slow

This is why large systems often:

* denormalize data
* avoid joins

---

# 17. Distributed Transactions Become Hard

Suppose transaction touches:

* Shard1
* Shard2

Now distributed transaction required.

Distributed transactions:

* slower
* operationally complex
* coordination-heavy

---

# 18. Replication + Sharding Together

Very important interview concept.

Large-scale systems need BOTH:

* replication
* sharding

because social media systems are BOTH:

* read heavy
* write heavy

---

# Why Replication Needed

Replication helps:

```text
Read scaling
High availability
Fault tolerance
```

---

# Why Sharding Needed

Sharding helps:

```text
Write scaling
Storage scaling
```

---

# 19. Real Production Architecture

```text
                    ┌────────────────┐
                    │ App Servers    │
                    └────────┬───────┘
                             │
         ┌───────────────────┼───────────────────┐
         ↓                   ↓                   ↓

   ┌────────────┐     ┌────────────┐     ┌────────────┐
   │ Shard 1    │     │ Shard 2    │     │ Shard 3    │
   │ Primary DB │     │ Primary DB │     │ Primary DB │
   └─────┬──────┘     └─────┬──────┘     └─────┬──────┘
         │                  │                  │
    ┌────┴────┐        ┌────┴────┐        ┌────┴────┐
    ↓         ↓        ↓         ↓        ↓         ↓

 Replica   Replica   Replica   Replica   Replica   Replica
```

---

# 20. What Happens Here?

System first:

```text
SHARDS DATA
```

Example:

```text
Shard1 → users 1–100M
Shard2 → users 100M–200M
Shard3 → users 200M–300M
```

Then:

```text
each shard gets replicas
```

for:

* read scaling
* failover
* availability

This is extremely common production architecture.

---

# 21. Cache vs Replicas

Very important interview topic.

Many beginners think:

> "If we add cache, replicas unnecessary."

Wrong.

Cache and replicas solve DIFFERENT problems.

---

# 22. What Cache Solves

Cache helps:

```text
Reduce repeated read load
Improve latency
```

Usually implemented using:

* Redis
* Memcached
* CDN

---

# 23. Why Replicas Still Needed Even with Cache

Because cache is:

* not source of truth
* not permanent storage
* not guaranteed hit
* can fail/restart

Cache misses still hit DB.

---

# 24. Cache Miss Example

Suppose:

```text
Post #123
```

not found in cache.

Request goes to:

```text
Database replica
```

NOT necessarily primary DB.

---

# 25. Cache Stampede / Thundering Herd

Suppose celebrity uploads viral reel.

Initially:

```text
cache empty
```

Millions of requests suddenly arrive.

Many requests simultaneously hit DB before cache warms up.

This is:

```text
Cache Stampede
```

Replicas help absorb this traffic.

---

# 26. Some Reads Cannot Be Cached

Examples:

* bank balance
* inventory count
* live bidding systems
* strongly consistent reads

These still require:

```text
database replicas
```

---

# 27. Real Production Architecture with Cache

```text
                Users
                   ↓
              Load Balancer
                   ↓
             App Servers
                   ↓
                Cache
             (Redis/CDN)
                   ↓
          ┌────────┴────────┐
          ↓                 ↓
     Read Replicas      Primary DB
```

---

# 28. What Happens in This Architecture?

## Step 1

Application checks:

```text
Cache first
```

---

## Step 2A — Cache Hit

Data returned immediately.

Fast.

DB untouched.

---

## Step 2B — Cache Miss

Request goes to:

```text
Read Replica
```

---

## Step 3 — Writes

Writes go to:

```text
Primary DB
```

Then:

* replicas updated
* cache invalidated/updated

---

# 29. Important Production Insight

Cache reduces:

```text
READ FREQUENCY
```

But does NOT reduce:

```text
WRITE VOLUME
```

Social media still generates:

* likes
* comments
* uploads
* messages

Need:

* sharding
* replication

regardless of cache.

---

# 30. Final Mental Model

## Replication

```text
Same data on multiple machines
```

Purpose:

* high availability
* fault tolerance
* read scaling

---

## Sharding

```text
Different data on different machines
```

Purpose:

* write scaling
* storage scaling

---

## Cache

```text
Temporary fast-access layer
```

Purpose:

* reduce repeated reads
* improve latency

---

# 31. Final Revision Summary

| Concept     | Main Goal                       |
| ----------- | ------------------------------- |
| Replication | Read scaling + availability     |
| Sharding    | Write scaling + storage scaling |
| Cache       | Reduce repeated DB reads        |

---

# 32. Senior-Level Insight

At internet scale:

```text
Replication + Sharding + Caching
```

are complementary techniques, not competing techniques.

Large-scale systems usually need:

* caching for latency reduction
* replication for availability/read scaling
* sharding for write/storage scaling
