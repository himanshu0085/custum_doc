# Fincart – Monitoring (Log Analytics & Application Insights) Cost Analysis & Observations

> **Document Type:** FinOps / Cost Analysis (Ad-hoc)
> **Scope:** Azure Subscription – Monitoring Services Only
> **Included Services:** Log Analytics, Application Insights, Alerts, Diagnostics
> **Source of Data:** Azure Cost Management – CSV Export (last ~30 days)

---

## 1. Purpose of This Document

This document is created as a **standalone, ad-hoc FinOps analysis** focused exclusively on Azure monitoring services.
Its purpose is to provide **cost visibility and usage context** without tying the analysis to operational ownership or architectural documentation.

The objectives are to:

* Provide **clear visibility** into monitoring-related costs
* Identify **environment-wise cost contributors** (Prod vs Non-Prod)
* Highlight **safe review areas** with appropriate risk context

This document does **not recommend direct changes**. It presents observations intended for discussion and informed decision-making only.

---

## 2. Methodology

* Azure Portal → Cost Management → Cost analysis
* Service filters applied:

  * **Log Analytics**
  * **Application Insights**
* Time range: ~Last 30 days
* Data exported as **CSV** and analyzed across:

  * Service and meter type (ingestion, retention, alerts)
  * Resource (Application Insights instances / workspaces)
  * Environment classification (Production / Stage / UAT)

---

## 3. High-Level Monitoring Cost Overview

Based on the Azure Cost Management CSV analysis, monitoring-related costs show the following distribution:

### Cost Distribution (Approximate)

* **Application Insights – Log Ingestion:** ~65–70%
* **Log Analytics Workspace (Retention & Ingestion):** ~20–25%
* **Alerts, Metrics & Diagnostics:** ~5–10%

**Key Insight:**

* The majority of monitoring cost is driven by **telemetry ingestion**, not alerting or metrics.
* This is a common pattern in microservices-heavy platforms with verbose logging.

**Conclusion:** Monitoring spend is **usage-driven and controlled**, with cost primarily proportional to log volume rather than configuration inefficiency.

---

## 4. Application Insights – Cost Characteristics

### 4.1 Telemetry Ingestion

**What it represents:**

* Application telemetry including requests, dependencies, traces, and exceptions

**Observation:**

* Continuous telemetry ingestion from:

  * Backend APIs
  * Web applications
  * Background and batch jobs
* Logging behavior in non-production environments is comparable to production

---

### 4.2 Application-Level Isolation

**Observed Pattern:**

* Dedicated Application Insights instance per application
* Separate instances for Prod, Stage, and UAT

**Observation:**

* Improves isolation, debugging, and incident analysis
* Results in cumulative ingestion costs across environments

---

## 5. Log Analytics Workspace Costs

**What it represents:**

* Centralized storage of diagnostic and telemetry logs
* Retention-based billing model

**Observation:**

* Cost correlates with:

  * Volume of ingested logs
  * Retention period configuration
* No abnormal spikes or unexpected usage patterns observed

---

## 6. Alerts & Health Monitoring

**Observed Configuration:**

* Health check alerts configured for multiple applications
* CPU and memory alerts present across Prod and Non-Prod

**Observation:**

* Alerting setup appears comprehensive and proactive
* Cost impact of alerts is relatively low compared to ingestion

---

## 7. Environment-Wise Cost Insight

### Production Environment

* High-value telemetry supporting:

  * Incident response
  * Performance monitoring
  * SLA and reliability tracking

### Stage / UAT Environments

* Substantial telemetry volume observed
* Similar logging patterns to production workloads

**Observation:**

* Non-production monitoring depth may be reviewed if cost optimization becomes a priority

---

## 8. Risk Classification

| Area                      | Risk Level | Rationale                                      |
| ------------------------- | ---------- | ---------------------------------------------- |
| Production Monitoring     | High       | Critical for reliability and incident response |
| Non-Production Monitoring | Medium     | Potential tuning opportunity with validation   |
| Log Retention             | Medium     | Balance between cost and troubleshooting needs |
| Alerts & Health Checks    | Low        | Low cost, high operational value               |

---

## 9. Safe Review Areas (Observation Only)

> These points are intended for **review and discussion**, not immediate action.

* Logging verbosity alignment in Stage and UAT environments
* Retention period suitability for non-production logs
* Identification of telemetry-heavy applications

---

## 10. Executive Summary (Shareable)

> *Monitoring costs for Log Analytics and Application Insights are usage-driven and distributed across multiple applications and environments. Production monitoring is critical and high-risk to alter, while non-production environments present low-to-medium risk review opportunities related to log volume and retention.*

---

