# Idempotency

# What Is Idempotency?

The formal definition:

> Performing the same operation multiple times produces the same result as performing it once.

---

# Real Life Example

Light switch:

```text id="fl0lx2"
Turn ON
```

Once:

```text id="n79jef"
OFF → ON
```

Again:

```text id="9z6433"
ON → ON
```

Again:

```text id="ksn3w2"
ON → ON
```

Final state unchanged.

This is idempotent.

---

Not idempotent:

```text id="c04h8z"
Increase Volume By 10
```

```text id="nfbifs"
50 → 60
60 → 70
70 → 80
```

Every execution changes state.

---

# Why Is Idempotency So Important?

Because distributed systems are full of retries.

Things fail all the time:

```text id="jwysxf"
Network timeout
Server crash
Kafka redelivery
Client retries
Load balancer retry
Webhook retry
```

Question:

> What happens if the same request executes twice?

This is where systems break.

---

# The Classic Payment Example

Suppose user clicks:

```text id="5axz6x"
Pay ₹1000
```

Request reaches server.

Server:

```text id="blod5l"
Charge Card
```

successfully.

---

Then:

```text id="nzzwza"
Response Lost
```

because network fails.

Client sees:

```text id="hj70ya"
Request Failed
```

and retries.

---

Now server receives:

```text id="xmw7kz"
Pay ₹1000
```

again.

Without idempotency:

```text id="zbj021"
Charge #1
Charge #2
```

Customer loses ₹2000.

---

This is one of the most common distributed systems failures.

---

# The Core Idea

Instead of identifying a request by:

```text id="nxyszw"
User ID
Amount
```

we introduce:

```text id="dz6btm"
Idempotency Key
```

Example:

```text id="1p6uav"
payment-abc123
```

---

Request:

```json id="lnd29u"
{
  "amount": 1000,
  "idempotencyKey": "payment-abc123"
}
```

---

# First Request

Server receives:

```text id="n8qk60"
payment-abc123
```

Checks database:

```text id="eqf6oa"
Seen Before?
```

No.

---

Process payment.

Store:

```text id="5d0jil"
payment-abc123
```

in database.

---

Return:

```text id="21gox2"
Success
```

---

# Retry Happens

Same request arrives.

```text id="qd2bnq"
payment-abc123
```

Server checks:

```text id="qzzz7h"
Seen Before?
```

Yes.

---

Instead of charging again:

```text id="pyeutn"
Return Existing Result
```

No duplicate charge.

---

# Visual Flow

```text id="3kob3v"
Client

  │

  ▼

Pay ₹1000
(id=abc123)

  │

  ▼

Payment Service

  │

  ▼

Processed?
   │
  No

  │

  ▼

Charge Card

  │

  ▼

Store abc123

  │

  ▼

Success
```

---

Retry:

```text id="mjydhy"
Client

  │

  ▼

Pay ₹1000
(id=abc123)

  │

  ▼

Payment Service

  │

  ▼

Processed?
   │
  Yes

  │

  ▼

Return Existing Response
```

---

# Where Is The Key Stored?

Usually database.

Example:

```sql id="nu0dez"
payments
---------
id
amount
idempotency_key
status
```

---

Often:

```sql id="mi1jl0"
UNIQUE(idempotency_key)
```

is added.

This is a very common production design.

---

# Real World Example: Stripe

Stripe popularized idempotency keys.

Client sends:

```http id="8n2lfm"
POST /payments

Idempotency-Key: abc123
```

---

Stripe stores:

```text id="d034y3"
abc123
```

and remembers result.

Any retry returns same response.

---

# Example: Order Creation

Suppose:

```text id="x5fd2p"
Create Order
```

request.

---

Without Idempotency

User double-clicks:

```text id="gxousi"
Place Order
```

Result:

```text id="qhbyru"
Order #101
Order #102
```

Two orders.

---

With Idempotency

```text id="gy66pu"
order-xyz789
```

used.

Second request:

```text id="nx47ql"
Already Processed
```

