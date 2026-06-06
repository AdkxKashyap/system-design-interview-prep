# Distributed Locks vs Idempotency Keys

# The Core Difference

## Idempotency

Answers:

> What if the same request/event arrives multiple times?

---

## Distributed Lock

Answers:

> What if multiple servers execute the same code at the same time?

---

Think:

```text id="q2"
Duplicate Requests
        ↓
Idempotency
```

vs

```text id="q3"
Concurrent Execution
        ↓
Distributed Lock
```

---

# Example 1: Payment Processing

User clicks:

```text id="q4"
Pay ₹1000
```

Network times out.

User clicks again.

Now:

```text id="q5"
Request1
Request2
```

arrive.

---

Question:

```text id="q6"
Can payment be processed twice?
```

Yes.

---

Solution:

```text id="q7"
Idempotency Key
```

Request:

```json id="q8"
{
  "idempotencyKey": "pay-123"
}
```

---

Why not a lock?

Because the problem is:

```text id="q9"
Duplicate Request
```

not

```text id="q10"
Concurrent Ownership
```

---

# Real World

Stripe.

PayPal.

Razorpay.

Almost all payment providers rely heavily on:

```text id="q11"
Idempotency Keys
```

---

# Example 2: Kafka Consumer

Kafka guarantees:

```text id="q12"
At Least Once Delivery
```

which means:

```text id="q13"
Event May Arrive Multiple Times
```

---

Event:

```json id="q14"
{
  "eventId":"evt123"
}
```

---

Consumer crashes.

Kafka re-delivers.

---

Problem:

```text id="q15"
Duplicate Event
```

---

Solution:

```text id="q16"
Store Processed Event IDs
```

Idempotency.

---

No lock needed.

---

# Example 3: Webhooks

Stripe sends:

```text id="q17"
payment_completed
```

three times.

---

Problem:

```text id="q18"
Duplicate Delivery
```

---

Solution:

```text id="q19"
Webhook ID
```

Idempotency.

---

No lock needed.

---

# Example 4: Order Creation

User double-clicks:

```text id="q20"
Place Order
```

---

Problem:

```text id="q21"
Duplicate Order Creation
```

---

Solution:

```text id="q22"
Idempotency Key
```

---

No lock needed.

---

# Notice the Pattern

Every time you hear:

```text id="q23"
Retry
Timeout
Webhook
Kafka
Queue
Payment
```

think:

```text id="q24"
IDEMPOTENCY
```

---

# When Locks Become Necessary

Locks solve a different problem.

---

# Example 5: Leader Election

Suppose:

```text id="q25"
Scheduler1
Scheduler2
Scheduler3
```

Need:

```text id="q26"
One Leader
```

---

Can idempotency help?

No.

---

Problem is:

```text id="q27"
Only One Server Must Act
```

---

Solution:

```text id="q28"
Distributed Lock
```

---

# Example 6: Distributed Cron Jobs

Kubernetes deployment:

```text id="q29"
Pod1
Pod2
Pod3
```

All execute:

```java id="q30"
dailyBilling()
```

at midnight.

---

Problem:

```text id="q31"
3 Pods Execute Same Job
```

---

Need:

```text id="q32"
Only One Pod Runs
```

---

Solution:

```text id="q33"
Distributed Lock
```

---

# Example 7: File Processing

Uploaded:

```text id="q34"
video.mp4
```

Workers:

```text id="q35"
Worker1
Worker2
Worker3
```

pick same task.

---

Problem:

```text id="q36"
Multiple Workers Process Same File
```

---

Solution:

```text id="q37"
Distributed Lock
```

---

# Example 8: Inventory Reservation

This one is interesting.

---

Suppose:

```text id="q38"
1 PS5 Left
```

Users:

```text id="q39"
UserA
UserB
```

buy simultaneously.

---

Can idempotency help?

No.

These are:

```text id="q40"
Different Requests
```

not duplicates.

---

The problem:

```text id="q41"
Concurrent Modification
```

---

Possible solutions:

### Option 1

```text id="q42"
Distributed Lock
```

---

### Option 2 (Preferred)

```text id="q43"
Atomic DB Update
```

Example:

```sql id="q44"
UPDATE inventory
SET quantity = quantity - 1
WHERE quantity > 0;
```

Most companies prefer this.

---

# Interview Gold

When interviewer says:

```text id="q45"
Only one worker should process this
```

Think:

```text id="q46"
Distributed Lock
```

---

When interviewer says:

```text id="q47"
Request may be retried
```

Think:

```text id="q48"
Idempotency
```

---

# Side-by-Side Comparison

| Scenario                    | Idempotency          | Distributed Lock |
| --------------------------- | -------------------- | ---------------- |
| Payment Retry               | ✅                    | ❌                |
| Kafka Duplicate Event       | ✅                    | ❌                |
| Webhook Retry               | ✅                    | ❌                |
| Double Click Order          | ✅                    | ❌                |
| Leader Election             | ❌                    | ✅                |
| Cron Job Execution          | ❌                    | ✅                |
| File Processing Ownership   | ❌                    | ✅                |
| Multiple Servers Competing  | ❌                    | ✅                |
| Concurrent Inventory Update | ⚠️ Usually Atomic DB | Sometimes        |

---

# The Senior Engineer Decision Tree

When you see a problem:

### Step 1

Ask:

```text id="q49"
Is this the same request arriving multiple times?
```

If YES:

```text id="q50"
Use Idempotency
```

---

### Step 2

Ask:

```text id="q51"
Are multiple servers competing to do the same work?
```

If YES:

```text id="q52"
Use Distributed Lock
```

---

### Step 3

Ask:

```text id="q53"
Can the database solve this atomically?
```

If YES:

```text id="q54"
Prefer Atomic DB Operations
```

because they're usually simpler than distributed locks.

---

# What Impresses Interviewers

Suppose interviewer asks:

> How would you prevent duplicate payment processing?

Weak answer:

```text id="q55"
Use Redis Lock
```

---

Strong answer:

```text id="q56"
Use Idempotency Key
```

---

Suppose interviewer asks:

> How would you ensure only one scheduler runs the billing job?

Strong answer:

```text id="q57"
Use Distributed Lock or Leader Election
```

because now the problem is ownership, not duplicate requests.

---

# Final Mental Model

Think:

```text id="q58"
Duplicate Requests
        ↓
Idempotency
```

and

```text id="q59"
Exclusive Ownership
        ↓
Distributed Lock
```

That's the distinction most senior engineers use in real production systems.
