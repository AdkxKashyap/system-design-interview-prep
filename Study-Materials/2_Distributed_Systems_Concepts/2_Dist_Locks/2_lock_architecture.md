# Distributed Locks vs Redis Cache

# 1. A Common Confusion

Many engineers initially think:

```text
Acquire Lock
      ↓
Check Cache
      ↓
Check DB
```

and assume locks and cache are part of the same workflow.

They are not.

A distributed lock and a cache solve completely different problems.

---

# 2. Redis Cache vs Redis Lock

Redis is simply a key-value store.

The same Redis cluster can be used for:

* Caching
* Distributed Locks

but they serve different purposes.

---

# Redis Used As Cache

Example:

```text
user:123
   ↓
{
  "name": "Akash"
}
```

Purpose:

```text
Speed Up Reads
```

Question cache answers:

```text
Do I already have this data?
```

---

# Redis Used As Lock Store

Example:

```text
lock:user:123
   ↓
server-1
```

Purpose:

```text
Coordination
```

Question lock answers:

```text
Am I allowed to perform this work?
```

---

# Visual Comparison

## Cache

```text
user:123
   ↓
User Object
```

Stores business data.

---

## Lock

```text
lock:user:123
   ↓
server-1
```

Stores ownership information.

---

# 3. Where Does the Lock Live?

Suppose Order Service runs on:

```text
Order Pod 1
Order Pod 2
Order Pod 3
```

Architecture:

```text
            Redis

              ▲
              │

     ┌────────┼────────┐
     ▼        ▼        ▼

 Order1   Order2   Order3
```

The lock exists:

```text
Inside Redis
```

NOT inside service memory.

---

# Why Not Store Lock In Memory?

Suppose:

```java
boolean locked = true;
```

inside Order1.

This is useless because:

```text
Order1 cannot see Order2 memory
Order2 cannot see Order3 memory
```

Need shared coordination.

Redis provides that shared state.

---

# 4. When Do We Need Locks?

The trigger question is:

```text
Can multiple servers perform this work simultaneously?
```

If yes:

You may need:

* Distributed Lock
* Idempotency
* Optimistic Locking
* Atomic DB Operations
* Unique Constraints

Locks are only one solution.

---

# 5. Example: User Profile Read

Request:

```http
GET /users/123
```

Flow:

```text
Service
   │
   ▼

Redis Cache

   │ Cache Miss
   ▼

Database
```

No lock needed.

Reason:

```text
Reading data doesn't require exclusive ownership.
```

---

# 6. Example: Payment Processing

Request:

```http
POST /pay
```

Flow:

```text
Payment Service

   │
   ▼

Acquire Lock

   │
   ▼

Charge Card

   │
   ▼

Update Database

   │
   ▼

Release Lock
```

Cache may not even be involved.

---

# Real Example

User double-clicks:

```text
Pay Now
```

Two requests arrive:

```text
Request1
Request2
```

Without lock:

```text
Charge Card Twice
```

---

With lock:

```text
lock:user:123:payment
```

Only one request proceeds.

---

# Architecture

```text
Client
   │
   ▼

API Gateway
   │
   ▼

Payment Service
   │
   ▼

Redis Lock
   │
   ▼

Stripe
```

---

# 7. Example: Kafka Message Processing

Event:

```json
{
  "orderId": 123
}
```

Due to retries:

```text
Consumer1 receives event
Consumer2 receives event
```

Both attempt:

```text
Process Order #123
```

---

Without Lock

```text
Consumer1 → Charge Card
Consumer2 → Charge Card
```

Customer charged twice.

---

With Lock

Consumer1:

```text
SET lock:order:123
```

Success.

---

Consumer2:

```text
SET lock:order:123
```

Fails.

---

Architecture

```text
Kafka

  │

  ▼

Consumer1
Consumer2
Consumer3

  │
  ▼

Redis Lock

  │
  ▼

Payment Service
```

---

# 8. Example: Inventory Reservation

Suppose:

```text
1 PS5 Left
```

Two users purchase simultaneously.

---

Without Lock

```text
UserA reads inventory=1
UserB reads inventory=1

Both purchase
```

