# 02 - S3 Access Control

S3 provides multiple layers of access control. Understanding which mechanism to use and when is key for building secure applications.

---

## IAM Policies

Identity-based permissions attached to AWS users, groups, or roles.

- Controls which S3 actions a principal can perform
- Written in JSON using the AWS Policy Language
- Follows the **least privilege** principle — grant only what is needed
- Applied at the identity level (not on the bucket itself)

**Example IAM Policy — Read-Only Access to a Specific Bucket**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/*"
      ]
    }
  ]
}
```

---

## Bucket Policies

Resource-based policies attached directly to an S3 bucket.

- Apply to the bucket and all objects within it
- Can grant access to other AWS accounts and services (cross-account)
- Maximum size: 20 KB per policy
- Written in JSON using the AWS Policy Language

**Policy Elements**

| Element   | Description                                              |
|-----------|----------------------------------------------------------|
| Principal | Who can access (AWS account, IAM user, service, etc.)   |
| Action    | What operations are allowed (e.g., `s3:GetObject`)      |
| Resource  | Which buckets or objects (specified as ARNs)             |
| Effect    | `Allow` or `Deny`                                        |
| Condition | Optional — restrict access by IP, time, MFA, etc.       |

**Example Bucket Policy — Public Read Access**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

---

## Access Control Lists (ACLs)

XML-based access control mechanism for buckets and individual objects.

**ACL Permissions**

| Permission  | Bucket Effect                    | Object Effect          |
|-------------|----------------------------------|------------------------|
| READ        | List objects in the bucket       | Read the object data   |
| WRITE       | Create, overwrite, delete objects| Not applicable         |
| READ_ACP    | Read the bucket ACL              | Read the object ACL    |
| WRITE_ACP   | Write the bucket ACL             | Write the object ACL   |
| FULL_CONTROL| All of the above                 | All of the above       |

> **Recommendation:** Prefer bucket policies or IAM policies over ACLs for new implementations. ACLs are a legacy mechanism with limited flexibility.

---

## S3 Access Points

Simplified access management for shared datasets.

- Named network endpoints attached to a bucket
- Each access point has its own IAM policy
- Useful when multiple applications or teams share one bucket
- Can be restricted to a specific VPC for private access

---

## Pre-Signed URLs

Secure, time-limited URLs that allow temporary access to private S3 objects without requiring AWS credentials.

**Use Cases**

- Share a private file with a user for a limited time (e.g., 10-minute download link)
- Allow direct client-side uploads to S3 without routing through your server
- Grant access to a specific object without changing bucket permissions

**Key Properties**

- Credentials are embedded as query parameters in the URL
- Tied to a specific object and a specific operation
- Has a configurable expiration time
- Supported operations: `GET` (download), `PUT` (upload), `DELETE`, `HEAD`

> **Security note:** Do not expose AWS access keys in frontend code. Always generate pre-signed URLs on the server side.

**Generate Pre-Signed URL — AWS SDK v2 (Java)**

```java
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.presigner.S3Presigner;
import software.amazon.awssdk.services.s3.model.GetObjectRequest;
import software.amazon.awssdk.services.s3.presigner.model.GetObjectPresignRequest;

import java.net.URL;
import java.time.Duration;

public class PreSignedUrlExample {

    public URL generateDownloadUrl(String bucketName, String objectKey) {
        try (S3Presigner presigner = S3Presigner.builder()
                .region(Region.AP_SOUTH_1)
                .build()) {

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
}
```

**Generate Pre-Signed URL — AWS SDK v1 (Java)**

```java
import com.amazonaws.HttpMethod;
import com.amazonaws.services.s3.AmazonS3;
import java.net.URL;
import java.util.Date;

public class PreSignedUrlExampleV1 {

    private final AmazonS3 s3Client;

    public PreSignedUrlExampleV1(AmazonS3 s3Client) {
        this.s3Client = s3Client;
    }

    public URL generateDownloadUrl(String bucketName, String objectKey) {
        Date expiration = new Date(System.currentTimeMillis() + 1000L * 60 * 10); // 10 minutes
        return s3Client.generatePresignedUrl(bucketName, objectKey, expiration, HttpMethod.GET);
    }
}
```

**Sample Output**

```
https://your-bucket.s3.ap-south-1.amazonaws.com/file.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...
```

---

## Access Control Decision Order

When S3 evaluates a request, it applies the following logic:

1. If any explicit **Deny** exists → **Deny**
2. If an explicit **Allow** exists (IAM policy, bucket policy, or ACL) → **Allow**
3. Otherwise → **Deny** (implicit deny)

---

## Summary — Which Mechanism to Use?

| Scenario                                | Recommended Mechanism       |
|-----------------------------------------|-----------------------------|
| Control access for IAM users/roles      | IAM Policies                |
| Control access at the bucket level      | Bucket Policies             |
| Cross-account access                    | Bucket Policies             |
| Temporary access for unauthenticated users | Pre-Signed URLs          |
| Shared bucket with multiple applications| S3 Access Points            |
| Legacy object-level permissions         | ACLs (avoid for new setups) |
