# LAB 08 — Infrastructure Security & Compliance

**Technologies:** IAM · SAST · CIS Benchmarks

> [!NOTE]
> This lab focuses on implementing security best practices across the entire infrastructure stack. You will strengthen identity and access management, secure application secrets, automate vulnerability scanning, and implement compliance controls aligned with AWS and industry security standards.

---

# Learning Objectives

After completing this lab, you will be able to:

- 🔐 Implement least-privilege IAM policies.
- 👤 Harden AWS identity and access management.
- 🔑 Secure application secrets using AWS Secrets Manager.
- ☸️ Synchronize cloud secrets with Kubernetes.
- 🛡️ Automate vulnerability scanning throughout the CI/CD pipeline.
- 📋 Measure and improve compliance using AWS Security Hub and CIS Benchmarks.
- 🚨 Configure continuous threat detection and security monitoring.

---

# Task 1 — IAM Hardening & Least Privilege

## Task Overview

Strengthen AWS identity and access management by removing unnecessary privileges, enforcing secure authentication, and enabling continuous auditing.

---

## Implementation Steps

### Step 1 — Audit IAM Users

Review all IAM users and perform the following actions:

- Delete unused IAM users.
- Enable Multi-Factor Authentication (MFA) for all human users.
- Remove AWS Management Console access from service accounts.

---

### Step 2 — Replace Long-Lived Credentials

Replace all long-lived IAM access keys with modern authentication mechanisms.

Use:

| Environment | Recommended Authentication |
|-------------|----------------------------|
| Amazon EKS | OIDC / IAM Roles for Service Accounts (IRSA) |
| Amazon EC2 | Instance Profiles |

---

### Step 3 — Implement IAM Permission Boundaries

Configure **IAM Permission Boundaries** for all developer-created IAM roles.

This prevents unintended privilege escalation while allowing delegated role creation.

---

### Step 4 — Enable AWS CloudTrail

Configure AWS CloudTrail with the following settings:

- Multi-Region logging
- Multi-Account logging
- Log File Validation

Store CloudTrail logs in a dedicated security-focused Amazon S3 bucket.

---

### Step 5 — Enable AWS Config

Enable AWS Config with the following managed rules:

- `restricted-ssh`
- `s3-bucket-public-read-prohibited`
- `iam-root-access-key-check`

> [!TIP]
> Combining AWS Config with CloudTrail provides continuous visibility into both configuration drift and API activity.

---

# Task 2 — Secrets Management

## Task Overview

Centralize and secure application credentials while automating secret rotation and synchronization across cloud-native workloads.

---

## Implementation Steps

### Step 1 — Migrate Secrets

Migrate **all** secrets from:

- Environment variables
- Configuration files

to **AWS Secrets Manager**.

---

### Step 2 — Enable Automatic Rotation

Enable automatic credential rotation for Amazon RDS.

Use:

- AWS Secrets Manager native rotation
- AWS Lambda rotation function

---

### Step 3 — Synchronize Kubernetes Secrets

Install the **External Secrets Operator** in Kubernetes.

Automatically synchronize AWS Secrets Manager values into Kubernetes Secrets.

---

### Step 4 — Prevent Secret Leakage

Configure secret scanning using one of the following tools:

- `git-secrets`
- `gitleaks`

Run these tools:

- As a Git pre-commit hook
- Within the CI pipeline

This prevents secrets from being committed to source control.

---

## External Secrets Operator Example

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret

metadata:
  name: magento-db-creds

spec:
  refreshInterval: 1h

  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore

  target:
    name: magento-db-secret

  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: prod/magento/db
        property: password
```

> [!SUCCESS]
> Application credentials are now securely managed, automatically rotated, and synchronized into Kubernetes without exposing secrets in source code.

---

# Task 3 — Vulnerability Scanning & Compliance

## Task Overview

Implement continuous security scanning, compliance monitoring, and threat detection across AWS infrastructure and container workloads.

---

## Implementation Steps

### Step 1 — Enable Amazon Inspector

Enable **Amazon Inspector** for continuous vulnerability scanning of:

- Amazon EC2 instances
- Container images

---

### Step 2 — Integrate Trivy into CI

Run **Trivy** during every CI pipeline.

Security policy:

| Severity | Action |
|----------|--------|
| CRITICAL | Fail the build |
| HIGH | Create remediation ticket |
| MEDIUM | Create remediation ticket for next sprint |

---

### Step 3 — Configure AWS Security Hub

Enable **AWS Security Hub**.

Activate the:

- CIS AWS Foundations Benchmark

Target:

- **90% or higher compliance score**

---

### Step 4 — Protect Applications with AWS WAF

Deploy AWS WAF on all Application Load Balancers (ALBs).

Enable the following managed rule groups:

- Core Rule Set
- SQL Injection Protection
- Known Bad Inputs

---

### Step 5 — Enable Amazon GuardDuty

Enable **Amazon GuardDuty** across all AWS accounts.

Configure:

- Amazon SNS notifications
- Automatic notification of **HIGH** severity findings to the security Slack channel

> [!SUCCESS]
> Your AWS environment now includes continuous vulnerability assessment, centralized compliance monitoring, web application protection, and automated threat detection.

---

# Best Practices — Infrastructure Security

> [!IMPORTANT]
> Apply the following security practices to reduce organizational risk and maintain a strong security posture.

## 🚨 Treat Security Findings Like Production Bugs

Every security finding should include:

- Severity
- Assigned Owner
- Service Level Agreement (SLA) for remediation

Security issues should be tracked with the same discipline as production defects.

---

## 🔒 Secure Administrative Access

Never allow unrestricted SSH access:

```text
0.0.0.0/0 → TCP 22
```

Instead, use:

- AWS Systems Manager Session Manager

This eliminates the need to expose SSH to the public internet.

---

## 🔐 Encrypt Everything

Enable encryption for all supported AWS services.

At a minimum:

- Amazon EBS Volumes
- Amazon RDS
- Amazon S3 (SSE-S3 minimum)
- Amazon S3 (SSE-KMS for sensitive data)
- Amazon SQS
- Amazon SNS

---

## 🔄 Rotate Credentials Regularly

Rotate all credentials on a defined schedule, even when there is no evidence of compromise.

Design security controls using an **assume breach** mindset.

---

## 🌐 Use VPC Endpoints

Implement VPC Endpoints for:

- Amazon S3
- Amazon DynamoDB

This keeps service traffic on the AWS private backbone rather than traversing the public internet.

---

## 📖 Document Security Controls

Maintain a centralized security runbook documenting:

- Security controls
- Operational procedures
- Compliance evidence

Well-maintained documentation supports governance activities and can assist with cyber insurance and audit requirements.
