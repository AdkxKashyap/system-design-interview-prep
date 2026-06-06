# Object Storage Deep Dive — Chunking

# 1. Why Do We Need Chunking?

Suppose user uploads:

```text
10 GB movie.mp4
```

A naive approach would be:

```text
Store Entire File As One Giant Blob
```

This creates several problems.

---

## Problem 1 — Upload Failure

Suppose:

```text
9.9 GB uploaded
Network disconnects
```

Without chunking:

```text
Restart Entire Upload
```

Very inefficient.

---

## Problem 2 — Replication

Suppose object needs replication.

Without chunking:

```text
Copy Entire 10 GB Object
```

every time.

Slow and expensive.

---

## Problem 3 — Storage Hotspots

Suppose:

```text
Entire file stored on Node1
```

Millions of users download it.

Node1 becomes bottleneck.

---

## Problem 4 — No Parallelism

Only one machine serves entire file.

Cannot leverage distributed infrastructure efficiently.

---

# 2. What is Chunking?

Chunking means:

```text
Large File
    ↓
Split Into Smaller Pieces
```

Example:

```text
10 GB Movie

Chunk1
Chunk2
Chunk3
...
Chunk2000
```

If chunk size is:

```text
5 MB
```

Then:

```text
10 GB ÷ 5 MB
≈ 2000 chunks
```

---

# 3. How Are Objects Chunked?

Suppose file:

```text
movie.mp4
```

contains bytes:

```text
Byte0
Byte1
Byte2
...
Byte10,000,000,000
```

Object storage simply splits byte ranges.

---

## Example

```text
Chunk1
Bytes 0 - 4,999,999
```

```text
Chunk2
Bytes 5,000,000 - 9,999,999
```

```text
Chunk3
Bytes 10,000,000 - 14,999,999
```

and so on.

Nothing magical.

Chunking is essentially:

```text
Split Byte Stream Into Fixed-Size Pieces
```

---

# 4. Metadata is Critical

Chunks alone are useless.

Storage system must know:

```text
Which chunks belong to which object?
What order are chunks in?
Where are chunks stored?
```

Metadata stores this information.

---

## Example Metadata

```json
{
  "object": "video.mp4",
  "chunks": [
    "chunk1",
    "chunk2",
    "chunk3"
  ]
}
```

---

# Core Mental Model

```text
Object
   │
   ▼

Metadata Record

   │
   ▼

Chunk Locations
```

Metadata is essentially the index of the object.

---

# 5. Where Does Each Chunk Live?

Suppose storage cluster contains:

```text
StorageNode1
StorageNode2
StorageNode3
StorageNode4
StorageNode5
```

Instead of:

```text
Entire File → Node1
```

chunks are distributed.

---

## Example

```text
Chunk1 → Node1
Chunk2 → Node3
Chunk3 → Node5
Chunk4 → Node2
Chunk5 → Node4
```

Architecture:

```text
video.mp4

Chunk1 ──► Node1
Chunk2 ──► Node3
Chunk3 ──► Node5
Chunk4 ──► Node2
Chunk5 ──► Node4
```

This is one reason object storage scales horizontally.

---

# Why Spread Chunks Across Nodes?

Suppose Netflix movie becomes popular.

Millions of users download it.

Without distribution:

```text
Node1 serves everything
```

Node1 becomes bottleneck.

---

With chunk distribution:

```text
Chunk1 → Node1
Chunk2 → Node2
Chunk3 → Node3
Chunk4 → Node4
```

Load spreads across cluster.

---

# 6. Replication Happens Per Chunk

This is one of the most important concepts.

A beginner often imagines:

```text
Entire File
  ├── Copy1
  ├── Copy2
  └── Copy3
```

At large scale, storage systems typically protect data at the chunk level.

---

## Example

Suppose:

```text
video.mp4 = 100 MB
```

Chunk size:

```text
10 MB
```

File becomes:

```text
Chunk1
Chunk2
Chunk3
...
Chunk10
```

---

Replication Factor:

```text
3
```

---

Instead of:

```text
Entire Video → Node1
Entire Video → Node2
Entire Video → Node3
```

Storage looks more like:

```text
Chunk1
 ├── Node1
 ├── Node7
 └── Node12

Chunk2
 ├── Node3
 ├── Node8
 └── Node15

Chunk3
 ├── Node2
 ├── Node9
 └── Node11
```

Much closer to reality.

---

# Why Replicate Each Chunk?

## Fault Tolerance

Suppose:

```text
Node7 dies
```

Chunk1 still exists on:

```text
Node1
Node12
```

No data loss.

---

## Better Load Distribution

Downloads can be served from:

```text
Node1
Node7
Node12
```

instead of a single machine.

---

## Faster Recovery

If:

```text
Node7 dies
```

Only affected chunk replica must be recreated.

Not the entire file.

---

# Visual Architecture

```text
Object: video.mp4

          Metadata
               │
               ▼

 ┌─────────┬─────────┬─────────┐
 ▼         ▼         ▼

Chunk1   Chunk2   Chunk3

 │ │ │     │ │ │     │ │ │
 ▼ ▼ ▼     ▼ ▼ ▼     ▼ ▼ ▼

N1 N7 N12 N3 N8 N15 N2 N9 N11
```

---

# 7. What Does Metadata Store?

Metadata might look like:

```json
{
  "object": "video.mp4",
  "chunks": [
    {
      "chunk": 1,
      "replicas": ["Node1","Node7","Node12"]
    },
    {
      "chunk": 2,
      "replicas": ["Node3","Node8","Node15"]
    }
  ]
}
```

Metadata stores:

* Chunk ordering
* Chunk locations
* Replica locations

Not actual file contents.

---

# 8. Are Chunks Stored Sequentially?

This is an excellent interview question.

Answer:

> Logically yes.
>
> Physically usually no.

---

# Logical Ordering

Metadata preserves sequence.

Example:

```json
{
  "chunks": [
    "chunk1",
    "chunk2",
    "chunk3",
    "chunk4"
  ]
}
```

Storage system knows:

```text
Chunk1
then Chunk2
then Chunk3
then Chunk4
```

---

# Physical Storage

Actual locations may be:

```text
Chunk1 → Node8
Chunk2 → Node2
Chunk3 → Node17
Chunk4 → Node4
```

Completely distributed.

Storage system doesn't care because metadata maintains ordering.

---

# Library Analogy

Imagine book pages stored on different shelves.

```text
Page1 → ShelfA
Page2 → ShelfZ
Page3 → ShelfC
Page4 → ShelfQ
```

Book can still be reconstructed because page numbers preserve order.

Metadata serves the same purpose.

---

# 9. Download Flow

Suppose user requests:

```text
video.mp4
```

---

## Step 1

Metadata service queried.

Returns:

```text
Chunk1 → Node8
Chunk2 → Node2
Chunk3 → Node17
Chunk4 → Node4
```

---

## Step 2

Chunks fetched.

Usually:

```text
IN PARALLEL
```

instead of:

```text
Chunk1
then Chunk2
then Chunk3
```

---

Storage system performs:

```text
Fetch Chunk1
Fetch Chunk2
Fetch Chunk3
Fetch Chunk4
```

simultaneously.

---

## Step 3

Chunks reassembled:

```text
Chunk1 + Chunk2 + Chunk3 + Chunk4
```

---

## Step 4

Original file returned.

---

# 10. Multipart Uploads (S3 Example)

Suppose:

```text
50 GB file
```

Client uploads:

```text
Part1
Part2
Part3
...
Part100
```

Each part uploaded independently.

---

After upload completes:

Metadata created:

```json
{
  "parts": [
    1,
    2,
    3,
    ...
    100
  ]
}
```

Now S3 treats all parts as one object.

---

# 11. Does Client Know About Chunking?

Sometimes yes.

---

## Multipart Upload

Client explicitly uploads:

```text
Part1
Part2
Part3
```

Client is aware of chunks.

---

## Internal Chunking

Sometimes client uploads:

```text
video.mp4
```

and storage system chunks internally.

Client never sees chunking.

---

# 12. Interview-Level Architecture

```text
Object
   │
   ▼

Metadata

   │
   ▼

Chunk List

   │
   ▼

Distributed Storage Nodes
```

Where:

* Object split into chunks
* Metadata tracks ordering
* Chunks distributed across nodes
* Each chunk replicated
* Chunks fetched in parallel
* Object reconstructed on read

---

# 13. Modern Object Storage Protection

Large systems such as:

* Amazon S3
* Google Cloud Storage
* Azure Blob Storage

often protect data at chunk level.

Data may be:

```text
Chunk
   ↓
Replication
```

or

```text
Chunk
   ↓
Erasure Coding
```

depending on implementation.

---

# 14. Senior Engineer Mental Model

The most important understanding is:

Object storage does NOT store large files as one giant blob on one server.

Instead:

```text
Object
   │
   ▼

Chunked
   │
   ▼

Distributed
   │
   ▼

Replicated / Erasure Coded
   │
   ▼

Tracked by Metadata
   │
   ▼

Reassembled During Reads
```

This architecture is one of the key reasons systems like Amazon S3 can scale to trillions of objects and exabytes of storage.
