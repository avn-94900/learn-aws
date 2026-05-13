# 07 — Security & Access Control

---

## Security Overview

SQS provides three layers of security:

1. **Authentication** — who is making the request (IAM identity)
2. **Authorization** — what they are allowed to do (IAM policies + queue policies)
3. **Encryption** — protecting message data at rest and in transit

---

## IAM Policies

### Identity-Based Policies (Attached to Users/Roles)

Apply the **least-privilege principle** — grant only the permissions each role actually needs.

**Producer Role Policy** — only allowed to send messages:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sqs:SendMessage",
                "sqs:GetQueueUrl"
            ],
            "Resource": "arn:aws:sqs:*:*:orders-*"
        }
    ]
}
```

**Consumer Role Policy** — only allowed to receive and delete:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:ChangeMessageVisibility"
            ],
            "Resource": "arn:aws:sqs:*:*:orders-*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": "us-east-1"
                }
            }
        }
    ]
}
```

### Full Access Policy (Admin/Operations)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes",
                "sqs:GetQueueUrl"
            ],
            "Resource": "arn:aws:sqs:us-east-1:123456789012:MyQueue"
        }
    ]
}
```

---

## Resource-Based Policies (Queue Policies)

Attached directly to the SQS queue. Used for cross-account access or allowing AWS services to write to your queue.

| Aspect | IAM Policies | Queue Policies |
|---|---|---|
| **Attached To** | Users, groups, or roles | SQS queue directly |
| **Scope** | Account-wide permissions | Queue-specific permissions |
| **Cross-Account** | Requires trust relationships | Supports direct external access |
| **Max Size** | — | 8,192 characters |

### Queue Policy Example — Allow Another Account to Send

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCrossAccountSend",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::555666777888:root"
            },
            "Action": "sqs:SendMessage",
            "Resource": "arn:aws:sqs:us-east-1:123456789012:MyQueue",
            "Condition": {
                "StringEquals": {
                    "aws:SourceAccount": "555666777888"
                }
            }
        }
    ]
}
```

---

## Cross-Account Access

To allow an AWS principal in Account B to access a queue in Account A, you need **both**:

1. A queue policy in Account A granting access to Account B's role/user
2. An IAM policy in Account B granting the role/user permission to use the foreign queue

**Step 1 — Queue Policy in Account A:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::ACCOUNT-B:role/SQSConsumerRole"
            },
            "Action": ["sqs:ReceiveMessage", "sqs:DeleteMessage"],
            "Resource": "arn:aws:sqs:us-east-1:ACCOUNT-A:CrossAccountQueue"
        }
    ]
}
```

**Step 2 — IAM Policy in Account B:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["sqs:ReceiveMessage", "sqs:DeleteMessage"],
            "Resource": "arn:aws:sqs:us-east-1:ACCOUNT-A:CrossAccountQueue"
        }
    ]
}
```

**Spring Boot — Assume Role for Cross-Account Access:**

```java
@Configuration
public class CrossAccountSQSConfig {

    @Bean
    @Qualifier("crossAccountSQS")
    public AmazonSQS crossAccountSQS() {
        STSAssumeRoleSessionCredentialsProvider credentialsProvider =
            new STSAssumeRoleSessionCredentialsProvider.Builder(
                "arn:aws:iam::ACCOUNT-B:role/SQSCrossAccountRole",
                "CrossAccountSession"
            ).build();

        return AmazonSQSClientBuilder.standard()
            .withCredentials(credentialsProvider)
            .withRegion(Regions.US_EAST_1)
            .build();
    }
}
```

---

## Encryption

### Server-Side Encryption at Rest (SSE)

| Option | Description | Key Management |
|---|---|---|
| **SSE-SQS** | SQS-managed keys, automatic rotation | AWS manages everything — zero config |
| **SSE-KMS (AWS Managed)** | AWS-managed KMS keys | Provides KMS audit trail; minimal overhead |
| **SSE-KMS (Customer Managed)** | Your own KMS CMK | Full control, fine-grained access, full audit |

**Creating an encrypted queue with a customer-managed KMS key:**

```java
@Service
public class EncryptedQueueService {

    @Autowired
    private AmazonSQS amazonSQS;

    public String createEncryptedQueue(String queueName, String kmsKeyId) {
        Map<String, String> attributes = new HashMap<>();
        attributes.put(QueueAttributeName.KmsMasterKeyId.toString(), kmsKeyId);
        attributes.put(
            QueueAttributeName.KmsDataKeyReusePeriodSeconds.toString(),
            "300"  // reuse data key for 5 minutes to reduce KMS API calls
        );

        CreateQueueRequest request = new CreateQueueRequest()
            .withQueueName(queueName)
            .withAttributes(attributes);

        return amazonSQS.createQueue(request).getQueueUrl();
    }
}
```

