# AWS SDK 2.0 S3 Client - Enhanced Reference Guide

## 🚀 Overview

AWS SDK 2.0 provides both synchronous (`S3Client`) and asynchronous (`S3AsyncClient`) interfaces for S3 operations. This guide covers the most commonly used methods with practical examples and best practices.

---

## ✅ Synchronous S3Client Methods

### 🔹 **Object Operations**

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| `putObject(PutObjectRequest, RequestBody)` | Upload a file | `bucket`, `key`, `contentType`, `metadata` | Use `RequestBody.fromFile()` for files |
| `getObject(GetObjectRequest, ResponseTransformer)` | Download a file | `bucket`, `key`, `versionId` | Use `ResponseTransformer.toFile()` for direct file download |
| `deleteObject(DeleteObjectRequest)` | Delete a file | `bucket`, `key`, `versionId` | Returns `DeleteObjectResponse` with deletion info |
| `copyObject(CopyObjectRequest)` | Copy object between buckets/keys | `sourceBucket`, `sourceKey`, `destinationBucket`, `destinationKey` | Supports server-side copy (no data transfer) |
| `listObjectsV2(ListObjectsV2Request)` | List objects in bucket | `bucket`, `prefix`, `delimiter`, `maxKeys`, `continuationToken` | Paginated response, use `isTruncated()` to check for more |
| `headObject(HeadObjectRequest)` | Get object metadata only | `bucket`, `key`, `versionId` | Lightweight operation, no data transfer |
| `getObjectAcl(GetObjectAclRequest)` | Get object ACL | `bucket`, `key`, `versionId` | Returns permissions and ownership info |
| `putObjectAcl(PutObjectAclRequest)` | Set object ACL | `bucket`, `key`, `acl` | Use predefined ACLs or custom grants |
| `getObjectAttributes(GetObjectAttributesRequest)` | Get specific object attributes | `bucket`, `key`, `objectAttributes` | Selective metadata retrieval |
| `restoreObject(RestoreObjectRequest)` | Restore archived object | `bucket`, `key`, `restoreRequest` | For Glacier/Deep Archive objects |

### 🔹 **Bucket Operations**

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| `createBucket(CreateBucketRequest)` | Create new bucket | `bucket`, `createBucketConfiguration` | Region-specific configuration required |
| `listBuckets()` | List all buckets | None | Returns all buckets owned by authenticated user |
| `deleteBucket(DeleteBucketRequest)` | Delete empty bucket | `bucket` | Bucket must be empty before deletion |
| `getBucketAcl(GetBucketAclRequest)` | Get bucket ACL | `bucket` | Returns bucket-level permissions |
| `putBucketAcl(PutBucketAclRequest)` | Set bucket ACL | `bucket`, `acl` | Configure bucket-level access |
| `getBucketLocation(GetBucketLocationRequest)` | Get bucket region | `bucket` | Returns AWS region of the bucket |
| `getBucketVersioning(GetBucketVersioningRequest)` | Get versioning status | `bucket` | Returns versioning configuration |
| `putBucketVersioning(PutBucketVersioningRequest)` | Enable/disable versioning | `bucket`, `versioningConfiguration` | Enable object versioning |
| `getBucketPolicy(GetBucketPolicyRequest)` | Get bucket policy | `bucket` | Returns IAM policy document |
| `putBucketPolicy(PutBucketPolicyRequest)` | Set bucket policy | `bucket`, `policy` | Apply IAM policy to bucket |
| `deleteBucketPolicy(DeleteBucketPolicyRequest)` | Remove bucket policy | `bucket` | Removes all bucket policies |
| `getBucketCors(GetBucketCorsRequest)` | Get CORS configuration | `bucket` | Returns cross-origin resource sharing rules |
| `putBucketCors(PutBucketCorsRequest)` | Set CORS configuration | `bucket`, `corsConfiguration` | Configure CORS rules |
| `getBucketWebsite(GetBucketWebsiteRequest)` | Get website configuration | `bucket` | Returns static website hosting config |
| `putBucketWebsite(PutBucketWebsiteRequest)` | Enable static website hosting | `bucket`, `websiteConfiguration` | Configure bucket for web hosting |
| `getBucketNotification(GetBucketNotificationConfigurationRequest)` | Get notification config | `bucket` | Returns event notification settings |
| `putBucketNotification(PutBucketNotificationConfigurationRequest)` | Set notification config | `bucket`, `notificationConfiguration` | Configure SNS/SQS/Lambda notifications |

