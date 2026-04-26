# AWS Interview Questions List

## Cloud Computing Basics

- What is Cloud Computing?
- What are the differences between Public, Private, Hybrid, and Multi-Cloud models?
- Explain On-Demand, Auto-Scaling, and Elasticity
- What is the difference between Auto-Scaling vs Elasticity?
- Classify services into IaaS, PaaS, and SaaS with examples
- Give real-world examples of SaaS (Salesforce, Office 365, Amazon WorkMail)
- What factors should you consider when choosing between IaaS, PaaS, and SaaS solutions?

---

## AWS Storage Services

- What is the difference between Blob Storage and Object Storage?
- How does Amazon S3 store data?
- What are the storage classes in S3 (Hot, Cold, Archival/Glacier)?
- How do you decide which storage service or class to use? What factors influence the decision?
- Difference between EBS, S3, and EFS
- What are typical use cases for EBS, S3, and EFS?

---

## AWS Compute Options

- Compare EC2, Lambda, Elastic Beanstalk, ECS, and EKS
- What factors influence which compute option to choose?
- Advantages and disadvantages of EC2 vs Lambda vs ECS vs EKS
- Which option is better for monolithic applications? Which for microservices?
- What are the cost benefits associated with each compute option?
- What is an Auto Scaling Group (ASG)?
- Difference between auto-scaling and elasticity
- What is ASG (Auto Scaling Group) and how does it work?
- What are EC2 instance types?
- When would you choose Spot Instances? Provide use cases

---

## AWS Lambda and Serverless

- What is AWS Lambda?
- Why would you use Lambda instead of embedding the logic directly in the main application?
- What are the drawbacks of using Lambda inside the application code?
- How does Lambda help with separation of concerns?
- Provide real-time use cases for Lambda (image compression, sending welcome email, log processing)
- How is Lambda billed?
- What are Lambda triggers (events)? Name a few event sources

---

## Networking and Security

- What is a VPC? Why is it needed?
- What are the components of a VPC (subnets, route tables, NAT, IGW, NACLs, Security Groups)?
- What is the default VPC? What happens if you don't create one?
- How does a web application inside a VPC respond to user requests? Which component helps here?
- What is a Load Balancer in AWS? What are the different types (ALB, NLB, CLB, GWLB)?
- What is Amazon Route 53? Why is it used?
- What happens if Route 53 goes down? How does failover strategy work?
- What is proximity routing in Route 53?
- What is the difference between Security Groups and NACLs?
- What are firewalls, inbound rules, outbound rules, and route tables?

---

## IAM and Security Management

- What are IAM roles vs IAM policies?
- What is a service principal in IAM?
- How does an application communicate with S3 or SQS securely (IAM roles/policies)?
- What is AWS Secrets Manager?
- Why do we use Secrets Manager instead of storing credentials in code or config files?
- How do you integrate AWS Secrets Manager with a Spring Boot application? (Packages, configuration, property injection)

---

## Hybrid Cloud and Direct Connect

- What is AWS Direct Connect?
- Difference between Direct Connect and VPN
- Why would some databases remain on-premises instead of moving to AWS?
- Explain a Hybrid Cloud approach with AWS
- What are the challenges with latency between on-premises and AWS resources?
- How do Availability Zones (AZs) ensure high availability?
- What happens when one AZ fails?
- Difference between Availability Zone and Region

---

## Application Security and Network Security

- What security measures should you consider for AWS applications?
- What is the difference between application security and network security?
- Why do you need isolation of failures?
- What is high availability and how is it achieved in AWS?
- What are SLAs and SLOs in cloud context?

---

## Databases

- What is RDS? How is it different from self-managed databases?
- What database engines does RDS support?
- What is Amazon Aurora? How is it different from RDS?
- What is DynamoDB? When should you use it over RDS?
- Explain read replicas and Multi-AZ deployments in RDS

---

## Messaging and Queueing

- What is Amazon SQS?
- Difference between SQS Standard and FIFO queues
- What is Amazon SNS? How is it different from SQS?
- What is Amazon EventBridge?
- When to use SQS vs SNS vs EventBridge?
- What is message visibility timeout?
- What is a Dead Letter Queue (DLQ)?

---

## Monitoring and Logging

- What is Amazon CloudWatch?
- What metrics can CloudWatch monitor?
- How do CloudWatch Alarms work?
- What is AWS X-Ray? When would you use it?
- Difference between CloudWatch Logs and CloudTrail
- How do you troubleshoot performance issues using CloudWatch?

---

## CI/CD and DevOps

- What is CI/CD?
- What are AWS CI/CD services (CodeBuild, CodeDeploy, CodePipeline, GitHub Actions)?
- Explain the CI/CD pipeline flow
- What is Infrastructure as Code (IaC)?
- Difference between Terraform and CloudFormation
- What is blue/green deployment?
- What is canary deployment?

---

## Containers and Orchestration

- What is Docker?
- What is Amazon ECS?
- What is Amazon EKS?
- Difference between ECS and EKS
- What is AWS Fargate?
- When to use EC2 launch type vs Fargate in ECS?

---

## Cost Optimization

- What are EC2 pricing models (On-Demand, Reserved, Spot)?
- How do you optimize costs in AWS?
- What is AWS Cost Explorer?
- What are AWS Budgets?
- How do you reduce S3 storage costs?

---

## Disaster Recovery and High Availability

- What are the disaster recovery strategies in AWS?
- What is RTO and RPO?
- How do you design for high availability?
- What is Multi-AZ deployment?
- What is cross-region replication?

---

## Serverless Architecture

- What is serverless computing?
- What are the benefits of serverless architecture?
- What are the limitations of Lambda?
- What is AWS API Gateway?
- How do you build a serverless REST API?

---

## Advanced Topics

- What is AWS CloudFormation?
- What are CloudFormation stacks and templates?
- What is AWS Systems Manager?
- What is AWS Config?
- What is AWS Organizations?
- What is Service Control Policy (SCP)?
- What is AWS PrivateLink?
- What is VPC Peering vs Transit Gateway?

---

## Scenario-Based Questions

- Design a highly available web application architecture on AWS
- How would you migrate an on-premises application to AWS?
- Design a CI/CD pipeline for a microservices application
- How would you handle a sudden spike in traffic?
- Design a disaster recovery solution for a critical application
- How would you secure sensitive data in AWS?
- Design a serverless data processing pipeline
- How would you implement blue/green deployment for zero-downtime releases?

---

## Troubleshooting

- How do you troubleshoot EC2 instance connectivity issues?
- How do you debug Lambda function errors?
- How do you investigate high latency in API Gateway?
- How do you troubleshoot SQS message processing delays?
- How do you identify and resolve security group misconfigurations?
- How do you diagnose RDS performance issues?