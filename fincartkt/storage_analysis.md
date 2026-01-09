# Fincart – Storage Cost Analysis & Optimization Observations

> **Document Type:** FinOps / Cost Analysis (Ad-hoc)  
> **Scope:** Azure Subscription – Storage Services Only  
> **Included Services:** Azure Blob Storage, Managed Disks, Geo-Replication, Bandwidth  
> **Source of Data:** Azure Cost Management – CSV Export (last ~30 days)

---

## 1. Purpose of This Document

This document is created as a **standalone, ad-hoc FinOps analysis** focused exclusively on Azure Storage services.

The objectives are to:
- Provide **data-backed visibility** into Azure Storage costs
- Support **cost optimization discussions** with clear risk context
- Keep financial analysis **separate from operational and architecture documentation**

This document does **not recommend direct changes**. It highlights **observations and safe review areas only**, intended for review and discussion.

---

## 2. Methodology

- Azure Portal → Cost Management → Cost analysis  
- Service filter applied: **Storage**  
- Time range: ~Last 30 days  
- Cost data exported as **CSV** and analyzed by:
  - Storage tier / meter
  - Region
  - Cost contribution percentage

---

## 3. High-Level Storage Cost Overview

### Cost Distribution (Approximate)

- **Tiered Block Blob (Data Stored):** ~73%  
- **Bandwidth / Data Transfer:** ~22%  
- **Managed Disks (Premium + Standard):** ~4%  
- **Blob Features & Change Feed:** <2%

### Cost Summary

| Metric | Value |
|------|------|
| Total Monthly Storage Cost | ~₹70,000 |
| Budget | ₹3.5L / month |
| Trend | ↓ ~8% |

**Key Insight:**
- Storage spend is primarily driven by **data volume, retention, and access patterns**, not by infrastructure misconfiguration.

**Conclusion:** Storage spend is **within budget and trending down**, behaving as expected for a finance-grade platform.

---

## 4. Primary Cost Drivers

### 4.1 Tiered Block Blob (~73%)

**What it represents:**
- Blob Storage data (Hot / LRS / GRS)
- Documents, reports, backups, exports, application files

**Observation:**
- Majority of storage cost is driven by **data volume and retention**
- Expected behavior for platforms handling financial and compliance-sensitive data

---

### 4.2 Bandwidth (~22%)

**What it represents:**
- Outbound data transfer
- Reads, downloads, report exports, blob access by applications and users

**Observation:**
- Indicates frequent blob consumption
- Usage-driven cost, not configuration inefficiency

---

### 4.3 Managed Disks (~4%)

**What it represents:**
- Premium SSD, Standard SSD, and HDD managed disks

**Observation:**
- Low overall cost impact
- Not a priority optimization area

---

### 4.4 Blob Features & Change Feed (<2%)

**What it represents:**
- Advanced blob capabilities
- Change tracking and audit-related features

**Observation:**
- Minimal cost footprint
- Operationally and compliance justified

---

## 5. Geo-Replication & Regional Insight

**Observed Regions:**
- Central India  
- US Central  

**Observation:**
- Geo-replication and cross-region data transfer contribute a noticeable portion of storage cost
- Confirms **DR, resiliency, and compliance-driven design**

**Risk Context:**
- Any optimization in this area is **business-risk sensitive**

---

## 6. Risk Classification

| Area | Risk Level | Rationale |
|----|-----------|-----------|
| Tiered Block Blob (Data Stored) | Medium | Driven by retention and data growth |
| Bandwidth | Medium | Application and user access dependent |
| Managed Disks | Low | Small cost footprint |
| Blob Features / Change Feed | Low | Expected and minimal |
| Geo-Replication | High | DR and compliance dependency |

---

## 7. Safe Review Areas (Observation Only)

> These are **discussion points**, not change recommendations.

- Non-production storage retention (Stage / UAT)
- Lifecycle policy review for cold or archival data
- Visibility into bandwidth-heavy consumers (reports, exports, integrations)

---

## 8. Executive Summary (Shareable)

> *Azure Storage cost (~₹70K/month) is primarily driven by Tiered Block Blob data storage (~73%) and bandwidth (~22%). Geo-replication contributes due to DR and compliance requirements. Overall spend is usage-driven rather than configuration waste, with limited low-to-medium risk review opportunities in non-production retention and lifecycle governance.*

---