### 🔹 **Multipart Upload Operations**

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| `createMultipartUpload(CreateMultipartUploadRequest)` | Initialize multipart upload | `bucket`, `key`, `contentType`, `metadata` | Returns `uploadId` for subsequent operations |
| `uploadPart(UploadPartRequest, RequestBody)` | Upload single part | `bucket`, `key`, `uploadId`, `partNumber` | Part numbers: 1-10,000, min 5MB except last |
| `completeMultipartUpload(CompleteMultipartUploadRequest)` | Finalize multipart upload | `bucket`, `key`, `uploadId`, `multipartUpload` | Requires list of part ETags |
| `abortMultipartUpload(AbortMultipartUploadRequest)` | Cancel multipart upload | `bucket`, `key`, `uploadId` | Cleans up uploaded parts |
| `listParts(ListPartsRequest)` | List uploaded parts | `bucket`, `key`, `uploadId` | Returns parts with ETags and sizes |
| `listMultipartUploads(ListMultipartUploadsRequest)` | List ongoing uploads | `bucket`, `prefix` | Find incomplete multipart uploads |
| `uploadPartCopy(UploadPartCopyRequest)` | Copy part from another object | `sourceBucket`, `sourceKey`, `destinationBucket`, `destinationKey`, `uploadId`, `partNumber` | Server-side copy for multipart |

### 🔹 **Object Versioning**

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| `listObjectVersions(ListObjectVersionsRequest)` | List all object versions | `bucket`, `key`, `prefix` | Returns all versions including delete markers |
| `deleteObject(DeleteObjectRequest)` | Delete specific version | `bucket`, `key`, `versionId` | Permanent deletion when version specified |
| `putBucketVersioning(PutBucketVersioningRequest)` | Configure versioning | `bucket`, `versioningConfiguration` | Enable/suspend versioning |
| `getObjectVersion(GetObjectRequest)` | Get specific version | `bucket`, `key`, `versionId` | Download specific object version |

### 🔹 **Security & Encryption**

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| `putObject()` with SSE | Upload with encryption | `serverSideEncryption`, `ssekmsKeyId`, `sseCustomerKey` | Supports SSE-S3, SSE-KMS, SSE-C |
| `getObject()` with SSE | Download encrypted object | `sseCustomerKey`, `sseCustomerAlgorithm` | Required for SSE-C objects |
| `getBucketEncryption(GetBucketEncryptionRequest)` | Get bucket encryption | `bucket` | Returns default encryption configuration |
| `putBucketEncryption(PutBucketEncryptionRequest)` | Set bucket encryption | `bucket`, `serverSideEncryptionConfiguration` | Configure default encryption |
| `getBucketPublicAccessBlock(GetPublicAccessBlockRequest)` | Get public access settings | `bucket` | Returns public access block configuration |
| `putBucketPublicAccessBlock(PutPublicAccessBlockRequest)` | Block public access | `bucket`, `publicAccessBlockConfiguration` | Prevent accidental public access |

### 🔹 **Lifecycle & Storage Classes**

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| `getBucketLifecycle(GetBucketLifecycleConfigurationRequest)` | Get lifecycle rules | `bucket` | Returns object lifecycle configuration |
| `putBucketLifecycle(PutBucketLifecycleConfigurationRequest)` | Set lifecycle rules | `bucket`, `lifecycleConfiguration` | Automate object transitions and deletion |
| `putObject()` with storage class | Upload with specific storage class | `storageClass` | IA, GLACIER, DEEP_ARCHIVE, etc. |
| `copyObject()` with storage class | Change storage class | `storageClass`, `metadataDirective` | Transition objects between storage classes |

---

## ✅ Asynchronous S3AsyncClient Methods

### 📦 **Object Operations (Async)**

