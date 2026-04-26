<!-- # AWS Interview Complete Guide
## Target Profile: 5-Year Spring Boot Developer | 3-Year AWS Experience

---

## 1. Cloud Computing Basics

- What is Cloud Computing?
- What are the differences between Public, Private, Hybrid, and Multi-Cloud models?
- Explain On-Demand, Auto-Scaling, and Elasticity
- What is the difference between Auto-Scaling vs Elasticity?
- Classify services into IaaS, PaaS, and SaaS with examples
- Give real-world examples of SaaS (Salesforce, Office 365, Amazon WorkMail)
- What factors should you consider when choosing between IaaS, PaaS, and SaaS solutions?

---

## 2. AWS Storage Services

### 2.1 S3
- What is the difference between Blob Storage and Object Storage?
- How does Amazon S3 store data?
- What are the storage classes in S3 (Hot, Cold, Archival/Glacier)?
- How do you decide which storage service or class to use?
- How do you reduce S3 storage costs?
- What is S3 versioning and when would you enable it?
- What is S3 lifecycle policy? Give a real-world example
- What is S3 Transfer Acceleration?
- What is S3 pre-signed URL? How do you generate one from a Spring Boot app?
- What is S3 event notification? How do you trigger a Lambda from an S3 upload?
- What is S3 Object Lock and when is it used?
- How do you secure an S3 bucket (bucket policy, ACL, Block Public Access)?

### 2.2 EBS and EFS
- Difference between EBS, S3, and EFS
- What are typical use cases for EBS, S3, and EFS?
- What are EBS volume types (gp2, gp3, io1, io2, st1, sc1)?
- When would you choose EFS over EBS?

---

## 3. AWS Compute Options

### 3.1 EC2
- Compare EC2, Lambda, Elastic Beanstalk, ECS, and EKS
- What factors influence which compute option to choose?
- Advantages and disadvantages of EC2 vs Lambda vs ECS vs EKS
- Which option is better for monolithic applications? Which for microservices?
- What are the cost benefits associated with each compute option?
- What are EC2 instance types?
- When would you choose Spot Instances? Provide use cases
- What is EC2 user data? How do you bootstrap an EC2 instance?
- What is an AMI? How do you create a custom AMI?
- What is EC2 Instance Connect vs SSH vs SSM Session Manager?

### 3.2 Auto Scaling
- What is an Auto Scaling Group (ASG)?
- Difference between auto-scaling and elasticity
- What are the types of scaling policies (target tracking, step scaling, scheduled)?
- What is a launch template vs launch configuration?
- How does ASG handle health checks (EC2 vs ELB health checks)?
- What is a warm pool in ASG?

---

## 4. AWS Lambda and Serverless

### 4.1 Lambda Fundamentals
- What is AWS Lambda?
- Why would you use Lambda instead of embedding the logic directly in the main application?
- What are the drawbacks of using Lambda inside the application code?
- How does Lambda help with separation of concerns?
- Provide real-time use cases for Lambda (image compression, sending welcome email, log processing)
- How is Lambda billed?
- What are Lambda triggers (events)? Name a few event sources

### 4.2 Lambda Advanced
- What is Lambda cold start? How do you minimize it?
- What is Provisioned Concurrency in Lambda?
- What is Lambda reserved concurrency vs provisioned concurrency?
- What are Lambda layers? When would you use them?
- What is Lambda destination (on success / on failure)?
- What is the maximum execution timeout for Lambda?
- How do you handle Lambda timeouts gracefully?
- How do you pass environment variables to Lambda securely?
- What is Lambda@Edge? How is it different from CloudFront Functions?
- How do you deploy a Spring Boot app as a Lambda function (AWS Lambda Web Adapter / Spring Cloud Function)?

### 4.3 API Gateway
- What is AWS API Gateway?
- How do you build a serverless REST API?
- What is the difference between REST API and HTTP API in API Gateway?
- What is Lambda Proxy integration vs Lambda Custom integration?
- How do you handle CORS in API Gateway?
- What is API Gateway throttling and how do you configure it?
- How do you secure API Gateway (API keys, Cognito authorizer, Lambda authorizer)?
- What is API Gateway caching?

