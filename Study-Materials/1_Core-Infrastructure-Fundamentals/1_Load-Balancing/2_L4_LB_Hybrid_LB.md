# How L4 Load Balancers Work + Hybrid L4/L7 Architecture

# 1. First Understand the Problem

Suppose millions of users try to access:

```text
https://youtube.com
```

If all traffic directly hits one backend server:
- server overloads
- latency increases
- crashes happen
- scalability impossible

So production systems place:
```text
Load Balancers
```

in front of backend servers.

---

# 2. What is an L4 Load Balancer?

L4 = Layer 4 of OSI model.

Layer 4 handles:
- TCP
- UDP

L4 load balancer works using:
- source IP
- destination IP
- source port
- destination port
- protocol

It DOES NOT understand:
- URLs
- HTTP headers
- cookies
- API paths

Because those belong to:
```text
Layer 7 (Application Layer)
```

---

# 3. Core Mental Model

L4 load balancer is fundamentally:

> Connection-aware packet routing

NOT request-aware routing.

---

# 4. What Happens When User Opens Website?

Suppose user types:

```text
https://youtube.com
```

in browser.

Internally MANY things happen.

---

# 5. Step 1 — DNS Resolution

Browser first needs IP address.

Browser asks DNS:

```text
What is IP of youtube.com?
```

DNS returns:

```text
142.x.x.x
```

This IP usually belongs to:
```text
Load Balancer
```

NOT actual backend server.

---

# 6. Why Not Expose Backend Server Directly?

Suppose backend IP exposed:

```text
10.0.0.5
```

Problems:
- one server overloaded
- no scalability
- failures impact users directly
- backend topology exposed

So traffic first reaches Load Balancer.

---

# 7. Step 2 — TCP Connection Begins

Before HTTP request:
- TCP connection created first

This is VERY important.

---

# 8. What is TCP?

TCP is:
> a reliable communication channel between two machines.

It guarantees:
- ordered delivery
- retransmission of lost packets
- corruption detection

Think of TCP like:
```text
Reliable phone call
```

---

# 9. TCP 3-Way Handshake

Client says:

```text
SYN
```

Meaning:
```text
Can we start communication?
```

Server replies:

```text
SYN-ACK
```

Meaning:
```text
Yes
```

Client replies:

```text
ACK
```

Meaning:
```text
Okay let's start
```

Now TCP connection established.

---

# 10. Where Does TCP Connection Go?

Connection goes to:

```text
L4 Load Balancer IP
```

NOT application server.

---

# 11. What Does L4 Load Balancer See?

L4 LB sees:

| Field | Example |
|---|---|
| Source IP | User IP |
| Destination IP | LB IP |
| Source Port | 52341 |
| Destination Port | 443 |
| Protocol | TCP |

It DOES NOT see:
- URLs
- cookies
- HTTP headers

Because HTTP payload not parsed.

---

# 12. What is a Port?

One machine runs multiple services:
- browser
- DB
- APIs
- SSH

Ports identify:
> which application should receive traffic.

Examples:

| Port | Meaning |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |
| 3306 | MySQL |

---

# 13. How L4 Load Balancer Chooses Server

L4 LB runs balancing algorithm:
- Round Robin
- Least Connections
- IP Hash
- Consistent Hashing

Suppose:

```text
L7-2 selected
```

---

# 14. What Does “Forwarding Traffic” Mean?

This is VERY important.

Original packet:

| Source | Destination |
|---|---|
| Client IP | LB IP |

L4 LB modifies packet:

| Source | Destination |
|---|---|
| Client IP | L7-2 IP |

Then forwards internally.

This is:
```text
Packet Forwarding
```

---

# 15. NAT (Network Address Translation)

VERY IMPORTANT production concept.

---

# What is NAT?

NAT means:
> modifying packet IP information before forwarding.

---

# Example

Original packet:

| Source | Destination |
|---|---|
| Client IP | LB IP |

LB rewrites destination:

| Source | Destination |
|---|---|
| Client IP | Backend IP |

Now backend receives traffic.

---

# Why NAT Needed?

Because:
- backend servers hidden
- security improved
- topology abstracted
- easier scaling

Clients never directly see backend servers.

---

# 16. Why L4 is Extremely Fast

L4 does NOT:
- parse HTTP
- inspect cookies
- decrypt HTTPS
- inspect payload

It only forwards based on:
- IP
- ports
- protocol

This requires very little CPU work.

---

# 17. Why L4 Excellent for Massive Scale

Suppose:

```text
50 million TCP connections
```

L7 parsing all traffic immediately would be expensive.

L4 efficiently distributes:
- connections
- packets

with minimal overhead.

---

# 18. What Happens Next?

After L4 forwards connection:

```text
Client
   ↓
L4 LB
   ↓
L7 Proxy
```

Now:
> L7 processing begins.

---

# 19. What is HTTP?

HTTP is application-layer protocol.

Contains:
- URL
- headers
- cookies
- methods
- body

Example:

