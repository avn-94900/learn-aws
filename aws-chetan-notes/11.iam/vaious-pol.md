### Complete IAM Policy Comparison

| Policy Type             | Attached To                | Reusable?    | Controls                     | Example                                  |
| ----------------------- | -------------------------- | ------------ | ---------------------------- | ---------------------------------------- |
| AWS Managed Policy      | User / Group / Role        | ✅ Yes        | What identity can do         | `AmazonS3ReadOnlyAccess`                 |
| Customer Managed Policy | User / Group / Role        | ✅ Yes        | What identity can do         | Custom S3 + DynamoDB permissions         |
| Inline Policy           | Single User / Group / Role | ❌ No         | What identity can do         | Special permission for one role          |
| Resource-Based Policy   | Resource                   | ✅ Yes        | Who can access resource      | S3 Bucket Policy, SQS Policy, SNS Policy |
| Bucket Policy           | S3 Bucket                  | ✅ Yes        | Who can access bucket        | Allow Account B to access bucket         |
| Object ACL              | Individual S3 Object       | ❌ Per Object | Who can access specific file | Make `logo.png` public                   |

---

## 1. AWS Managed Policy

Created and maintained by AWS.

Example:

```text
AmazonS3ReadOnlyAccess
```

Attach to:

```text
User
Role
Group
```

Benefits:

```text
Ready-made
Easy to use
AWS updates automatically
```

---

## 2. Customer Managed Policy

Created by your team.

Example:

```json
{
  "Effect":"Allow",
  "Action":[
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource":"*"
}
```

Benefits:

```text
Reusable
Custom permissions
Best practice for enterprises
```

---

## 3. Inline Policy

Attached directly to one identity.

```text
Role: PaymentRole
```

Policy:

```text
Allow access only to payment bucket
```

Cannot be reused elsewhere.

Benefits:

```text
Very specific access
```

Drawback:

```text
Harder to manage
```

---

## 4. Resource-Based Policy

Attached directly to resource.

Examples:

* S3 Bucket Policy
* SQS Queue Policy
* SNS Topic Policy
* KMS Key Policy

Example:

```text
Bucket Policy
    ↓
Allow Account B
```

Answers:

```text
Who can access this resource?
```

---

## 5. Bucket Policy

A special type of Resource-Based Policy.

Attached to:

```text
Entire Bucket
```

Example:

```text
Bucket: company-files

Allow:
  Account B
  EC2 Role
```

Affects:

```text
All objects in bucket
```

---

## 6. Object ACL

Attached to:

```text
Single File
```

Example:

```text
logo.png
```

Permission:

```text
Public Read
```

Only affects:

```text
logo.png
```

Not:

```text
banner.png
secret.pdf
```

---

# Visual Comparison

```text
S3 Bucket
│
├── Bucket Policy
│      ↓
│   Applies to bucket
│
├── logo.png
│      ↓
│   Object ACL
│
├── banner.jpg
│      ↓
│   Object ACL
│
└── secret.pdf
       ↓
    Object ACL
```

---

# Resource-Based Policies Examples

| AWS Service | Resource Policy Name |
| ----------- | -------------------- |
| S3          | Bucket Policy        |
| SQS         | Queue Policy         |
| SNS         | Topic Policy         |
| KMS         | Key Policy           |
| ECR         | Repository Policy    |
| Lambda      | Function Policy      |

---

# Important Interview Point

Many candidates think:

```text
Resource-Based Policy
=
Bucket Policy
```

This is incorrect.

Relationship:

```text
Resource-Based Policy
        │
        ├── Bucket Policy
        ├── SQS Queue Policy
        ├── SNS Topic Policy
        ├── KMS Key Policy
        └── ECR Repository Policy
```

So:

```text
Bucket Policy is a Resource-Based Policy
But not every Resource-Based Policy is a Bucket Policy
```

---

# Quick Memory Table

| Question                               | Policy Type                             |
| -------------------------------------- | --------------------------------------- |
| What can this user/role do?            | AWS Managed / Customer Managed / Inline |
| Who can access this bucket?            | Bucket Policy                           |
| Who can access this SQS queue?         | Resource-Based Policy                   |
| Who can access this single file?       | Object ACL                              |
| Reusable custom permissions?           | Customer Managed                        |
| AWS provided permissions?              | AWS Managed                             |
| One-time identity-specific permission? | Inline Policy                           |

### Interview Summary

* **AWS Managed** → Created by AWS, reusable.
* **Customer Managed** → Created by you, reusable.
* **Inline** → Attached to one identity only.
* **Resource-Based** → Attached to a resource.
* **Bucket Policy** → Resource-based policy for an S3 bucket.
* **Object ACL** → Permissions on an individual S3 object/file.
