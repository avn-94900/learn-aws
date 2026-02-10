# DevOps Tools and Practices

## CI/CD Fundamentals

**What is CI/CD?**

CI/CD (Continuous Integration/Continuous Deployment) automates code integration, testing, and deployment to improve development velocity and reliability.

**CI/CD Pipeline Steps**

1. Source: Code stored in version control (Git, CodeCommit)
2. Build: Compile and test using build automation tools (CodeBuild, Jenkins)
3. Deploy: Rollout to environments using deployment automation (CodeDeploy, Elastic Beanstalk)
4. Monitor: Track performance and errors using observability tools (CloudWatch, Prometheus)

**Benefits of CI/CD**

- Faster time to market
- Reduced manual errors
- Improved code quality through automated testing
- Quick rollback capability
- Consistent deployment process

---

## AWS CI/CD Services

| Service | Purpose | Key Features |
|---------|---------|--------------|
| CodeCommit | Source control | Git-based repositories, integrated with AWS |
| CodeBuild | Build automation | Compile, test, produce artifacts |
| CodeDeploy | Deployment automation | Blue/green deployments, rolling updates |
| CodePipeline | Pipeline orchestration | Automated workflow from source to production |

**Example AWS CI/CD Pipeline**

1. Developer pushes code to CodeCommit
2. CodePipeline detects change and triggers CodeBuild
3. CodeBuild runs tests and creates artifacts
4. CodeDeploy deploys to EC2, ECS, or Lambda
5. CloudWatch monitors application performance

---

## Version Control and Collaboration

| Tool | Purpose | Key Features |
|------|---------|--------------|
| Git | Distributed version control | Branching, merging, local repositories |
| GitHub | Git hosting platform | Pull requests, code review, collaboration |
| GitHub Actions | CI/CD integrated with GitHub | Workflows triggered by events, marketplace actions |
| GitLab | Complete DevOps platform | Source control, CI/CD, issue tracking |

**Git Best Practices**

- Use feature branches for development
- Write meaningful commit messages
- Keep commits small and focused
- Review code before merging
- Tag releases for version tracking

---

## Build and Artifact Management

| Tool | Purpose | Key Features |
|------|---------|--------------|
| Jenkins | Open source automation server | Plugins, distributed builds, pipeline as code |
| JFrog Artifactory | Binary repository manager | Maven, npm, Docker, version control for artifacts |
| Nexus | Artifact repository | Component management, proxy for remote repos |

**Why Use Artifact Repositories?**

- Store versioned build artifacts
- Enable reproducible builds
- Cache dependencies for faster builds
- Centralized binary management
- Support for multiple package formats

---

## Infrastructure as Code

| Tool | Type | Description | Use Case |
|------|------|-------------|----------|
| Terraform | Declarative IaC | Multi-cloud infrastructure provisioning | AWS, Azure, GCP, infrastructure versioning |
| Ansible | Agentless configuration | Configuration management and automation | OS configuration, application deployment |
| Puppet | Agent-based configuration | Large-scale infrastructure management | Enterprise infrastructure, compliance |
| CloudFormation | AWS-native IaC | AWS resource provisioning | AWS-only deployments, tight AWS integration |

**Infrastructure as Code Benefits**

- Version-controlled infrastructure
- Reproducible environments
- Faster provisioning
- Reduced manual errors
- Documentation through code

**Terraform vs CloudFormation**

| Feature | Terraform | CloudFormation |
|---------|-----------|----------------|
| Cloud Support | Multi-cloud | AWS only |
| State Management | External state file | AWS-managed |
| Language | HCL (HashiCorp Configuration Language) | JSON/YAML |
| Modules | Terraform Registry | CloudFormation Stack Sets |
| Cost | Free (open source) | Free (AWS service) |

---

## Containerization and Orchestration

**Docker**

- Container runtime for packaging applications with dependencies
- Portable units from development to production
- Consistent environments across systems
- Lightweight compared to virtual machines

