# 07 - AWS SDK v2 S3 Client — Methods Reference

AWS SDK v2 provides both synchronous (`S3Client`) and asynchronous (`S3AsyncClient`) interfaces for S3 operations.

---

## Synchronous S3Client

### Object Operations

| Method                                           | Purpose                        | Key Parameters                                          | Notes                                                   |
|--------------------------------------------------|--------------------------------|---------------------------------------------------------|---------------------------------------------------------|
| `putObject(PutObjectRequest, RequestBody)`       | Upload a file                  | `bucket`, `key`, `contentType`, `metadata`              | Use `RequestBody.fromFile()` for file uploads           |
| `getObject(GetObjectRequest, ResponseTransformer)` | Download a file              | `bucket`, `key`, `versionId`                            | Use `ResponseTransformer.toFile()` for direct file save |
| `deleteObject(DeleteObjectRequest)`              | Delete a file                  | `bucket`, `key`, `versionId`                            | Specify `versionId` to delete a specific version        |
| `copyObject(CopyObjectRequest)`                  | Copy object between locations  | `sourceBucket`, `sourceKey`, `destinationBucket`, `destinationKey` | Server-side copy — no data leaves AWS      |
| `listObjectsV2(ListObjectsV2Request)`            | List objects in a bucket       | `bucket`, `prefix`, `delimiter`, `maxKeys`, `continuationToken` | Paginated; check `isTruncated()` for more pages |
| `headObject(HeadObjectRequest)`                  | Get object metadata only       | `bucket`, `key`, `versionId`                            | Lightweight — no data transfer                          |
| `restoreObject(RestoreObjectRequest)`            | Restore archived object        | `bucket`, `key`, `restoreRequest`                       | For Glacier / Deep Archive objects                      |
| `getObjectAttributes(GetObjectAttributesRequest)`| Get specific object attributes | `bucket`, `key`, `objectAttributes`                     | Selective metadata retrieval                            |

### Bucket Operations

| Method                                           | Purpose                        | Notes                                                   |
|--------------------------------------------------|--------------------------------|---------------------------------------------------------|
| `createBucket(CreateBucketRequest)`              | Create a new bucket            | Region-specific configuration required                  |
| `listBuckets()`                                  | List all buckets               | Returns all buckets owned by the authenticated user     |
| `deleteBucket(DeleteBucketRequest)`              | Delete a bucket                | Bucket must be empty before deletion                    |
| `getBucketVersioning(GetBucketVersioningRequest)`| Get versioning status          | Returns versioning configuration                        |
| `putBucketVersioning(PutBucketVersioningRequest)`| Enable / suspend versioning    | Cannot fully disable once enabled                       |
| `getBucketPolicy(GetBucketPolicyRequest)`        | Get bucket policy              | Returns IAM policy document as JSON string              |
| `putBucketPolicy(PutBucketPolicyRequest)`        | Set bucket policy              | Replaces the existing policy                            |
| `deleteBucketPolicy(DeleteBucketPolicyRequest)`  | Remove bucket policy           | Removes all bucket policies                             |
| `getBucketCors(GetBucketCorsRequest)`            | Get CORS configuration         | Returns cross-origin resource sharing rules             |
| `putBucketCors(PutBucketCorsRequest)`            | Set CORS configuration         | Configure allowed origins, methods, headers             |
| `getBucketEncryption(GetBucketEncryptionRequest)`| Get encryption config          | Returns default encryption configuration                |
| `putBucketEncryption(PutBucketEncryptionRequest)`| Set default encryption         | Configure SSE-S3 or SSE-KMS as bucket default           |
| `putBucketLifecycle(...)`                        | Set lifecycle rules            | Automate transitions and deletions                      |
| `getBucketLifecycle(...)`                        | Get lifecycle rules            | Returns object lifecycle configuration                  |
| `putBucketPublicAccessBlock(...)`                | Block public access            | Prevent accidental public exposure                      |
| `getBucketPublicAccessBlock(...)`                | Get public access settings     | Returns public access block configuration               |
| `getBucketLocation(GetBucketLocationRequest)`    | Get bucket region              | Returns AWS region of the bucket                        |
| `putBucketNotification(...)`                     | Set event notifications        | Configure SNS / SQS / Lambda triggers                   |
| `putBucketWebsite(...)`                          | Enable static website hosting  | Configure index/error documents                         |

### Multipart Upload Operations

| Method                                                   | Purpose                        | Notes                                               |
|----------------------------------------------------------|--------------------------------|-----------------------------------------------------|
| `createMultipartUpload(CreateMultipartUploadRequest)`    | Initiate multipart upload      | Returns `uploadId` for subsequent calls             |
| `uploadPart(UploadPartRequest, RequestBody)`             | Upload a single part           | Part numbers: 1–10,000; minimum 5 MB except last part |
| `completeMultipartUpload(CompleteMultipartUploadRequest)`| Finalize multipart upload      | Pass list of completed parts with ETags             |
| `abortMultipartUpload(AbortMultipartUploadRequest)`      | Cancel multipart upload        | Cleans up uploaded parts and avoids charges         |
| `listParts(ListPartsRequest)`                            | List uploaded parts            | Returns parts with ETags and sizes                  |
| `listMultipartUploads(ListMultipartUploadsRequest)`      | List ongoing uploads           | Find incomplete multipart uploads                   |

