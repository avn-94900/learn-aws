# AWS Certificate Manager (ACM) + Route 53 + Application Load Balancer + EC2

## Complete Hands-on Notes (Step-by-Step Documentation)

---

# Objective

Configure a secure website using:

```
User
  |
  v
Route 53 (Domain)
  |
  v
Application Load Balancer (ALB)
  |
  v
EC2 Instance (Apache Web Server)
```

Then:

* Enable HTTPS using AWS Certificate Manager (ACM)
* Attach SSL certificate to ALB
* Redirect HTTP → HTTPS

---

# Final Architecture

```text
Internet User
      |
      v
Domain (example.com)
      |
      v
Route 53
      |
      v
Application Load Balancer
      |
      v
Target Group
      |
      v
EC2 Instance (Apache)
```

---

# Prerequisites

Before starting:

* AWS Account
* Purchased Domain

  * GoDaddy
  * Namecheap
  * Google Domains
  * AWS Route 53

---

# Step 1: Create VPC

## Navigate

```text
AWS Console
→ VPC
→ Create VPC
```

## Configuration

| Property  | Value       |
| --------- | ----------- |
| Name      | test-vpc    |
| IPv4 CIDR | 12.0.0.0/16 |
| Tenancy   | Default     |

Click:

```text
Create VPC
```

---

# Step 2: Create Public Subnets

## Navigate

```text
VPC
→ Subnets
→ Create Subnet
```

Select:

```text
test-vpc
```

---

## Subnet 1

| Property | Value                 |
| -------- | --------------------- |
| Name     | test-public-subnet-1a |
| AZ       | eu-central-1a         |
| CIDR     | 12.0.1.0/24           |

---

## Subnet 2

| Property | Value                 |
| -------- | --------------------- |
| Name     | test-public-subnet-1b |
| AZ       | eu-central-1b         |
| CIDR     | 12.0.2.0/24           |

Click:

```text
Create Subnet
```

---

# Why Two Subnets?

For High Availability.

If one Availability Zone fails:

```text
AZ-1A ❌
AZ-1B ✅
```

Application remains available.

---

# Step 3: Create Internet Gateway

## Navigate

```text
VPC
→ Internet Gateways
→ Create Internet Gateway
```

Name:

```text
test-igw
```

Click:

```text
Create
```

---

## Attach to VPC

Select:

```text
Actions
→ Attach to VPC
```

Choose:

```text
test-vpc
```

Click:

```text
Attach
```

---

# Why Internet Gateway?

Without IGW:

```text
EC2 cannot access Internet
Users cannot access EC2
```

IGW provides Internet connectivity.

---

# Step 4: Create Route Table

## Navigate

```text
VPC
→ Route Tables
→ Create Route Table
```

Configuration:

| Property | Value          |
| -------- | -------------- |
| Name     | test-rt-public |
| VPC      | test-vpc       |

Click:

```text
Create
```

---

# Associate Subnets

Open Route Table

```text
Subnet Associations
→ Edit
```

Select:

```text
test-public-subnet-1a
test-public-subnet-1b
```

Save.

---

# Add Internet Route

Open:

```text
Routes
→ Edit Routes
→ Add Route
```

Add:

| Destination | Target   |
| ----------- | -------- |
| 0.0.0.0/0   | test-igw |

Save.

---

# Why 0.0.0.0/0?

Means:

```text
Allow traffic from anywhere
```

Internet Route.

---

# Step 5: Launch EC2 Instance

## Navigate

```text
EC2
→ Instances
→ Launch Instance
```

---

## Basic Details

| Property      | Value    |
| ------------- | -------- |
| Name          | test-ec2 |
| AMI           | Ubuntu   |
| Instance Type | t2.micro |

---

## Key Pair

Choose:

```text
Existing Key Pair
```

OR

```text
Create New Key Pair
```

Download PEM file.

---

# Network Configuration

Click:

```text
Edit
```

Choose:

| Property  | Value                 |
| --------- | --------------------- |
| VPC       | test-vpc              |
| Subnet    | test-public-subnet-1a |
| Public IP | Enable                |

---

# Security Group

Allow:

| Type | Port |
| ---- | ---- |
| SSH  | 22   |
| HTTP | 80   |

Source:

```text
Anywhere
```

---

# User Data Script

Paste in:

```text
Advanced Details
→ User Data
```

```bash
#!/bin/bash

apt update -y
apt install apache2 -y

echo "<h1>Welcome from EC2 Instance</h1>" > /var/www/html/index.html

systemctl restart apache2
```

Launch Instance.

---

# Verify EC2

Copy:

```text
Public IP
```

Open Browser:

```text
http://PUBLIC-IP
```

Expected:

```text
Welcome from EC2 Instance
```

---

# Step 6: Create Target Group

## Navigate

```text
EC2
→ Target Groups
→ Create Target Group
```

Configuration:

| Property | Value     |
| -------- | --------- |
| Type     | Instances |
| Name     | test-tg   |
| Protocol | HTTP      |
| Port     | 80        |
| VPC      | test-vpc  |

Click:

```text
Next
```

---

# Register Target

Select:

```text
test-ec2
```

Click:

```text
Include as Pending
```

Then:

```text
Create Target Group
```

---

# Why Target Group?

```text
Load Balancer
      |
      v
Target Group
      |
      v
EC2 Instances
```

ALB forwards traffic to Target Group.

---

# Step 7: Create Application Load Balancer

## Navigate

```text
EC2
→ Load Balancers
→ Create Load Balancer
```

Choose:

```text
Application Load Balancer
```

---

# Configuration

