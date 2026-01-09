# Fincart – Azure SQL Database Cost Analysis & Observations

> **Document Type:** FinOps / Cost Analysis (Ad-hoc)  
> **Scope:** Azure Subscription – Azure SQL Databases Only  
> **Included Services:** Azure SQL Database (Hyperscale), Backups, LTR Storage  
> **Source of Data:** Azure Cost Management – CSV Export (last ~30 days)

---

## 1. Purpose of This Document

This document is created as a **standalone, ad-hoc FinOps analysis** focused exclusively on Azure SQL Database services.

The objectives are to:
- Provide **data-backed visibility** into Azure SQL Database costs
- Identify **primary cost drivers** across compute, storage, and backups
- Support **cost optimization discussions** with clear risk context
- Keep financial analysis **separate from operational and architecture documentation**

This document does **not recommend direct changes**. It highlights **observations and safe review areas only**, intended for review and discussion.

---

## 2. Methodology

- Azure Portal → Cost Management → Cost analysis  
- Service filter applied: **SQL Database**  
- Time range: ~Last 30 days  
- Cost data exported as **CSV** and analyzed by:
  - Compute vs storage vs backup
  - Database tier and deployment model
  - Cost contribution percentage

---

## 3. High-Level SQL Cost Overview

### Cost Distribution (Approximate)

- **Hyperscale Compute (Provisioned vCore):** ~83%  
- **Hyperscale Serverless Compute:** ~8%  
- **Hyperscale Storage:** ~5%  
- **Backups & Long-Term Retention (LTR):** <4%

### Cost Summary

| Metric | Value |
|------|------|
| Total Monthly SQL Cost | ~₹52,000 |
| Budget | ₹3.5L / month |
| Trend | Stable |

**Key Insight:**
- Azure SQL cost is **heavily compute-driven**, reflecting a performance-first architecture rather than storage or backup overhead.

**Conclusion:** SQL spend is controlled and expected for a production-grade financial platform running on Hyperscale.

---

## 4. Primary Cost Drivers

### 4.1 Hyperscale Compute – Provisioned (~83%)

**What it represents:**
- Provisioned vCore compute for primary SQL workloads
- Always-on performance capacity

**Observation:**
- Dominant cost component
- Indicates sustained workload demand and performance requirements
- Expected for transaction-heavy systems

---

### 4.2 Hyperscale Serverless Compute (~8%)

**What it represents:**
- Auto-scaling compute for variable or intermittent workloads

**Observation:**
- Smaller share of cost
- Useful for non-continuous or burst-driven usage patterns

---

### 4.3 Hyperscale Storage (~5%)

**What it represents:**
- Data files stored in Hyperscale architecture

**Observation:**
- Storage cost is relatively low compared to compute
- Indicates efficient separation of compute and storage layers

---

### 4.4 Backups & Long-Term Retention (<4%)

**What it represents:**
- Automated backups
- RA-GRS / LRS backup storage
- Long-term retention policies

**Observation:**
- Minimal cost footprint
- Operationally and compliance justified

---

## 5. Environment-Wise Cost Insight

### Production Environment

- Hosts primary transactional databases (e.g., FINPROD)
- Majority of compute cost originates here
- Performance and availability are business-critical

### Stage / UAT Environments

- Lower overall cost compared to production
- Still contribute to baseline compute spend

**Observation:**
- Non-production SQL costs are reasonable but visible
- Any changes must consider testing and release reliability

---

## 6. Risk Classification

| Area | Risk Level | Rationale |
|----|-----------|-----------|
| Production Compute | High | Direct impact on transactions and availability |
| Non-Production Compute | Medium | Potential tuning with validation |
| SQL Storage | Low | Small cost footprint |
| Backups & LTR | Low | Compliance and recovery dependent |

---

## 7. Safe Review Areas (Observation Only)

> These are **discussion points**, not change recommendations.

- Compute sizing alignment for non-production databases
- Review of serverless vs provisioned usage patterns
- Validation of backup retention duration against compliance needs

---

## 8. Executive Summary (Shareable)

> *Azure SQL Database cost (~₹52K/month) is predominantly driven by Hyperscale compute (~83%), reflecting a performance-first design for transactional workloads. Storage and backup costs are comparatively low and operationally justified. Overall spend is controlled, with limited medium-risk review opportunities in non-production compute sizing.*

---
