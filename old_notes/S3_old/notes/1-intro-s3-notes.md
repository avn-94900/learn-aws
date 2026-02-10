# AWS S3 Core Concepts

Amazon Simple Storage Service (S3) is a highly scalable, durable, and secure object storage service that allows you to store and retrieve any amount of data from anywhere on the web. Understanding the following core concepts is essential for working with S3 effectively.

- **Fully managed service** (no infrastructure management required)
- **Object-based storage model** (store files as objects in buckets)
- **Global accessibility** with regional data residency
- **99.999999999% (11 9's) durability** for data protection

## 1. S3 Bucket

An S3 bucket is a container that holds objects (files) in Amazon S3.

- **Definition**: A top-level container for storing objects in S3
- **Purpose**: Organizes and manages access to your data
- **Naming**: Globally unique names across all AWS accounts and regions
- **Capacity**: Virtually unlimited storage capacity
- **Regional**: Created in a specific AWS region but accessible globally

## 2. S3 Object

Objects are the fundamental entities stored in Amazon S3, consisting of data and metadata.

- **Definition**: Individual files or data items stored in S3 buckets
- **Components**: Object data, metadata, and unique key (name)
- **Size Limit**: Single object can be up to 5 TB
- **Key**: Unique identifier within a bucket (acts as filename)
- **Versioning**: Multiple versions of the same object can be stored

## 3. Object Key

The object key is the unique identifier for an object within a bucket.

- **Definition**: The name that you assign to an object
- **Format**: UTF-8 encoded string up to 1,024 bytes long
- **Path Structure**: Can include prefixes and delimiters (like folders)
- **Example**: `documents/2024/invoice-12345.pdf`
- **Uniqueness**: Must be unique within the bucket

## 4. Storage Classes

S3 offers multiple storage classes optimized for different use cases and cost requirements.

- **S3 Standard**: General-purpose storage with high durability and availability
- **S3 Intelligent-Tiering**: Automatically moves data between access tiers
- **S3 Standard-IA**: Infrequent access with lower cost but retrieval charges
- **S3 One Zone-IA**: Lower cost for infrequently accessed, non-critical data
- **S3 Glacier**: Long-term archival with retrieval times from minutes to hours
- **S3 Glacier Deep Archive**: Lowest cost for long-term retention (12+ hours retrieval)

## 5. Bucket Policies

Bucket policies are JSON-based access control policies that define permissions for S3 resources.

- **Definition**: Resource-based policies attached to S3 buckets
- **Format**: JSON documents using AWS Policy Language
- **Scope**: Apply to the bucket and all objects within it
- **Elements**: Principal, Action, Resource, Condition, Effect
- **Size Limit**: Maximum 20 KB per policy

## 6. Access Control Lists (ACLs)

ACLs provide a legacy method for controlling access to buckets and objects.

- **Definition**: XML-based access control mechanism
- **Granularity**: Can be applied to individual objects or entire buckets
- **Permissions**: READ, WRITE, READ_ACP, WRITE_ACP, FULL_CONTROL
- **Grantees**: AWS accounts, predefined groups, or email addresses
- **Recommendation**: Use bucket policies or IAM policies instead for new implementations

## 7. Versioning

S3 versioning allows you to keep multiple variants of an object in the same bucket.

- **Definition**: Feature that maintains multiple versions of objects
- **States**: Unversioned (default), Enabled, or Suspended
- **Version ID**: Unique identifier assigned to each object version
- **Current Version**: The latest version retrieved by default
- **Protection**: Prevents accidental deletion or modification

## 8. Lifecycle Management

Lifecycle rules automate the transition and expiration of objects based on defined criteria.

- **Definition**: Rules that automatically manage objects over their lifetime
- **Transitions**: Move objects between storage classes
- **Expiration**: Automatically delete objects after specified time
- **Scope**: Apply to entire bucket, prefixes, or tags
- **Cost Optimization**: Reduces storage costs by moving to cheaper classes

---

## S3 Object Anatomy
Each S3 object includes the following components:

* **Object Data** – The actual file content (up to 5 TB)
* **Object Key** – Unique identifier/name within the bucket
* **Version ID** – Unique identifier for object versions (if versioning enabled)
* **Metadata** – System and user-defined key-value pairs
* **ETag** – Entity tag for object integrity verification
* **Storage Class** – Current storage class of the object
* **Timestamps** – Last modified date and creation time

---

## Object Lifecycle States
S3 objects progress through different states during their lifecycle:

* **Created** – Object is successfully uploaded to S3
* **Accessible** – Object is available for read/write operations
* **Transitioned** – Object moved to different storage class via lifecycle rule
* **Archived** – Object stored in Glacier or Deep Archive
* **Expired** – Object marked for deletion by lifecycle rule
* **Deleted** – Object permanently removed from S3

---

## Data Consistency Model

### Strong Consistency
* **Read-after-Write**: Immediately consistent for new objects
* **List Consistency**: Bucket listings reflect recent changes
* **Update Consistency**: Overwrite and delete operations are strongly consistent
* **Global**: Consistent across all AWS regions and edge locations

### Previous Model (Historical)
* **Eventual Consistency**: Used for overwrites and deletes (legacy behavior)
* **Read-after-Write**: Immediate consistency for new objects only
* **Timeline**: S3 achieved strong consistency in December 2020

---

## Multipart Upload

### Purpose and Benefits
* **Large Files**: Required for objects larger than 5 GB
* **Performance**: Parallel uploads improve speed
* **Resilience**: Resume failed uploads from last completed part
* **Threshold**: Recommended for objects larger than 100 MB

### Process Flow
* **Initiate**: Start multipart upload and receive Upload ID
* **Upload Parts**: Upload parts (5 MB to 5 GB each) with part numbers
* **Complete**: Combine all parts into single object
* **Abort**: Cancel incomplete uploads to avoid charges

---

## Cross-Origin Resource Sharing (CORS)

### Configuration
* **Purpose**: Allow web applications to access S3 resources from different domains
* **Rules**: Define allowed origins, methods, headers, and credentials
* **XML Format**: Configuration stored as XML document on bucket
* **Preflight**: Handles browser preflight requests for complex operations

### Example CORS Configuration
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

---

## Data Transfer Methods

### Upload Methods
* **Single Put**: Direct upload for objects up to 5 GB
* **Multipart Upload**: For large objects or improved performance
* **Transfer Acceleration**: Uses CloudFront edge locations for faster uploads
* **AWS CLI/SDK**: Programmatic uploads with retry logic
* **Pre-signed URLs**: Temporary URLs for secure uploads without AWS credentials

### Download Methods
* **Direct Download**: Standard GET requests for object retrieval
* **Range Requests**: Download specific byte ranges of objects
* **Transfer Acceleration**: Accelerated downloads via CloudFront
* **Pre-signed URLs**: Temporary URLs for secure downloads
* **CloudFront Distribution**: Content delivery network for global distribution

---

## Security and Encryption

### Encryption at Rest
* **SSE-S3**: Server-side encryption with S3-managed keys
* **SSE-KMS**: Server-side encryption with AWS KMS keys
* **SSE-C**: Server-side encryption with customer-provided keys
* **Client-Side**: Encrypt data before uploading to S3

### Encryption in Transit
* **HTTPS/TLS**: All API calls encrypted using SSL/TLS
* **VPC Endpoints**: Private connectivity without internet transit
* **AWS PrivateLink**: Secure connection to S3 through AWS backbone

### Access Control
* **IAM Policies**: Identity-based permissions for users and roles
* **Bucket Policies**: Resource-based permissions for buckets
* **ACLs**: Object and bucket-level access control (legacy)
* **Pre-signed URLs**: Time-limited access without AWS credentials
* **S3 Access Points**: Simplified access management for shared datasets

---

## Monitoring and Logging

### CloudTrail Integration
* **API Logging**: Records all S3 API calls for auditing
* **Data Events**: Tracks object-level operations (optional)
* **Management Events**: Logs bucket-level operations
* **Cross-Account**: Supports logging across AWS accounts

### Access Logging
* **Server Access Logs**: Detailed records of requests made to bucket
* **Log Format**: Standard fields including requester, operation, response
* **Delivery**: Logs delivered to specified S3 bucket
* **Analysis**: Use with analytics tools for insights

### CloudWatch Metrics
* **Storage Metrics**: Bucket size and object count
* **Request Metrics**: Request rates and error rates
* **Replication Metrics**: Cross-region replication status
* **Custom Metrics**: Application-specific metrics via CloudWatch API

---

## Best Practices and Patterns

### Naming Conventions
* **Bucket Names**: Use DNS-compliant names with lowercase letters
* **Object Keys**: Avoid sequential prefixes for better performance
* **Hierarchical Structure**: Use prefixes to organize objects logically
* **Reserved Characters**: Avoid special characters that may cause issues

### Performance Optimization
* **Request Rate**: Distribute load across different prefixes
* **Multipart Upload**: Use for large files and parallel processing
* **Transfer Acceleration**: Enable for global data transfer
* **CloudFront**: Use CDN for frequently accessed content

### Cost Optimization
* **Storage Classes**: Use appropriate class for access patterns
* **Lifecycle Rules**: Automatically transition or delete objects
* **Intelligent Tiering**: Let S3 optimize storage class automatically
* **Monitoring**: Track storage costs and usage patterns

A **pre-signed URL** is a secure, time-limited URL that allows users to access a private object in an Amazon S3 bucket without requiring them to have AWS credentials. It can be used for both **uploading (PUT)** and **downloading (GET)** files to/from S3.

---

### ✅ Use Cases of Pre-Signed URLs

* Temporary file sharing (e.g., send a download link that expires in 10 minutes)
* Secure file uploads directly to S3 from clients (bypassing your server)
* Controlling access without exposing AWS credentials

---

### 🔐 Key Properties of Pre-Signed URLs

* Includes AWS access credentials as query parameters
* Has an **expiration time**
* Limited to a **specific operation** (GET, PUT, DELETE)
* Can only access **one specific object**

---

<br/><br/>

###  `TransferManager`
- `TransferManager` class (part of the Amazon S3 Transfer Manager utility) **automatically manages multipart uploads for large files** to Amazon S3.
- `TransferManager` is a high-level utility provided by the AWS SDK that:

* Automatically splits large files into parts
* Uploads them in parallel using multiple threads
* Resumes uploads if they are interrupted
* Manages retries and thread pooling
* Handles cleanup if the upload fails

---

### ✅ Multipart Upload: Why & When?

Multipart upload is recommended when:

* File size > 5 MB (technically possible)
* Strongly recommended for files > 100 MB
* Mandatory for files > 5 GB

Multipart upload **improves performance**, **reliability**, and **resumability**.

---

### ✅ How `TransferManager` Does It Internally

1. **Splits the File**: The file is split into parts (default part size: 5 MB, can be configured).
2. **Parallel Upload**: Uploads parts in parallel using a thread pool.
3. **Tracks Progress**: Monitors and manages upload progress.
4. **Merges Parts**: After all parts are uploaded, it finalizes the upload by calling the `CompleteMultipartUpload` API.

---

### ✅ Configuration Options

You can customize:

| Option                                         | Description                                         |
| ---------------------------------------------- | --------------------------------------------------- |
| `withMultipartUploadThreshold(long threshold)` | Minimum size (in bytes) to trigger multipart upload |
| `withMinimumUploadPartSize(long size)`         | Size of each part                                   |
| `withExecutorFactory(...)`                     | Custom thread pool for uploads                      |
| `withShutDownThreadPools(...)`                 | Controls thread pool shutdown behavior              |



---

### ✅ Benefits of `TransferManager`

* Automatic multipart upload
* Multi-threaded: improves speed
* Reliable: can resume interrupted uploads
* Simple to use compared to raw multipart APIs

<br/>

| Feature                   | `AmazonS3` | `TransferManager` |
| ------------------------- | ---------- | ----------------- |
| Multipart Upload Handling | Manual     | ✅ Automatic       |
| Progress Tracking         | Manual     | ✅ Built-in        |
| Asynchronous Support      | Limited    | ✅ Yes             |
| Parallel Uploads          | ❌ No       | ✅ Yes             |


---

### ⚠️ Note

`TransferManager` is part of **AWS SDK v1**. For AWS SDK v2 (Java), you use the **TransferManager from the S3 Transfer Manager module** (`software.amazon.awssdk.transfer.s3`).

Let me know if you want a v2 example too.

