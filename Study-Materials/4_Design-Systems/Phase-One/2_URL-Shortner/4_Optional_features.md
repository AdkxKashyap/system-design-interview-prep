# URL Shortener - Custom Alias & URL Expiration Notes

This document covers common follow-up requirements that interviewers ask after the core URL Shortener design.

These features should be implemented with minimal architectural changes.

---

# Concepts & Patterns Learned

## Concepts

* Custom Aliases
* Unique Constraints
* URL Expiration
* Metadata-Based Features
* Background Cleanup Jobs
* Cache Invalidation
* Redis TTL
* Source of Truth
* Cache Aside Pattern

---

## Design Patterns

### Validation On Write

```text id="cg6ehg"
Create Request
      |
      v

Validate
      |
      v

Store
```

Used for:

* Custom aliases
* Username creation
* Unique identifiers

---

### Metadata-Based Features

Store additional metadata instead of creating new services.

Example:

```text id="g7jt97"
expires_at
```

---

### Cache Aside Invalidation

```text id="5nv7t4"
Update DB
    |
Delete Cache
```

Used in:

* URL Shortener
* Twitter Feed
* YouTube Metadata
* User Profiles

---

# Custom Alias

## Requirement

Allow users to create:

```text id="nzrrj7"
tiny.com/openai
```

instead of:

```text id="wvk0j6"
tiny.com/A9X2P
```

---

# API Changes

Current API:

```http id="cnw9y7"
POST /shorten
```

Request:

```json id="uz0kwv"
{
  "url":"https://openai.com"
}
```

---

New Request:

```json id="l93wpr"
{
  "url":"https://openai.com",
  "customAlias":"openai"
}
```

---

# URL Creation Flow

Current Flow

```text id="n1l88q"
Generate ID
    |
Base62
    |
Store
```

---

Custom Alias Flow

```text id="prm89x"
Client
   |
   v

URL Service
   |
   v

Alias Exists?
   |
  / \
 Y   N
 |   |
Error Store Mapping
```

---

# Database Design

No schema changes required.

Current table:

| short_code | long_url   |
| ---------- | ---------- |
| A9X2P      | google.com |

---

Can now store:

| short_code | long_url   |
| ---------- | ---------- |
| openai     | openai.com |
| A9X2P      | google.com |

---

# Uniqueness Problem

Two users may request:

```text id="jsztly"
openai
```

at the same time.

---

Visual

```text id="ijvrcg"
Server A

Server B

Both want:
openai
```

---

Need uniqueness guarantee.

---

# Solution

Database unique constraint.

Example:

```sql id="xemyyq"
UNIQUE(short_code)
```

---

Why?

Application checks are not enough.

Example:

```text id="k2zcgf"
Server A -> Alias Available

Server B -> Alias Available
```

Race condition.

---

Database constraint is the final protection.

---

# Interview Answer

> I would enforce alias uniqueness at the database level using a unique index on short_code. This guarantees correctness even when multiple servers process requests concurrently.

---

# Performance Impact

Custom aliases are typically rare.

Example:

```text id="7guppr"
<1% of URLs
```

Therefore:

```text id="l5jzh8"
Extra validation lookup
```

is acceptable.

---

# URL Expiration

## Requirement

Allow users to create URLs that expire.

Example:

```text id="wyjlwm"
tiny.com/openai
```

expires after:

```text id="1r4sjl"
30 days
```

---

After expiration:

```text id="mqsjyo"
Redirect should stop working
```

---

# Schema Changes

Current table:

| short_code | long_url | created_at |
| ---------- | -------- | ---------- |

---

Add:

| short_code | long_url | expires_at |
| ---------- | -------- | ---------- |

Example:

| openai | openai.com | 2026-08-01 |

---

# Redirect Flow

Current:

```text id="vbm65s"
Lookup URL
    |
Redirect
```

---

New Flow:

```text id="3wlfm4"
Lookup URL
    |
Check Expiration
    |
Expired?
 /      \
Y        N
|        |
404   Redirect
```

---

# Expiration Check

Example:

```text id="jlwmx1"
current_time > expires_at
```

---

If true:

Return:

```http id="jlwmx2"
410 Gone
```

or

```http id="jlwmx3"
404 Not Found
```

---

Otherwise:

```text id="jlwmx4"
Redirect User
```

---

# Source Of Truth

