# AWS SDK 2.0 S3 Client Methods Reference

AWS SDK 2.0 provides both synchronous (S3Client) and asynchronous (S3AsyncClient) interfaces for S3 operations.

---

## Synchronous S3Client Methods

### Object Operations

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| putObject | Upload a file | bucket, key, contentType, metadata | Use RequestBody.fromFile() for files |
| getObject | Download a file | bucket, key, versionId | Use ResponseTransformer.toFile() for direct download |
| deleteObject | Delete a file | bucket, key, versionId | Returns DeleteObjectResponse with deletion info |
| copyObject | Copy object between buckets/keys | sourceBucket, sourceKey, destinationBucket, destinationKey | Supports server-side copy (no data transfer) |
| listObjectsV2 | List objects in bucket | bucket, prefix, delimiter, maxKeys, continuationToken | Paginated response, use isTruncated() to check for more |
| headObject | Get object metadata only | bucket, key, versionId | Lightweight operation, no data transfer |
| getObjectAcl | Get object ACL | bucket, key, versionId | Returns permissions and ownership info |
| putObjectAcl | Set object ACL | bucket, key, acl | Use predefined ACLs or custom grants |
| getObjectAttributes | Get specific object attributes | bucket, key, objectAttributes | Selective metadata retrieval |
| restoreObject | Restore archived object | bucket, key, restoreRequest | For Glacier/Deep Archive objects |

### Bucket Operations

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| createBucket | Create new bucket | bucket, createBucketConfiguration | Region-specific configuration required |
| listBuckets | List all buckets | None | Returns all buckets owned by authenticated user |
| deleteBucket | Delete empty bucket | bucket | Bucket must be empty before deletion |
| getBucketAcl | Get bucket ACL | bucket | Returns bucket-level permissions |
| putBucketAcl | Set bucket ACL | bucket, acl | Configure bucket-level access |
| getBucketLocation | Get bucket region | bucket | Returns AWS region of the bucket |
| getBucketVersioning | Get versioning status | bucket | Returns versioning configuration |
| putBucketVersioning | Enable/disable versioning | bucket, versioningConfiguration | Enable object versioning |
| getBucketPolicy | Get bucket policy | bucket | Returns IAM policy document |
| putBucketPolicy | Set bucket policy | bucket, policy | Apply IAM policy to bucket |
| deleteBucketPolicy | Remove bucket policy | bucket | Removes all bucket policies |
| getBucketCors | Get CORS configuration | bucket | Returns cross-origin resource sharing rules |
| putBucketCors | Set CORS configuration | bucket, corsConfiguration | Configure CORS rules |
| getBucketWebsite | Get website configuration | bucket | Returns static website hosting config |
| putBucketWebsite | Enable static website hosting | bucket, websiteConfiguration | Configure bucket for web hosting |
| getBucketNotification | Get notification config | bucket | Returns event notification settings |
| putBucketNotification | Set notification config | bucket, notificationConfiguration | Configure SNS/SQS/Lambda notifications |

### Multipart Upload Operations

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| createMultipartUpload | Initialize multipart upload | bucket, key, contentType, metadata | Returns uploadId for subsequent operations |
| uploadPart | Upload single part | bucket, key, uploadId, partNumber | Part numbers: 1-10,000, min 5MB except last |
| completeMultipartUpload | Finalize multipart upload | bucket, key, uploadId, multipartUpload | Requires list of part ETags |
| abortMultipartUpload | Cancel multipart upload | bucket, key, uploadId | Cleans up uploaded parts |
| listParts | List uploaded parts | bucket, key, uploadId | Returns parts with ETags and sizes |
| listMultipartUploads | List ongoing uploads | bucket, prefix | Find incomplete multipart uploads |
| uploadPartCopy | Copy part from another object | sourceBucket, sourceKey, destinationBucket, destinationKey, uploadId, partNumber | Server-side copy for multipart |

### Object Versioning

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| listObjectVersions | List all object versions | bucket, key, prefix | Returns all versions including delete markers |
| deleteObject | Delete specific version | bucket, key, versionId | Permanent deletion when version specified |
| putBucketVersioning | Configure versioning | bucket, versioningConfiguration | Enable/suspend versioning |
| getObjectVersion | Get specific version | bucket, key, versionId | Download specific object version |

