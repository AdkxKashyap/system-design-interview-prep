# Chat Systems Using WebSockets

This note covers how modern chat applications such as:

* WhatsApp
* Slack
* Discord
* Microsoft Teams

typically work internally.

The focus is on:

* Message flow
* Presence tracking
* WebSocket architecture
* Message delivery
* Scaling
* Read receipts
* Typing indicators
* Multiple devices

---

# High-Level Architecture

Suppose:

```text
User A
User B
```

are connected to different WebSocket servers.

```text
User A
   │
   ▼

 WS1

   │
   ▼

 Kafka / Redis PubSub

   │
   ▼

 WS4

   │
   ▼

User B
```

---

# Step 1: User Connects

User opens WhatsApp.

Phone establishes a WebSocket connection.

```text
User A
   │
   ▼
 WS1
```

WS1 stores:

```text
userA -> WS1
```

---

User B connects.

```text
User B
   │
   ▼
 WS4
```

WS4 stores:

```text
userB -> WS4
```

---

# How Does The System Know User Locations?

This is usually handled by:

```text
Presence Service
```

backed by:

```text
Redis
```

Example:

```text
userA -> WS1
userB -> WS4
userC -> WS2
```

Think of this as:

```text
Connection Registry
```

---

When user disconnects:

```text
userA removed
```

from registry.

---

When reconnecting:

```text
userA -> WS3
```

updated.

---

# Step 2: User Sends Message

User A types:

```text
Hi
```

Phone sends through WebSocket.

```text
User A
   │
   ▼
 WS1
```

WS1 receives:

```json
{
  "from":"A",
  "to":"B",
  "message":"Hi"
}
```

---

# Step 3: Persist Message First

Production systems always save before delivery.

```text
Save To Database
```

Usually:

* Cassandra
* DynamoDB
* Message Store

Architecture:

```text
WS1
 │
 ▼

Message Store

 │
 ▼

Ack Success
```

---

# Why Save First?

Suppose:

```text
WS4 crashes
```

before delivery.

Without persistence:

```text
Message Lost
```

Bad.

With persistence:

```text
Message Still Exists
```

and can be delivered later.

---

# Step 4: Find Recipient Location

WS1 asks:

```text
Where is User B?
```

Presence Service responds:

```text
userB -> WS4
```

---

# Step 5: Route Message

WS1 publishes event.

```json
{
 "recipient":"B",
 "message":"Hi"
}
```

Through:

```text
Kafka
Redis PubSub
RabbitMQ
```

Architecture:

```text
WS1
 │
 ▼

Kafka

 │
 ▼

WS4
```

---

# Step 6: Deliver Message

WS4 already has an active socket.

```text
WS4
 │
 ▼
User B
```

Message appears instantly.

```text
Hi
```

---

# Complete Message Flow

```text
User A

   │

   ▼

WS1

   │

   ▼

Database

   │

   ▼

Kafka

   │

   ▼

WS4

   │

   ▼

User B
```

---

# Offline Users

Suppose:

```text
User B Offline
```

Registry:

```text
userB -> null
```

No active connection.

Message still gets stored.

```text
delivered = false
```

---

When User B reconnects:

```sql
SELECT *
FROM Messages
WHERE delivered = false
```

Pending messages are delivered.

---

# Read Receipts

User B reads message.

WS4 receives:

```text
Read
```

event.

WS4 publishes:

```text
Message Read
```

Flow:

```text
WS4
 │
 ▼

Kafka

 │
 ▼

WS1

 │
 ▼

User A
```

Result:

```text
Blue Tick
```

appears.

---

# Typing Indicators

User B starts typing.

WS4 receives:

```text
Typing
```

Usually:

```text
NOT stored in database
```

because it is temporary.

WS4 publishes:

```text
User B Typing
```

WS1 pushes:

```text
Typing...
```

to User A.

---

# Presence / Online Status

When WebSocket connects:

```text
User Online
```

Presence Service updates:

```text
userB = online
```

---

When socket disconnects:

```text
userB = offline
```

Friends query status from:

```text
Redis Presence Store
```

---

# Why Redis Is Common

Presence data changes constantly.

Examples:

```text
Online
Offline
Typing
Last Seen
```

Redis is ideal because:

```text
Fast
In-Memory
Temporary Data
```

---

# Multiple Devices

Suppose User B uses:

```text
Phone
Laptop
Tablet
```

Registry becomes:

```text
userB ->
    WS4:Phone
    WS2:Laptop
    WS7:Tablet
```

Message delivered to:

```text
All Active Devices
```

---

# Sticky Sessions

Important interview keyword.

Suppose:

```text
User A
```

connects to:

```text
WS1
```

Every WebSocket message must continue reaching:

```text
WS1
```

Load balancer uses:

```text
Sticky Session
```

or:

```text
Connection Affinity
```

Otherwise:

```text
Message1 -> WS1
Message2 -> WS3
Message3 -> WS5
```

Chaos.

---

# Heartbeats

How does server know a user disconnected?

Suppose mobile network dies.

No disconnect event arrives.

WebSocket periodically sends:

```text
PING
```

Client responds:

```text
PONG
```

If:

```text
No PONG
```

for a timeout period:

```text
Mark User Offline
```

---

# Real WhatsApp-Level Architecture

```text
User A

   │

   ▼

Load Balancer

   │

   ▼

WS1

   │

   ├─────────────┐
   ▼             ▼

Message DB     Redis Presence

   │

   ▼

Kafka

   │

   ▼

WS4

   │

   ▼

User B
```

---

# Components And Their Responsibilities

## WebSocket Servers

Responsible for:

```text
Maintaining Connections
Sending Messages
Receiving Messages
```

---

## Redis Presence Service

Responsible for:

```text
User Location Mapping
Online Status
Typing Indicators
Last Seen
```

---

## Kafka / PubSub

Responsible for:

```text
Cross-Server Routing
Event Delivery
Scalable Communication
```

---

## Message Database

Responsible for:

```text
Durable Storage
Offline Messages
Message History
```

---

# Things Senior Engineers Mention

### WebSockets

```text
Persistent Connections
```

---

### Presence Service

```text
user -> websocket mapping
```

---

### Redis

```text
online users
typing indicators
last seen
```

---

### Kafka / PubSub

```text
cross-server message routing
```

---

### Database

```text
durable message storage
```

---

### Sticky Sessions

```text
keep socket on same server
```

---

### Heartbeats

```text
PING/PONG
```

---

# Strong Interview Answer

> Each user maintains a persistent WebSocket connection to a WebSocket server. A presence service tracks which WebSocket node each user is connected to. When a message is sent, it is first persisted for durability, then routed through a pub/sub system such as Kafka or Redis to the recipient's WebSocket server. That server pushes the message over the existing WebSocket connection. Presence information, typing indicators, and online status are typically stored in Redis, while durable messages are stored in a database such as Cassandra or DynamoDB.

---

# Senior Engineer Mental Model

```text
Client
  ⇅
WebSocket Server
  ⇅
Presence Service (Redis)
  ⇅
Kafka / PubSub
  ⇅
Other WebSocket Servers
  ⇅
Recipients
```

Most important takeaway:

> WebSockets provide the real-time connection, Redis tracks where users are connected, Kafka routes messages between WebSocket servers, and the database ensures messages are never lost.
