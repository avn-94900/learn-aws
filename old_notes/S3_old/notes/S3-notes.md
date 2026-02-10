# AWS S3 Core Concepts

Amazon Simple Storage Service (S3) is a highly scalable, durable, and secure object storage service that allows you to store and retrieve any amount of data from anywhere on the web.

**Key Characteristics**

- Fully managed service with no infrastructure management required
- Object-based storage model
- Global accessibility with regional data residency
- 99.999999999% (11 nines) durability for data protection
- Virtually unlimited storage capacity

---

## S3 Bucket

**What is an S3 Bucket?**

A bucket is a top-level container for storing objects in S3.

**Bucket Properties**

- Globally unique names across all AWS accounts and regions
- Created in a specific AWS region but accessible globally
- Virtually unlimited storage capacity
- Can contain unlimited number of objects
- Organizes and manages access to data

**Bucket Naming Rules**

- Use DNS-compliant names with lowercase letters
- Length between 3 and 63 characters
- Cannot contain uppercase letters or underscores
- Must start with lowercase letter or number
- Cannot be formatted as IP addresses

---

## S3 Object

**What is an S3 Object?**

Objects are the fundamental entities stored in Amazon S3, consisting of data and metadata.

**Object Components**

| Component | Description |
|-----------|-------------|
| Object Data | The actual file content (up to 5 TB) |
| Object Key | Unique identifier/name within the bucket |
| Version ID | Unique identifier for object versions (if versioning enabled) |
| Metadata | System and user-defined key-value pairs |
| ETag | Entity tag for object integrity verification |
| Storage Class | Current storage class of the object |
| Timestamps | Last modified date and creation time |

**Object Size Limits**

- Single object: up to 5 TB
- Single PUT operation: up to 5 GB
- For objects larger than 5 GB: multipart upload required
- Recommended multipart upload for objects larger than 100 MB

---

## Object Key

**What is an Object Key?**

The object key is the unique identifier for an object within a bucket, similar to a filename.

**Key Properties**

- UTF-8 encoded string up to 1,024 bytes long
- Must be unique within the bucket
- Can include prefixes and delimiters to simulate folder structure
- Case-sensitive

**Example Keys**

- Simple: invoice-12345.pdf
- Hierarchical: documents/2024/invoices/invoice-12345.pdf
- With prefix: project-alpha/reports/quarterly-report.pdf

**Best Practices**

- Avoid sequential prefixes for better performance
- Use prefixes to organize objects logically
- Avoid special characters that may cause issues
- Distribute load across different prefixes

---

## Object Lifecycle States

S3 objects progress through different states during their lifetime:

| State | Description |
|-------|-------------|
| Created | Object successfully uploaded to S3 |
| Accessible | Object available for read/write operations |
| Transitioned | Object moved to different storage class via lifecycle rule |
| Archived | Object stored in Glacier or Deep Archive |
| Expired | Object marked for deletion by lifecycle rule |
| Deleted | Object permanently removed from S3 |

---

## Storage Classes

S3 offers multiple storage classes optimized for different use cases and cost requirements.

**Storage Class Comparison**

| Storage Class | Use Case | Retrieval Time | Durability | Availability |
|---------------|----------|----------------|------------|--------------|
| S3 Standard | Frequently accessed data | Instant | 11 nines | 99.99% |
| S3 Intelligent-Tiering | Unknown or changing access patterns | Instant | 11 nines | 99.9% |
| S3 Standard-IA | Infrequent access | Instant | 11 nines | 99.9% |
| S3 One Zone-IA | Non-critical, infrequent access | Instant | 11 nines | 99.5% |
| S3 Glacier Instant Retrieval | Archive with instant access | Instant | 11 nines | 99.9% |
| S3 Glacier Flexible Retrieval | Archive data | Minutes to hours | 11 nines | 99.99% |
| S3 Glacier Deep Archive | Long-term retention | 12+ hours | 11 nines | 99.99% |

**Choosing the Right Storage Class**

- S3 Standard: Active data, websites, content distribution
- Intelligent-Tiering: Unpredictable access patterns, automatic cost optimization
- Standard-IA: Infrequently accessed but needs instant retrieval
- One Zone-IA: Non-critical data, can be recreated if lost
- Glacier Instant Retrieval: Archive with instant access requirements
- Glacier Flexible Retrieval: Archive with flexible retrieval needs
- Glacier Deep Archive: Compliance archives, long-term retention

---

## Versioning

**What is Versioning?**

S3 versioning allows you to keep multiple variants of an object in the same bucket.

**Versioning States**

- Unversioned: Default state, no versioning
- Enabled: All new and modified objects get version IDs
- Suspended: Stops creating new versions but retains existing ones

**Key Features**

- Unique version ID assigned to each object version
- Latest version retrieved by default
- Prevents accidental deletion or modification
- Can restore previous versions
- Once enabled, cannot be fully disabled (only suspended)

**Use Cases**

- Protect against accidental deletions
- Recover from application failures
- Archive and audit data changes
- Compliance requirements

