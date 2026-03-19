# EKS Observability Architecture – Per Environment Deployment

## 1. Objective

The objective of this design is to implement observability within each Amazon EKS cluster (Production and Development) instead of using a centralized observability VPC. This approach is driven by cost optimization, simplicity, and environment isolation.

---

## 2. Architecture Overview

In the proposed architecture:

- Observability is deployed **within each EKS cluster**
- Separate stacks are maintained for:
  - Production environment
  - Development environment
- No centralized observability VPC is used
- No Transit Gateway is deployed

### Environment Mapping:

| Environment | VPC | EKS | Observability |
|------------|-----|-----|--------------|
| Production | Prod VPC | Prod EKS | Inside Prod EKS |
| Development | Dev VPC | Dev EKS | Inside Dev EKS |

---

## 3. Design Decision

### Removed Components:
- Centralized Observability VPC
- Transit Gateway (TGW)

### Reason:
- Reduce infrastructure cost
- Simplify network architecture
- Avoid cross-VPC data transfer cost
- Lower operational overhead

---

## 4. Observability Stack

Each EKS cluster will include the following components:

### 4.1 Metrics Monitoring
- Prometheus
- Kube-state-metrics
- Node Exporter

### 4.2 Visualization
- Grafana dashboards for:
  - Cluster health
  - Node and pod metrics
  - Application performance

### 4.3 Logging

Two possible approaches:

**Option A (Recommended - Lightweight):**
- Loki + Promtail

**Option B (Advanced):**
- ELK Stack (Elasticsearch, Logstash, Kibana)

### 4.4 Tracing (Optional)
- Jaeger or Grafana Tempo

---

## 5. Deployment Approach

- Deployment via Helm charts
- Dedicated namespace: `monitoring`
- Same structure in both environments for consistency

---

## 6. Benefits of the Proposed Approach

### 6.1 Cost Optimization
- No Transit Gateway cost
- No additional VPC cost
- Reduced inter-VPC data transfer charges

### 6.2 Simplicity
- No cross-VPC networking complexity
- Easier deployment and troubleshooting
- Fewer components to manage

### 6.3 Environment Isolation
- Production and Development are fully isolated
- Failures or spikes in one environment do not impact the other

### 6.4 Easier Operations
- Observability stack is co-located with the application
- Faster debugging and root cause analysis
- Reduced latency in metric and log collection

### 6.5 Flexible Retention Management
- Different retention policies per environment:
  - Production: Higher retention (e.g., 15–30 days)
  - Development: Lower retention (e.g., 3–7 days)
- Helps control storage cost effectively

### 6.6 Single Cluster Ownership
- Each team manages observability within its own EKS
- No dependency on centralized platform team

---

## 7. Limitations of the Approach

### 7.1 No Centralized Visibility
- No single pane of glass across environments
- Requires switching between clusters for monitoring

### 7.2 Duplicate Setup
- Observability stack deployed in each cluster
- Increased resource usage compared to shared model

### 7.3 Scaling Pressure per Cluster
- Monitoring load increases with application scale:
  - More pods → more metrics
  - More traffic → more logs
- Each cluster must independently handle its observability load

### 7.4 Operational Overhead at Scale
- Managing multiple Prometheus/Grafana instances
- Version consistency across environments

---

## 8. Scaling Considerations

As application traffic grows:

- Increase in:
  - Metrics volume
  - Log ingestion rate
  - Query load on Grafana

### Required Actions:
- Scale Prometheus resources (CPU/Memory)
- Increase storage capacity
- Optimize retention policies
- Use distributed modes if required:
  - Prometheus with Thanos/Cortex
  - Loki distributed setup

---

## 9. Future Centralization Strategy

Although the current architecture is decentralized, centralization is still possible in the future.

### Challenge
- Transit Gateway is not present
- No direct cross-VPC connectivity

### Possible Solutions

#### Option 1: VPC Peering
- Connect VPCs for observability data sharing
- Suitable for limited number of VPCs

#### Option 2: Reintroduce Central Layer
- Add centralized observability later if scale justifies cost

#### Option 3: Secure Public Endpoints
- Expose services securely (less preferred)

---

## 10. Recommended Approach

### Phase 1 (Current State)
- Observability inside each EKS cluster
- Optimized for cost and simplicity

### Phase 2 (Future - If Needed)
- Introduce central aggregation layer
- Enable cross-cluster monitoring if required

---

## 11. Trade-off Summary

| Factor | Current Approach |
|------|----------------|
| Cost | Low |
| Simplicity | High |
| Isolation | Strong |
| Central Visibility | Limited |
| Scalability | Requires per-cluster scaling |
| Future Centralization | Possible with additional setup |

---

## 12. Conclusion

The proposed architecture prioritizes cost optimization, operational simplicity, and environment isolation by deploying observability within each EKS cluster.

While centralized observability is not part of the current design, the architecture allows future enhancements if required. This ensures that the system remains flexible and scalable without introducing unnecessary upfront cost.
