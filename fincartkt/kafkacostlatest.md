# Kafka Cost Optimization Summary

## Overview

This document summarizes the Kafka-related and infrastructure optimization activities performed between the following billing periods:

* 06-Mar-2026 to 05-Apr-2026
* 06-Apr-2026 to 05-May-2026

---

# Overall Cost Comparison

| Billing Period             | Total Cost |
| -------------------------- | ---------- |
| 06-Mar-2026 to 05-Apr-2026 | ₹763.2K    |
| 06-Apr-2026 to 05-May-2026 | ₹691.5K    |

## Total Cost Reduction

* ₹71,680 reduction
* Approx 9.39% decrease

---

# Kafka Environment Details

| Environment | Resource Name             | Provisioned On | Decommissioned / Stopped On | Status  |
| ----------- | ------------------------- | -------------- | --------------------------- | ------- |
| UAT Kafka   | uat-fincart-confluent-org | 02-02-2026     | 21-04-2026                  | Removed |
| PROD Kafka  | fincart-confluent-org     | 19-02-2026     | Active                      | Running |

---

# Resource Level Cost Comparison

| Resource / Service                          | 06-Mar-2026 to 05-Apr-2026 | 06-Apr-2026 to 05-May-2026 | Difference | Observation                               |
| ------------------------------------------- | -------------------------- | -------------------------- | ---------- | ----------------------------------------- |
| cft_d8b0b87c_uat-fincart-confluent-org      | ₹44,509                    | ₹20,283                    | -₹24,226   | Kafka SaaS cost reduced                   |
| fincart-confluent-org-1                     | ₹42,149                    | ₹26,611                    | -₹15,539   | UAT Kafka cost reduced after decommission |
| mainfincart/databases/finprod               | ₹134.4K                    | ₹130.4K                    | -₹3,950    | SQL Database optimization                 |
| fincart-stage-linux-app-plan-002            | ₹9,949                     | ₹6,808                     | -₹3,140    | Stage App Service optimization            |
| fincart-stage-linux-app-plan-001            | ₹9,805                     | ₹6,808                     | -₹2,996    | Stage App Service optimization            |
| fincart-prod-linux-app-svc-log-analytics-ws | ₹18,913                    | ₹17,130                    | -₹1,783    | Log Analytics optimization                |
| fincart-uat-linux-appsvc-log-analytics-ws   | ₹8,373                     | ₹6,349                     | -₹2,024    | UAT monitoring optimization               |
| finstage                                    | ₹30,692                    | ₹29,757                    | -₹935      | Stage environment optimization            |
| fincart-uat-linux-app-plan-001              | ₹8,514                     | ₹6,811                     | -₹1,703    | UAT App Service optimization              |
| stagefinca-ca24b0a48a-cdnprofile            | ₹6.39                      | ₹2,070                     | +₹2,064    | CDN usage/scaling increased               |

---

# Total Identified Operational Savings

| Optimization Area              | Savings |
| ------------------------------ | ------- |
| Kafka SaaS Optimization        | ₹24,226 |
| UAT Kafka Decommission         | ₹15,539 |
| SQL Database Optimization      | ₹3,950  |
| Stage App Service Optimization | ₹6,136  |
| Log Analytics Optimization     | ₹1,783  |
| UAT Monitoring Optimization    | ₹2,024  |
| Stage Environment Optimization | ₹935    |
| UAT App Service Optimization   | ₹1,703  |

## Total Identified Savings

24226+15539+3950+6136+1783+2024+935+1703=56396

## Increased Cost

| Resource            | Increased Cost |
| ------------------- | -------------- |
| CDN Profile Scaling | ₹2,064         |

## Net Operational Savings

56396-2064=54332

---

# Key Findings

1. The primary reason for the cost reduction was Kafka-related optimization and UAT Kafka decommissioning.
2. Kafka SaaS charges reduced significantly between the two billing periods.
3. Additional savings were achieved through SQL Database, Log Analytics, and App Service optimization.
4. CDN profile cost increased due to additional usage/scaling activities.
5. Total identified operational savings were approximately ₹54.3K.

---

# Conclusion

The Kafka optimization initiative successfully reduced Azure and Marketplace-related operational costs. The major contribution came from Kafka SaaS optimization and UAT Kafka decommissioning, resulting in significant monthly savings across the infrastructure stack.