Return:

```text id="nof3pi"
Order #101
```

No duplicate.

---

# Kafka Example

This is extremely common in interviews.

---

Suppose Kafka event:

```json id="bgtd7a"
{
  "eventId": "evt123",
  "orderId": 10
}
```

---

Consumer processes:

```text id="26g2tj"
Send Welcome Email
```

---

Then crashes.

Kafka re-delivers event.

---

Without Idempotency

```text id="7kqv25"
Email #1
Email #2
```

Duplicate email.

---

With Idempotency

Store:

```text id="w29gzz"
evt123
```

in processed-events table.

---

Retry arrives.

Check:

```text id="qmctzk"
evt123 exists?
```

Yes.

Skip processing.

---

# Architecture

```text id="grcxyt"
Kafka

   │

   ▼

Consumer

   │

   ▼

Processed Events Table

   │

   ▼

Business Logic
```

---

# Webhook Example

Very common.

Suppose Stripe sends:

```text id="474axw"
payment_completed
```

webhook.

---

Network issue.

Stripe retries.

You receive:

```text id="pg324q"
payment_completed
```

three times.

---

Without Idempotency

```text id="vc0wgw"
Give Credits
Give Credits
Give Credits
```

User gets triple credits.

---

With Idempotency

Store:

```text id="phdbnr"
webhook_id
```

Process only once.

---

# Food Delivery Example

Suppose Swiggy-like system.

User places order.

---

Request:

```text id="twqn2l"
order-123
```

---

Network timeout.

App retries.

---

Without Idempotency

Restaurant receives:

```text id="xlh96r"
2 Orders
```

Bad.

---

With Idempotency

System recognizes:

```text id="1zeut6"
order-123
```

already processed.

Returns existing order.

---

# Does Idempotency Mean No Locks?

Not always.

This is a very important interview insight.

---

Idempotency prevents:

```text id="jcnoby"
Duplicate Requests
```

---

Locks prevent:

```text id="ojelc2"
Concurrent Execution
```

Different problems.

---

Example:

```text id="qxwrtj"
Charge Payment
```

Idempotency is usually enough.

---

Example:

```text id="ipbpbg"
Elect One Leader
```

Need lock.

Idempotency doesn't help.

---

# Common Implementation Patterns

## Pattern 1: Processed Requests Table

```sql id="b1ypt0"
processed_requests
-------------------
idempotency_key
response
created_at
```

Most common.

---

## Pattern 2: Unique Constraint

```sql id="vs4k01"
UNIQUE(idempotency_key)
```

Very elegant.

Database guarantees uniqueness.

---

## Pattern 3: Redis

Store:

```text id="c1k9s1"
idempotency:abc123
```

with TTL.

Useful for short-lived deduplication.

---

# Interview Gold Nugget

Suppose interviewer asks:

> How would you prevent duplicate payment processing?

Many candidates say:

```text id="jr8mvg"
Distributed Lock
```

A stronger answer is:

> I would make the payment API idempotent using an idempotency key. Every payment request carries a unique key. The payment service stores processed keys and returns the previous result for retries. This prevents duplicate charging even under retries, network failures, and client resubmissions.

That answer immediately signals production experience.

---

# When Should You Think About Idempotency?

Whenever you hear:

```text id="b7n5ni"
Retry
Webhook
Payment
Order Creation
Kafka
Message Queue
At-Least-Once Delivery
Network Failure
```

your brain should automatically think:

```text id="0y8qm4"
IDEMPOTENCY
```

---

# Senior Engineer Mental Model

The biggest lesson is:

Distributed systems cannot prevent retries.

They can only handle retries safely.

Idempotency is the mechanism that allows systems to safely process:

```text id="r829bs"
1 Request
2 Requests
10 Requests
100 Requests
```

and still produce:

```text id="oof034"
Exactly One Business Outcome
```

That is why nearly every payment system, order system, webhook processor, and message consumer at companies like Stripe, Amazon, Uber, and Netflix relies heavily on idempotency.
