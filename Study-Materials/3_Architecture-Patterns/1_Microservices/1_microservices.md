# Microservices Architecture Patterns

# Why Did Microservices Exist?

Let's start with the problem.

Imagine you're building Amazon.

Initially:

```text id="2ieurz"
Amazon Application

├── Orders
├── Payments
├── Inventory
├── Search
├── Reviews
├── Notifications
└── Recommendations
```

Everything is inside:

```text id="gn31gt"
One Application
One Deployment
One Database
```

This is called:

```text id="dqfb50"
Monolith
```

---

# Monolith Works Initially

For a startup:

```text id="1b5xnj"
10 Engineers
1000 Users
```

Monolith is fantastic.

Why?

Because:

```text id="akfyia"
Simple
Easy Debugging
Easy Deployment
Single Codebase
```

---

# Then Scale Arrives

Now imagine Amazon scale.

Search traffic:

```text id="cyaicw"
100 Million Requests
```

Order traffic:

```text id="ffhul4"
10 Million Requests
```

Review traffic:

```text id="1x9g9i"
500 Million Requests
```

Question:

```text id="0vgs7p"
Why should Order Service
scale because Reviews are busy?
```

It shouldn't.

This is the first motivation for microservices.

---

# Service Decomposition

This is probably the most important microservices concept.

Interviewers love asking:

> How would you split the system?

Not:

> Which database would you use?

---

# What Is Service Decomposition?

Breaking a large application into smaller services.

Instead of:

```text id="54l58y"
Amazon

├── Orders
├── Payments
├── Inventory
├── Search
└── Reviews
```

we create:

```text id="kam9zr"
Order Service
Payment Service
Inventory Service
Search Service
Review Service
```

Each becomes independently deployable.

---

# Real World Example: E-Commerce

Bad decomposition:

```text id="djdhe4"
User Service
Product Service
Everything Else Service
```

This tells interviewer nothing.

---

Better decomposition:

```text id="1v4ay3"
User Service
Catalog Service
Inventory Service
Order Service
Payment Service
Notification Service
Recommendation Service
```

Each owns a business capability.

This is the key phrase:

```text id="ubq63w"
Business Capability
```

---

# Senior Interview Insight

Services should be split around:

```text id="o7idqq"
Business Domains
```

not technical layers.

Bad:

```text id="dkmiff"
Controller Service
Database Service
Validation Service
```

Good:

```text id="bfhcc3"
Order Service
Payment Service
Inventory Service
```

---

# Amazon Order Flow

Imagine customer purchases:

```text id="yhumeq"
PS5
```

Flow:

```text id="et6gqp"
Client
   │
   ▼

Order Service

   │
   ├───────────────┐
   ▼               ▼

Inventory      Payment

   │               │
   └──────┬────────┘
          ▼

Notification
```

Each service owns its own responsibility.

---

# Independent Scaling

This is probably the biggest reason companies adopt microservices.

Suppose:

```text id="u293fu"
Black Friday
```

Search traffic explodes.

Search requests:

```text id="rtfqeq"
500M/day
```

Orders:

```text id="tio0lh"
20M/day
```

Monolith:

```text id="ffwkpz"
Scale Entire Application
```

Need 100 servers.

---

Microservices:

```text id="55gkcf"
Search Service → 100 Servers

Order Service → 10 Servers

Payment Service → 5 Servers
```

Only scale what needs scaling.

---

# Netflix Example

Imagine:

```text id="r4rzar"
Playback Service
Recommendation Service
User Service
Search Service
```

Movie playback:

```text id="i57oqq"
Extremely High Traffic
```

Recommendations:

```text id="puap8m"
Moderate Traffic
```

Netflix scales:

```text id="bde8xr"
Playback Service
```

far more aggressively.

This is independent scaling.

---

# Independent Technology Choices

Another major benefit.

Order Service:

```text id="3aqh3e"
PostgreSQL
```

Search Service:

```text id="4h8wy2"
Elasticsearch
```

Cache:

```text id="m6i6ge"
Redis
```

Analytics:

```text id="92pwwx"
Kafka
```

Each service chooses the right technology.

This is called:

```text id="5yxj63"
Polyglot Persistence
```

You don't need to memorize the term, but understand the idea.

---

# The Biggest Microservices Problem

Most interview candidates talk only about benefits.

Senior engineers immediately discuss costs.

---

# Distributed Systems Complexity

Monolith:

```text id="f30jgv"
Function Call
```

Example:

```java id="p7jwk5"
paymentService.pay();
```

Microservices:

```text id="ks7hfm"
Network Call
```

Example:

```http id="o5118m"
POST /payment
```

Now failures appear:

```text id="qq2kjs"
Timeout
Latency
Packet Loss
Retries
Circuit Breakers
```

Suddenly everything becomes harder.

---

# Eventual Consistency

This is the most important tradeoff to understand.

Suppose:

```text id="7ml78u"
Order Service
Inventory Service
Payment Service
```

each have their own database.

Customer buys:

```text id="sxrw8l"
PS5
```

Order Service:

```text id="2s0o9w"
Order Created
```

Inventory Service:

```text id="96xsje"
Stock Reserved
```

Payment Service:

```text id="iuynq1"
Payment Success
```

Question:

```text id="ke60sv"
How do we update
3 databases atomically?
```

Answer:

```text id="9tpbdm"
Usually We Don't
```

This surprises many engineers.

---

# Why Not?

Traditional transaction:

```sql id="9pbpab"
BEGIN
UPDATE Orders
UPDATE Inventory
UPDATE Payment
COMMIT
```

works inside one database.

Microservices:

```text id="rl5iuu"
Different Services
Different Databases
Different Machines
```

No single transaction.

---

# Eventual Consistency

Instead of:

```text id="075tm5"
Everything Updated Immediately
```

we accept:

```text id="uzdlrh"
Temporary Inconsistency
```

that eventually resolves.

Example:

Customer places order.

Immediately:

```text id="nz8a0t"
Order DB = Updated
Inventory DB = Pending
```

A few seconds later:

```text id="txvjmt"
Inventory Updated
```

Eventually:

```text id="tzi4lz"
Entire System Consistent
```

Hence:

```text id="bcbuww"
Eventual Consistency
```

---

# Real Example: Amazon

You buy a laptop.

Immediately:

```text id="7j5aul"
Order Confirmed
```

Inventory update:

```text id="v25pkd"
Still Processing
```

Shipping:

```text id="wajkuw"
Still Processing
```

Eventually:

```text id="yai7v1"
Everything Converges
```

This is normal.

---

# How Is Eventual Consistency Achieved?

Usually through events.

Example:

```text id="v00ots"
Order Created
```

published to Kafka.

Consumers:

```text id="4in29s"
Inventory Service
Payment Service
Notification Service
```

receive event.

---

# Architecture

```text id="8dxgsd"
Order Service

      │
      ▼

Kafka

 ┌────┼────┐
 ▼    ▼    ▼

Inventory
Payment
Email
```

Every service updates itself independently.

---

# Why Companies Love This

Loose coupling.

Order Service doesn't need to know:

```text id="0lmuvw"
Inventory Internals
Payment Internals
Email Internals
```

It simply publishes:

```text id="c8kdma"
Order Created
```

---

# Trade-Off #1: Operational Complexity

The biggest downside.

Monolith:

```text id="kppf4e"
1 Service
```

Microservices:

```text id="vhep53"
50 Services
```

Need:

```text id="nyv6mg"
Monitoring
Logging
Tracing
Deployment
Discovery
Load Balancing
Security
```

for all 50.

This is why many companies fail with microservices.

---

# Real Example

Bug occurs.

Monolith:

```text id="mut4a8"
One Log File
```

Microservices:

```text id="n5nw9g"
API Gateway
Order Service
Payment Service
Kafka
Inventory Service
```

Need distributed tracing.

Much harder.

---

# Trade-Off #2: Network Overhead

Monolith:

```java id="8sxzjt"
functionCall();
```

takes:

```text id="e379os"
Microseconds
```

Microservice:

```http id="mweepl"
POST /payment
```

takes:

```text id="espdci"
Milliseconds
```

Now multiply by:

```text id="t7w5cv"
10 Services
```

Latency adds up.

---

# Example

```text id="3zrxlb"
API Gateway

    ↓

Order Service

    ↓

Payment Service

    ↓

Inventory Service

    ↓

Notification Service
```

Every hop adds latency.

---

# When NOT To Use Microservices

Interviewers love this question.

Small startup:

```text id="8l6aic"
5 Engineers
```

Single product.

Traffic:

```text id="inbdxy"
10k users
```

Choose:

```text id="f3x5ar"
Monolith
```

Microservices would add:

```text id="fnihg4"
Massive Complexity
```

without benefits.

---

# When To Use Microservices

Usually when:

```text id="2ukv1n"
Large Engineering Teams
Large Scale
Independent Deployments
Independent Scaling
```

are required.

---

# Interview Answer Template

If asked:

> Why do companies adopt microservices?

Strong answer:

> Microservices allow systems to be decomposed around business capabilities, enabling independent deployments and independent scaling. For example, a search service can scale separately from a payment service. However, this introduces distributed systems challenges such as network latency, service discovery, retries, circuit breakers, and eventual consistency. Most large-scale systems use asynchronous messaging and events to achieve eventual consistency across service boundaries.

---

# Senior Engineer Mental Model

Think:

```text id="wlr6eq"
Growing System
      ↓
Monolith Becomes Large
      ↓
Split By Business Domains
      ↓
Microservices
      ↓
Independent Scaling
Independent Deployment
      ↓
But...
      ↓
Network Calls
Operational Complexity
Eventual Consistency
```

The most important interview takeaway:

> Microservices are not a scalability solution. They are an organizational and architectural solution that enables large teams to build and deploy systems independently. The scalability benefits come later. This distinction is something many candidates miss and interviewers appreciate hearing.