### Encryption in Transit

All SQS API calls use **HTTPS** by default — no additional configuration needed.

---

## VPC Endpoints (PrivateLink)

Route SQS traffic through your private VPC — no internet required. This improves security and reduces data transfer costs.

- **Type**: Interface Endpoint (AWS PrivateLink) — creates an ENI in your subnet
- **Supported**: All SQS APIs
- **Access Control**: Security groups + endpoint policies

### Endpoint Policy — Restrict to Your VPC

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": ["sqs:SendMessage", "sqs:ReceiveMessage", "sqs:DeleteMessage"],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalVpc": "vpc-12345678"
                }
            }
        }
    ]
}
```

### Spring Boot — Use VPC Endpoint URL

```java
@Bean
public AmazonSQS vpcEndpointSQS() {
    return AmazonSQSClientBuilder.standard()
        .withRegion(Regions.US_EAST_1)
        .withEndpointConfiguration(
            new AwsClientBuilder.EndpointConfiguration(
                "https://vpce-12345-abcdef.sqs.us-east-1.vpce.amazonaws.com",
                "us-east-1"
            )
        )
        .build();
}
```

---

## Temporary Credentials with AWS STS

Use temporary credentials instead of long-lived access keys — especially for cross-account or short-lived operations.

```java
@Service
public class AssumeRoleService {

    @Autowired
    private AWSSecurityTokenService stsClient;

    public AmazonSQS buildSQSClientWithRole(String roleArn) {
        AssumeRoleRequest request = new AssumeRoleRequest()
            .withRoleArn(roleArn)
            .withRoleSessionName("SQSSession-" + System.currentTimeMillis())
            .withDurationSeconds(3600);  // 1 hour

        AssumeRoleResult result = stsClient.assumeRole(request);
        Credentials credentials = result.getCredentials();

        BasicSessionCredentials sessionCredentials = new BasicSessionCredentials(
            credentials.getAccessKeyId(),
            credentials.getSecretAccessKey(),
            credentials.getSessionToken()
        );

        return AmazonSQSClientBuilder.standard()
            .withCredentials(new AWSStaticCredentialsProvider(sessionCredentials))
            .withRegion(Regions.US_EAST_1)
            .build();
    }
}
```

### Auto-Refreshing Credentials

For long-running services, implement automatic credential refresh before expiry:

```java
@Component
public class RefreshableCredentialsProvider implements AWSCredentialsProvider {

    private final String roleArn;
    private final AWSSecurityTokenService stsClient;
    private volatile AWSCredentials credentials;
    private volatile Date credentialsExpiration;

    @Override
    public AWSCredentials getCredentials() {
        if (needsRefresh()) {
            synchronized (this) {
                if (needsRefresh()) {
                    refreshCredentials();
                }
            }
        }
        return credentials;
    }

    private boolean needsRefresh() {
        // Refresh 1 minute before actual expiration
        return credentials == null ||
               credentialsExpiration == null ||
               credentialsExpiration.before(new Date(System.currentTimeMillis() + 60_000));
    }

    private void refreshCredentials() {
        AssumeRoleRequest request = new AssumeRoleRequest()
            .withRoleArn(roleArn)
            .withRoleSessionName("AutoRefresh-" + System.currentTimeMillis())
            .withDurationSeconds(3600);

        AssumeRoleResult result = stsClient.assumeRole(request);
        Credentials creds = result.getCredentials();

        this.credentials = new BasicSessionCredentials(
            creds.getAccessKeyId(),
            creds.getSecretAccessKey(),
            creds.getSessionToken()
        );
        this.credentialsExpiration = creds.getExpiration();
    }

    @Override
    public void refresh() {
        refreshCredentials();
    }
}
```

---

## Security Best Practices

| Practice | Description |
|---|---|
| **Least privilege** | Grant only the actions each role actually needs |
| **No hardcoded credentials** | Use IAM roles for EC2/ECS/Lambda; never store keys in code |
| **Enable SSE** | Use SSE-SQS at minimum; SSE-KMS for regulated workloads |
| **VPC endpoints** | Route SQS traffic privately for sensitive environments |
| **Queue policies for cross-account** | Always pair a queue policy (Account A) with an IAM policy (Account B) |
| **Temporary credentials** | Prefer STS AssumeRole over long-lived access keys |
| **Auto-refresh credentials** | Implement refresh 60 seconds before expiry for long-running services |
| **Audit with CloudTrail** | Enable CloudTrail to log all SQS API calls for compliance |
