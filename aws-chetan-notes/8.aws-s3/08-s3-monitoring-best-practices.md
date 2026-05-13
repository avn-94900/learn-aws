# 08 - S3 Monitoring, CORS, and Best Practices

---

## CORS (Cross-Origin Resource Sharing)

CORS allows web browsers to make requests to S3 from a different domain than the one that served the page.

**When You Need CORS**
- A single-page application (React, Angular, etc.) uploads files directly to S3
- A web app fetches S3 resources from a different domain
- Browser preflight requests (`OPTIONS`) need to be handled

**Configuration Format (XML)**

```xml
<CORSConfiguration>
  <CORSRule>
    <AllowedOrigin>https://example.com</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
    <MaxAgeSeconds>3000</MaxAgeSeconds>
  </CORSRule>
</CORSConfiguration>
```

CORS configuration is stored as an XML document on the bucket and applied via `putBucketCors()`.

---

## Data Transfer Methods

### Upload Methods

| Method               | Use Case                                       | Max Size |
|----------------------|------------------------------------------------|----------|
| Single PUT           | Small files, simple uploads                    | 5 GB     |
| Multipart Upload     | Large files, parallel parts, resumable         | 5 TB     |
| Transfer Acceleration| Global uploads via CloudFront edge locations   | Any      |
| Pre-signed URLs      | Secure client-side uploads without credentials | Any      |
| AWS CLI / SDK        | Programmatic uploads with built-in retry logic | Any      |

### Download Methods

| Method               | Use Case                                       |
|----------------------|------------------------------------------------|
| Direct GET           | Standard object retrieval                      |
| Range Requests       | Download specific byte ranges of an object     |
| Transfer Acceleration| Faster global downloads via CloudFront         |
| Pre-signed URLs      | Temporary secure downloads                     |
| CloudFront Distribution | CDN for globally distributed content        |

---

## Monitoring and Logging

### AWS CloudTrail

Records all S3 API calls for auditing and compliance.

| Event Type         | What It Captures                                 |
|--------------------|--------------------------------------------------|
| Management Events  | Bucket-level operations (create, delete, policy) |
| Data Events        | Object-level operations (put, get, delete) — opt-in |
| Cross-Account      | Supports logging across AWS accounts             |

> Enable data events carefully — they can generate high log volume and cost.

### S3 Server Access Logs

Detailed logs of every HTTP request made to a bucket.

- Delivered asynchronously to a target S3 bucket
- Fields include: requester, operation, HTTP status, error code, response time
- Useful for security audits and analysing access patterns
- Use with Amazon Athena for querying log data

### CloudWatch Metrics

| Metric Category  | Examples                                   |
|------------------|--------------------------------------------|
| Storage Metrics  | `BucketSizeBytes`, `NumberOfObjects`       |
| Request Metrics  | Request counts, error rates, latency       |
| Replication Metrics | Cross-region replication lag and status |

> Request metrics must be enabled per bucket — they are not on by default.

---

## Best Practices

### Naming and Key Design

- Use DNS-compliant bucket names (lowercase, 3–63 characters)
- Avoid sequential key prefixes (e.g., `001-`, `002-`) — they create S3 hotspots at high request rates
- Use logical hierarchical prefixes (e.g., `invoices/2024/jan/`) to organise objects
- Use UUIDs or timestamps in object keys to ensure uniqueness

### Performance

- Use multipart upload for files > 100 MB
- Use `S3AsyncClient` for concurrent, non-blocking operations
- Use Transfer Acceleration for uploads from geographically distant clients
- Use CloudFront as a CDN for read-heavy, globally distributed content
- Configure `maxConnections` in the HTTP client for high-throughput applications

### Cost Optimisation

- Choose the appropriate storage class for your access patterns
- Implement lifecycle rules to transition objects to cheaper classes over time
- Use Intelligent-Tiering for objects with unpredictable access patterns
- Set a lifecycle rule to abort incomplete multipart uploads (e.g., after 7 days)
- Delete unnecessary old versions when versioning is enabled
- Use S3 Storage Lens for cost visibility and usage analysis

### Security

- Enable versioning for critical data buckets
- Enable **MFA Delete** on versioned buckets to prevent accidental permanent deletion
- Enable default bucket encryption (SSE-S3 or SSE-KMS)
- Block public access at the account and bucket level unless explicitly required
- Use IAM roles (not hardcoded keys) for all application access
- Use VPC endpoints for private connectivity to S3 from within your VPC
- Enable CloudTrail data events for sensitive buckets
- Conduct regular access reviews and audit bucket policies

### Error Handling

```java
// Configure retry policy on the S3 client
S3Client s3Client = S3Client.builder()
    .region(Region.AP_SOUTH_1)
    .overrideConfiguration(ClientOverrideConfiguration.builder()
        .retryPolicy(RetryPolicy.builder()
            .numRetries(3)
            .retryCondition(RetryCondition.defaultRetryCondition())
            .backoffStrategy(BackoffStrategy.defaultStrategy())
            .build())
        .build())
    .build();
```

- Always handle `NoSuchKeyException` when retrieving objects
- Check for `S3Exception` with error code `AccessDenied` for permission issues
- Implement exponential backoff for retry logic on throttled requests
- Always close response streams to avoid connection leaks

---

## Local Development — Testing with LocalStack

[LocalStack](https://github.com/localstack/localstack) provides a local AWS S3 emulator for development and testing without real AWS infrastructure.

```bash
# Start LocalStack with S3 support
docker run -d -p 4566:4566 localstack/localstack
```

Configure your S3 client to point to LocalStack:

```java
S3Client s3Client = S3Client.builder()
    .region(Region.US_EAST_1)
    .endpointOverride(URI.create("http://localhost:4566"))
    .credentialsProvider(StaticCredentialsProvider.create(
        AwsBasicCredentials.create("test", "test")))
    .build();
```
