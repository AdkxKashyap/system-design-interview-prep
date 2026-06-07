# Stream Processing

## Topics Covered

### Basics

* Stream Processing
* Batch vs Stream Processing
* Filtering
* Aggregations
* Windowing

### Advanced

* Stateful Processing
* Event Time vs Processing Time
* Watermarks
* Kafka Streams
* Flink
* Fault Tolerance
* Real-Time Analytics

---

# The Problem Stream Processing Solves

Imagine Uber.

Every second:

```text
Ride Requested
Driver Assigned
Ride Started
Ride Completed
Payment Completed
```

Millions of events are generated continuously.

Questions:

```text
How many rides happened in the last minute?
```

```text
How much revenue did we make in the last 5 minutes?
```

```text
Detect fraudulent transactions immediately.
```

```text
Update live dashboard in real time.
```

---

# Traditional Approach

```text
Events
   ↓
Database
   ↓
Nightly Batch Job
   ↓
Report Tomorrow
```

Problem:

```text
Results Delayed
```

Could be:

```text
Hours
```

or

```text
Days
```

old.

---

# Modern Requirement

Businesses want:

```text
Real-Time Insights
```

This is why stream processing exists.

---

# What Is Stream Processing?

Think:

```text
Event
Event
Event
Event
Event
```

continuously arriving.

Instead of:

```text
Store Everything
Process Later
```

we:

```text
Process As Events Arrive
```

---

# Basic Architecture

```text
Kafka
   │
   ▼

Stream Processor

   │
   ▼

Output
```

---

# Real World Example

Food delivery platform.

Events:

```text
Order Created
Order Prepared
Order Delivered
```

Stream processor continuously computes:

```text
Orders Per Minute
```

Dashboard updates instantly.

No nightly batch required.

---

# Batch Processing vs Stream Processing

## Batch Processing

Process large chunks of data.

```text
Today's Orders
      ↓
Process At Midnight
```

Example:

```text
Generate Monthly Salary
```

Perfect batch workload.

---

## Stream Processing

Process continuously.

```text
Order Arrives
      ↓
Process Immediately
```

Example:

```text
Fraud Detection
```

Cannot wait until midnight.

---

# Mental Model

Batch:

```text
Pile Of Data
        ↓
Process
```

Stream:

```text
Data Arriving
        ↓
Process Continuously
```

---

# Core Architecture

```text
Applications

      │

      ▼

Kafka

      │

      ▼

Flink / Kafka Streams

      │

      ▼

Database
Dashboard
Alert System
```

---

# Example: Live Revenue Dashboard

Events:

```json
{
  "orderId":1,
  "amount":500
}
```

```json
{
  "orderId":2,
  "amount":1000
}
```

Processor:

```text
Revenue += amount
```

Output:

```text
Revenue = ₹1500
```

updated instantly.

---

# Why Not Just Consume Kafka?

Consumer:

```text
Read Event
Do Something
```

---

Stream Processor:

```text
Read Event
Aggregate
Join
Filter
Window
Transform
State Management
```

Much more powerful.

---

# Filtering

Events:

```text
OrderCreated
PaymentCompleted
UserLoggedIn
```

Need only:

```text
PaymentCompleted
```

Processor:

```text
Filter(eventType)
```

Output:

```text
PaymentCompleted
```

events only.

---

# Aggregations

Most common interview topic.

Events:

```text
Order1 ₹100
Order2 ₹200
Order3 ₹300
```

Processor:

```text
Sum
```

Output:

```text
₹600
```

---

Common Aggregations

```text
Count
Sum
Average
Min
Max
```

---

# The Window Problem

Question:

```text
How many orders happened?
```

Meaningless.

Need:

```text
How many orders happened
in the last 5 minutes?
```

This introduces:

```text
Windows
```

---

# Tumbling Window

Most common.

Example:

```text
12:00 - 12:05
12:05 - 12:10
12:10 - 12:15
```

