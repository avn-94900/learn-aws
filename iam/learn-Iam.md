# AWS IAM — Compact Notes

---

## 1. What is IAM?

IAM (Identity and Access Management) controls **who** can access **what** in AWS.

| Concept | Meaning |
|---------|---------|
| Authentication | Verifies identity (who are you?) |
| Authorization | Checks permissions (what can you do?) |
| Least Privilege | Grant only the minimum access needed |

---

## 2. Core Components

| Component | What it is | Example |
|-----------|-----------|---------|
| **User** | Individual identity with login credentials | Developer, Admin |
| **Group** | Collection of users sharing same permissions | `dev-team`, `ops-team` |
| **Role** | Temporary identity assumed by services or users | EC2 accessing S3 |
| **Policy** | JSON document defining permissions | Allow `s3:GetObject` |

> Groups cannot be nested. A user can belong to multiple groups.

---

## 3. Policies

Policies define **what actions are allowed or denied** on which resources.

**Policy Structure**

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

| Field | Values | Meaning |
|-------|--------|---------|
| `Effect` | `Allow` / `Deny` | Permit or block |
| `Action` | e.g. `s3:*`, `ec2:StartInstances` | What operation |
| `Resource` | ARN of resource or `*` | Which resource |

**Policy Types**

| Type | Scope | Use Case |
|------|-------|---------|
| AWS Managed | AWS-owned, reusable | `AmazonS3ReadOnlyAccess` |
| Customer Managed | You create, reusable | Custom app permissions |
| Inline | Tied to one identity | One-off specific access |
| Resource-Based | Attached to resource | S3 bucket policy, SQS policy |

---

## 4. Access Methods

| Method | Used By | Credentials |
|--------|---------|-------------|
| Console Login | Humans | Username + Password + MFA |
| Programmatic (CLI/SDK) | Apps, scripts | Access Key ID + Secret Key |
| Temporary Credentials | Roles, federation | STS token (short-lived) |

> Always prefer **Roles + temporary credentials** over long-lived access keys.

Identity-Based Policy vs Resource-Based Policy

| Feature | Identity-Based Policy | Resource-Based Policy |
|--------|----------------------|----------------------|
| Main Purpose | Controls what an identity can do | Controls who can access a resource |
| Attached To | User, Group, or Role | Resource itself |
| Core Question | What can this user/role do? | Who can access this resource? |
| Defines Principal | NO | YES |
| Defines Resource | YES | YES |
| Typical Services | IAM Users, Roles | S3, SQS, SNS |
| Cross-Account Access | Possible but needs role | Very common |
| Used Most By | Applications and users | Shared resources |
| Policy Location | IAM section | Resource settings |
| Real Example | Role allowed to read S3 | Bucket allows external user |

---

## 5. Roles

A Role is an identity with **no permanent credentials** — it issues temporary access via AWS STS.

**How it works**

```
Service/User → AssumeRole → STS issues temp credentials → Access granted
```

| Concept | Meaning |
|---------|---------|
| **Trust Policy** | Defines who can assume the role (e.g. `ec2.amazonaws.com`) |
| **Permission Policy** | Defines what the role can do |
| **STS** | Issues temporary credentials when role is assumed |
| **Cross-Account** | Role in Account A assumed by user in Account B |

---

## 6. Security Features

| Feature | Purpose |
|---------|---------|
| **MFA** | Adds OTP layer on top of password — required for admins |
| **Password Policy** | Enforce strength, expiry, reuse rules |
| **Access Key Rotation** | Rotate regularly, disable unused keys |
| **Permission Boundaries** | Cap the maximum permissions a role/user can have |

---

## 7. Policy Evaluation Logic

```
Explicit DENY  →  wins always
Explicit ALLOW →  granted if no deny
No match       →  default DENY
```

> An explicit `Deny` always overrides any `Allow`.

---

## 8. Federation & SSO

| Option | Use Case |
|--------|---------|
| **Identity Federation** | Login via Google, corporate LDAP, or SAML provider |
| **IAM Identity Center (SSO)** | Centralized login across multiple AWS accounts |
| **SAML 2.0** | Enterprise identity provider integration |

---

## 9. Service Control Policies (SCP)

- Used with **AWS Organizations**
- Restricts what accounts in an org can do — even if their IAM allows it
- Acts as a **guardrail**, not a permission grant

```
SCP (org-level guardrail)
  └── IAM Policy (account-level permission)
       └── Final access = intersection of both
```

---

## 10. AWS Service Integrations

| Service | IAM Mechanism |
|---------|--------------|
| EC2 | Instance Profile (IAM Role attached to instance) |
| Lambda | Execution Role |
| ECS / EKS | Task Role |
| S3 | IAM Policy + Bucket Policy (resource-based) |
| RDS | IAM DB Authentication |
| API Gateway | IAM Auth or resource policy |

---

## 11. Monitoring & Auditing

| Tool | What it does |
|------|-------------|
| **CloudTrail** | Logs every API call — who did what, when |
| **IAM Access Analyzer** | Detects public or cross-account access risks |
| **CloudWatch** | Alerts on suspicious activity or threshold breaches |
| **Credential Report** | Lists all users and their credential status |

---

## 12. Best Practices

| Practice | Why |
|----------|-----|
| Use Roles over Access Keys | No long-lived credentials to leak |
| Enable MFA | Protects against password compromise |
| Least Privilege | Limits blast radius of a breach |
| Use Groups | Easier permission management at scale |
| Rotate credentials | Reduces risk of stale key exposure |
| Monitor with CloudTrail | Full audit trail of all actions |
| Use SCPs in Organizations | Enforce guardrails across all accounts |

