# Fincart – Storage Cost Analysis & Optimization Observations

> **Document Type:** FinOps / Cost Analysis
>
> **Scope:** Azure Subscription – Storage Service Only
> **Exclusions:** App Services (handled separately), CI/CD, Application Architecture
>
> **Source of Data:** Azure Cost Management CSV Export (last ~30 days)

---

## 1. Purpose of This Document

This document is created **separately from KT / Architecture documentation** to:

* Provide **data-backed visibility** into Azure Storage costs
* Support **cost optimization discussions** with clear risk context
* Avoid mixing **operational KT** with **FinOps analysis**

This document does **not recommend direct changes**. It highlights **observations and safe review areas only**.

---

## 2. Methodology

* Azure Portal → Cost Management → Cost analysis
* Service filter: **Storage**
* Time range: ~Last 30 days
* Data exported as **CSV** and analyzed by:

  * Tier / Meter
  * Region
  * Cost contribution

---

## 3. High-Level Storage Cost Summary

| Metric                     | Value         |
| -------------------------- | ------------- |
| Total Monthly Storage Cost | ~₹70,000      |
| Budget                     | ₹3.5L / month |
| Trend                      | ↓ ~8%         |

**Conclusion:** Storage spend is within budget and trending down.

---

## 4. Primary Cost Drivers

### 4.1 Tiered Block Blob (~73%)

**What it represents:**

* Data stored in Blob Storage (Hot / LRS / GRS)
* Documents, backups, exports, files

**Observation:**

* Majority of storage cost is driven by **data volume and retention**
* Expected behavior for a finance-grade platform

---

### 4.2 Bandwidth (~22%)

**What it represents:**

* Outbound data transfer
* Reads, downloads, report exports

**Observation:**

* Indicates frequent blob access by applications and users
* Usage-driven, not misconfiguration-driven

---

### 4.3 Managed Disks (~4%)

**What it represents:**

* Premium / Standard SSD & HDD

**Observation:**

* Low cost impact
* Not a priority optimization area

---

### 4.4 Blob Features & Change Feed (<2%)

**What it represents:**

* Advanced blob capabilities
* Change tracking / audit features

**Observation:**

* Minimal cost
* Operationally justified

---

## 5. Geo-Replication & Regional Insight

* Storage usage observed in:

  * Central India
  * US Central
* Geo-replication data transfer contributes a noticeable portion of cost

**Observation:**

* Confirms DR / compliance-driven cross-region replication
* Any optimization here is **business-risk sensitive**

---

## 6. Risk Classification

| Area                            | Risk Level | Reason                       |
| ------------------------------- | ---------- | ---------------------------- |
| Tiered Block Blob (Data Stored) | Medium     | Driven by retention & volume |
| Bandwidth                       | Medium     | Application usage dependent  |
| Managed Disks                   | Low        | Small cost footprint         |
| Blob Features / Change Feed     | Low        | Expected & minimal           |
| Geo-Replication                 | High       | DR & compliance dependency   |

---

## 7. Safe Review Areas (Observation Only)

> These are **discussion points**, not change recommendations.

* Non-production storage retention (Stage / UAT)
* Lifecycle policy review for cold or archival data
* Visibility into bandwidth-heavy consumers (reports / exports)

---

## 8. Executive Summary (Shareable)

> *Azure Storage cost (~₹70K/month) is primarily driven by Tiered Block Blob data storage and bandwidth. Geo-replication contributes due to DR/compliance requirements. Overall spend is usage-driven rather than configuration waste, with limited low-risk review opportunities in non-production retention and lifecycle governance.*

---