---

## 5. Networking and Security

### 5.1 VPC
- What is a VPC? Why is it needed?
- What are the components of a VPC (subnets, route tables, NAT, IGW, NACLs, Security Groups)?
- What is the default VPC? What happens if you don't create one?
- How does a web application inside a VPC respond to user requests?
- What is the difference between public subnet and private subnet?
- What is a NAT Gateway vs NAT Instance?
- What is VPC Peering? What are its limitations?
- What is Transit Gateway? How is it different from VPC Peering?
- What is AWS PrivateLink?
- What is a VPC Endpoint? Difference between Interface Endpoint and Gateway Endpoint?
- What is VPC Flow Logs? How do you use them for troubleshooting?

### 5.2 Load Balancing and DNS
- What is a Load Balancer in AWS? What are the different types (ALB, NLB, CLB, GWLB)?
- When would you use ALB vs NLB?
- What is a target group in ALB?
- What is sticky session (session affinity) in ALB?
- What is ALB path-based routing vs host-based routing?
- What is Amazon Route 53? Why is it used?
- What happens if Route 53 goes down? How does failover strategy work?
- What is proximity routing in Route 53?
- What are Route 53 routing policies (simple, weighted, latency, failover, geolocation)?
- What is Route 53 health check?

### 5.3 Security Groups and NACLs
- What is the difference between Security Groups and NACLs?
- What are firewalls, inbound rules, outbound rules, and route tables?
- Security Groups are stateful — what does that mean?
- NACLs are stateless — what does that mean?

---

## 6. IAM and Security Management

### 6.1 IAM Core
- What are IAM roles vs IAM policies?
- What is a service principal in IAM?
- How does an application communicate with S3 or SQS securely (IAM roles/policies)?
- What is the difference between identity-based policy and resource-based policy?
- What is IAM permission boundary?
- What is the principle of least privilege?
- What is an IAM instance profile?
- What is STS (Security Token Service)? What is AssumeRole?
- What is cross-account access in AWS? How do you implement it?

### 6.2 Secrets and Encryption
- What is AWS Secrets Manager?
- Why do we use Secrets Manager instead of storing credentials in code or config files?
- How do you integrate AWS Secrets Manager with a Spring Boot application?
- What is AWS Systems Manager Parameter Store? How is it different from Secrets Manager?
- What is AWS KMS?
- What is envelope encryption?
- What is the difference between AWS-managed keys and customer-managed keys (CMK)?
- How do you encrypt an S3 bucket (SSE-S3, SSE-KMS, SSE-C)?
- How do you rotate secrets automatically in Secrets Manager?

### 6.3 Cognito and Application Security
- What is Amazon Cognito?
- What is the difference between Cognito User Pool and Identity Pool?
- How do you integrate Cognito with Spring Security (JWT validation)?
- What is OAuth2 / OIDC flow in Cognito?
- What is AWS WAF? When would you use it?
- What is AWS Shield? Difference between Shield Standard and Advanced?
- What is Amazon GuardDuty?
- What is AWS Security Hub?

---

## 7. Hybrid Cloud and Connectivity

- What is AWS Direct Connect?
- Difference between Direct Connect and VPN
- Why would some databases remain on-premises instead of moving to AWS?
- Explain a Hybrid Cloud approach with AWS
- What are the challenges with latency between on-premises and AWS resources?
- How do Availability Zones (AZs) ensure high availability?
- What happens when one AZ fails?
- Difference between Availability Zone and Region
- What is AWS Outposts?

---

## 8. Databases

### 8.1 RDS
- What is RDS? How is it different from self-managed databases?
- What database engines does RDS support?
- Explain read replicas and Multi-AZ deployments in RDS
- What is the difference between Multi-AZ and Read Replica?
- What is RDS Proxy? Why would you use it with Lambda or ECS?
- How do you tune HikariCP connection pool for RDS in a Spring Boot app?
- What is RDS automated backup vs manual snapshot?
- How do you perform a zero-downtime RDS upgrade?

### 8.2 Aurora
- What is Amazon Aurora? How is it different from RDS?
- What is Aurora Serverless v2? When would you use it?
- What is Aurora Global Database?
- What is Aurora Multi-Master?

