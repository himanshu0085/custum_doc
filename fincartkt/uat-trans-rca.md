# Postmortem Report \| UAT Service Interruption -- Container Recycle & External API Failure \| February 21, 2026 (UTC)

------------------------------------------------------------------------

## 1. Introduction

A UAT service interruption occurred on February 21, 2026, impacting the
transaction processing service (`uat-fincart-transaction-ind`).

The incident involved repeated Azure App Service container recycle
events along with application runtime failures caused by malformed
responses from the external Axis mandate API.

Temporary service instability was observed before automatic platform
recovery.

## 2. Key Information (Metadata)

| Field | Details |
|-------|---------|
| Incident Type | Service Interruption [Container Recycle + External API Failure] |
| Severity | Major |
| Repeated Incident | Yes (Multiple recycle events observed) |
| Affected Services | uat-fincart-transaction-ind |
| Start Time (UTC) | February 21, 2026, 10:34:24 UTC |
| End Time (UTC) | February 21, 2026, 13:44:41 UTC |
| Incident Duration | ~3 hours 10 minutes |
| Acknowledge Duration | 10 minutes |
| Issue Detection | Azure App Service Diagnostics – Web App Restart + AppServiceConsoleLogs |
| Incident Lead | Priyanshu Yadav |
| Participants | Himanshu Parashar |

## 3. Issue Summary

### What Happened

During transaction processing, the application invoked the external Axis
mandate API.

The API returned the following error:

    errorCode: 300
    Invalid serialized unsecured/JWS/JWE object: Missing part delimiters

This caused repeated `RuntimeException` errors in:

    ThirdPartyConnector.unifiedMandate()
    MFTransactionServiceImpl

During the same period, Azure App Service recorded multiple container
recycle events.

When the container recycled: - Application process stopped - Service
became temporarily unavailable - Container restarted automatically

------------------------------------------------------------------------

## 4. Timeline (UTC)

  Time (UTC)     Event
  -------------- -----------------------------------------
  10:30--10:40   Elevated HTTP server errors observed
  10:34:24       First container recycle event triggered
  10:35          Container restarted automatically
  12:39:32       Second container recycle event
  13:19:22       Third container recycle event
  13:44:41       Fourth container recycle event
  Post 13:45     Service stabilized

------------------------------------------------------------------------

<img width="1837" height="875" alt="Screenshot from 2026-02-24 11-00-14" src="https://github.com/user-attachments/assets/70671c86-01a1-46cc-8520-f29a06b2448e" />


## 5. Root Cause Analysis

### Problem

Transaction processing service experienced temporary instability due to
repeated container recycle events and runtime exceptions triggered by
malformed external API responses.

------------------------------------------------------------------------

### Root Cause

The external Axis mandate API returned a malformed JWT/JWS response
(`errorCode 300`).

The invalid token caused repeated runtime exceptions during parsing and
mandate processing.

Concurrently, Azure App Service initiated container recycle events at:

-   10:34:24 UTC\
-   12:39:32 UTC\
-   13:19:22 UTC\
-   13:44:41 UTC

During each recycle event: - The application container stopped - The
working process restarted - Temporary service interruption occurred

No confirmed evidence of memory saturation was observed during this
window.

------------------------------------------------------------------------

### Three Whys Analysis

**Why did the service become unavailable?**\
Because the container restarted during recycle events.

**Why did the container restart?**\
Azure App Service initiated recycle events.

**Why was instability occurring?**\
Repeated runtime failures were triggered by malformed external API
responses.

------------------------------------------------------------------------

## 6. Impact and Mitigation

### Customer Impact

-   Temporary mandate/transaction failures\
-   Brief service unavailability during recycle windows\
-   No data loss observed

------------------------------------------------------------------------

### Mitigation Steps

-   Azure platform automatically restarted containers\
-   Application logs reviewed for API error pattern\
-   Infrastructure health validated\
-   Monitoring continued after stabilization

------------------------------------------------------------------------

## 7. Lessons Learned

### What Went Well

-   Automatic recovery by Azure platform\
-   Logs clearly identified external API failure\
-   No manual restart required

### What Could Improve

-   Add defensive validation for JWT before parsing\
-   Implement retry logic for external API calls\
-   Configure proactive alerts for container recycle events\
-   Investigate detailed triggers behind recycle events

------------------------------------------------------------------------

## 8. Recommended Preventive Actions

-   Implement structured exception handling\
-   Add exponential backoff retry for Axis API calls\
-   Enable alerts for container restart events\
-   Monitor HTTP 5xx spike thresholds\
-   Coordinate with Axis API team regarding JWT compliance

------------------------------------------------------------------------

## 9. Final Root Cause Statement

The incident was caused by repeated runtime exceptions triggered by a
malformed JWT/JWS response returned from the external Axis mandate API.

These failures coincided with Azure App Service container recycle events
occurring at 10:34:24, 12:39:32, 13:19:22, and 13:44:41 UTC, resulting
in temporary service interruptions.

The Azure platform automatically restarted the containers, and the
service stabilized without manual intervention.
