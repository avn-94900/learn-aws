

## Best Way to Integrate S3 in EKS for Spring Boot Applications

### Use IAM Roles for Service Accounts (IRSA)

This is AWS’s recommended and secure method to grant fine-grained permissions (such as access to S3) to your Spring Boot containers running inside EKS.

---

## Steps to Securely Integrate S3 from EKS

### Step 1: Create an IAM Role with S3 Access Policy

Create an IAM role that allows only the actions you need. For example, to allow only read-only access to a specific S3 bucket:

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

### Step 2: Configure IAM Role Trust Policy

The IAM role should trust the EKS OIDC provider, allowing it to be assumed by a Kubernetes service account. The trust policy looks like this:

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
<a href="https://youtu.be/S6Pn8-bAtd8?si=afMhSbOxj6mBTllC" target="_blank" rel="noopener noreferrer">
Configure IAM Role Trust Policy - step by step explanation
</a>

> 

### Step 3: Create a Kubernetes Service Account with Role Annotation

Define a Kubernetes service account in the namespace your Spring Boot app runs in:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-access-sa
  namespace: your-namespace
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<account-id>:role/YourS3AccessRole
```

Apply this service account to your Kubernetes cluster.

### Step 4: Attach Service Account to Your K8s Deployment

In your pod spec or deployment YAML, ensure the application uses this service account:

```yaml
spec:
  serviceAccountName: s3-access-sa
```

### Step 5: Use Default AWS Credentials Provider in Spring Boot

Inside your Spring Boot application, use the default credential provider chain so AWS SDK picks up the temporary credentials injected by IRSA:

```java
AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
    .withRegion(Regions.AP_SOUTH_1)
    .withCredentials(DefaultAWSCredentialsProviderChain.getInstance())
    .build();
```

This allows your app to communicate with S3 without hardcoding credentials.

---

## Why IRSA Is the Recommended Production Setup

* You do not store or distribute access keys or secret keys.
* AWS handles temporary credential injection and automatic rotation.
* The IAM policy grants only the minimal required access.
* Access is logged and auditable through AWS CloudTrail.
* If compromised, credentials are temporary and limited in scope.

---

## Practices to Avoid

* Do not hardcode access keys or secret keys in the application or `application.properties`.
* Avoid passing long-lived credentials via environment variables.
* Avoid using shared IAM users or credentials across multiple applications.

---

## Optional: Use Spring Cloud AWS for Simpler Integration

You can use `spring-cloud-starter-aws` to inject the `AmazonS3` bean automatically:

```xml
<dependency>
  <groupId>io.awspring.cloud</groupId>
  <artifactId>spring-cloud-aws-starter-s3</artifactId>
</dependency>
```

Then simply use:

```java
@Autowired
private AmazonS3 amazonS3;
```

Just ensure the default credentials provider chain is used, which is automatically compatible with IRSA.

---

## Summary

* Use IAM Roles for Service Accounts (IRSA) in EKS for S3 access.
* Attach least-privilege IAM policies to those roles.
* Create Kubernetes service accounts and annotate them with the role.
* Configure your pods to use these service accounts.
* Use the default AWS credential provider in your Spring Boot code.

Let me know if you want help with Terraform scripts, `kubectl` commands, or full YAML manifests to set up this securely.
