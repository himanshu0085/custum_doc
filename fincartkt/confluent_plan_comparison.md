# Confluent Cloud Kafka: Basic vs Standard vs Enterprise (Official Limits & Features)

## 1) Overview — What These Plans Are

Confluent Cloud provides fully managed Apache Kafka clusters with different capabilities depending on the selected tier.

| Cluster Type        | Target Use Case                                                   |
| ------------------- | ----------------------------------------------------------------- |
| Basic               | Experimentation, development, and small workloads                 |
| Standard            | Production-ready streaming workloads                              |
| Enterprise          | Mission-critical production with enhanced networking and security |
| Freight / Dedicated | Very high throughput and specialized enterprise needs             |

---

## 2) Uptime SLA & Service Guarantees

| Feature    | Basic            | Standard                | Enterprise |
| ---------- | ---------------- | ----------------------- | ---------- |
| Uptime SLA | No SLA guarantee | 99.9% SLA + autoscaling | 99.99% SLA |

Standard and Enterprise tiers provide service level guarantees, with Enterprise offering the highest availability and priority support.

---

## 3) Topic & Partition Limits

### Partitions

| Plan       | Included Partitions       | Billing Beyond Limit |
| ---------- | ------------------------- | -------------------- |
| Basic      | 10 partitions             | Charged beyond 10    |
| Standard   | 500 partitions            | Charged beyond 500   |
| Enterprise | No partition-based limits | No partition charges |

Partition count directly impacts throughput, consumer parallelism, and overall cost.

### Topics

* No strict hard limit on number of topics
* Very large topic counts may have UI display limits (around ~1000), while full access remains available through APIs

---

## 3.1) Included Partitions – 40% Capacity Cap

### Definition

* Basic Plan: First 10 partitions – capped at 40% cluster utilization
* Standard Plan: First 500 partitions – capped at 40% cluster utilization

The 40% cap refers to **40% of total cluster performance capacity**, not 40% of partition count.

### Factors Used in Calculation

* Ingress throughput (produce MB/sec)
* Egress throughput (consume MB/sec)
* Active client connections
* Request rate
* Disk I/O and replication load
* Consumer lag

### Behavior When Exceeded

* Requests may be throttled
* Latency increases
* Quota exceeded errors may appear
* New connections can be restricted

### Recommended Actions

* Upgrade tier (Basic → Standard → Enterprise)
* Increase CKUs
* Redistribute partitions
* Optimize batching and compression

### Purpose

* Protect cluster stability
* Prevent noisy-neighbor impact
* Maintain predictable performance

---

## 4) Message Size Limits

| Plan       | Maximum Message Size |
| ---------- | -------------------- |
| Basic      | 8 MB                 |
| Standard   | 8 MB                 |
| Enterprise | Up to 20 MB          |

---

## 5) Throughput (Ingress / Egress)

| Plan       | Ingress                       | Egress    |
| ---------- | ----------------------------- | --------- |
| Basic      | ~250 MB/s                     | ~750 MB/s |
| Standard   | ~250 MB/s                     | ~750 MB/s |
| Enterprise | Much higher depending on CKUs |           |

---

## 6) Storage & Retention

| Plan       | Storage               | Durability          |
| ---------- | --------------------- | ------------------- |
| Basic      | Up to ~5 TB           | Managed replication |
| Standard   | Infinite              | Managed replication |
| Enterprise | Infinite + governance | Managed replication |

---

## 7) Producers & Consumers

* Consumer parallelism equals number of partitions
* One partition → one active consumer thread
* More consumers than partitions → idle consumers

### Connection Limits

* Approximately 10,000 active connections on Standard
* Practical limits depend on CKUs and partition count

---

## 8) Networking & Security

| Feature            | Basic   | Standard | Enterprise |
| ------------------ | ------- | -------- | ---------- |
| Public endpoints   | Yes     | Yes      | Yes        |
| Private networking | No      | Yes      | Yes        |
| RBAC               | Partial | Yes      | Yes        |

---

# 9) Cluster-to-Cluster Data Transfer – Detailed

Confluent Cloud enables native replication using **Cluster Linking**, eliminating the need for MirrorMaker or external tools.

### 9.1 Replication Models

* Active → Passive (Disaster Recovery)
* Active → Active (Multi-Region)
* Migration Mode (On-prem to Cloud)

### 9.2 What Is Replicated

* Topic data and partitions
* Consumer group offsets
* Schema information
* Topic configurations

Replication occurs at Kafka protocol level preserving ordering and exactly-once semantics when used with transactional producers.

### 9.3 Billing Model

* Mirrored data volume
* Source egress
* Destination ingress
* CKU consumption on destination

### 9.4 Throughput Characteristics

**There is no fixed per-second transfer limit.**

Effective transfer rate depends on:

* Destination ingress capacity
* Source egress capacity
* Number of partitions
* CKUs allocated
* Network latency

#### Practical Ceiling

* Standard clusters: approx **up to ~250 MB/sec** (destination ingress bound)
* Enterprise/Dedicated: **multi-GB/sec possible**

### 9.5 Quota Impact

Replication traffic counts toward:

* Destination ingress quotas
* Source egress quotas
* 40% utilization cap (Basic/Standard)

If exceeded:

* Replication lag increases
* Throttling occurs
* Link enters degraded state

### 9.6 Operational Limitations

* Links are topic level
* Partition count must match
* Some config changes require recreation
* Cross-region latency impact

### 9.7 Best Practices

* Use dedicated service accounts
* Mirror only required topics
* Align partitions before linking
* Enable compression
* Monitor lag metrics
* Prefer Standard/Enterprise for DR

---

## 10) Partition Pricing

| Plan       | Billing              |
| ---------- | -------------------- |
| Basic      | Charged beyond 10    |
| Standard   | Charged beyond 500   |
| Enterprise | No partition charges |

---

## Quick Summary

| Feature            | Basic    | Standard   | Enterprise   |
| ------------------ | -------- | ---------- | ------------ |
| Use Case           | Dev/Test | Production | Enterprise   |
| SLA                | No       | Yes        | Yes (higher) |
| Partitions         | 10       | 500        | Unlimited    |
| 40% Cap            | Yes      | Yes        | No           |
| Message Size       | 8 MB     | 8 MB       | 20 MB        |
| Storage            | ~5 TB    | Infinite   | Infinite     |
| Private Networking | No       | Yes        | Yes          |

---

# Official Reference Links

* [https://docs.confluent.io/cloud/current/clusters/cluster-types.html](https://docs.confluent.io/cloud/current/clusters/cluster-types.html)
* [https://www.confluent.io/confluent-cloud/pricing/](https://www.confluent.io/confluent-cloud/pricing/)
* [https://docs.confluent.io/cloud/current/billing/overview.html](https://docs.confluent.io/cloud/current/billing/overview.html)
* [https://docs.confluent.io/cloud/current/topics/topics-faq.html](https://docs.confluent.io/cloud/current/topics/topics-faq.html)
* [https://www.confluent.io/learn/kafka-message-size-limit/](https://www.confluent.io/learn/kafka-message-size-limit/)
* [https://docs.confluent.io/cloud/current/client-apps/optimizing/throughput.html](https://docs.confluent.io/cloud/current/client-apps/optimizing/throughput.html)