| Property | Value           |
| -------- | --------------- |
| Name     | test-lb         |
| Scheme   | Internet Facing |
| IP Type  | IPv4            |
| VPC      | test-vpc        |

---

# Select Subnets

Choose:

```text
test-public-subnet-1a
test-public-subnet-1b
```

---

# Create Security Group

Allow:

| Type | Port |
| ---- | ---- |
| SSH  | 22   |
| HTTP | 80   |

Source:

```text
Anywhere
```

---

# Listener

```text
HTTP : 80
```

---

# Target Group

Choose:

```text
test-tg
```

Click:

```text
Create Load Balancer
```

Wait until:

```text
Status = Active
```

---

# Verify Load Balancer

Copy:

```text
ALB DNS Name
```

Example:

```text
test-lb-123456.amazonaws.com
```

Open Browser.

Expected:

```text
Welcome from EC2 Instance
```

---

# Step 8: Create Route 53 Hosted Zone

## Navigate

```text
Route 53
→ Hosted Zones
→ Create Hosted Zone
```

---

# Configuration

| Property | Value              |
| -------- | ------------------ |
| Domain   | yourdomain.com     |
| Type     | Public Hosted Zone |

Click:

```text
Create Hosted Zone
```

---

# Records Created Automatically

AWS creates:

```text
NS Record
SOA Record
```

---

# Step 9: Update Domain Registrar

Copy:

```text
NS Records
```

From Route 53.

Go to:

```text
GoDaddy
Namecheap
Google Domains
```

Replace old nameservers with Route 53 nameservers.

Example:

```text
ns-123.awsdns.com
ns-456.awsdns.net
ns-789.awsdns.org
ns-111.awsdns.co.uk
```

Save.

Wait:

```text
15 mins to 48 hrs
```

DNS propagation.

---

# Step 10: Point Domain to ALB

Inside Hosted Zone:

```text
Create Record
```

---

Choose:

```text
A Record
```

Enable:

```text
Alias = Yes
```

Target:

```text
Application Load Balancer
```

Select:

```text
test-lb
```

Create Record.

---

# Verify Domain

Open:

```text
http://yourdomain.com
```

Expected:

```text
Welcome from EC2 Instance
```

---

# Step 11: Request SSL Certificate

## Navigate

```text
Certificate Manager (ACM)
→ Request Certificate
```

---

Choose:

```text
Public Certificate
```

Click:

```text
Next
```

---

# Domain Name

Example:

```text
yourdomain.com
```

---

# Validation Method

Choose:

```text
DNS Validation
```

---

# Key Algorithm

```text
RSA 2048
```

Click:

```text
Request
```

---

# Step 12: Validate Certificate

Open Certificate.

Click:

```text
Create Records in Route 53
```

AWS automatically creates:

```text
CNAME Validation Record
```

Wait few minutes.

Status becomes:

```text
Issued
```

---

# Step 13: Add HTTPS Listener to ALB

## Navigate

```text
Load Balancer
→ Listeners
→ Add Listener
```

---

Configuration:

| Property | Value |
| -------- | ----- |
| Protocol | HTTPS |
| Port     | 443   |

---

Forward To:

```text
test-tg
```

---

# Attach ACM Certificate

Choose:

```text
yourdomain.com certificate
```

Click:

```text
Add
```

---

# Step 14: Update Security Group

Open ALB Security Group.

Add Inbound Rule:

| Type  | Port |
| ----- | ---- |
| HTTPS | 443  |

Source:

```text
Anywhere
```

Save.

---

# Verify HTTPS

Open:

```text
https://yourdomain.com
```

Expected:

```text
Secure Connection
🔒 Lock Symbol
```

Certificate is working.

---

# Step 15: Redirect HTTP → HTTPS

Navigate:

```text
Load Balancer
→ Listeners
→ HTTP : 80
→ Rules
```

Edit Rule.

---

Current:

```text
Forward to Target Group
```

Change To:

```text
Redirect
```

Configuration:

| Property    | Value |
| ----------- | ----- |
| Protocol    | HTTPS |
| Port        | 443   |
| Status Code | 301   |

Save.

---

# Verify Redirect

Open:

```text
http://yourdomain.com
```

Automatically redirects to:

```text
https://yourdomain.com
```

---

# Security Groups Summary

## EC2

| Port | Purpose |
| ---- | ------- |
| 22   | SSH     |
| 80   | HTTP    |

---

## Load Balancer

| Port | Purpose      |
| ---- | ------------ |
| 80   | HTTP         |
| 443  | HTTPS        |
| 22   | Optional SSH |

---

# Important Interview Questions

### Why use ACM?

* Free SSL certificates
* Automatic renewal
* Easy integration with ALB

---

### Why use Route 53?

* DNS management
* Health checks
* Routing policies

---

### Why attach SSL to ALB instead of EC2?

Because:

```text
SSL Termination at ALB
```

Benefits:

* Less CPU usage on EC2
* Easier certificate management
* Centralized security

---

### Why DNS Validation?

* Automated
* Faster
* Easier with Route 53

---

### Difference between HTTP and HTTPS?

| HTTP          | HTTPS     |
| ------------- | --------- |
| Port 80       | Port 443  |
| Not encrypted | Encrypted |
| Less secure   | Secure    |
| No SSL        | SSL/TLS   |

---

# Final Flow

```text
User
 |
 | HTTPS Request
 v
Route 53
 |
 v
Application Load Balancer
 |
 | SSL Termination
 v
Target Group
 |
 v
EC2 Instance (Apache)
```

This is the exact end-to-end setup demonstrated in the video and is a very common production pattern used in AWS environments.
