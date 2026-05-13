# 09 - S3 Interview Questions (Spring Boot Developer)

Questions are organised from beginner to advanced. Each links back to the relevant topic files.

---

## Basic Level

**1. What is Amazon S3? What are its main features?**

S3 is a fully managed object storage service. Key features: 11 nines durability, unlimited scalable storage, global accessibility, multiple storage classes, fine-grained access control, built-in encryption, and strong consistency since December 2020.

**2. How do you upload a file to S3 using Spring Boot?**

Use `S3Client.putObject(PutObjectRequest, RequestBody)` (SDK v2) or `AmazonS3.putObject(PutObjectRequest)` (SDK v1). For large files, use `TransferManager` (v1) or `S3TransferManager` (v2) to automatically handle multipart upload.

**3. How do you download a file from S3 in Spring Boot?**

Use `s3Client.getObject(GetObjectRequest, ResponseTransformer)`. Call `readAllBytes()` on the response input stream or use `ResponseTransformer.toFile()` to write directly to disk.

**4. How do you list all files in an S3 bucket or folder?**

Use `listObjectsV2(ListObjectsV2Request)` with a `prefix` to simulate folder-based listing. Handle pagination by checking `isTruncated()` and using `continuationToken`.

**5. What permissions or policies are required for an application to access S3?**

An IAM role or policy granting the needed actions (`s3:GetObject`, `s3:PutObject`, `s3:ListBucket`, etc.) on the specific bucket ARN. Follow the least privilege principle.

---

## Spring Boot Integration

**6. How do you configure AWS credentials in a Spring Boot application?**

Use the **default credential provider chain** (`DefaultCredentialsProvider`) — it automatically resolves credentials from environment variables, instance profiles, or IRSA (in EKS). Avoid hardcoding credentials.

```yaml
# application.yml — only use this pattern for local dev, not production
aws:
  s3:
    bucket-name: my-bucket
    region: ap-south-1
```

**7. How do you define an `S3Client` bean?**

```java
@Bean
public S3Client s3Client() {
    return S3Client.builder()
            .region(Region.AP_SOUTH_1)
            .credentialsProvider(DefaultCredentialsProvider.create())
            .build();
}
```

**8. What is the difference between `AmazonS3` and `S3Client`?**

`AmazonS3` is the SDK v1 interface (synchronous only). `S3Client` is the SDK v2 synchronous interface; SDK v2 also provides `S3AsyncClient` for non-blocking operations. SDK v2 has a redesigned API, better performance, and is actively maintained.

**9. How do you handle large file uploads to S3?**

Use multipart upload. `TransferManager` (SDK v1) or `S3TransferManager` (SDK v2) handles this automatically. Both split the file into parts, upload in parallel, and assemble the object.

**10. How do you upload a file via a REST endpoint?**

Accept a `MultipartFile` in the controller, convert it to bytes or an `InputStream`, and pass it to the S3 service layer that calls `putObject`.

---

## Intermediate to Advanced

**11. How do you secure S3 file uploads in a Spring Boot application?**

- Use pre-signed PUT URLs so clients upload directly to S3 without routing through your server
- Enable server-side encryption (SSE-S3 or SSE-KMS)
- Restrict access with least-privilege IAM policies
- Block public access at the bucket level

**12. What is a pre-signed URL and how do you generate one?**

A pre-signed URL is a time-limited, signed URL granting temporary access to a specific S3 object without requiring AWS credentials. Generated server-side using `S3Presigner` (SDK v2) with a `signatureDuration`.

**13. How do you ensure uploaded files are not overwritten?**

Use unique object keys — include a UUID or timestamp in the key. Alternatively, enable versioning on the bucket so every upload creates a new version rather than overwriting.

**14. How do you handle versioning in S3 from your Spring Boot app?**

Enable versioning on the bucket (`putBucketVersioning`). When downloading, specify a `versionId` to retrieve a specific version. To list all versions, use `listObjectVersions`.

**15. How do you delete files from S3 programmatically?**

Use `deleteObject(DeleteObjectRequest)` with the bucket and key. For versioned buckets, specify the `versionId` for permanent deletion; omitting it creates a delete marker instead.

**16. How do you serve S3 files securely in a Spring Boot app?**

- Generate pre-signed GET URLs with a short expiration (e.g., 10–60 minutes)
- Use CloudFront with signed URLs or signed cookies for large-scale content delivery

**17. How do you monitor and handle upload failures or retries?**

Configure retry policy on the `S3Client` with `numRetries` and an exponential backoff strategy. Use `upload.waitForCompletion()` to catch `AmazonClientException` from `TransferManager`. Log errors and implement dead-letter handling for critical uploads.

---

## Scenario-Based

**18. Users need to upload profile pictures. How do you implement this?**

1. Validate file type and size on the server
2. Generate a unique key: `users/{userId}/avatar/{uuid}.jpg`
3. Generate a pre-signed PUT URL and return it to the client
4. Client uploads directly to S3 using the pre-signed URL
5. On success, save the object key (or public URL) to your database

**19. How do you restrict S3 file access per user in a multi-tenant system?**

- Organise objects by user: `users/{userId}/files/{filename}`
- Generate pre-signed URLs per user — each URL is scoped to that user's object
- Use S3 Access Points with VPC restrictions for internal services
- Never expose other users' keys

**20. Your application handles 1 GB video uploads. How do you handle this?**

- Use multipart upload via `TransferManager` or `S3TransferManager`
- Generate a pre-signed PUT URL for client-side direct upload (bypass your server)
- Track upload progress using `ProgressListener`
- Set a lifecycle rule to abort incomplete multipart uploads after 7 days

---

## Bonus / DevOps

**21. How do you test S3 integration locally?**

Use [LocalStack](https://github.com/localstack/localstack). It provides a local S3 emulator accessible on `http://localhost:4566`. Override the endpoint in your S3 client configuration during testing.

**22. AWS SDK v1 vs v2 — what are the main differences?**

| Aspect              | SDK v1                    | SDK v2                          |
|---------------------|---------------------------|---------------------------------|
| Client              | `AmazonS3`                | `S3Client` / `S3AsyncClient`    |
| Async Support       | Limited                   | First-class with `CompletableFuture` |
| HTTP Client         | Apache HttpClient (default)| Netty (async), Apache (sync)   |
| Credentials        | `AWSCredentialsProvider`  | `AwsCredentialsProvider`        |
| Maintenance        | Maintenance mode           | Actively developed              |
| TransferManager    | `com.amazonaws.services.s3.transfer` | `software.amazon.awssdk.transfer.s3` |

**23. How do you configure lifecycle policies for automated cleanup?**

Use `putBucketLifecycleConfiguration` with rules that define transitions (e.g., move to Glacier after 90 days) and expirations (delete after 365 days). Rules can target the entire bucket, a key prefix, or object tags.

**24. What are alternatives to S3? When would you not use it?**

Alternatives: Google Cloud Storage, Azure Blob Storage, MinIO (self-hosted). You might avoid S3 if you need: very low latency block storage (use EBS), structured querying of data (use a database), or if data residency regulations prohibit cloud storage in your region.
