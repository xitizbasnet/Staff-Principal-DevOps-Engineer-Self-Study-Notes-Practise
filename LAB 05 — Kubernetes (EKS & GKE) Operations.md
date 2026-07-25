# LAB 05 — Kubernetes (EKS & GKE) Operations

**Technologies:** Kubernetes · Amazon EKS · Google GKE · Helm · ArgoCD

> [!NOTE]
> This lab focuses on provisioning a production-ready Kubernetes platform using **Amazon EKS**, deploying applications with **Helm**, implementing **GitOps** using **ArgoCD**, and configuring intelligent autoscaling to ensure scalability, resilience, and operational excellence.

---

# Learning Objectives

After completing this lab, you will be able to:

- ☸️ Provision a production-ready Amazon EKS cluster.
- ⚙️ Configure managed node groups for system and application workloads.
- 📦 Install and manage Kubernetes applications using Helm.
- 🔄 Implement GitOps workflows with ArgoCD.
- 🔐 Configure IAM Roles for Service Accounts (IRSA).
- 📈 Configure Horizontal Pod Autoscaling (HPA).
- 🚀 Scale workloads dynamically using KEDA.
- 🖥️ Automatically scale cluster infrastructure with Cluster Autoscaler or Karpenter.

---

# Task 1 — EKS Cluster Provisioning with eksctl / Terraform

## Task Overview

Provision a highly available Amazon EKS cluster using Terraform, configure managed node groups, install required Kubernetes add-ons, and integrate AWS-native networking and identity management.

---

## Implementation Steps

### Step 1 — Provision an Amazon EKS Cluster

Provision an Amazon EKS cluster using the **terraform-aws-modules/eks** module.

Requirements:

- Kubernetes version **1.29+**

---

### Step 2 — Create Managed Node Groups

Create the following managed node groups.

| Node Group | Instance Type | Capacity Type | Purpose |
|------------|---------------|---------------|---------|
| **system** | `t3.medium` | On-Demand | Core Kubernetes system pods |
| **app** | `m5.large` | Mixed Spot + On-Demand | Application workloads |

---

### Step 3 — Enable Amazon EKS Managed Add-ons

Enable the following managed add-ons:

- CoreDNS
- kube-proxy
- Amazon VPC CNI
- Amazon EBS CSI Driver

---

### Step 4 — Install AWS Load Balancer Controller

Install the AWS Load Balancer Controller using Helm.

This controller manages:

- Application Load Balancer (ALB) Ingress resources
- AWS-native load balancing for Kubernetes workloads

---

### Step 5 — Configure IAM Roles for Service Accounts (IRSA)

Configure **IAM Roles for Service Accounts (IRSA)**.

Best practice:

- Assign IAM roles directly to Kubernetes pods.
- Never rely on node-level instance profiles.

---

## Install AWS Load Balancer Controller via Helm

```bash
helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=prod-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

> [!TIP]
> IRSA provides fine-grained AWS permissions for individual workloads while eliminating the need to expose node-level credentials.

---

# Task 2 — GitOps with ArgoCD

## Task Overview

Implement a GitOps deployment strategy using ArgoCD to continuously synchronize Kubernetes resources from a dedicated Git repository.

---

## Implementation Steps

### Step 1 — Install ArgoCD

Install ArgoCD into the `argocd` namespace.

Expose the ArgoCD server through:

- Application Load Balancer (ALB)
- TLS-enabled Ingress

---

### Step 2 — Create a GitOps Repository

Create a dedicated Git repository for Kubernetes manifests.

Follow the **GitOps repository pattern** by separating infrastructure manifests from application source code.

---

### Step 3 — Define ArgoCD Applications

Create ArgoCD **Application** Custom Resource Definitions (CRDs) that reference Helm charts stored in the GitOps repository.

---

### Step 4 — Enable Automated Synchronization

Configure ArgoCD to:

- Automatically synchronize changes.
- Enable **Self-Heal** to revert manual `kubectl` modifications.

---

### Step 5 — Configure ArgoCD Image Updater

Install and configure **ArgoCD Image Updater**.

Automatically update image tags within the GitOps repository whenever a new image is pushed to Amazon ECR.

---

## ArgoCD Application Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: magento
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/company/k8s-gitops
    targetRevision: HEAD
    path: apps/magento

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

> [!SUCCESS]
> GitOps ensures that the desired cluster state is continuously reconciled with the contents of the Git repository, reducing configuration drift and enabling repeatable deployments.

---

# Task 3 — Autoscaling (HPA + KEDA + Cluster Autoscaler)

## Task Overview

Configure application and infrastructure autoscaling to dynamically respond to workload demand while maintaining high availability.

---

## Implementation Steps

### Step 1 — Configure Horizontal Pod Autoscaler (HPA)

Install **Metrics Server**.

Configure a Horizontal Pod Autoscaler (HPA) for the Magento deployment.

Configuration:

| Metric | Target |
|--------|--------|
| CPU Utilization | 60% |

---

### Step 2 — Configure Event-Driven Scaling with KEDA

Install **KEDA**.

Create a **ScaledObject** that automatically scales background worker pods based on:

- Amazon SQS queue depth

---

### Step 3 — Configure Cluster Autoscaling

Install one of the following:

- Cluster Autoscaler
- Karpenter

Automatically add or remove EC2 worker nodes based on pending Kubernetes pods.

---

### Step 4 — Configure Pod Disruption Budgets (PDB)

Create **PodDisruptionBudgets (PDBs)** for all production deployments.

Ensure:

- At least one pod remains available during node maintenance or draining operations.

---

# Best Practices — Kubernetes

> [!IMPORTANT]
> Apply the following Kubernetes best practices to improve cluster security, reliability, and operational efficiency.

## ⚙️ Resource Management

Always define both:

- Resource Requests
- Resource Limits

Without them, individual pods can consume all available node resources.

---

## 🔒 Workload Isolation

Use Kubernetes namespaces together with **NetworkPolicies**.

Recommended approach:

- Default-deny all inter-namespace traffic.
- Explicitly allow only required communication.

---

## 🛡️ Pod Security

Never run application containers as the root user.

Apply **Pod Security Standards** using the **Restricted** profile across the cluster.

---

## 📦 Helm Configuration

Maintain:

- A single reusable Helm chart.
- Separate `values.yaml` files for each environment (Development, Staging, Production).

---

## 📋 Audit Logging

Enable Amazon EKS control plane audit logging.

Ship audit logs to **Amazon CloudWatch** for centralized monitoring and security analysis.

---

## 🔍 Troubleshooting

A fast first-pass diagnostic command:

```bash
kubectl get events --sort-by=timestamp
```

Use this command regularly to identify scheduling issues, deployment failures, and cluster events in chronological order.