Non-overlapping.

Orders:

```text
12:01
12:02
12:04
```

Count:

```text
3
```

for first window.

---

# Sliding Window

Windows overlap.

Example:

```text
Every Minute
Look Back 5 Minutes
```

At:

```text
12:05
```

calculate:

```text
12:00 - 12:05
```

At:

```text
12:06
```

calculate:

```text
12:01 - 12:06
```

Used heavily in:

```text
Fraud Detection
Monitoring
```

---

# Session Window

Based on user activity.

User active:

```text
10:00
10:05
10:10
```

Inactive:

```text
30 Minutes
```

New session starts.

Used in:

```text
Analytics
User Tracking
```

---

# Stateful Processing

Most important advanced concept.

Stateless:

```text
Event
   ↓
Output
```

No memory.

---

Stateful:

```text
Remember Previous Events
```

---

# Fraud Detection Example

Events:

```text
User A paid ₹500
User A paid ₹700
User A paid ₹1000
```

within:

```text
30 Seconds
```

Rule:

```text
> 3 Payments In 1 Minute
```

Generate:

```text
Fraud Alert
```

Requires remembering previous events.

That memory is called:

```text
State
```

---

# Event Time vs Processing Time

Suppose event occurred:

```text
12:00
```

but arrives:

```text
12:05
```

because network delayed it.

---

## Processing Time

Use:

```text
12:05
```

when event arrived.

---

## Event Time

Use:

```text
12:00
```

when event actually happened.

Most modern systems prefer:

```text
Event Time
```

because it is more accurate.

---

# Watermarks

Popular Flink interview topic.

Problem:

```text
Late Events Arrive
```

Question:

```text
How Long Should We Wait?
```

Watermark:

```text
I believe all events
before this timestamp
have arrived.
```

Then window closes.

---

# Kafka Streams

Think:

```text
Kafka
+
Stream Processing
```

Architecture:

```text
Kafka Topic

      │

      ▼

Kafka Streams App

      │

      ▼

Output Topic
```

---

Advantages

```text
Simple
Embedded
Easy To Operate
```

Best for:

```text
Moderate Scale
```

---

# Flink

Enterprise-grade stream processing engine.

Capabilities:

```text
State Management
Windowing
Watermarks
Event Time
Checkpointing
Fault Tolerance
```

Architecture:

```text
Kafka

  │

  ▼

Flink Cluster

  │

  ▼

Results
```

Used by:

* Uber
* Netflix
* LinkedIn
* Alibaba

---

# Fault Tolerance

Suppose:

```text
Flink Node Crashes
```

Question:

```text
Lose Data?
```

No.

---

Flink periodically saves:

```text
Checkpoint
```

Example:

```text
Revenue = ₹10M
```

saved.

Crash.

Restart.

Continue from checkpoint.

---

# Real World Examples

## Uber

```text
Ride Events
```

↓

```text
Flink
```

↓

```text
Live Surge Pricing
```

---

## Netflix

```text
Viewing Events
```

↓

```text
Stream Processing
```

↓

```text
Recommendations
Monitoring
```

---

## Amazon

```text
Purchase Events
```

↓

```text
Stream Processing
```

↓

```text
Fraud Detection
Inventory Updates
```

---

# Why Kafka Is Used For Analytics

Many engineers think:

```text
Kafka = Message Queue
```

But at scale:

```text
Kafka = Event Backbone
```

---

# Basic Messaging Use Case

```text
Order Service
      ↓
Kafka
      ↓
Email Service
```

Email consumes event.

Done.

---

# Analytics Use Case

Order created:

```json
{
  "event":"OrderCreated",
  "amount":500
}
```

Many consumers care.

```text
Email Service
Inventory Service
Fraud Detection
Analytics Service
Recommendation Service
```

Architecture:

```text
Order Service

      │

      ▼

Kafka

  ┌───┼────┬─────┬─────┐
  ▼   ▼    ▼     ▼     ▼

Email
Inventory
Fraud
Analytics
Recommendations
```

---

# Why Not Read Production Database?

Naive approach:

```text
Analytics Service
      ↓
Orders Table
```

Problem:

Analytics queries compete with:

```text
Order Creation
Payments
Inventory
```

This creates:

```text
Operational Load
```

on the production database.

---

# Better Approach

```text
Business Events
      ↓
Kafka
      ↓
Analytics
```

Analytics never touches production database.

---

# Do We Need A Separate Analytics Service?

Usually:

```text
YES
```

Production services focus on:

```text
Orders
Payments
Inventory
```

Analytics focuses on:

```text
Metrics
Aggregation
Dashboards
Reporting
```

Different responsibilities.

---

# Real Analytics Architecture

```text
Order Service

      │

      ▼

Kafka

      │

      ▼

Flink

      │

 ┌────┼─────┐
 ▼    ▼     ▼

Revenue
Dashboard
Alerts
```

---

# Revenue Dashboard Example

Events:

```text
₹500
₹1000
₹700
```

Flink computes:

```text
Revenue = ₹2200
```

continuously.

Stores result in:

```text
Redis
ClickHouse
Druid
Analytics DB
```

Dashboard reads:

```text
Precomputed Metrics
```

Very fast.

---

# Instagram Analytics Example

Events:

```text
Like
Comment
Follow
Share
Watch Video
```

Stream processing computes:

```text
Most Viewed Reel
Most Followed Creator
Trending Hashtag
```

No service queries production databases.

Everything comes from event streams.

---

# Why Kafka Beats Traditional MQ For Analytics

Traditional MQ mindset:

```text
Message Delivered
      ↓
Delete Message
```

Kafka mindset:

```text
Store Events
For Days/Weeks
```

---

New analytics service appears tomorrow.

Can replay:

```text
Last 30 Days Of Events
```

from Kafka.

Huge advantage.

---

Traditional queue:

```text
Events Already Gone
```

Impossible to replay.

---

# Real Company Architecture

```text
Applications

     │

     ▼

Kafka

     │

 ┌───┼────────┬─────────┐
 ▼   ▼        ▼         ▼

Email
Inventory
Flink
Fraud
```

Flink performs:

```text
Aggregations
Windowing
Analytics
Metrics
```

Outputs:

```text
Dashboards
Reports
Alerts
```

---

# Stream Processing Interview Answer

```text
Applications

      │

      ▼

Kafka

      │

      ▼

Flink

      │

 ┌────┼─────┐
 ▼    ▼     ▼

Alerts
Dashboard
Database
```

---

# What Interviewers Expect

Know:

### Stream Processing

```text
Process Events Continuously
```

---

### Aggregations

```text
Count
Sum
Average
```

---

### Windowing

```text
Tumbling
Sliding
Session
```

---

### Stateful Processing

```text
Remember Previous Events
```

---

### Event Time

```text
When Event Happened
```

---

### Watermarks

```text
Handle Late Events
```

---

### Kafka Streams

```text
Lightweight Stream Processing
```

---

### Flink

```text
Large Scale Stateful Processing
```

---

### Why Kafka For Analytics

```text
Event Backbone
Replayability
Decoupled Analytics
Real-Time Computation
```

---

# Senior Engineer Mental Model

```text
Business Events

      ↓

Kafka

      ↓

Flink / Kafka Streams

      ↓

Analytics
Dashboards
Fraud Detection
Monitoring
Recommendations
```

---

# Most Important Interview Takeaway

> Stream processing continuously computes business insights as events arrive rather than waiting for batch jobs. Kafka serves as the central event backbone, while engines such as Flink and Kafka Streams perform filtering, aggregation, windowing, and stateful processing. This enables real-time dashboards, fraud detection, monitoring, recommendations, and analytics without impacting transactional systems.
