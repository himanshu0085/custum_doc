# Fincart – Azure SQL Database Cost Analysis & Observations

> **Document Type:** FinOps / Cost Analysis
> **Scope:** Azure Subscription – **SQL Database Service Only**
> **Source of Data:** Azure Cost Management – CSV Export (≈637 records, last ~30 days)

---

## 1. Purpose of This Document

This document provides a **standalone, data-backed analysis** of Azure SQL Database costs for the Fincart subscription.

The objective is to:

* Explain **where SQL cost is coming from**
* Classify **risk and sensitivity** of optimization
* Enable **informed decision-making** by tech and business stakeholders

---

## 2. Methodology

* Azure Portal → Cost Management → Cost analysis
* Filter: **Service name = SQL Database**
* Time range: ~Last 30 days
* Exported data as **CSV** and analyzed by:

  * Service Tier
  * Meter (Compute / Storage / Backup)
  * Resource (Database)
  * Environment (Prod / Stage / UAT)

---

## 3. High-Level SQL Cost Summary

| Metric                   | Value                 |
| ------------------------ | --------------------- |
| Total SQL Cost (30 days) | ~₹52,600              |
| Budget                   | ₹3.5L / month         |
| Cost Trend               | Stable (↓ ~8% recent) |

**Conclusion:** SQL spend is controlled and within budget.

---

## 4. Primary Cost Drivers (Service Tier Level)

### 4.1 Hyperscale – Compute Gen5 (≈83%)

**What it represents:**

* Provisioned vCore-based compute
* Always-on performance tier

**Observation:**

* Dominant cost component
* Indicates performance-first design for finance-grade workloads

---

### 4.2 Hyperscale – Serverless Compute (≈8%)

**What it represents:**

* Auto-scale compute for variable workloads
* Commonly used in non-prod or burst scenarios

**Observation:**

* Elastic usage pattern
* Lower cost footprint than provisioned compute

---

### 4.3 Hyperscale Storage (≈5%)

**What it represents:**

* Data stored in Hyperscale architecture

**Observation:**

* Storage cost is relatively low compared to compute
* Indicates cost is **performance-driven, not data-volume-driven**

---

### 4.4 Backup & Long-Term Retention (<4%)

**What it represents:**

* Automated backups (LRS / RA-GRS)
* LTR backup storage

**Observation:**

* Minimal cost
* Operationally mandatory for recovery and compliance

---

## 5. Key Insight

> **≈83% of Azure SQL cost is driven by compute, not storage or backups.**

This implies:

* SQL cost reflects **chosen performance tier**
* Optimization opportunities are **risk-sensitive** and require workload validation

---

## 6. Cost by Database (Resource-Level View)

Based on CSV analysis, SQL spend is distributed across:

* `FINPROD`
* `FINPRODPORTFOLIO`
* `FINSTAGE`
* `UATFIN`

### Observations

* Production databases use **Hyperscale provisioned compute**
* Non-production databases (Stage/UAT) also leverage high-performance tiers

> Any resizing or tier change in non-prod must be validated against testing and release requirements.

---

## 7. Risk Classification

| Area                     | Risk Level | Reason                                 |
| ------------------------ | ---------- | -------------------------------------- |
| PROD SQL (FINPROD)       | High       | Customer impact & SLA sensitivity      |
| Non-PROD SQL (Stage/UAT) | Medium     | Potential optimization with validation |
| Hyperscale Compute       | High       | Performance-critical                   |
| SQL Storage              | Low        | Small cost footprint                   |
| Backup & LTR             | Low        | Mandatory & minimal                    |

---

## 8. Safe Review Areas (Observation Only)

> These are **review considerations**, not recommendations.

* Validate performance needs of **Stage/UAT Hyperscale databases**
* Review serverless vs provisioned usage alignment
* Periodic cost trend monitoring for compute spikes

---

## 9. Executive Summary (Shareable)

> *Azure SQL Database cost (~₹52.6K/month) is primarily driven by Hyperscale compute (~83%), reflecting a performance-first architecture. Storage and backup costs are minimal. Any optimization, especially in non-production environments, requires workload validation due to performance and reliability considerations.*

---

