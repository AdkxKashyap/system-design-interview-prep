# ACID Properties — System Design Interview Notes

# 1. Introduction

ACID is one of the MOST IMPORTANT concepts in databases and system design interviews.

Interviewers LOVE ACID because it tests whether you understand:
- transactions
- consistency
- concurrency
- reliability
- database correctness

Most candidates memorize:

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

But senior interviews expect MUCH deeper understanding:
- WHY these properties exist
- WHAT real-world problems they solve
- WHY they are expensive
- WHY internet-scale systems sometimes relax them

The most important thing to understand:

> ACID exists to guarantee correctness in transactional systems.

---

# 2. First Understand the REAL Problem ACID Solves

Imagine you are building:
- banking system
- payment gateway
- airline booking system
- stock trading platform

Now imagine:
- servers crash
- multiple users update same data
- power failures happen
- concurrent transactions occur

Without safeguards:
- data corruption happens
- inconsistent state appears
- money disappears
- overselling occurs

ACID properties exist to prevent this.

---

# 3. What is a Transaction?

Before understanding ACID:
we must understand transactions.

A transaction is:
> a group of database operations treated as one logical unit.

Example:

```text id="lp8nka"
Transfer ₹1000 from Account A → Account B
```

This involves:
1. Deduct ₹1000 from A
2. Add ₹1000 to B

These two operations must behave as:
```text id="d7u0m8"
ONE UNIT
```

Either:
- both succeed
OR
- both fail

Partial execution unacceptable.

This is the foundation of ACID.

---

# 4. Atomicity (A)

# Core Idea

Atomicity means:

> Either the entire transaction succeeds OR nothing happens.

No partial execution allowed.

---

# Real World Banking Example

Suppose:

```text id="5o9y3n"
Account A = ₹5000
Account B = ₹2000
```

User transfers:

```text id="djlwm3"
₹1000 from A → B
```

Steps:
1. Deduct ₹1000 from A
2. Add ₹1000 to B

Now imagine:
- server crashes after deduction
- addition never happens

Without atomicity:

```text id="g7jlwm"
A = ₹4000
B = ₹2000
```

₹1000 vanished.

Catastrophic failure.

---

# What Atomicity Does

Database ensures:

If ANY step fails:
- rollback entire transaction

Final state becomes:

```text id="5zjlwm"
A = ₹5000
B = ₹2000
```

as if transaction never happened.

---

# Real Production Systems Where Atomicity Critical

## Banking Systems

Money cannot disappear.

---

## Payment Systems

Suppose:
- payment deducted
- order creation failed

Without rollback:
- customer charged
- no order created

Disaster.

---

## Airline Booking

Suppose:
- seat reserved
- payment failed

Need rollback.

---

# How Databases Implement Atomicity

Usually using:
- transaction logs
- undo logs
- rollback mechanisms

Database keeps track of changes.

If failure occurs:
- undo incomplete operations.

---

# Important Interview Insight

Atomicity prevents:
> partial state corruption.

This phrase sounds very strong in interviews.

---

# 5. Consistency (C)

# Core Idea

Consistency means:

> Database always moves from one VALID state to another VALID state.

Rules and constraints must always remain true.

---

# Important Clarification

ACID consistency is NOT same as distributed systems consistency.

This confuses MANY candidates.

---

# ACID Consistency Means

Database constraints remain valid.

Examples:
- no duplicate primary keys
- no invalid foreign keys
- no negative inventory
- no invalid references

---

# Real World Example — Airline Booking

Suppose flight has:

```text id="0yjlwm"
100 seats
```

System should NEVER allow:

```text id="jlwm66"
101 bookings
```

That would violate consistency.

---

# Real World Example — Banking

Suppose rule:

```text id="jlwm91"
Balance cannot go below 0
```

Transaction violating rule rejected.

Database prevents invalid state.

---

# Example — Foreign Key Consistency

Suppose:
```text id="jlwm82"
orders.user_id
```

references:
```text id="jlwm43"
users.id
```

If user does not exist:
- order insertion rejected

Consistency preserved.

---

# Why Consistency Important?

