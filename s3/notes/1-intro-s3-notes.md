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

<br/><br/>

---
Here's a clear breakdown with **realistic examples** showing what **S3 operations look like** — both **programmatically** and via **AWS SDK** (Java/Spring Boot).

---

## 📦 **S3 Object Operations**

### ✅ **Object Structure Example**

| Component                    | Description                 | Example                                |
| ---------------------------- | --------------------------- | -------------------------------------- |
| **Bucket Name**              | Container name              | `my-app-documents-prod`                |
| **Object Key**               | Full path/name              | `invoices/2024/INV-12345.pdf`         |
| **Size**                     | Object size in bytes        | `2048576` (2 MB)                       |
| **ETag**                     | Object integrity hash       | `"d41d8cd98f00b204e9800998ecf8427e"`   |
| **Storage Class**            | Current storage tier        | `STANDARD`                             |
| **Last Modified**            | Modification timestamp      | `2024-01-15T10:30:45.000Z`            |
| **Metadata**                 | Custom key-value pairs      | `Content-Type: application/pdf`        |

---

## 🔍 **Sample S3 Object (JSON-like format)**

```json
{
  "Key": "invoices/2024/INV-12345.pdf",
  "LastModified": "2024-01-15T10:30:45.000Z",
  "ETag": "\"d41d8cd98f00b204e9800998ecf8427e\"",
  "Size": 2048576,
  "StorageClass": "STANDARD",
  "Owner": {
    "DisplayName": "mycompany",
    "ID": "bcaf1ffd86f41161ca5fb16fd081034f"
  },
  "Metadata": {
    "content-type": "application/pdf",
    "x-amz-meta-department": "finance",
    "x-amz-meta-uploaded-by": "john.doe@company.com"
  }
}
```

---

## 🧪 **Spring Boot S3 Service Example**

```java
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.*;
import software.amazon.awssdk.core.sync.RequestBody;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.time.LocalDateTime;
import java.util.Map;
import java.util.UUID;

@Service
public class S3DocumentService {

    private final S3Client s3Client;
    private final String bucketName = "my-app-documents-prod";

    public S3DocumentService(S3Client s3Client) {
        this.s3Client = s3Client;
    }

    // Upload file with metadata
    public String uploadDocument(MultipartFile file, String department, String userId) {
        try {
            String key = generateDocumentKey(file.getOriginalFilename(), department);
            
            Map<String, String> metadata = Map.of(
                "department", department,
                "uploaded-by", userId,
                "upload-date", LocalDateTime.now().toString(),
                "original-filename", file.getOriginalFilename()
            );

            PutObjectRequest putRequest = PutObjectRequest.builder()
                    .bucket(bucketName)
                    .key(key)
                    .contentType(file.getContentType())
                    .metadata(metadata)
                    .storageClass(StorageClass.STANDARD)
                    .build();

            s3Client.putObject(putRequest, RequestBody.fromBytes(file.getBytes()));
            
            System.out.println("Document uploaded: " + key);
            return key;

        } catch (IOException e) {
            throw new RuntimeException("Failed to upload document", e);
        }
    }

    // Download file
    public byte[] downloadDocument(String key) {
        try {
            GetObjectRequest getRequest = GetObjectRequest.builder()
                    .bucket(bucketName)
                    .key(key)
                    .build();

            return s3Client.getObject(getRequest).readAllBytes();

        } catch (Exception e) {
            throw new RuntimeException("Failed to download document: " + key, e);
        }
    }

    // List documents with prefix
    public List<S3Object> listDocuments(String department) {
        String prefix = "documents/" + department + "/";
        
        ListObjectsV2Request listRequest = ListObjectsV2Request.builder()
                .bucket(bucketName)
                .prefix(prefix)
                .maxKeys(100)
                .build();

        ListObjectsV2Response response = s3Client.listObjectsV2(listRequest);
        return response.contents();
    }

    // Get object metadata
    public Map<String, String> getDocumentMetadata(String key) {
        HeadObjectRequest headRequest = HeadObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();

        HeadObjectResponse response = s3Client.headObject(headRequest);
        return response.metadata();
    }

    // Generate presigned URL for temporary access
    public String generatePresignedUrl(String key, int expirationMinutes) {
        GetObjectPresignRequest presignRequest = GetObjectPresignRequest.builder()
                .signatureDuration(Duration.ofMinutes(expirationMinutes))
                .getObjectRequest(GetObjectRequest.builder()
                        .bucket(bucketName)
                        .key(key)
                        .build())
                .build();

        return s3Client.utilities().getUrl(presignRequest).toString();
    }

    // Delete document
    public void deleteDocument(String key) {
        DeleteObjectRequest deleteRequest = DeleteObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();

        s3Client.deleteObject(deleteRequest);
        System.out.println("Document deleted: " + key);
    }

    // Copy document to another location
    public String copyDocument(String sourceKey, String destinationKey) {
        CopyObjectRequest copyRequest = CopyObjectRequest.builder()
                .sourceBucket(bucketName)
                .sourceKey(sourceKey)
                .destinationBucket(bucketName)
                .destinationKey(destinationKey)
                .build();

        s3Client.copyObject(copyRequest);
        System.out.println("Document copied from " + sourceKey + " to " + destinationKey);
        return destinationKey;
    }

    private String generateDocumentKey(String originalFilename, String department) {
        String timestamp = LocalDateTime.now().toString().replace(":", "-");
        String uniqueId = UUID.randomUUID().toString().substring(0, 8);
        return String.format("documents/%s/%s-%s-%s", 
                            department, timestamp, uniqueId, originalFilename);
    }
}
```

