# Service Discovery

# The Problem Service Discovery Solves

Let's say we have:

```text
Order Service
Payment Service
Inventory Service
Email Service
```

Initially:

```text
Payment Service

IP = 10.0.1.5
```

Order Service calls:

```text
http://10.0.1.5/pay
```

Everything works.

---

Now Payment Service crashes.

Kubernetes launches a new instance:

```text
Old IP = 10.0.1.5
New IP = 10.0.3.17
```

Suddenly:

```text
Order Service
     ↓
10.0.1.5
```

fails.

Because IP changed.

---

In modern cloud systems:

```text
Containers
Pods
VMs
```

come and go constantly.

IPs are not stable.

Hardcoding addresses becomes impossible.

---

# What Is Service Discovery?

Service discovery is simply:

> A mechanism that allows services to find other services dynamically.

Instead of:

```text
Call 10.0.1.5
```

we do:

```text
Call Payment Service
```

and some system resolves:

```text
Payment Service
       ↓
10.0.3.17
10.0.4.21
10.0.7.12
```

---

# Real World Analogy

Think of your phone contacts.

You don't remember:

```text
+91 9999999999
```

You remember:

```text
Mom
```

Phone resolves:

```text
Mom
   ↓
Phone Number
```

Service discovery works similarly.

---

# Core Components

A service discovery system usually has:

### Service Registry

Stores:

```text
Service Name
     ↓
Instances
```

Example:

```text
Payment Service
    ↓
10.0.1.5
10.0.1.8
10.0.1.10
```

---

### Service Provider

Registers itself.

Example:

```text
Payment Service
```

says:

```text
I am alive
My address is 10.0.1.5
```

---

### Service Consumer

Queries registry.

Example:

```text
Order Service
```

asks:

```text
Where is Payment Service?
```

---

# Basic Flow

```text
Payment Service Starts

       │

       ▼

Register With Registry

       │

       ▼

Registry Stores Address

       │

       ▼

Order Service Requests Address

       │

       ▼

Registry Returns Instances

       │

       ▼

Order Service Calls Payment Service
```

---

# Example: Eureka

Created by Netflix during the early microservices era.

---

# Architecture

```text
           Eureka

              ▲
              │

   ┌──────────┼──────────┐
   ▼          ▼          ▼

Order     Payment     Inventory
```

---

# Registration

Payment Service starts.

Registers:

```text
Service = payment-service
IP = 10.0.1.5
```

with Eureka.

---

Inventory Service registers:

```text
Service = inventory-service
IP = 10.0.1.8
```

---

Registry stores:

```text
payment-service
    ↓
10.0.1.5

inventory-service
    ↓
10.0.1.8
```

---

# Service Lookup

Order Service wants payment.

Queries Eureka:

```text
Where is payment-service?
```

---

Eureka responds:

```text
10.0.1.5
```

---

Order Service calls:

```text
10.0.1.5
```

---

# Heartbeats

How does Eureka know service is alive?

Every few seconds:

```text
Payment Service
     ↓
Heartbeat
```

---

If heartbeat stops:

```text
Service Removed
```

from registry.

---

# Why Eureka Was Popular

Netflix had:

```text
Thousands of Services
```

running on cloud VMs.

IPs changed constantly.

Eureka solved:

```text
Dynamic Discovery
```

---

# Kubernetes Service Discovery

This is what most modern companies use.

---

# Problem

Suppose deployment:

```text
Payment Service

Pod1
Pod2
Pod3
```

---

Pods get random IPs:

```text
10.0.2.5
10.0.4.7
10.0.8.2
```

These change constantly.

---

How can Order Service find them?

---

# Kubernetes Service

Kubernetes introduces:

```text
Service
```

resource.

---

Example:

```yaml
name: payment-service
```

---

Kubernetes creates stable DNS:

```text
payment-service.default.svc.cluster.local
```

---

Now Order Service calls:

```text
http://payment-service
```

instead of IP.

---

Kubernetes resolves:

```text
payment-service
       ↓
Pod1
Pod2
Pod3
```

---

# Architecture

```text
Order Service

       │

       ▼

payment-service

       │

       ▼

Kubernetes Service

       │

       ▼

Pod1
Pod2
Pod3
```

---

# Service Discovery + Load Balancing

Important interview concept.

Suppose:

```text
Payment Pod1
Payment Pod2
Payment Pod3
```

---

Service discovery returns:

```text
Available Instances
```

---

Load balancing decides:

```text
Which Instance?
```

Example:

```text
Request1 → Pod1
Request2 → Pod2
Request3 → Pod3
```

---

These concepts work together.

---

# Client-Side Discovery

Used heavily in Netflix architecture.

Flow:

```text
Order Service
      │
      ▼

Eureka Lookup

      │
      ▼

Choose Instance

      │
      ▼

Call Service
```

Client chooses destination.

---

Example tools:

* Eureka
* Ribbon

---

# Server-Side Discovery

More common today.

Flow:

```text
Order Service

      │

      ▼

Load Balancer

      │

      ▼

Payment Pods
```

Load balancer performs discovery.

---

Examples:

* Kubernetes Service
* Envoy
* Istio

---

# Real World Example: Uber

Imagine:

```text
Ride Service
Payment Service
Driver Service
Notification Service
```

Thousands of instances.

---

Drivers go online/offline.

Containers restart.

Autoscaling happens constantly.

---

Service discovery allows:

```text
Ride Service
      ↓
Find Driver Service
```

without knowing actual IPs.

---

# Real World Example: Netflix

When a movie starts:

```text
API Gateway
      ↓
Recommendation Service
      ↓
User Service
      ↓
Playback Service
```

Each service may have:

```text
100+
Instances
```

running.

Service discovery tells:

```text
Where are those instances?
```

---

