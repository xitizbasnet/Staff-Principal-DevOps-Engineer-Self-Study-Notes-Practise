# LAB 06 — Magento Platform Stability

**Technologies:** PHP · Nginx · Redis · Varnish

> [!NOTE]
> This lab focuses on operating and maintaining a production Magento e-commerce platform. You will implement zero-downtime deployments, optimize application performance, harden the hosting environment, and ensure high availability using industry best practices.

---

# Learning Objectives

After completing this lab, you will be able to:

- 🚀 Implement zero-downtime deployment strategies.
- ⚙️ Optimize Magento performance using PHP-FPM, Redis, and Varnish.
- 🔄 Configure release management with fast rollback capabilities.
- 🧪 Perform automated smoke testing before production releases.
- 📈 Improve application responsiveness through caching and tuning.
- 🔒 Apply production best practices for a stable Magento platform.

---

# Task 1 — Zero-Downtime Deployment Strategy

## Task Overview

Implement a deployment strategy that minimizes service interruption, validates application health before releasing traffic, and enables rapid rollback when necessary.

---

## Implementation Steps

### Step 1 — Configure Rolling Deployments

Adopt a rolling deployment strategy for Magento.

Deployment requirements:

- Deploy to one server at a time.
- Drain existing connections before stopping a server.

---

### Step 2 — Configure Maintenance Mode

Use Magento Maintenance Mode **only** when database schema changes are required.

Configure:

- Maintenance IP allowlist
- QA verification before opening production traffic

---

### Step 3 — Execute Magento Upgrade Commands

Run the following Magento commands **before** switching production traffic:

- `setup:upgrade`
- `setup:di:compile`

Do **not** execute these commands after traffic has been switched.

---

### Step 4 — Perform Smoke Testing

Implement a pre-deployment smoke test.

Verify that at least the following critical pages return an HTTP **200 OK** response:

- Home (`/`)
- Catalog (`/catalog`)
- Checkout (`/checkout`)

Abort the deployment if any endpoint returns a non-200 response.

---

### Step 5 — Enable Instant Rollback

Retain the previous release directory and use a symbolic link for the active deployment.

Example:

```bash
ln -sfn releases/v1.2.3 current
```

This enables immediate rollback to the previous release if required.

---

## Zero-Downtime Deployment Script (Bash)

```bash
RELEASE_DIR=/var/www/releases/$(date +%Y%m%d_%H%M%S)

rsync -az --delete build/ $RELEASE_DIR/

cd $RELEASE_DIR && php bin/magento setup:upgrade --keep-generated

php bin/magento setup:di:compile

php bin/magento setup:static-content:deploy -f

# Smoke test
for URL in / /catalog /checkout; do
  STATUS=$(curl -s -o /dev/null -w '%{http_code}' http://localhost$URL)

  [ "$STATUS" != "200" ] && echo "SMOKE FAIL $URL" && exit 1
done

ln -sfn $RELEASE_DIR /var/www/current

php bin/magento cache:flush
```

> [!SUCCESS]
> Your deployment process now supports rolling releases, automated validation, and instant rollback with minimal production downtime.

---

# Task 2 — Performance Tuning (PHP-FPM, Redis, Varnish)

## Task Overview

Optimize the Magento platform by tuning PHP-FPM, configuring Redis for caching and sessions, and implementing Varnish as the primary HTTP cache.

---

## Implementation Steps

### Step 1 — Tune PHP-FPM

Configure the PHP-FPM process manager with the following settings:

| Parameter | Value |
|-----------|-------|
| `pm` | `dynamic` |
| `pm.max_children` | `50` |
| `pm.start_servers` | `10` |

Enable PHP OPcache with Huge Pages for improved performance.

---

### Step 2 — Configure Redis

Enable Redis for both:

- Magento cache backend
- Magento session storage

Use **separate Redis instances** for each workload.

---

### Step 3 — Configure Varnish

Deploy **Varnish 7.x** in front of Nginx.

Target:

- Cache full-page HTML for anonymous users.
- Maintain a cache hit ratio greater than **90%**.

---

### Step 4 — Configure Nginx FastCGI Cache

Enable Nginx FastCGI caching as a fallback mechanism when Varnish is unavailable.

---

### Step 5 — Configure Magento Full Page Cache

Enable Magento Full Page Cache using **Varnish** as the backend.

After each deployment:

- Warm the cache automatically before exposing the release to users.

---

## PHP-FPM Pool Configuration

**File:** `/etc/php/8.3/fpm/pool.d/magento.conf`

```ini
pm = dynamic
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.max_requests = 500
```

---

## PHP OPcache Configuration

**File:** `php.ini`

```ini
opcache.enable = 1
opcache.memory_consumption = 512
opcache.max_accelerated_files = 100000
opcache.validate_timestamps = 0 ; disable in production
```

> [!TIP]
> Disabling `opcache.validate_timestamps` improves production performance because PHP no longer checks file modification times on every request. Ensure deployments include cache resets whenever application code changes.

---

# Best Practices — Magento Platform

> [!IMPORTANT]
> Follow these operational best practices to maximize Magento performance, availability, and maintainability.

## 🚀 Deployment Strategy

- Never run `setup:static-content:deploy` during peak business hours.
- Pre-generate static assets and deploy them as part of the release package.

---

## ⚡ Cache Performance

Monitor the Varnish cache hit ratio using:

```bash
varnishstat
```

If the cache hit ratio falls below **85%**, investigate cache invalidation rules and application behavior.

---

## 🔴 Redis High Availability

Deploy Redis using one of the following high availability solutions:

- Redis Sentinel
- Redis Cluster

Avoid using a single Redis instance, as it represents a **Single Point of Failure (SPOF)** in production.

---

## ⏰ Background Processing

Run Magento cron jobs separately from PHP-FPM.

Recommended approach:

- Dedicated cron worker instance

This prevents scheduled tasks from competing with web traffic.

---

## 📦 Index Management

Regularly verify index status:

```bash
bin/magento indexer:status
```

Stuck indexers can result in stale catalog and pricing data.

---

## 📊 Application Performance Monitoring

Enable application tracing using one of the following:

- Magento New Relic Integration
- Datadog APM for PHP

These tools help identify slow transactions, bottlenecks, and application performance issues.
