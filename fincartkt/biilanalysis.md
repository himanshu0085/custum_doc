
# Billing Cost Increase – Explanation

---

## Overview

The purpose of this document is to explain the **month-on-month increase in Azure billing costs**, identify the services responsible for the change, and clearly demonstrate how the total billing difference has been calculated.

This analysis compares billing data for the period **6 Nov – 5 Dec** with **6 Dec – 5 Jan**.

---

## 1. Billing Summary Comparison

| Particulars      | 6 Nov – 5 Dec (₹) | 6 Dec – 5 Jan (₹) | Difference (₹) |
| ---------------- | ----------------- | ----------------- | -------------- |
| Charges          | 6,44,376.95       | 6,78,676.22       | **+34,299.27** |
| Credits          | 0.00              | 0.00              | 0              |
| Tax              | 1,17,908.80       | 1,22,174.25       | **+4,265.45**  |
| **Total Amount** | **7,62,285.75**   | **8,00,850.47**   | **+38,564.72** |

**Summary:**
Overall, the Azure bill increased by approximately **₹38.5K**. The tax increase is proportional to the rise in base charges.

---

## 2. Service-wise Cost Increase

| Service Name      | Last Month (₹) | Current Month (₹) | Increase (₹)  |
| ----------------- | -------------- | ----------------- | ------------- |
| fincartstorage    | 28,702         | 38,147            | **+9,445**    |
| fincartstorageind | 28,368         | 34,222            | **+5,854**    |
| One Azure         | 13,580.34      | 19,478.44         | **+5,898.10** |
| uatfincart        | 53,285         | 58,361            | **+5,076**    |
| SSL               | NA             | 11,600            | **+11,600**   |
| Tax               | 1,17,908.80    | 1,22,174.25       | **+4,265**    |

**Observation:**
The cost increase is mainly driven by **storage usage**, **compute usage**, and **new SSL additions**, with storage-related charges being the largest contributor.

---

## 3. Storage Usage Change (Reference Data)

### fincartstorageind

| Period        | Ingress | Egress  |
| ------------- | ------- | ------- |
| 6 Nov – 5 Dec | 1.2 TiB | 3.5 GiB |
| 6 Dec – 5 Jan | 1.5 TiB | 5.6 GiB |

Ingress increased by **0.3 TiB**, increasing the stored data footprint.
Although egress remained minimal, the higher retained data resulted in increased storage capacity charges.

---

### fincartstorage

| Period        | Ingress | Egress  |
| ------------- | ------- | ------- |
| 6 Nov – 5 Dec | 1.2 TiB | 2.3 TiB |
| 6 Dec – 5 Jan | 1.6 TiB | 3.2 TiB |

**Changes observed:**

* **Ingress increase:** 0.4 TiB (higher storage capacity usage)
* **Egress increase:** 0.9 TiB (**921.6 GB**), directly impacting bandwidth costs

This storage account is the **primary contributor** to storage-related cost increases.

---

## 4. Base Charges Increase – Exact Cost Breakdown

### A. Storage Cost Impact

| Cost Component                            | Calculation / Basis                        | Increase (₹) |
| ----------------------------------------- | ------------------------------------------ | ------------ |
| Blob storage capacity – fincartstorage    | Increase in average stored data (GB-month) | **9,445**    |
| Blob storage capacity – fincartstorageind | Increase in storage footprint (GB-month)   | **5,854**    |
| Blob data egress (Bandwidth)              | 0.9 TiB = 921.6 GB × ₹4.96 per GB          | **4,570**    |
| **Subtotal – Storage Impact**             |                                            | **19,869**   |

---

### How these storage costs were calculated

* **Ingress (data upload) is free in Azure** and does not incur direct charges.
* Increased ingress leads to a higher **average stored data size**, billed as **GB-month**.
* **fincartstorage:** ₹9,445 increase due to higher retained data.
* **fincartstorageind:** ₹5,854 increase due to increased storage footprint.

#### Blob Data Egress (Bandwidth) Calculation

* Previous egress: **2.3 TiB**
* Current egress: **3.2 TiB**
* **Increase:** 0.9 TiB

**Conversion:**

* 1 TiB = 1024 GB
* 0.9 × 1024 = **921.6 GB**

**Effective rate:** **₹4.96 per GB**

```
921.6 GB × ₹4.96 ≈ ₹4,570
```

Only **storage capacity** and **data egress** are chargeable components.

---

### B. Compute & Environment Usage

| Environment                   | Reason                                           | Increase (₹) |
| ----------------------------- | ------------------------------------------------ | ------------ |
| One Azure                     | New resources provisioned and higher utilization | **5,898**    |
| uatfincart                    | Increased workload usage                         | **5,076**    |
| **Subtotal – Compute Impact** |                                                  | **10,974**   |

---

### C. Fixed Cost Addition

| Component        | Reason                             | Increase (₹) |
| ---------------- | ---------------------------------- | ------------ |
| SSL Certificates | Addition of 2 new SSL certificates | **11,600**   |

---

## 5. Base Charges Reconciliation

| Category                       | Amount (₹)    |
| ------------------------------ | ------------- |
| Storage impact                 | 19,869        |
| Compute & environment usage    | 10,974        |
| SSL certificates               | 11,600        |
| **Total Base Charge Increase** | **34,299.27** |

✔ Matches exactly with the increase in **Charges**.

---

## 6. Tax Impact

| Component                     | Increase (₹) |
| ----------------------------- | ------------ |
| GST on increased base charges | **4,265.45** |

Tax is automatically calculated and increases proportionally with the base charges.

---

## 7. Final Executive Summary

| Reason                                      | Amount (₹)    |
| ------------------------------------------- | ------------- |
| Increased storage usage (capacity + egress) | **19,869**    |
| Increased compute & environment usage       | **10,974**    |
| New SSL certificates                        | **11,600**    |
| Tax increase                                | **4,265**     |
| **Total Billing Increase**                  | **38,564.72** |

---

## Key Clarification

Ingress does **not** directly incur any charges.
The billing increase is driven by:

* Storage capacity growth
* Data egress (bandwidth)
* Increased compute utilization
* Fixed SSL costs
* Proportional tax impact

**No Azure pricing changes were applied during this billing period.**

---
