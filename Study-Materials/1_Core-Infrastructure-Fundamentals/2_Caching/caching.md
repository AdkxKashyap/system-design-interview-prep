# Caching — Deep Dive for System Design Interviews

Caching is one of the MOST IMPORTANT topics in system design interviews.

Interviewers ask caching questions in almost every design:
- social media
- e-commerce
- messaging
- streaming
- search systems
- ride sharing
- payment systems

Why?

Because scaling modern systems WITHOUT caching is almost impossible.

For senior engineer interviews, you must understand:
- WHY caching exists
- WHERE to cache
- WHAT to cache
- WHEN NOT to cache
- cache invalidation tradeoffs
- consistency problems
- operational concerns

Most candidates only say:

> “Use Redis.”

That is NOT enough.

Interviewers want to know:
- what bottleneck you are solving
- how caching changes system behavior
- what consistency guarantees are acceptable
- what happens during cache failures
- how cache affects scaling

---

# 1. The REAL Purpose of Caching

Most beginners think:

> “Caching makes systems faster.”

That is only partially true.

Senior-level understanding:

Caching primarily exists to:
- reduce expensive computations
- reduce database load
- reduce network calls
- reduce latency
- improve throughput
- absorb traffic spikes

The BIGGEST reason in production:
> protecting backend systems from overload.

---

# 2. Real Engineering Motivation

Suppose you build Instagram.

```text
100 million users open same post

Without cache:

100 million DB reads

Database dies.

With cache:

Cache serves repeated reads
DB load drastically reduced

This is why caching is fundamental.
```

# 3. What Exactly Is a Cache?

A cache is:

a high-speed storage layer storing frequently accessed data.

Usually:

- memory-based
- extremely fast
- temporary

Important Tradeoff

Cache gives:

speed

But introduces:

consistency complexity

This tradeoff is CENTRAL to caching discussions.

# 4. Why Databases Alone Are Not Enough

Interviewers LOVE this discussion.

Database Reads Are Expensive

Reasons:

- disk I/O
- network latency
- query execution
- locks
- joins
- indexing overhead

Even optimized DB queries are slower than memory.

Example Latency

| Operation | Latency |
| --- | --- |
| CPU Cache | nanoseconds |
| RAM | microseconds |
| SSD | milliseconds |
| DB Query | milliseconds to seconds |

Huge difference.

Real Engineering Thought Process

If:

same data repeatedly accessed

then:

store in faster layer

That faster layer is cache.

# 5. Types of Caching

This is VERY important.

Interviewers expect:

multiple cache layers
understanding of tradeoffs

### A. Client-Side Cache

Stored in:

- browser
- mobile app
- frontend memory

Examples:

- browser cache
- CDN browser cache
- mobile local storage

**Use Cases**

Good for:

static assets
images
CSS
JS bundles

Why Useful?

Avoids even reaching backend.

Fastest possible cache.

Real Example

Browser caches:

logo.png

Future requests served locally.

No server hit.

Interview Insight

Client-side cache reduces:

bandwidth
backend traffic
latency

Massively important at scale.

### B. CDN Cache (Edge Cache)

One of the MOST important production caches.

Problem

Suppose users worldwide request:

video.mp4

Without CDN:

every request hits origin server

Very expensive.

Solution

Store content near users geographically.

Architecture
User
 ↓
CDN Edge Server
 ↓
Origin Server

Why Powerful?

User in:

India
US
Europe

gets content from nearby edge.

Reduces:

latency
backbone traffic
origin load

Best For
images
videos
static assets
large downloads

Real Systems
Netflix
YouTube
Cloudflare
Akamai

### C. Application Cache

Most discussed interview cache.

Stored between:

application
database

Usually:

Redis
Memcached

Architecture
App Server
   ↓
Cache
   ↓
Database

Why Important?

Database queries expensive.

Frequently accessed data cached.

Best For
user profiles
sessions
timelines
product details
recommendations

Real Example

Instagram user profile:

user:123

instead of querying DB every time:

cache profile in Redis

### D. Database Cache

Some databases internally cache:

