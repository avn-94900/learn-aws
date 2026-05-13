# 05 - S3 Integration with Spring Boot

---

## Maven Dependency (AWS SDK v2)

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <version>2.25.0</version>
</dependency>
```

For Spring Cloud AWS (auto-configures the S3 bean):

```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter-s3</artifactId>
</dependency>
```

---

## S3 Client Configuration Bean

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
                .region(Region.AP_SOUTH_1)
                .credentialsProvider(DefaultCredentialsProvider.create())
                .build();
    }
}
```

> Always use `DefaultCredentialsProvider` in production. Never hardcode access keys.

---

## application.yml

```yaml
aws:
  s3:
    bucket-name: my-app-documents-prod
    region: ap-south-1

spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 15MB
```

---

## S3 Service — Core Operations

```java
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.*;
import software.amazon.awssdk.core.sync.RequestBody;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.time.Duration;
import java.time.LocalDateTime;
import java.util.List;
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
            return key;

        } catch (IOException e) {
            throw new RuntimeException("Failed to upload document", e);
        }
    }

    // Download file as byte array
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

    // List objects by prefix (acts like a folder)
    public List<S3Object> listDocuments(String department) {
        ListObjectsV2Request listRequest = ListObjectsV2Request.builder()
                .bucket(bucketName)
                .prefix("documents/" + department + "/")
                .maxKeys(100)
                .build();

        return s3Client.listObjectsV2(listRequest).contents();
    }

    // Get object metadata only (no data transfer)
    public Map<String, String> getDocumentMetadata(String key) {
        HeadObjectRequest headRequest = HeadObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();
        return s3Client.headObject(headRequest).metadata();
    }

    // Generate a pre-signed download URL
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

    // Delete a document
    public void deleteDocument(String key) {
        DeleteObjectRequest deleteRequest = DeleteObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();
        s3Client.deleteObject(deleteRequest);
    }

    // Copy a document within the same bucket
    public String copyDocument(String sourceKey, String destinationKey) {
        CopyObjectRequest copyRequest = CopyObjectRequest.builder()
                .sourceBucket(bucketName)
                .sourceKey(sourceKey)
                .destinationBucket(bucketName)
                .destinationKey(destinationKey)
                .build();
        s3Client.copyObject(copyRequest);
        return destinationKey;
    }

    private String generateDocumentKey(String originalFilename, String department) {
        String timestamp = LocalDateTime.now().toString().replace(":", "-");
        String uniqueId = UUID.randomUUID().toString().substring(0, 8);
        return String.format("documents/%s/%s-%s-%s", department, timestamp, uniqueId, originalFilename);
    }
}
```

---

## REST Controller

```java
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

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
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + filename + "\"")
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .body(documentData);
    }

    @GetMapping("/list/{department}")
    public ResponseEntity<List<DocumentInfo>> listDocuments(@PathVariable String department) {
        List<S3Object> objects = s3Service.listDocuments(department);

        List<DocumentInfo> documents = objects.stream()
                .map(obj -> new DocumentInfo(
                        obj.key(),
                        obj.size(),
                        obj.lastModified().toString(),
                        obj.storageClass().toString()))
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

## S3 Object Structure Reference

| Component     | Example                                          |
|---------------|--------------------------------------------------|
| Bucket Name   | `my-app-documents-prod`                          |
| Object Key    | `invoices/2024/INV-12345.pdf`                   |
| Size          | `2048576` (2 MB)                                |
| ETag          | `"d41d8cd98f00b204e9800998ecf8427e"`            |
| Storage Class | `STANDARD`                                       |
| Last Modified | `2024-01-15T10:30:45.000Z`                      |
| Metadata      | `Content-Type: application/pdf`, dept, user etc.|
