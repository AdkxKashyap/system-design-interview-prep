# API Gateway + Reverse Proxy — System Design Interview Notes

# 1. Introduction

API Gateway and Reverse Proxy are among the MOST important concepts in modern backend and microservices architecture.

Almost every large-scale system uses:

* API Gateways
* Reverse Proxies
* Edge Proxies
* Load Balancers

Examples:

* NGINX
* Envoy
* HAProxy
* Kong
* Apache APISIX

Interviewers ask these topics because they test whether you understand:

* how production traffic enters systems
* microservices architecture
* authentication flow
* traffic management
* distributed systems operations
* cross-cutting concerns

---

# 2. First Understand the Problem

Suppose you built an e-commerce system using microservices.

Services:

* User Service
* Order Service
* Payment Service
* Inventory Service
* Notification Service

Naive architecture:

```text id="api1"
Mobile App
   ├── User Service
   ├── Order Service
   ├── Payment Service
   ├── Inventory Service
   └── Notification Service
```

Client directly communicates with all services.

This creates MANY problems.

---

# 3. Problems with Direct Client → Service Communication

# A. Authentication Logic Everywhere

Every service must implement:

* JWT validation
* authentication
* permission checks

Code duplication everywhere.

---

# B. Rate Limiting Everywhere

Every service independently handles:

* abuse prevention
* DDoS protection
* request quotas

Operational nightmare.

---

# C. Client Needs Internal Architecture Knowledge

Mobile app must know:

* service locations
* endpoints
* versions
* infrastructure topology

Very poor design.

---

# D. Security Problems

Internal services exposed publicly.

Huge attack surface.

---

# E. Cross-Cutting Concerns Duplicated

Things like:

* logging
* tracing
* SSL/TLS
* authentication
* monitoring

repeated across all services.

---

# 4. Solution — API Gateway

Instead of:

```text id="api2"
Client → Multiple Services
```

System becomes:

```text id="api3"
Client → API Gateway → Internal Services
```

Gateway becomes:

```text id="api4"
Single Entry Point
```

for backend system.

---

# 5. Real Production Architecture

```text id="api5"
                 Clients
         (Mobile/Web/Frontend)
                       │
                       ▼
               ┌──────────────┐
               │ API Gateway  │
               └──────┬───────┘
                      │
      ┌───────────────┼───────────────┐
      ▼               ▼               ▼

 User Service   Order Service   Payment Service
```

This is extremely common in production systems.

---

# 6. What is API Gateway REALLY?

API Gateway is fundamentally:

> centralized traffic management layer for backend services.

Think of it as:

```text id="api6"
Front Door of Backend System
```

Very important mental model.

---

# 7. Real World Analogy

Think airport security.

Passengers do NOT directly walk onto planes.

They first go through:

* security checks
* verification
* routing
* validation

API Gateway behaves similarly:

* authenticates requests
* routes traffic
* applies policies
* protects services

---

# 8. Reverse Proxy vs Forward Proxy

Very important interview topic.

---

# Forward Proxy

Acts on behalf of:

```text id="api7"
CLIENT
```

Example:

* corporate proxy
* VPN proxy

Client says:

```text id="api8"
"Fetch internet for me"
```

---

# Reverse Proxy

Acts on behalf of:

```text id="api9"
SERVER
```

Client thinks:

```text id="api10"
talking directly to backend
```

Actually communicating with proxy.

Reverse proxy hides backend infrastructure.

---

# Important Understanding

API Gateway is usually:

```text id="api11"
Specialized Reverse Proxy
```

---

# 9. Routing (Most Important Gateway Feature)

Suppose request arrives:

```http id="api12"
/orders/123
```

Gateway routes request to:

```text id="api13"
Order Service
```

---

# Another Example

```http id="api14"
/payments/pay
```

Gateway routes request to:

```text id="api15"
Payment Service
```

Gateway acts like:

```text id="api16"
Traffic Controller
```

---

# Why Routing Important?

Clients no longer need to know:

* service locations
* internal networking
* infrastructure changes

Backend architecture hidden.

Very important microservices principle.

---

# 10. Authentication

At scale:

> authentication should NOT be duplicated across services.

Gateway centralizes authentication.

---

# Real Authentication Flow

Suppose client sends:

```http id="api17"
Authorization: Bearer JWT_TOKEN
```

Gateway:

1. validates JWT
2. checks expiration
3. verifies signature
4. extracts user identity

Then forwards trusted request internally.

---

# Why This Powerful

Without gateway:
EVERY service implements:

* JWT parsing
* token validation
* auth middleware

Massive duplication.

Gateway centralizes everything.

---

# Real Production Example

Companies like:

* Netflix
* Uber
* Airbnb

commonly handle:

* authentication
* TLS termination
* traffic shaping
* observability

at edge gateway layer.

---

# 11. Important Production Insight

Internal services often TRUST gateway.

Meaning:

* services assume request already authenticated
* internal logic simplified

Reduces service complexity significantly.

---

# 12. Authentication vs Authorization

Very common interview question.

---

# Authentication

Means:

```text id="api18"
Who are you?
```

Examples:

* login validation
* JWT verification

---

# Authorization

Means:

```text id="api19"
What are you allowed to do?
```

Examples:

* admin permissions
* role access
* resource permissions

Gateway may handle BOTH.

---

# 13. Rate Limiting

One of MOST important gateway responsibilities.

Suppose malicious user sends:

```text id="api20"
1 million requests/sec
```

Without protection:
backend collapses.

Gateway enforces:

```text id="api21"
Rate Limits
```

---

# Example

```text id="api22"
100 requests/minute/user
```

Beyond limit:

```http id="api23"
429 Too Many Requests
```

returned.

---

# Real Production Examples

Public APIs like:

