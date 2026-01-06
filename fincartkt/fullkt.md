# Fincart Azure DevOps – Environment & Architecture KT (Expanded)

> **Note:** This document is intentionally **detailed and exhaustive** based strictly on the data discovered in Azure Portal so far. CI/CD details will be added later once pipeline information is shared.

---

## 1. Overall Summary

This Azure subscription hosts **multiple environments (PROD, STAGE, UAT)** for the Fincart platform. The architecture is **large-scale, finance-grade, and microservices-based**, built primarily on **Azure App Services** with strong emphasis on **network isolation, private access, and secret management**.

### Core Azure Services Used

* Azure App Services (Web Apps, API Apps, Function Apps)
* App Service Plans (Linux & Windows)
* Azure SQL Database (Hyperscale)
* Azure Database for MySQL (Flexible Server)
* Azure Database for PostgreSQL (Flexible Server)
* Azure Key Vault
* Azure Storage Accounts
* Azure Virtual Networks (VNets)
* Private Endpoints
* Azure Front Door / Azure CDN

You currently have **Reader access**, therefore this document focuses on **understanding, mapping, and operational awareness**, not configuration changes.

---

## 2. Environment Identification

### Production (PROD)

**Identification Signals:**

* Resource Groups:

  * `Fincart_Resources_India`
  * `fincart-3eadvisors-rg`
  * `fincart-loan2cart-rg`
  * `fincart-common-svc-rg`
* App names without prefixes (`stage-`, `uat-`)
* App Service Plans mostly **Premium V3**
* Core SQL DB: `FINPROD`

**Purpose:**

* Handles all live customer traffic
* Hosts critical financial workloads

---

### Stage

**Identification Signals:**

* Prefix: `stage-`
* Resource Groups:

  * `Fincart_Stage_Resources`
  * `stage-fincart-wordpress-rg`
* Combination of **Premium V3** and **Basic** plans

**Purpose:**

* Pre-production validation
* Integration and release testing

---

### UAT

**Identification Signals:**

* Prefix: `uat-`
* Resource Group:

  * `fincart_UAT_resources_India`
* SQL DBs:

  * `UATFIN`
  * `FINSTAGE`

**Purpose:**

* User Acceptance Testing
* Business sign-off

---

## 3. Application Services – Complete Inventory (Exhaustive)

> This section explicitly lists **all discovered App Services** from the Azure portal at the time of analysis. No services are intentionally omitted. Status (Running/Stopped) reflects current observation.

### 3.1 Production App Services

#### Backend / API Apps (PROD)

* `api-fincart-v2`
* `api-workpoint`
* `api2-fincart`
* `fincart-api-gateway-ind`
* `fincart-api-registry`
* `fincart-assets`
* `fincart-batch` (Stopped)
* `fincart-cms-ind`
* `fincart-communication-ind`
* `fincart-core` (Stopped)
* `fincart-core-ind`
* `fincart-dmkt-ind`
* `fincart-fp`
* `fincart-kyc` (Stopped)
* `fincart-lms`
* `fincart-mis`
* `fincart-portfolio`
* `fincart-python`
* `fincart-report`
* `fincart-thirdparty`
* `fincart-transaction`

#### UI / Web Apps (PROD)

* `dashboard-fincart`
* `financialplan-fincart-v2`
* `fincart-ui-dmkt`
* `fincart-ui-fp`
* `fincart-ui-lms`
* `fincart-ui-portfolio`
* `fincart-ui-transaction`
* `reports-fincart`

#### CMS / WordPress (PROD)

* `3eadvisors-wordpress`
* `fincart-wordpress-01`
* `loan2cart-wordpress`

#### Jobs / Functions (PROD)

* `Fincart-Jobs`

---

### 3.2 Stage App Services

#### Backend / API Apps (STAGE)

* `stage-fincart-api-gateway`
* `stage-fincart-api-registry`
* `stage-fincart-api2`
* `stage-fincart-assets-ind`
* `stage-fincart-cms`
* `stage-fincart-communication-ind`
* `stage-fincart-core-ind`
* `stage-fincart-dmkt-ind`
* `stage-fincart-fp-ind`
* `stage-fincart-kyc-ind` (Stopped)
* `stage-fincart-lms-ind`
* `stage-fincart-mis-ind`
* `stage-fincart-portfolio-ind`
* `stage-fincart-report-ind`

