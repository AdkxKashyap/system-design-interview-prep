# Load Balancers + API Gateway + TLS Termination — Real Production Architecture

# 1. The Big Picture

After learning:

* Load Balancers
* Reverse Proxies
* API Gateways
* TLS Termination

the natural question becomes:

> Where does each component actually sit in a real production architecture?

Understanding this is important because interviewers often ask:

> Walk me through how a request reaches your microservice.

---

# 2. Responsibilities of Each Component

Think of a shopping mall.

---

# Security Gate

Checks:

* IDs
* security
* bags

Equivalent:

```text
API Gateway
```

---

# Reception Desk

Directs visitors to:

* electronics
* clothing
* food court

Equivalent:

```text
Routing Layer
```

(often implemented inside API Gateway)

---

# Multiple Billing Counters

Distributes customers among:

* Counter 1
* Counter 2
* Counter 3

Equivalent:

```text
Load Balancer
```

---

# Key Understanding

Load Balancer and API Gateway solve DIFFERENT problems.

| Component       | Responsibility                                    |
| --------------- | ------------------------------------------------- |
| Load Balancer   | Which machine gets request?                       |
| API Gateway     | Is request valid? Which service should handle it? |
| TLS Termination | Secure communication                              |
| Reverse Proxy   | Hide backend infrastructure                       |

---

# 3. Simple Load Balancer Architecture

Suppose:

```text
Order Service
```

runs on:

```text
Server1
Server2
Server3
```

Without load balancer:

```text
Client
   │
   ▼
Server1
```

Server1 overloaded.

Servers2 and 3 idle.

---

# With Load Balancer

```text
Client
   │
   ▼

Load Balancer

   │
 ┌─┼─┐
 ▼ ▼ ▼

S1 S2 S3
```

Load Balancer distributes requests.

---

# 4. Why API Gateway Exists

As company grows:

Services become:

```text
User Service
Order Service
Payment Service
Inventory Service
Notification Service
```

Without API Gateway:

```text
Client
  ├── User Service
  ├── Order Service
  ├── Payment Service
  └── Inventory Service
```

Problems:

* Client knows internal architecture
* Authentication duplicated
* Rate limiting duplicated
* Security difficult

---

# Solution

```text
Client
   │
   ▼

API Gateway

   │
 ┌─┼─────────┐
 ▼ ▼         ▼

UserSvc OrderSvc PaymentSvc
```

Gateway becomes:

```text
Single Entry Point
```

---

# 5. Why Load Balancer Still Needed

Suppose:

```text
Gateway1
Gateway2
Gateway3
```

exist for high availability.

Need something to distribute traffic across gateways.

Therefore:

```text
Internet
   │
   ▼

Load Balancer

   │
 ┌─┼─────────┐
 ▼ ▼         ▼

GW1 GW2 GW3
```

---

# Real Production Architecture

```text
                Internet
                    │
                    ▼

          ┌─────────────────┐
          │ Load Balancer   │
          └────────┬────────┘
                   │

      ┌────────────┼────────────┐
      ▼            ▼            ▼

 Gateway1     Gateway2     Gateway3

      └────────────┼────────────┘
                   │

      ┌────────────┼────────────┐
      ▼            ▼            ▼

 UserSvc      OrderSvc     PaymentSvc
```

---

# 6. Where TLS Termination Happens

Most commonly:

```text
Client
   │ HTTPS
   ▼

Load Balancer
(TLS Termination)

   │ HTTP
   ▼

API Gateway

   │ HTTP
   ▼

Services
```

---

# What TLS Termination Means

Load Balancer performs:

* TLS Handshake
* Certificate Validation
* Session Key Setup
* Traffic Decryption

Internet traffic:

```text
Encrypted HTTPS
```

becomes:

```text
Decrypted Internal Traffic
```

---

# Why Do This?

Instead of every service handling:

* certificates
* key exchange
* encryption

Load Balancer handles it once.

Reduces:

* CPU usage
* operational complexity

---

# 7. Real Login Request Flow

Suppose user logs into Amazon.

Request:

```http
POST /login
```

---

# Step 1

Browser sends:

```text
HTTPS Request
```

to:

```text
amazon.com
```

---

# Step 2

DNS resolves domain.

Request reaches:

```text
External Load Balancer
```

---

