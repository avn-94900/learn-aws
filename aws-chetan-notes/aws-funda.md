# AWS Fundamentals

## Cloud Computing Basics

**What is Cloud Computing?**

On-demand delivery of IT resources such as compute, storage, databases, and networking over the internet with pay-as-you-go pricing.

**Cloud Deployment Models**

| Model | Description | Use Case |
|-------|-------------|----------|
| Public Cloud | Fully hosted by provider | AWS, Azure, GCP |
| Private Cloud | Dedicated for one organization | On-premises or hosted |
| Hybrid Cloud | Combination of on-premises and cloud | Gradual migration, compliance requirements |
| Multi-Cloud | Using multiple cloud providers together | Avoid vendor lock-in, best-of-breed services |

**Key Cloud Concepts**

- On-Demand: Pay only when you use resources
- Auto-Scaling: Increase or decrease resources automatically based on demand using rules or policies
- Elasticity: Cloud's inherent ability to expand or shrink instantly

---

## Cloud Service Models

**Service Model Comparison**

| Layer | On-Premises | IaaS | PaaS | SaaS |
|-------|-------------|------|------|------|
| Applications | You manage | You manage | You manage | Managed |
| Data | You manage | You manage | You manage | Managed |
| Runtime | You manage | You manage | Managed | Managed |
| Middleware | You manage | You manage | Managed | Managed |
| OS | You manage | You manage | Managed | Managed |
| Virtualization | You manage | Managed | Managed | Managed |
| Servers/Storage/Network | You manage | Managed | Managed | Managed |

**Model Definitions**

- IaaS: Infrastructure as a Service (manage apps, data, runtime, middleware, OS)
- PaaS: Platform as a Service (manage apps and data only)
- SaaS: Software as a Service (fully managed applications)

---

## Hybrid Cloud

**What is Hybrid Cloud?**

A mix of on-premises infrastructure and cloud infrastructure working together.

**Why Use Hybrid Cloud?**

- Trust and security concerns for sensitive data
- Large data volumes hard to move immediately
- Compliance and data residency requirements
- Gradual migration instead of full cloud adoption

**Example**

Keep databases on-premises but run applications in AWS cloud.

---

## AWS Advantages Over On-Premises

**Key Advantages**

1. Scalability: Easily scale up or down with demand
2. Cost-Effective: Pay-as-you-go with no upfront infrastructure cost
3. High Availability: 99.99% uptime SLA across multiple Availability Zones
4. Disaster Recovery: Automated DR and backup across data centers
5. Advanced Security: AI-driven threat detection, encryption, IAM controls

---

## AWS Regions and Availability Zones

**AWS Region**

A geographical area such as us-east-1 or ap-south-1.

**Availability Zone (AZ)**

A physically separate data center within a region. Each region has multiple AZs for redundancy.

**Purpose**

AZs provide redundancy inside a region for high availability. If one AZ fails, services continue running in other AZs.

---

## AWS Services vs Resources

**AWS Service**

A tool or feature provided by AWS such as EC2, S3, or RDS.

**AWS Resource**

An instance of a service you create and use such as a specific EC2 instance or S3 bucket.

**Analogy**

Service is like a class, resource is like an object.

**AWS Resource Groups**

A logical grouping of AWS resources for easier management. Helps identify and manage related resources together such as EC2, RDS, and S3 that belong to one project.

---

## Categories of AWS Services

| Category | Services |
|----------|----------|
| Compute | EC2, Lambda, Elastic Beanstalk, ECS, Auto Scaling |
| Databases | RDS (MySQL, PostgreSQL, Oracle, SQL Server), DynamoDB, Aurora |
| Storage | S3, EBS, EFS |
| Networking and Content Delivery | VPC, Route 53, CloudFront, ELB, API Gateway |
| Messaging and Integration | SNS, SQS, EventBridge |
| Security | IAM, Secrets Manager, KMS |
| Infrastructure and DevOps | CloudFormation, CloudWatch, CodePipeline, CodeDeploy |

---

## Scaling in AWS

**Horizontal Scaling (Scale-Out/In)**

- Add more EC2 instances
- Best for stateless apps like microservices and web apps
- Cost-effective with auto scaling support

**Vertical Scaling (Scale-Up/Down)**

- Increase CPU or RAM of a single EC2 instance
- Best for stateful apps like monolithic applications
- More expensive with no auto scaling support

**When to Use Which**

| Factor | Horizontal Scaling | Vertical Scaling |
|--------|-------------------|------------------|
| Application Type | Stateless, distributed | Stateful, monolithic |
| Cost | Lower, pay for what you use | Higher, fixed larger instance |
| Availability | Higher, multiple instances | Lower, single point of failure |
| Auto Scaling | Supported | Not supported |

**AWS Auto Scaling Groups**

A group of EC2 instances that automatically scale in or out based on demand or custom rules.

---

## AWS Pricing Models

**EC2 Instance Pricing**

| Model | Description | Use Case |
|-------|-------------|----------|
| On-Demand | Pay-as-you-go, no commitment | Short-term workloads, testing, development |
| Reserved | Commit for 1 or 3 years | Steady-state usage, cost savings up to 75% |
| Spot | Use spare capacity, can be interrupted | Batch jobs, flexible workloads, cost savings up to 90% |

---

## IT Infrastructure Layers

| Layer | Definition | Examples |
|-------|------------|----------|
| Application | Software programs that users interact with | Web apps, mobile apps, data processing apps |
| Data | Information stored, managed, and processed | Databases, files, logs |
| Runtime | Environment where applications run | CLR (.NET), JVM (Java), Node.js runtime |
| Middleware | Software connecting applications to systems | APIs, databases, messaging systems (Kafka, RabbitMQ) |
| Operating System | Manages hardware resources and provides services | Windows, Linux, macOS |
| Virtualization | Creates virtual versions of servers, OS, storage | VMware, Hyper-V, KVM |
| Servers | Physical or virtual machines running workloads | On-premises servers, EC2, GCP Compute Engine |
| Data Centers | Physical facilities housing servers and networking | AWS Region/AZ, Azure Data Center |
| Storage | Storage and retrieval of digital information | SSDs, HDDs, S3, EBS, EFS |
| Networking | Connecting computers, devices, and resources | Ethernet, Wi-Fi, VPN, VPC, Load Balancer |

---

## AWS Elastic Beanstalk

**What is Elastic Beanstalk?**

A Platform-as-a-Service (PaaS) for deploying and scaling applications without managing infrastructure.

**Key Features**

1. Supports multiple languages: .NET, Java, Python, Node.js, PHP, Ruby
2. Built-in scalability
3. Fully managed: patching, security, updates
4. High availability: auto failover, backup
5. Secure and compliant: authentication, IAM integration

**When to Use**

- Quickly deploy web applications without infrastructure management
- Focus on code instead of servers
- Need automatic scaling and load balancing

---

## AWS Accounts and Organizations

**AWS Account**

An isolated security and billing boundary for AWS resources.

**AWS Organization**

A group of accounts under a single master account for consolidated billing and policies.

**Benefits**

- Central billing across multiple accounts
- Consolidated policies and access controls
- Best for multi-team and multi-project environments
- Segregate resources by environment (dev, staging, production) or department