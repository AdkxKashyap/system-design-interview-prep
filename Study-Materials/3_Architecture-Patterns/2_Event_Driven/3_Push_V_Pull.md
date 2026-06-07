# Push vs Pull Systems

This is a broader concept than fanout.

Push vs Pull appears everywhere:

* Kafka
* Databases
* Caches
* Notifications
* WebSockets
* Feed Systems
* Replication
* Streaming Systems

A senior engineer eventually realizes that many architecture decisions boil down to one question:

> Should the producer send data proactively, or should the consumer ask for it?

---

# The Fundamental Problem

Imagine two systems:

```text id="xj7qf2"
Producer
```

and

```text id="jgh7f4"
Consumer
```

Example:

```text id="n63r6l"
Instagram Server
      ↓
User Phone
```

Question:

```text id="hjhdy5"
How does data move?
```

---

# Push Model

Producer sends data.

Consumer does nothing.

---

# Architecture

```text id="t5v6i0"
Producer
    │
    ▼
Consumer
```

Producer decides:

```text id="qmx6fd"
When To Send
```

---

# Real World Example

Instagram notification.

Someone likes your photo.

Instagram immediately sends:

```text id="8fkjko"
"You received a new like"
```

to your phone.

Phone never asked.

Instagram pushed it.

This is:

```text id="rqjj4i"
Push
```

---

# Pull Model

Consumer asks for data.

Producer waits.

---

# Architecture

```text id="c4vjlwm"
Consumer
    │
    ▼
Producer
```

Consumer decides:

```text id="tjlwmn"
When To Fetch
```

---

# Real World Example

Email.

You open Gmail.

Phone asks:

```text id="kws3y2"
Any new emails?
```

Server responds.

No request.

No data.

This is:

```text id="5djlwm"
Pull
```

---

# Why Does This Matter?

Because systems face two competing goals:

```text id="tq2pgp"
Freshness
```

vs

```text id="opqk9w"
Scalability
```

---

# Push Optimizes Freshness

Data arrives instantly.

---

# Example

```text id="r4qgkj"
WhatsApp Message
```

You expect:

```text id="e7zqsi"
Immediate Delivery
```

not:

```text id="8bjsz8"
Check Every Minute
```

Push is excellent for:

```text id="r7t7af"
Real-Time Systems
```

---

# Pull Optimizes Scalability

Producer does less work.

Consumer asks only when needed.

---

# Example

```text id="q67r0x"
Weather App
```

Do you need updates every second?

No.

Open app.

Fetch latest weather.

Done.

Pull avoids unnecessary traffic.

---

# Real World Example: YouTube

Suppose:

```text id="w18n0r"
New Video Uploaded
```

Push approach:

```text id="m50lcf"
Notify 2 Billion Users
```

Ridiculous.

Instead:

```text id="ihfj3e"
User Opens YouTube
```

↓

```text id="7qhd83"
Fetch Recommendations
```

This is pull.

Much cheaper.

---

# Push Example: WhatsApp

Message sent:

```text id="sdr4z3"
Hi
```

WhatsApp immediately delivers.

```text id="jlwmq2"
Sender
   │
   ▼
Server
   │
   ▼
Receiver
```

No polling.

No refreshing.

Push is perfect.

---

# Pull Example: Dashboard

Imagine company dashboard.

Metrics:

```text id="b3w3ik"
Revenue
Orders
Users
```

User opens dashboard.

Browser requests:

```text id="jlwm6d"
Give Me Latest Metrics
```

Pull.

Simple.

Efficient.

---

# Push Problems

Push sounds amazing.

Why not always use it?

Suppose:

```text id="jlwm3h"
100 Million Users
```

Producer must track:

```text id="jlwm89"
Who Is Connected?
Where To Send?
Delivery State?
```

Huge complexity.

---

# Example

Celebrity posts.

Followers:

```text id="jlwm40"
100 Million
```

Push system:

```text id="jlwm41"
Send To Everyone
```

Massive load.

Producer becomes bottleneck.

---

# Pull Problems

Suppose:

```text id="jlwm42"
Client Polls Every Second
```

100M users.

Now server receives:

```text id="jlwm43"
100M Requests Every Second
```

even if nothing changed.

Wasteful.

---

# Polling Problem

Classic pull architecture:

```text id="jlwm44"
Client
   │
Every 5 Seconds
   ▼
Server
```

Most requests:

```text id="jlwm45"
Nothing Changed
```

Still consume:

* CPU
* Network
* Database

This is why polling scales poorly.

---

# Long Polling

A compromise.

Client asks:

```text id="jlwm46"
Any Updates?
```

Server waits.

If update arrives:

```text id="jlwm47"
Return Response
```

Otherwise:

```text id="jlwm48"
Timeout
```

Used before WebSockets became common.

---

# WebSockets