# Step 3

Load Balancer performs:

```text
TLS Handshake
```

* verifies certificate
* establishes session key
* decrypts request

This is:

```text
TLS Termination
```

---

# Step 4

Load Balancer selects:

```text
Gateway2
```

using:

* Round Robin
* Least Connections
* Weighted Routing

---

# Step 5

Gateway Receives Request

Gateway performs:

### Authentication

Validate:

* JWT
* Session Token

---

### Rate Limiting

Example:

```text
100 requests/minute
```

---

### Routing

```text
/login
```

routes to:

```text
User Service
```

---

### Logging

Captures:

* request id
* latency
* traces

---

# Step 6

Gateway calls:

```text
User Service
```

---

# Step 7

User Service queries database.

```text
User Service
     │
     ▼
 Database
```

---

# Step 8

Response Returns

```text
Database
   │
   ▼
User Service
   │
   ▼
Gateway
   │
   ▼
Load Balancer
   │ HTTPS
   ▼
Client
```

Response follows reverse path.

---

# 8. Service-Level Load Balancing

Suppose:

```text
Order Service
```

has:

```text
Order1
Order2
Order3
Order4
```

Gateway should not directly choose instance.

Usually:

```text
Gateway
   │
   ▼

Service Load Balancer

   │
 ┌─┼─┬─┐
 ▼ ▼ ▼ ▼

O1 O2 O3 O4
```

---

# 9. Complete Production Architecture

```text
Client
   │
 HTTPS
   ▼

External Load Balancer
(TLS Termination)

   │
   ▼

API Gateway Cluster

   │
   ▼

Service Load Balancer

   │
 ┌─┼──────────────┐
 ▼ ▼              ▼

UserSvc      OrderSvc     PaymentSvc

   │
   ▼

Database
```

---

# 10. Which Type of Load Balancer Is Used?

This is a VERY common interview question.

---

# External Load Balancer

Usually:

```text
L7 Load Balancer
```

Examples:

* AWS ALB
* NGINX
* Envoy
* GCP HTTP Load Balancer

---

# Why L7?

Because it understands:

```http
/orders
/users
/payments
```

and can inspect:

* URLs
* Headers
* Cookies
* JWTs

---

# Example

```http
/orders/*
```

goes to:

```text
Order Gateway Cluster
```

while:

```http
/payments/*
```

goes to:

```text
Payment Gateway Cluster
```

Only L7 can do this.

---

# Why Not L4 Here?

L4 only sees:

```text
IP
Port
TCP
UDP
```

It cannot understand:

```http
/orders/123
```

or

```http
Authorization: Bearer ...
```

---

# 11. API Gateway Is Essentially L7

Examples:

* Kong
* Envoy
* APISIX
* NGINX
* Spring Cloud Gateway

Because gateway performs:

* Authentication
* Authorization
* Routing
* Rate Limiting
* Header Inspection

These are application-layer concerns.

---

# 12. Internal Service Load Balancing

Suppose:

```text
Order Service
```

runs on:

```text
Order1
Order2
Order3
Order4
```

---

# Common Approach

Use:

```text
L4 Load Balancer
```

Architecture:

```text
Gateway
   │
   ▼

L4 Load Balancer

   │
 ┌─┼─┬─┐
 ▼ ▼ ▼ ▼

O1 O2 O3 O4
```

---

# Why L4 Internally?

Gateway already decided:

```text
This request belongs to Order Service
```

Now only question is:

```text
Which Order Service instance?
```

No need for:

* URL inspection
* JWT validation
* Header parsing

L4 is faster.

---

# Benefits of L4 Internally

Because it only understands:

```text
IP
Port
TCP
UDP
```

it is:

* simpler
* faster
* lower latency
* higher throughput

---

# 13. Modern Kubernetes Architecture

Often there is NO dedicated internal load balancer.

Instead:

```text
Gateway
   │
   ▼

Kubernetes Service

   │
 ┌─┼──────────┐
 ▼ ▼          ▼

Pod1 Pod2 Pod3
```

Kubernetes Service effectively behaves like:

```text
L4 Load Balancer
```

---

# Common Service Discovery Systems

Examples:

* Kubernetes Services
* Consul
* Eureka
* Envoy Service Discovery

---

# 14. Where L4 Really Shines

