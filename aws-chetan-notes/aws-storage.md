# AWS Storage and Database Services

## Amazon S3 (Simple Storage Service)

### What is S3?

S3 is an object storage service that provides scalability, data availability, security, and performance. It stores data as objects (files with metadata) accessed via HTTP API or URL.

### How S3 Works

Each object has a unique key within a bucket. Objects are accessed through HTTP endpoints or AWS APIs. S3 provides unlimited storage capacity.

---

### S3 Storage Classes

| Class | Use Case | Retrieval Time | Cost Level |
|-------|----------|----------------|------------|
| **S3 Standard** | Frequently accessed data | Instant | Higher storage cost |
| **S3 Intelligent-Tiering** | Unknown or changing access patterns | Instant | Automatic cost optimization |
| **S3 Standard-IA** | Infrequently accessed data | Instant | Lower storage, retrieval fee |
| **S3 One Zone-IA** | Infrequent access, non-critical | Instant | Lowest IA cost |
| **S3 Glacier Instant Retrieval** | Archive with instant access | Instant | Very low storage cost |
| **S3 Glacier Flexible Retrieval** | Archive data | Minutes to hours | Lower cost |
| **S3 Glacier Deep Archive** | Long-term archive | 12 hours | Lowest cost |

---

### Key Features

- **Versioning**: Protects data from accidental deletion or overwrites
- **Lifecycle Policies**: Automatically transitions objects between storage classes
- **Encryption**: Server-side encryption for security
- **Replication**: Cross-region replication for disaster recovery
- **Event Notifications**: Triggers workflows based on object changes

### Common Use Cases

- Backup and archival storage
- Media storage and content distribution
- Data lakes for analytics
- Static website hosting
- Application logs and audit trails

---

## Amazon EBS (Elastic Block Store)

### What is EBS?

EBS is block-level storage for EC2 instances. It behaves like a physical disk that can be attached to EC2 instances.

### Key Characteristics

- **Low latency**: Sub-millisecond response times
- **Persistent storage**: Survives instance stop/termination
- **Single attachment**: Attached to one EC2 instance at a time (except io2 multi-attach)
- **Snapshots**: Point-in-time backups stored in S3
- **Availability Zone scoped**: Must be in same AZ as EC2 instance

### Performance Factors

| Metric | Performance |
|--------|-------------|
| **IOPS** | High (up to 64,000+ provisioned IOPS) |
| **Throughput** | Consistent and high (up to 4,000 MB/s) |
| **Latency** | Very low (sub-millisecond) |

### Common Use Cases

- Database storage (RDS, NoSQL)
- Boot volumes for EC2 instances
- Applications requiring consistent low-latency performance
- Data warehouses
- Enterprise applications (SAP, Oracle ERP)

---

## Amazon EFS (Elastic File System)

### What is EFS?

EFS is a managed NFS file system that can be shared across multiple EC2 instances simultaneously.

### Key Characteristics

- **Automatic scaling**: Grows and shrinks automatically
- **NFS protocol**: Standard file system mount
- **Multi-AZ**: Regional service with high availability
- **Shared access**: Multiple EC2 instances can access simultaneously
- **On-premises support**: Can be mounted via VPN or Direct Connect

### Performance

| Metric | Performance |
|--------|-------------|
| **Throughput** | Scales with storage size |
| **Latency** | Low (milliseconds, higher than EBS) |
| **IOPS** | Medium (shared across clients) |

### Common Use Cases

- Shared file storage for microservices
- Content management systems
- Big data and analytics workloads
- Container persistent storage
- Web serving and development environments

---

## Amazon FSx

### What is FSx?

FSx provides fully managed third-party file systems optimized for specific workloads.

### FSx Types

| Type | Protocol | Best For |
|------|----------|----------|
| **FSx for Windows File Server** | SMB | Windows applications, enterprise workloads |
| **FSx for Lustre** | Lustre | High-performance computing (HPC), ML training |

### Common Use Cases

- Windows enterprise application migrations
- High-performance computing workloads
- Machine learning training jobs
- Media processing workflows

---

## Storage Comparison

### Feature Comparison

| Feature | S3 | EBS | EFS | FSx |
|---------|-----|-----|-----|-----|
| **Type** | Object storage | Block storage | File storage | Managed file storage |
| **Access Method** | HTTP API/URL | Attached to EC2 | NFS mount | SMB or Lustre |
| **Sharing** | Unlimited | Single EC2 | Multiple EC2 | Multiple instances |
| **Scope** | Regional | Availability Zone | Regional | AZ or Regional |
| **Latency** | Higher (tens of ms) | Very low (sub-ms) | Low (ms) | Low |
| **Persistence** | Durable by default | Independent of EC2 | Independent | Independent |
| **Scalability** | Unlimited | Fixed size (resizable) | Auto-scaling | Configured capacity |

---

### Performance Comparison

| Metric | EBS | EFS/FSx | S3 | Glacier |
|--------|-----|---------|-----|---------|
| **IOPS** | Very high (64k+) | Medium | Low | Very low |
| **Throughput** | High and consistent | Scales with demand | Good for sequential | Not performance-focused |
| **Latency** | Sub-millisecond | Milliseconds | Tens of milliseconds | Minutes to hours |

---

### Cost Comparison

| Service | Pricing Model | Approximate Cost |
|---------|---------------|------------------|
| **EBS** | Provisioned capacity | ~$0.10/GB/month |
| **EFS** | Pay-as-you-go | ~$0.30/GB/month |
| **S3 Standard** | Pay-as-you-go | ~$0.023/GB/month + requests |
| **S3 Glacier** | Pay-as-you-go | ~$0.004/GB/month |