### 8.3 DynamoDB
- What is DynamoDB? When should you use it over RDS?
- What is a partition key vs sort key in DynamoDB?
- What is a Global Secondary Index (GSI) vs Local Secondary Index (LSI)?
- What is DynamoDB single-table design pattern?
- What is DynamoDB Streams? How do you trigger Lambda from it?
- What is DynamoDB TTL?
- What is DynamoDB DAX?
- What is DynamoDB on-demand vs provisioned capacity mode?

### 8.4 ElastiCache
- What is Amazon ElastiCache?
- Difference between Redis and Memcached on ElastiCache
- How do you integrate ElastiCache Redis with Spring Boot (Spring Cache, Lettuce)?
- What is cache-aside pattern vs write-through pattern?
- What is Redis cluster mode vs non-cluster mode?
- How do you handle cache invalidation?

---

## 9. Messaging and Queueing

### 9.1 SQS
- What is Amazon SQS?
- Difference between SQS Standard and FIFO queues
- What is message visibility timeout?
- What is a Dead Letter Queue (DLQ)?
- What is long polling vs short polling in SQS?
- What is SQS message deduplication?
- How do you consume SQS messages in Spring Boot using @SqsListener?
- How do you configure batch size and concurrency for SQS listeners in Spring Boot?
- How do you handle poison pill messages in SQS?
- What is SQS Extended Client Library? When would you use it?

### 9.2 SNS
- What is Amazon SNS? How is it different from SQS?
- What is SNS fan-out pattern?
- What is SNS message filtering?
- How do you send SNS notifications from a Spring Boot app?

### 9.3 EventBridge and Kinesis
- What is Amazon EventBridge?
- When to use SQS vs SNS vs EventBridge?
- What is an EventBridge rule and event pattern?
- What is Amazon Kinesis Data Streams?
- Difference between Kinesis and SQS
- What is Kinesis Data Firehose?
- When would you use Kinesis over SQS?

---

## 10. Monitoring, Logging, and Observability

### 10.1 CloudWatch
- What is Amazon CloudWatch?
- What metrics can CloudWatch monitor?
- How do CloudWatch Alarms work?
- What is a composite alarm in CloudWatch?
- What is CloudWatch Logs Insights? Give an example query
- What is CloudWatch Embedded Metrics Format (EMF)?
- How do you publish custom metrics from a Spring Boot app to CloudWatch using Micrometer?
- What is CloudWatch Container Insights?
- What is CloudWatch Application Insights?

### 10.2 Distributed Tracing and Audit
- What is AWS X-Ray? When would you use it?
- How do you integrate X-Ray with a Spring Boot application?
- What is a trace, segment, and subsegment in X-Ray?
- Difference between CloudWatch Logs and CloudTrail
- What is CloudTrail? What events does it capture?
- What is AWS Config? How is it different from CloudTrail?
- How do you set up structured JSON logging in Spring Boot for CloudWatch Logs Insights?

---

## 11. CI/CD and DevOps

### 11.1 CI/CD Fundamentals
- What is CI/CD?
- What are AWS CI/CD services (CodeBuild, CodeDeploy, CodePipeline, GitHub Actions)?
- Explain the CI/CD pipeline flow
- What is blue/green deployment?
- What is canary deployment?
- What is rolling deployment?
- How do you implement zero-downtime deployment for a Spring Boot app on ECS?

### 11.2 Infrastructure as Code
- What is Infrastructure as Code (IaC)?
- Difference between Terraform and CloudFormation
- What are CloudFormation stacks and templates?
- What is a CloudFormation change set?
- What is CloudFormation drift detection?
- What is AWS CDK? How is it different from CloudFormation?
- What is Terraform state? How do you manage remote state on S3?
- What is Terraform plan vs apply vs destroy?

### 11.3 CodeBuild and CodeDeploy
- What is a buildspec.yml in CodeBuild?
- How do you build and push a Docker image to ECR in CodeBuild?
- What is an appspec.yml in CodeDeploy?
- How does CodeDeploy work with ECS (blue/green)?

---

## 12. Containers and Orchestration

