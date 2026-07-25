# LAB 10 — Developer Experience (DevEx)

**Technologies:** Dev Containers · LocalStack · GitHub Codespaces

> [!NOTE]
> This lab focuses on building a developer experience platform that enables engineering teams to deliver software faster and more consistently. You will create standardized local environments, automate onboarding workflows, introduce local AWS service emulation, and implement inner-loop CI practices that provide rapid developer feedback.

---

# Learning Objectives

After completing this lab, you will be able to:

- 💻 Build standardized development environments using Dev Containers.
- 🐳 Configure local development dependencies using Docker Compose.
- ☁️ Emulate AWS services locally using LocalStack.
- ⚡ Reduce developer onboarding time through automation.
- 🔍 Implement pre-commit quality checks.
- 🌐 Configure GitHub Codespaces for cloud-based development.
- 📚 Establish a Developer Portal for engineering visibility.
- 📈 Measure and optimize developer productivity metrics.

---

# Task 1 — Standardised Local Development with Dev Containers

## Task Overview

Create consistent, reproducible development environments where every developer receives the same tools, dependencies, and IDE configuration without manual setup.

---

## Implementation Steps

### Step 1 — Create Dev Container Configurations

Create a `.devcontainer/devcontainer.json` file for each service:

- Next.js
- Magento
- API

A single configuration file should define the complete development environment.

---

### Step 2 — Include Required Development Tools

Include all required tools inside the Dev Container image.

Required tools:

- Node.js
- PHP
- Composer
- AWS CLI
- Terraform

Goal:

- Zero manual environment setup.

---

### Step 3 — Configure Dependent Services

Add `docker-compose.yml` configuration for required development dependencies.

Include:

- MySQL
- Redis
- Elasticsearch
- MailHog

---

### Step 4 — Standardize VS Code Extensions

Configure VS Code extensions inside `devcontainer.json`.

Every developer should automatically receive the same IDE experience.

---

## Dev Container Example — `.devcontainer/devcontainer.json`

```json
{
  "name": "Magento Dev",
  "dockerComposeFile": [
    "docker-compose.yml",
    "docker-compose.devcontainer.yml"
  ],
  "service": "app",
  "workspaceFolder": "/var/www/html",
  "extensions": [
    "bmewburn.vscode-intelephense-client",
    "ms-azuretools.vscode-docker",
    "hashicorp.terraform"
  ],
  "postCreateCommand": "composer install && php bin/magento setup:install"
}
```

> [!TIP]
> Dev Containers eliminate environment drift by ensuring every developer works from the same preconfigured development environment.

---

# Task 2 — LocalStack for AWS Service Emulation

## Task Overview

Use LocalStack to emulate AWS services locally, allowing developers to test cloud integrations without consuming real AWS resources.

---

## Implementation Steps

### Step 1 — Add LocalStack to Docker Compose

Add LocalStack to `docker-compose.yml`.

Supported local AWS services:

- Amazon S3
- Amazon SQS
- Amazon SNS
- AWS Secrets Manager
- AWS Lambda

---

### Step 2 — Create LocalStack Initialization Scripts

Create startup initialization scripts that automatically create:

- Queues
- Buckets
- Secrets

when LocalStack starts.

---

### Step 3 — Configure Local AWS CLI Profile

Create an AWS CLI profile pointing to:

```text
http://localhost:4566
```

Use this profile for local testing.

---

### Step 4 — Configure Applications for Local AWS Services

Update application configuration to use LocalStack endpoints when:

```text
APP_ENV=development
```

Benefits:

- No real AWS costs during development.
- Faster local testing.
- Reduced dependency on shared cloud environments.

---

# Task 3 — Onboarding Automation

## Task Overview

Automate the developer onboarding process so new engineers can become productive quickly with minimal manual setup.

---

## Implementation Steps

### Step 1 — Create a Single Setup Command

Create a Makefile target:

```bash
make setup
```

The command should perform:

- Clone repositories
- Install dependencies
- Configure local environment
- Seed database
- Start services

---

### Step 2 — Define Onboarding Target

Target:

> A new engineer should have a running local environment in under 15 minutes.

---

### Step 3 — Implement Pre-Commit Hooks

Use the **pre-commit framework**.

Required checks:

- Linting
- Formatting
- Secret scanning

Run checks locally before code reaches CI.

---

### Step 4 — Configure GitHub Codespaces

Set up GitHub Codespaces as a cloud development fallback.

Use cases:

- Low-powered developer machines
- Restricted network environments
- Remote development scenarios

---

### Step 5 — Create a Developer Portal

Create a Developer Portal using:

- Backstage
- Similar internal developer platform tools

The portal should list:

- Services
- On-call owners
- Runbooks
- Deployment status

---

## Makefile Example

```makefile
.PHONY: setup

setup:
	@echo '--- Cloning repositories ---'
	git clone git@github.com:company/magento.git

	@echo '--- Installing dependencies ---'
	cd magento && composer install

	@echo '--- Starting services ---'
	docker compose up -d

	@echo '--- Seeding database ---'
	docker compose exec app php bin/magento setup:install --use-sample-data

	@echo '✅ Setup complete! Visit http://localhost:8080'
```

> [!SUCCESS]
> Developers can now onboard quickly using a repeatable workflow that provides consistent environments, automated setup, and immediate productivity.

---

# Best Practices — Developer Experience

> [!IMPORTANT]
> Apply these practices to continuously improve developer productivity and reduce engineering friction.

---

## ⏱️ Measure Onboarding Time

Measure how long it takes new engineers to become productive.

Guideline:

- If onboarding takes more than **1 day**, treat it as a Developer Experience issue requiring immediate attention.

---

## 🛠️ Maintain a Single Entry Point

Keep the Makefile as the primary developer interface.

Examples:

```bash
make setup
make test
make build
```

Avoid undocumented tribal knowledge.

---

## ⚡ Optimize Pre-Commit Hooks

Pre-commit hooks should complete in:

```text
< 30 seconds
```

Slow hooks are likely to be disabled or ignored by developers.

---

## 📦 Provide Realistic Development Data

Provide developers with:

- Realistic
- Anonymized

production data dumps for local testing.

---

## 💬 Run DevEx Office Hours

Hold monthly **DevEx office hours**.

Purpose:

- Collect developer feedback.
- Identify friction points.
- Improve engineering workflows before issues become cultural problems.

---

## 📊 Track Inner-Loop Metrics

Monitor developer workflow performance metrics:

- Local build time
- Test execution time
- Hot-reload time

Continuously optimize the inner development loop.
