
# SonarQube Quality Gate Update – Restructuring Projects

## Objective

Apply a custom Quality Gate for Restructuring projects to reduce the **minimum code coverage requirement to 10%** while keeping other quality validations unchanged.

---

## Steps Performed

### 1. Reviewed Existing Quality Gates

Opened:

```text
SonarQube → Quality Gates
```

Reviewed existing Quality Gate configurations and identified applicable quality conditions.

---

### 2. Created New Quality Gate

Created:

```text
Restructuring-Gate
```

Configured conditions:

| Metric                     | Condition |
| -------------------------- | --------- |
| Issues                     | Must be 0 |
| Security Hotspots Reviewed | 100%      |
| Coverage                   | ≥ 10%     |
| Duplicated Lines           | ≤ 3%      |

Only the **Coverage threshold** was customized as per requirement.

---

### 3. Assigned Gate to Restructuring Projects

Associated **Restructuring-Gate** with applicable restructuring repositories/projects **excluding Common Services and Batch Processing projects**.

(Reference screenshots attached)

---

## Conclusion

Implemented a dedicated **Restructuring-Gate** for restructuring repositories with **minimum code coverage set to 10%**, while preserving existing quality validation conditions.

Common Services and Batch Processing projects were excluded from this configuration.