### 12.1 Docker and ECR
- What is Docker?
- What is Amazon ECR?
- How do you push a Docker image to ECR from a CI/CD pipeline?
- What is a multi-stage Docker build? Why is it useful for Spring Boot?
- How do you optimize Docker image size for a Spring Boot app?

### 12.2 ECS
- What is Amazon ECS?
- What is AWS Fargate?
- When to use EC2 launch type vs Fargate in ECS?
- What is an ECS task definition?
- What is an ECS service vs ECS task?
- How do you configure health checks for an ECS service?
- How do you handle graceful shutdown in a Spring Boot app on ECS?
- What is ECS service auto scaling?
- What is ECS service discovery using AWS Cloud Map?
- How do you pass secrets to ECS containers (Secrets Manager / Parameter Store integration)?
- What is ECS capacity provider?

### 12.3 EKS
- What is Amazon EKS?
- Difference between ECS and EKS
- What is a Kubernetes pod, deployment, service, and ingress?
- What is the AWS Load Balancer Controller in EKS?
- What is Karpenter? How is it different from Cluster Autoscaler?
- What is a Kubernetes ConfigMap vs Secret?
- How do you manage secrets in EKS using AWS Secrets Manager (External Secrets Operator)?
- What is Helm? How do you deploy a Spring Boot app using Helm charts?
- What is a Kubernetes namespace?
- What is a Kubernetes liveness probe vs readiness probe?
- How do you configure resource requests and limits for a Spring Boot container in EKS?

---

## 13. Spring Boot + AWS Integration

### 13.1 Spring Cloud AWS
- What is Spring Cloud AWS?
- How do you configure AWS SDK v2 in a Spring Boot application?
- How do you use Spring Cloud AWS to read from S3?
- How do you integrate Spring Boot with SQS using Spring Cloud AWS (@SqsListener)?
- How do you send messages to SNS from Spring Boot?
- How do you read configuration from AWS Parameter Store in Spring Boot?
- How do you integrate AWS Secrets Manager with Spring Boot for database credentials?

### 13.2 Spring Boot Deployment Patterns on AWS
- How do you deploy a Spring Boot app on ECS Fargate?
- How do you configure Spring Boot health endpoints for ECS/EKS health checks (/actuator/health)?
- How do you implement graceful shutdown in Spring Boot on ECS?
- How do you deploy a Spring Boot app as a Lambda function using Spring Cloud Function?
- How do you handle Lambda cold starts with Spring Boot?
- How do you configure Spring Boot logging to output JSON for CloudWatch Logs Insights?
- How do you publish custom metrics from Spring Boot to CloudWatch using Micrometer?
- How do you integrate Spring Boot with X-Ray for distributed tracing?
- How do you configure HikariCP for RDS in a Spring Boot app running on ECS?
- How do you use RDS Proxy with Spring Boot to handle connection pooling at scale?

### 13.3 Spring Boot Security with AWS
- How do you validate Cognito JWT tokens in Spring Security?
- How do you implement OAuth2 resource server in Spring Boot with Cognito?
- How do you use IAM roles for service accounts (IRSA) in EKS for Spring Boot apps?

---

## 14. Cost Optimization

- What are EC2 pricing models (On-Demand, Reserved, Spot)?
- How do you optimize costs in AWS?
- What is AWS Cost Explorer?
- What are AWS Budgets?
- How do you reduce S3 storage costs?
- What is Savings Plans vs Reserved Instances?
- How do you right-size EC2 instances using AWS Compute Optimizer?
- What is S3 Intelligent-Tiering?
- How do you identify idle or underutilized resources?
- What is AWS Trusted Advisor?

---

## 15. Disaster Recovery and High Availability

- What are the disaster recovery strategies in AWS (Backup & Restore, Pilot Light, Warm Standby, Multi-Site Active-Active)?
- What is RTO and RPO?
- How do you design for high availability?
- What is Multi-AZ deployment?
- What is cross-region replication?
- What is AWS Backup?
- How do you implement cross-region failover for a Spring Boot application?
- What is Route 53 failover routing?

---

## 16. Advanced Topics