pages
query plans
indexes

Example:

MySQL buffer pool
PostgreSQL shared buffers

Important Insight

Even databases use caching internally.

Because disk access is expensive.

### E. Distributed Cache

Critical at scale.

Problem

Single cache server becomes bottleneck.

Need:

multiple cache nodes
Solution

Distributed cache cluster.

Examples:

Redis Cluster
Memcached Cluster

Challenge Introduced

Now we need:

partitioning
consistent hashing
replication

This connects directly with previous topic.

---


# 6. Cache Access Patterns

This is one of the MOST IMPORTANT interview areas.

### A. Cache Aside (*Lazy Loading*)

MOST COMMON pattern.

Also called:

- Lazy Loading

Flow
1. Check cache
2. If miss → query DB
3. Store result in cache
4. Return response

Example
*GET user:123*

Flow:

```
Cache miss
   ↓
DB query
   ↓
Store in cache
   ↓
Return data
```

- Why Popular?

   - Application controls caching logic.

   - Simple and flexible.

- Major Advantage

   - Only frequently accessed data cached.

   - Efficient memory usage.

- Major Problem

   - First request slow:

   - cache miss penalty

- Real Production Concern

   - Cache stampede.

   - Suppose:

      - popular cache expires

      - Millions of requests simultaneously hit DB.

      - DB overload happens.

VERY important interview topic.

- When To Use

   - Best default strategy for:

   - read-heavy systems
   - social media
   - e-commerce
   - APIs

**Interview Insight**

- Most production systems use some form of cache-aside.

### B. Read Through Cache

- Cache itself fetches data from DB.

- Application only talks to cache.

Flow:
```
App
 ↓
Cache
 ↓
DB
```
Advantage

- Simpler application logic.

- Problem

   - Less flexibility.

   - Application loses control.

- Common Use Cases

   - Used in:

   - managed caching layers
   - ORM abstractions

Less common in interviews compared to cache-aside.

### C. Write Through Cache

- Write goes:

   - cache + DB simultaneously

- Why Useful?

   - Cache always fresh after write.

- Problem

   - Higher write latency.

   - Every write waits for:

      - cache update
      - DB write

- Best For

   - Systems requiring:

      - strong consistency
      - high read frequency

   - Example

   - User profile updates.

   - Need cache immediately updated.

### D. Write Back Cache (Write Behind)

- Write goes ONLY to cache initially.

- DB updated asynchronously later.

- Why Powerful?

   - Very fast writes.

- Problem

   - Risk of data loss.

   - If cache crashes before DB flush:

      - data lost

   - Use Cases

   - Useful for:

      - analytics
      - logging
      - metrics ingestion

      - NOT ideal for financial systems.

### E. Refresh Ahead Cache

- Cache refreshes proactively before expiration.

- Why Useful?

   - Avoids:

      - cache misses
      - latency spikes

- Good For

   - Highly predictable traffic:

   - trending videos
   - popular feeds
   
---

# 7. Cache Eviction Policies

- Cache memory limited.

- Need eviction strategy.


### A. LRU (Least Recently Used)

- Remove least recently accessed item.

- Why Effective?

   - Assumes:

      - recently used data likely reused soon

      - Usually true.

- Widely Used In

   - Redis
   - OS memory management
   - browser caches

- Example
   - A accessed recently
   - B not accessed long time

   - *Evict:*

      - **B**

### B. LFU (Least Frequently Used)

- Remove least frequently accessed item.

- *Better For:*

   - Stable access patterns.

- Example:

   - trending content

- Problem

   - More tracking overhead.

### C. FIFO

- Remove oldest item.

- Simple but less intelligent.

- Rarely ideal.

### D. TTL (Time To Live)

- Expire data after fixed duration.

- Very important production strategy.

- Example:
   - user_profile TTL = 5 mins

- After expiration:

   - removed automatically

- Why Important?

   - Prevents:

      - stale data forever

# 8. Cache Invalidation — THE HARDEST PART

- Famous quote:

*“There are only two hard things in Computer Science:
cache invalidation and naming things.”*

