# Grafana VM Monitoring & Retention Document

## 1. Objective

This document contains the implemented monitoring alerts, retention policies, and notification configurations configured for the Grafana monitoring VM.

---

# 2. Monitoring Alerts Configured

## CPU Alerts

| Alert Name     | Threshold |
|----------------|-----------|
| CPU 60 PERCENT | 60% |
| CPU 70 PERCENT | 70% |
| CPU 80 PERCENT | 80% |

---

## Disk Alerts

| Alert Name      | Threshold |
|-----------------|-----------|
| DISK 60 PERCENT | 60% |
| DISK 70 PERCENT | 70% |
| DISK 80 PERCENT | 80% |

---

## Memory Alerts

| Alert Name                | Threshold |
|---------------------------|-----------|
| USED MEMORY 60 PERCENT    | 60% |
| USED MEMORY 70 PERCENT    | 70% |
| USED MEMORY 80 PERCENT    | 80% |
| AVERAGE MEMORY 60 PERCENT | 60% |
| AVERAGE MEMORY 70 PERCENT | 70% |
| AVERAGE MEMORY 80 PERCENT | 80% |

---

## Log Usage Alerts

| Alert Name           | Threshold |
|----------------------|-----------|
| LOG_USAGE_60_PERCENT | 60% |
| LOG_USAGE_70_PERCENT | 70% |
| LOG_USAGE_80_PERCENT | 80% |

---

# 3. Loki Retention Configuration

## Retention Policy

| Environment | Retention |
|-------------|-----------|
| UAT | 7 Days |
| Stage | 95 Days |

---

## Retention Implementation Details

### UAT Environment

UAT log retention is directly configured through the Loki retention configuration.

Configured retention:

- `168h (7 days)`

---

### Stage Environment

Stage environment retention has been configured for 95 days to support archival safety and prevent accidental log loss during archival operations.

A cron job is configured for automated archival and cleanup operations using:

```bash
/opt/observability/scripts/loki-archive-stage.sh
````

### Stage Retention Workflow

* Logs older than 91 days are archived
* Archived logs are uploaded to Azure Blob Storage
* Logs are deleted from VM after successful archival
* Additional retention buffer is maintained to avoid log loss during upload or cleanup operations

This ensures:

* storage optimization
* archival safety
* operational stability
* retention compliance

---

# 4. Retention Configuration Screenshots

## Screenshot 1 — Loki Retention Configuration

Capture:

* `retention_enabled`
* UAT → `168h`
* Stage → `2280h`

---

## Screenshot 2 — Stage Cron Job Configuration

Run:

```bash
sudo crontab -l
```

Capture:

```bash
0 20 * * * /opt/observability/scripts/loki-archive-stage.sh
```

---

## Screenshot 3 — Journal Retention

Run:

```bash
sudo grep -i SystemMaxUse /etc/systemd/journald.conf
```

Capture:

```bash
SystemMaxUse=500M
```

---

# 5. Monitoring Alert Screenshots

## Screenshot 4 — Alert Rules List

Capture Azure Monitor Alert Rules page showing:

* CPU alerts
* DISK alerts
* MEMORY alerts
* LOG_USAGE alerts

---

## Screenshot 5 — Log Usage Alert Configuration

Capture:

* LOG_USAGE alerts
* Threshold configuration
* Severity

---

# 6. Action Group Configuration

## Configured Action Group

```text
DiskAlertGroupMonitoring
```

---

## Notification Recipients

| Name      | Email                                                                 |
| ----------| --------------------------------------------------------------------- |
| Gaurav    | [singh.gaurav@fincart.com](mailto:singh.gaurav@fincart.com)           |
| Kewal     | [kewal.sharma@fincart.com](mailto:kewal.sharma@fincart.com)           |
| Himanshu  | [himanshu.parashar@opstree.com](mailto:himanshu.parashar@opstree.com) |
| Priyanshu | [priyanshu.yadav@opstree.com](mailto:priyanshu.yadav@opstree.com)     |

---

## Screenshot 6 — Action Group Configuration

Capture:

* Action group name
* Email recipients
* Enabled status

---

# 7. Monitoring Summary

The Grafana VM monitoring setup has been successfully configured with:

* CPU utilization alerts
* Disk utilization alerts
* Memory utilization alerts
* Log usage alerts
* Loki retention policies
* Automated archival cron jobs
* Azure Monitor action groups

The implemented monitoring and retention configuration ensures:

* proactive monitoring
* storage visibility
* retention compliance
* alert-based notifications
* operational stability

---

# 8. Final Monitoring & Retention Overview

| Component            | Status     |
| -------------------- | ---------- |
| CPU Monitoring       | Configured |
| Disk Monitoring      | Configured |
| Memory Monitoring    | Configured |
| Log Usage Monitoring | Configured |
| Action Groups        | Configured |
| Loki Retention       | Configured |
| Cron Automation      | Configured |
| Journal Retention    | Configured |


