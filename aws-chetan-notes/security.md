# AWS Security

## AWS IAM

**What is IAM?**

IAM (Identity and Access Management) manages authentication and authorization for AWS resources, controlling who can access what.

**IAM Components**

| Component | Description | Use Case |
|-----------|-------------|----------|
| Users | Individual accounts for people | Developers, administrators |
| Groups | Collections of users with shared permissions | Development team, operations team |
| Roles | Temporary permissions for services | EC2, Lambda accessing other AWS services |
| Policies | JSON documents defining permissions | Define what actions are allowed or denied |
| MFA | Multi-Factor Authentication | Enhanced security for privileged accounts |

**IAM Roles vs IAM Policies**

- IAM Role: An identity that can be assumed temporarily by users, services, or applications
- IAM Policy: A document that defines permissions and can be attached to users, groups, or roles

**Service Principal**

An identifier for a service that can assume a role. Example: ec2.amazonaws.com, lambda.amazonaws.com.

**Common Use Cases**

- Secure access for developers, administrators, and applications
- Enforce least privilege principle
- Enable cross-account access
- Grant temporary credentials to applications

---

## IAM Best Practices

**Security Recommendations**

- Use IAM roles instead of hardcoded credentials
- Enable MFA for privileged accounts
- Apply least privilege principle: grant minimum permissions needed
- Rotate access keys regularly
- Use IAM policies to enforce conditions (IP restrictions, time-based access)
- Audit IAM permissions regularly
- Use AWS Organizations for multi-account management

**How Applications Access AWS Services**

Applications communicate with S3, SQS, and other services securely using:

1. IAM Roles attached to EC2 instances or Lambda functions
2. Temporary credentials automatically provided by AWS
3. No need to store access keys in code

---

## AWS Secrets Manager

**What is Secrets Manager?**

A fully managed service to store, rotate, and retrieve secrets such as database credentials, API keys, and OAuth tokens.

**Why Use Secrets Manager?**

- Centralized secure secret storage
- Removes hardcoded secrets from code and config files
- Automatic rotation of secrets (RDS credentials rotate automatically)
- Auditing via CloudTrail
- Automatic encryption using AWS KMS
- Avoids plaintext passwords in Git repositories

**Common Use Cases**

- Database credentials for applications
- API keys for third-party services
- OAuth tokens
- Encryption keys
- TLS certificates

---

## Spring Boot Integration with Secrets Manager

**Setup Steps**

1. Add Maven dependency

```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter-secrets-manager</artifactId>
</dependency>
```

2. Configure application.yml

```yaml
spring:
  application:
    name: my-spring-app
  cloud:
    aws:
      region:
        static: ap-south-1
      secretsmanager:
        enabled: true
```

3. Create secret in AWS Secrets Manager

Secret name: my-spring-app/dev

Secret value (JSON):

```json
{
  "db.username": "admin",
  "db.password": "mypassword"
}
```

4. Reference in application.properties

```properties
spring.datasource.username=${db.username}
spring.datasource.password=${db.password}
```

5. At runtime, Spring Cloud AWS fetches from Secrets Manager and injects values

**Key Packages**

- io.awspring.cloud.autoconfigure.secretsmanager.AwsSecretsManagerAutoConfiguration
- software.amazon.awssdk.services.secretsmanager.SecretsManagerClient (for direct SDK usage)

---

## Application Security

**Security Measures**

- Use IAM roles instead of hardcoded credentials
- Store secrets in Secrets Manager or Parameter Store
- Enable encryption in transit using TLS
- Enable encryption at rest using KMS or SSE-S3
- Use AWS WAF to prevent attacks (SQL injection, DDoS, XSS)
- Use AWS Shield for DDoS protection
- Implement input validation and output encoding
- Keep dependencies and libraries updated

**Secrets Manager vs Parameter Store**

| Feature | Secrets Manager | Parameter Store |
|---------|----------------|-----------------|
| Purpose | Secrets with automatic rotation | Configuration and secrets |
| Rotation | Built-in automatic rotation | Manual rotation |
| Cost | Pay per secret and API call | Free tier available |
| Encryption | Automatic with KMS | Optional with KMS |
| Use Case | Database credentials, API keys | Application configuration, simple secrets |

---

## Network Security

**Network Security Components**

- Security Groups: Instance-level firewall (stateful)
- NACLs: Subnet-level firewall (stateless)
- Route Tables: Control traffic flow between subnets
- VPC Flow Logs: Capture network traffic for monitoring
- AWS WAF: Web application firewall for HTTP/HTTPS traffic
- AWS Shield: DDoS protection

**Security Monitoring and Detection**

| Service | Purpose |
|---------|---------|
| GuardDuty | Threat detection using machine learning |
| Inspector | Vulnerability scanning for EC2 and containers |
| CloudTrail | Audit logs for API calls |
| Security Hub | Centralized security findings from multiple services |
| Config | Track resource configuration changes |

---

## Encryption

**Encryption at Rest**

- S3: Server-side encryption (SSE-S3, SSE-KMS, SSE-C)
- EBS: Encrypted volumes using KMS
- RDS: Database encryption using KMS
- DynamoDB: Encryption using KMS

**Encryption in Transit**

- Use TLS/SSL for all data transmission
- HTTPS for web traffic
- VPN or Direct Connect for on-premises connectivity
- Secure protocols for database connections

**AWS KMS (Key Management Service)**

- Managed service for creating and controlling encryption keys
- Integrates with most AWS services
- Audit key usage via CloudTrail
- Regional service with automatic key rotation

---

## Compliance and Auditing

**Compliance Services**

| Service | Purpose |
|---------|---------|
| CloudTrail | Log all API calls for auditing |
| Config | Track resource configuration and compliance |
| Artifact | Download compliance reports and agreements |
| Audit Manager | Automate evidence collection for audits |

**Best Practices for Compliance**

- Enable CloudTrail in all regions
- Use AWS Config rules to enforce compliance policies
- Implement log aggregation and analysis
- Regular security assessments and penetration testing
- Document security controls and procedures

---

## High Availability and Isolation

**High Availability**

The ability of a system to remain operational and accessible even during failures.

**How to Achieve High Availability in AWS**

- Deploy across multiple Availability Zones
- Use Auto Scaling Groups
- Implement load balancing
- Use managed services with built-in redundancy
- Regular backups and disaster recovery planning

**Isolation of Failures**

Preventing failures in one component from affecting other components.

**Isolation Techniques**

- Separate environments using VPCs
- Use multiple Availability Zones
- Implement circuit breakers in application code
- Deploy microservices independently
- Use separate security groups and network segments

**SLA vs SLO**

- SLA (Service Level Agreement): Commitment to customers about service availability
- SLO (Service Level Objective): Internal target for service reliability

---

## Security Perspectives Summary

**Application Security Focus**

- IAM roles and policies
- Secrets Manager for credentials
- Encryption in transit and at rest
- WAF and Shield for application protection
- Input validation and secure coding practices

**Network Security Focus**

- Security Groups and NACLs
- VPC isolation and segmentation
- Route tables for traffic control
- GuardDuty for threat detection
- Inspector for vulnerability scanning
- CloudTrail for audit logs

**Data Security Focus**

- Encryption at rest using KMS
- Encryption in transit using TLS
- Access control using IAM policies
- Data lifecycle management
- Regular backups and replication