# LAB 02 — Terraform IaC Deep Dive

**Technologies:** Terraform · CloudFormation

> [!NOTE]
> This lab focuses on advanced **Infrastructure as Code (IaC)** practices using **Terraform**. You will learn how to build reusable modules, manage remote state securely, organize multiple environments, and implement production-ready CI/CD workflows for Terraform deployments.

---

# Learning Objectives

After completing this lab, you will be able to:

- 📦 Build reusable Terraform modules.
- 🌍 Organize infrastructure by environment.
- 🗄️ Configure secure remote state management.
- 🔒 Implement state locking with DynamoDB.
- 🌿 Manage environments using Terraform Workspaces.
- 🚀 Automate Terraform validation and deployment using GitHub Actions.
- 🛡️ Integrate security scanning into Infrastructure as Code pipelines.

---

# Task 1 — Modularise Your Terraform Code

## Task Overview

Structure your Terraform repository into reusable modules so teams can compose infrastructure like Lego blocks.

---

## Implementation Steps

### Step 1 — Create the Module Structure

Create the following directory structure:

```text
modules/
├── vpc/
├── ec2/
├── rds/
└── alb/
```

---

### Step 2 — Define Module Interfaces

Each module must expose:

- `variables.tf`
- `outputs.tf`

This ensures modules are reusable, configurable, and self-contained.

---

### Step 3 — Create Environment Roots

Create dedicated environment directories:

```text
environments/
├── dev/
├── staging/
└── prod/
```

---

### Step 4 — Reference Modules

Call modules from each environment root using environment-specific variable files (`.tfvars`).

## Example — `environments/prod/main.tf`

```terraform
module "vpc" {
  source      = "../../modules/vpc"
  cidr_block  = var.vpc_cidr
  environment = "production"
}

module "rds" {
  source         = "../../modules/rds"
  vpc_id         = module.vpc.vpc_id
  subnet_ids     = module.vpc.private_subnet_ids
  instance_class = "db.r6g.large"
  multi_az       = true
}
```

---

# Task 2 — Remote State & Workspace Management

## Task Overview

Configure a secure and scalable remote backend for Terraform state while enabling collaborative infrastructure development.

---

## Implementation Steps

### Step 1 — Create an S3 Backend

Create an Amazon S3 bucket named:

```text
company-terraform-state
```

Enable:

- Versioning
- Server-side encryption

---

### Step 2 — Configure State Locking

Create a DynamoDB table:

```text
terraform-locks
```

Partition key:

```text
LockID
```

---

### Step 3 — Configure the Backend

Configure the S3 backend within each environment root using unique state key paths.

## Example

```terraform
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/networking/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

---

### Step 4 — Manage Terraform Workspaces

Use Terraform Workspaces to isolate feature branch environments.

Example:

```bash
terraform workspace new feature-x
```

---

# Task 3 — CI/CD Enforcement of Terraform Plans

## Task Overview

Implement automated validation, planning, security scanning, and controlled deployment of Terraform infrastructure using GitHub Actions.

---

## Implementation Steps

### Step 1 — Validate Every Pull Request

Create a GitHub Actions workflow that executes:

- `terraform fmt -check`
- `terraform validate`

on every Pull Request.

---

### Step 2 — Generate Terraform Plans

Run `terraform plan` and publish the plan output as a Pull Request comment using the **hashicorp/setup-terraform** action.

---

### Step 3 — Protect Production Deployments

Gate `terraform apply` behind a manual approval step using:

- GitHub Environments
- Required Reviewers

---

### Step 4 — Perform Security Scanning

Integrate one of the following tools into the CI pipeline:

- tfsec
- Checkov

Perform static security analysis on all Terraform code before deployment.

---

## GitHub Actions Example — `.github/workflows/terraform.yml`

```yaml
- name: Terraform Plan
  run: |
    terraform init
    terraform plan -out=tfplan -var-file=prod.tfvars

- name: Security Scan
  run: checkov -d . --framework terraform

- name: Apply (prod only, manual gate)
  if: github.ref == 'refs/heads/main'
  environment: production
  run: terraform apply tfplan
```

---

# Best Practices — Terraform Infrastructure as Code (IaC)

> [!IMPORTANT]
> Follow these production best practices to ensure Terraform deployments remain secure, maintainable, and scalable.

### 📌 Version Management

- Pin provider and module versions.
- Never use `latest` or `>= any` in production.

---

### 📚 Module Documentation

- Use **terraform-docs** to automatically generate module documentation.
- Keep documentation synchronized with the source code.

---

### ✅ Plan Before Apply

- Always run `terraform plan` before **every** `terraform apply`.
- Never skip the planning stage, including automated deployments.

---

### 🔍 Use Data Sources

Avoid hardcoding resource identifiers.

Example:

```terraform
data.aws_ami.latest
```

Using data sources keeps configurations portable and easier to maintain.

---

### 🚨 Drift Detection

Schedule automated `terraform plan` executions in CI every night.

Alert when:

- Infrastructure drift is detected.
- Plans contain non-empty differences.

---

### 🗂️ State Management

Separate Terraform state files:

- Per environment
- Per logical infrastructure layer

Examples:

- Networking
- Compute
- Data

This improves isolation, collaboration, and recovery.