Without consistency:
- corrupted data spreads
- business logic breaks
- analytics become unreliable

Data integrity critical in production systems.

---

# Important Interview Insight

Consistency ensures:
> business rules and database invariants remain preserved.

---

# 6. Isolation (I)

# Core Idea

Isolation means:

> Concurrent transactions should not interfere incorrectly with each other.

This is one of the HARDEST database concepts.

---

# Why Isolation Needed?

Modern databases handle:
- thousands of users simultaneously

Many transactions run concurrently.

Without isolation:
- race conditions occur
- corrupted state appears

---

# Real World Example — Last iPhone Problem

Suppose inventory:

```text id="jlwm35"
1 iPhone remaining
```

Two users simultaneously buy it.

Transaction A:
```text id="jlwm83"
Reads inventory = 1
```

Transaction B:
```text id="jlwm94"
Reads inventory = 1
```

Both purchase successfully.

Now:
```text id="3jlwm4"
2 phones sold
1 existed
```

Overselling problem.

---

# Isolation Prevents This

Database ensures transactions behave:
> as if running alone.

This prevents concurrency corruption.

---

# Common Concurrency Problems

VERY important interview topic.

---

# A. Dirty Reads

Transaction reads:
```text id="jlwm52"
uncommitted data
```

Example:
- Transaction A updates balance
- Transaction B reads new balance
- Transaction A later rolls back

Transaction B saw invalid temporary state.

---

# B. Non-Repeatable Reads

Same query inside transaction returns different values.

Example:
- read balance = ₹5000
- another transaction updates balance
- second read = ₹3000

Results inconsistent.

---

# C. Phantom Reads

Rows appear/disappear during transaction.

Example:
- query all pending orders
- another transaction inserts new order
- same query returns extra row

---

# Non-Repeatable Reads vs Phantom Reads

## Non-Repeatable Read

### Definition

Occurs when:

> the SAME ROW returns different values within the same transaction because another transaction modified it.

### Example

Transaction T1 reads:

```sql
SELECT balance FROM accounts WHERE id = 1;
```

Result:

```text
₹5000
```

Another transaction updates balance to:

```text
₹3000
```

T1 reads again:

```text
₹3000
```

### Key Idea

```text
Existing row changed
```

---

## Phantom Read

### Definition

Occurs when:

> the SAME QUERY returns different sets of rows because another transaction inserted/deleted rows.

### Example

T1 runs:

```sql
SELECT * FROM orders WHERE status='pending';
```

Result:

```text
2 rows
```

Another transaction inserts new pending order.

T1 runs same query again:

```text
3 rows
```

### Key Idea

```text
Result set changed
```

---

# Main Difference

| Problem             | What Changes              |
| ------------------- | ------------------------- |
| Non-Repeatable Read | Existing row modified     |
| Phantom Read        | New rows appear/disappear |

---

# How We Avoid Them

| Isolation Level | Prevents             |
| --------------- | -------------------- |
| Repeatable Read | Non-repeatable reads |
| Serializable    | Phantom reads        |

---

# Important Insight

Higher isolation levels:

* improve correctness
* reduce concurrency
* increase locking overhead

This is a classic:

```text
Correctness vs Performance
```

tradeoff in databases.


# Isolation Levels

Higher isolation:
- stronger correctness
- lower concurrency
- lower performance

This tradeoff VERY important.

---

# Common Isolation Levels

| Isolation Level | Guarantees | Performance |
|---|---|---|
| Read Uncommitted | Weakest | Fastest |
| Read Committed | Prevents dirty reads | Common |
| Repeatable Read | Stable reads | Stronger |
| Serializable | Full isolation | Slowest |

---

# Serializable Isolation

Strongest level.

Transactions behave:
> completely sequentially.

Very safe.
Very expensive.

---

# Real Production Tradeoff

Most systems DO NOT use strongest isolation everywhere because:
- locking expensive
- throughput drops
- latency increases

Again:
> correctness vs scalability tradeoff.

---

# 7. Durability (D)

# Core Idea

Durability means:

> Once transaction commits, data survives crashes.

