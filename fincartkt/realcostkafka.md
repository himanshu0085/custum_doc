# Kafka Cost Optimization Summary

## Overview

This document summarizes only the Kafka-related cost optimization activities performed between the following billing periods:

* 06-Mar-2026 to 05-Apr-2026
* 06-Apr-2026 to 05-May-2026

---

# Kafka Environment Details

| Environment | Resource Name             | Provisioned On | Decommissioned / Stopped On | Status  |
| ----------- | ------------------------- | -------------- | --------------------------- | ------- |
| UAT Kafka   | uat-fincart-confluent-org | 02-02-2026     | 21-04-2026                  | Removed |
| PROD Kafka  | fincart-confluent-org     | 19-02-2026     | Active                      | Running |

---

# Kafka Related Cost Comparison

| Kafka Resource / Service               | 06-Mar-2026 to 05-Apr-2026 | 06-Apr-2026 to 05-May-2026 | Difference | Observation                      |
| -------------------------------------- | -------------------------- | -------------------------- | ---------- | -------------------------------- |
| cft_d8b0b87c_uat-fincart-confluent-org | ₹44,509                    | ₹20,283                    | -₹24,226   | Kafka SaaS usage reduced         |
| fincart-confluent-org-1                | ₹42,149                    | ₹26,611                    | -₹15,539   | UAT Kafka cluster decommissioned |

---

# Kafka Optimization Savings

| Optimization Activity   | Savings |
| ----------------------- | ------- |
| Kafka SaaS Optimization | ₹24,226 |
| UAT Kafka Decommission  | ₹15,539 |

## Total Kafka Related Savings

24226+15539=39765

---

# Key Findings

1. The primary reason for the cost reduction was Kafka-related optimization activities.
2. UAT Kafka environment was decommissioned on 21-04-2026, significantly reducing Kafka-related charges.
3. Kafka SaaS consumption costs were also optimized during the second billing cycle.
4. Total identified Kafka-related savings were approximately ₹39.8K between the two billing periods.

---

# Conclusion

Kafka optimization activities resulted in a significant reduction in Marketplace/SaaS operational costs. The major savings came from UAT Kafka decommissioning and reduced Kafka SaaS consumption across the environment.
