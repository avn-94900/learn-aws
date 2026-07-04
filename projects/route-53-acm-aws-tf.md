Below is a complete Terraform project structure for the architecture shown in the video:

```text
Route53 Domain
      |
      v
Application Load Balancer (HTTP + HTTPS)
      |
      v
Target Group
      |
      v
EC2 Instance (Apache)
```

Resources created:

* VPC
* 2 Public Subnets
* Internet Gateway
* Route Table
* Security Groups
* EC2 Instance
* Target Group
* Application Load Balancer
* ACM Certificate
* Route53 Hosted Zone Records
* HTTPS Listener
* HTTP → HTTPS Redirect

---

# Project Structure

```text
aws-acm-demo/
│
├── provider.tf
├── variables.tf
├── vpc.tf
├── security-groups.tf
├── ec2.tf
├── alb.tf
├── route53.tf
├── acm.tf
├── outputs.tf
├── terraform.tfvars
└── userdata.sh
```

---

# provider.tf

```hcl
terraform {
  required_version = ">= 1.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

---

# variables.tf

```hcl
variable "aws_region" {
  default = "eu-central-1"
}

variable "vpc_cidr" {
  default = "12.0.0.0/16"
}

variable "public_subnet_1" {
  default = "12.0.1.0/24"
}

variable "public_subnet_2" {
  default = "12.0.2.0/24"
}

variable "domain_name" {
  description = "your domain"
}

variable "key_name" {
  description = "existing EC2 keypair"
}
```

---

# vpc.tf

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = "test-vpc"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "test-igw"
  }
}

resource "aws_subnet" "public_1a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_1
  availability_zone       = "${var.aws_region}a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-1a"
  }
}

resource "aws_subnet" "public_1b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_2
  availability_zone       = "${var.aws_region}b"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-1b"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "public-rt"
  }
}

resource "aws_route" "internet" {
  route_table_id         = aws_route_table.public.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
}

resource "aws_route_table_association" "subnet1" {
  subnet_id      = aws_subnet.public_1a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "subnet2" {
  subnet_id      = aws_subnet.public_1b.id
  route_table_id = aws_route_table.public.id
}
```

---

# security-groups.tf

```hcl
resource "aws_security_group" "ec2_sg" {
  name   = "ec2-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb_sg.id]
  }

  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "alb_sg" {
  name   = "alb-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port = 80
    to_port   = 80
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port = 443
    to_port   = 443
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

# userdata.sh

```bash
#!/bin/bash

apt-get update -y

apt-get install apache2 -y

echo "<h1>Hello from Terraform EC2</h1>" \
> /var/www/html/index.html

systemctl enable apache2
systemctl restart apache2
```

---

# ec2.tf

```hcl
data "aws_ami" "ubuntu" {

  most_recent = true

  owners = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {

  ami                    = data.aws_ami.ubuntu.id
  instance_type          = "t2.micro"

  subnet_id              = aws_subnet.public_1a.id

  vpc_security_group_ids = [
    aws_security_group.ec2_sg.id
  ]

  key_name = var.key_name

  user_data = file("userdata.sh")

  tags = {
    Name = "apache-server"
  }
}
```

---

# alb.tf

```hcl
resource "aws_lb" "alb" {

  name               = "test-alb"

  internal           = false

  load_balancer_type = "application"

  security_groups = [
    aws_security_group.alb_sg.id
  ]

  subnets = [
    aws_subnet.public_1a.id,
    aws_subnet.public_1b.id
  ]
}
```

---

# Target Group

```hcl
resource "aws_lb_target_group" "tg" {

  name = "test-tg"

  port     = 80
  protocol = "HTTP"

  vpc_id = aws_vpc.main.id

  health_check {
    path = "/"
  }
}

resource "aws_lb_target_group_attachment" "web" {

  target_group_arn = aws_lb_target_group.tg.arn

  target_id = aws_instance.web.id

  port = 80
}
```

---

# HTTP Listener

```hcl
resource "aws_lb_listener" "http" {

  load_balancer_arn = aws_lb.alb.arn

  port     = 80
  protocol = "HTTP"

  default_action {

    type = "redirect"

    redirect {

      port = "443"

      protocol = "HTTPS"

      status_code = "HTTP_301"
    }
  }
}
```

---

# acm.tf

```hcl
resource "aws_acm_certificate" "cert" {

  domain_name       = var.domain_name

  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}
```

---

# Route53 Hosted Zone Lookup

```hcl
data "aws_route53_zone" "zone" {

  name         = var.domain_name

  private_zone = false
}
```

---

# ACM Validation Record

```hcl
resource "aws_route53_record" "cert_validation" {

  for_each = {

    for dvo in aws_acm_certificate.cert.domain_validation_options :

    dvo.domain_name => {

      name  = dvo.resource_record_name

      record = dvo.resource_record_value

      type = dvo.resource_record_type
    }
  }

  zone_id = data.aws_route53_zone.zone.zone_id

  name    = each.value.name

  type    = each.value.type

  ttl     = 60

  records = [each.value.record]
}
```

---

# ACM Validation

```hcl
resource "aws_acm_certificate_validation" "validation" {

  certificate_arn = aws_acm_certificate.cert.arn

  validation_record_fqdns = [
    for record in aws_route53_record.cert_validation :
    record.fqdn
  ]
}
```

---

# HTTPS Listener

```hcl
resource "aws_lb_listener" "https" {

  load_balancer_arn = aws_lb.alb.arn

  port     = 443

  protocol = "HTTPS"

  certificate_arn =
    aws_acm_certificate_validation.validation.certificate_arn

  default_action {

    type = "forward"

    target_group_arn =
      aws_lb_target_group.tg.arn
  }
}
```

---

# Route53 Alias Record

```hcl
resource "aws_route53_record" "app" {

  zone_id = data.aws_route53_zone.zone.zone_id

  name = var.domain_name

  type = "A"

  alias {

    name = aws_lb.alb.dns_name

    zone_id = aws_lb.alb.zone_id

    evaluate_target_health = true
  }
}
```

---

# outputs.tf

```hcl
output "alb_dns_name" {

  value = aws_lb.alb.dns_name
}

output "website_url" {

  value = "https://${var.domain_name}"
}
```

---

# terraform.tfvars

```hcl
aws_region = "eu-central-1"

domain_name = "example.com"

key_name = "my-keypair"
```

---

# Commands

```bash
terraform init
```

```bash
terraform fmt
```

```bash
terraform validate
```

```bash
terraform plan
```

```bash
terraform apply
```

Destroy everything:

```bash
terraform destroy
```

## Interview Discussion Points

If this comes up in a Java Lead/DevOps interview, be ready to explain:

1. Why ACM certificate is attached to the ALB rather than EC2.
2. Why DNS validation is preferred over email validation.
3. Why ALB requires at least two subnets in different Availability Zones.
4. Difference between Route 53 Alias Record and CNAME.
5. Why HTTP listener redirects to HTTPS using a 301 redirect.
6. Security Group design (Internet → ALB → EC2) instead of exposing EC2 directly.
7. How ACM certificate auto-renewal works.
8. How you would extend this design to multiple EC2 instances with Auto Scaling Groups.