Even if:
- server crashes
- power outage occurs
- process dies

Committed data must persist.

---

# Real World Example — Banking

Suppose transfer completed successfully.

Immediately after:
```text id="jlwm19"
power outage
```

Money MUST still remain transferred after restart.

Otherwise:
- catastrophic financial corruption.

---

# How Databases Achieve Durability

Usually using:
- Write Ahead Log (WAL)
- disk persistence
- replication
- fsync operations

---

# What is WAL (Write Ahead Log)?

Before modifying actual DB:
- changes written to durable log first

If crash happens:
- DB recovers using log replay.

This is fundamental database technique.

---

# Why Durability Expensive?

Disk writes slower than memory.

Durability requires:
- flushing to disk
- synchronization
- logging

This increases latency.

---

# Real Engineering Tradeoff

Durability improves reliability BUT:
- reduces write performance

Some systems intentionally relax durability for speed.

Example:
- analytics systems
- temporary caches

---

# 8. Why ACID Is Expensive

VERY important system design insight.

ACID requires:
- locking
- synchronization
- transaction coordination
- rollback support
- durable logging

All these reduce:
- scalability
- throughput
- write performance

This is why internet-scale systems sometimes relax ACID guarantees.

---

# 9. Why Social Media Systems Often Relax ACID

Suppose Instagram like count delayed:

```text id="jlwm07"
2 seconds
```

Usually acceptable.

Strict ACID everywhere would:
- reduce scalability
- increase coordination overhead

So systems often choose:
- eventual consistency
- asynchronous updates

---

# 10. SQL Databases and ACID

Traditional SQL databases strongly support:
- ACID transactions
- consistency guarantees

Examples:
- PostgreSQL
- MySQL
- Oracle

This is why they dominate:
- banking
- finance
- ERP systems

---

# 11. NoSQL and ACID

Traditional NoSQL systems often relaxed:
- consistency
- transactions

to improve:
- scalability
- availability

Modern NoSQL systems now support limited transactions, but:
- distributed ACID remains expensive

---

# 12. Real Interview Scenarios

# Banking System

Use:
- strong ACID
- SQL database

Because correctness critical.

---

# Social Media Feed

Can relax:
- consistency
- isolation

Because scale more important.

---

# Inventory System

Need:
- strong consistency
- transaction guarantees

To avoid overselling.

---

# Analytics System

May relax:
- durability
- consistency

for higher throughput.

---

# 13. Important Interview Questions

# Q1. Why Is ACID Important?

Because it prevents:
- data corruption
- inconsistent state
- partial failures

---

# Q2. Why Is ACID Expensive?

Requires:
- coordination
- locking
- disk synchronization
- transaction management

---

# Q3. Why Don’t Large Systems Always Use Full ACID?

Because:
- scalability suffers
- latency increases
- distributed coordination expensive

---

# Q4. Can NoSQL Support ACID?

Some modern NoSQL databases partially support transactions.

But distributed ACID remains difficult and expensive.

---

# 14. Most Important Interview Insight

The key tradeoff is:

> Strong correctness guarantees vs scalability/performance.

This is the heart of distributed systems design.

---

# 15. Senior Engineer Thinking

Junior engineer says:
> "ACID means transactions safe."

Senior engineer says:
> "ACID guarantees transactional correctness under concurrency and failure scenarios, but these guarantees introduce coordination and durability overhead that impact scalability."

That sounds production-level.

---

# 16. Final Revision Summary

| Property | Meaning | Real Problem Solved |
|---|---|---|
| Atomicity | All or nothing | Prevent partial updates |
| Consistency | Valid state preserved | Prevent invalid data |
| Isolation | Transactions don't interfere | Prevent race conditions |
| Durability | Data survives crashes | Prevent data loss |

---

# 17. Final Mental Model

ACID fundamentally exists to guarantee:

```text id="jlwm84"
Correctness + reliability + transactional safety
```

under:
- failures
- crashes
- concurrency
- distributed complexity

---

# 18. Senior-Level Insight

ACID is fundamentally about:
> preserving correctness in the presence of failures and concurrent operations.