Expiration must always be stored in database.

Example:

```text id="jlwmx5"
expires_at
```

---

Database remains:

```text id="jlwmx6"
Source of Truth
```

---

Even if:

```text id="jlwmx7"
Redis crashes
```

or

```text id="jlwmx8"
Cache misses occur
```

the system can still determine whether URL is expired.

---

# Redis TTL Optimization

Best practical solution.

Suppose:

```text id="jlwmx9"
Expiration Date

2026-08-01
```

Current date:

```text id="jlwmy0"
2026-07-01
```

Remaining life:

```text id="jlwmy1"
30 days
```

---

Store in Redis:

```text id="jlwmy2"
Key: openai

Value: openai.com

TTL: 30 days
```

---

Redis automatically removes key when TTL expires.

---

Benefits

```text id="jlwmy3"
No cache cleanup service

No scheduled cache deletion

No extra infrastructure
```

---

# Redirect Flow With TTL

```text id="jlwmy4"
Request
   |
   v

Redis
```

---

Cache Hit

```text id="jlwmy5"
Redirect
```

---

Cache Miss

```text id="jlwmy6"
Database Lookup
     |
Check Expiration
     |
Return Result
```

---

# Why DB Check Is Still Required

TTL is only an optimization.

Suppose:

```text id="jlwmy7"
Redis Restart
```

---

Cache becomes empty.

Request:

```text id="jlwmy8"
GET /openai
```

---

Database must still verify:

```text id="jlwmy9"
expires_at
```

---

Otherwise expired URLs become valid again.

---

# URL Expiration Cleanup

Over time:

```text id="jlwmz0"
Millions of expired URLs
```

may remain in database.

---

Need cleanup process.

---

# Option 1 - Hard Delete

Visual

```text id="jlwmz1"
Scheduler
    |
    v

Cleanup Service
    |
    v

Delete Expired URLs
```

---

Example:

```text id="jlwmz2"
Run nightly
```

---

# Option 2 - Soft Delete

Add:

```text id="jlwmz3"
is_expired = true
```

---

Benefits:

```text id="jlwmz4"
Recovery

Audit

Analytics Preservation
```

---

Later:

```text id="jlwmz5"
Hard Delete
```

after:

```text id="jlwmz6"
90 days
```

---

Recommended interview answer:

```text id="jlwmz7"
Soft Delete
→ Later Hard Delete
```

---

# Cache Invalidation

Suppose URL originally expires in:

```text id="jlwmz8"
30 days
```

---

User updates:

```text id="jlwmz9"
90 days
```

---

Database:

```text id="jlwmza"
Updated
```

---

Redis:

```text id="jlwmzb"
Still has old TTL
```

---

Cache is now stale.

---

# Solution

Use Cache Aside Invalidation.

```text id="jlwmzc"
Update DB
    |
Delete Cache
```

---

Next request:

```text id="jlwmzd"
Cache Miss
    |
Rebuild Cache
```

with correct expiration.

---

# Combined Architecture

Schema:

| short_code | long_url | expires_at |
| ---------- | -------- | ---------- |

---

Creation Flow

```text id="jlwmze"
Client
   |
   v

URL Service
   |
Alias Provided?
 /      \
Y        N
|        |
Validate Generate ID
Alias     |
           v
         Store
```

---

Redirect Flow

```text id="jlwmzf"
Client
   |
   v

Redis

Hit?
 / \
Y   N
|   |
Redirect DB
          |
          v
Check Expiration
          |
       Expired?
        /    \
       Y      N
       |      |
     404  Redirect
```

---

# Interview Summary

## Custom Alias

Changes:

* New API field
* Alias validation
* Database unique constraint

Key takeaway:

```text id="jlwmzg"
UNIQUE(short_code)
```

guarantees correctness.

---

## Expiration

Changes:

* expires_at column
* Expiration validation during redirect
* Redis TTL optimization
* Cleanup process

Key takeaway:

```text id="jlwmzh"
Database is source of truth.

Redis TTL is an optimization.
```

---

# Core Interview Takeaways

```text id="jlwmzi"
Use database constraints for uniqueness.

Store expiration as metadata.

Set Redis TTL equal to URL lifetime.

Always validate expiration in DB on cache misses.

Use Cache Aside invalidation when expiration changes.

Prefer soft delete followed by hard delete.
```
