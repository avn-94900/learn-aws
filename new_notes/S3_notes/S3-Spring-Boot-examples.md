# S3 Spring Boot Examples

Practical examples showing S3 operations in Spring Boot applications using AWS SDK.

---

## S3 Object Structure Example

| Component | Description | Example |
| --- | --- | --- |
| Bucket Name | Container name | my-app-documents-prod |
| Object Key | Full path/name | invoices/2024/INV-12345.pdf |
| Size | Object size in bytes | 2048576 (2 MB) |
| ETag | Object integrity hash | "d41d8cd98f00b204e9800998ecf8427e" |
| Storage Class | Current storage tier | STANDARD |
| Last Modified | Modification timestamp | 2024-01-15T10:30:45.000Z |
| Metadata | Custom key-value pairs | Content-Type: application/pdf |

---

## Sample S3 Object (JSON-like format)

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

## S3 Configuration

### S3Config.java

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

### application.yml

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

## Spring Boot S3 Service

### S3DocumentService.java

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

## REST Controller

### DocumentController.java

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

## Pre-Signed URL Generation

### AWS SDK v2 (Recommended)

```java
import software.amazon.awssdk.auth.credentials.AwsBasicCredentials;
import software.amazon.awssdk.auth.credentials.StaticCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.presigner.S3Presigner;
import software.amazon.awssdk.services.s3.model.GetObjectRequest;
import software.amazon.awssdk.services.s3.presigner.model.GetObjectPresignRequest;

import java.net.URL;
import java.time.Duration;

public class S3PreSignedUrlGenerator {

    public URL generatePreSignedUrl(String bucketName, String objectKey) {
        S3Presigner presigner = S3Presigner.builder()
            .region(Region.AP_SOUTH_1)
            .credentialsProvider(StaticCredentialsProvider.create(
                AwsBasicCredentials.create("ACCESS_KEY", "SECRET_KEY")))
            .build();

        GetObjectRequest getObjectRequest = GetObjectRequest.builder()
            .bucket(bucketName)
            .key(objectKey)
            .build();

        GetObjectPresignRequest presignRequest = GetObjectPresignRequest.builder()
            .signatureDuration(Duration.ofMinutes(10))
            .getObjectRequest(getObjectRequest)
            .build();

        return presigner.presignGetObject(presignRequest).url();
    }
}
```

### AWS SDK v1

```java
import com.amazonaws.HttpMethod;
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import java.net.URL;
import java.util.Date;

public class S3PreSignedUrlGeneratorV1 {

    public URL generatePresignedUrl(String bucketName, String objectKey) {
        BasicAWSCredentials credentials = new BasicAWSCredentials("ACCESS_KEY", "SECRET_KEY");

        AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
            .withRegion("ap-south-1")
            .withCredentials(new AWSStaticCredentialsProvider(credentials))
            .build();

        // Set expiration (10 minutes from now)
        Date expiration = new Date(System.currentTimeMillis() + 1000 * 60 * 10);

        return s3Client.generatePresignedUrl(bucketName, objectKey, expiration, HttpMethod.GET);
    }
}
```

### Sample Output

```
https://your-bucket.s3.ap-south-1.amazonaws.com/file.txt?X-Amz-Algorithm=...
```

---

## Tips

- Do not expose AWS credentials to frontend code
- Set minimum necessary permissions in the IAM policy
- Use HttpMethod.PUT for uploads and HttpMethod.GET for downloads