# Service Discovery vs DNS

Traditional DNS:

```text
google.com
```

↓

```text
IP Address
```

---

Service discovery is essentially:

```text
Internal DNS For Microservices
```

but with:

* Dynamic registration
* Health checks
* Load balancing
* Instance awareness

---

# What Interviewers Expect

You should understand:

### Problem

```text
IPs change constantly
```

---

### Solution

```text
Service Registry
```

---

### Workflow

```text
Register
Discover
Call
```

---

### Popular Systems

* Kubernetes Service
* Eureka
* Consul
* etcd

---

# Interview Answer Template

If asked:

> How do microservices find each other?

Strong answer:

> In a microservices environment, service instances are dynamic and their IP addresses change frequently. A service discovery system maintains a registry mapping service names to healthy instances. Services register themselves on startup and send heartbeats. Consumers query the registry or use DNS-based discovery to find available instances. Examples include Eureka in Netflix OSS and Kubernetes Services with built-in DNS-based discovery.

---

# Senior Engineer Mental Model

Think:

```text
Microservices
      ↓
IPs Change Constantly
      ↓
Need Stable Names
      ↓
Service Discovery
      ↓
Registry Maps
Name → Instances
      ↓
Consumers Find Services Dynamically
```

The most important thing to remember:

> Service Discovery solves "Where is the service?" while Load Balancing solves "Which instance should receive the request?" They are different problems that usually work together.

---

# Service Discovery vs Load Balancing

# A Common Confusion

Many engineers think:

```text
Service Discovery = Load Balancer
```

This is not entirely correct.

They solve different problems.

---

# The Core Difference

## Service Discovery

Answers:

```text
Where is the service?
```

---

## Load Balancer

Answers:

```text
Which instance should handle this request?
```

---

# Example

Suppose Payment Service has:

```text
Pod1 → 10.0.1.5
Pod2 → 10.0.1.6
Pod3 → 10.0.1.7
```

Order Service wants to call:

```text
payment-service
```

---

# Service Discovery's Job

Returns:

```text
Payment Service
       ↓
10.0.1.5
10.0.1.6
10.0.1.7
```

That's all.

It discovered the service instances.

---

# Load Balancer's Job

Now choose:

```text
Request 1 → Pod1
Request 2 → Pod2
Request 3 → Pod3
```

using:

* Round Robin
* Least Connections
* Weighted Routing

---

# Restaurant Analogy

Service Discovery answers:

```text
Which restaurants are available?
```

Example:

```text
Dominos A
Dominos B
Dominos C
```

---

Load Balancer answers:

```text
Which restaurant should receive this order?
```

Different responsibilities.

---

# Why They Look Like The Same Thing

Because modern systems often combine them.

---

# Kubernetes Example

Suppose:

```text
Payment Deployment

Pod1
Pod2
Pod3
```

---

Kubernetes Service knows:

```text
payment-service
```

maps to:

```text
Pod1
Pod2
Pod3
```

This is:

```text
Service Discovery
```

---

Then Kubernetes Service forwards:

```text
Request1 → Pod1
Request2 → Pod2
Request3 → Pod3
```

This is:

```text
Load Balancing
```

---

So Kubernetes Service does BOTH.

---

# Kubernetes Architecture

```text
Order Service
      │
      ▼

payment-service

      │
      ▼

K8s Service

      ├── Pod1
      ├── Pod2
      └── Pod3
```

---

# Eureka Example

This makes the distinction much clearer.

---

Eureka stores:

```text
payment-service

10.0.1.5
10.0.1.6
10.0.1.7
```

---

Order Service asks:

```text
Where is payment-service?
```

---

Eureka returns:

```text
10.0.1.5
10.0.1.6
10.0.1.7
```

---

Now Order Service itself chooses:

```text
10.0.1.6
```

using:

* Ribbon
* Round Robin
* Random
* Custom Logic

---

Here:

```text
Eureka = Discovery
Ribbon = Load Balancing
```

Completely separate.

---

# Client-Side Discovery

Used heavily in Netflix architecture.

---

Flow

```text
Order Service
      │
      ▼

Eureka

      │
Returns All Instances

      ▼

Order Service Chooses Instance
```

---

Service discovery and load balancing are separate responsibilities.

---

# Server-Side Discovery

More common today.

---

Flow

```text
Order Service
      │
      ▼

K8s Service

      │
Discovers + Balances

      ▼

Pod1
Pod2
Pod3
```

---

One component handles both.

---

# Mental Model For Interviews

Always think:

### Step 1

```text
Where are the instances?
```

↓

```text
Service Discovery
```

---

### Step 2

```text
Which instance gets this request?
```

↓

```text
Load Balancing
```

---

# Real Production Examples

| System             | Service Discovery  | Load Balancing     |
| ------------------ | ------------------ | ------------------ |
| Eureka + Ribbon    | Eureka             | Ribbon             |
| Kubernetes Service | Kubernetes Service | Kubernetes Service |
| Consul + Envoy     | Consul             | Envoy              |
| DNS + NGINX        | DNS                | NGINX              |
| Istio              | Istio Registry     | Envoy              |

---

# Interview Answer

If asked:

> Does service discovery act as a load balancer?

A strong answer:

> Conceptually they solve different problems. Service discovery answers "where are the available service instances?", while load balancing decides "which instance should receive the request?". In some platforms such as Kubernetes Services, both capabilities are bundled together. In older Netflix-style architectures, Eureka handled discovery and Ribbon handled load balancing separately.

---

# Senior Engineer Mental Model

Think:

```text
Service Discovery
        ↓
Where Is The Service?
```

and

```text
Load Balancing
        ↓
Which Instance Gets The Request?
```

They are different problems.

Modern platforms like Kubernetes often combine both into a single component, which is why they are frequently confused.
