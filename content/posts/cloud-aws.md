---
title: "AWS Cloud Interview Questions"
date: 2026-08-01
lastmod: 2026-08-01
weight: 4
draft: false
tags: ["AWS", "Cloud", "Interview"]
categories: ["Cloud"]
---

Quick reference for AWS/cloud interview questions organized by company and topic. Click on any question to expand details.

<!--more-->

---

<details>
<summary><strong>Aurora vs RDS</strong> - <code>Database</code></summary>

### Problem Statement

What's the difference between Amazon Aurora and Amazon RDS?

### Key Difference

**RDS** is AWS's managed relational database *service* — it runs several engines (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, **and** Aurora). **Aurora** is AWS's own cloud-native database *engine* (MySQL/PostgreSQL-compatible) that runs *inside* RDS, built for higher performance and availability.

| | RDS (MySQL/Postgres/etc.) | Aurora |
|---|---|---|
| Performance | Standard engine performance | Up to 5x MySQL, 3x PostgreSQL throughput |
| Storage | Fixed, manually provisioned | Auto-scales in 10GB increments up to 128TB |
| Replication | Async, replica lag can be seconds | 6 copies across 3 AZs, <10ms replica lag |
| Read Replicas | Up to 5 | Up to 15 |
| Failover | ~60-120s | ~30s or less |
| Cost | Cheaper | ~20% more than RDS for same instance class |

### When to Use Which

- **RDS**: need a specific engine (Oracle/SQL Server) or lower cost, standard workloads
- **Aurora**: need higher throughput, faster failover, or serverless auto-scaling

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Do You Need RDS to Run MySQL?</strong> - <code>Database</code></summary>

### Problem Statement

Can you run MySQL without RDS? What's the trade-off?

### Answer

Yes — you can self-host MySQL on a plain EC2 instance. RDS is a *convenience layer*, not a requirement.

| | Self-Managed MySQL (EC2) | RDS/Aurora MySQL |
|---|---|---|
| Setup | You install/configure everything | Provisioned in minutes |
| Patching | Manual | Automated (managed window) |
| Backups | You script/schedule them | Automated snapshots + point-in-time recovery |
| High Availability | You build Multi-AZ/replication yourself | Built-in Multi-AZ failover |
| Monitoring | You wire up CloudWatch/exporters | Built-in Performance Insights |
| Cost | Cheaper (just EC2 + EBS) | Higher (managed service premium) |
| Control | Full OS/DB-level access | Limited — no OS access, some config restricted |

### When Self-Managed Makes Sense

Very tight budget, need OS-level access/custom extensions, or a very specific tuning requirement RDS doesn't expose. For most production workloads, RDS/Aurora's operational simplicity outweighs the cost premium.

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>What is Amazon Aurora?</strong> - <code>Database</code></summary>

### Problem Statement

What is Aurora and what makes it different from a regular managed MySQL/PostgreSQL database?

### Answer

Aurora is AWS's cloud-native relational database engine, wire-compatible with MySQL and PostgreSQL, designed for high performance and availability.

### Key Features

- **Distributed, self-healing storage**: auto-scales up to 128TB in 10GB increments, data replicated 6 ways across 3 Availability Zones
- **Fast failover**: typically under 30 seconds
- **Up to 15 read replicas** with single-digit millisecond replica lag
- **Aurora Serverless v2**: compute auto-scales up/down per-second based on load — good for unpredictable/spiky workloads
- **Backtrack**: rewind the database to an earlier point in time without restoring from backup
- **Global Database**: replicate across AWS regions with <1s lag for disaster recovery/low-latency global reads

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Why EventBridge Over SNS/SQS/Kafka?</strong> - <code>Messaging</code></summary>

### Problem Statement

Why choose EventBridge for an integration instead of SNS, SQS, or Kafka?

### Comparison

| Service | Pattern | Best For |
|---|---|---|
| **EventBridge** | Event bus with content-based routing | Loosely-coupled event-driven architecture, native integration with 200+ AWS services & SaaS partners, schema registry |
| **SNS** | Pub/Sub fan-out | Simple broadcast to multiple subscribers, no advanced routing |
| **SQS** | Point-to-point queue | Reliable decoupling between two services, built-in retry + DLQ |
| **Kafka/Kinesis** | High-throughput event streaming | Real-time analytics, replayable event log, very high volume |

### Why EventBridge Specifically

- **Rules engine**: route events to different targets based on event content, without writing routing code
- **Schema registry**: auto-discovers and versions event schemas
- **SaaS integrations**: native connectors to third-party SaaS (Datadog, Zendesk, etc.) — SNS/SQS don't have this
- **Decoupled by default**: producers don't need to know about consumers at all (unlike SQS's tighter producer→queue coupling)

Use SQS/Kafka instead when you need guaranteed ordering, replay, or very high throughput — EventBridge is optimized for routing, not raw throughput.

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>Scaling a Database Up and Down</strong> - <code>Database</code></summary>

### Problem Statement

How do you scale a database up (for more load) and down (to save cost) — and can you actually scale down?

### Vertical Scaling (Up/Down) — Instance Class

Change the DB instance type (e.g., `db.r5.large` → `db.r5.xlarge` and back). Yes, scaling down is fully supported — same mechanism as scaling up, just picking a smaller class.

- **RDS/Aurora Provisioned**: instance class change causes a brief interruption (or none, with Multi-AZ — failover to the resized standby)
- **Aurora Serverless v2**: scales compute up/down automatically, per-second, with no manual step and no downtime — scales close to zero when idle

### Horizontal Scaling (Reads) — Read Replicas

Add/remove read replicas to handle read traffic; doesn't help write throughput since writes still go to a single primary.

### Scaling Writes (the hard part)

A single primary is the ceiling for write throughput. Options: sharding, or Aurora Limitless Database (auto-sharded writes).

### Monitoring to Decide

Watch CloudWatch metrics — CPU utilization, DB connections, read/write IOPS — to trigger scale up/down decisions (manually or via Aurora Serverless auto-scaling).

**Companies:** <code>Nike</code>

</details>

---

<details>
<summary><strong>What is AWS SQS?</strong> - <code>Messaging</code></summary>

### Problem Statement

What is Amazon SQS, and how have you used it in a project?

### Definition

SQS (Simple Queue Service) is a fully managed message queue used to decouple and scale microservices — producers push messages, consumers poll and process them asynchronously.

### Queue Types

| Type | Ordering | Delivery | Throughput |
|---|---|---|---|
| **Standard** | Best-effort (not guaranteed) | At-least-once (possible duplicates) | Nearly unlimited |
| **FIFO** | Strict order | Exactly-once | Up to 3,000 msg/sec (with batching) |

### Example Usage

Order Service publishes an `OrderCreated` message to an SQS queue → Inventory/Notification services poll the queue and process independently. If processing fails, the message becomes visible again after the **visibility timeout**; after N failed attempts it's routed to a **Dead Letter Queue (DLQ)** for investigation instead of being retried forever.

```java
// Producer
sqsClient.sendMessage(SendMessageRequest.builder()
    .queueUrl(queueUrl)
    .messageBody(orderJson)
    .build());

// Consumer
List<Message> messages = sqsClient.receiveMessage(
    ReceiveMessageRequest.builder().queueUrl(queueUrl).maxNumberOfMessages(10).build()
).messages();
```

**Companies:** <code>Nike</code>

</details>
