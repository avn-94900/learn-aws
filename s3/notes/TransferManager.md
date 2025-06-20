
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

### ✅ Code Example Using `TransferManager`

```java
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import com.amazonaws.services.s3.transfer.TransferManager;
import com.amazonaws.services.s3.transfer.TransferManagerBuilder;
import com.amazonaws.services.s3.transfer.Upload;

import java.io.File;

public class S3UploadExample {
    public static void main(String[] args) throws Exception {
        BasicAWSCredentials awsCreds = new BasicAWSCredentials("ACCESS_KEY", "SECRET_KEY");

        AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
            .withRegion("ap-south-1")
            .withCredentials(new AWSStaticCredentialsProvider(awsCreds))
            .build();

        TransferManager transferManager = TransferManagerBuilder.standard()
            .withS3Client(s3Client)
            .withMultipartUploadThreshold((long) (5 * 1024 * 1024)) // 5 MB
            .build();

        File file = new File("path/to/largefile.zip");

        // ObjectMetadata metadata = new ObjectMetadata();
        // metadata.setContentType("application/zip"); 

        // Start upload
        Upload upload = transferManager.upload("your-bucket-name", "largefile.zip", file);

        // Wait for the upload to finish
        upload.waitForCompletion();

        System.out.println("Upload complete.");

        transferManager.shutdownNow();
    }
}
```

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
