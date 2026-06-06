# Circuit Breakers + Retries

## Reliability Engineering Basics

This is a very important senior-level topic because once systems become distributed, failures become normal.

A junior engineer thinks:

```text
Service call failed
```

A senior engineer thinks:

```text
Why did it fail?
Should I retry?
How many times?
What if everyone retries?
Could retries make the outage worse?
```

That's Reliability Engineering.

---

# First Principle

In a distributed system:

```text
Order Service
      ↓
Payment Service
      ↓
Inventory Service
      ↓
Email Service
```

Failures are guaranteed.

Not possible.

Guaranteed.

Because:

* Network packets get dropped
* Containers restart
* Nodes crash
* Databases slow down
* DNS fails
* Load balancers fail
* Timeouts occur

The question is not:

```text
Will failures happen?
```

The question is:

```text
How does the system react?
```

---

# Example: Simple Failure

Order Service calls Payment Service.

```text
Order Service
      │
      ▼
Payment Service
```

Payment Service takes:

```text
100ms
```

normally.

One day:

```text
Database slow
```

Payment Service now takes:

```text
10 seconds
```

Order Service timeout:

```text
2 seconds
```

Request fails.

Should we retry?

Maybe.

This is where retries come in.

---

# Retries

Basic idea:

```text
Request Failed
      ↓
Try Again
```

Example:

```text
Attempt 1 → Fail
Attempt 2 → Success
```

Problem solved.

Many failures are temporary.

Examples:

```text
Temporary Network Glitch
Container Restart
Transient Database Issue
```

Retry often fixes them.

---

# Real World Example

User clicks:

```text
Place Order
```

Order Service calls:

```text
Inventory Service
```

Inventory pod restarts.

Request fails.

Retry after:

```text
200ms
```

Inventory pod recovers.

Request succeeds.

No user impact.

---

# The Naive Retry Problem

Most engineers initially do:

```java
for(int i=0;i<5;i++)
{
   callService();
}
```

Bad idea.

Suppose service is already overloaded.

```text
1000 Requests
```

arrive.

Each request retries:

```text
5 Times
```

Now backend receives:

```text
5000 Requests
```

during an outage.

You just made things worse.

This is called:

# Retry Storm

---

# Retry Storm

Imagine:

```text
10,000 Clients
```

calling:

```text
Payment Service
```

Payment Service becomes slow.

Every client retries immediately.

Now:

```text
10,000 Original Requests
+
10,000 Retries
```

Then another retry.

```text
20,000
+
20,000
```

Traffic explodes.

System crashes harder.

The outage becomes self-inflicted.

This happens in real companies.

Many major outages have been amplified by retry storms.

---

# How We Fix It

Answer:

```text
Exponential Backoff
```

---

# Exponential Backoff

Instead of:

```text
Retry Immediately
```

we wait.

Example:

```text
Retry 1 → Wait 1 sec
Retry 2 → Wait 2 sec
Retry 3 → Wait 4 sec
Retry 4 → Wait 8 sec
Retry 5 → Wait 16 sec
```

Notice:

```text
1
2
4
8
16
```

doubles each time.

This gives the failing service time to recover.

---

# Visual Timeline

Without backoff:

```text
0s
0s
0s
0s
0s
```

All retries happen instantly.

With backoff:

```text
0s
1s
3s
7s
15s
```

Requests spread out.

Much safer.

---

# Why Exponential Backoff Works

Suppose database restarts.

Recovery time:

```text
5 seconds
```

Immediate retries:

```text
Retry
Retry
Retry
Retry
Retry
```

all fail.

Exponential backoff:

```text
1s
2s
4s
8s
```

By retry #4:

```text
Database recovered
```

Success.

---

# Add Jitter (Very Important)

Even exponential backoff isn't enough.

Suppose:

```text
10000 Clients
```

all retry at:

```text
1s
2s
4s
8s
```

Exactly together.

Still causes spikes.

Add randomness.

Example:

```text
1.2s
1.7s
0.9s
1.4s
```

instead of:

```text
1s
1s
1s
1s
```

This is called:

```text
Jitter
```

Google, AWS, Netflix all strongly recommend:

```text
Exponential Backoff + Jitter
```

---

# Circuit Breaker

Now let's discuss the more important concept.

---

# Motivation

Suppose Payment Service is down.

Without circuit breaker:

```text
Order Service
      ↓
Payment Service (Down)
```

Every request:

```text
Timeout
Timeout
Timeout
Timeout
Timeout
```