- Core Problem

   - Cache introduces:

   - duplicate copies of data

   - Now:

      - DB updated
      - Cache stale

      - Consistency issue appears.

- Example

   - User changes username.

   - DB:

   - new_name

   - Cache:

   - old_name

   - Users see stale data.

# 9. Cache Invalidation Strategies

### A. TTL Expiration

   - Simplest strategy.

   - Advantage

      - Simple.

   - Problem

      - Stale data until expiration.

      - Best For

      - Eventually consistent systems:

      - feeds
      - analytics
      - recommendations

### B. Write Invalidate

   - When DB updated:

      - delete cache entry

      - Next read:

         - fetch fresh data

   - Very Common

   - Used heavily in production.

   - Simple and reliable.

### C. Write Update

   - Update cache immediately after DB write.

   - Advantage

   - Fresh cache.

   - Problem

      - More write complexity.

# 10. Cache Consistency Tradeoffs

   - Strong Consistency

   - Cache always latest.

   - Difficult and expensive.

   - Eventual Consistency

      - Small stale window acceptable.

      - Most internet systems choose this.

      - Because:

      - scalability more important
      - perfect consistency expensive

      - Example:

      - Instagram like count delayed few seconds:

      - acceptable

      - Bank balance stale:

      - unacceptable

## 11. Cache Stampede (VERY IMPORTANT)

   - Problem:

      - Popular key expires.

      - Millions of requests:

      - cache miss simultaneously

      - All hit DB.

      - DB collapses.

   - Example
      - celebrity_post

      - expires during peak traffic.

      - Massive backend overload.

      - Detailed Solutions and Patterns

Below are widely used techniques (mixing them gives the strongest production resilience):

### A. Request Coalescing / Single-flight

- Only a single request performs the DB/cache rebuild while others wait for result.

- Implementation notes:

- Use in-memory single-flight (per app instance) or distributed single-flight (across instances) via a shared lock service.
- Libraries: Go's `singleflight`, Java/C# equivalents, or custom mutex maps.

- Pseudocode (single app-instance single-flight):

```pseudo
if cache.miss(key):
  if singleflight.start(key):
    value = db.query(key)
    cache.set(key, value)
    singleflight.done(key, value)
  else:
    value = singleflight.wait(key)
return value
```

### B. Distributed Locking with Short TTL

Use a distributed lock (Redis SETNX with TTL, ZooKeeper, etc.) so only one worker rebuilds the cache. Keep lock TTL short and robust to crashes.

Considerations:

- Ensure lock TTL > expected rebuild time or implement lock renewal.
- Always have fallback path (serve stale data) if lock holder fails.

C. Serve Stale While Revalidate (Stale-While-Revalidate)

Return slightly stale cached value immediately and kick off an async refresh to update cache in background. This avoids blocking requests and shields DB.

Behavior:

- Read: if cache expired but value exists, return it and asynchronously refresh.
- If no cached value, fall back to single-flight/lock to rebuild.

D. TTL Jitter / Staggered Expiration

Add randomized jitter to TTLs so not all keys expire at the same instant.

Example: `TTL = base_ttl + random(-jitter, +jitter)`

E. Pre-warming / Refresh Ahead

Predict hot keys and refresh them before TTL expires (cron, background worker, or based on access patterns).

F. Rate-Limit Rebuilds and Circuit Breaker

Throttle the rate of background rebuilds when DB is under pressure; fall back to serving stale data or returning errors.

G. Negative Caching

Cache negative results for short duration to avoid repeated DB hits for missing data.

H. Use of Local L1 Cache in Front of Distributed Cache

Keep a small local in-process cache (L1) to absorb read spikes and reduce pressure on shared Redis (L2). Local caches reduce network hops and provide very low latency.

I. Priority Queue For Rebuilds

For very large numbers of expirations, push rebuild jobs to a prioritized queue so the most critical keys are rebuilt first.

J. Example Combined Strategy (robust in production)

