# URL Shortener - Scaling & Global Architecture Notes

This document covers the scaling phase that follows the basic URL Shortener design.

Assumes the following already exist:

* URL Service
* Base62 Encoding
* ID Generation
* Redis Cache
* Database
* Cache Aside Pattern

---

# Concepts & Patterns Learned

## Scaling Concepts

* Database Sharding
* Cache Sharding
* Partition Keys
* Hash Based Routing
* Consistent Hashing
* Hot Key Problem
* Cache Replication
* Cache Stampede
* Local In-Memory Cache

---

## Global Architecture Concepts

* Geo Routing
* Multi-Region Deployment
* Read Replicas
* Eventual Consistency
* Single Leader Replication
* Global Traffic Routing

---

# Problem Statement

Initial architecture:

```text id="fksd3g"
Client
   |
   v

URL Service
   |
   +--> Redis
   |
   +--> Database
```

Works initially.

Problems appear when:

```text id="h28k5g"
100M URLs/day

1M Reads/sec
```

Need:

* More storage
* More read throughput
* Global deployment

---

# Database Sharding

## Why Shard?

Single database eventually becomes:

```text id="2q9swm"
Storage Bottleneck

Write Bottleneck

Backup Bottleneck
```

---

Example

```text id="odrz0e"
100M URLs/day
```

Assume:

```text id="d5xxsy"
1 KB per record
```

Storage:

```text id="kn6k9m"
100 GB/day
```

Per year:

```text id="qof3l4"
36 TB+
```

Single database is no longer practical.

---

# What Is Sharding?

Instead of:

```text id="xjht3l"
DB1
```

Use:

```text id="vjwvhy"
DB1
DB2
DB3
DB4
```

Visual

```text id="35nq34"
             Router
                |
      -------------------
      |    |    |    |
      v    v    v    v

     DB1 DB2 DB3 DB4
```

---

# Bad Strategy - Range Sharding

Example:

```text id="djpj1j"
1 - 1M      -> DB1
1M - 2M     -> DB2
2M - 3M     -> DB3
```

---

Problem

All new writes go to latest shard.

```text id="h5h7fh"
Current IDs

10M+
```

All writes hit:

```text id="v4fwh8"
DB10
```

Hot shard.

---

# Recommended Strategy - Hash Sharding

Partition Key:

```text id="7azkfx"
short_code
```

Routing:

```text id="0uh4iv"
hash(short_code) % N
```

where:

```text id="vbjyqs"
N = Number of Shards
```

---

Example

```text id="zwmuul"
hash(A9X2P)%4
```

returns:

```text id="khznku"
2
```

Store in:

```text id="n86zy0"
DB2
```

---

Example

```text id="fj7ehd"
hash(XYZ12)%4
```

returns:

```text id="xw7yn0"
0
```

Store in:

```text id="j7t6z8"
DB0
```

---

Benefits

* Uniform storage distribution
* Uniform write distribution
* Uniform read distribution

---

# Consistent Hashing

Problem with:

```text id="p4efm8"
hash(key)%N
```

When:

```text id="0d6qfi"
4 shards
```

becomes:

```text id="oc0g5y"
8 shards
```

Most keys move.

Massive migration required.

---

Solution:

Consistent Hashing

Visual

```text id="6k7w0q"
          DB1

    DB4         DB2

          DB3
```

New shard:

```text id="kh4v5w"
DB5
```

Only a small subset of keys move.

---

Interview Note

For URL Shortener:

```text id="hhzfdg"
Hash Sharding
```

is sufficient.

Consistent Hashing is a bonus discussion.

---

# Cache Sharding

## Why?

Single Redis node eventually becomes bottleneck.

Example:

```text id="7l7ggs"
1M Reads/sec
```

---

Instead of:

```text id="h0x4z2"
Redis
```

Use:

```text id="ecaxqm"
Redis1
Redis2
Redis3
Redis4
```

Visual

```text id="9p0grl"
          Cache Router
                |
      -------------------
      |    |    |    |
      v    v    v    v

      R1   R2   R3   R4
```

---

Routing

```text id="vns2w3"
hash(short_code)%4
```

---

Example

```text id="bj1q9s"
A9X2P -> R2

XYZ12 -> R1
```

---

Benefits

* More memory
* More throughput
* Horizontal scaling

---

# Partition Key

Most important interview concept.

Current request:

```http id="ffv7xl"
GET /A9X2P
```

Lookup key:

```text id="uoz00l"
short_code
```

Therefore:

```text id="wy6fh7"
short_code
```

is the partition key.

---

Both:

```text id="6xy0zd"
Database
```

and

```text id="jqc69w"
Redis
```

should shard using the same key.

---

# Hot Key Problem

## Problem

One URL becomes viral.

Example:

```text id="7gwx5j"
tiny.com/openai
```

Traffic:

```text id="1qgghs"
1M requests/sec
```

---

Hashing sends:

```text id="5u2ewq"
openai
```

to exactly one Redis node.

Example:

```text id="rjv9w8"
Redis Node 2
```

Visual

