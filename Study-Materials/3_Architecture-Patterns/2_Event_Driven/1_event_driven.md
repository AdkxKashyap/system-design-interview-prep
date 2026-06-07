# Event-Driven Architecture (EDA)

## The Problem Event-Driven Architecture Solves

Imagine an e-commerce order flow.

Customer clicks:

```text
Place Order
```

Traditional synchronous design:

```text
Order Service
      │
      ▼
Inventory Service
      │
      ▼
Payment Service
      │
      ▼
Notification Service
```

Order Service waits for:

```text
Inventory
```

then waits for:

```text
Payment
```

then waits for:

```text
Email
```

Question:

What happens if Notification Service is down?

```text
Order Service
      │
      ▼
Inventory ✅
Payment ✅
Email ❌
```

Should the customer's order fail?

No.

But in tightly coupled systems it often does.

This is the first motivation for Event-Driven Architecture.

---

# What Is Event-Driven Architecture?

Instead of saying:

```text
Do This
```

services say:

```text
This Happened
```

Traditional Request/Response:

```text
Order Service
      │
      ▼
Inventory Service
```

means:

```text
Reserve inventory now.
```

Event-Driven:

```text
Order Created
```

means:

```text
Something happened.
Do whatever you need.
```

The publisher doesn't care who reacts.

---

# Real World Analogy

Imagine a wedding invitation.

You announce:

```text
Wedding Happening On Sunday
```

You don't individually call:

```text
Photographer
Caterer
Decorator
Musicians
```

and coordinate everything.

You simply publish information.

People who care react.

That's Event-Driven Architecture.

---

# What Is An Event?

An event represents:

```text
Something That Already Happened
```

Examples:

```text
Order Created
Payment Completed
User Registered
Ride Booked
Product Added To Cart
```

Notice the tense.

Good event:

```text
Order Created
```

Bad event:

```text
Create Order
```

because that is a command.

---

# Core Components

## Producer

Creates event.

Example:

```text
Order Service
```

publishes:

```text
Order Created
```

---

## Event Broker

Carries event.

Examples:

* Kafka
* RabbitMQ
* Pulsar

---

## Consumer

Receives event.

Examples:

```text
Inventory Service
Payment Service
Email Service
Analytics Service
```

---

Architecture:

```text
Order Service

      │

      ▼

Kafka Topic

 ┌────┼────┬────┐
 ▼    ▼    ▼    ▼

Inventory
Payment
Email
Analytics
```

---

# Real Amazon Example

Customer buys:

```text
MacBook
```

Order Service creates order.

Publishes:

```json
{
  "event":"OrderCreated",
  "orderId":"123"
}
```

Order Service is done.

It doesn't wait.

Consumers react independently.

Inventory Service:

```text
Reserve Stock
```

Payment Service:

```text
Charge Customer
```

Email Service:

```text
Send Confirmation
```

Analytics Service:

```text
Track Revenue
```

Order Service knows nothing about these services.

---

# Loose Coupling

Without events:

```text
Order Service
      │
      ├── Payment
      ├── Inventory
      ├── Email
      ├── Analytics
      └── Fraud
```

Order Service knows everyone.

With events:

```text
Order Service
      │
      ▼
OrderCreated Event
```

Consumers decide whether they care.

This is called:

```text
Loose Coupling
```

---

# Why Companies Love Event-Driven Systems

Suppose CEO says:

```text
Track Customer Purchase Metrics
```

Traditional architecture:

Modify:

```text
Order Service
```

Event-driven architecture:

Create:

```text
Analytics Consumer
```

Subscribe:

```text
OrderCreated
```

Done.

No Order Service changes.

---

# Netflix Example

When you start a movie:

```text
Play Movie
```

produces events.

Examples:

```text
PlaybackStarted
PlaybackPaused
PlaybackStopped
```

Consumers:

```text
Recommendation Engine
Analytics
Billing
Monitoring
```

react independently.

---

# Uber Example

User books ride.

Produces:

```text
RideRequested
```

Consumers:

```text
Driver Matching
Pricing
Notifications
Analytics
Fraud Detection
```

all react.

Ride Service doesn't call them directly.

---

# Eventual Consistency

Suppose:

```text
Order Created
```

published.

Inventory updates immediately.

Payment is delayed.

Current state:

```text
Order = Created
Inventory = Reserved
Payment = Pending
```

System is inconsistent.

Temporarily.

Eventually:

```text
Payment = Success
```

Now everything aligns.

This is:

```text
Eventual Consistency
```

and is normal in event-driven systems.

---

# Synchronous vs Event-Driven

## Synchronous

```text
Order
  ↓
Inventory
  ↓
Payment
  ↓
Email
```

Pros:

```text
Simple
Immediate Consistency
```

Cons:

```text
Tight Coupling
Slow
Failure Propagation
```

---

## Event-Driven

```text
Order
  ↓
Kafka
  ↓
Consumers
```

Pros:

```text
Scalable
Loose Coupling
Independent Evolution
```

Cons:

```text
Eventual Consistency
Debugging Complexity
```

---

# Fan-Out Pattern

One event.

Many consumers.

Example:

```text
Order Created
```

Consumers:

```text
Inventory
Payment
Email
Analytics
Fraud
```

Architecture:

```text
                OrderCreated

                      │

                      ▼

                    Kafka

     ┌──────┬──────┬──────┬──────┐
     ▼      ▼      ▼      ▼      ▼

 Inventory Payment Email Analytics Fraud
```

This is probably the most common event-driven pattern.

---

# Choreography vs Orchestration

A common misconception:

```text
Choreography = Event Driven
Orchestration = Synchronous
```

This is NOT true.

These are different dimensions.

---

# Dimension 1: Communication Style

## Synchronous

```text
Order
   ↓
Payment
   ↓
Inventory
   ↓
Email
```

Request/Response.

---

## Asynchronous

```text
Order
   ↓
Kafka
   ↓
Consumers
```

Event-driven.

---

# Dimension 2: Workflow Ownership

## Choreography

```text
Nobody
```

owns the workflow.

Each service reacts independently.

---

## Orchestration

```text
One Coordinator
```

owns the workflow.

---

# Choreography Example

Order Service publishes:

```text
OrderCreated
```

Inventory Service listens.

Publishes:

```text
InventoryReserved
```

Payment Service listens.

Publishes:

```text
PaymentSucceeded
```

Email Service listens.

Publishes:

```text
EmailSent
```

Flow:

```text
OrderCreated
      ↓
InventoryReserved
      ↓
PaymentSucceeded
      ↓
EmailSent
```

Nobody coordinates.

Services react to events.

Advantages:

```text
Very Loosely Coupled
```

Disadvantages:

```text
Hard To Understand Flow
```

---

# Orchestration Example (Synchronous)

```text
Order Orchestrator
       │
       ▼

Reserve Inventory

       │
       ▼

Charge Payment

       │
       ▼

Send Email
```

Architecture:

```text
Order Orchestrator
      │
      ├── Inventory Service
      ├── Payment Service
      └── Email Service
```

Uses:

```text
HTTP/gRPC
```

Very common.

---

# Orchestration Example (Event Driven)

The orchestrator still exists.

But communication happens through events.

Flow:

```text
Order Orchestrator

       │

       ▼

ReserveInventory Command
```

Inventory Service processes.

Publishes:

```text
InventoryReserved
```

Orchestrator listens.

Then publishes:

```text
ChargePayment
```

Payment Service processes.

Publishes:

```text
PaymentSucceeded
```

Orchestrator listens.

Then publishes:

```text
SendEmail
```

Architecture:

```text
Order Orchestrator

       │

       ▼

ReserveInventory

       │

       ▼

InventoryReserved

       │

       ▼

ChargePayment

       │

       ▼

PaymentSucceeded

       │

       ▼

SendEmail
```

Notice:

```text
Still Event Driven
```

but:

```text
Workflow Centrally Controlled
```

This is:

```text
Event-Driven Orchestration
```

---

# Real World Example: Saga Pattern

Many distributed transaction implementations use:

```text
Order Saga Orchestrator
```

to coordinate:

```text
Order Service
Inventory Service
Payment Service
Shipping Service
```

Communication often happens through:

```text
Kafka
RabbitMQ
```

but it is still orchestration because one component owns the workflow.

---

# Easy Interview Rule

Ask:

```text
Who decides what happens next?
```

If answer is:

```text
Each Service Decides
```

↓

```text
Choreography
```

If answer is:

```text
Central Coordinator Decides
```

↓

```text
Orchestration
```

Notice we never asked:

```text
HTTP?
Kafka?
RabbitMQ?
```

Those are implementation details.

---

# When To Use Choreography

Good for:

```text
OrderCreated
       ↓
Analytics
Inventory
Email
Fraud
```

Simple fan-out workflows.

---

# When To Use Orchestration

Good for:

```text
Order
Payment
Inventory
Shipping
Refund
Compensation
```

Complex workflows where:

* Steps must happen in order
* Rollbacks are required
* Business processes are complicated

---

# What Interviewers Expect

Know:

### Event

```text
Something Happened
```

### Producer

```text
Publishes Event
```

### Broker

```text
Kafka
RabbitMQ
```

### Consumer

```text
Processes Event
```

### Benefits

```text
Loose Coupling
Scalability
Independent Evolution
```

### Trade-Offs

```text
Eventual Consistency
Debugging Difficulty
Operational Complexity
```

---

# Strong Interview Answer

> Event-driven architecture allows services to communicate asynchronously through events rather than direct service-to-service calls. This reduces coupling, improves scalability, and allows new consumers to be added without modifying existing services. For example, an Order Service can publish an OrderCreated event that is consumed independently by Inventory, Payment, Notification, and Analytics services. The primary trade-off is eventual consistency and increased operational complexity.

---

# Senior Engineer Mental Model

```text
Business Event Occurs
         ↓
Publish Event
         ↓
Broker
         ↓
Multiple Consumers React
         ↓
Loose Coupling
Independent Scaling
```

Most important takeaway:

> Event-Driven Architecture is not about Kafka. It's about designing systems around business events instead of direct service-to-service dependencies. Kafka is simply one implementation choice.

---

# Choreography vs Orchestration Mental Model

Think:

```text
Event Driven
      ↓
How Services Communicate
```

and

```text
Orchestration vs Choreography
      ↓
Who Owns The Workflow
```

These are independent decisions.

You can absolutely have:

```text
Event Driven + Orchestration
```

and many large production systems, especially Saga-based workflows, do exactly that.
