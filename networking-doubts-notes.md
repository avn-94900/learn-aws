## AWS Networking Components — Quick Comparison

| Component              | Main Purpose                    | OSI Layer     | Understands / Features                                                                                     | Main Routing Logic                     | Common Targets                    |
| ---------------------- | ------------------------------- | ------------- | ---------------------------------------------------------------------------------------------------------- | -------------------------------------- | --------------------------------- |
| Route 53               | DNS routing                     | DNS Layer     | Domain names, health checks, failover routing, latency routing                                             | Domain → Endpoint mapping              | ALB, NLB, CloudFront, API Gateway |
| CloudFront             | CDN caching                     | Layer 7       | URL path, hostname, cookies, query params, caching, HTTPS, edge delivery                                   | Cached content + nearest edge location | S3, ALB, API Gateway              |
| Internet Gateway (IGW) | Connect VPC to internet         | Network level | Only internet connectivity, does NOT understand HTTP, headers, URLs, auth                                  | Connects VPC ↔ Internet                | VPC                               |
| NLB                    | High-performance load balancing | Layer 4       | IP address, port, TCP/UDP, ultra low latency, static IP                                                    | IP + Port based routing                | EC2, ECS                          |
| ALB                    | Smart web traffic routing       | Layer 7       | URL path, hostname, headers, cookies, query params, host/path routing                                      | HTTP/HTTPS request routing             | EC2, ECS, Lambda                  |
| API Gateway            | API management                  | Layer 7       | Authentication, authorization, throttling, API validation, headers, query params, transformations, caching | API policies + request routing         | Lambda, ALB, ECS                  |

---

## Typical Request Flows

### Web Application

```text id="e7l7wg"
User
  |
Route 53
  |
CloudFront
  |
Internet Gateway
  |
ALB
  |
EC2 / ECS
```

---

### Serverless API

```text id="5d0l90"
User
  |
Route 53
  |
API Gateway
  |
Lambda
```

---

### High Performance TCP System

```text id="gbt8w9"
User
  |
Route 53
  |
Internet Gateway
  |
NLB
  |
EC2
```

---

## Layer 4 vs Layer 7

| Feature                  | Layer 4 (Transport Layer)   | Layer 7 (Application Layer)        |
| ------------------------ | --------------------------- | ---------------------------------- |
| AWS Service              | NLB                         | ALB, API Gateway                   |
| Protocols                | TCP, UDP                    | HTTP, HTTPS, WebSocket             |
| Understands HTTP Data    | No                          | Yes                                |
| Understands URL Path     | No                          | Yes                                |
| Understands Headers      | No                          | Yes                                |
| Understands Cookies      | No                          | Yes                                |
| Understands Query Params | No                          | Yes                                |
| Routing Based On         | IP + Port                   | Path, Hostname, Headers            |
| Performance              | Very high                   | Moderate                           |
| Latency                  | Very low                    | Slightly higher                    |
| Best For                 | Raw network traffic         | Web applications & APIs            |
| Common Usage             | Gaming, TCP apps, streaming | Websites, REST APIs, microservices |

---

## NLB vs ALB vs API Gateway

