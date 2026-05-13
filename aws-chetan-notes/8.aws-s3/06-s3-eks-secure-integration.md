# 06 - Secure S3 Integration in EKS (Spring Boot)

The recommended approach for granting S3 access to Spring Boot containers running inside EKS is **IAM Roles for Service Accounts (IRSA)**.

IRSA allows Kubernetes pods to assume an IAM role without using hardcoded access keys. AWS injects temporary credentials automatically.

---

## Why IRSA?

- No access keys or secret keys to store or rotate
- AWS handles temporary credential injection and automatic rotation
- Grants only the minimal required permissions (least privilege)
- All access is logged and auditable through AWS CloudTrail
- Temporary credentials limit blast radius if compromised

---

## Step-by-Step Setup

### Step 1 — Create an IAM Policy with Least-Privilege S3 Access

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ReadOnly",
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

### Step 2 — Configure IAM Role Trust Policy

The IAM role must trust the EKS OIDC provider so it can be assumed by a Kubernetes service account:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/oidc.eks.<region>.amazonaws.com/id/<OIDC_ID>"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "oidc.eks.<region>.amazonaws.com/id/<OIDC_ID>:sub": "system:serviceaccount:<namespace>:<service-account-name>"
    }
  }
}
```

### Step 3 — Create a Kubernetes Service Account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-access-sa
  namespace: your-namespace
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<account-id>:role/YourS3AccessRole
```

Apply it:

```bash
kubectl apply -f service-account.yaml
```

### Step 4 — Attach Service Account to Your Deployment

In your Kubernetes deployment spec, reference the service account:

```yaml
spec:
  serviceAccountName: s3-access-sa
```

### Step 5 — Use DefaultCredentialsProvider in Spring Boot

Your application code should use the default credential provider chain so the AWS SDK automatically picks up the IRSA-injected temporary credentials:

```java
import com.amazonaws.auth.DefaultAWSCredentialsProviderChain;
import com.amazonaws.regions.Regions;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;

@Configuration
public class S3Config {

    @Bean
    public AmazonS3 amazonS3() {
        return AmazonS3ClientBuilder.standard()
                .withRegion(Regions.AP_SOUTH_1)
                .withCredentials(DefaultAWSCredentialsProviderChain.getInstance())
                .build();
    }
}
```

No access keys, no secrets — credentials are resolved at runtime through the IRSA mechanism.

---

## Optional — Spring Cloud AWS

If you use `spring-cloud-aws-starter-s3`, the `AmazonS3` bean is auto-configured:

```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter-s3</artifactId>
</dependency>
```

Then inject it directly:

```java
@Autowired
private AmazonS3 amazonS3;
```

Spring Cloud AWS uses the default credentials provider chain automatically, which is fully compatible with IRSA.

---

## Practices to Avoid

- Do not hardcode access keys or secret keys in application code or `application.properties`
- Do not pass long-lived credentials via environment variables
- Do not share IAM users or credentials across multiple applications

---

## Summary

| Step | Action                                                   |
|------|----------------------------------------------------------|
| 1    | Create an IAM policy with least-privilege S3 permissions |
| 2    | Create an IAM role with an OIDC trust policy             |
| 3    | Create a Kubernetes service account annotated with the role ARN |
| 4    | Attach the service account to your pod/deployment        |
| 5    | Use `DefaultCredentialsProvider` in Spring Boot code     |