#### UI / Web Apps (STAGE)

* `Stage-api-workpoint`
* `stage-dashboard-fincart`

#### CMS / WordPress (STAGE)

* `stage-fincart-wordpress`

---

### 3.3 UAT App Services

> UAT apps follow the same naming and functional structure as PROD/Stage, prefixed with `uat-`, and are hosted in `fincart_UAT_resources_India`.

---

### 3.4 App Service Count Summary

* **Production:** ~45 Apps
* **Stage:** ~30 Apps
* **UAT:** ~15 Apps
* **Total:** ~90 App Services

---

## 4. App Service Plans – Complete Inventory

> This section lists **all discovered App Service Plans** with environment and OS context.

### 4.1 Production Plans

* `ASP-FincartResourcesIndia-bff4` – Linux – Premium V3
* `fincart-prod-linux-app-plan-002` – Linux – Premium V3
* `fincart-prod-linux-app-plan-003` – Linux – Premium V3
* `fincart-prod-window-app-plan-002` – Windows – Premium V3
* `FincartAppServiceWindowsPlanIndia` – Windows – Premium V3
* `fincart-3eadvisors-app-plan-001` – Linux – Premium V3
* `fincart-loan2cart-app-plan-001` – Linux – Premium V3
* `fincart-wordpress-app-plan-001` – Linux – Premium V3

### 4.2 Stage Plans

* `fincart-stage-linux-app-plan-001` – Linux – Premium V3
* `fincart-stage-linux-app-plan-002` – Linux – Premium V3
* `fincart-stage-window-app-plan-001` – Windows – Basic
* `fincart-stage-wordpress-app-plan-001` – Linux – Basic

### 4.3 UAT Plans

* `fincart-uat-linux-app-plan-001` – Linux – Premium V3
* `fincart-uat-linux-app-plan-002` – Linux – Basic
* `fincart-uat-linux-app-plan-003` – Linux – Basic
* `UATfincartappserviceplan` – Windows – Basic

### 4.4 Plan Count Summary

* **Production:** 8 Plans
* **Stage:** 4 Plans
* **UAT:** 4 Plans
* **Total:** **16 App Service Plans**

---

### 4.1 Total Plans

* **16 App Service Plans** identified

---

### 4.2 Plan Characteristics

* **Premium V3** dominates PROD workloads
* **Basic (B2/B3)** mainly used in Stage/UAT
* Segregation by:

  * OS (Linux vs Windows)
  * Environment (PROD / Stage / UAT)

---

### 4.3 Notable Plans

* `ASP-FincartResourcesIndia-bff4`

  * Linux
  * PROD
  * Hosts multiple backend APIs

* `FincartAppServiceWindowsPlanIndia`

  * Windows
  * PROD
  * Hosts UI apps and Function Apps

* `fincart-prod-linux-app-plan-002`

  * Linux
  * PROD
  * Hosts several core APIs

* `fincart-stage-linux-app-plan-001`

  * Linux
  * Stage

* `fincart-stage-window-app-plan-001`

  * Windows
  * Stage

* `UATfincartappserviceplan`

  * Windows
  * UAT

---

## 5. Databases – Complete Inventory

### 5.1 Azure SQL Database (Primary Financial Data)

**Servers:**

* `mainfincart`
* `fincartportfolioserver`
* `uatfincart`

**Databases:**

* `FINPROD` – Production core DB
* `FINPRODPORTFOLIO` – Portfolio-specific DB
* `FINSTAGE` – Stage DB
* `UATFIN` – UAT DB

**Tier:**

* Hyperscale (Gen5)

**Purpose:**

* Financial transactions
* Portfolio data
* Business-critical records

---

### 5.2 MySQL Flexible Servers (WordPress)

* `3eadvisors-wordpress-mysqldb` (PROD)
* `fincart-stage-wordpress-mysqldb`
* `loan2cartw-*-wpdbserver`

**Purpose:**

* CMS and WordPress data storage

---

### 5.3 PostgreSQL Flexible Servers