### 16.1 Organizations and Governance
- What is AWS Organizations?
- What is Service Control Policy (SCP)?
- What is AWS Control Tower?
- What is AWS Config? How do you use it for compliance?
- What is AWS Systems Manager?

### 16.2 Advanced Networking
- What is VPC Peering vs Transit Gateway?
- What is AWS Global Accelerator? How is it different from CloudFront?
- What is Amazon CloudFront? What is an origin?
- What is CloudFront signed URL vs signed cookie?
- What is AWS App Mesh?

### 16.3 Event-Driven and Async Patterns
- What is the Saga pattern? How do you implement it with SQS/SNS on AWS?
- What is the outbox pattern? How do you implement it with DynamoDB Streams or RDS?
- What is event sourcing? How does EventBridge support it?
- What is the CQRS pattern? How do you implement it on AWS?

---

## 17. Scenario-Based Questions

- Design a highly available web application architecture on AWS
- How would you migrate an on-premises Spring Boot monolith to AWS microservices?
- Design a CI/CD pipeline for a Spring Boot microservices application on ECS
- How would you handle a sudden spike in traffic for a Spring Boot app?
- Design a disaster recovery solution for a critical Spring Boot application
- How would you secure sensitive data (DB credentials, API keys) in a Spring Boot app on AWS?
- Design a serverless data processing pipeline using Lambda, SQS, and S3
- How would you implement blue/green deployment for zero-downtime releases on ECS?
- Design an event-driven order processing system using SQS, SNS, and Lambda
- How would you implement distributed tracing across multiple Spring Boot microservices on ECS?
- How would you implement rate limiting for an API built with Spring Boot and API Gateway?
- Design a multi-tenant SaaS application on AWS

---

## 18. Troubleshooting

- How do you troubleshoot EC2 instance connectivity issues?
- How do you debug Lambda function errors?
- How do you investigate high latency in API Gateway?
- How do you troubleshoot SQS message processing delays?
- How do you identify and resolve security group misconfigurations?
- How do you diagnose RDS performance issues?
- How do you troubleshoot a Spring Boot app that fails to start on ECS?
- How do you debug a Spring Boot app that cannot connect to RDS inside a VPC?
- How do you investigate memory leaks in a Spring Boot app running on ECS using CloudWatch?
- How do you troubleshoot SQS message duplication in a Spring Boot consumer?
- How do you debug slow DynamoDB queries?
- How do you troubleshoot ECS task failures (exit codes, OOM kills)?
- How do you investigate 5xx errors in ALB access logs? -->





## Final Merged AWS Interview Question List

<br/>

## 1. Cloud Fundamentals

### 1.1 Cloud Computing Basics

* What is **cloud computing**, and what are its key characteristics?
* What is the difference between **Public, Private, Hybrid, and Multi-Cloud** models?
* What is the difference between **on-demand provisioning, elasticity, and auto scaling**?
* What is the difference between **vertical scaling and horizontal scaling**?
* What is the difference between **scale-in and scale-out**, and when should each be used?

---

### 1.2 AWS Core Concepts

* What is AWS, and what are its key features?
* Explain the **AWS Shared Responsibility Model**, and how responsibilities differ across **IaaS, PaaS, and Serverless** services.
* What is the difference between **Regions and Availability Zones**, and how do they help achieve high availability?
* What is **Multi-AZ deployment**, and when should it be used?

---

### 1.3 High Availability and Fault Tolerance

* How do you design a **highly available and fault-tolerant architecture** in AWS?
* How do you ensure an application remains **highly available** during infrastructure failures?
* How does **cross-region replication** support high availability?
* What happens when **one Availability Zone fails**, and how should your system respond?
* Why is **failure isolation** important in distributed systems?
* How do **Availability Zones** ensure high availability?
* What is **high availability**, and how is it achieved in AWS architectures?
* What are **Service Level Agreements (SLA)** and **Service Level Objectives (SLO)** in a cloud environment?

---

## 2. Networking and Traffic Flow

### 2.1 VPC Fundamentals

* What is a **VPC**, and why is it needed?
* What are the core **components of a VPC**, and how do they work together?
  (subnets, route tables, IGW, NAT Gateway, NACLs, Security Groups)
