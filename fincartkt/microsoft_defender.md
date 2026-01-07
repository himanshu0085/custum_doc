# 🔐 Microsoft Defender for Cloud
## Remediation Runbook + Secure Score 80% Roadmap + Ownership Mapping

---

## 1️⃣ STEP-BY-STEP REMEDIATION RUNBOOK

> Ye runbook incident nahi, balki posture hardening ke liye hai.

### 🔴 Priority 1 – Remediate Vulnerabilities (+14%)
**Scope:**  
- AKS  
- VM Scale Sets  
- Virtual Machines  

**Owner:** Security + Infra

**Steps:**  
1. Azure Portal → Defender for Cloud → Environment Settings  
2. Enable Defender plans:  
   - Defender for Servers  
   - Defender for Containers  
3. Enable Vulnerability Assessment:  
   - For VMs: Defender VA  
   - For AKS: Container image scanning  
4. Verify findings under: Defender → Recommendations → Vulnerabilities  
5. Create Jira tickets per critical finding  

**Validation:**  
- Vulnerability recommendation = Healthy  
- Secure score improves immediately

---

### 🔴 Priority 2 – Manage Access & Permissions (+8%)
**Scope:**  
- Azure AD identities  
- Service principals  
- Managed identities  

**Owner:** Security

**Steps:**  
1. Defender for Cloud → Permissions Management (CIEM)  
2. Identify:  
   - Over-privileged identities  
   - Unused identities  
3. Apply:  
   - Least privilege RBAC  
   - Remove Owner roles where not needed  
4. Enable:  
   - MFA for privileged roles  
   - PIM for just-in-time access  

**Validation:**  
- “Manage access & permissions” = Healthy

---

### 🟠 Priority 3 – Enable Enhanced Security Features
**Scope:** SQL, App Services, Storage, Key Vault, AKS  
**Owner:** Security

**Steps:**  
1. Defender → Environment Settings  
2. Enable:  
   - Defender for SQL  
   - Defender for App Service  
   - Defender for Storage  
   - Defender for Key Vault  
3. Review cost impact before enabling PROD  

**Validation:**  
- Defender plans = Enabled  
- Related recommendations disappear

---

### 🟠 Priority 4 – Protect Applications Against DDoS (+5%)
**Scope:** Internet-facing VNets, App Gateways / Front Door  
**Owner:** Infra

**Steps:**  
1. Azure Portal → DDoS Protection Plans  
2. Create DDoS Standard plan  
3. Attach to:  
   - fc-vnet-prod  
   - Public-facing VNets  
4. Validate protection status  

**Validation:**  
- DDoS recommendation = Healthy

---

### 🟡 Priority 5 – Enable Auditing & Logging (+2%)
**Scope:** App Services, SQL, Storage, Event Hubs  
**Owner:** DevOps

**Steps:**  
1. Enable Diagnostic Settings:  
   - Logs → Log Analytics  
   - Metrics → Log Analytics  
2. Standardize workspace usage  
3. Validate logs ingestion  

**Validation:**  
- Logging recommendation = Healthy

---

### 🟡 Priority 6 – Remediate Security Configurations (+4%)
**Scope:** App Services, Databases, Storage  
**Owner:** DevOps

**Steps:**  
1. Defender → Recommendations → Security configurations  
2. Apply:  
   - HTTPS only  
   - Secure transfer required  
   - Firewall restrictions  
3. Re-evaluate resource health

---

## 2️⃣ SECURE SCORE → 80% ROADMAP

**📈 Current State:**  
- Secure Score: 51%  
- Risk Type: Configuration gaps  

**🎯 Target:**  
- Secure Score: 80%+  
- Timeline: 4–6 weeks  

**🗓 Roadmap:**  

| Week | Activity | Expected Secure Score |
|------|----------|--------------------|
| 1–2 | Enable Defender plans + Enable vulnerability scanning | ~65% |
| 3 | IAM cleanup (CIEM) + DDoS Standard | ~72% |
| 4 | Logging & security configs | 80%+ |

**KPI Tracking:**  
- Secure Score %  
- Number of unhealthy resources  
- Vulnerabilities count

---

## 3️⃣ RECOMMENDATION → OWNER MAPPING

| Recommendation | Owner |
|----------------|-------|
| Remediate vulnerabilities | Security + Infra |
| Manage access & permissions | Security |
| Enable enhanced security features | Security |
| Secure management ports | Infra |
| Apply system updates | Infra |
| Encrypt data in transit | DevOps |
| Enable encryption at rest | DevOps |
| Restrict network access | Infra |
| Protect against DDoS | Infra |
| Enable auditing & logging | DevOps |
| Endpoint protection | Infra |
| Implement best practices | DevOps |

---

## 4️⃣ EXECUTIVE SUMMARY

> “The Secure Score improvement roadmap focuses on enabling Defender plans, vulnerability management, IAM hygiene, and network protection. No active threats exist; improvements are preventive and posture-driven.”
