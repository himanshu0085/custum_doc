# Kafka Cost Optimization Summary

## Overview

This document summarizes the Kafka-related cost optimization activities performed in Azure and Confluent Cloud between the last two billing cycles.

---

# Invoice-Level Cost Comparison

| Billing Period             | Invoice Type                     | Amount       |
| -------------------------- | -------------------------------- | ------------ |
| 06-Feb-2026 to 05-Mar-2026 | Azure Services                   | ₹7,98,280.81 |
| 06-Mar-2026 to 05-Apr-2026 | Azure Services                   | ₹7,60,521.35 |
| 01-Mar-2026 to 31-Mar-2026 | Azure Marketplace & Reservations | ₹86,566.03   |
| 01-Apr-2026 to 30-Apr-2026 | Azure Marketplace & Reservations | ₹73,538.57   |

## Billing Clarification

* Azure Services invoice is calculated from 06-Mar-2026 to 05-Apr-2026.
* Marketplace & Reservations invoice is calculated on calendar month basis.
* The latest available billing data is up to 05-May-2026.
* Invoice-level differences can include taxes, billing adjustments, and rounding differences.

---

# Kafka Environment Details

| Environment | Resource Name             | Provisioned On | Decommissioned / Stopped On | Status  |
| ----------- | ------------------------- | -------------- | --------------------------- | ------- |
| UAT Kafka   | uat-fincart-confluent-org | 02-02-2026     | 21-04-2026                  | Removed |
| PROD Kafka  | fincart-confluent-org     | 19-02-2026     | Active                      | Running |

---

# Kafka Related Cost Impact

| Resource / Service                          | 06-Mar-2026 to 05-Apr-2026 Cost | 06-Apr-2026 to 05-May-2026 Cost | Difference | Observation                          |
| ------------------------------------------- | ------------------------------- | ------------------------------- | ---------- | ------------------------------------ |
| fincart-confluent-org-1                     | ₹18,160                         | ₹0                              | -₹18,160   | UAT Kafka cluster removed            |
| cft_d8b0b87c_uat-fincart-confluent-org      | ₹15,480                         | ₹0                              | -₹15,480   | Kafka SaaS charges optimized/reduced |
| fincart-uat-linux-appsvc-log-analytics-ws   | ₹2,750                          | ₹1,063                          | -₹1,687    | UAT monitoring/logging cost reduced  |
| fincart-uat-linux-app-plan-001              | ₹3,021                          | ₹1,606                          | -₹1,415    | UAT App Service optimization         |
| mainfincart/databases/finprod               | ₹47,800                         | ₹42,026                         | -₹5,774    | SQL Database optimization            |
| fincart-prod-linux-app-svc-log-analytics-ws | ₹7,912                          | ₹4,895                          | -₹3,016    | Log Analytics optimization           |
| fincartstorageind                           | ₹12,084                         | ₹10,325                         | -₹1,759    | Storage optimization                 |
| finstage                                    | ₹10,931                         | ₹9,545                          | -₹1,386    | Stage environment optimization       |

---

# Net Optimization Summary

| Type                                 | Amount  |
| ------------------------------------ | ------- |
| Total Identified Operational Savings | ₹48,677 |

---

# Key Findings

1. The primary reason for the cost reduction was the removal of the UAT Kafka cluster from Confluent Cloud on 21-04-2026.
2. Kafka Marketplace/SaaS charges were significantly reduced after optimization activities.
3. Additional savings were achieved through SQL Database, Log Analytics, Storage, and App Service optimization.
4. The identified operational savings from optimized resources is approximately ₹48.7K.

---

# Conclusion

The Kafka optimization initiative successfully reduced Azure and Marketplace-related operational costs. The major contribution came from decommissioning the UAT Kafka environment and optimizing Kafka SaaS usage, resulting in measurable monthly savings across the infrastructure stack.
