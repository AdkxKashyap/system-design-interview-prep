# URL Shortener - System Design Notes

# Concepts & Patterns Learned

This is the "Two Sum" of System Design.

The following concepts appear repeatedly in Twitter, Instagram, Uber, YouTube, Dropbox, Search and Notification systems.

## Concepts

* Capacity Estimation
* API Design
* Database Design
* Read Heavy vs Write Heavy Systems
* Base62 Encoding
* ID Generation
* Cache Aside Pattern
* Redis Caching
* Database Scaling
* Sharding
* Replication
* High Availability

## Patterns

### Cache Aside

```text
Request
   |
Cache
 /   \
Hit  Miss
 |     |
Return DB
```

Used in:

* URL Shortener
* Twitter
* Instagram
* YouTube
* Uber

---

### Read Heavy System

```text
Reads >> Writes
```

Example:

```text
1K Writes/sec
1M Reads/sec
```

Optimization focus:

* Cache
* Read scalability

---

# Phase 1 - Requirements Gathering

Never start drawing architecture immediately.

Clarify requirements first.

## Functional Requirements

### Mandatory

#### Create Short URL

Input:

```http
POST /shorten
```

```json
{
  "url":"https://google.com"
}
```

Output:

```json
{
  "shortUrl":"tiny.com/A9X2P"
}
```

---

#### Redirect

Input:

```http
GET /A9X2P
```

Output:

```text
302 Redirect
```

Redirects to:

```text
https://google.com
```

---

### Optional Features

* Custom aliases
* Expiration
* Analytics

---

## Non Functional Requirements

### Availability > Consistency

If a replica is stale for a few seconds:

```text
Acceptable
```

If service is unavailable:

```text
Not acceptable
```

Therefore:

```text
Availability > Consistency
```

---

### Low Latency

Redirects should be fast.

Target:

```text
< 50 ms
```

---

### Durability

URL mappings should never be lost.

---

# Phase 2 - Capacity Estimation

## Assumptions

### URL Creation

```text
100M URLs/day
```

```text
100M = 10^8
```

---

### Writes Per Second

```text
10^8 / 10^5

= 10^3

= 1K writes/sec
```

---

### Redirects

Assume:

```text
100 redirects per URL
```

---

Daily Reads:

```text
100M × 100

= 100B
```

```text
100B = 10^11
```

---

Reads Per Second

```text
10^11 / 10^5

= 10^6

= 1M reads/sec
```

---

## Key Observation

```text
Reads >> Writes
```

This is a read-heavy system.

Most design decisions come from this observation.

---
## Total URLs required
```text
    100M url/day
    Suppose we need service for 10 yrs
    100M * 365 * 10 = 365B URLs.
    Our system should support 365B unique url creation
```
---

# Phase 3 - Data Model

## URL Mapping Table

| Field      | Type      |
| ---------- | --------- |
| short_code | varchar   |
| long_url   | text      |
| created_at | timestamp |

---

Example

| short_code | long_url    |
| ---------- | ----------- |
| A9X2P      | google.com  |
| G7X9K      | youtube.com |

---

## Primary Key

```text
short_code
```

Reason:

```text
short_code -> long_url
```

is the primary lookup pattern.

---

## Duplicate Long URLs

Current design allows:

```text
google.com
```

to generate:

```text
tiny.com/abc
tiny.com/xyz
```

Storage is cheap.

Write simplicity is preferred.

---

# Phase 4 - High Level Design

```text
               Client
                  |
                  v
            Load Balancer
                  |
                  v
             URL Service
             /         \
            /           \
           v             v

       Redis Cache    Database
```

---

## Components

### Load Balancer

Distributes traffic across servers.

---

### URL Service

Handles:

* URL creation
* Redirect requests

---

### Redis

Stores:

```text
short_code -> long_url
```

Provides low latency lookups.

---

### Database

Stores permanent mappings.

---

# Phase 5 - URL Generation

The most important design decision.

