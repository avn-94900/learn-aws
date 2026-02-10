# AWS Storage and Database Services

## Amazon S3

**What is S3?**

S3 is an object storage service offering industry-leading scalability, data availability, security, and performance.

**How S3 Stores Data**

S3 stores data as objects (file and metadata) accessed via HTTP API or URL. Each object has a unique key within a bucket.

**S3 Storage Classes**

| Class | Use Case | Retrieval Time | Cost |
|-------|----------|----------------|------|
| Standard | Frequently accessed data | Instant | Higher storage cost |
| Intelligent-Tiering | Unknown or changing access patterns | Instant | Automatic cost optimization |
| Standard-IA | Infrequently accessed data | Instant | Lower storage, retrieval fee |
| One Zone-IA | Infrequent access, non-critical | Instant | Lowest IA cost |
| Glacier Instant Retrieval | Archive with instant access | Instant | Very low storage cost |
| Glacier Flexible Retrieval | Archive data | Minutes to hours | Lower cost |
| Glacier Deep Archive | Long-term archive | 12 hours | Lowest cost |

**S3 Features**

- Versioning for data protection
- Lifecycle policies for automatic transitions between storage classes
- Server-side encryption for security
- Cross-region replication for disaster recovery
- Event notifications to trigger workflows

**Common Use Cases**

- Backups and archives
- Media storage and distribution
- Data lakes for analytics
- Static website hosting
- Application logs storage

---

## Amazon EBS

**What is EBS?**

EBS (Elastic Block Store) is block storage for EC2 instances. It behaves like a physical disk that can be attached to EC2.

**Characteristics**

- Low latency and high performance
- Persistent storage that survives instance termination
- Can be attached to only one EC2 instance at a time (except io2 multi-attach)
- Supports snapshots for backup

**Common Use Cases**

- Database storage
- Operating system disks
- Applications requiring consistent low-latency performance
- Boot volumes for EC2 instances

---

## Amazon EFS

**What is EFS?**

EFS (Elastic File System) is a managed NFS file system that can be shared across multiple EC2 instances.

**Characteristics**

- Scales automatically
- NFS-based shared file storage
- Multiple EC2 instances can access simultaneously
- Regional service with high availability

**Common Use Cases**

- Shared file storage for microservices
- Content management systems
- Web serving and development environments
- Big data analytics workloads

---

## Storage Type Comparison

| Feature | S3 | EBS | EFS |
|---------|-----|-----|-----|
| Type | Object storage | Block storage | File storage |
| Access | HTTP API/URL | Attached to EC2 | NFS mount |
| Sharing | Unlimited access | Single EC2 (except io2) | Multiple EC2 instances |
| Latency | Higher | Very low | Low |
| Use Case | Backups, media, static content | Databases, OS disks | Shared files, microservices |
| Persistence | Durable by default | Persists independently | Persists independently |
| Scalability | Unlimited | Fixed size (resizable) | Auto-scaling |

---

## Storage Decision Factors

**Choosing the Right Storage**

Consider these factors when selecting storage:

- Access frequency: Hot (frequent) vs Cold (infrequent) vs Archival (rare)
- Latency requirements: Milliseconds for databases vs hours for archives
- Durability and availability SLA needed
- Cost optimization: Hot storage is expensive, archival is cheap
- Compliance: Data retention, WORM, auditing requirements
- Performance: IOPS for EBS vs throughput for EFS

**Storage Selection Guide**

| Scenario | Recommended Storage |
|----------|-------------------|
| Frequently accessed files | S3 Standard |
| Infrequently accessed files | S3 Standard-IA or One Zone-IA |
| Long-term archives | S3 Glacier or Glacier Deep Archive |
| Database storage | EBS |
| Shared application files | EFS |
| Static website hosting | S3 |
| Unknown access patterns | S3 Intelligent-Tiering |

---

## Amazon RDS

**What is RDS?**

RDS (Relational Database Service) is a managed relational database service. AWS handles provisioning, patching, backup, scaling, and high availability automatically.

**Supported Database Engines**

| Engine | Description |
|--------|-------------|
| Amazon Aurora | MySQL and PostgreSQL compatible, high performance |
| MySQL | Open source relational database |
| PostgreSQL | Advanced open source database |
| MariaDB | MySQL fork with enhancements |
| Oracle | Enterprise database |
| Microsoft SQL Server | Microsoft enterprise database |

**RDS Deployment Types**

- RDS Standard: Fully managed by AWS
- RDS Custom: You manage OS and database configuration

**Common Use Cases**

- Web and mobile applications requiring relational databases
- Reporting and analytics workloads
- Enterprise transactional systems
- E-commerce platforms

---

## Amazon Aurora

**What is Aurora?**

Aurora is an AWS-developed high-performance relational database compatible with MySQL and PostgreSQL.

**Key Features**

- 5x faster than standard MySQL
- 3x faster than standard PostgreSQL
- Auto-scaling storage up to 128 TB
- Up to 15 read replicas
- Multi-AZ deployment for high availability
- Automatic backup and point-in-time recovery

**When to Use Aurora**

- Need enterprise-grade relational database performance
- Require high availability and automatic failover
- Want MySQL or PostgreSQL compatibility with better performance
- Have demanding read-heavy workloads

---

## Amazon DynamoDB

**What is DynamoDB?**

DynamoDB is a fully managed NoSQL key-value and document database.

**Key Features**

- Multi-region replication
- Single-digit millisecond latency
- Automatic scaling
- Built-in security and backup
- Event-driven architecture with DynamoDB Streams

**Common Use Cases**

- Global applications requiring low latency
- IoT applications with high write throughput
- Gaming leaderboards and session storage
- Real-time applications
- Mobile and web applications with unpredictable traffic

---

## Database Selection Guide

| Factor | RDS/Aurora | DynamoDB |
|--------|-----------|----------|
| Data Model | Relational (tables, rows, columns) | NoSQL (key-value, document) |
| Query Type | Complex SQL queries, joins | Simple key-based queries |
| Scalability | Vertical scaling, read replicas | Automatic horizontal scaling |
| Latency | Low (milliseconds) | Very low (single-digit ms) |
| Use Case | Traditional apps, analytics, complex queries | High-scale apps, real-time data, simple access patterns |
| Cost | Based on instance size | Based on throughput and storage |