# Fanout Patterns

## Used In

* Instagram Feed
* Twitter/X Timeline
* Facebook News Feed
* LinkedIn Feed
* TikTok Following Feed

The core problem is deceptively simple:

> User opens the app. How do we generate their feed quickly?

---

# The Feed Problem

Imagine Instagram.

You follow:

```text
Friend A
Friend B
Friend C
```

Friend A posts:

```text
Photo A
```

Friend B posts:

```text
Photo B
```

Friend C posts:

```text
Photo C
```

When you open Instagram:

```text
Show Latest Posts
```

Simple at first.

---

Now imagine scale.

Instagram:

```text
2+ Billion Users
```

Celebrities:

```text
100 Million Followers
```

Posts:

```text
Millions Per Minute
```

Question:

```text
How do we build the feed efficiently?
```

This leads to:

```text
Fanout On Write
Fanout On Read
```

---

# First Understand Fanout

Fanout means:

```text
One Action
        ↓
Many Recipients
```

Example:

```text
Cristiano Ronaldo Posts
```

Need to notify:

```text
100M Followers
```

One write.

Millions of destinations.

That is:

```text
Fanout
```

---

# Fanout On Write

Also called:

```text
Push Model
```

---

# Core Idea

When a user creates a post:

```text
Immediately push
that post into
followers' feeds.
```

---

# Example

User A has:

```text
100 Followers
```

User A posts:

```text
Hello World
```

System immediately writes:

```text
Hello World
```

into:

```text
Follower1 Feed
Follower2 Feed
Follower3 Feed
...
Follower100 Feed
```

---

# Architecture

```text
User Creates Post

        │

        ▼

Post Service

        │

        ▼

Fanout Service

        │

 ┌──────┼──────┐
 ▼      ▼      ▼

Feed1 Feed2 Feed3
```

---

# What Happens During Read?

User opens app.

Feed already exists.

Simply:

```sql
SELECT * FROM Feed
```

Very fast.

---

# Real World Example

Suppose:

```text
100 Followers
```

Post created.

Write operation:

```text
100 Feed Writes
```

Read operation:

```text
1 Feed Read
```

So:

```text
Writes Expensive
Reads Cheap
```

---

# Why Instagram Likes This

Instagram users:

```text
Read Far More Than Write
```

Typical user:

```text
1 Post Per Day
100 Feed Views Per Day
```

Therefore:

```text
Optimize Reads
```

makes sense.

---

# Example Numbers

Suppose:

```text
1 Million Users
```

Each:

```text
1 Post Per Day
```

Writes:

```text
1 Million
```

Feed reads:

```text
100 Million
```

Read traffic dominates.

Fanout on write moves work to write time.

Then reads become lightning fast.

---

# The Celebrity Problem

Now imagine:

```text
Taylor Swift
```

or

```text
Cristiano Ronaldo
```

with:

```text
100 Million Followers
```

New post arrives.

Need:

```text
100 Million Feed Writes
```

immediately.

That's enormous.

Possible result:

```text
Database Overload
Kafka Overload
Feed Service Overload
```

This is the biggest weakness of Fanout On Write.

---

# Fanout On Read

Also called:

```text
Pull Model
```

---

# Core Idea

Instead of pushing posts into feeds:

Store posts only once.

---

# Example

Friend A posts:

```text
Photo A
```

Store:

```text
Posts Table
```

Only once.

When user opens feed:

System computes feed.

---

# Architecture

```text
User Opens Feed

       │

       ▼

Get Following List

       │

       ▼

Fetch Posts

       │

       ▼

Merge Results

       │

       ▼

Return Feed
```

---

# Example

You follow:

```text
A
B
C
```

Feed Service:

```text
Get Recent Posts From A
Get Recent Posts From B
Get Recent Posts From C
```

Merge:

```text
C Post
B Post
A Post
```

Return feed.

---

# Cost Analysis

Post creation:

```text
1 Write
```

Very cheap.

Feed read:

```text
Fetch Many Posts
Merge
Sort
Rank
```

Expensive.

So:

```text
Writes Cheap
Reads Expensive
```

---

# Why Twitter Originally Used This

Celebrities.

Suppose:

```text
100M Followers
```

Fanout on write:

```text
100M Feed Writes
```

per tweet.

Very expensive.

Fanout on read:

```text
1 Tweet Write
```

Much cheaper.

---

# Comparison

| Metric             | Fanout On Write | Fanout On Read |
| ------------------ | --------------- | -------------- |
| Write Cost         | High            | Low            |
| Read Cost          | Low             | High           |
| Feed Generation    | Fast            | Slow           |
| Celebrity Handling | Poor            | Good           |
| Normal Users       | Excellent       | Good           |

---

# Real World Hybrid Approach

This is what interviewers love.

Because almost nobody uses pure fanout anymore.

---

# Instagram Style

Normal users:

```text
Fanout On Write
```

Celebrities:

```text
Fanout On Read
```

---

# Why?

Suppose:

```text
Akash
```

has:

```text
300 Followers
```

Fanout cost:

```text
300 Writes
```

Easy.

Suppose:

```text
Cristiano Ronaldo
```

has:

```text
600 Million Followers
```

Fanout cost:

```text
600 Million Writes
```

Crazy.

System decides:

```text
Followers < Threshold
```

↓

```text
Fanout On Write
```

Else:

```text
Fanout On Read
```

---

# Hybrid Architecture

```text
Post Created

      │

      ▼

Follower Count?

      │

 ┌────┴─────┐
 ▼          ▼

Small     Celebrity

 ▼          ▼

Write     Read
Fanout    Fanout
```

---

# How Feed Is Usually Stored

Feed table:

```text
UserID
PostID
Timestamp
```

Example:

```text
101 | PostA
101 | PostB
101 | PostC
```

When user opens app:

```sql
SELECT *
FROM Feed
WHERE userId=101
ORDER BY timestamp DESC
LIMIT 50
```

Very fast.

---

# Interview Example

Suppose interviewer asks:

> Design Instagram Feed

Good answer:

> Since feed reads vastly outnumber feed writes, I would use fanout-on-write for most users so feed retrieval remains extremely fast. However, for celebrity accounts with millions of followers, fanout-on-write becomes too expensive. Therefore I would use a hybrid approach where normal users use fanout-on-write while celebrity accounts use fanout-on-read.

This immediately signals senior-level understanding.

---

# When To Use Fanout On Write

Good for:

```text
Instagram Following Feed
Facebook Friends Feed
LinkedIn Feed
```

where:

```text
Reads >> Writes
```

---

# When To Use Fanout On Read

Good for:

```text
Celebrity Timelines
Twitter/X
Large Follower Graphs
```

where:

```text
Followers Extremely Large
```

---

# Senior Engineer Mental Model

Think:

```text
User Creates Post
        ↓

Need Followers To See It
```

Question:

```text
Do work now?
or
Do work later?
```

---

## Fanout On Write

```text
Do Work During Write
```

↓

```text
Fast Reads
Expensive Writes
```

---

## Fanout On Read

```text
Do Work During Read
```

↓

```text
Cheap Writes
Expensive Reads
```

---

# Most Important Interview Takeaway

> Fanout-on-write optimizes read latency by precomputing feeds, while fanout-on-read optimizes write scalability by generating feeds at read time. Most large social networks use a hybrid model, applying fanout-on-write for normal users and fanout-on-read for celebrities.
