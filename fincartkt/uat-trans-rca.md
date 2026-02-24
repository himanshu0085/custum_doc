# Postmortem Report \| UAT Service Interruption -- Container Recycle & External API Failure \| February 21, 2026

------------------------------------------------------------------------

## 1. Introduction

A UAT service interruption occurred on February 21, 2026, impacting the
transaction processing service. The incident involved repeated container
recycle events within Azure App Service and concurrent failures in the
external Axis mandate API integration. The combination of
application-level runtime exceptions and container restarts resulted in
temporary transaction disruption before automatic recovery.

------------------------------------------------------------------------

## 2. Key Information (Metadata)

  -----------------------------------------------------------------------
  Field                         Details
  ----------------------------- -----------------------------------------
  Incident Type                 Service Interruption \[Container
                                Recycle + External API Failure\]

  Severity                      Major

  Repeated Incident             Yes (Multiple recycle events observed)

  Affected Services             uat-fincart-transaction-ind

  Start Time                    February 21, 2026, 18:30 IST

  Progression                   CRON / Mandate flow triggered → Axis API
                                returned malformed JWT → Runtime
                                exceptions logged → Container recycle
                                events observed → Temporary service
                                disruption

  End Time                      February 21, 2026, \~18:45 IST

  Incident Duration             \~15 minutes

  Acknowledge Duration          10 minutes

  Issue Detection               Azure AppServiceConsoleLogs & App Service
                                Container Events

  Incident Lead                 Priyanshu Yadav

  Participants                  Transaction Service Team
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 3. Issue Summary

### What Happened

During scheduled transaction processing, the application invoked the
external Axis mandate API. The API returned the following error:

    errorCode: 300
    Invalid serialized unsecured/JWS/JWE object: Missing part delimiters

The malformed response triggered a `RuntimeException` within:

    ThirdPartyConnector.unifiedMandate()

During the same window, Azure App Service recorded container recycle
events. During recycling, the application container restarted
automatically, causing brief service interruption.

------------------------------------------------------------------------

### Impact

-   Mandate / transaction processing failed temporarily\
-   Runtime exceptions observed in logs\
-   Container restarted automatically\
-   Temporary unavailability during recycle window\
-   No permanent data loss

Azure infrastructure recovered automatically after container restart.

------------------------------------------------------------------------

### Resolution

-   Container restarted automatically by Azure platform\
-   Service stabilized post-restart\
-   Application logs reviewed to identify malformed JWT response\
-   Infrastructure metrics validated and found stable

------------------------------------------------------------------------

## 4. Timeline (IST)

  Time           Event
  -------------- ------------------------------------------
  18:29          Scheduled flow / CRON triggered
  18:30:39       Axis API returned malformed JWT response
  18:30:39       RuntimeException logged
  18:30--18:31   Multiple API failures observed
  18:34:24       Container recycle event triggered
  18:35          Container restarted automatically
  \~18:45        Service stabilized

------------------------------------------------------------------------

## 5. Root Cause Analysis

### Problem

Transaction processing was interrupted due to application runtime
failures and container recycle events.

------------------------------------------------------------------------

### Root Cause

The external Axis mandate API returned a malformed JWT/JWS response
(`errorCode 300`). The invalid token caused repeated `RuntimeException`
errors during response parsing.

These application-level failures coincided with Azure App Service
container recycle events. During recycle, the container stopped and
restarted automatically, leading to temporary service interruption.

There was no confirmed evidence of memory saturation during this
incident window.

------------------------------------------------------------------------

### Three Whys Analysis

**Why did the service go down?**\
The container restarted, causing temporary service interruption.

**Why did the container restart?**\
Azure App Service initiated recycle events during the instability
window.

**Why was instability observed?**\
The application experienced repeated runtime failures due to malformed
external API responses.

------------------------------------------------------------------------

## 6. Impact and Mitigation

### Customer Impact

UAT users experienced temporary transaction failures and brief service
unavailability. No permanent service outage or data loss occurred.

------------------------------------------------------------------------

### Mitigation Steps

-   Container restarted automatically\
-   Logs reviewed for API failure pattern\
-   Infrastructure stability validated\
-   Monitoring continued post-recovery

------------------------------------------------------------------------

## 7. Lessons Learned

### What went well

-   Azure platform auto-recovered service\
-   Detailed logs helped isolate API error\
-   No manual restart required

### What could improve

-   Add defensive JWT validation before parsing\
-   Implement retry logic for external API calls\
-   Configure alerts for repeated runtime failures\
-   Investigate detailed triggers of container recycle events

------------------------------------------------------------------------

## 8. Recommended Preventive Actions

-   Implement structured exception handling to prevent cascading
    failures\
-   Add retry with exponential backoff for Axis API calls\
-   Enable proactive alerts for container recycle events\
-   Review App Service health metrics during failure window\
-   Coordinate with Axis API team regarding JWT format compliance

------------------------------------------------------------------------

## 9. Final Root Cause Statement

The incident was caused by repeated runtime exceptions triggered by a
malformed JWT/JWS response returned from the external Axis mandate API.
These application-level failures coincided with Azure App Service
container recycle events, resulting in temporary service interruption.
The container restarted automatically, and service stabilized without
manual intervention. Preventive improvements in external API validation,
retry logic, and monitoring are recommended to prevent recurrence.
