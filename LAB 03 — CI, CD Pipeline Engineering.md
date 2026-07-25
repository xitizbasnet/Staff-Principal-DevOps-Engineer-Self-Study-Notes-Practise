# LAB 03 — CI, CD Pipeline Engineering

**Technologies:** GitHub Actions · GitLab CI · Jenkins

> [!NOTE]
> This lab focuses on designing and implementing production-grade Continuous Integration and Continuous Deployment (CI/CD) pipelines for modern applications. You will build automated deployment workflows for a **Next.js frontend**, **Capacitor mobile application**, and **Magento PHP backend**, following DevOps best practices for quality, security, and zero-downtime deployments.

---

# Learning Objectives

After completing this lab, you will be able to:

- ⚙️ Design automated CI/CD pipelines using GitHub Actions.
- ✅ Implement automated code quality checks and testing.
- 📦 Build and publish application artifacts.
- ☁️ Deploy static and containerized applications to AWS.
- 📱 Automate Android and iOS mobile application builds.
- 🚀 Implement Blue-Green deployment strategies.
- 🔄 Configure automated deployment rollback mechanisms.
- 📊 Measure and improve deployment performance using DevOps metrics.

---

# Task 1 — Next.js Frontend CI/CD Pipeline

## Task Overview

Build a complete CI/CD workflow that validates, builds, and deploys a **Next.js** application using **GitHub Actions** and AWS services.

---

## Implementation Steps

### Step 1 — Configure Workflow Triggers

Set up a GitHub Actions workflow triggered on:

- Push to the `main` branch
- All `pull_request` events

---

### Step 2 — Stage 1: Lint and Test

Run the following validation steps:

- ESLint
- TypeScript checks
- Jest unit tests

Fail the pipeline immediately if any validation fails.

---

### Step 3 — Stage 2: Build

Run:

```bash
next build
```

Upload the generated `.next` directory as an artifact for later deployment.

---

### Step 4 — Stage 3: Deploy

Deploy the application by:

- Syncing built assets to Amazon S3
- Triggering an Amazon CloudFront cache invalidation for:

```text
/*
```

---

### Step 5 — Configure Environment Promotion

Implement deployment promotion rules:

| Environment | Deployment Strategy |
|-------------|---------------------|
| Development | Automatic deployment after every merge |
| Production | Manual approval required |

---

## GitHub Actions Example — `.github/workflows/frontend.yml`

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm ci
      - run: npm run lint && npm test -- --coverage

  build:
    needs: test

    steps:
      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: nextjs-build
          path: .next/

  deploy-prod:
    needs: build
    environment: production

    steps:
      - run: aws s3 sync .next/static s3://$BUCKET/_next/static --delete
      - run: aws cloudfront create-invalidation --distribution-id $CF_ID --paths '/*'
```

---

# Task 2 — Capacitor Mobile Build Pipeline

## Task Overview

Create dedicated CI/CD workflows for building, signing, and publishing Android and iOS applications using Capacitor.

---

## Implementation Steps

### Step 1 — Configure Workflow Triggers

Create a separate workflow triggered on Git tags matching:

```text
v*.*.*
```

---

### Step 2 — Configure Build Runners

Use platform-specific GitHub runners:

| Platform | Runner |
|----------|--------|
| iOS | `macos-latest` |
| Android | `ubuntu-latest` |

---

### Step 3 — Cache Dependencies

Cache the following dependencies between workflow runs:

- Gradle
- CocoaPods

This should reduce build times by approximately **60%**.

---

### Step 4 — Sign Application Packages

Configure secure application signing.

**Android**

- Sign APK/AAB using a keystore stored in GitHub Secrets.

**iOS**

- Sign IPA using a provisioning profile.

---

### Step 5 — Publish Release Artifacts

Automatically upload build artifacts to the corresponding GitHub Release associated with the Git tag.

---

# Task 3 — Magento PHP Backend Pipeline

## Task Overview

Implement a production-ready CI/CD pipeline for a Magento PHP application, including quality checks, container image creation, Blue-Green deployment, and automated rollback.

---

## Implementation Steps

### Step 1 — Static Code Analysis

Run the following tools on every Pull Request:

- PHP_CodeSniffer (PSR-12)
- PHPStan (Level 6 or higher)

---

### Step 2 — Integration Testing

Execute PHPUnit integration tests against a MySQL service container within the CI pipeline.

---

### Step 3 — Build and Publish Docker Images

Build a Docker image using:

```text
php:8.3-fpm
```

Push the image to Amazon ECR using both tags:

- SHA
- latest

---

### Step 4 — Deploy Using Blue-Green Strategy

Deploy the application to Amazon ECS.

Shift production traffic using:

- Application Load Balancer (ALB)
- Listener Rule updates

---

### Step 5 — Configure Automatic Rollback

Monitor the ALB **HTTP 5xx error rate**.

Automatically trigger a rollback when:

- Error rate exceeds **1%**
- Duration exceeds **5 minutes**

---

## Automatic Rollback Check (Bash)

```bash
ERROR_RATE=$(aws cloudwatch get-metric-statistics \
--metric-name HTTPCode_Target_5XX_Count \
--start-time $(date -u -d '5 minutes ago' +%Y-%m-%dT%H:%M:%SZ) \
--end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
--period 300 --statistics Sum | jq '.Datapoints[0].Sum')

if [ "$ERROR_RATE" -gt 50 ]; then
    echo 'High error rate — triggering rollback'

    aws ecs update-service \
        --cluster prod \
        --service magento \
        --task-definition $PREVIOUS_TASK_DEF
fi
```

> [!SUCCESS]
> Your deployment pipeline can automatically recover from unhealthy deployments, reducing downtime and improving service reliability.

---

# Best Practices — CI/CD Pipelines

> [!IMPORTANT]
> Adopt the following practices to build secure, efficient, and maintainable CI/CD pipelines.

## 🚀 Pipeline Optimization

- Fail fast by placing linting and static analysis in the first pipeline stage.
- These checks are inexpensive and catch most issues early.

---

## ⚡ Dependency Caching

Cache dependencies whenever possible:

- `node_modules`
- Python packages (`pip`)
- Gradle
- Maven

This can reduce pipeline execution time by **50–70%**.

---

## 🔐 Secure Cloud Authentication

Use **OIDC federation** for cloud authentication.

Avoid storing long-lived cloud access keys in GitHub Secrets.

---

## 📁 Pipeline as Code

Store pipeline definitions (YAML files) alongside application code.

Benefits include:

- Version control
- Pull Request reviews
- Auditability
- Easier collaboration

---

## 🛡️ Deployment Protection

Never deploy directly from feature branches to production.

Enforce:

- Branch protection rules
- Required Pull Request reviews
- Manual production approvals

---

## 📊 Measure Pipeline Performance

Track DORA metrics to continuously improve delivery performance:

- Deployment Frequency
- Lead Time for Changes
- Mean Time to Recovery (MTTR)

These metrics provide visibility into pipeline health and engineering efficiency.
