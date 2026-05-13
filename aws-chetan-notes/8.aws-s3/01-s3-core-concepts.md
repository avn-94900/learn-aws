# 01 - AWS S3 Core Concepts

Amazon Simple Storage Service (S3) is a highly scalable, durable, and secure object storage service that lets you store and retrieve any amount of data from anywhere on the web.

**Key Characteristics**

- Fully managed service — no infrastructure management required
- Object-based storage model (store files as objects inside buckets)
- Global accessibility with regional data residency
- 99.999999999% (11 nines) durability for data protection
- Virtually unlimited storage capacity

---

## S3 Bucket

A bucket is a top-level container for storing objects in S3.

**Bucket Properties**

- Globally unique names across all AWS accounts and regions
- Created in a specific AWS region but accessible globally
- Can contain an unlimited number of objects
- Organizes and manages access to data

**Bucket Naming Rules**

- Must use DNS-compliant names with lowercase letters
- Length between 3 and 63 characters
- Cannot contain uppercase letters or underscores
- Must start with a lowercase letter or number
- Cannot be formatted as IP addresses (e.g., `192.168.1.1`)

---

## S3 Object

Objects are the fundamental entities stored in Amazon S3, consisting of data and metadata.

**Object Components**

| Component    | Description                                               |
|--------------|-----------------------------------------------------------|
| Object Data  | The actual file content (up to 5 TB)                      |
| Object Key   | Unique identifier / name within the bucket                |
| Version ID   | Unique identifier for object versions (if versioning enabled) |
| Metadata     | System and user-defined key-value pairs                   |
| ETag         | Entity tag for object integrity verification              |
| Storage Class| Current storage class of the object                      |
| Timestamps   | Last modified date and creation time                      |

**Object Size Limits**

| Operation          | Limit  |
|--------------------|--------|
| Single object max  | 5 TB   |
| Single PUT upload  | 5 GB   |
| Multipart required | > 5 GB |
| Multipart recommended | > 100 MB |

---

## Object Key

The object key is the unique identifier for an object within a bucket — similar to a file path.

**Key Properties**

- UTF-8 encoded string, up to 1,024 bytes long
- Must be unique within the bucket
- Can include prefixes and delimiters to simulate a folder structure
- Case-sensitive

**Example Keys**

```
invoice-12345.pdf
documents/2024/invoices/invoice-12345.pdf
project-alpha/reports/quarterly-report.pdf
```

**Key Best Practices**

- Avoid sequential prefixes (e.g., `0001-`, `0002-`) for better performance at high request rates
- Use logical prefixes to organize objects (e.g., by date, department, type)
- Avoid special characters that may require URL encoding
- Distribute load across varied prefixes to prevent hotspotting

---

## Object Lifecycle States

S3 objects progress through different states during their lifetime:

| State       | Description                                          |
|-------------|------------------------------------------------------|
| Created     | Object successfully uploaded to S3                   |
| Accessible  | Object available for read/write operations           |
| Transitioned| Object moved to a different storage class via lifecycle rule |
| Archived    | Object stored in Glacier or Deep Archive             |
| Expired     | Object marked for deletion by a lifecycle rule       |
| Deleted     | Object permanently removed from S3                  |

---

## Storage Classes

S3 offers multiple storage classes optimized for different use cases and cost requirements.

| Storage Class               | Use Case                              | Retrieval Time   | Availability |
|-----------------------------|---------------------------------------|------------------|--------------|
| S3 Standard                 | Frequently accessed data              | Instant          | 99.99%       |
| S3 Intelligent-Tiering      | Unknown or changing access patterns   | Instant          | 99.9%        |
| S3 Standard-IA              | Infrequent access, instant retrieval  | Instant          | 99.9%        |
| S3 One Zone-IA              | Non-critical, infrequent access       | Instant          | 99.5%        |
| S3 Glacier Instant Retrieval| Archive with instant access           | Instant          | 99.9%        |
| S3 Glacier Flexible Retrieval| Archive with flexible retrieval      | Minutes to hours | 99.99%       |
| S3 Glacier Deep Archive     | Long-term retention, lowest cost      | 12+ hours        | 99.99%       |

All storage classes provide **11 nines (99.999999999%) durability**.

**Choosing the Right Storage Class**

- **Standard** — Active data, websites, content distribution
- **Intelligent-Tiering** — Unpredictable access patterns; AWS auto-optimizes cost
- **Standard-IA** — Infrequently accessed but requires instant retrieval
- **One Zone-IA** — Non-critical data that can be recreated if lost
- **Glacier Instant Retrieval** — Archived data needing immediate access occasionally
- **Glacier Flexible Retrieval** — Archives where minutes-to-hours delay is acceptable
- **Glacier Deep Archive** — Compliance archives, long-term retention (years)

---

## Data Consistency Model

Since December 2020, S3 provides **strong read-after-write consistency** for all operations.

| Consistency Type  | Behaviour                                                  |
|-------------------|------------------------------------------------------------|
| Read-after-Write  | New objects are immediately visible after upload           |
| List Consistency  | Bucket listings reflect recent changes immediately         |
| Update Consistency| Overwrites and deletes are strongly consistent             |

> **Historical note:** Before December 2020, S3 used eventual consistency for overwrites and deletes. This no longer applies.

---

## Versioning

S3 versioning allows you to keep multiple variants of an object in the same bucket.

**Versioning States**

| State       | Behaviour                                              |
|-------------|--------------------------------------------------------|
| Unversioned | Default — no version IDs assigned                     |
| Enabled     | All new and modified objects receive unique version IDs|
| Suspended   | Stops creating new versions; retains existing ones     |

> Once versioning is enabled on a bucket, it can only be **suspended**, not fully disabled.

**Key Features**

- Unique version ID assigned to each object version
- Latest version retrieved by default (unless version ID is specified)
- Prevents accidental deletion or modification
- Supports restoring previous versions
- MFA Delete can be enabled to prevent accidental permanent deletion

**Use Cases**

- Protect against accidental deletions
- Recover from application failures
- Archive and audit data changes
- Meet compliance requirements

---

## Lifecycle Management

Lifecycle rules automate the transition and expiration of objects based on defined criteria.

**Lifecycle Actions**

| Action                            | Description                                      |
|-----------------------------------|--------------------------------------------------|
| Transition                        | Move objects between storage classes             |
| Expiration                        | Automatically delete objects after specified time|
| Abort Incomplete Multipart Uploads| Clean up stale incomplete uploads                |
| Noncurrent Version Expiration     | Delete old versions of versioned objects         |

**Lifecycle Rule Components**

- Rule name and status (enabled / disabled)
- Scope: entire bucket, key prefix, or object tags
- Actions: transitions and expirations
- Timeline: number of days after object creation or after last transition

**Cost Optimization Example**

```
Day 0–30    →  S3 Standard
Day 30–90   →  S3 Standard-IA  (transition after 30 days)
Day 90–365  →  S3 Glacier      (transition after 90 days)
Day 365+    →  Delete          (expire after 365 days)
```
