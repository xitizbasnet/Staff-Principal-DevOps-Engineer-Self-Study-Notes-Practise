# LAB 01 — Cloud Infrastructure Architecture

**Technologies:** AWS · Terraform · Infrastructure as Code (IaC)

> [!NOTE]
> This lab guides you through building a production-grade AWS networking and infrastructure foundation using **Terraform**. The architecture follows AWS best practices, including network segmentation, high availability, secure access control, and scalable application deployment.

---

# Learning Objectives

After completing this lab, you will be able to:

- 🌐 Design a production-ready Amazon VPC.
- 🛡️ Implement secure network segmentation using public and private subnets.
- 🔐 Configure Security Groups following the principle of least privilege.
- ⚙️ Deploy scalable compute resources using Auto Scaling Groups.
- 🗄️ Provision a highly available PostgreSQL database.
- ☁️ Configure object storage with Amazon S3.
- 🚀 Deliver applications globally using Amazon CloudFront.

---

# Task 1 — Build the Network Foundation (VPC / Subnets / Security Groups)

## Task Overview

A well-designed network is the backbone of any production environment. You will create a VPC with public and private subnets across two Availability Zones, configure an Internet Gateway, NAT Gateway, and strict Security Groups.

---

## Implementation Steps

### Step 1 — Create the VPC

Create a VPC with CIDR **10.0.0.0/16** in your chosen region (**ap-south-1** for Mumbai).

---

### Step 2 — Create Public Subnets

Create **2 Public Subnets**:

- Public Subnet A — `10.0.1.0/24`
- Public Subnet B — `10.0.2.0/24`

Deploy them in:

- Availability Zone **ap-south-1a**
- Availability Zone **ap-south-1b**

---

### Step 3 — Create Private Subnets

Create **2 Private Subnets** for the application and database tiers.

- App Tier — `10.0.10.0/24`
- Database Tier — `10.0.11.0/24`

---

### Step 4 — Configure Internet Connectivity

Attach an **Internet Gateway (IGW)** and update the public route table.

| Destination | Target |
|------------|--------|
| `0.0.0.0/0` | Internet Gateway (IGW) |

---

### Step 5 — Configure Private Internet Access

Create a **NAT Gateway** inside a public subnet.

Route all outbound traffic from the private subnets through the NAT Gateway.

---

### Step 6 — Configure Security Groups

Define the following Security Groups:

| Security Group | Allowed Traffic |
|----------------|-----------------|
| **ALB** | TCP 80 / TCP 443 open |
| **Application** | TCP 8080 from **ALB Security Group** |
| **Database** | TCP 5432 from **Application Security Group only** |

---

## Terraform Example — VPC

```terraform
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name        = "prod-vpc"
    Environment = "production"
  }
}

resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true
}
```

> [!SUCCESS]
> You have a production-grade VPC with isolated tiers and no direct Internet exposure to private resources.

---

# Task 2 — Provision Compute, Database & Storage

## Task Overview

Deploy a scalable, highly available application platform using AWS managed services.

---

## Implementation Steps

### Step 1 — Deploy an Application Load Balancer

Launch an **Application Load Balancer (ALB)** across both public subnets.

Configure:

- HTTPS Listener
- ACM Certificate

---

### Step 2 — Create an EC2 Launch Template

Deploy an EC2 Launch Template using:

- Latest Amazon Linux 2023 AMI
- Instance type: **t3.medium**
- User-data script to install application dependencies

---

### Step 3 — Configure Auto Scaling

Create an Auto Scaling Group with the following configuration.

| Setting | Value |
|---------|------|
| Minimum | 2 |
| Desired | 2 |
| Maximum | 6 |

Attach the Auto Scaling Group to the ALB Target Group.

---

### Step 4 — Deploy Amazon RDS

Provision an **Amazon RDS PostgreSQL** instance with:

- Instance Class: **db.t3.medium**
- Multi-AZ Deployment
- Private DB Subnet Group

---

### Step 5 — Configure Amazon S3

Create an S3 bucket for static assets.

Enable:

- Versioning
- Block all public access

---

### Step 6 — Configure CloudFront

Attach an **Amazon CloudFront Distribution** in front of:

- Amazon S3
- Application Load Balancer

This provides:

- Content caching
- Global content delivery

---

## Terraform Example — Auto Scaling Group

```terraform
resource "aws_autoscaling_group" "app" {
  desired_capacity = 2
  max_size         = 6
  min_size         = 2

  target_group_arns = [
    aws_lb_target_group.app.arn
  ]

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }

  health_check_type = "ELB"
}
```

> [!SUCCESS]
> Scalable compute behind a load balancer, with managed database and CDN-accelerated static content.

---

# Best Practices — Cloud Infrastructure

> [!IMPORTANT]
> Follow these production best practices throughout all deployments.

- 🏷️ **Resource Tagging**
  - Always tag every resource with:
    - `Environment`
    - `Owner`
    - `CostCenter`

- 🛡️ **High Availability**
  - Use **Multi-AZ** for all stateful services (RDS, ElastiCache).
  - Single-AZ deployments are never acceptable in production.

- 🔐 **Instance Metadata Security**
  - Enforce **IMDSv2** on all EC2 instances to prevent SSRF-based metadata attacks.

- 📦 **Terraform State Management**
  - Store Terraform state remotely using:
    - Amazon S3
    - DynamoDB state locking
  - Never commit Terraform state files to Git repositories.

- 📊 **Network Visibility**
  - Enable **VPC Flow Logs**.
  - Ship logs to Amazon S3 or Amazon CloudWatch for network forensics.

- 👤 **IAM Security**
  - Implement least-privilege IAM policies.
  - Use Instance Profiles.
  - Never hardcode credentials.