- L1 in-process cache for microbursts
- L2 distributed cache (Redis Cluster)
- Cache-aside reads
- On miss: try single-flight; acquire distributed lock; if lock unavailable, serve stale or wait with timeout
- TTL jitter + refresh-ahead for known hot keys
- Negative caching for non-existent keys

These combined approaches prevent DB overload even under extreme traffic.

## 12. Hot Keys Problem

Another REAL production issue.

Problem

One key receives enormous traffic.

Example:

celebrity_profile

One cache node overloaded.

Solutions

- replication
- local caches
- request fanout
- sharding hot data

# 13. Distributed Cache Challenges

At scale:

single Redis node insufficient

Need:

clustering
replication
partitioning

Important Concepts
| Concept | Why |
| --- | --- |
| Consistent Hashing | Stable partitioning |
| Replication | High availability |
| Sharding | Horizontal scaling |
| Failover | Reliability |

# 14. Redis vs Memcached

VERY common interview question.

Redis

Supports:

- persistence
- replication
- pub/sub
- advanced data structures

Used heavily today.

Memcached

Simpler:

- pure in-memory KV store

Extremely fast.

Less feature rich.

Interview Insight

Redis dominates modern architectures.

# 15. Multi-Level Caching

Senior-level topic.

Large systems use multiple cache layers.

Example
Browser Cache
    ↓
CDN Cache
    ↓
Application Cache
    ↓
Database Cache
    ↓
Database

Why?

Each layer reduces load further down.

Huge scalability benefit.

# 16. What Should You Cache?

VERY important engineering decision.

Good Candidates
read-heavy data
expensive queries
repeated computations
popular content

Bad Candidates
highly volatile data
sensitive transactional data
rapidly changing counters

Example

Good:
product catalog

Bad:
bank account balance

# 17. Real Interview Thought Process

Suppose interviewer says:

“Design Twitter timeline.”

Senior thinking:

Step 1

Reads >> writes?

YES.

Caching important.

Step 2

What data hot?

timelines
user profiles
tweets

Step 3

Where cache?

CDN for media
Redis for timelines
browser cache for static assets

Step 4

Consistency requirements?

Timeline slightly stale acceptable.

Strong consistency unnecessary.

Step 5

Potential issues?

cache stampede
hot celebrities
invalidation complexity

# 18. Common Interview Questions

Q1. Why Not Cache Everything?

Memory expensive.
Stale data risk.
Low-value data wastes cache.

Q2. What Happens If Cache Fails?

Traffic shifts to DB.

Need:

gracious degradation
rate limiting
fallback mechanisms

Q3. Why Redis Instead Of Database?

Memory access much faster than disk/database queries.

Q4. Why TTL Needed?

Prevent stale data forever.

Q5. What Happens During Cache Miss Storm?

Backend overload.

Need:

coalescing
locks
staggered TTLs

# 19. Senior Engineer Mindset

Junior engineer says:

> “Use Redis cache.”

Senior engineer says:

> “We have a read-heavy workload with repeated access patterns. Introducing Redis reduces database amplification and absorbs traffic spikes, but we must carefully handle cache invalidation, hot keys, and cache stampedes.”

That is production engineering thinking.

# 20. Final Revision Notes

Why Caching Exists
- reduce DB load
- reduce latency
- improve throughput
- absorb spikes

MOST Important Cache Types

| Cache Type | Best For |
| --- | --- |
| Browser Cache | Static frontend assets |
| CDN Cache | Global content delivery |
| Application Cache | APIs & DB acceleration |
| Distributed Cache | Large-scale systems |

MOST Important Patterns

| Pattern | Use Case |
| --- | --- |
| Cache Aside | Default production choice |
| Write Through | Stronger consistency |
| Write Back | Fast writes |
| Refresh Ahead | Predictable traffic |

MOST Important Problems

| Problem | Solution |
| --- | --- |
| Cache Stampede | Coalescing, locks, stale-while-revalidate |
| Hot Keys | Replication, sharding |
| Stale Data | TTL, invalidation |
| Scaling | Distributed cache |

Senior-Level Insight

Caching is fundamentally about:

trading consistency complexity for scalability and performance.
