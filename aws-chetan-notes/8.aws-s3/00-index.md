# AWS S3 Notes — Index

These notes cover Amazon S3 from beginner to production-level, with a focus on Spring Boot / Java integration.

---

| File | Topic | Level |
|------|-------|-------|
| [01-s3-core-concepts.md](01-s3-core-concepts.md) | Buckets, Objects, Keys, Storage Classes, Versioning, Lifecycle | Beginner |
| [02-s3-access-control.md](02-s3-access-control.md) | IAM Policies, Bucket Policies, ACLs, Access Points, Pre-Signed URLs | Beginner–Intermediate |
| [03-s3-encryption.md](03-s3-encryption.md) | SSE-S3, SSE-KMS, SSE-C, Client-Side Encryption with code examples | Intermediate |
| [04-multipart-upload-transfer-manager.md](04-multipart-upload-transfer-manager.md) | Multipart Upload, TransferManager (SDK v1 & v2) | Intermediate |
| [05-s3-springboot-integration.md](05-s3-springboot-integration.md) | S3Config bean, Service layer, REST Controller, full CRUD | Intermediate |
| [06-s3-eks-secure-integration.md](06-s3-eks-secure-integration.md) | IRSA, IAM trust policies, Kubernetes service accounts | Advanced |
| [07-s3-sdk-methods-reference.md](07-s3-sdk-methods-reference.md) | S3Client & S3AsyncClient method tables, client configuration | Intermediate–Advanced |
| [08-s3-monitoring-best-practices.md](08-s3-monitoring-best-practices.md) | CORS, CloudTrail, CloudWatch, performance & cost best practices | Intermediate–Advanced |
| [09-s3-interview-questions.md](09-s3-interview-questions.md) | Interview Q&A from basic to scenario-based | All levels |

---

## Suggested Reading Order

**If you're new to S3:**
Start with `01` → `02` → `03`

**If you're integrating S3 into a Spring Boot project:**
Start with `01` → `05` → `03` → `04`

**If you're preparing for interviews:**
Read all files, then use `09` to self-test

**If you're deploying to EKS / production:**
Focus on `06` + `08` for security and monitoring
