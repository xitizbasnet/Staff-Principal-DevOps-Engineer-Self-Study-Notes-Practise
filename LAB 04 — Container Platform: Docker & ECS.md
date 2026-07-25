# LAB 04 — Container Platform: Docker & ECS

**Technologies:** Docker · Amazon ECS · Amazon ECR

> [!NOTE]
> This lab focuses on containerizing applications using **Docker**, storing container images in **Amazon Elastic Container Registry (ECR)**, and orchestrating workloads with **Amazon Elastic Container Service (ECS)**. You will implement production-ready container images, secure image management, auto scaling, and Blue-Green deployments.

---

# Learning Objectives

After completing this lab, you will be able to:

- 🐳 Build production-ready Docker images using multi-stage builds.
- 🔒 Minimize container attack surfaces using lightweight runtime images.
- 👤 Run containers securely as non-root users.
- 🛡️ Integrate vulnerability scanning into CI pipelines.
- 📦 Store and manage container images in Amazon ECR.
- 🚀 Deploy containerized workloads to Amazon ECS.
- ⚖️ Configure service auto scaling based on application demand.
- 🔄 Perform Blue-Green deployments using AWS CodeDeploy.

---

# Task 1 — Production Docker Image (Multi-Stage Build)

## Task Overview

Create an optimized, secure Docker image suitable for production deployments.

---

## Implementation Steps

### Step 1 — Create a Multi-Stage Dockerfile

Write a multi-stage Dockerfile where:

- The **builder** stage compiles application assets.
- The **runtime** stage uses a minimal base image.

---

### Step 2 — Use a Minimal Runtime Image

Use one of the following images for the final stage:

- Distroless
- Alpine

This minimizes the container attack surface.

---

### Step 3 — Run as a Non-Root User

Configure the Dockerfile to run the application as a non-root user by adding a `USER` directive.

---

### Step 4 — Scan Container Images

Integrate **Trivy** into the CI pipeline.

Configure the pipeline to:

- Scan every image.
- Fail the build if any **CRITICAL** vulnerability is detected.

---

### Step 5 — Publish Images to Amazon ECR

Tag images using:

- Git SHA
- Semantic Version

Push images to Amazon ECR with **immutable tags enabled**.

---

## Dockerfile Example — Multi-Stage Magento/PHP

```dockerfile
FROM composer:2 AS vendor

WORKDIR /app

COPY composer.json composer.lock ./

RUN composer install --no-dev --optimize-autoloader

FROM php:8.3-fpm-alpine AS runtime

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /var/www/html

COPY --from=vendor /app/vendor ./vendor
COPY . .

RUN chown -R appuser:appgroup /var/www/html

USER appuser

EXPOSE 9000

CMD ["php-fpm"]
```

> [!TIP]
> Multi-stage builds significantly reduce image size by excluding build-time dependencies from the final runtime image.

---

# Task 2 — ECS Cluster & Service Configuration

## Task Overview

Deploy and manage containerized applications using Amazon ECS with production-grade configuration and deployment strategies.

---

## Implementation Steps

### Step 1 — Create an ECS Cluster

Deploy an Amazon ECS Cluster using:

| Workload | Launch Type |
|----------|-------------|
| Stateless Services | Fargate |
| Magento | EC2 |

Use **Fargate** for serverless container execution and **EC2** where additional operating system tuning is required.

---

### Step 2 — Create a Task Definition

Configure an ECS Task Definition with:

- CPU limits
- Memory limits
- CloudWatch Logs integration
- AWS Secrets Manager references for credentials

---

### Step 3 — Create an ECS Service

Deploy an ECS Service with the following configuration:

| Setting | Value |
|---------|------|
| Desired Count | 2 |
| Availability | Multi-AZ |
| Load Balancer | Application Load Balancer Target Group |

---

### Step 4 — Configure Service Auto Scaling

Enable ECS Service Auto Scaling.

Scale out when:

| Metric | Threshold |
|--------|-----------|
| ALB `RequestCountPerTarget` | 500 requests per target |

---

### Step 5 — Configure Blue-Green Deployment

Implement Blue-Green deployments using **AWS CodeDeploy**.

Deployment configuration:

- Traffic shift bake time: **10 minutes**

---

## ECS Task Definition Example (JSON)

```json
"containerDefinitions": [{
  "name": "magento",
  "image": "123456.dkr.ecr.ap-south-1.amazonaws.com/magento:v1.2.3",
  "cpu": 1024,
  "memory": 2048,
  "secrets": [{
    "name": "DB_PASSWORD",
    "valueFrom": "arn:aws:secretsmanager:...:db_password"
  }],
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/magento"
    }
  }
}]
```

> [!SUCCESS]
> Your application is now deployed as a highly available ECS service with secure secret management, centralized logging, and automated deployment capabilities.

---

# Best Practices — Docker & ECS

> [!IMPORTANT]
> Follow these production best practices when building, publishing, and deploying containerized applications.

## 🏷️ Image Versioning

- Always use specific image tags in production Task Definitions.
- Never deploy using the `latest` tag.

---

## ⚙️ Resource Management

Configure **hard CPU and Memory limits** for every container.

Benefits include:

- Preventing noisy neighbors
- Improving workload stability
- Better resource utilization

---

## 🔐 Secure Container Access

Use **ECS Exec** only for emergency ("break-glass") troubleshooting.

Avoid routine access to production containers.

---

## 🔑 Secrets Management

Store all application secrets in:

- AWS Secrets Manager
- AWS Systems Manager Parameter Store

Never store credentials as plain-text environment variables.

---

## 🛡️ Image Security

Enable **Amazon ECR Image Scanning** on every image push.

Configure an ECR lifecycle policy to:

- Automatically delete untagged images older than **14 days**.

---

## 🚀 Launch Type Selection

Choose the appropriate ECS launch type based on workload requirements.

| Launch Type | Recommended Workloads |
|-------------|-----------------------|
| Fargate | Stateless microservices |
| EC2 | Magento and workloads requiring operating system tuning |