---

## Lifecycle Management

**What is Lifecycle Management?**

Lifecycle rules automate the transition and expiration of objects based on defined criteria.

**Lifecycle Actions**

| Action | Description |
|--------|-------------|
| Transition | Move objects between storage classes |
| Expiration | Automatically delete objects after specified time |
| Abort Incomplete Multipart Uploads | Clean up incomplete uploads |
| Noncurrent Version Expiration | Delete old versions |

**Lifecycle Rule Components**

- Rule name and status (enabled/disabled)
- Scope: entire bucket, prefix, or object tags
- Actions: transitions and expirations
- Timeline: number of days after creation or transition

**Cost Optimization Example**

1. Standard storage for first 30 days
2. Transition to Standard-IA after 30 days
3. Transition to Glacier after 90 days
4. Expire after 365 days

---

## Access Control

### Bucket Policies

**What are Bucket Policies?**

JSON-based access control policies that define permissions for S3 resources.

**Policy Elements**

| Element | Description |
|---------|-------------|
| Principal | Who can access (AWS accounts, users, services) |
| Action | What operations are allowed (s3:GetObject, s3:PutObject) |
| Resource | Which buckets or objects (ARN) |
| Effect | Allow or Deny |
| Condition | Optional conditions for access |

**Policy Characteristics**

- Resource-based policies attached to S3 buckets
- Apply to bucket and all objects within it
- Maximum size: 20 KB per policy
- Written in JSON format using AWS Policy Language

### Access Control Lists (ACLs)

**What are ACLs?**

XML-based access control mechanism for buckets and objects.

**ACL Permissions**

- READ: List objects in bucket or read object data
- WRITE: Create, overwrite, delete objects
- READ_ACP: Read ACL permissions
- WRITE_ACP: Write ACL permissions
- FULL_CONTROL: All permissions

**Recommendation**

Use bucket policies or IAM policies instead of ACLs for new implementations. ACLs are legacy and have limited functionality.

### IAM Policies

Identity-based permissions for users and roles to access S3 resources.

### S3 Access Points

Simplified access management for shared datasets with named network endpoints.

---

## Data Consistency Model

**Strong Consistency (Current Model)**

Since December 2020, S3 provides strong read-after-write consistency for all operations.

**Consistency Guarantees**

- Read-after-Write: Immediately consistent for new objects
- List Consistency: Bucket listings reflect recent changes immediately
- Update Consistency: Overwrite and delete operations are strongly consistent
- Global: Consistent across all AWS regions and edge locations

**Previous Model (Historical)**

- Eventual Consistency: Used for overwrites and deletes
- Read-after-Write: Immediate consistency for new objects only
- This model is no longer in use

---

## Multipart Upload

**What is Multipart Upload?**

A mechanism for uploading large objects in parts for improved performance and reliability.

**When to Use**

| File Size | Recommendation |
|-----------|---------------|
| Less than 100 MB | Single PUT operation |
| 100 MB to 5 GB | Multipart upload recommended |
| Greater than 5 GB | Multipart upload required |

**Benefits**

- Improved upload speed through parallel uploads
- Resume failed uploads from last completed part
- Upload files while still creating them
- Better network utilization

**Process Flow**

1. Initiate: Start multipart upload and receive Upload ID
2. Upload Parts: Upload parts (5 MB to 5 GB each) with part numbers
3. Complete: Combine all parts into single object
4. Abort: Cancel incomplete uploads to avoid storage charges

**Best Practices**

- Use parts of 5 MB to 5 GB (except last part)
- Upload parts in parallel for better performance
- Implement retry logic for failed parts
- Clean up incomplete uploads using lifecycle rules

---

## TransferManager

**What is TransferManager?**

A high-level utility provided by the AWS SDK that automatically manages multipart uploads for large files.

**Features**

- Automatically splits large files into parts
- Uploads parts in parallel using multiple threads
- Resumes uploads if interrupted
- Manages retries and thread pooling
- Handles cleanup if upload fails

**How It Works Internally**

1. Splits the file into parts (default: 5 MB per part)
2. Uploads parts in parallel using thread pool
3. Tracks progress and manages upload state
4. Calls CompleteMultipartUpload API after all parts uploaded

**Configuration Options**

| Option | Description |
|--------|-------------|
| withMultipartUploadThreshold | Minimum size to trigger multipart upload |
| withMinimumUploadPartSize | Size of each part |
| withExecutorFactory | Custom thread pool for uploads |
| withShutDownThreadPools | Controls thread pool shutdown behavior |

**AmazonS3 vs TransferManager**

| Feature | AmazonS3 | TransferManager |
|---------|----------|-----------------|
| Multipart Upload Handling | Manual | Automatic |
| Progress Tracking | Manual | Built-in |
| Asynchronous Support | Limited | Yes |
| Parallel Uploads | No | Yes |

**SDK Versions**

- AWS SDK v1: Use TransferManager class
- AWS SDK v2: Use TransferManager from software.amazon.awssdk.transfer.s3 module

