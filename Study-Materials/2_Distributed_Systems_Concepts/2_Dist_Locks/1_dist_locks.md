# Distributed Locks

# 1. Why Do We Need Distributed Locks?

Distributed locks solve coordination problems in distributed systems. Fix issues due to race conditions.

The moment you have:

```text
Multiple Servers
```

you need a way to ensure:

```text
Only One Server Performs Certain Work
```

at a given time.

Without coordination, duplicate processing can occur.

---

# Example Problem

Suppose Order Service runs on:

```text
Server1
Server2
Server3
```

All three consume messages from Kafka.

A message arrives:

```text
Process Order #123
```

Due to:

* Retries
* Consumer Rebalancing
* Duplicate Delivery
* Bugs

multiple servers may process:

```text
Order #123
```

simultaneously.

Result:

```text
Customer Charged Twice
```

Distributed locks prevent this.

---

# 2. What Is a Distributed Lock?

Think of a bathroom key.

Only one person can hold it at a time.

Workflow:

```text
Acquire Lock
      ↓
Do Work
      ↓
Release Lock
```

Everyone else waits.

---

# Distributed System Version

```text
Server1
Server2
Server3
```

all try to acquire:

```text
lock:order:123
```

Only one succeeds.

---

# Example

Server1:

```text
Acquire Lock
```

Success.

---

Server2:

```text
Acquire Lock
```

Fails.

---

Server3:

```text
Acquire Lock
```

Fails.

---

Only Server1 processes the order.

---

# 3. Real World Use Cases

# A. Prevent Duplicate Payments

Suppose user double-clicks:

```text
Pay Now
```

Requests arrive simultaneously:

```text
Payment Request 1
Payment Request 2
```

Without locking:

```text
Customer Charged Twice
```

---

With lock:

```text
lock:user:123:payment
```

Only one request proceeds.

---

# B. Inventory Reservation

Suppose:

```text
1 iPhone Left
```

Users:

```text
User A
User B
```

purchase simultaneously.

Without locking:

```text
Inventory = -1
```

Overselling occurs.

---

With lock:

```text
lock:product:iphone
```

Only one reservation succeeds.

---

# C. Scheduled Jobs

Suppose nightly billing job runs on:

```text
Server1
Server2
Server3
```

Without lock:

```text
Billing Executes 3 Times
```

Disaster.

---

With lock:

```text
lock:billing-job
```

Only one server executes the job.

---

# 4. Requirements of a Good Distributed Lock

## Mutual Exclusion

Guarantee:

```text
Only One Owner
```

at a time.

---

## Fault Tolerance

Suppose server acquires lock and crashes.

Lock cannot remain forever.

Need:

```text
Automatic Expiration
```

---

## Performance

Lock acquisition should be:

```text
Fast
```

because it may happen millions of times.

---

# 5. Redis Locks

Most common interview answer.

---

# Why Redis?

Redis provides:

```redis
SET key value NX EX 30
```

---

# Meaning

```text
NX = Set Only If Key Doesn't Exist
EX = Expire After 30 Seconds
```

---

# Example

```redis
SET lock:order:123 server1 NX EX 30
```

---

# Case 1

Key does not exist.

Result:

```text
Lock Acquired
```

---

# Case 2

Key already exists.

Result:

```text
Lock Acquisition Failed
```

Another server owns lock.

---

# Why Expiration Is Important

Suppose:

```text
Server1 Acquires Lock
```

then crashes.

Without expiration:

```text
Lock Never Released
```

Deadlock.

---

With:

```redis
EX 30
```

lock automatically disappears after 30 seconds.

---

# Redis Lock Workflow

```text
Acquire Lock
      ↓
Process Work
      ↓
Delete Lock
```

---

# Example

Acquire:

```redis
SET lock:payment server1 NX EX 30
```

Process:

```text
Charge Customer
```

Release:

```redis
DEL lock:payment
```

---

# Problem With Naive Redis Locks

Suppose:

```text
Lock Expires
```

before work completes.

Now:

```text
Server2 Acquires Same Lock
```

while Server1 still processing.

Duplicate execution occurs.

---

# Common Solutions

