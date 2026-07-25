# LAB 09 — Automation & Workflow Optimisation

**Technologies:** Bash · Python · Ansible

> [!NOTE]
> This lab focuses on building an automation-first engineering culture by eliminating repetitive operational work. You will create automated runbooks, self-service engineering tools, and ChatOps workflows that reduce manual effort and improve operational efficiency across the organization.

---

# Learning Objectives

After completing this lab, you will be able to:

- 🤖 Identify and automate repetitive operational tasks.
- ⚙️ Build reliable Ansible automation workflows.
- 🔐 Secure automation workflows using Ansible Vault.
- 💬 Create ChatOps integrations for self-service operations.
- ☁️ Trigger AWS workflows through automation tools.
- 📝 Build auditable automation processes.
- 🔄 Apply idempotent automation design principles.

---

# Task 1 — Runbook Automation with Ansible

## Task Overview

Identify recurring operational activities and convert them into reliable, repeatable automation workflows using Ansible.

---

## Implementation Steps

### Step 1 — Identify Manual Operational Tasks

Identify the top five manual operational tasks performed monthly by the team.

Examples:

- Server patching
- Certificate renewal
- Cache flushing
- Other repetitive operational activities

---

### Step 2 — Create Ansible Playbooks

Write an Ansible playbook for each identified task.

Requirements:

- Follow proper idempotency principles.
- Running the playbook multiple times must produce the same desired result.
- Avoid unnecessary changes when the system is already configured correctly.

---

### Step 3 — Secure and Version Control Playbooks

Store Ansible automation in a dedicated Git repository.

Requirements:

- Use Ansible Vault to encrypt sensitive variables.
- Maintain version history through Git.

---

### Step 4 — Schedule Automated Operations

Schedule recurring operational tasks using:

- AWS Systems Manager Automation
- Maintenance Windows

Example use case:

- Automated operating system patching

---

## Ansible Playbook Example — Redis Cache Flush

```yaml
---
- name: Flush Magento Redis Cache
  hosts: magento_app

  tasks:

    - name: Flush Redis cache backend
      community.general.redis:
        command: flush
        db: 0
      when: flush_cache | default(false) | bool

    - name: Run Magento cache clean
      command: /var/www/current/bin/magento cache:clean
      become: true
      become_user: www-data
```

> [!TIP]
> Idempotent automation ensures that operational scripts can be executed repeatedly without causing unexpected system changes.

---

# Task 2 — Self-Service Internal Tools

## Task Overview

Build internal automation tools that allow engineers to safely perform common operational activities without requiring manual platform team intervention.

---

## Implementation Steps

### Step 1 — Create a Slack Deployment Command

Build a Slack slash command:

```text
/deploy service=magento env=staging version=v1.2.3
```

The command should be backed by an AWS Lambda function.

---

### Step 2 — Validate Requests

The Lambda function must:

- Validate command inputs.
- Verify that the caller belongs to the `engineers` Slack group.
- Trigger a CodePipeline execution.

---

### Step 3 — Provide Deployment Notifications

Post deployment status updates back to Slack.

Deployment states:

- 🔄 In Progress
- ✅ Success
- ❌ Failure

---

### Step 4 — Build Database Snapshot Automation

Create a self-service database snapshot tool.

Workflow:

1. Engineer requests a snapshot through Slack.
2. Lambda creates the database snapshot.
3. Slack notification is sent when the snapshot is ready.

---

## Lambda Handler Example — Slack Deploy Command (Python)

```python
import boto3
import json

codepipeline = boto3.client('codepipeline')


def handler(event, context):

    params = parse_slack_payload(event['body'])

    if not is_authorised(params['user_id']):
        return {
            'statusCode': 403,
            'body': 'Not authorised'
        }

    codepipeline.start_pipeline_execution(
        name=f"{params['service']}-{params['env']}-pipeline",
        variables=[
            {
                'name': 'VERSION',
                'value': params['version']
            }
        ]
    )

    return slack_respond(
        f"Deploy started: {params['service']} {params['version']}"
    )
```

> [!SUCCESS]
> Engineers can now perform common operational tasks through controlled self-service workflows while maintaining security, auditability, and operational governance.

---

# Best Practices — Automation

> [!IMPORTANT]
> Follow these automation principles to ensure workflows remain safe, reliable, and maintainable.

---

## 🚀 Automate Before Incidents

Automate runbooks **before** incidents occur.

Create automation workflows when there is enough time to:

- Test properly.
- Validate expected behavior.
- Document the process.

---

## 📋 Maintain Auditability

Every automated action must record:

- Who triggered it.
- When it was triggered.
- What parameters were used.

---

## 🧪 Build Dry-Run Capabilities

Every automation script should include a dry-run mode.

Benefits:

- Safely test changes.
- Validate execution logic.
- Reduce production risk.

---

## ⏱️ Measure Operational Toil

Measure repetitive manual work.

Guideline:

> If a manual task takes more than 1 hour per week, it is worth approximately 1 week of automation investment.

---

## 📝 Document Before Automating

Never automate an undefined process.

Required sequence:

1. Document the process.
2. Validate the workflow.
3. Automate the process.

---

## 🔄 Design for Idempotency

Idempotency must be a core automation principle.

A well-designed automation script should:

- Be safe to execute multiple times.
- Produce consistent results.
- Avoid unnecessary changes.