* What is the **default VPC**, and what happens if you don't create one?
* What is the difference between **public and private subnets**, and how are they accessed?
* How does **traffic flow** from an internet user to backend services inside a VPC?
* What is the role of **route tables**, and how do they control traffic flow?

---

### 2.2 Connectivity and Routing

* What is the difference between **Internet Gateway (IGW)** and **NAT Gateway**, and when should each be used?
* What is the difference between **Elastic IP, Public IP, and Private IP**, and when should each be used?
* What is the difference between **VPC Peering and AWS Transit Gateway**, and when should each be used?
* What is **AWS PrivateLink**, and when should it be used instead of VPC Peering?
* What is **AWS Direct Connect**, and when should it be used?
* What is the difference between **AWS Direct Connect and VPN** connectivity?

---

### 2.3 DNS and Load Balancing

* What is **Amazon Route 53**, and why is it used?
* What routing policies are available in Route 53, including **proximity routing**?
* What happens if Route 53 fails, and how does failover work?
* What is a **Load Balancer** in AWS, and what are the different types?
  (ALB, NLB, CLB, GWLB)
* What is the difference between **Application Load Balancer (ALB)** and **Network Load Balancer (NLB)**?
* How does an **Application Load Balancer** route traffic?
* How does **SSL termination** work in load balancers?

---

### 2.4 Network Security

* What is the difference between **Security Groups and Network ACLs**, and how do they work together?
* How do you design **production-ready network security** across multiple layers?
* How do inbound rules, outbound rules, firewalls, and routing work together?

---

### 2.5 Secure Service Communication

* How does an application securely communicate with **S3 or SQS** using IAM roles and policies?
* What are the best practices for granting **least-privilege access** between AWS services?

---

## 3. Identity and Access Management (IAM)

### 3.1 IAM Fundamentals

* What is IAM, and why is it critical for AWS security?
* What is the difference between **IAM Roles and IAM Policies**?
* What are the different types of **IAM policies**, and when should each be used?
* What are the different types of **IAM roles**, and where are they commonly used?
* What is a **service principal** in IAM?

---

### 3.2 Secrets and Authentication

* What is **AWS Secrets Manager**, and why should it be used instead of storing credentials in code?
* How do you implement **Multi-Factor Authentication (MFA)** in AWS?
* How do you implement secure authentication for applications running on AWS?

---

## 4. Compute Services

### 4.1 EC2 and Compute Models

* What are the different **EC2 pricing models** (On-Demand, Reserved, Spot)?
* When should you choose **Spot Instances**, and what are typical use cases?
* What are the different ways to **connect to an EC2 instance**, and when should each be used?

---

### 4.2 Serverless and Event-Driven

* What are **Lambda triggers**, and how do they enable event-driven architectures?
* What is **serverless computing**, and how does it differ from traditional compute models?
* What are the **benefits** of serverless architecture?
* What are the **limitations** of AWS Lambda?
* What is **AWS API Gateway**, and how does it integrate with Lambda?
* How do you design and build a **serverless REST API** using AWS services?

---

## 5. Storage Services

### 5.1 Storage Fundamentals

* What is the difference between **Blob Storage and Object Storage**?
* How does **Amazon S3 store data internally**?
* What are the different **storage classes in S3**, and when should each be used?
* How do you decide which **storage service or storage class** to use?

---

### 5.2 Storage Comparison

* What is the difference between **Amazon S3, EBS, and EFS**, including typical use cases?

---

### 5.3 S3 Advanced Features

* What is **S3 lifecycle management**, and how does it reduce storage costs?
* What is **S3 Cross-Region Replication (CRR)**, and when should it be used?
* What is **S3 Transfer Acceleration**, and when should it be used?
* How do you **secure data at rest in S3**, including encryption options?
* How do you manage **public access control** for S3 objects?

---

## 6. Database Services

### 6.1 Database Selection

* What is the difference between **Amazon RDS and DynamoDB**, and when should each be used?
* What is the difference between **read replicas and Multi-AZ deployments** in RDS?
* When should you use **DynamoDB instead of RDS**?
* How do you design highly available database architectures in AWS?

---

### 6.2 DynamoDB Advanced Concepts