---

## Storage Selection Guide

### Decision Framework

**Step 1: Who needs access?**

- EC2 only → EBS, EFS, or FSx
- From anywhere (global access) → S3

**Step 2: What is the access pattern?**

- Write heavy / Read heavy → Block storage (EBS)
- Write once / Read heavy → File storage (EFS/FSx) or S3

**Step 3: Is persistence needed?**

- No persistence → Instance Store (temporary)
- Persistence required → EBS, EFS, S3

**Step 4: Does application support S3?**

- Yes → Use S3 directly
- No → Check operating system requirements

**Step 5: Which operating system?**

- Linux → EFS
- Windows → FSx for Windows File Server

---

### Use Case Mapping

| Requirement | Best Service |
|-------------|--------------|
| Operating system boot volumes | **EBS** |
| Database storage | **EBS** |
| Shared file storage (Linux) | **EFS** |
| Shared file storage (Windows) | **FSx** |
| Distributed computing | **EFS** or **FSx** |
| Media storage (images/videos) | **S3** |
| Static website hosting | **S3** |
| Long-term archival | **S3 Glacier** |
| Container persistent storage | **EFS** |
| Data lake storage | **S3** |
| Low latency data access | **EBS** |

---

## Amazon RDS (Relational Database Service)

### What is RDS?

RDS is a managed relational database service. AWS handles provisioning, patching, backup, scaling, and high availability automatically.

### Supported Database Engines

| Engine | Description |
|--------|-------------|
| **Amazon Aurora** | MySQL and PostgreSQL compatible, high performance |
| **MySQL** | Popular open source relational database |
| **PostgreSQL** | Advanced open source database with rich features |
| **MariaDB** | MySQL fork with additional enhancements |
| **Oracle** | Enterprise-grade commercial database |
| **Microsoft SQL Server** | Microsoft's enterprise database solution |

### Deployment Options

- **RDS Standard**: Fully managed by AWS
- **RDS Custom**: You manage OS and database configuration for more control

### Common Use Cases

- Web and mobile applications
- Reporting and analytics workloads
- Enterprise transactional systems
- E-commerce platforms
- Content management systems

---

## Amazon Aurora

### What is Aurora?

Aurora is AWS's high-performance relational database engine compatible with MySQL and PostgreSQL.

### Key Features

- **Performance**: 5x faster than standard MySQL, 3x faster than standard PostgreSQL
- **Storage**: Auto-scaling up to 128 TB
- **Read Replicas**: Up to 15 read replicas
- **High Availability**: Multi-AZ deployment with automatic failover
- **Backup**: Automatic continuous backup with point-in-time recovery

### When to Use Aurora

- Need enterprise-grade relational database performance
- Require high availability and automatic failover
- Want MySQL or PostgreSQL compatibility with better performance
- Have demanding read-heavy workloads
- Need to scale without downtime

---

## Amazon DynamoDB

### What is DynamoDB?

DynamoDB is a fully managed NoSQL database service providing key-value and document data structures.

### Key Features

- **Performance**: Single-digit millisecond latency at any scale
- **Scalability**: Automatic horizontal scaling
- **Global Tables**: Multi-region replication
- **Streams**: Event-driven architecture support
- **Security**: Built-in encryption and access control
- **Backup**: Automatic backup and point-in-time recovery

### Common Use Cases

- Global applications requiring low latency
- IoT applications with high write throughput
- Gaming leaderboards and session management
- Real-time bidding and ad tech
- Mobile and web applications with unpredictable traffic
- Shopping carts and user profiles

---

## Database Selection Guide

### RDS/Aurora vs DynamoDB

| Factor | RDS/Aurora | DynamoDB |
|--------|------------|----------|
| **Data Model** | Relational (tables, rows, columns) | NoSQL (key-value, document) |
| **Query Type** | Complex SQL queries with joins | Simple key-based queries |
| **Scalability** | Vertical scaling, read replicas | Automatic horizontal scaling |
| **Latency** | Low (milliseconds) | Very low (single-digit ms) |
| **Schema** | Fixed schema | Flexible schema |
| **Cost Model** | Based on instance size | Based on throughput and storage |
| **Management** | Some management needed | Fully serverless |

---

### When to Use Each Database

| Scenario | Recommended Database |
|----------|---------------------|
| Complex queries with joins | **RDS** or **Aurora** |
| ACID transactions required | **RDS** or **Aurora** |
| Existing SQL applications | **RDS** or **Aurora** |
| High-scale key-value access | **DynamoDB** |
| Unpredictable traffic patterns | **DynamoDB** |
| Global multi-region apps | **DynamoDB Global Tables** |
| Real-time applications | **DynamoDB** |
| Traditional enterprise apps | **RDS** or **Aurora** |

---

## Storage and Database Best Practices

### Storage Best Practices

- Use **S3 Lifecycle Policies** to automatically move data to cheaper storage classes
- Enable **versioning** on S3 buckets for critical data
- Take regular **EBS snapshots** for backup and disaster recovery
- Use **EFS** when multiple EC2 instances need shared access
- Consider **S3 Intelligent-Tiering** when access patterns are unknown
- Use **encryption** for sensitive data at rest and in transit

### Database Best Practices

- Enable **automated backups** for RDS and Aurora
- Use **Multi-AZ deployments** for production databases
- Configure **read replicas** to offload read traffic
- Monitor **database performance metrics** regularly
- Use **DynamoDB auto-scaling** for variable workloads
- Implement proper **indexing strategies** for query performance