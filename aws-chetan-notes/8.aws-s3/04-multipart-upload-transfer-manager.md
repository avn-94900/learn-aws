# 04 - Multipart Upload and TransferManager

---

## Why Multipart Upload?

Multipart upload breaks a large file into smaller parts and uploads them in parallel.

| File Size   | Recommendation                           |
|-------------|------------------------------------------|
| < 5 MB      | Single PUT upload is fine                |
| > 5 MB      | Multipart upload is possible             |
| > 100 MB    | Multipart upload is strongly recommended |
| > 5 GB      | Multipart upload is **mandatory**        |

**Benefits**
- Improved upload speed through parallel part uploads
- Resume a failed upload from the last completed part
- Better network utilization

---

## Multipart Upload Process Flow

```
1. Initiate  →  CreateMultipartUpload → receive Upload ID
2. Upload    →  Upload each part (5 MB – 5 GB each), numbered 1–10,000
3. Complete  →  CompleteMultipartUpload with part ETags → S3 assembles object
4. Abort     →  AbortMultipartUpload on failure → S3 cleans up parts
```

> Incomplete multipart uploads incur storage charges. Use lifecycle rules to auto-abort stale uploads.

---

## TransferManager (AWS SDK v1)

`TransferManager` is a high-level utility that automates multipart uploads internally.

**What It Does**
- Splits large files into parts (default: 5 MB per part)
- Uploads parts in parallel using a thread pool
- Retries failed parts automatically
- Calls `CompleteMultipartUpload` API after all parts finish

**AmazonS3 vs TransferManager**

| Feature                   | `AmazonS3` | `TransferManager`  |
|---------------------------|------------|--------------------|
| Multipart Upload Handling | Manual     | Automatic          |
| Progress Tracking         | Manual     | Built-in           |
| Asynchronous Support      | Limited    | Yes                |
| Parallel Uploads          | No         | Yes                |

---

## Code Example — TransferManager (SDK v1)

```java
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import com.amazonaws.services.s3.transfer.TransferManager;
import com.amazonaws.services.s3.transfer.TransferManagerBuilder;
import com.amazonaws.services.s3.transfer.Upload;
import java.io.File;

public class S3UploadExample {

    public static void main(String[] args) throws Exception {
        AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
                .withRegion("ap-south-1")
                .withCredentials(DefaultAWSCredentialsProviderChain.getInstance())
                .build();

        TransferManager transferManager = TransferManagerBuilder.standard()
                .withS3Client(s3Client)
                .withMultipartUploadThreshold((long) (5 * 1024 * 1024)) // 5 MB
                .build();

        File file = new File("path/to/largefile.zip");
        Upload upload = transferManager.upload("your-bucket-name", "largefile.zip", file);

        upload.waitForCompletion();
        System.out.println("Upload complete.");

        transferManager.shutdownNow();
    }
}
```

---

## TransferManager Configuration Options

| Option                                         | Description                                           |
|------------------------------------------------|-------------------------------------------------------|
| `withMultipartUploadThreshold(long threshold)` | Minimum size (bytes) to trigger multipart upload      |
| `withMinimumUploadPartSize(long size)`         | Size of each individual part                          |
| `withExecutorFactory(...)`                     | Provide a custom thread pool                          |
| `withShutDownThreadPools(...)`                 | Controls thread pool shutdown behaviour               |

---

## SDK Version Note

| SDK Version | Class to Use                                           |
|-------------|--------------------------------------------------------|
| AWS SDK v1  | `com.amazonaws.services.s3.transfer.TransferManager`   |
| AWS SDK v2  | `software.amazon.awssdk.transfer.s3.S3TransferManager` |

---

## Best Practices

- Prefer `TransferManager` (v1) or `S3TransferManager` (v2) over manual multipart logic
- Always abort failed uploads to avoid orphaned part charges
- Add a lifecycle rule to auto-abort incomplete uploads after N days
- Use parallel uploads for better throughput
- Implement retry logic with exponential backoff