```http
GET /videos/123 HTTP/1.1
Host: youtube.com
Authorization: Bearer xyz
```

THIS is what L7 understands.

L4 cannot understand this.

---

# 20. What Does L7 Load Balancer Do?

L7 can inspect:
- URL
- headers
- cookies
- request path

Example routing:

```text
/videos → Video Service
/chat → Chat Service
/images → Image Service
```

This is:
```text
Application-aware routing
```

---

# 21. Why Not Directly Use L7 Only?

Because L7 processing expensive.

L7 must:
- parse HTTP
- inspect headers
- decrypt HTTPS
- inspect cookies

Consumes:
- CPU
- memory

At internet scale:
> one L7 layer alone becomes bottleneck.

---

# 22. Why Hybrid Architecture Exists

Hybrid architecture combines:

| Layer | Responsibility |
|---|---|
| L4 | Efficient connection scaling |
| L7 | Intelligent HTTP routing |

---

# 23. Hybrid Architecture Flow

```text
Client
   ↓
L4 Load Balancer
   ↓
L7 Reverse Proxy
   ↓
Application Servers
```

---

# 24. Step-by-Step Hybrid Flow

## Step 1

Client creates TCP connection.

---

## Step 2

TCP packets reach L4 LB.

---

## Step 3

L4 LB selects one L7 proxy.

Example:

```text
L7-2
```

---

## Step 4

L4 forwards packets using NAT.

---

## Step 5

L7 proxy receives TCP stream.

---

## Step 6

L7 decrypts HTTPS traffic.

This is:
```text
TLS/SSL Termination
```

---

## Step 7

L7 parses HTTP request.

Example:

```http
GET /api/payment
```

---

## Step 8

L7 routes request to proper microservice.

```text
/payment → Payment Service
/search → Search Service
```

---

## Step 9

Application server processes request.

May involve:
- cache lookup
- DB queries
- business logic

---

## Step 10

Response travels back:

```text
App Server
   ↓
L7 Proxy
   ↓
L4 LB
   ↓
Client
```

---

# 25. What is SSL/TLS Termination?

HTTPS traffic encrypted.

Encrypted traffic cannot be inspected.

L7 must inspect:
- URLs
- headers

Therefore:
> HTTPS traffic must be decrypted.

Usually:
```text
L7 performs TLS termination
```

---

# 26. Why TLS Termination Important?

Without centralized TLS termination:
- every backend handles encryption
- huge CPU overhead

L7 centralizes crypto work.

Improves efficiency.

---

# 27. What is Reverse Proxy?

Reverse proxy sits in front of backend servers.

Client thinks talking to:

```text
youtube.com
```

But reverse proxy internally routes traffic to:
- microservices
- APIs
- backend servers

Client never sees actual servers.

Examples:
- NGINX
- Envoy
- HAProxy

---

# 28. Connection Tracking in L4

Suppose TCP connection assigned to:

```text
L7-2
```

All packets of that TCP session MUST continue reaching:
```text
L7-2
```

Otherwise:
- TCP breaks
- sequence mismatch happens

Therefore L4 maintains:

```text
Connection Table
```

Example:

| Client Connection | Backend |
|---|---|
| 10.0.0.5:1234 | L7-2 |
| 10.0.0.8:5678 | L7-1 |

---

# 29. Health Checks

L4 continuously checks:
- Is backend alive?
- Is port responding?

Example:

```text
TCP connect check on port 443
```

If server dead:
```text
Remove from traffic rotation
```

---

# 30. Real Production Example

Large companies often use:

```text
Users
   ↓
Global DNS
   ↓
L4 Load Balancer
   ↓
NGINX / Envoy L7 Proxies
   ↓
Microservices
   ↓
Databases / Caches
```

Used by:
- Google
- Netflix
- Meta
- Amazon

---

# 31. Real Engineering Analogy

Think of airport.

---

# L4 Load Balancer

Airport entry gate.

Goal:
- distribute people quickly

No deep inspection.

---

# L7 Load Balancer

Security + immigration.

Now:
- documents inspected
- routing decisions made
- validation happens

More intelligent but slower.

---

# 32. Important Terminologies

| Term | Meaning |
|---|---|
| Packet | Small chunk of network data |
| TCP | Reliable communication protocol |
| Socket | IP + Port combination |
| NAT | Packet IP rewriting |
| Forwarding | Sending packet to another server |
| Routing | Choosing server destination |
| Reverse Proxy | Proxy in front of backend servers |
| TLS/SSL | Encryption for HTTPS |

---

# 33. Most Important Interview Insight

Hybrid architecture exists because:

> L4 optimizes connection scalability.
> L7 optimizes application-aware routing.

At internet scale:
you usually need BOTH.

---

# 34. Senior-Level Understanding

Junior engineer says:
> “L4 forwards traffic to L7.”

Senior engineer says:
> “At high scale, an L4 layer efficiently distributes TCP connections across L7 proxies, which then terminate TLS and perform application-aware HTTP routing to backend microservices.”