| Feature                         | NLB (Network Load Balancer)       | ALB (Application Load Balancer) | API Gateway                  |
| ------------------------------- | --------------------------------- | ------------------------------- | ---------------------------- |
| Main Purpose                    | Fast TCP/UDP traffic distribution | Smart HTTP traffic distribution | API management               |
| OSI Layer                       | Layer 4                           | Layer 7                         | Layer 7                      |
| Protocols                       | TCP, UDP                          | HTTP, HTTPS, WebSocket          | HTTP, HTTPS, WebSocket       |
| Understands                     | IP + Port                         | URL, Hostname, Headers, Cookies | Full API request details     |
| Path Routing                    | No                                | Yes                             | Yes                          |
| Host-Based Routing              | No                                | Yes                             | Yes                          |
| Authentication                  | No                                | Basic/Limited                   | Advanced                     |
| Rate Limiting                   | No                                | No                              | Yes                          |
| Request Validation              | No                                | No                              | Yes                          |
| Request/Response Transformation | No                                | No                              | Yes                          |
| Caching                         | No                                | No                              | Yes                          |
| Lambda Integration              | No                                | Supported                       | Native                       |
| Performance                     | Very high, ultra low latency      | High                            | Moderate                     |
| Best For                        | TCP/UDP apps, gaming, streaming   | Web apps, microservices         | Public APIs, serverless APIs |
| Common Backends                 | EC2, ECS                          | EC2, ECS, Lambda                | Lambda, ALB, ECS, EC2        |
| Pricing                         | Based on traffic + hours          | Based on traffic + hours        | Based on API requests        |


---

## Simple Understanding

| Service     | Think As                   |
| ----------- | -------------------------- |
| NLB         | Fast traffic forwarder     |
| ALB         | Smart web traffic router   |
| API Gateway | Full API management system |


---

## Simple Analogy

| Component   | Analogy                            |
| ----------- | ---------------------------------- |
| Route 53    | GPS / Phonebook                    |
| CloudFront  | Nearby cached warehouse            |
| IGW         | Building entrance gate             |
| NLB         | Traffic signal forwarding vehicles |
| ALB         | Smart receptionist                 |
| API Gateway | Security desk + API manager        |



<br/>
<br/>
<br/>

## AWS Networking & Traffic Components — Quick Notes

## Route 53

### Purpose

AWS DNS service.

Converts:

```text id="1v7l4v"
Domain Name -> IP Address
```

Example:

```text id="xgk2t1"
api.example.com -> ALB/API Gateway/CloudFront
```

---

### Main Features

* DNS management
* Domain registration
* Health checks
* Traffic routing
* Failover routing
* Latency-based routing
* Weighted routing

---

### Common Targets

* ALB
* NLB
* CloudFront
* API Gateway
* S3 static website

---

### Request Flow

```text id="w8x7n0"
Client
  |
Route 53 DNS lookup
  |
Returns target endpoint
```

---

## CloudFront

### Purpose

AWS CDN (Content Delivery Network).

Caches content closer to users for low latency.

---

### Works With

* S3
* ALB
* API Gateway
* EC2
* Media streaming

---

### Benefits

* Faster content delivery
* Reduced latency
* Global edge locations
* DDoS protection
* HTTPS support
* Content caching

---

### Can Cache

* Images
* Videos
* CSS/JS
* API responses
* Static websites

---

### Typical Flow

```text id="cz7ncc"
Client
  |
Nearest CloudFront Edge Location
  |
Origin (ALB/S3/API Gateway)
```

---

### Common Origins

* S3 bucket
* ALB
* API Gateway
* EC2

---

## Internet Gateway (IGW)

### Purpose

Provides internet connectivity between VPC and Internet.

---

### Attached To

```text id="xow2zy"
VPC
```

---

### Enables

* Public internet access
* Outbound internet from instances
* Inbound internet to public resources

---

### Requirements For Public EC2

* Internet Gateway attached to VPC
* Public IP / Elastic IP
* Route table with:

```text id="wvng3f"
0.0.0.0/0 -> IGW
```

---

### Does NOT

* Route based on URL
* Inspect HTTP headers
* Do authentication
* Balance traffic
* Understand HTTP/HTTPS

---

### Works At

```text id="n7a84l"
Network connectivity level
```

---

## Elastic Load Balancer (ELB)

### Purpose

Distributes incoming traffic across multiple targets.

---

### Supports

* EC2
* ECS
* Containers
* IP targets
* Lambda (ALB)

---

### Benefits

* High availability
* Fault tolerance
* Auto scaling support
* Health checks

---

## Types of Load Balancers