Result:

```text
Inventory = -1
```

Oversold.

---

With Lock

```text
lock:product:ps5
```

Only one reservation proceeds.

---

Flow

```text
Checkout Service

      │
      ▼

Redis Lock

      │
      ▼

Inventory DB
```

---

# Complete Flow

User A:

```text
Acquire Lock
      │
      ▼

Read Inventory

      │
      ▼

Update Inventory

      │
      ▼

Release Lock
```

---

User B:

```text
Acquire Lock
      │
      ▼

Fails
```

Waits or retries.

---

# 9. Example: Distributed Cron Jobs

Suppose:

```text
Billing Service

Pod1
Pod2
Pod3
```

All contain:

```java
@Scheduled
dailyBilling()
```

At midnight:

```text
Pod1 starts
Pod2 starts
Pod3 starts
```

---

Without Lock

```text
Generate Invoice
Generate Invoice
Generate Invoice
```

Three executions.

---

With Lock

```text
lock:daily-billing
```

Only one pod runs billing.

---

Architecture

```text
Pod1
Pod2
Pod3

   │
   ▼

Redis

   │
   ▼

Run Billing Job
```

---

# 10. Example: Leader Election

Suppose:

```text
Scheduler1
Scheduler2
Scheduler3
```

Need:

```text
One Leader
```

---

Architecture

```text
Scheduler1
Scheduler2
Scheduler3

      │
      ▼

ZooKeeper
```

---

Leader acquires:

```text
lock:leader
```

Only leader performs:

```text
Task Assignment
```

---

If leader dies:

```text
Lock Released
```

New leader elected.

---

# 11. Example: File Processing

File uploaded:

```text
video.mp4
```

Event:

```text
Generate Thumbnail
```

Workers:

```text
Worker1
Worker2
Worker3
```

---

Without Lock

```text
Generate Thumbnail 3 Times
```

Waste of CPU.

---

With Lock

```text
lock:file:video123
```

Only one worker processes file.

---

# 12. Example: Email Sending

Event:

```text
Send Welcome Email
```

Processed twice.

---

Without Lock

```text
Email #1
Email #2
```

Duplicate emails.

---

With Lock

```text
lock:welcome-email:user123
```

Only one email sent.

---

# 13. Redis Lock Flow in Production

Typical implementation:

```java
lock = redis.acquire("payment:user123")

if(!lock)
    return;

try {
    chargeCard();
}
finally {
    releaseLock();
}
```

---

Production Architecture:

```text
Payment Service

      │
      ├──────────────┐
      ▼              ▼

 Redis Lock      Redis Cache

      │              │
      ▼              ▼

  Coordination   Fast Reads

      └──────┬───────┘
             ▼

         Database
```

---

# 14. Senior Engineer Insight

Many engineers overuse distributed locks.

Before introducing one, ask:

```text
Can I solve this using:
```

* Idempotency Keys - Check Idempotency Notes
* Optimistic Locking
* Atomic Database Operations
* Unique Constraints
* Compare-And-Swap

These solutions are often simpler and more scalable.

---

# Example: Inventory Without Lock

Instead of:

```text
Acquire Lock
Read Inventory
Update Inventory
```

use:

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = 'ps5'
AND quantity > 0;
```

Database guarantees atomicity.

No distributed lock required.

---

# Example: Payments Without Lock

Use:

```text
Idempotency Key
```

Example:

```text
payment-request-123
```

Even if request arrives twice:

```text
Charge Once
```

because duplicate request detected.

---

# 15. Final Mental Model

Ask two separate questions during system design.

### Question 1

```text
How do I make this faster?
```

Answer:

```text
Cache
```

---

### Question 2

```text
How do I ensure only one server performs this action?
```

Answer:

```text
Distributed Lock
```

---

# Key Takeaway

Redis Cache:

```text
Used For Speed
```

Redis Lock:

```text
Used For Coordination
```

They solve completely different problems, even though Redis is often used for both.

A distributed lock typically lives in a central coordination system such as Redis or ZooKeeper, while services acquire and release the lock before entering a critical section of business logic.