**Docker Best Practices**

- Use official base images
- Minimize image layers
- Use .dockerignore to exclude unnecessary files
- Run containers as non-root users
- Tag images with version numbers

**Kubernetes (K8s)**

- Container orchestration platform
- Automated scheduling, scaling, and management
- Service discovery and load balancing
- Self-healing and automatic rollouts/rollbacks

**Kubernetes Components**

| Component | Purpose |
|-----------|---------|
| Pods | Smallest deployable units containing containers |
| Deployments | Manage replicas and updates |
| Services | Expose pods with stable endpoints |
| ConfigMaps | Configuration data separate from images |
| Secrets | Sensitive data like passwords |
| Ingress | HTTP/HTTPS routing to services |

---

## AWS Container Services

**Amazon ECS (Elastic Container Service)**

- Fully managed container orchestration
- Deep AWS integration
- Launch types: EC2 (manage instances) or Fargate (serverless)
- Simple to set up and manage

**Amazon EKS (Elastic Kubernetes Service)**

- Managed Kubernetes service
- Runs upstream Kubernetes
- Multi-cloud portability
- Large ecosystem and tooling

**ECS vs EKS Comparison**

| Factor | ECS | EKS |
|--------|-----|-----|
| Learning Curve | Easier to learn | More complex |
| AWS Integration | Deep integration | Standard Kubernetes |
| Portability | AWS-specific | Multi-cloud portable |
| Ecosystem | AWS-focused | Large Kubernetes ecosystem |
| Management Overhead | Lower | Higher |
| Cost | Lower | Higher (control plane cost) |
| Use Case | AWS-native workloads, simpler apps | Multi-cloud, Kubernetes expertise, complex workloads |

---

## Web Servers and Proxies

**NGINX**

- High-performance web server
- Reverse proxy and load balancer
- TLS termination
- Static content serving
- Rate limiting and caching

**NGINX Common Uses**

- Reverse proxy to application servers
- Load balancing across backends
- SSL/TLS termination
- Serving static files
- API gateway functionality
- WebSocket proxy

**NGINX vs Application Load Balancer**

| Feature | NGINX | ALB |
|---------|-------|-----|
| Deployment | Self-managed on EC2 | Fully managed AWS service |
| Flexibility | High customization | Limited to AWS features |
| Cost | EC2 instance costs | Pay per hour and data processed |
| Maintenance | Requires patching and updates | AWS-managed |
| Use Case | Custom requirements, on-premises | Simple setup, AWS-native |

---

## Monitoring and Observability

**What You Need for Observability**

- Metrics: CPU, latency, request rates, custom business metrics
- Logs: Application and infrastructure logs
- Traces: Distributed request tracing across services
- Dashboards: Visual representation of system health
- Alerts: Automated notifications for anomalies
- Actions: Automated responses to alerts

---

## AWS Native Monitoring

**Amazon CloudWatch**

| Feature | Purpose |
|---------|---------|
| Metrics | System and custom metrics (CPU, memory, custom counters) |
| Logs | Log aggregation and storage |
| Alarms | Threshold-based alerting |
| Dashboards | Visualization of metrics and logs |
| Logs Insights | Query and analyze logs with SQL-like syntax |
| Events/EventBridge | Event-driven automation |

**AWS X-Ray**

- Distributed tracing for microservices
- Analyze latency and bottlenecks
- Service map visualization
- Request and error tracking
- Integration with Lambda, ECS, API Gateway

**CloudWatch Use Cases**

- Alert on abnormal traffic or resource usage
- Trigger Auto Scaling based on metrics
- Troubleshoot issues using log insights
- Create custom dashboards for operations
- Set up alarms for SLA monitoring

---

## Third-Party Monitoring Tools

