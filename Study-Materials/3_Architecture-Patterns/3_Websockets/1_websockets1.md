# Real-Time Systems

## Topics Covered

* Polling
* Long Polling
* WebSockets
* Server-Sent Events (SSE)
* Scaling WebSockets
* Real-World Architectures

For senior engineer interviews, WebSockets are by far the most important topic.

---

# The Problem Real-Time Systems Solve

Imagine WhatsApp.

Friend sends:

```text
Hi
```

Question:

```text
How does your phone know?
```

Option 1:

Your phone asks every second:

```text
Any messages?
```

This works.

But:

```text
1 Billion Users
```

asking every second means:

```text
1 Billion Requests/Second
```

Most requests:

```text
No New Message
```

Wasteful.

This is why real-time systems exist.

---

# Traditional HTTP

HTTP works as:

```text
Request
    ↓
Response
    ↓
Connection Closed
```

Example:

```text
Browser
   │
GET /profile
   │
   ▼
Server
   │
Returns Data
   ▼
Connection Closed
```

Important limitation:

```text
Server Cannot Talk First
```

The client must initiate communication.

This becomes a problem for chat, notifications, gaming, and live updates.

---

# Polling

The simplest solution.

Client repeatedly asks:

```text
Any updates?
```

---

## Architecture

```text
Client

   │

Every 5 Seconds

   ▼

Server
```

---

## Example

```text
00:00 → Any Messages?
00:05 → Any Messages?
00:10 → Any Messages?
00:15 → Any Messages?
```

Most responses:

```text
Nothing New
```

---

## Problems

Waste of:

* CPU
* Network
* Database
* Load Balancer Capacity

---

## Scale Example

Suppose:

```text
10 Million Users
```

poll every:

```text
5 Seconds
```

Requests generated:

```text
2 Million Requests/Second
```

even when no messages exist.

Very inefficient.

---

# Long Polling

An improvement over polling.

Instead of immediately responding:

```text
No Updates
```

the server keeps the request open.

---

## Flow

Client asks:

```text
Any Messages?
```

Server:

```text
No Messages Yet
```

but keeps waiting.

When a message arrives:

```text
Respond Immediately
```

---

## Architecture

```text
Client
    │
    ▼
Any Updates?

    │
    ▼

Server Holds Connection

    │
Update Arrives
    ▼

Response Sent
```

---

## Benefits

Fewer useless requests.

Lower latency than polling.

---

## Problems

Still repeatedly performs:

```text
HTTP Request
HTTP Response
HTTP Request
HTTP Response
```

Connections are constantly recreated.

Not ideal for:

* Chat
* Gaming
* Trading Platforms

---

# WebSockets

The most important real-time technology for interviews.

---

# Core Idea

Create a connection once.

Keep it open.

---

## Architecture

```text
Client
    ⇅
WebSocket
    ⇅
Server
```

---

Unlike HTTP:

```text
Request
Response
Close
```

WebSocket becomes:

```text
Connect Once
Talk Forever
```

---

# Full Duplex Communication

Important interview keyword.

HTTP:

```text
Client → Server
```

WebSocket:

```text
Client ⇄ Server
```

Both sides can send messages at any time.

This is called:

```text
Full Duplex Communication
```

---

# WhatsApp Example

Friend sends:

```text
Hi
```

Flow:

```text
Friend Phone

      │

      ▼

WhatsApp Server

      │

      ▼

Your Phone
```

Server immediately pushes:

```text
Hi
```

through the already-open WebSocket connection.

No polling.

No waiting.

---

# Why WebSockets Are Fast

HTTP:

```text
Open Connection
Send Request
Receive Response
Close Connection
```

for every interaction.

WebSocket:

```text
Open Once
Send Many Messages
Receive Many Messages
```

Much less overhead.

---

# WebSocket Handshake

Interviewers sometimes ask this.

Initially communication starts as HTTP.

Client sends:

```http
GET /chat
Upgrade: websocket
```

Server agrees.

Connection upgrades from:

```text
HTTP
```

to:

```text
WebSocket
```

After that:

```text
No More HTTP
```

Messages flow directly through the WebSocket connection.

---

# Real World Use Cases

### WhatsApp

```text
Messages
Typing Indicators
Read Receipts
Online Status
```

---

### Slack

```text
Messages
Channel Updates
Presence Information
```

---

### Discord

```text
Messages
Voice Status
Online Presence
```

---

### Trading Platforms

```text
Stock Prices
Market Updates
```

must arrive instantly.

---

### Multiplayer Games

```text
Player Movement
Shots Fired
Game State
```

requires extremely low latency.

---

# Scaling WebSockets

This is where senior-level discussions begin.

---

## Single Server

```text
Users

  ⇅
  ⇅
  ⇅

WebSocket Server
```