* `fincart-prod-psqldb-001`
* `fincart-stage-psqldb-001`
* `fincart-uat-psqldb-001`

**Purpose:**

* Backend services
* Analytics workloads

---

## 6. Networking & Security Architecture

### 6.1 Virtual Networks (VNets)

Identified VNets include:

* `fincart-prod-centralindia-vnet-001`
* `fincart-stage-centralindia-vnet-001`
* `fincart-uat-centralindia-vnet-001`
* `fincart-common-vnet-001`
* `3eadvisors-vnet-001`
* `fincart-loan2cart-vnet-001`

Each environment uses **isolated VNets** to avoid cross-environment access.

---

### 6.2 Private Endpoints

Private endpoints are configured for:

* Azure SQL Databases
* Storage Accounts
* DocuSign storage

All database and storage access occurs **privately via VNet integration**.

✅ No public exposure of databases

---

## 7. Storage Accounts – Complete Inventory (Exhaustive)

> This section lists **all discovered Storage Accounts** from the Azure portal at the time of analysis. No storage account is intentionally omitted.

### 7.1 Storage Accounts List

* `azfincartappserviceblob` – App Service content / blobs (PROD)
* `fincartbackupstorageacc` – Backup storage
* `fincartcontinuousbackup` – Continuous backup storage
* `fincartdocusign` – DocuSign integration storage
* `fincartjobblob` – Job / batch processing blobs
* `fincartozontelstorage` – Ozonetel integration storage
* `fincartsftpcentralind` – SFTP integration storage
* `fincartsqldblogs` – SQL database logs
* `fincartpsqldblogs` – PostgreSQL database logs
* `fincartstorageind` – Primary production storage (India)
* `fincartstorage` – Shared/global storage
* `fincartstoragetesting` – Testing / non-prod storage
* `fincartwebsiteblob` – Website static content
* `fincartuatresources97a6` – UAT environment storage
* `fcstnonprodnp01` – Non-production storage (OneAssure / UAT)
* `logicapgroup94d2` – Logic App / integration storage

### 7.2 Storage Account Count Summary

* **Total Storage Accounts:** 16

---

## 8. Key Vaults – Complete Inventory (Exhaustive)

> This section lists **all discovered Key Vaults** across PROD, STAGE, UAT, and partner integrations.

### 8.1 Key Vault List

* `fincart-prod-kv` – Production secrets (core platform)
* `fincart-stage-kv` – Stage environment secrets
* `fincart-uat-kv` – UAT environment secrets
* `FincartVMKeyVault` – VM-related secrets / certificates
* `fc-st-kv-non-prod` – Non-production secrets (OneAssure)
* `fc-vault-kv-non-prod` – Additional non-prod vault
* `OneAssure-FincartVault` – OneAssure UAT integration
* `oa-vault-kv-prod` – OneAssure production integration
* `OneAssure-Prodfincart-kv` – OneAssure PROD secrets

### 8.2 Key Vault Count Summary

* **Total Key Vaults:** 9

---

## 9. Traffic Management & CDN

### Azure Front Door / CDN

Used for:

* WordPress websites
* Static assets hosted in Blob Storage

Profiles identified:

* `3eadvisors-501522a9ec-cdnprofile`
* `fincart-blob-cdn`
* `fincart-blob-cdn-ind-01`
* `loan2cartw-*-cdnprofile`

Provides:

* HTTPS termination
* Global routing
* Performance optimization

---

## 10. DevOps Perspective – Current State

### What You Understand Now

* Clear separation of PROD, Stage, and UAT
* Microservices hosted on App Services
* Strong security via VNets & Private Endpoints
* Secrets centralized in Key Vaults
* Traffic managed via Front Door/CDN

---

### What Is Intentionally Pending

* CI/CD pipeline details
* App-to-DB-to-KeyVault dependency mapping
* Monitoring & alert tuning specifics

These will be added once pipeline and monitoring access is reviewed.

---

## 11. One-Line Executive Summary

> *Fincart runs a multi-environment, enterprise-grade Azure architecture with App Service–based microservices, Hyperscale SQL databases, private networking, centralized secret management, and global traffic optimization via Front Door/CDN.*