```text id="w43ru5"
R1   R2   R3   R4

     ^
     |
1M req/sec
```

---

Result

```text id="ymb1wy"
Hot Key
```

One node overloaded.

Others mostly idle.

---

# Solution 1 - Cache Replication

Replicate hot keys.

Visual

```text id="7stecx"
         openai

            |

   ------------------

   |       |       |

   v       v       v

  R2A     R2B     R2C
```

---

Traffic:

```text id="wgs1lb"
1M req/sec
```

becomes:

```text id="x68k0m"
333K
333K
333K
```

---

# Solution 2 - CDN

Most effective solution.

Store:

```text id="zvwltm"
302 Redirect Response
```

at edge locations.

Visual

```text id="ym9tko"
User
 |
 v

CDN
 |
Hit?
/ \
Y  N
|   |
Return Backend
```

Most requests never reach Redis.

---

# Solution 3 - Local Cache

Application memory cache.

Visual

```text id="g69tpo"
Service
   |
   +--> Local Cache
   |
   +--> Redis
   |
   +--> DB
```

Popular URLs stay in process memory.

---

# Cache Stampede

## Problem

Redis crashes.

Traffic:

```text id="r6ffk4"
1M req/sec
```

suddenly hits database.

Visual

```text id="8fgt8e"
Users
  |
  v

Database
```

Database becomes overloaded.

---

Also called:

```text id="jszkr3"
Thundering Herd
```

---

Mitigations

* Redis Replicas
* CDN
* Local Cache
* Request Coalescing
* Rate Limiting

---

# Multi Region Deployment

## Problem

All infrastructure is in US.

User in India:

```text id="a0qfbc"
India -> US
```

Latency:

```text id="sbp50k"
200-300 ms
```

Too slow.

---

# Solution

Deploy globally.

Visual

```text id="09afbj"
           Global DNS

                |

      --------------------

      |        |        |

      v        v        v

      US     Europe    India
```

---

# Geo Routing

User requests:

```text id="m01f5v"
tiny.com/abc123
```

DNS routes to nearest region.

Example:

```text id="f2m2lv"
India User
```

goes to:

```text id="dcj6kr"
India Region
```

---

Benefits

```text id="k0wg9i"
10-30 ms latency
```

instead of:

```text id="4lsf7e"
200-300 ms
```

---

# Data Replication Problem

URL created in US.

```text id="u4x6t0"
abc123 -> google.com
```

stored in:

```text id="2hgs2e"
US Database
```

---

India region needs same data.

Requires replication.

---

# Single Leader Replication

Visual

```text id="1f10j4"
           Primary

              US

               |

    --------------------

    |                  |

    v                  v

India Replica     Europe Replica
```

---

Write Flow

```text id="u9um5m"
Create URL
    |
    v
US Primary
```

---

Replication

```text id="v7z7yd"
US Primary
    |
    v
Replicas
```

---

Read Flow

```text id="om5z3x"
User
   |
   v
Nearest Replica
```

---

# Eventual Consistency

URL created:

```text id="ux2nmh"
abc123 -> google.com
```

---

Immediately:

```text id="w7ndg5"
US knows
```

---

500ms later:

```text id="3opxdt"
India knows
```

---

1 second later:

```text id="4ahzws"
Europe knows
```

---

Eventually:

```text id="r6phki"
All regions agree
```

---

Acceptable because:

```text id="03rkvg"
Availability > Consistency
```

for URL Shortener.

---

# Write Strategy

## Option 1

Single Write Region

Visual

```text id="xho9vg"
All Writes
    |
    v
US Primary
```

Pros:

* Simple
* No conflicts

Cons:

* Higher write latency

---

## Option 2

Multi Primary

Visual

```text id="5n94n6"
US Primary

India Primary

Europe Primary
```

Pros:

* Fast local writes

Cons:

* Conflict resolution
* More complexity

---

Recommended for interview:

```text id="7bpr3n"
Single Primary
Multiple Read Replicas
```

---

# CDN vs Multi Region

## CDN

Stores:

```text id="sxqokl"
Responses
Cached Redirects
```

---

Example

```text id="ebzbkp"
302 Redirect
```

---

## Multi Region

Stores:

```text id="sh0nkf"
Application
Database
Cache
```

in multiple geographic regions.

---

# Final Global Architecture

```text id="9rqfoz"
User
  |
  v

Geo DNS
  |
  v

Nearest Region
  |
  v

Local Cache
  |
Hit?
/ \
Y  N
|   |
Return Local Replica
```

---

# Interview Summary

Scaling Strategy:

* Hash-based DB Sharding
* Redis Sharding
* Consistent Hashing for future growth
* Cache Replication
* CDN for hot URLs

Global Strategy:

* Geo Routing
* Multi Region Deployment
* Single Primary Database
* Read Replicas
* Eventual Consistency

Core Interview Takeaway:

```text id="24upg6"
Partition by short_code.

Scale reads using cache and replicas.

Handle viral URLs using CDN and cache replication.

Serve users from the nearest region using geo routing.

Accept eventual consistency to maximize availability.
```