| Type | Layer   | Protocol   |
| ---- | ------- | ---------- |
| ALB  | Layer 7 | HTTP/HTTPS |
| NLB  | Layer 4 | TCP/UDP    |

---

## Network Load Balancer (NLB)

### Layer

```text id="53kgt7"
Layer 4 (Transport Layer)
```

---

### Understands

* IP
* Port
* TCP
* UDP

---

### Does NOT Understand

* URL path
* Hostname
* Headers
* Cookies
* Query params

---

### Best For

* Very high performance
* Low latency
* TCP/UDP traffic
* Static IP
* Millions of connections

---

### Routing Based On

```text id="k0m1fm"
IP + Port
```

---

## Application Load Balancer (ALB)

### Layer

```text id="qocv5k"
Layer 7 (Application Layer)
```

---

### Understands

* URL path
* Hostname
* Headers
* Cookies
* Query params

---

### Supports

* Path-based routing
* Host-based routing
* Microservices
* REST APIs
* Web applications

---

### Example Routing

```text id="40f5hf"
/api/*      -> Service A
/orders/*   -> Service B
/images/*   -> Service C
```

---

### Best For

* HTTP/HTTPS applications
* Web apps
* APIs
* Container-based apps

---

## API Gateway

### Purpose

Fully managed API management service.

---

### Can Do

* Authentication
* Authorization
* Rate limiting
* Request validation
* API versioning
* Monitoring
* Request/response transformation
* Caching

---

### Common Backends

* Lambda
* ECS
* EC2
* ALB

---

### Best For

* Serverless APIs
* Public APIs
* Secure APIs
* Microservices APIs

---

## API Gateway vs ALB

| Feature           | API Gateway | ALB         |
| ----------------- | ----------- | ----------- |
| API Management    | Yes         | No          |
| Authentication    | Advanced    | Limited     |
| Rate Limiting     | Yes         | No          |
| Path Routing      | Yes         | Yes         |
| Works With Lambda | Native      | Supported   |
| Best For          | APIs        | Web traffic |

---

## Layer 4 vs Layer 7

| Feature             | Layer 4   | Layer 7          |
| ------------------- | --------- | ---------------- |
| Protocol Level      | TCP/UDP   | HTTP/HTTPS       |
| Understands URL     | No        | Yes              |
| Understands Headers | No        | Yes              |
| Routing Capability  | IP + Port | Path/Host/Header |
| AWS Service         | NLB       | ALB              |

---

## Common End-to-End Request Flows

---

### Route 53 → CloudFront → ALB → EC2

```text id="s8v9gj"
Client
  |
Route 53
  |
CloudFront
  |
Internet Gateway
  |
ALB
  |
EC2/ECS
```

Used for:

* Web applications
* Global websites
* Low latency apps

---

### Route 53 → API Gateway → Lambda

```text id="9t7mr7"
Client
  |
Route 53
  |
API Gateway
  |
Lambda
```

Used for:

* Serverless APIs
* REST APIs

---

### Route 53 → CloudFront → API Gateway

```text id="2zvqfx"
Client
  |
Route 53
  |
CloudFront
  |
API Gateway
  |
Lambda/Backend
```

Used for:

* Cached APIs
* Global API acceleration

---

### Route 53 → NLB → EC2

```text id="hgw4n8"
Client
  |
Route 53
  |
Internet Gateway
  |
NLB
  |
EC2
```

Used for:

* TCP/UDP systems
* High-performance networking

---

## One-Line Interview Definitions

### Route 53

> AWS managed DNS service for routing domain names to AWS resources.

---

### CloudFront

> AWS CDN service that caches content at edge locations for low latency delivery.

---

### Internet Gateway

> Enables internet connectivity for a VPC.

---

### NLB

> Layer 4 load balancer routing traffic using IP and port.

---

### ALB

> Layer 7 load balancer routing HTTP/HTTPS traffic using application-level data.

---

### API Gateway

> Fully managed service for creating, securing, and managing APIs.