---

# Base62 Encoding

Instead of exposing:

```text
1000000
```

we convert it to:

```text
4C92
```

using Base62.

---

## Character Set

```text
a-z
A-Z
0-9
```

Total:

```text
26 + 26 + 10

= 62
```

---

## Why Base62?

### URL Friendly

Avoids characters:

```text
+
/
=
```

which appear in Base64.

---

### Compact

```text
62^6

≈ 56 Billion
```

Only 6 characters can represent 56 billion values.
Max 7 chars needed for 365B urls.
---

## Example

```text
ID = 1000000
```

Base62:

```text
4C92
```

Result:

```text
tiny.com/4C92
```

---

# ID Generation

The real challenge.

Need:

```text
Unique IDs
```

across multiple servers.

---

# Approach 1 - Database Auto Increment

Database:

```text
1
2
3
4
5
```

Each request:

```text
Server
   |
   v
Database
```

gets next ID.

---

## Problem

At scale:

```text
100K requests/sec
```

means:

```text
100K DB calls/sec
```

Single DB becomes bottleneck.

---

# Approach 2 - Range Allocation

Most common interview solution.

---

## Database State

Store only:

```text
next_id
```

Example:

```text
next_id = 1
```

---

## Server Requests Range

Server A:

```text
Give me 10000 IDs
```

Database allocates:

```text
1 - 10000
```

Updates:

```text
next_id = 10001
```

---

Server B:

```text
10001 - 20000
```

---

Server C:

```text
20001 - 30000
```

---

Visual

```text
              Database

             next_id
                |
    ---------------------------
    |            |            |
    v            v            v

Server A     Server B     Server C

1-10000    10001-20000  20001-30000
```

---

## What Is Stored In Server?

Only:

```java
currentId
maxId
```

Example:

```java
currentId = 1
maxId = 10000
```

---

Request Flow

```java
id = currentId++;
```

No DB call.

---

After:

```text
currentId > maxId
```

Server requests another range.

---

## Why Is This Better?

Without range allocation:

```text
100K URL requests

=
100K DB calls
```

---

With range allocation:

```text
10000 URL requests

=
1 DB call
```

Reduction:

```text
10000x fewer DB calls
```

---

## Server Crash Scenario

Suppose:

```text
Range

1 - 10000
```

---

Server uses:

```text
1 - 5000
```

and crashes.

---

Unused:

```text
5001 - 10000
```

are lost.

---

Result:

```text
ID gaps exist
```

Example:

```text
1
2
3
...
5000

10001
10002
```

This is acceptable.

Requirement is:

```text
Unique IDs
```

not

```text
Continuous IDs
```

---

# URL Creation Flow

```text
Client
   |
   v

URL Service
   |
   v

Get ID
   |
   v

Base62 Encode
   |
   v

Store Mapping
   |
   v

Database
```

---

Example

Input:

```text
google.com
```

---

Generated ID:

```text
10001
```

---

Base62:

```text
A9X2P
```

---

Store:

```text
A9X2P -> google.com
```

---

Return:

```text
tiny.com/A9X2P
```

---

# Redirect Flow

Most critical path.

```text
Browser
   |
   v

URL Service
   |
   v

Redis Cache
```

---

Cache Hit

```text
A9X2P -> google.com
```

Return immediately.

---

Cache Miss

```text
Redis
   |
   v
Database
   |
   v
Update Cache
   |
   v
Return URL
```

---

This is the Cache Aside Pattern.

---

# Interview Summary

For a URL Shortener:

* Read-heavy system
* Base62 for compact URLs
* Range Allocation for scalable ID generation
* Redis for low latency reads
* Database for durable storage
* Cache Aside pattern for redirects

Core interview takeaway:

```text
Reads dominate writes.

Optimize redirects using caching.

Generate unique IDs efficiently without creating a database bottleneck.
Refer: https://bytebytego.com/courses/system-design-interview/design-a-url-shortener
```