---

## 13. IAM Policy Writing Patterns

IAM policies are built from **4 building blocks** — everything else is just a variation.

```json
{
  "Effect": "Allow | Deny",
  "Action": "...",
  "Resource": "...",
  "Condition": {}
}
```

---

### Pattern 1 — Single Action, All Resources

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:StartInstances",
      "Resource": "*"
    }
  ]
}
```

> Simplest valid policy. One action, all resources.

---

### Pattern 2 — Multiple Actions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

> `Action` becomes a list. Still all resources.

---

### Pattern 3 — Wildcard Action (Full Service Access)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

> `s3:*` = all S3 actions. Full S3 access.

---

### Pattern 4 — Specific Resource

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

> Scoped to one bucket only. Much safer than `"*"`.

---

### Pattern 5 — Multiple Resources

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": [
        "arn:aws:s3:::bucket-one/*",
        "arn:aws:s3:::bucket-two/*"
      ]
    }
  ]
}
```

> `Resource` becomes a list. One action, many buckets.

---

### Pattern 6 — Allow + Explicit Deny

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*"
    }
  ]
}
```

> Allow everything, but block delete. **Deny always wins.**

---

### Pattern 7 — Wildcard in Resource Name

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-app-*/*"
    }
  ]
}
```

> Matches `my-app-logs`, `my-app-data`, `my-app-images` — all at once.

---

### Pattern 8 — Condition: IP Restriction

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::secure-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "192.168.1.0/24"
        }
      }
    }
  ]
}
```

> Access allowed only from a specific IP range.

---

### Pattern 9 — Condition: Time-Based Access

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:StartInstances",
      "Resource": "*",
      "Condition": {
        "DateGreaterThan": {
          "aws:CurrentTime": "2024-01-01T09:00:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "2024-01-01T18:00:00Z"
        }
      }
    }
  ]
}
```

> Start EC2 only during working hours.

---

### Pattern 10 — Typical App Policy (DynamoDB)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/Orders"
    }
  ]
}
```

> Read + write on one table only. Real-world app pattern.

---

### Pattern 11 — Deny Without MFA (Condition on Deny)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": "s3:*", "Resource": "*" },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*",
      "Condition": {
        "Bool": { "aws:MultiFactorAuthPresent": "false" }
      }
    }
  ]
}
```

> Delete allowed **only if MFA is active**. Strong security rule.

---

### 8 Core Variations — Quick Reference

| # | Pattern | Key Change |
|---|---------|-----------|
| 1 | Single action | `"Action": "ec2:Start..."` |
| 2 | Multiple actions | `"Action": [...]` |
| 3 | Wildcard action | `"Action": "s3:*"` |
| 4 | Specific resource | `"Resource": "arn:..."` |
| 5 | Multiple resources | `"Resource": [...]` |
| 6 | Allow + Deny | Two statements, Deny wins |
| 7 | Wildcard resource | `"Resource": "arn:...:my-app-*/*"` |
| 8 | Condition | `"Condition": { ... }` |

> Master these 8 patterns and you can read or write any IAM policy in real systems.


<br/>

### below is example for trust policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",

      "Principal": {
        "Service": [
          "ec2.amazonaws.com",
          "ecs-tasks.amazonaws.com"
        ]
      },

      "Action": "sts:AssumeRole"
    }
  ]
}
```
Trust Policy vs Permission Policy

| Feature | Trust Policy | Permission Policy |
|--------|--------------|-------------------|
| Main Purpose | Defines WHO can assume a role | Defines WHAT actions are allowed |
| Core Question Answered | Who can use this role? | What can this identity do? |
| Attached To | Only Roles | Users, Groups, Roles |
| Required For | Role usage | Access to AWS resources |
| Used With | sts:AssumeRole | Service actions (s3, ec2, dynamodb etc.) |
| Defines Principal | YES | NO |
| Defines Actions | Only assume-role actions | Real AWS actions |
| Resource Field | Usually not required | Required |
| Condition Support | YES | YES |
| Mandatory for Role | YES (Role cannot work without it) | YES (Role/User cannot access resources without it) |
| Typical Usage | Allow EC2 to assume role | Allow S3 read/write access |
| Evaluation Time | Before role usage | During resource access |
| Controls | Role assumption | Resource permissions |
| Used in Cross-Account Access | YES (Very important) | YES |
| Common Example | Allow EC2 service to assume role | Allow S3 GetObject access |


<br/>

Resource-Based Policy vs Trust Policy vs Role

| Feature | Resource-Based Policy | Trust Policy | Role |
|--------|----------------------|--------------|------|
| What it is | Policy attached to resource | Policy attached to role | IAM identity |
| Main Purpose | Controls who can access resource | Controls who can assume role | Provides temporary identity |
| Core Question | Who can access this resource? | Who can use this role? | What temporary identity is used? |
| Attached To | Resource (S3, SQS, SNS etc.) | Role only | Created as IAM entity |
| Contains Principal | YES | YES | NO (role contains trust policy) |
| Contains Permissions | YES | NO | YES (via permission policies) |
| Uses sts:AssumeRole | NO | YES | YES |
| Typical Usage | Sharing bucket or queue | Allow EC2 to use role | Give EC2 temporary access |
| Required for Service Roles | NO | YES | YES |
| Real Example | S3 Bucket Policy | EC2 Trust Policy | EC2 IAM Role |