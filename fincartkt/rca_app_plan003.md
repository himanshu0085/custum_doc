# Postmortem Report | UAT Service Interruption due to Memory Saturation | February 13, 2026

---

## 1. Introduction

A UAT service interruption affected critical services (LMS, MIS) due to memory saturation in the Azure App Service Plan (`fincart-uat-linux-app-plan-003`), leading to temporary service disruption and application container restart. Memory pressure triggered platform-level recycle events, resulting in temporary unavailability before automatic recovery.

---

## 2. Key Information (Metadata)

| Field | Details |
|------|---------|
| Incident Type | Service Interruption [memory pressure / container recycle] |
| Severity | Major |
| Repeated Incident | No |
| Affected Services | LMS, MIS (Azure App Service) |
| Start Time | February 13, 2026, 11:15 AM IST (memory saturation observed) |
| Progression | Memory utilization increased beyond safe limits → containers restarted → services temporarily unavailable → automatic recovery observed |
| End Time | February 13, 2026, 1:45 PM IST (memory stabilized and services recovered) |
| Incident Duration | ~2 hours 30 minutes |
| Acknowledge Duration | 10 minutes |
| Issue Detection | Azure Monitor Metrics |
| Incident Lead | DevOps Team |
| Participants | Platform Engineering Team |

---

## 3. Issue Summary

**What Happened:**  
Two UAT services (LMS and MIS) experienced temporary interruption due to memory saturation in the App Service Plan. High memory utilization triggered Azure platform container recycle events, temporarily stopping application processes.

**Impact:**  
Both LMS and MIS services experienced temporary service disruption during container restart. Memory working set dropped to 0 GB during recycle, confirming service interruption. Azure platform remained available.

**Resolution:**  
Azure App Service platform automatically restarted application containers, restoring service availability. Scaling App Service Plan to a higher memory tier was recommended to prevent recurrence.

---

## 4. Timeline

| Time | Event |
|------|------|
| 10:30 AM | Memory utilization elevated (~70–85%) |
| 11:15 AM | Memory utilization exceeded critical threshold (>85%) |
| 12:05 PM | Memory working set dropped to 0 GB, indicating container recycle |
| 12:06 PM | Application containers restarted automatically |
| 1:20 PM | Peak memory utilization observed (~90%) |
| 1:30 PM | Memory utilization began decreasing |
| 1:45 PM | Memory stabilized (~55–65%), services fully recovered |

---

## 5. Root Cause Analysis

**Problem:**  
Service interruption occurred due to insufficient memory headroom in the Azure App Service Plan.

**Root Cause:**  

App Service Plan (`fincart-uat-linux-app-plan-003`, B3 tier, 7 GB RAM) was hosting two Java-based services:

- `uat-fincart-lms-ind`
- `uat-fincart-mis-ind`

Both services consumed significant memory (~1.0–1.3 GB each), and App Service Plan memory utilization reached critical levels (~85–92%).

Due to insufficient memory headroom, Azure App Service platform initiated container recycle events to maintain infrastructure stability. During recycle, application containers were temporarily stopped, causing memory working set to drop to 0 GB and resulting in temporary service interruption.

---

### Three Whys Analysis

**Why did services go down?**  
Application containers restarted, causing temporary service interruption.

**Why did application containers restart?**  
Azure platform initiated recycle due to high memory utilization.

**Why was memory utilization high?**  
App Service Plan (B3, 7 GB RAM) had insufficient memory capacity for combined workload of LMS and MIS Java services.

---

## 6. Impact and Mitigation

**Customer Impact:**  
UAT users experienced temporary service interruption and performance degradation during container recycle. No permanent service failure occurred.

**Mitigation Steps:**

- Azure platform automatically restarted affected containers
- Services recovered automatically after restart
- Memory utilization monitored and validated post-recovery
- Scaling App Service Plan to higher memory tier recommended

---

## 7. Lessons Learned

### What went well:

- Azure platform automatically recovered services after container restart
- Monitoring metrics helped identify memory saturation
- No manual intervention required for service recovery

### What could improve:

- Memory utilization monitoring should include proactive alerts
- Capacity planning should consider memory requirements of Java-based services
- App Service Plan should maintain sufficient memory headroom to prevent recycle events
- Higher memory tier should be used for memory-intensive workloads

---

## 8. Recommended Preventive Actions

- Upgrade App Service Plan from B3 to P1V3 or higher tier
- Configure Azure Monitor alerts for memory utilization >70%
- Implement autoscaling based on memory metrics
- Periodically review infrastructure capacity for hosted services

---

## 9. Final Root Cause Statement

The incident was caused by insufficient memory headroom in the Azure App Service Plan (`fincart-uat-linux-app-plan-003`, B3 tier, 7 GB RAM) hosting two Java-based services (`uat-fincart-lms-ind` and `uat-fincart-mis-ind`). Combined memory utilization reached critical levels (~85–92%), triggering Azure platform container recycle events. During the recycle process, application containers were temporarily stopped, resulting in temporary service interruption. Services recovered automatically after restart. Scaling the App Service Plan to a higher memory tier is recommended to prevent recurrence.

---