Because L4 works only with:

```text
IP
Port
TCP
UDP
```

it is ideal for:

---

# Kafka

```text
Producer
   │
   ▼

L4 Load Balancer

   │
   ▼

Kafka Brokers
```

---

# Databases

```text
Application
    │
    ▼

L4 Load Balancer

    │
 ┌──┼──┐
 ▼  ▼  ▼

DB1 DB2 DB3
```

---

# Redis

Same idea.

TCP traffic only.

L4 perfect.

---

# 15. Netflix/Uber Style Architecture

Simplified:

```text
Client
   │
   ▼

L7 Edge Load Balancer

   │
   ▼

API Gateway / Edge Proxy

   │
   ▼

Service Mesh (Envoy)

   │
   ▼

Microservices
```

---

Inside service mesh:

```text
Envoy → Envoy
```

communication often behaves closer to:

```text
L4 + Service Discovery
```

than traditional L7 load balancing.

---

# 16. Final Mental Model

A production microservices request typically flows like:

```text
Client
   ↓
L7 Load Balancer
   ↓
API Gateway / Reverse Proxy
   ↓
L4 Load Balancer / Service Discovery
   ↓
Microservice
   ↓
Cache / Kafka / Database
```

---

# What Each Layer Solves

| Layer             | Responsibility                                        |
| ----------------- | ----------------------------------------------------- |
| L7 Load Balancer  | TLS termination, routing, edge traffic management     |
| API Gateway       | Authentication, authorization, routing, rate limiting |
| L4 Load Balancer  | Fast internal traffic distribution                    |
| Service Discovery | Find healthy service instances                        |
| Microservice      | Business logic                                        |
| Cache/DB/Kafka    | Data and messaging                                    |

---

# 17. Senior-Level Interview Insight

A modern production architecture typically uses:

```text
Internet
   ↓
L7 Load Balancer
   ↓
API Gateway
   ↓
L4 Load Balancer or Service Discovery
   ↓
Microservices
```

Reason:

* L7 is used at the edge because application-level decisions are required.
* API Gateway centralizes authentication, authorization, routing and rate limiting.
* L4 is preferred internally because routing decisions have already been made and maximum throughput is desired.
* Modern cloud-native systems often replace internal L4 load balancers with service discovery mechanisms such as Kubernetes Services, Consul, Envoy, or Eureka.

# What If the External Load Balancer Becomes a Bottleneck?

# 1. The Problem

Consider the architecture:

```text
Client
   │
   ▼

External Load Balancer

   │
   ▼

API Gateway Cluster
```

A natural question is:

> What if the External Load Balancer itself becomes overloaded or crashes?

This is an important distributed systems question because every layer that exists as a single machine can become:

* a bottleneck
* a single point of failure

---

# 2. Single Load Balancer Problem

Suppose:

```text
100 Million Users
        │
        ▼

External Load Balancer

        │
        ▼

API Gateway Cluster
```

---

## Failure Scenario

If the load balancer crashes:

```text
Application = Down
```

This is called:

```text
Single Point of Failure (SPOF)
```

---

## Capacity Scenario

Suppose load balancer capacity:

```text
100K Requests/sec
```

Traffic grows to:

```text
500K Requests/sec
```

Load balancer becomes bottleneck.

Even if:

* API Gateway scales
* Services scale
* Databases scale

traffic still cannot reach them.

---

# 3. Real Production Solution

The solution is:

> Load balancers themselves are replicated and load balanced.

---

# Architecture

```text
                DNS
                 │
                 ▼

      ┌────────────────────┐
      │  api.company.com   │
      └────────────────────┘

           │          │
           ▼          ▼

         LB1        LB2

           │          │
           └────┬─────┘
                ▼

       API Gateway Cluster
```

---

# Benefits

If:

```text
LB1 dies
```

traffic automatically moves to:

```text
LB2
```

Application remains available.

---

# 4. DNS Load Balancing

DNS often performs the first level of traffic distribution.

---

# Example

Suppose:

```text
api.netflix.com
```

resolves to:

```text
LB1 IP
LB2 IP
LB3 IP
```

DNS returns multiple addresses.

Clients naturally spread across them.

---

# Example Flow

```text
Client A → LB1
Client B → LB2
Client C → LB3
```

This technique is called:

```text
DNS Load Balancing
```

