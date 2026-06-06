# Object Storage — System Design Notes

# 1. Why Do We Need Object Storage?

Suppose users upload:

* Profile Pictures
* Videos
* PDFs
* Resumes
* Documents
* Backups
* Logs

A beginner might think:

```sql
Users
---------
id
name
photo
```

and store images directly in a database.

This works initially.

However, imagine:

```text
100 Million Users
```

Each uploads:

```text
5 MB Image
```

Storage becomes:

```text
500 TB+
```

Problems:

* Database becomes huge
* Expensive storage
* Slow backups
* Slow replication
* Poor performance

Databases are optimized for:

```text
Structured Data
```

not massive files.

This is why object storage exists.

---

# 2. What is Object Storage?

Object storage stores:

```text
Objects
```

instead of:

* Rows (SQL)
* Documents (MongoDB)
* Blocks (Disk Storage)

Each object contains:

```text
Data
Metadata
Unique Identifier
```

---

# Example Object

Suppose user uploads:

```text
vacation.jpg
```

Object might look like:

```json
{
  "object_id": "img_123",
  "data": "<binary image bytes>",
  "size": "5MB",
  "uploaded_by": "user42",
  "created_at": "2026-06-04"
}
```

Everything is stored together.

---

# 3. Core Mental Model

Think of object storage as:

```text
Key → Object
```

Example:

```text
photos/user42/vacation.jpg
          │
          ▼
        Object
```

This is how Amazon S3 fundamentally works.

---

# 4. Object Storage vs Filesystem

## Traditional Filesystem

```text
/home/user/docs/file.pdf
```

Uses:

* Directories
* Folders
* Hierarchical Trees

---

## Object Storage

Uses:

```text
Unique Key
    │
    ▼
 Object
```

No real directory traversal.

Folders are often just naming conventions.

---

# Why This Scales Better

Instead of managing:

```text
Folder
 ├── File1
 ├── File2
 ├── File3
```

Object storage manages:

```text
Key → Object
```

which can be distributed across thousands of machines.

---

# 5. Immutable Objects (Very Important)

One of the most important object storage concepts.

---

# What Does Immutable Mean?

Once object is written:

```text
Object Created
```

it cannot be modified in-place.

---

Instead:

```text
Delete Old Object
Create New Object
```

or

```text
Create New Version
```

---

# Example

Suppose:

```text
profile.jpg
```

already exists.

User uploads a new photo.

Instead of modifying bytes inside existing object:

```text
Create New Object
```

or

```text
Create New Version
```

---

# Why Immutability Matters

Distributed systems love immutable data.

---

## Benefit 1 — Easier Replication

Suppose object replicated to:

```text
Server A
Server B
Server C
```

If object never changes:

```text
No Synchronization Required
```

Huge simplification.

---

## Benefit 2 — Better Caching

CDNs love immutable objects.

Example:

```text
profile_v1.jpg
```

can be cached forever.

No invalidation issues.

---

## Benefit 3 — No Concurrent Update Conflicts

Avoids problems like:

```text
Who Updated Last?
```

No write-write conflicts.

---

# Interview Insight

One reason S3 scales to trillions of objects is:

> Objects are largely immutable.

---

# 6. Chunking

Suppose user uploads:

```text
100 GB Video
```

Can we store as one giant file?

Possible.

Bad idea.

---

# Problem

Suppose upload fails at:

```text
99 GB Uploaded
```

Need to restart entire upload.

Very inefficient.

---

# Solution — Chunking

Split large file into smaller pieces.

Example:

```text
100 GB Video

Chunk1
Chunk2
Chunk3
...
Chunk1000
```

---

# Example

Suppose:

```text
5 GB Video
```

Split into:

```text
5 MB Chunks
```

Now:

```text
Chunk1
Chunk2
Chunk3
...
Chunk1000
```

stored independently.

---

# Benefits of Chunking

## Resume Uploads

If upload fails:

```text
Upload Missing Chunks Only
```

instead of entire file.

---

## Parallel Uploads

Can upload:

```text
Chunk1 → Server A
Chunk2 → Server B
Chunk3 → Server C
```

simultaneously.

Much faster.

---

## Parallel Downloads

Multiple chunks downloaded at same time.