| Method | Return Type | Purpose | Key Differences |
|--------|-------------|---------|----------------|
| `putObject(PutObjectRequest, AsyncRequestBody)` | `CompletableFuture<PutObjectResponse>` | Async upload | Use `AsyncRequestBody.fromFile()` or `fromPublisher()` |
| `getObject(GetObjectRequest, AsyncResponseTransformer)` | `CompletableFuture<ResponseT>` | Async download | Use `AsyncResponseTransformer.toFile()` or `toPublisher()` |
| `deleteObject(DeleteObjectRequest)` | `CompletableFuture<DeleteObjectResponse>` | Async delete | Non-blocking deletion |
| `copyObject(CopyObjectRequest)` | `CompletableFuture<CopyObjectResponse>` | Async copy | Server-side async copy |
| `headObject(HeadObjectRequest)` | `CompletableFuture<HeadObjectResponse>` | Async metadata | Lightweight async operation |
| `listObjectsV2(ListObjectsV2Request)` | `CompletableFuture<ListObjectsV2Response>` | Async list | Non-blocking pagination |

### 🪣 **Bucket Operations (Async)**

| Method | Return Type | Purpose |
|--------|-------------|---------|
| `createBucket(CreateBucketRequest)` | `CompletableFuture<CreateBucketResponse>` | Async bucket creation |
| `listBuckets()` | `CompletableFuture<ListBucketsResponse>` | Async bucket listing |
| `deleteBucket(DeleteBucketRequest)` | `CompletableFuture<DeleteBucketResponse>` | Async bucket deletion |
| `getBucketLocation(GetBucketLocationRequest)` | `CompletableFuture<GetBucketLocationResponse>` | Async region lookup |

### 🧩 **Multipart Upload (Async)**

| Method | Return Type | Purpose |
|--------|-------------|---------|
| `createMultipartUpload(CreateMultipartUploadRequest)` | `CompletableFuture<CreateMultipartUploadResponse>` | Async multipart initialization |
| `uploadPart(UploadPartRequest, AsyncRequestBody)` | `CompletableFuture<UploadPartResponse>` | Async part upload |
| `completeMultipartUpload(CompleteMultipartUploadRequest)` | `CompletableFuture<CompleteMultipartUploadResponse>` | Async multipart finalization |
| `abortMultipartUpload(AbortMultipartUploadRequest)` | `CompletableFuture<AbortMultipartUploadResponse>` | Async multipart abortion |

---

## 🔧 Advanced Features

### 🔹 **Presigned URLs**

Use `S3Presigner` for temporary access without AWS credentials:

```java
S3Presigner presigner = S3Presigner.create();

// Presigned GET URL (download)
PresignedGetObjectRequest presignedGet = presigner.presignGetObject(
    GetObjectPresignRequest.builder()
        .signatureDuration(Duration.ofMinutes(10))
        .getObjectRequest(GetObjectRequest.builder()
            .bucket("my-bucket")
            .key("file.txt")
            .build())
        .build());

// Presigned PUT URL (upload)
PresignedPutObjectRequest presignedPut = presigner.presignPutObject(
    PutObjectPresignRequest.builder()
        .signatureDuration(Duration.ofMinutes(10))
        .putObjectRequest(PutObjectRequest.builder()
            .bucket("my-bucket")
            .key("file.txt")
            .build())
        .build());
```

### 🔹 **Transfer Manager**

High-level API for optimized transfers:

```java
S3TransferManager transferManager = S3TransferManager.create();

// Upload with automatic multipart
UploadFileRequest uploadRequest = UploadFileRequest.builder()
    .putObjectRequest(PutObjectRequest.builder()
        .bucket("my-bucket")
        .key("large-file.zip")
        .build())
    .source(Paths.get("local-file.zip"))
    .build();

FileUpload upload = transferManager.uploadFile(uploadRequest);
upload.completionFuture().join(); // Wait for completion
```

### 🔹 **Batch Operations**

Process multiple objects efficiently:

```java
// Create batch job for bulk operations
CreateJobRequest createJobRequest = CreateJobRequest.builder()
    .accountId("123456789012")
    .operation(JobOperation.builder()
        .s3PutObjectCopy(S3CopyObjectOperation.builder()
            .targetResource("arn:aws:s3:::destination-bucket")
            .cannedAccessControlList(S3CannedAccessControlList.PRIVATE)
            .build())
        .build())
    .manifest(JobManifest.builder()
        .spec(JobManifestSpec.builder()
            .format(JobManifestFormat.S3_BATCH_OPERATIONS_CSV_20180820)
            .fields(JobManifestFieldName.BUCKET, JobManifestFieldName.KEY)
            .build())
        .location(JobManifestLocation.builder()
            .objectArn("arn:aws:s3:::manifest-bucket/manifest.csv")
            .eTag("example-etag")
            .build())
        .build())
    .priority(10)
    .roleArn("arn:aws:iam::123456789012:role/batch-operations-role")
    .build();
```