---

# Why Useful?

Provides:

* high availability
* traffic distribution
* geographic routing

before requests even reach a load balancer.

---

# 5. Cloud Providers Hide This Complexity

When using AWS:

```text
Application Load Balancer (ALB)
```

appears to be a single load balancer.

Reality is very different.

---

# Actual Mental Model

Internally AWS runs:

```text
LB Node 1
LB Node 2
LB Node 3
LB Node 4
LB Node 5
...
```

distributed across:

* Availability Zones
* Data Centers

AWS automatically:

* scales nodes
* replaces failed nodes
* distributes traffic

You never manage this directly.

---

# Important Interview Insight

When cloud providers show:

```text
One Load Balancer
```

internally it is actually:

```text
Load Balancer Fleet
```

---

# 6. Netflix-Style Architecture

Simplified:

```text
Users
   │
   ▼

Global DNS

   │
   ▼

Regional Load Balancers

   │
   ▼

Gateway Cluster

   │
   ▼

Services
```

Notice:

```text
DNS
 ↓
Regional LB
 ↓
Gateway
```

Multiple layers provide redundancy.

---

# 7. Multi-Region Load Balancing

Suppose application runs in:

* US-East
* Europe
* Asia

Architecture:

```text
                     DNS
                      │

        ┌─────────────┼─────────────┐
        ▼             ▼             ▼

     US LB       Europe LB      Asia LB
```

---

# Benefits

Users routed to nearest region.

Provides:

* lower latency
* fault tolerance
* regional failover
* higher throughput

---

# Example

Indian user:

```text
India → Asia LB
```

US user:

```text
USA → US LB
```

This reduces network latency significantly.

---

# 8. Traffic Spike Handling

Suppose:

* IPL Final
* World Cup Final
* Black Friday

Traffic suddenly becomes:

```text
10x normal traffic
```

Cloud providers automatically scale:

```text
5 LB Nodes
    ↓
50 LB Nodes
```

This is often invisible to application teams.

---

# 9. Can API Gateway Become Bottleneck?

Absolutely.

Just like load balancers,
API Gateways must also be replicated.

---

# Architecture

```text
            LB Cluster
                 │

      ┌──────────┼──────────┐
      ▼          ▼          ▼

    GW1        GW2        GW3
```

---

# Why?

Because gateway performs:

* authentication
* authorization
* routing
* rate limiting
* logging

All consume CPU and memory.

---

# Solution

Scale gateways horizontally:

```text
Gateway1
Gateway2
Gateway3
Gateway4
Gateway5
```

---

# 10. Can DNS Become Bottleneck?

Usually no.

Reasons:

DNS infrastructure is:

* globally distributed
* highly cached
* massively replicated

Providers such as:

* Cloudflare
* Route53
* Akamai

handle enormous traffic volumes.

---

# 11. Real Production Architecture

A more realistic architecture looks like:

```text
Client
   │
   ▼

Global DNS

   │
   ▼

Load Balancer Fleet
(L7)

   │
   ▼

API Gateway Fleet

   │
   ▼

Service Discovery

   │
   ▼

Microservice Fleet

   │
   ▼

Database / Kafka / Cache
```

---

# Key Observation

There is no single machine anywhere.

Every critical layer is replicated.

---

# 12. Distributed Systems Design Principle

Never think:

```text
Load Balancer
Gateway
Database
Cache
Kafka
```

Think:

```text
Load Balancer Fleet
Gateway Fleet
Database Cluster
Cache Cluster
Kafka Cluster
```

Everything scales horizontally.

---

# 13. Most Important Interview Insight

Any component that exists as:

```text
Single Machine
```

eventually becomes either:

* bottleneck
* single point of failure

Therefore production systems replicate and scale every critical layer.

---

# 14. Final Mental Model

A realistic large-scale architecture looks like:

```text
Client
   │
   ▼

Global DNS

   │
   ▼

L7 Load Balancer Fleet

   │
   ▼

API Gateway Fleet

   │
   ▼

Service Discovery / Internal L4 Load Balancing

   │
   ▼

Microservice Fleet

   │
   ▼

Cache / Kafka Cluster / Database Cluster
```

Every layer is:

* horizontally scalable
* replicated
* fault tolerant

This is how modern internet-scale systems are built.