Threads get blocked.

CPU wasted.

Resources exhausted.

Eventually:

```text
Order Service
```

also becomes unhealthy.

Failure spreads.

This is called:

```text
Cascading Failure
```

---

# Cascading Failure

Example:

```text
API Gateway
      ↓
Order Service
      ↓
Payment Service
      ↓
Database
```

Database dies.

Payment waits.

Order waits.

Gateway waits.

Everything starts timing out.

Entire system becomes unhealthy.

One failure spreads through the system.

---

# Circuit Breaker Analogy

Think about home electricity.

When current becomes dangerous:

```text
Circuit Breaker Trips
```

Connection is cut.

Protecting the house.

Software circuit breaker does exactly the same thing.

---

# Circuit Breaker States

Three states.

This is interview gold.

---

# Closed State

Normal operation.

```text
Request
      ↓
Service
```

Everything allowed.

---

# Open State

Too many failures.

Circuit opens.

```text
Request
      ↓
Rejected Immediately
```

No call made.

Instead of:

```text
Timeout After 10 Seconds
```

you fail immediately.

This protects the system.

---

# Half Open State

After some cooldown:

```text
Try Small Number Of Requests
```

If successful:

```text
Open → Closed
```

If failures continue:

```text
Half Open → Open
```

---

# Visual Flow

```text
Closed
   │
Too Many Failures
   ▼
Open
   │
Cooldown Period
   ▼
Half Open
   │
Success?
 ┌─┴─┐
Yes  No
 │    │
 ▼    ▼

Closed Open
```

---

# Real World Example

Suppose:

```text
Order Service
```

depends on:

```text
Recommendation Service
```

Recommendations are non-critical.

Recommendation service dies.

Without circuit breaker:

```text
Every Order Request
      ↓
Timeout
```

Users cannot place orders.

Bad design.

With circuit breaker:

```text
Recommendation Service Down
```

↓

```text
Skip Recommendations
```

↓

```text
Place Order Anyway
```

System degrades gracefully.

---

# Netflix Example

Netflix popularized circuit breakers through Hystrix.

Example:

```text
Movie Playback
```

depends on:

```text
Recommendations
Ratings
Watch History
```

If Recommendations fail:

```text
Show Movie
Without Recommendations
```

instead of:

```text
Entire Site Down
```

---

# When To Retry

Good candidates:

```text
Network Timeout
Temporary Failure
Connection Reset
503 Service Unavailable
```

---

# When NOT To Retry

Bad candidates:

```text
400 Bad Request
401 Unauthorized
404 Not Found
Validation Error
```

Retrying won't help.

---

# Retry + Circuit Breaker Together

Production systems use both.

Flow:

```text
Request
   │
   ▼

Retry 3 Times

   │
   ▼

Failures Continue?

   │
   ▼

Circuit Opens

   │
   ▼

Fail Fast
```

---

# Example Architecture

```text
Client
   │
   ▼

Order Service

   │
   ▼

Circuit Breaker

   │
   ▼

Payment Service
```

If Payment fails repeatedly:

```text
Circuit Opens
```

Order Service stops calling it temporarily.

---

# What Interviewers Expect

You should know:

### Retry

```text
Temporary failures
```

### Exponential Backoff

```text
1s
2s
4s
8s
```

### Jitter

```text
Randomize retries
```

### Retry Storm

```text
Retries amplify outage
```

### Circuit Breaker

```text
Prevent cascading failures
```

### States

```text
Closed
Open
Half Open
```

---

# Interview Answer Template

If asked:

> How do you make service-to-service communication resilient?

A strong answer:

> I would use retries for transient failures, combined with exponential backoff and jitter to avoid retry storms. To prevent cascading failures when a dependency is unhealthy, I would place a circuit breaker in front of the dependency. If failures exceed a threshold, the circuit opens and requests fail fast. After a cooldown period, the circuit enters a half-open state to test recovery before returning to normal operation.

---

# Senior Engineer Mental Model

Think:

```text
Failure Happens
      ↓
Retry?
      ↓
Use Exponential Backoff
      ↓
Avoid Retry Storms
      ↓
Dependency Still Failing?
      ↓
Open Circuit Breaker
      ↓
Fail Fast
      ↓
Prevent Cascading Failure
```

The most important insight is:

> Retries help recover from temporary failures, but uncontrolled retries can destroy a system. Circuit breakers exist to stop a failing dependency from taking down everything around it. This combination forms the foundation of reliability engineering in distributed systems.