* What are **DynamoDB indexes**, and when are they needed?
* What are **DynamoDB streams**, and what use cases require them?

---

## 7. Security Services

### 7.1 Encryption and Key Management

* What is **AWS Key Management Service (KMS)**, and how is it used?
* What encryption types are supported by **AWS KMS**?

---

### 7.2 Threat Detection and Protection

* What is **AWS Shield**, and how does it protect against attacks?
* What is **Amazon GuardDuty**, and how does it detect threats?
* What is **AWS WAF**, and how does it protect web applications?
* What is **Amazon Inspector**, and what vulnerabilities does it detect?

---

## 8. Monitoring, CI/CD and DevOps Automation

### 8.1 Monitoring and Observability

* What is **Amazon CloudWatch**, and what metrics can it monitor?
* How do **CloudWatch Alarms** work?
* What is the difference between **CloudWatch Logs and CloudTrail**?
* What is **AWS X-Ray**, and when should it be used?
* How do you troubleshoot **performance issues** using CloudWatch?

---

### 8.2 CI/CD Fundamentals

* What is **CI/CD**, and why is it important in modern cloud development?
* What AWS services are commonly used for CI/CD pipelines?
  (CodeBuild, CodeDeploy, CodePipeline, GitHub Actions)

---

### 8.3 Deployment Strategies

* Explain the flow of a **typical CI/CD pipeline** in AWS.
* What is **Infrastructure as Code (IaC)**, and why is it important?
* What is the difference between **Terraform and CloudFormation**?
* What is **blue-green deployment**, and when should it be used?
* What is **canary deployment**, and how does it reduce deployment risk?

---

## 9. Containers and Orchestration

* What is **Docker**, and how is it used in cloud environments?
* What is **Amazon ECS**, and how does it work?
* What is **Amazon EKS**, and how does it work?
* What is the difference between **Amazon ECS and Amazon EKS**?
* What is **AWS Fargate**, and when should it be used?
* When should you use **EC2 launch type vs Fargate**?

---

## 10. Cost Optimization

* How do you optimize AWS costs in production environments?
* What is **AWS Cost Explorer**, and how is it used?
* What are **AWS Budgets**, and how do they help manage spending?
* What strategies help reduce **S3 storage costs**?

---

## 11. Disaster Recovery and Reliability

* What are the different **disaster recovery strategies** in AWS?
* What is the difference between **RTO and RPO**?
* How do you design **backup and recovery solutions**?
* How do you design **high availability across multiple AZs**?

---

## 12. Infrastructure and Governance

* What is **AWS CloudFormation**, and why is it used?
* What are **CloudFormation stacks and templates**?
* What is **AWS Systems Manager**, and what problems does it solve?
* What is **AWS Config**, and how does it help governance?
* What is **AWS Organizations**, and why is it used?
* What is a **Service Control Policy (SCP)**, and how does it restrict access?

---

## 13. Architecture and System Design (High-Value Section)

* How would you design a **highly available web application architecture** on AWS?
* How would you migrate an **on-premises application** to AWS?
* What is a **Hybrid Cloud architecture**, and how is it implemented using AWS?
* Why might some databases remain **on-premises** instead of moving to AWS?
* What are the **latency challenges** between on-premises systems and AWS resources?
* How do you design applications to handle **network latency** in hybrid environments?
* How would you design a **CI/CD pipeline** for microservices on AWS?
* How would you handle a **sudden spike in traffic**?
* How would you design a **disaster recovery solution** for a critical application?
* How would you secure **sensitive data** in AWS?
* How would you design a **serverless data processing pipeline**?
* How would you implement **blue-green deployment** for zero-downtime releases?

---

## 14. Troubleshooting and Debugging (Very High Interview Value)

### 14.1 Compute Troubleshooting

* How do you troubleshoot **EC2 connectivity issues**?
* How do you debug **Lambda function errors**?

---

### 14.2 Performance Troubleshooting

* How do you investigate **high latency** in API Gateway?
* How do you troubleshoot **SQS message processing delays**?

---

### 14.3 Infrastructure Troubleshooting

* How do you identify and resolve **security group misconfigurations**?
* How do you diagnose **RDS performance issues**?

---