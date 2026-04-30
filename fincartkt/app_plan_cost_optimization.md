# Auto-Scaling Strategy for App Service Plans (UAT, Stage, Production)

## 1. Objective
This document defines the auto-scaling strategy for App Service Plans across UAT, Stage, and Production environments in Microsoft Azure. It identifies which plans support autoscaling and provides recommended configurations.

---

## 2. Key Constraint

Auto-scaling is supported only on higher tiers:

| Pricing Tier | Autoscale Support |
|-------------|------------------|
| Basic (B1/B2/B3) | ❌ Not Supported |
| Standard (S1+) | ✅ Supported |
| Premium (P1v3/Pv3) | ✅ Supported |

---

# 3. Environment-wise Plan Assessment

---

## 3.1 UAT Environment

| App Service Plan | Tier | Autoscale | Recommended Action |
|------------------|------|----------|-------------------|
| fincart-uat-linux-app-plan-001 | Premium V3 | ✅ Yes | Enable autoscale |
| fincart-uat-linux-app-plan-002 | Basic B3 | ❌ No | Keep as is / Upgrade if needed |
| fincart-uat-linux-app-plan-003 | Basic B3 | ❌ No | Keep as is / Upgrade if needed |

### UAT Strategy
- Enable autoscale only on **one Premium plan**
- Keep scaling minimal to control cost
- Avoid enabling autoscale across all plans

### Recommended Configuration
- Min Instances: 1  
- Max Instances: 2  
- Scale Out: CPU > 70% (10 min)  
- Scale In: CPU < 30% (10 min)  

---

## 3.2 Stage Environment

| App Service Plan | Tier | Autoscale | Recommended Action |
|------------------|------|----------|-------------------|
| fincart-stage-linux-app-plan-001 | Premium V3 | ✅ Yes | Enable autoscale |
| fincart-stage-linux-app-plan-002 | Premium V3 | ✅ Yes | Enable autoscale |
| fincart-stage-windows-app-plan-001 | Basic | ❌ No | Upgrade if scaling required |
| fincart-stage-wordpress-app-plan-001 | Basic | ❌ No | Keep as is |

### Stage Strategy
- Enable autoscale on **Premium plans only**
- Use moderate scaling (testing workload)
- Combine with **schedule-based scaling**

### Recommended Configuration
- Min Instances: 1  
- Max Instances: 2–3  
- Scale Out: CPU > 60% (5–10 min)  
- Scale In: CPU < 30% (10–15 min)  
- Optional: Scale down after office hours  

---

## 3.3 Production Environment

| App Service Plan | Tier | Autoscale | Recommended Action |
|------------------|------|----------|-------------------|
| fincart-prod-linux-app-plan-002 | Premium V3 | ✅ Yes | Enable autoscale |
| fincart-prod-linux-app-plan-003 | Premium V3 | ✅ Yes | Enable autoscale |
| fincart-prod-windows-app-plan-002 | Premium V3 | ✅ Yes | Enable autoscale |

### Production Strategy
- Autoscale is **mandatory**
- Ensure high availability and performance
- Use multiple scaling metrics

### Recommended Configuration
- Min Instances: 2  
- Max Instances: 5–10  
- Scale Out:  
  - CPU > 60% (5 min) → +1 instance  
  - CPU > 75% (5 min) → +2 instances  
- Scale In: CPU < 35% (15 min)  

---

# 4. Best Practices

## 4.1 General
- Autoscaling applies to the **entire App Service Plan**, not individual apps  
- Configure **cooldown period (5–10 mins)** to avoid rapid scaling  
- Monitor performance using Application Insights  

---

## 4.2 Cost Optimization
- **UAT** → Minimal scaling  
- **Stage** → Controlled scaling  
- **Production** → Performance-first approach  

---

## 4.3 Architecture Recommendations
- Consolidate apps into fewer scalable plans where feasible  
- Separate high-load and low-load applications  
- Avoid unnecessary Premium plans in UAT  

---

# 5. Risks & Considerations

| Risk | Mitigation |
|------|-----------|
| Single instance in Production | Maintain minimum 2 instances |
| Autoscale not triggering | Validate metrics and thresholds |
| High cost due to scaling | Set appropriate max limits |
| Frequent scale-in/out | Configure proper cooldown |

---

# 6. Summary

| Environment | Autoscale Approach |
|------------|------------------|
| UAT | Limited (single Premium plan) |
| Stage | Moderate (Premium plans only) |
| Production | Mandatory (all plans) |

---

# 7. Next Steps
- Enable autoscale on identified Premium plans  
- Upgrade Basic plans where scaling is required  
- Monitor usage and fine-tune thresholds  
- Review plan consolidation opportunities  
