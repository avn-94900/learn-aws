For a **production-ready version**, I would not blindly copy the video setup.

### Problems with the video setup

The demo is good for learning, but in production:

❌ CIDR `12.0.0.0/16` should not be used (public address range)

❌ EC2 directly in public subnet

❌ SSH open to `0.0.0.0/0`

❌ Single EC2 instance

❌ No Auto Scaling

❌ No CloudWatch monitoring

❌ No access logs

❌ No WAF

❌ No SSM Session Manager

❌ Hardcoded values

❌ No tagging strategy

❌ No remote Terraform state

---

# Production-ready Architecture

```text
Internet
   |
   v
Route53
   |
   v
ALB (Public Subnets)
   |
   v
Target Group
   |
   v
Auto Scaling Group
   |
   v
EC2 (Private Subnets)

ACM Certificate
CloudWatch
SSM
```

However, if your goal is:

> "Create Terraform for exactly the ACM + Route53 + ALB exercise shown in the video"

then I would simplify and make it clean, modular, and interview-friendly.

---

# Folder Structure

```text
aws-acm-demo/

main.tf
variables.tf
network.tf
security.tf
ec2.tf
alb.tf
acm.tf
route53.tf
outputs.tf
terraform.tfvars
userdata.sh
```

---

# variables.tf

```hcl
variable "region" {
  default = "eu-central-1"
}

variable "domain_name" {}

variable "key_name" {}

variable "instance_type" {
  default = "t3.micro"
}
```

---

# main.tf

```hcl
terraform {

  required_version = ">=1.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~>5.50"
    }
  }
}

provider "aws" {
  region = var.region
}
```

---

# network.tf

```hcl
resource "aws_vpc" "main" {

  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags = {
    Name = "acm-demo-vpc"
  }
}

resource "aws_internet_gateway" "igw" {

  vpc_id = aws_vpc.main.id
}

resource "aws_subnet" "public_a" {

  vpc_id = aws_vpc.main.id

  cidr_block = "10.0.1.0/24"

  availability_zone = "${var.region}a"

  map_public_ip_on_launch = true
}

resource "aws_subnet" "public_b" {

  vpc_id = aws_vpc.main.id

  cidr_block = "10.0.2.0/24"

  availability_zone = "${var.region}b"

  map_public_ip_on_launch = true
}

resource "aws_route_table" "public" {

  vpc_id = aws_vpc.main.id
}

resource "aws_route" "internet" {

  route_table_id         = aws_route_table.public.id

  destination_cidr_block = "0.0.0.0/0"

  gateway_id = aws_internet_gateway.igw.id
}

resource "aws_route_table_association" "a" {

  subnet_id = aws_subnet.public_a.id

  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "b" {

  subnet_id = aws_subnet.public_b.id

  route_table_id = aws_route_table.public.id
}
```

---

# security.tf

### ALB Security Group

```hcl
resource "aws_security_group" "alb_sg" {

  name   = "alb-sg"

  vpc_id = aws_vpc.main.id

  ingress {

    from_port = 80
    to_port   = 80

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {

    from_port = 443
    to_port   = 443

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {

    from_port = 0
    to_port   = 0

    protocol = "-1"

    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

### EC2 Security Group

```hcl
resource "aws_security_group" "ec2_sg" {

  name   = "ec2-sg"

  vpc_id = aws_vpc.main.id

  ingress {

    from_port = 80
    to_port   = 80

    protocol = "tcp"

    security_groups = [
      aws_security_group.alb_sg.id
    ]
  }

  egress {

    from_port = 0
    to_port   = 0

    protocol = "-1"

    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Notice:

✅ No SSH exposed

---

# userdata.sh

```bash
#!/bin/bash

apt-get update -y

apt-get install apache2 -y

echo "<h1>Terraform ACM Demo</h1>" > /var/www/html/index.html

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

    name = "name"

    values = [
      "ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"
    ]
  }
}

resource "aws_instance" "web" {

  ami = data.aws_ami.ubuntu.id

  instance_type = var.instance_type

  subnet_id = aws_subnet.public_a.id

  vpc_security_group_ids = [
    aws_security_group.ec2_sg.id
  ]

  key_name = var.key_name

  user_data = file("${path.module}/userdata.sh")

  tags = {
    Name = "apache-server"
  }
}
```

---

# alb.tf

```hcl
resource "aws_lb" "alb" {

  name = "demo-alb"

  internal = false

  load_balancer_type = "application"

  security_groups = [
    aws_security_group.alb_sg.id
  ]

  subnets = [
    aws_subnet.public_a.id,
    aws_subnet.public_b.id
  ]
}
```

### Target Group

```hcl
resource "aws_lb_target_group" "tg" {

  name = "demo-tg"

  port = 80

  protocol = "HTTP"

  vpc_id = aws_vpc.main.id

  health_check {
    path = "/"
  }
}
```

### Register EC2

```hcl
resource "aws_lb_target_group_attachment" "ec2" {

  target_group_arn =
    aws_lb_target_group.tg.arn

  target_id =
    aws_instance.web.id

  port = 80
}
```

---

# acm.tf

```hcl
resource "aws_acm_certificate" "cert" {

  domain_name = var.domain_name

  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}
```

---

# route53.tf

Hosted Zone already exists.

```hcl
data "aws_route53_zone" "zone" {

  name = var.domain_name

  private_zone = false
}
```

Validation record:

```hcl
resource "aws_route53_record" "validation" {

  for_each = {

    for dvo in aws_acm_certificate.cert.domain_validation_options :

    dvo.domain_name => {

      name  = dvo.resource_record_name

      value = dvo.resource_record_value

      type  = dvo.resource_record_type
    }
  }

  zone_id = data.aws_route53_zone.zone.zone_id

  name = each.value.name

  type = each.value.type

  ttl = 60

  records = [
    each.value.value
  ]
}
```

Certificate validation:

```hcl
resource "aws_acm_certificate_validation" "cert" {

  certificate_arn =
    aws_acm_certificate.cert.arn

  validation_record_fqdns = [
    for r in aws_route53_record.validation :
    r.fqdn
  ]
}
```

---

### HTTPS Listener

```hcl
resource "aws_lb_listener" "https" {

  load_balancer_arn =
    aws_lb.alb.arn

  port = 443

  protocol = "HTTPS"

  certificate_arn =
    aws_acm_certificate_validation.cert.certificate_arn

  default_action {

    type = "forward"

    target_group_arn =
      aws_lb_target_group.tg.arn
  }
}
```

---

### HTTP Redirect

```hcl
resource "aws_lb_listener" "http" {

  load_balancer_arn =
    aws_lb.alb.arn

  port = 80

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

### Alias Record

```hcl
resource "aws_route53_record" "root" {

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

output "application_url" {

  value = "https://${var.domain_name}"
}
```

---

# terraform.tfvars

```hcl
region      = "eu-central-1"
domain_name = "example.com"
key_name    = "my-keypair"
```

---

# Deploy

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

### Result

After apply:

```text
User
  |
  v
https://example.com
  |
  v
Route53
  |
  v
ALB (443)
  |
  v
Target Group
  |
  v
EC2 Apache
```

This is the cleanest Terraform implementation that matches the video exercise while removing a few bad practices (public SSH, invalid CIDR range, unnecessary rules) and keeping the code interview-ready.