---

## Pre-Signed URLs

**What are Pre-Signed URLs?**

Secure, time-limited URLs that allow users to access private S3 objects without AWS credentials.

**Use Cases**

- Temporary file sharing with expiration time
- Secure file uploads directly to S3 from clients
- Controlling access without exposing AWS credentials
- Sharing private content with specific users

**Key Properties**

- Includes AWS access credentials as query parameters
- Has expiration time (configurable)
- Limited to specific operation (GET, PUT, DELETE)
- Can only access one specific object
- Generated using AWS SDK or CLI

**Operations Supported**

- GET: Download objects
- PUT: Upload objects
- DELETE: Delete objects
- HEAD: Get object metadata

---

## Data Transfer Methods

### Upload Methods

| Method | Use Case | Max Size |
|--------|----------|----------|
| Single PUT | Small files | 5 GB |
| Multipart Upload | Large files, improved performance | 5 TB |
| Transfer Acceleration | Global uploads, faster transfers | Any |
| AWS CLI/SDK | Programmatic uploads with retry logic | Any |
| Pre-signed URLs | Secure uploads without credentials | Any |

### Download Methods

| Method | Use Case |
|--------|----------|
| Direct Download | Standard GET requests |
| Range Requests | Download specific byte ranges |
| Transfer Acceleration | Accelerated downloads via CloudFront |
| Pre-signed URLs | Temporary secure downloads |
| CloudFront Distribution | Global content delivery |

---

## CORS (Cross-Origin Resource Sharing)

**What is CORS?**

Configuration that allows web applications to access S3 resources from different domains.

**CORS Configuration**

- Define allowed origins (domains)
- Specify allowed methods (GET, PUT, POST, DELETE)
- Configure allowed headers
- Set credentials support
- Stored as XML document on bucket

**Example CORS Configuration**

```xml
<CORSConfiguration>
  <CORSRule>
    <AllowedOrigin>https://example.com</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
  </CORSRule>
</CORSConfiguration>
```

**Use Cases**

- Web applications accessing S3 from browser
- Single-page applications uploading files
- Cross-domain API calls to S3

---

## Security and Encryption

### Encryption at Rest

| Method | Description | Key Management |
|--------|-------------|----------------|
| SSE-S3 | Server-side encryption with S3-managed keys | AWS managed |
| SSE-KMS | Server-side encryption with KMS keys | You control via KMS |
| SSE-C | Server-side encryption with customer keys | You provide keys |
| Client-Side | Encrypt before uploading | You manage |

### Encryption in Transit

- HTTPS/TLS: All API calls encrypted using SSL/TLS
- VPC Endpoints: Private connectivity without internet transit
- AWS PrivateLink: Secure connection through AWS backbone

### Access Control Methods

- IAM Policies: Identity-based permissions
- Bucket Policies: Resource-based permissions
- ACLs: Object and bucket-level control (legacy)
- Pre-signed URLs: Time-limited access
- S3 Access Points: Simplified management for shared datasets

---

## Monitoring and Logging

### CloudTrail Integration

**What is CloudTrail for S3?**

Records all S3 API calls for auditing and compliance.

**Features**

- API Logging: All S3 API calls recorded
- Data Events: Object-level operations (optional)
- Management Events: Bucket-level operations
- Cross-Account: Logging across AWS accounts

### Server Access Logs

**What are Access Logs?**

Detailed records of requests made to S3 bucket.

**Log Information**

- Requester information
- Bucket and object details
- Operation and response status
- Error codes
- Turn-around time

**Configuration**

- Logs delivered to specified S3 bucket
- Can be analyzed with analytics tools
- Use for security auditing and access patterns

### CloudWatch Metrics

**Available Metrics**

| Metric Type | Examples |
|-------------|----------|
| Storage Metrics | Bucket size, object count |
| Request Metrics | Request rates, error rates, latency |
| Replication Metrics | Cross-region replication status |
| Custom Metrics | Application-specific via API |

---

## Best Practices

### Naming Conventions

- Use DNS-compliant bucket names with lowercase letters
- Avoid sequential prefixes for better performance
- Use prefixes to organize objects logically
- Avoid special characters that may cause issues

### Performance Optimization

- Distribute load across different prefixes for high request rates
- Use multipart upload for large files and parallel processing
- Enable Transfer Acceleration for global data transfer
- Use CloudFront CDN for frequently accessed content
- Implement retry logic with exponential backoff

### Cost Optimization

- Use appropriate storage class for access patterns
- Implement lifecycle rules to automatically transition or delete objects
- Use Intelligent-Tiering for unpredictable access patterns
- Monitor storage costs and usage patterns
- Clean up incomplete multipart uploads
- Delete unnecessary object versions

### Security Best Practices

- Enable versioning for critical data
- Use encryption at rest and in transit
- Implement least privilege access with IAM policies
- Enable MFA delete for versioned buckets
- Use VPC endpoints for private connectivity
- Enable logging and monitoring
- Regular security audits and access reviews