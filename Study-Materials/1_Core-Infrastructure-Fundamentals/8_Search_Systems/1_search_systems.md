# Search Systems

## Topics Covered

### Basics

* Inverted Index
* Posting Lists
* Indexing Pipeline
* Ranking

### Advanced Concepts

* Stop Words
* Stemming
* Search Architecture
* Elasticsearch
* Index Compression
* Distributed Search
* Vocabulary Growth
* Zipf's Law

---

# The Problem Search Systems Solve

Imagine Amazon.

Products:

```text
PS5
iPhone
MacBook
Samsung TV
...
100 Million Products
```

User searches:

```text
wireless headphones
```

Question:

```text
How do we find relevant products in milliseconds?
```

---

Naive solution:

```sql
SELECT *
FROM products
WHERE description LIKE '%wireless%'
```

For:

```text
100 Million Rows
```

this becomes extremely slow.

We need something better.

---

# Core Idea

Instead of searching documents every time:

```text
Search Request
      ↓
Scan Entire Database
```

we preprocess data once.

Build:

```text
Search Index
```

Then queries become fast.

---

# What Is A Document?

In search systems:

```text
Document
```

means any searchable item.

Examples:

### Google

```text
Web Page
```

### Amazon

```text
Product
```

### YouTube

```text
Video
```

### LinkedIn

```text
Profile
```

Everything becomes a document.

---

# Inverted Index

This is the most important search concept.

Interviewers absolutely expect you to know this.

---

# Why Normal Search Doesn't Work

Suppose:

```text
Doc1 = "apple iphone charger"
Doc2 = "samsung wireless charger"
Doc3 = "apple macbook"
```

User searches:

```text
charger
```

Question:

```text
Which documents contain charger?
```

Naive approach:

```text
Read Doc1
Read Doc2
Read Doc3
```

Too slow at scale.

---

# Inverted Index Idea

Instead of:

```text
Document → Words
```

store:

```text
Word → Documents
```

Hence:

```text
Inverted
```

---

# Example

Documents:

```text
Doc1 = apple iphone charger
Doc2 = samsung wireless charger
Doc3 = apple macbook
```

Index:

```text
apple     → Doc1, Doc3

iphone    → Doc1

charger   → Doc1, Doc2

samsung   → Doc2

wireless  → Doc2

macbook   → Doc3
```

---

Search:

```text
charger
```

returns instantly:

```text
Doc1
Doc2
```

No scanning.

---

# Visual Representation

```text
apple
  ↓
[1,3]

charger
  ↓
[1,2]

wireless
  ↓
[2]
```

These lists are called:

```text
Posting Lists
```

---

# Search Query Example

User searches:

```text
apple charger
```

Lookup:

```text
apple   → [1,3]

charger → [1,2]
```

Intersection:

```text
[1]
```

Result:

```text
Doc1
```

---

# Why It Scales

Google doesn't scan:

```text
100 Billion Pages
```

per search.

Instead:

```text
word
   ↓
posting list
```

Lookup is extremely fast.

---

# Indexing Pipeline

Question:

```text
How does the index get built?
```

This is:

```text
Indexing Pipeline
```

---

# Example

New product:

```text
Apple iPhone 17 Wireless Charger
```

arrives.

Before indexing:

```text
Raw Text
```

Pipeline processes it.

---

# Step 1: Tokenization

Split text into words.

```text
Apple
iPhone
17
Wireless
Charger
```

becomes:

```text
apple
iphone
17
wireless
charger
```

---

# Step 2: Normalization

Convert:

```text
APPLE
Apple
apple
```

into:

```text
apple
```

Reduces duplicates.

---

# Step 3: Stop Word Removal

Remove:

```text
the
is
and
of
to
```

Example:

```text
The best wireless charger
```

becomes:

```text
best
wireless
charger
```

---

# Why Remove Stop Words?

Words like:

```text
the
a
of
is
and
```

appear in millions of documents.

Example:

```text
the
```

may appear in:

```text
500 Million Documents
```

Creating enormous posting lists.

Removing them saves:

* Storage
* Memory
* Query Time

---

# Step 4: Stemming

Reduce words to root form.

Examples:

```text
running
runs
ran
```

becomes:

```text
run
```

Search:

```text
run
```

matches:

```text
running
runs
ran
```

---

# Step 5: Build Inverted Index

Final tokens:

```text
apple
iphone
wireless
charger
```

are added to the index.

---

# Indexing Architecture

```text
Document

   │

   ▼

Tokenizer

   │

   ▼

Normalizer

   │

   ▼

Stop Word Removal

   │

   ▼

Stemmer

   │

   ▼

Inverted Index
```

---

# Why Indexing Is Separate

Search traffic:

```text
Millions Queries/Sec
```

Product updates:

```text
Thousands Updates/Sec
```

Different workloads.

Usually:

```text
Write Path
```

and

```text
Search Path
```

are separated.

---

# Real Search Architecture

```text
Product DB

      │

      ▼

Kafka

      │

      ▼

Indexer Service

      │

      ▼

Elasticsearch Index
```

Very common architecture.

---

# Ranking

Suppose search:

```text
iphone charger
```

