A **pre-signed URL** is a secure, time-limited URL that allows users to access a private object in an Amazon S3 bucket without requiring them to have AWS credentials. It can be used for both **uploading (PUT)** and **downloading (GET)** files to/from S3.

---

### ✅ Use Cases of Pre-Signed URLs

* Temporary file sharing (e.g., send a download link that expires in 10 minutes)
* Secure file uploads directly to S3 from clients (bypassing your server)
* Controlling access without exposing AWS credentials

---

### 🔐 Key Properties of Pre-Signed URLs

* Includes AWS access credentials as query parameters
* Has an **expiration time**
* Limited to a **specific operation** (GET, PUT, DELETE)
* Can only access **one specific object**

---

## 🛠 How to Generate a Pre-Signed URL in Spring Boot (Using AWS SDK v2 or v1)

### ✅ Option 1: AWS SDK for Java **v2** (Recommended)

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
            .region(Region.AP_SOUTH_1)  // use your region
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

---

### ✅ Option 2: AWS SDK for Java **v1**

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

---

### 🧪 Sample Output

```
https://your-bucket.s3.ap-south-1.amazonaws.com/file.txt?X-Amz-Algorithm=...
```

---

### 🧾 Tips

* Do **not expose** AWS credentials to frontend code.
* Set minimum necessary permissions in the IAM policy.
* Use `HttpMethod.PUT` for uploads and `HttpMethod.GET` for downloads.

---

Would you like an example with **upload (PUT)** or want to **return the URL via a REST controller**?