Improves throughput.

---

# Real World Example

Netflix does not stream:

```text
2 GB Movie
```

as one file.

Instead:

```text
Thousands of Chunks
```

are downloaded progressively.

---

# 7. Replication

Suppose storage server dies.

Without replication:

```text
Data Lost
```

Unacceptable.

---

# Solution

Replicate objects.

---

# Architecture

```text
Object
   │
   ├── Replica A
   ├── Replica B
   └── Replica C
```

Stored on different machines.

---

# Failure Scenario

Suppose:

```text
Replica A Dies
```

Still have:

```text
Replica B
Replica C
```

No data loss.

---

# 8. Multi-AZ Replication

Cloud providers commonly replicate across:

```text
Availability Zone 1
Availability Zone 2
Availability Zone 3
```

Architecture:

```text
Object

 ├── AZ1
 ├── AZ2
 └── AZ3
```

---

# Why Important?

Suppose:

```text
Entire Datacenter Fails
```

Object still exists elsewhere.

---

# 9. Durability vs Availability

Common interview question.

---

# Availability

Means:

```text
Can I Access Object Right Now?
```

---

# Durability

Means:

```text
Will Object Survive Long-Term?
```

---

# Example

```text
99.99% Availability
```

means occasional downtime possible.

---

```text
99.999999999% Durability
```

means object loss is extremely unlikely.

---

# Why S3 Durability Is So High

Because of:

* Replication
* Checksums
* Background Repair
* Multi-AZ Storage

---

# 10. Amazon S3 Architecture (Simplified)

Suppose user uploads:

```text
vacation.jpg
```

---

## Step 1

Object assigned key:

```text
photos/user42/vacation.jpg
```

---

## Step 2

Object chunked internally.

---

## Step 3

Object replicated across storage nodes.

---

## Step 4

Metadata stored separately.

---

# Architecture

```text
Client
   │
   ▼

S3 Frontend

   │
   ▼

Metadata Service

   │
   ▼

Storage Nodes

 ├── Replica A
 ├── Replica B
 └── Replica C
```

---

# 11. Blob Storage

Azure's object storage service is:

Azure Blob Storage

Blob means:

```text
Binary Large Object
```

Examples:

* Images
* Videos
* PDFs
* Backups
* Documents

---

# Core Concept

Blob Storage and S3 are fundamentally:

```text
Object Storage Systems
```

with different cloud providers.

---

# Terminology Mapping

| AWS    | Azure        |
| ------ | ------------ |
| S3     | Blob Storage |
| Bucket | Container    |
| Object | Blob         |

Underlying concepts are nearly identical.

---

# 12. Real World Design Example

Suppose building Instagram.

---

## Bad Design

```text
App
 │
 ▼

MySQL

 │
 ▼

Store Image
```

Database becomes enormous.

---

## Good Design

```text
User
 │
 ▼

Upload Service

 │
 ▼

Object Storage (S3)

 │
 ▼

Store URL in Database
```

Database contains:

```sql
users
------
id
name
photo_url
```

Image itself stored in object storage.

---

# Why This Is Better

Database stores:

```text
Metadata
```

Object Storage stores:

```text
Large Binary Files
```

Each system does what it is optimized for.

---

# 13. When to Use Object Storage

Use object storage for:

* Images
* Videos
* Audio Files
* PDFs
* Documents
* Backups
* Data Lakes
* Logs
* ML Datasets

---

# Do NOT Use Object Storage For

* Transactions
* User Records
* Relational Queries
* Frequent Updates
* Complex Joins

Use databases instead.

---

# 14. Interview Takeaway

When interviewer asks:

> Where would you store user uploaded files?

Expected answer:

```text
Object Storage
```

Examples:

* Amazon S3
* Azure Blob Storage
* Google Cloud Storage

because object storage provides:

* Massive Scalability
* High Durability
* Replication
* Chunked Uploads
* CDN Integration
* Low Cost Storage

---

# 15. Senior Engineer Mental Model

Object storage fundamentally works because:

```text
Immutable Objects
        +
Chunking
        +
Replication
        +
Distributed Metadata
```

allow systems like Amazon S3 and Azure Blob Storage to store trillions of objects while providing extremely high durability and availability at global scale.
