# Infrastructure Architecture – Deep Dive Explanation

<img width="3620" height="2338" alt="image" src="https://github.com/user-attachments/assets/6a9cdd53-a914-45a9-b0e1-502f26b00986" />



## 1. Entry Layer – Internet & DNS
**Components**
- Internet
- Public DNS (GoDaddy)

**Flow**
- User accesses application using a domain name.
- DNS resolves the domain and routes traffic to Azure App Services.

**Purpose**
- Acts only as a name resolution layer.
- No application logic or data processing happens here.

**Security**
- Only App Services are publicly reachable.
- No database or internal service is exposed to the internet.

---

## 2. Web Hosting Layer – Azure App Services

### 2.1 WordPress Main Website (Linux App Service)
**App Service Plan**
- Linux-based App Service Plan.
- Hosts only the WordPress main website (cost-optimized).

**Traffic Flow**
- User → DNS → WordPress App Service.

**Security & Secrets**
- SSL certificate is purchased using **Azure App Service Certificates**.
- Certificates are securely stored in **Azure Key Vault**.
- WordPress application retrieves SSL certificates from Key Vault.

**Data Access**
- Uses **Private Endpoints** to connect to:
  - Azure Database for MySQL
  - Azure Key Vault

**Why This Design**
- Ensures databases and secrets are never exposed publicly.
- All sensitive communication happens over private Azure networking.

---

### 2.2 Frontend & API Applications (Windows App Service – India)
**App Service Plan**
- Windows-based App Service Plan.
- Hosts multiple applications:
  - React UI applications
  - Angular UI applications
  - .NET Core backend APIs
  - Azure Function App for background jobs

**Frontend Role**
- Serves UI to end users.
- Communicates only with APIs, never directly with databases.

**Backend Role (.NET APIs & Function App)**
- Implements business logic.
- Handles API requests from frontend.
- Executes background and scheduled jobs.

**Security**
- Backend services access:
  - Azure Key Vault
  - Azure Storage Account
  - Databases
- Access is controlled using:
  - Firewall rules
  - “Allow trusted Azure services” configuration

---

## 3. API Gateway Layer – Azure API Management (APIM)
**Purpose**
- Serves as a centralized gateway for all backend APIs.

**Traffic Flow**
- Frontend applications → APIM → Backend APIs.

**Key Capabilities**
- Authentication and authorization
- Rate limiting and throttling
- API versioning
- Centralized monitoring and logging

**Security**
- Backend APIs are not exposed directly to the internet.
- Only APIM is allowed to invoke backend services.

---

## 4. Backend Services – Java Microservices (Linux App Service)
**App Service Plan**
- Linux-based App Service Plan.
- Java (Spring Boot) stack.
- Hosts multiple backend microservices such as:
  - Core services
  - Batch processing
  - Communication services
  - MIS and portfolio services

**Traffic Flow**
- Requests reach Java services through APIM.

**Data Access**
- Azure SQL Database (FINPROD)
- PostgreSQL database

**Security**
- Database access restricted using:
  - Firewall rules
  - Trusted Azure resource policies
- No public database access is enabled.

---

## 5. Data Layer – Databases

### Databases in Use
- Azure Database for MySQL
- Azure SQL Database (FINPROD)
- PostgreSQL

### Access Model
- Databases are private and not internet-facing.
- Access allowed only from:
  - Approved App Services
  - Trusted Azure resources
  - Private Endpoints (where applicable)

### Security Controls
- Network isolation
- Firewall rules
- Credentials stored securely in Azure Key Vault

---

## 6. Secrets & Certificate Management – Azure Key Vault
**Stored Items**
- SSL certificates
- Database credentials
- Application secrets

**Access Control**
- Only trusted Azure services can access secrets.
- Secrets are never stored in code or configuration files.

**Benefits**
- Centralized secret management
- Easy secret rotation
- Full audit logging

---

## 7. Storage Layer – Azure Storage Account
**Usage**
- Application storage
- Files and artifacts
- Logs (where required)

**Security**
- Accessible only from backend services.
- Restricted via firewall rules and trusted Azure services.

---

## 8. CI/CD & DevOps Flow

### Code Lifecycle
1. Developer pushes code to Azure DevOps repositories.
2. CI pipeline triggers automatically:
   - Build
   - Test
3. CD pipeline:
   - Requires manual approval
   - Deploys to the respective App Services

### Supported Technologies
- React
- Angular
- .NET Core
- Java

**Benefits**
- Controlled deployments
- Reduced risk of accidental production releases
- Full deployment audit trail

---

## 9. Network & Access Control Philosophy
**Core Principles**
- No direct internet access to databases
- No hardcoded secrets
- Least privilege access
- Clear separation between frontend, backend, and data layers

**Controls Used**
- Private Endpoints
- Firewall rules
- Azure Trusted Services
- API gateway isolation

---

## 10. End-to-End Request Flow (Summary)
1. User accesses application domain.
2. DNS resolves the request to Azure App Service.
3. Frontend UI loads in the browser.
4. UI sends API requests through APIM.
5. Backend services:
   - Retrieve secrets from Key Vault
   - Perform database operations
6. Response is returned to the user.

---

## 11. Architectural Benefits
- Secure by design
- Scalable and modular
- Cost-optimized App Service usage
- Strong separation of concerns
- Audit and compliance ready