returns:

```text
10000 Products
```

Question:

```text
Which one should appear first?
```

This is:

```text
Ranking
```

---

# Relevance Score

Each document gets a score.

Higher score:

```text
Appears First
```

Example:

```text
Product A = score 95

Product B = score 82

Product C = score 60
```

Order:

```text
A
B
C
```

---

# Ranking Signal 1: Term Frequency

Document:

```text
iphone charger charger charger
```

is usually more relevant than:

```text
iphone charger
```

More occurrences:

```text
Higher Score
```

---

# Ranking Signal 2: Field Importance

Match in title:

```text
Apple iPhone Charger
```

is usually more valuable than match in description.

Example:

```text
Title Match = +100

Description Match = +20
```

---

# Ranking Signal 3: Popularity

Product sold:

```text
1 Million Times
```

likely ranks above:

```text
3 Sales
```

---

# Ranking Signal 4: Freshness

Important for news.

Example:

```text
Election Results
```

Today's article usually ranks above:

```text
2019 Article
```

---

# Ranking Signal 5: Click Through Rate

If users consistently click:

```text
Result A
```

more than:

```text
Result B
```

A gets boosted.

---

# Example Ranking Formula

Conceptually:

```text
Score =
Keyword Match
+ Popularity
+ Freshness
+ CTR
```

Higher score wins.

---

# Elasticsearch

Most companies do not build search engines.

They use Elasticsearch.

Elasticsearch internally provides:

* Inverted Indexes
* Ranking
* Tokenization
* Distributed Search
* Fast Query Processing

---

# Search Flow

User searches:

```text
wireless charger
```

Search Service:

```text
Query Elasticsearch
```

Elasticsearch:

```text
Find Matching Documents
```

using:

```text
Inverted Index
```

Compute:

```text
Ranking Scores
```

Return:

```text
Top Results
```

Typically:

```text
< 100 ms
```

---

# Doesn't The Inverted Index Become Huge?

Excellent interview question.

---

Suppose:

```text
1 Million Documents
```

Each contains:

```text
100 Words
```

Total words:

```text
100 Million Words
```

Many repeat.

The index stores:

```text
Unique Word
      ↓
Posting List
```

instead of:

```text
Word
      ↓
Entire Documents
```

which is much smaller.

---

# Unique Terms Can Be Massive

Dictionary words:

```text
500,000+
```

Real systems include:

```text
Product Names
Usernames
URLs
Hashtags
Misspellings
IDs
```

Large search systems may contain:

```text
Hundreds Of Millions
or
Billions Of Unique Terms
```

---

# The Real Cost Is Posting Lists

The expensive part isn't the number of words.

It's huge posting lists.

Example:

```text
the
```

may map to:

```text
500 Million Documents
```

which is enormous.

This is one reason stop words are removed.

---

# Compression

Search engines heavily compress posting lists.

Example:

Instead of:

```text
1
2
3
4
5
6
```

store:

```text
1
+1
+1
+1
+1
+1
```

This technique is called:

```text
Delta Encoding
```

and dramatically reduces storage.

---

# Distributed Search

At Google scale:

```text
Entire Index
```

cannot fit on one machine.

The index is distributed.

Example:

```text
Shard 1 → Terms A-F

Shard 2 → Terms G-M

Shard 3 → Terms N-Z
```

or:

```text
Hash(term)
```

based partitioning.

---

When searching:

```text
iphone charger
```

multiple shards participate.

---

# Zipf's Law

An important observation about language.

A few terms:

```text
the
is
and
of
```

appear extremely frequently.

Most terms:

```text
wirelessgamingheadset2026
```

appear very rarely.

This uneven distribution is known as:

```text
Zipf's Law
```

---

# Common Interview Questions

## Why Not Search The Database Directly?

Because:

```text
LIKE '%query%'
```

doesn't scale.

Need:

```text
Inverted Index
```

---

## Why Separate Search Database?

Because:

```text
Search Workloads
```

and

```text
Transactional Workloads
```

are very different.

---

## Why Elasticsearch?

Because it provides:

```text
Distributed Search
Ranking
Inverted Indexes
Fast Queries
```

out of the box.

---

# What Interviewers Expect

Know:

### Inverted Index

```text
Word → Documents
```

---

### Posting List

```text
List Of Documents For Word
```

---

### Indexing Pipeline

```text
Tokenization
Normalization
Stop Word Removal
Stemming
Index Build
```

---

### Ranking

```text
Sort Results By Relevance
```

---

### Search Architecture

```text
DB
 ↓
Kafka
 ↓
Indexer
 ↓
Elasticsearch
```

---

# Senior Engineer Mental Model

```text
Documents
      ↓
Indexing Pipeline
      ↓
Inverted Index
      ↓
Search Query
      ↓
Candidate Documents
      ↓
Ranking
      ↓
Top Results
```

---

# Most Important Interview Takeaway

> Search systems are fast because they don't search documents directly. They precompute an inverted index that maps words to documents. Queries retrieve candidate documents from the index, and ranking algorithms determine which results are most relevant to return. Large-scale systems further optimize storage through stop-word removal, compression, and sharding.