* Stripe
* GitHub
* Twitter

heavily use rate limiting.

Without rate limiting:

* abuse
* scraping
* DDoS attacks
* infrastructure overload

become huge problems.

---

# 14. Common Rate Limiting Algorithms

Important interview topic.

---

# Token Bucket

Bucket contains tokens.

Each request consumes token.

Tokens refill over time.

Allows:

* burst traffic
* controlled sustained rate

Most common production algorithm.

---

# Example

```text id="api24"
100 tokens capacity
10 tokens/sec refill
```

User may burst initially,
but long-term abuse blocked.

---

# Leaky Bucket

Requests processed at fixed constant rate.

Smooths traffic spikes.

Useful for:

* traffic shaping
* networking control

---

# Sliding Window

Tracks requests within moving time window.

More accurate than fixed window counters.

Common in API gateways.

---

# 15. SSL/TLS Termination

Extremely important production concept.

---

# First Understand Problem

HTTPS encryption expensive.

If EVERY backend service handles:

* TLS handshake
* encryption/decryption

CPU overhead huge.

---

# Solution — SSL Termination at Gateway

Client communicates securely:

```text id="api25"
HTTPS
```

Gateway decrypts traffic.

Internal services may communicate using:

```text id="api26"
HTTP
```

inside trusted internal network.

---

# Architecture

```text id="api27"
Client
   │ HTTPS
   ▼
API Gateway
   │ HTTP
   ▼
Internal Services
```

---

# Why This Powerful

Gateway centralizes:

* certificates
* TLS handling
* encryption management

Backend services simplified.

---

# Real Production Example

NGINX and Envoy commonly handle:

* TLS termination
* certificate rotation
* HTTPS enforcement

at edge layer.

---

# 16. Why TLS Handshake Expensive

TLS handshake involves:

* asymmetric cryptography
* certificate validation
* key exchange

CPU intensive operation.

Offloading TLS to gateway improves scalability.

---

# 17. API Gateway vs Load Balancer

Very common interview question.

---

# Load Balancer

Primarily distributes traffic.

Main concern:

```text id="api28"
Which server should receive request?
```

Mostly transport/network focused.

---

# API Gateway

Much richer functionality:

* authentication
* authorization
* routing
* rate limiting
* observability
* API policies
* transformations

Gateway operates at:

```text id="api29"
Application/API Layer
```

---

# 18. Real Production Request Flow

# Step 1 — Client Sends Request

```http id="api30"
POST /orders
Authorization: Bearer JWT
```

---

# Step 2 — Gateway Receives Request

Gateway:

* terminates TLS
* validates JWT
* applies rate limiting
* logs request
* traces request

---

# Step 3 — Gateway Routes Request

Routes:

```text id="api31"
/orders → Order Service
```

---

# Step 4 — Internal Service Processes Request

Order Service:

* creates order
* stores data in DB
* publishes Kafka event

---

# Step 5 — Response Returns

Gateway sends response back to client.

---

# Full Architecture

```text id="api32"
Client
   │
 HTTPS
   │
   ▼
┌──────────────┐
│ API Gateway  │
│--------------│
│ Auth         │
│ Rate Limit   │
│ Routing      │
│ TLS          │
│ Logging      │
└──────┬───────┘
       │
 ┌─────┼─────┐
 ▼     ▼     ▼

User  Order  Payment
Svc    Svc     Svc
```

This is VERY close to real production architecture.

---

# 19. Gateway in Kubernetes / Cloud Native Systems

Modern systems commonly use:

* Envoy
* Istio
* Kong
* NGINX Ingress

for:

* ingress traffic
* service mesh
* observability
* security

Very common cloud-native pattern.

---

# 20. Important Tradeoff — Gateway Bottleneck

Gateway centralizes traffic.

Potential issue:

```text id="api33"
Single Point of Failure
```

Production systems therefore:

* scale gateways horizontally
* replicate gateways
* place gateways behind load balancers

Very important operational insight.

---

# 21. Another Important Tradeoff

Too much gateway logic dangerous.

If gateway handles:

* business workflows
* orchestration
* domain logic

it becomes:

```text id="api34"
Monolith Bottleneck
```

Gateway should mainly handle:

* cross-cutting concerns
* traffic management

NOT core business logic.

---

# 22. Most Important Interview Insight

API Gateway fundamentally exists to:

> centralize cross-cutting concerns and simplify client interaction with distributed microservices systems.

---

# 23. Final Mental Models

# Reverse Proxy

```text id="api35"
Proxy sitting in front of servers
```

Hides backend infrastructure.

---

# API Gateway

```text id="api36"
Smart reverse proxy for APIs/microservices
```

Handles:

* authentication
* authorization
* routing
* rate limiting
* TLS termination
* observability

---

# Authentication

```text id="api37"
Who are you?
```

---

# Authorization

```text id="api38"
What are you allowed to do?
```

---

# Rate Limiting

```text id="api39"
Protect backend from abuse
```

---

# SSL/TLS Termination

```text id="api40"
Gateway handles HTTPS/TLS
```

Backend simplified.

---

# Routing

```text id="api41"
Forward request to correct service
```

---

# 24. Final Revision Summary

| Concept         | Purpose                            |
| --------------- | ---------------------------------- |
| Reverse Proxy   | Hide backend servers               |
| API Gateway     | Centralized API traffic management |
| Authentication  | Verify identity                    |
| Authorization   | Verify permissions                 |
| Routing         | Send request to correct service    |
| Rate Limiting   | Prevent abuse                      |
| SSL Termination | Offload TLS handling               |
| Observability   | Logging/Tracing/Metrics            |

---

# 25. Senior-Level Insight

API gateways fundamentally trade:

> centralized control and operational simplicity for additional network hops and infrastructure complexity in distributed microservices architectures.