---

## 🧠 Production Best Practices

### 📊 **Performance Optimization**

* **Multipart Upload**: Use for files > 100MB, required for files > 5GB
* **Parallel Operations**: Leverage `S3AsyncClient` for concurrent operations
* **Connection Pooling**: Configure `maxConcurrency` in `S3AsyncClientBuilder`
* **Request Compression**: Enable gzip compression for text content
* **Intelligent Tiering**: Use for unpredictable access patterns

### 🔒 **Security Best Practices**

* **IAM Roles**: Use IAM roles instead of hardcoded credentials
* **Bucket Policies**: Implement least-privilege access policies
* **Encryption**: Enable default bucket encryption (SSE-S3 or SSE-KMS)
* **Access Logging**: Enable S3 access logging for audit trails
* **Public Access**: Block public access unless explicitly required
* **MFA Delete**: Enable MFA delete for versioned buckets

### 🔄 **Retry and Error Handling**

```java
// Configure retry policy
S3Client s3Client = S3Client.builder()
    .region(Region.US_WEST_2)
    .overrideConfiguration(ClientOverrideConfiguration.builder()
        .retryPolicy(RetryPolicy.builder()
            .numRetries(3)
            .retryCondition(RetryCondition.defaultRetryCondition())
            .backoffStrategy(BackoffStrategy.defaultStrategy())
            .build())
        .build())
    .build();
```

### 📈 **Monitoring and Logging**

* **CloudTrail**: Enable API call logging
* **CloudWatch Metrics**: Monitor request metrics and error rates
* **S3 Access Logs**: Track object-level access patterns
* **Cost Optimization**: Use S3 Storage Lens for cost analysis

### 🌐 **Regional Considerations**

* **Cross-Region Replication**: Automatic replication to other regions
* **Transfer Acceleration**: Use S3 Transfer Acceleration for global uploads
* **Regional Endpoints**: Use region-specific endpoints for better performance
* **Data Residency**: Ensure compliance with data locality requirements

### 🔧 **Configuration Tips**

```java
// Optimized S3 client configuration
S3Client s3Client = S3Client.builder()
    .region(Region.US_WEST_2)
    .credentialsProvider(DefaultCredentialsProvider.create())
    .overrideConfiguration(ClientOverrideConfiguration.builder()
        .putAdvancedOption(SdkAdvancedClientOption.USER_AGENT_PREFIX, "MyApp/1.0")
        .build())
    .httpClientBuilder(ApacheHttpClient.builder()
        .maxConnections(100)
        .socketTimeout(Duration.ofSeconds(30))
        .connectionTimeout(Duration.ofSeconds(10))
        .build())
    .build();

// Async client with custom executor
S3AsyncClient s3AsyncClient = S3AsyncClient.builder()
    .region(Region.US_WEST_2)
    .httpClientBuilder(NettyNioAsyncHttpClient.builder()
        .maxConcurrency(50)
        .maxPendingConnectionAcquires(10000)
        .connectionTimeout(Duration.ofSeconds(30))
        .build())
    .build();
```

### 📋 **Common Patterns**

1. **Pagination**: Always handle paginated responses for list operations
2. **Etag Validation**: Use ETags for conditional operations and caching
3. **Metadata Management**: Store application metadata in object metadata
4. **Lifecycle Management**: Implement automated data lifecycle policies
5. **Cost Monitoring**: Regular review of storage costs and usage patterns

---

## 🚨 Common Pitfalls to Avoid

* **Large Objects**: Don't use single-part upload for files > 100MB
* **Hot Spotting**: Avoid sequential key naming patterns
* **Error Handling**: Always handle `NoSuchKey` and `AccessDenied` exceptions
* **Resource Cleanup**: Always close clients and streams
* **Rate Limiting**: Implement exponential backoff for retry logic
* **Versioning Costs**: Monitor costs when versioning is enabled
* **Public Access**: Never make buckets public without explicit business need

---

## 📚 Additional Resources

* [AWS S3 Developer Guide](https://docs.aws.amazon.com/s3/latest/dev/)
* [SDK Performance Best Practices](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/best-practices.html)
* [S3 Pricing Calculator](https://calculator.aws/)
* [AWS SDK 2.0 Migration Guide](https://docs.aws.amazon.com/sdk-for-java/latest/migration-guide/)