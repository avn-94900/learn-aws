# AWS Storage Services

## Amazon S3 (Simple Storage Service)

Amazon S3 is an **object storage service** offering high scalability, durability (11 nines), and security. Data is stored as **objects** (file + metadata + unique key) inside **buckets**, accessible via HTTP/HTTPS, AWS APIs, or SDKs.

### Key Features

| Feature | Description |
|---------|-------------|
| **Versioning** | Protects against accidental deletion or overwrites |
| **Lifecycle Policies** | Auto-transitions objects between storage classes |
| **Encryption** | Server-side encryption at rest |
| **Replication** | Cross-region replication for disaster recovery |
| **Event Notifications** | Triggers workflows on object changes |
| **Durability** | 99.999999999% across multiple devices and AZs |

### Storage Classes

| Storage Class | Use Case | Retrieval Time | Cost |
|---------------|----------|----------------|------|
| **Standard** | Frequently accessed data | Instant | High |
| **Intelligent-Tiering** | Unknown/changing access patterns | Instant | Auto-optimized |
| **Standard-IA** | Infrequent access, fast retrieval needed | Instant | Lower + retrieval fee |
| **One Zone-IA** | Non-critical, re-creatable data (single AZ) | Instant | Lower (no multi-AZ) |
| **Glacier Instant Retrieval** | Archive with immediate access | Instant | Very low |
| **Glacier Flexible Retrieval** | Long-term archival | Minutes to hours | Low |
| **Glacier Deep Archive** | Rarely accessed (1–2×/year) | ~12 hours | Lowest |

> **One Zone-IA warning:** Data may be lost if the AZ fails — avoid for critical data.

### Common Use Cases

- Backup, archival, and data lakes
- Static website hosting and media distribution
- Application logs and audit trails

### AWS Service Integrations

| Service | Integration |
|---------|-------------|
| **EC2** | EBS snapshots stored in S3 |
| **CloudFront** | S3 as CDN origin |
| **Lambda** | Event triggers on object upload |
| **Athena / Glue** | Query and catalog data in S3 |

### Pricing

| Factor | Details |
|--------|---------|
| Storage | Per GB/month by storage class |
| Requests | PUT, GET, COPY operations |
| Data Transfer | Outbound transfer charges |
| **Free Tier** | 5 GB Standard, 20K GET, 2K PUT/month (12 months) |

---

## Amazon EBS (Elastic Block Store)

EBS is **block storage** for EC2 instances — behaves like a virtual hard disk. Scoped to a single **Availability Zone** with sub-millisecond latency and data persistence beyond instance stop/start.

### Key Characteristics

| Feature | Details |
|---------|---------|
| **Storage Type** | Block |
| **Attachment** | Single EC2 instance |
| **Latency** | Sub-millisecond |
| **IOPS** | Up to 256,000 (io2 Block Express) / 64,000 (io1/io2) |
| **Throughput** | Up to 4,000 MB/s (io2 Block Express) / 1,000 MB/s (io1) |
| **Scope** | Availability Zone |

### Snapshots

Point-in-time backups stored in S3 — used for backup and disaster recovery.

### Common Use Cases

- EC2 boot volumes
- Database storage (RDS, NoSQL)
- Enterprise apps (SAP, Oracle ERP)
- Low-latency transactional workloads

---

## Amazon EFS (Elastic File System)

EFS is a **fully managed, shared file system** for Linux-based workloads. Multiple EC2 instances access it simultaneously via **NFS**, and capacity scales automatically.

### Key Features

| Feature | Details |
|---------|---------|
| **Protocol** | NFS |
| **Scalability** | Auto grows/shrinks |
| **Availability** | Regional (multi-AZ) |
| **Latency** | Milliseconds |
| **IOPS** | Medium (shared across clients) |

### Common Use Cases

- Shared storage for microservices and containers (EKS, Fargate)
- Content management and web serving
- Big data and analytics workloads

---

## Amazon FSx

FSx provides **fully managed third-party file systems** for specialized workloads.

| Type | Protocol | Best For |
|------|----------|----------|
| **FSx for Windows File Server** | SMB | Windows enterprise apps, AD integration |
| **FSx for Lustre** | Lustre | HPC, ML training, media processing |

---

## Storage Comparison

### Feature & Performance

| Feature | S3 | EBS | EFS | FSx |
|---------|----|-----|-----|-----|
| **Type** | Object | Block | File | File |
| **Access** | HTTP/API | EC2 attach | NFS | SMB/Lustre |
| **Multi-instance** | Yes | No | Yes | Yes |
| **Scope** | Regional | AZ | Regional | AZ or Regional |
| **Scalability** | Unlimited | Fixed (resizable) | Auto | Configured |
| **Latency** | 10–100 ms | Sub-ms | ms | Low |
| **IOPS** | Low | Very high (64k+) | Medium | High |

### Cost (Approximate)

| Service | Model | Cost |
|---------|-------|------|
| **EBS** | Provisioned | ~$0.10/GB/month |
| **EFS** | Pay-as-you-go | ~$0.30/GB/month |
| **S3 Standard** | Pay-as-you-go | ~$0.023/GB/month |
| **S3 Glacier** | Pay-as-you-go | ~$0.004/GB/month |

---

## Storage Selection Guide

| Requirement | Best Service |
|-------------|-------------|
| EC2 boot volumes | **EBS** |
| Database / low-latency storage | **EBS** |
| Shared file storage (Linux) | **EFS** |
| Shared file storage (Windows) | **FSx for Windows** |
| HPC / ML training | **FSx for Lustre** |
| Media, backups, data lakes | **S3** |
| Static website hosting | **S3** |
| Long-term archival | **S3 Glacier / Deep Archive** |
| Container persistent storage | **EFS** |
| Global / internet access | **S3** |

**Decision checklist:**
1. Single EC2 + low latency → **EBS**
2. Multiple EC2 + Linux shared → **EFS**
3. Multiple EC2 + Windows shared → **FSx**
4. Object/unstructured data or global access → **S3**
5. Rarely accessed archives → **S3 Glacier**