---

## 🔐 **REST Controller Example**

```java
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/api/documents")
public class DocumentController {

    private final S3DocumentService s3Service;

    public DocumentController(S3DocumentService s3Service) {
        this.s3Service = s3Service;
    }

    @PostMapping("/upload")
    public ResponseEntity<Map<String, String>> uploadDocument(
            @RequestParam("file") MultipartFile file,
            @RequestParam("department") String department,
            @RequestParam("userId") String userId) {
        
        String documentKey = s3Service.uploadDocument(file, department, userId);
        
        return ResponseEntity.ok(Map.of(
            "message", "Document uploaded successfully",
            "key", documentKey,
            "size", String.valueOf(file.getSize())
        ));
    }

    @GetMapping("/download/{department}/{filename}")
    public ResponseEntity<byte[]> downloadDocument(
            @PathVariable String department,
            @PathVariable String filename) {
        
        String key = "documents/" + department + "/" + filename;
        byte[] documentData = s3Service.downloadDocument(key);
        
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, 
                       "attachment; filename=\"" + filename + "\"")
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .body(documentData);
    }

    @GetMapping("/list/{department}")
    public ResponseEntity<List<DocumentInfo>> listDocuments(
            @PathVariable String department) {
        
        List<S3Object> objects = s3Service.listDocuments(department);
        
        List<DocumentInfo> documents = objects.stream()
                .map(obj -> new DocumentInfo(
                    obj.key(),
                    obj.size(),
                    obj.lastModified().toString(),
                    obj.storageClass().toString()
                ))
                .collect(Collectors.toList());
        
        return ResponseEntity.ok(documents);
    }

    @GetMapping("/presigned-url/{department}/{filename}")
    public ResponseEntity<Map<String, String>> getPresignedUrl(
            @PathVariable String department,
            @PathVariable String filename,
            @RequestParam(defaultValue = "60") int expirationMinutes) {
        
        String key = "documents/" + department + "/" + filename;
        String presignedUrl = s3Service.generatePresignedUrl(key, expirationMinutes);
        
        return ResponseEntity.ok(Map.of(
            "url", presignedUrl,
            "expiresIn", expirationMinutes + " minutes"
        ));
    }

    record DocumentInfo(String key, Long size, String lastModified, String storageClass) {}
}
```

---

## ⚙️ **Configuration Example**

```java
import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class S3Config {

    @Bean
    public S3Client s3Client() {
        return S3Client.builder()
                .region(Region.AP_SOUTH_1) // Mumbai region
                .credentialsProvider(DefaultCredentialsProvider.create())
                .build();
    }
}
```

---

## 📋 **Sample application.yml**

```yaml
aws:
  s3:
    bucket-name: my-app-documents-prod
    region: ap-south-1
  credentials:
    access-key: ${AWS_ACCESS_KEY_ID}
    secret-key: ${AWS_SECRET_ACCESS_KEY}

spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 15MB
```

---