### Security and Encryption

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| putObject with SSE | Upload with encryption | serverSideEncryption, ssekmsKeyId, sseCustomerKey | Supports SSE-S3, SSE-KMS, SSE-C |
| getObject with SSE | Download encrypted object | sseCustomerKey, sseCustomerAlgorithm | Required for SSE-C objects |
| getBucketEncryption | Get bucket encryption | bucket | Returns default encryption configuration |
| putBucketEncryption | Set bucket encryption | bucket, serverSideEncryptionConfiguration | Configure default encryption |
| getBucketPublicAccessBlock | Get public access settings | bucket | Returns public access block configuration |
| putBucketPublicAccessBlock | Block public access | bucket, publicAccessBlockConfiguration | Prevent accidental public access |

### Lifecycle and Storage Classes

| Method | Purpose | Key Parameters | Notes |
|--------|---------|----------------|-------|
| getBucketLifecycle | Get lifecycle rules | bucket | Returns object lifecycle configuration |
| putBucketLifecycle | Set lifecycle rules | bucket, lifecycleConfiguration | Automate object transitions and deletion |
| putObject with storage class | Upload with specific storage class | storageClass | IA, GLACIER, DEEP_ARCHIVE, etc. |
| copyObject with storage class | Change storage class | storageClass, metadataDirective | Transition objects between storage classes |

---

## Asynchronous S3AsyncClient Methods

### Object Operations (Async)

| Method | Return Type | Purpose | Key Differences |
|--------|-------------|---------|----------------|
| putObject | CompletableFuture<PutObjectResponse> | Async upload | Use AsyncRequestBody.fromFile() or fromPublisher() |
| getObject | CompletableFuture<ResponseT> | Async download | Use AsyncResponseTransformer.toFile() or toPublisher() |
| deleteObject | CompletableFuture<DeleteObjectResponse> | Async delete | Non-blocking deletion |
| copyObject | CompletableFuture<CopyObjectResponse> | Async copy | Server-side async copy |
| headObject | CompletableFuture<HeadObjectResponse> | Async metadata | Lightweight async operation |
| listObjectsV2 | CompletableFuture<ListObjectsV2Response> | Async list | Non-blocking pagination |

### Bucket Operations (Async)

| Method | Return Type | Purpose |
|--------|-------------|---------|
| createBucket | CompletableFuture<CreateBucketResponse> | Async bucket creation |
| listBuckets | CompletableFuture<ListBucketsResponse> | Async bucket listing |
| deleteBucket | CompletableFuture<DeleteBucketResponse> | Async bucket deletion |
| getBucketLocation | CompletableFuture<GetBucketLocationResponse> | Async region lookup |

### Multipart Upload (Async)

| Method | Return Type | Purpose |
|--------|-------------|---------|
| createMultipartUpload | CompletableFuture<CreateMultipartUploadResponse> | Async multipart initialization |
| uploadPart | CompletableFuture<UploadPartResponse> | Async part upload |
| completeMultipartUpload | CompletableFuture<CompleteMultipartUploadResponse> | Async multipart finalization |
| abortMultipartUpload | CompletableFuture<AbortMultipartUploadResponse> | Async multipart abortion |

---

## Advanced Features

### Presigned URLs

Use S3Presigner for temporary access without AWS credentials:

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

URL url = presignedGet.url();
```

### Transfer Manager

High-level API for efficient uploads and downloads:

```java
S3TransferManager transferManager = S3TransferManager.create();

// Upload with progress tracking
Upload upload = transferManager.upload(UploadRequest.builder()
    .putObjectRequest(PutObjectRequest.builder()
        .bucket("my-bucket")
        .key("large-file.zip")
        .build())
    .source(Paths.get("/path/to/file"))
    .build());

upload.completionFuture().join(); // Wait for completion
```

### Batch Operations

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

## Configuration Tips

### Optimized S3 Client Configuration

```java
// Synchronous client
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

### Retry and Error Handling

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

---

## Additional Resources

- AWS S3 Developer Guide: https://docs.aws.amazon.com/s3/latest/dev/
- SDK Performance Best Practices: https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/best-practices.html
- S3 Pricing Calculator: https://calculator.aws/
- AWS SDK 2.0 Migration Guide: https://docs.aws.amazon.com/sdk-for-java/latest/migration-guide/