Modern push systems.

Connection stays open.

```text id="jlwm49"
Client ───────── Server
```

persistent connection.

Producer can push instantly.

Used by:

* WhatsApp
* Slack
* Discord
* Trading Platforms

---

# Kafka Example

Interesting case.

Most people assume Kafka is push.

Actually:

```text id="jlwm50"
Kafka Uses Pull Consumers
```

Consumer asks:

```text id="jlwm51"
Give Me Next Messages
```

Broker responds.

Why?

Because consumers process data at different speeds.

Fast consumer:

```text id="jlwm52"
Fetch More
```

Slow consumer:

```text id="jlwm53"
Fetch Slowly
```

Kafka doesn't overwhelm consumers.

---

# Database Example

SQL queries:

```sql id="jlwm54"
SELECT *
FROM users
```

Database does not push.

Application asks.

Pure pull.

---

# Cache Example

Redis cache.

```text id="jlwm55"
Cache Miss
```

Application asks Redis.

Pull.

---

# Push vs Pull Comparison

| Factor                | Push      | Pull               |
| --------------------- | --------- | ------------------ |
| Who Initiates?        | Producer  | Consumer           |
| Latency               | Very Low  | Higher             |
| Freshness             | Excellent | Depends On Polling |
| Scalability           | Harder    | Easier             |
| Producer Load         | High      | Low                |
| Consumer Control      | Low       | High               |
| Real-Time Systems     | Excellent | Poor               |
| Simple Data Retrieval | Overkill  | Excellent          |

---

# When To Use Push

Use when:

```text id="jlwm56"
Data Must Arrive Immediately
```

Examples:

* WhatsApp
* Slack
* Trading Platforms
* Live Sports Scores
* Real-Time Notifications
* Multiplayer Games

---

# When To Use Pull

Use when:

```text id="jlwm57"
Consumer Doesn't Need Constant Updates
```

Examples:

* Dashboards
* Search
* Product Catalogs
* Reports
* Analytics
* Weather Apps

---

# Hybrid Systems

Most large systems use both.

---

# Instagram

Push:

```text id="jlwm58"
Notifications
```

Pull:

```text id="jlwm59"
Home Feed
Profile Data
Comments
```

---

# YouTube

Push:

```text id="jlwm60"
Live Stream Notifications
```

Pull:

```text id="jlwm61"
Homepage
Search
Recommendations
```

---

# The Real Question

Every architecture decision becomes:

```text id="jlwm62"
How Fresh Must Data Be?
```

If answer is:

```text id="jlwm63"
Milliseconds
```

↓

```text id="jlwm64"
Push
```

If answer is:

```text id="jlwm65"
Minutes
Hours
User Initiated
```

↓

```text id="jlwm66"
Pull
```

---

# Senior Engineer Mental Model

Think:

```text id="jlwm67"
Producer Has Data
```

Question:

```text id="jlwm68"
Who Should Initiate Transfer?
```

If:

```text id="jlwm69"
Producer Initiates
```

↓

```text id="jlwm70"
Push
```

If:

```text id="jlwm71"
Consumer Initiates
```

↓

```text id="jlwm72"
Pull
```

---

# Fanout vs Push/Pull

Many engineers confuse these concepts.

They are related but not identical.

---

## Fanout

Answers:

```text id="jlwm73"
When Is Feed Computation Performed?
```

---

## Push/Pull

Answers:

```text id="jlwm74"
How Is Data Delivered?
```

---

## Fanout On Write ≈ Push

When someone posts:

```text id="jlwm75"
User A Posts
```

the system immediately pushes the post into followers' feeds.

```text id="jlwm76"
Post Created
      ↓
Push To Feed Tables
```

This is why people often say:

```text id="jlwm77"
Fanout On Write = Push Model
```

---

## Fanout On Read ≈ Pull

When someone posts:

```text id="jlwm78"
User A Posts
```

the system stores the post once.

Nothing is pushed.

Later:

```text id="jlwm79"
User Opens Feed
```

The system pulls posts from followed users and builds the feed.

```text id="jlwm80"
Open Feed
      ↓
Pull Posts
      ↓
Generate Timeline
```

This is why people often say:

```text id="jlwm81"
Fanout On Read = Pull Model
```

---

# The Real Difference

Push/Pull is a broader concept.

It appears in:

* Kafka
* Notifications
* Databases
* WebSockets
* APIs
* Replication

Fanout is specifically a feed-generation strategy.

---

# Final Interview Takeaway

> Push systems optimize for freshness and low latency at the cost of scalability and complexity. Pull systems optimize for scalability and simplicity at the cost of freshness. Fanout-on-write is similar to a push model because work is done proactively, while fanout-on-read is similar to a pull model because work is deferred until the consumer requests it. Most large-scale systems use a hybrid approach depending on latency and scalability requirements.
