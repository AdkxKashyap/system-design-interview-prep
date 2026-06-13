# URL Shortener - Analytics Subsystem Notes

This document covers the Analytics subsystem for a URL Shortener.

The goal is to collect click statistics without impacting redirect latency.

---

# Concepts & Patterns Learned

## Core Concepts

* Event Driven Architecture
* Kafka
* Producer / Consumer Pattern
* Asynchronous Processing
* Aggregation
* Analytics Databases
* Real-Time Analytics
* Stream Processing

---

## Design Patterns

### Producer → Kafka → Consumer

```text
Producer
   |
   v
 Kafka
   |
   v
Consumer
```

Used in:

* URL Analytics
* Twitter Events
* Notification Systems
* Uber Events
* YouTube View Tracking
* Monitoring Systems

---

### Asynchronous Processing

```text
User Request
    |
    +-----> Immediate Response
    |
    +-----> Background Processing
```

User never waits for analytics.

---

### Event Aggregation

```text
1000 Events
    |
    v
1 Aggregated Write
```

Reduces database load significantly.

---

# Business Requirements

We want analytics for every short URL.

Example:

```text
tiny.com/openai
```

Dashboard:

```text
Total Clicks

Clicks Per Day

Country Distribution

Browser Distribution

Device Distribution
```

---

# Naive Design

Every redirect updates analytics directly.

Visual:

```text
User Click
    |
    v
Update Analytics DB
    |
    v
Redirect User
```

---

# Problems

Suppose:

```text
1M redirects/sec
```

Now:

```text
1M analytics writes/sec
```

Analytics database becomes bottleneck.

---

Redirect latency increases.

Current goal:

```text
< 50ms redirect latency
```

Analytics should never affect redirect performance.

---

# Key Design Principle

Analytics is:

```text
Important
```

but not:

```text
Latency Critical
```

Redirect is latency critical.

Analytics is not.

---

# Event Driven Architecture

Instead of updating analytics synchronously:

Visual:

```text
User Click
     |
     v

URL Service

     |

     +-----> Redirect User

     |

     +-----> Publish Event
```

Redirect happens immediately.

Analytics happens later.

---

# Kafka Introduction

Kafka acts as:

```text
Distributed Event Buffer
```

Visual:

```text
URL Service
     |
     v
   Kafka
     |
     v
Analytics Service
```

---

# Redirect Flow

User clicks:

```text
tiny.com/openai
```

---

URL Service performs lookup:

```text
openai -> openai.com
```

---

Immediately returns:

```http
302 Redirect
```

---

At the same time publishes event:

```json
{
  "shortCode":"openai",
  "timestamp":"2026-01-01T10:00:00",
  "country":"India",
  "browser":"Chrome",
  "device":"Mobile"
}
```

to Kafka.

---

User does not wait.

---

# Why Kafka?

Assume:

```text
1M click events/sec
```

Analytics service can process:

```text
100K events/sec
```

Without Kafka:

```text
Events Lost
```

---

With Kafka:

```text
Events Buffered
```

Visual:

```text
1M Events/sec

      |

      v

    Kafka

      |

      v

100K Consumers/sec
```

Kafka absorbs traffic spikes.

---

# Buffering

Kafka allows:

```text
Producer Speed
```

and

```text
Consumer Speed
```

to be independent.

---

This is called:

```text
Decoupling
```

---

# Analytics Consumer

Analytics service consumes events.

Visual:

```text
Kafka
   |
   v
Analytics Consumer
```

Example event:

```json
{
  "shortCode":"openai",
  "country":"India"
}
```

---

Consumer processes analytics events.

---

# Analytics Database

Do NOT use URL database.

Different workload.

---

## URL Database

Optimized for:

```text
Point Lookups
```

Example:

```text
short_code -> long_url
```

---

## Analytics Database

Optimized for:

```text
Aggregation
```

Examples:

```text
Clicks Per Day

Top Countries

Top Browsers

Top URLs
```

---

Typical Choices

* ClickHouse
* Druid
* BigQuery
* Elasticsearch

Interview answer:

```text
Analytics Database optimized for reporting and aggregations
```

is sufficient.

---

# Analytics Event Model

Example:

```json
{
  "shortCode":"openai",
  "timestamp":"2026-01-01T10:00:00",
  "country":"India",
  "browser":"Chrome",
  "device":"Mobile"
}
```

Each click produces one event.

---

# Problem With Writing Every Event

Suppose:

```text
1M clicks/sec
```

Writing:

```text
1M DB writes/sec
```

is expensive.

---

Need aggregation.

---

# Aggregation Layer

Visual:

```text
Kafka
   |
   v
Aggregator
   |
   v
Analytics DB
```

---

Events:

```text
openai
openai
openai
openai
openai
```

---

Aggregator keeps counters:

```text
openai = 5000
```

in memory.

---

Every minute:

```text
openai
count = 5000
```

written to database.

---

Benefits

```text
5000 events

becomes

1 write
```

Huge reduction in DB load.

---

# Dashboard Flow

User opens:

```text
Analytics Dashboard
```

---

Request:

```http
GET /analytics/openai
```

---

Analytics Service queries:

```text
Analytics Database
```

---

Response:

```json
{
  "totalClicks": 1200000,
  "india": 700000,
  "usa": 300000,
  "uk": 200000
}
```

---

# Why Not Redis?

Redis works well for:

```text
Current Counters
```

Examples:

```text
Current Click Count
```

---

Redis is not ideal for:

```text
Historical Analytics

Monthly Reports

Yearly Reports

Country Trends
```

Need persistent analytics storage.

---

# Kafka Streams Discussion

## Basic Version

Recommended interview solution.

```text
Producer
   |
Kafka
   |
Consumer
   |
Aggregator
   |
Analytics DB
```

Simple.

Easy to explain.

Sufficient for most interviews.

---

## Kafka Streams Version

Useful when requirements become:

```text
Real-Time Dashboards

Top URLs

Sliding Windows

Per Minute Analytics

Top Countries
```

Visual:

```text
Producer
   |
Kafka
   |
Kafka Streams
   |
Analytics DB
```

---

Benefits

* Built-in aggregation
* Windowing
* Stateful processing
* Fault recovery

---

Tradeoff

More complexity.

For URL Shortener:

```text
Consumer + Aggregator
```

is usually enough.

Mention Kafka Streams only as future evolution.

---

# End-to-End Analytics Architecture

```text
User Click
     |
     v

URL Service

     |

     +------> Redirect User

     |

     +------> Kafka

                  |

                  v

         Analytics Consumer

                  |

                  v

             Aggregator

                  |

                  v

           Analytics DB

                  |

                  v

          Analytics Dashboard
```

---

# Interview Summary

A strong answer:

> Analytics should not be part of the redirect path. I would publish click events asynchronously to Kafka, process them using analytics consumers, aggregate metrics, and store results in an analytics database optimized for reporting. This keeps redirect latency low while allowing the analytics pipeline to scale independently.

---

# Core Interview Takeaways

```text
Never block user redirects for analytics.

Use Kafka to decouple producers and consumers.

Use asynchronous processing.

Aggregate before writing to analytics storage.

Use a separate analytics database.

Consider Kafka Streams only for advanced real-time analytics.
```
