# Fincart Azure DevOps – Environment & Architecture KT

## 1. Overall Summary

This Azure subscription hosts **multiple environments (PROD, STAGE, UAT)** for the Fincart platform.  
The setup is **microservices-based**, heavily using **Azure App Services, App Service Plans, SQL/MySQL/PostgreSQL databases, VNets, Private Endpoints, Key Vaults, and Front Door/CDN**.

You currently have **Reader access**, so this document is focused on **understanding, mapping, and operating awareness**, not changes.

---

## 2. Environment Identification

### Production (PROD)

Identified mainly by:
- Resource Groups:  
  `Fincart_Resources_India`, `fincart-3eadvisors-rg`, `fincart-loan2cart-rg`, `fincart-common-svc-rg`
- App names **without prefix** (no `stage-` / `uat-`)
- **Premium V3** App Service Plans
- SQL DB name: `FINPROD`

---

### Stage

Identified by:
- Prefix: `stage-`
- Resource Groups:  
  `Fincart_Stage_Resources`, `stage-fincart-wordpress-rg`
- Mix of **Premium V3** and **Basic** plans

---

### UAT

Identified by:
- Prefix: `uat-`
- Resource Group: `fincart_UAT_resources_India`
- SQL DBs: `UATFIN`, `FINSTAGE`

---

## 3. App Services Overview

### Total App Services
- **~90 App Services**  
  (Web Apps + API Apps + Function Apps)

---

### Major Categories

#### A. Core Backend APIs (Microservices)

Examples:
- `fincart-core-ind`
- `fincart-portfolio`
- `fincart-transaction`
- `fincart-thirdparty`
- `fincart-communication-ind`

**Purpose:**
- Business logic (Finance, Portfolio, Transactions, KYC, LMS, MIS, Reporting)

---

#### B. UI / Frontend Apps

Examples:
- `dashboard-fincart`
- `fincart-ui-portfolio`
- `fincart-ui-transaction`
- `fincart-ui-fp`

**Purpose:**
- Customer / Advisor-facing web applications

---

#### C. WordPress Sites

Examples:
- `3eadvisors-wordpress` (PROD)
- `fincart-wordpress-01`
- `loan2cart-wordpress`
- `stage-fincart-wordpress`

**Purpose:**
- Marketing websites
- Advisor portals
- CMS-based content

---

#### D. Background Jobs / Workers

Examples:
- `Fincart-Jobs` (Function App)
- `fincart-batch`

**Purpose:**
- Scheduled jobs
- Async/background processing

---

## 4. App Service Plans

### Total App Service Plans
- **16 App Service Plans**

---

### Key Observations
- **Premium V3** used for PROD and critical workloads
- **Basic** plans mainly for Stage/UAT
- Plans segregated by:
  - Linux vs Windows
  - PROD vs Stage vs UAT

---

### Examples
- `ASP-FincartResourcesIndia-bff4`  
  → PROD Linux (multiple APIs)

- `FincartAppServiceWindowsPlanIndia`  
  → PROD Windows (UI + Jobs)

- `fincart-stage-linux-app-plan-001`  
  → Stage Linux APIs

---

## 5. Databases

### Azure SQL (Core Transactional DB)

**Servers:**
- `mainfincart`
- `fincartportfolioserver`
- `uatfincart`

**Databases:**
- `FINPROD` (PROD)
- `FINPRODPORTFOLIO`
- `FINSTAGE`
- `UATFIN`

**Tier:**
- Hyperscale (Gen5)

**Purpose:**
- Core financial & transactional data

---

### MySQL Flexible Servers (WordPress)

Examples:
- `3eadvisors-wordpress-mysqldb` (PROD)
- `fincart-stage-wordpress-mysqldb`
- `loan2cartw-*-wpdbserver`

**Purpose:**
- WordPress CMS data

---

### PostgreSQL Flexible Servers

Examples:
- `fincart-prod-psqldb-001`
- `fincart-stage-psqldb-001`
- `fincart-uat-psqldb-001`

**Purpose:**
- Backend services
- Analytics workloads

---

## 6. Networking & Security

### VNets
- Separate VNets per environment:
  - `fincart-prod-centralindia-vnet-001`
  - `fincart-stage-centralindia-vnet-001`
  - `fincart-uat-centralindia-vnet-001`

- Dedicated VNets for:
  - WordPress
  - Loan2Cart
  - Common / Shared services

---

### Private Endpoints
- Azure SQL Databases
- Storage Accounts
- DocuSign integrations

✅ **No public DB exposure**  
(Finance-grade security posture)

---

## 7. Storage Accounts

### Count
- **16 Storage Accounts**

### Usage Types
- App Service blobs
- SQL / PostgreSQL logs
- Backup storage
- SFTP integrations
- DocuSign & third-party integrations

---

## 8. Key Vaults

### Count
- **9 Key Vaults**

### Separation
- `fincart-prod-kv` → PROD secrets
- `fincart-stage-kv` → Stage
- `fincart-uat-kv` → UAT
- OneAssure-specific Key Vaults for partner integrations

**Purpose:**
- Secrets
- DB credentials
- API keys

---

## 9. Traffic & CDN

### Front Door / CDN
- Azure Front Door
- Azure CDN (Classic + Standard)

**Used for:**
- WordPress websites
- Blob/static content

**Examples:**
- `3eadvisors-501522a9ec-cdnprofile`
- `fincart-blob-cdn-ind-01`

---

## 10. DevOps Perspective – What This Means for You

### Responsibilities (Understanding Phase)
- Know **which app runs where**
- Know **which DB & Key Vault an app depends on**
- Know **which App Service Plan hosts which apps**
- Understand **PROD vs Stage vs UAT isolation**

---

### Next Logical DevOps Checks
- CI/CD pipelines (Azure DevOps / GitHub)
- Deployment strategy (Auto vs Manual)
- Monitoring & alert tuning
- Cost optimization (large Premium V3 footprint)

---

## 11. One-Line Summary (For Manager / Seniors)

> *The Fincart Azure setup is a multi-environment, microservices-based architecture using Azure App Services, Premium V3 plans, SQL/MySQL/PostgreSQL databases with private networking, Front Door/CDN for traffic, and Key Vault–based secret management.*