### Versioning Operations

| Method                                           | Purpose                        | Notes                                                   |
|--------------------------------------------------|--------------------------------|---------------------------------------------------------|
| `listObjectVersions(ListObjectVersionsRequest)`  | List all versions of objects   | Returns all versions including delete markers           |
| `deleteObject(DeleteObjectRequest)` with versionId | Delete a specific version   | Permanent deletion — no delete marker created           |

---

## Asynchronous S3AsyncClient

All methods return `CompletableFuture<T>` — non-blocking.

### Object Operations (Async)

| Method                                                         | Return Type                              |
|----------------------------------------------------------------|------------------------------------------|
| `putObject(PutObjectRequest, AsyncRequestBody)`                | `CompletableFuture<PutObjectResponse>`   |
| `getObject(GetObjectRequest, AsyncResponseTransformer)`        | `CompletableFuture<ResponseT>`           |
| `deleteObject(DeleteObjectRequest)`                            | `CompletableFuture<DeleteObjectResponse>`|
| `copyObject(CopyObjectRequest)`                                | `CompletableFuture<CopyObjectResponse>`  |
| `headObject(HeadObjectRequest)`                                | `CompletableFuture<HeadObjectResponse>`  |
| `listObjectsV2(ListObjectsV2Request)`                          | `CompletableFuture<ListObjectsV2Response>`|

### Bucket Operations (Async)

| Method                           | Return Type                                  |
|----------------------------------|----------------------------------------------|
| `createBucket(CreateBucketRequest)` | `CompletableFuture<CreateBucketResponse>` |
| `listBuckets()`                  | `CompletableFuture<ListBucketsResponse>`     |
| `deleteBucket(DeleteBucketRequest)` | `CompletableFuture<DeleteBucketResponse>` |

### Multipart Upload (Async)

| Method                                                          | Return Type                                          |
|-----------------------------------------------------------------|------------------------------------------------------|
| `createMultipartUpload(CreateMultipartUploadRequest)`           | `CompletableFuture<CreateMultipartUploadResponse>`   |
| `uploadPart(UploadPartRequest, AsyncRequestBody)`               | `CompletableFuture<UploadPartResponse>`              |
| `completeMultipartUpload(CompleteMultipartUploadRequest)`        | `CompletableFuture<CompleteMultipartUploadResponse>` |
| `abortMultipartUpload(AbortMultipartUploadRequest)`             | `CompletableFuture<AbortMultipartUploadResponse>`    |

---

## Presigned URLs (S3Presigner)

```java
S3Presigner presigner = S3Presigner.create();

// Generate presigned GET URL (download)
PresignedGetObjectRequest presignedGet = presigner.presignGetObject(
    GetObjectPresignRequest.builder()
        .signatureDuration(Duration.ofMinutes(10))
        .getObjectRequest(GetObjectRequest.builder()
            .bucket("my-bucket")
            .key("file.txt")
            .build())
        .build());

URL downloadUrl = presignedGet.url();
```

---

## S3 Transfer Manager (SDK v2)

For large file uploads with automatic multipart management:

```java
S3TransferManager transferManager = S3TransferManager.builder()
    .s3Client(s3AsyncClient)
    .build();

FileUpload upload = transferManager.uploadFile(UploadFileRequest.builder()
    .putObjectRequest(r -> r.bucket("my-bucket").key("large-file.zip"))
    .source(Paths.get("path/to/large-file.zip"))
    .build());

upload.completionFuture().join();
```

---

## Client Configuration Examples

### Synchronous Client with Retry Policy

```java
S3Client s3Client = S3Client.builder()
    .region(Region.AP_SOUTH_1)
    .credentialsProvider(DefaultCredentialsProvider.create())
    .overrideConfiguration(ClientOverrideConfiguration.builder()
        .retryPolicy(RetryPolicy.builder()
            .numRetries(3)
            .retryCondition(RetryCondition.defaultRetryCondition())
            .backoffStrategy(BackoffStrategy.defaultStrategy())
            .build())
        .build())
    .build();
```

### Async Client with Connection Pool Tuning

```java
S3AsyncClient s3AsyncClient = S3AsyncClient.builder()
    .region(Region.AP_SOUTH_1)
    .httpClientBuilder(NettyNioAsyncHttpClient.builder()
        .maxConcurrency(50)
        .maxPendingConnectionAcquires(10000)
        .connectionTimeout(Duration.ofSeconds(30))
        .build())
    .build();
```

---

## Common Pitfalls

- Do not use single-part PUT for files larger than 100 MB
- Avoid sequential key naming (e.g., `0001-`, `0002-`) — it causes hotspotting at high request rates
- Always handle `NoSuchKeyException` and `S3Exception` (check for `AccessDenied` error code)
- Always close clients and response streams when done
- Never make a bucket public without an explicit business requirement
- Monitor costs when versioning is enabled — old versions accumulate silently