| Tool | Purpose | Key Features |
|------|---------|--------------|
| Splunk | Commercial log/metric/tracing platform | Enterprise SIEM, advanced analytics, machine learning |
| Prometheus | Open source metrics collection | Time-series database, powerful query language (PromQL) |
| Grafana | Open source dashboards | Visualization, multiple data sources, alerting |
| ELK Stack | Log aggregation and search | Elasticsearch (storage), Logstash (processing), Kibana (visualization) |
| EFK Stack | Alternative to ELK | Elasticsearch, Fluentd (lighter than Logstash), Kibana |
| Jaeger | Distributed tracing | Open source, OpenTracing compatible |
| Zipkin | Distributed tracing | Request flow analysis, latency tracking |
| OpenTelemetry | Instrumentation framework | Metrics, traces, logs, vendor-neutral |

**Prometheus + Grafana Stack**

- Prometheus: Collects and stores metrics
- Alertmanager: Handles alerts from Prometheus
- Grafana: Creates dashboards and visualizations
- Common in Kubernetes environments

**ELK/EFK Stack**

- Elasticsearch: Stores and indexes logs
- Logstash/Fluentd: Collects and processes logs
- Kibana: Searches and visualizes logs
- Popular for centralized logging

---

## Observability Best Practices

**What to Monitor**

- How many services are running: Orchestration metrics (ECS/EKS API, CloudWatch, Prometheus)
- Which resources are spiking: CPU and memory per service
- Average response time per API: Instrument endpoints with custom metrics
- Request counts and top features: Request metrics by route
- Error rates and types: Application and HTTP error codes

**Taking Action from Alarms**

- CloudWatch Alarm to Auto Scaling policy: Add ECS tasks or EC2 instances
- Alarm to SNS: Send notifications to email, Slack, PagerDuty
- Alarm to Lambda: Run remedial actions (flush cache, rotate logs, restart services)
- Alarm to SSM Run Command: Patch or restart services automatically

---

## Example End-to-End Workflow

**Image Processing Pipeline**

1. User uploads image to S3
2. S3 event triggers Lambda function for image resizing
3. Lambda writes metadata to RDS database
4. Lambda sends message to SQS for downstream analytics
5. Analytics consumers (ECS service) poll SQS in batches with long polling
6. Consumers process and store results in Redshift or S3
7. Prometheus/Grafana and CloudWatch display metrics
8. CloudWatch alarm on queue age scales ECS tasks via Auto Scaling

**Key Protections**

- Idempotency keys on jobs to prevent duplicate processing
- Dead Letter Queues for poison messages
- Visibility timeout tuned to processing time
- Batch size optimized for throughput
- Alerts for oldest message age

---

## DevOps Interview Checklist

**Infrastructure**

- Use Terraform for infrastructure as code
- Provision VPC, subnets, security groups, SQS, Lambda declaratively
- Version and review infrastructure changes
- Use modules for reusable components

**CI/CD**

- Use GitHub Actions or Jenkins for continuous integration
- Build and push artifacts to JFrog Artifactory or Docker Registry
- Automate testing (unit, integration, security scans)
- Implement deployment automation with rollback capability

**Containers**

- Use Docker for reproducible application images
- Optimize Dockerfile for smaller images and faster builds
- Orchestrate with ECS Fargate for simplicity or EKS for Kubernetes portability
- Use multi-stage builds to reduce image size

**Monitoring**

- Instrument applications with OpenTelemetry for traces
- Use Prometheus for metrics and Grafana for dashboards
- Use ELK/EFK stack for centralized logging
- Alternative: CloudWatch and X-Ray for AWS-native monitoring
- Set up alerts for critical metrics and SLA violations

**Configuration Management**

- Use Ansible for agentless configuration management
- Store configuration in code (infrastructure as code)
- Separate configuration from code using environment variables
- Use Secrets Manager for sensitive data

**Best Practices**

- Implement automated testing at all levels
- Use feature flags for gradual rollouts
- Practice blue/green or canary deployments
- Maintain runbooks for incident response
- Regular security scanning and patching
- Document architecture and procedures