Works initially.

---

Problem:

```text
10 Million Users
```

cannot fit on one machine.

---

# Multi-Server Architecture

```text
Users

   │

   ▼

Load Balancer

   │

 ┌─┴────┬────┬────┐
 ▼      ▼    ▼    ▼

WS1    WS2  WS3  WS4
```

Each server maintains persistent client connections.

---

# Cross-Server Communication Problem

Suppose:

```text
User A → WS1
User B → WS4
```

User A sends a message.

How does WS1 notify WS4?

---

# Solution: Pub/Sub

Typically:

* Redis Pub/Sub
* Kafka
* RabbitMQ

Architecture:

```text
WS1

  │

  ▼

Redis/Kafka

  │

  ▼

WS4
```

WS4 then pushes the message to User B.

---

# WhatsApp-Style Architecture

```text
Phone

   ⇅

Load Balancer

   ⇅

WebSocket Servers

   ⇅

Redis/Kafka

   ⇅

Other WebSocket Servers
```

This is a common production architecture.

---

# Server-Sent Events (SSE)

Less important than WebSockets but occasionally discussed.

---

# What Is SSE?

Server can send data to client.

Client cannot send data through the same connection.

---

## Architecture

```text
Client  ←──── Server
```

One-way communication.

---

WebSocket:

```text
Client ⇄ Server
```

Two-way communication.

---

# Example

Live cricket score.

User only receives:

```text
Score Updates
```

No need to send data back.

SSE works well.

---

# SSE vs WebSocket

| Feature           | SSE     | WebSocket |
| ----------------- | ------- | --------- |
| Direction         | One-Way | Two-Way   |
| Real Time         | Yes     | Yes       |
| Chat Apps         | Poor    | Excellent |
| Notifications     | Good    | Excellent |
| Multiplayer Games | No      | Yes       |
| Complexity        | Lower   | Higher    |

---

# Polling vs Long Polling vs WebSocket

| Feature      | Polling   | Long Polling | WebSocket |
| ------------ | --------- | ------------ | --------- |
| Latency      | High      | Medium       | Very Low  |
| Requests     | Very High | Medium       | Very Low  |
| Scalability  | Poor      | Better       | Best      |
| Real Time    | Poor      | Moderate     | Excellent |
| Chat Systems | Bad       | Okay         | Excellent |

---

# When To Use What?

## Polling

Use when:

```text
Updates Are Rare
```

Examples:

* Weather Apps
* Reports
* Admin Dashboards

---

## Long Polling

Use when:

```text
WebSockets Not Available
```

Mostly legacy environments.

---

## WebSockets

Use when:

```text
Low Latency Required
```

Examples:

* WhatsApp
* Slack
* Discord
* Trading Platforms
* Multiplayer Games
* Collaborative Editors

---

## SSE

Use when:

```text
Server Push Only
```

Examples:

* Live Scores
* Stock Tickers
* Monitoring Dashboards
* Notification Streams

---

# Why Not Use WebSockets Everywhere?

Persistent connections consume:

* Memory
* CPU
* Network Resources

Need additional complexity:

* Connection Management
* Reconnection Logic
* Load Balancing
* Heartbeats
* Scaling Infrastructure

If updates happen once per hour:

```text
WebSocket = Overkill
```

Simple HTTP is better.

---

# Strong Interview Answer

### How would you build WhatsApp messaging?

> I would use WebSockets because messaging requires low-latency bidirectional communication. Each client maintains a persistent WebSocket connection to a WebSocket server. To scale horizontally, multiple WebSocket servers would be deployed behind a load balancer. Since users may connect to different WebSocket nodes, I would use a pub/sub system such as Redis or Kafka to route messages between servers.

---

# What Interviewers Expect You To Know

## WebSockets

```text
Persistent Connection
Full Duplex
Real-Time Communication
```

---

## Long Polling

```text
Server Holds Request
Responds When Update Arrives
```

---

## SSE

```text
Server → Client Only
```

---

## Scaling WebSockets

```text
Load Balancer
+
Multiple WebSocket Servers
+
Redis/Kafka Pub/Sub
```

---

# Senior Engineer Mental Model

```text
Need Real-Time Updates?
          │
          ▼
How Fast?
```

If:

```text
Milliseconds
```

↓

```text
WebSockets
```

If:

```text
Server Push Only
```

↓

```text
SSE
```

If:

```text
Legacy Environment
```

↓

```text
Long Polling
```

---

# Most Important Interview Takeaway

> WebSockets are the dominant solution for modern real-time systems because they provide low-latency, bidirectional communication over a persistent connection. Most large-scale chat, collaboration, gaming, and trading systems rely on WebSockets, typically combined with pub/sub systems such as Redis or Kafka for horizontal scalability.
