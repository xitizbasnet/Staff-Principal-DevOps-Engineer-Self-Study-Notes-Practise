# LAB 07 — Observability: Metrics, Logs, and Traces

**Technologies:** Datadog · Prometheus · Grafana

> [!NOTE]
> This lab focuses on implementing a comprehensive observability strategy using **Datadog**, **Prometheus**, and **Grafana**. You will define Service Level Objectives (SLOs), configure monitoring and alerting, build operational dashboards, and establish actionable incident response workflows.

---

# Learning Objectives

After completing this lab, you will be able to:

- 📊 Define Service Level Indicators (SLIs) and Service Level Objectives (SLOs).
- 📈 Deploy and configure a Prometheus and Grafana monitoring stack.
- 🖥️ Instrument Magento and infrastructure services for metrics collection.
- 🚨 Implement an alerting strategy using Alertmanager.
- 📚 Create operational dashboards for application and infrastructure health.
- 🔍 Monitor SLO error budgets and burn rates.
- 📖 Develop operational runbooks for critical production alerts.

---

# Task 1 — Define SLOs and SLIs

## Task Overview

Before instrumenting any application or infrastructure component, define what "good" looks like for each production service.

The following table defines the Service Level Indicators (SLIs), Service Level Objectives (SLOs), and available error budgets.

| Service | SLI Metric | SLO Target | Error Budget (30 Days) |
|----------|------------|-----------:|-----------------------:|
| Magento Checkout | Success rate of `/checkout` POST | **99.9%** | **43.2 minutes** |
| Magento Catalog | p99 latency < 800 ms | **99.5%** | **3.6 hours** |
| API Gateway | HTTP 5xx rate < 0.1% | **99.9%** | **43.2 minutes** |
| Background Jobs | Job completion within 10 minutes | **99.0%** | **7.2 hours** |

> [!TIP]
> SLOs provide measurable reliability targets that help engineering teams balance feature delivery with operational stability.

---

# Task 2 — Prometheus + Grafana Stack Setup

## Task Overview

Deploy a complete observability platform that collects infrastructure and application metrics, visualizes system health, and tracks service reliability.

---

## Implementation Steps

### Step 1 — Deploy the Monitoring Stack

Deploy **kube-prometheus-stack** using Helm.

The deployment includes:

- Prometheus
- Grafana
- Alertmanager
- node-exporter

---

### Step 2 — Configure Metrics Collection

Configure Prometheus scrape configurations for all services using **ServiceMonitor** Custom Resource Definitions (CRDs).

---

### Step 3 — Instrument Magento

Install the Magento PHP exporter or use the **Datadog APM Agent** to expose:

- PHP-FPM metrics
- OPcache metrics

---

### Step 4 — Build Operational Dashboards

Create a Grafana dashboard that includes:

- 📈 Request Rate
- ❌ Error Rate
- ⏱️ p50 Latency
- ⏱️ p95 Latency
- ⏱️ p99 Latency
- ⚙️ Saturation (USE Method)

---

### Step 5 — Import the SLO Dashboard

Import an SLO dashboard that displays:

- Current Error Budget Burn Rate
- 1-hour Burn Rate Alert
- 6-hour Burn Rate Alert

---

## PromQL Example — SLO Error Budget Burn Rate (1-Hour Window)

### Checkout Success Rate

```promql
sum(rate(http_requests_total{
  job="magento",
  handler="/checkout",
  status=~"2.."
}[1h]))
/
sum(rate(http_requests_total{
  job="magento",
  handler="/checkout"
}[1h]))
```

---

### p99 Request Latency

```promql
histogram_quantile(
  0.99,
  sum(
    rate(http_request_duration_seconds_bucket{
      job="magento"
    }[5m])
  ) by (le)
)
```

> [!SUCCESS]
> Your monitoring platform now provides visibility into service health, request latency, error rates, and SLO compliance.

---

# Task 3 — Alerting Strategy

## Task Overview

Implement an alerting strategy that routes incidents to the appropriate teams while minimizing alert fatigue through intelligent routing and inhibition.

---

## Implementation Steps

### Step 1 — Configure Alert Routing

Configure Alertmanager to route alerts based on severity.

| Priority | Destination |
|----------|-------------|
| 🔴 P1 | PagerDuty (On-Call Engineer) |
| 🟠 P2 | Slack `#alerts-prod` |
| 🟡 P3 | Jira Ticket |

---

### Step 2 — Create Alert Rules

Implement alert rules for the following scenarios.

| Alert | Severity |
|--------|----------|
| Error Budget Burn Rate > 2× | P1 |
| Pod CrashLoopBackOff > 3 times in 15 minutes | P1 |
| Disk Utilization > 85% | P2 |

---

### Step 3 — Configure Alert Inhibition

Implement alert inhibition rules.

Suppress downstream alerts whenever the upstream service is already generating an alert to prevent alert storms.

---

### Step 4 — Create Operational Runbooks

Create runbooks for every:

- P1 alert
- P2 alert

Include the corresponding runbook URL within each alert annotation.

> [!TIP]
> Engineers should be able to navigate directly from an alert to the associated operational procedure.

---

# Best Practices — Observability

> [!IMPORTANT]
> Adopt the following observability best practices to improve operational visibility, reduce incident response time, and increase service reliability.

## 🎯 Alert on User Impact

Alert on symptoms rather than infrastructure metrics.

Example:

✅ **Checkout error rate is high**

Instead of:

❌ **CPU utilization is high**

---

## 📊 Monitor the Four Golden Signals

Start with the four Golden Signals defined in the Google SRE Book:

- Latency
- Traffic
- Errors
- Saturation

---

## 📖 Every Alert Requires a Runbook

Every production alert should reference a documented runbook.

An alert without a runbook trains operational teams to ignore alerts over time.

---

## 🚨 Configure Burn Rate Alerts

Configure SLO burn rate alerts using:

| Window | Burn Rate |
|--------|----------:|
| 1 Hour | 2× |
| 5 Minutes | 5× |

Using both fast and slow burn detection provides early warning while reducing unnecessary noise.

---

## 📝 Use Structured Logging

Adopt structured JSON logging across all applications and services.

Benefits include:

- Easier searching
- Better filtering
- Improved analytics
- Faster troubleshooting

Free-text logs become difficult to query at scale.

---

## 🔍 Implement Distributed Tracing

Enable distributed tracing using one of the following platforms:

- Datadog APM
- Jaeger

Distributed tracing helps correlate requests, logs, and metrics across multiple microservices, reducing mean time to resolution (MTTR).