## Lock Renewal

Periodically extend lock expiration.

Also called:

```text
Heartbeat
```

---

## Redlock

Redis distributed lock algorithm.

Provides stronger guarantees.

Note:

Redlock is somewhat controversial in distributed systems discussions.

For interviews, simple Redis locks are usually sufficient.

---

# 6. ZooKeeper Locks

Before Redis became popular, many systems used:

```text
Apache ZooKeeper
```

for distributed coordination.

---

# What Is ZooKeeper?

ZooKeeper's primary job is:

```text
Coordination
```

Not caching.

Not database storage.

---

# Common ZooKeeper Uses

* Distributed Locks
* Leader Election
* Service Discovery
* Configuration Management

---

# ZooKeeper Lock Flow

Suppose servers want:

```text
/locks/order123
```

---

Servers create:

```text
/locks/order123/node1
/locks/order123/node2
/locks/order123/node3
```

---

ZooKeeper assigns sequence numbers:

```text
/locks/order123/0001
/locks/order123/0002
/locks/order123/0003
```

---

Smallest sequence number wins.

Winner:

```text
0001
```

gets lock.

---

When winner finishes:

```text
Delete Node
```

Next node automatically acquires lock.

---

# Why ZooKeeper Locks Are Stronger

ZooKeeper provides:

* Strong Consistency
* Ordering Guarantees
* Reliable Leader Election

---

Trade-offs:

* More Complex
* Slower Than Redis
* Operationally Heavy

---

# 7. Redis vs ZooKeeper

| Redis                      | ZooKeeper                      |
| -------------------------- | ------------------------------ |
| Simple                     | Complex                        |
| Fast                       | Slower                         |
| Easy Setup                 | Operationally Heavy            |
| Caching + Locks            | Coordination Only              |
| Best For Most Applications | Strong Coordination Guarantees |

---

# 8. Distributed Lock vs Database Lock

Many engineers confuse these concepts.

---

# Database Lock

Coordinates access within:

```text
Single Database
```

Examples:

```text
Rows
Tables
Transactions
```

---

# Distributed Lock

Coordinates access across:

```text
Multiple Servers
Multiple Services
```

throughout an entire distributed system.

---

# 9. Common Interview Use Cases

## Kafka Duplicate Processing

```text
lock:event:123
```

Ensures only one consumer processes event.

---

## Payment Processing

```text
lock:payment:user123
```

Prevents double charging.

---

## Inventory Reservation

```text
lock:product:iphone
```

Prevents overselling.

---

## Scheduled Jobs

```text
lock:billing-job
```

Ensures only one scheduler executes.

---

## Leader Election

Choose:

```text
One Active Leader
```

among multiple servers.

---

# 10. Important Senior-Level Insight

Many engineers overuse distributed locks.

Before introducing a lock, ask:

> Can I avoid coordination entirely?

Often better solutions exist:

* Idempotency Keys
* Optimistic Locking
* Unique Constraints
* Compare-And-Swap (CAS)
* Atomic Database Operations

Distributed locks should be used only when true mutual exclusion is required.

---

# Interview Answer Template

If asked:

> How would you prevent duplicate processing of the same message?

Strong answer:

> First I would try to make processing idempotent. If strict mutual exclusion is required across multiple service instances, I would use a distributed lock. A common implementation is Redis using `SET key value NX EX ttl`, where only one worker acquires the lock. After processing completes, the lock is released or automatically expires if the worker crashes.

---

# Senior Engineer Mental Model

Think of distributed locks as:

```text
Multiple Servers
        ↓
Need Exclusive Ownership
        ↓
Acquire Lock
        ↓
Perform Work
        ↓
Release Lock
```

The most common real-world uses are:

* Prevent duplicate payment processing
* Prevent duplicate Kafka message handling
* Prevent overselling inventory
* Ensure only one scheduler executes
* Leader election in distributed systems

---

# Final Takeaway

Redis locks are the most common practical solution.

ZooKeeper is the classic distributed coordination system offering stronger consistency guarantees.

Use distributed locks when:

```text
Only One Server Must Perform Work
```

and simpler approaches such as idempotency or database constraints are insufficient.
