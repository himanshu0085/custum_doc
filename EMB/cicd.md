# Complete CI/CD Architecture Assessment

**Assessment date:** 2026-09-17  
**Authorization:** Read-only  
**Repository:** Azure DevOps deployment/configuration repository  
**Refs inspected:** `main`, `origin/qa`, `origin/uat`, `origin/prod`

## 1. Executive Summary

This repository contains multiple environment-specific implementations rather than one uniform UAT pipeline. The available refs show four different operational shapes:

- `main` is predominantly UAT-oriented service deployment YAML.
- `origin/qa` uses `qa` application refs but targets DEV-named ACR/AKS resources and adds diagnostics, ingress, network policy, and rollback assets.
- `origin/uat` contains UAT deployment YAML plus a separate `deploy/azurepipeline/prod/` production pipeline family and production copies under `k8s/prod/`.
- `origin/prod` contains a distinct production-oriented pipeline family, production private ingress/certificates, network policy, diagnostics, and rollback.

The repository therefore demonstrates **branch-based environment implementations with rebuilds**, not a confirmed build-once/promote-same-artifact model. Standard service pipelines build external application code, push both a Build ID tag and `latest`, and deploy through `az aks command invoke`. Production pipeline variants add manual validation, production secret/config application, longer rollout verification, diagnostics, and health-triggered `rollout undo`, but they still build in the production pipeline and still publish/use mutable tags.

The most important conclusions are:

1. **QA is not proven to be a separate QA runtime.** Its branch is `qa`, but its visible resources are DEV-named (`adpgampdevuaencr`, `az-automotivemarketplace-dev-uaen-aks`). The intended relationship between QA and DEV is UNKNOWN.
2. **PROD is not merely UAT with different values.** It has different pipeline families, production manual approvals, production secret/config application, production-specific manifests, diagnostics, and rollback behavior.
3. **UAT contains production assets.** `origin/uat` includes `deploy/azurepipeline/prod/` and `k8s/prod/`, creating source-of-truth and branch-boundary risk.
4. **No dedicated DEV pipeline/ref was found.** DEV appears as a target in the QA ref, not as an independently governed environment implementation.
5. **STAGING is not independently represented.** Staging hostname/configuration values exist, but no dedicated STAGING pipeline, registry, cluster, or branch was confirmed.
6. **Release integrity is weak across environments.** Images are rebuilt per pipeline/environment, `latest` is used, deployment is usually by tag, and no complete digest/SBOM/signing/provenance promotion contract is visible.
7. **Security and quality controls differ by ref and are incomplete.** Tests are commonly skipped, Checkmarx is often disabled, and production has additional controls that are not consistently present in QA/UAT.

The safest modernization path is to first document and isolate the actual environments, then standardize common CI, introduce immutable artifact promotion, preserve environment-specific CD policy, and only then migrate Kubernetes delivery toward Helm or GitOps.

## 2. Scope & Evidence

### Evidence labels

- **CONFIRMED:** directly observed in a file, ref, or read-only Git diff.
- **UNKNOWN — requires Azure DevOps/Azure verification:** not determinable from repository content; no absence is claimed.
- **NOT FOUND IN REPOSITORY:** no corresponding tracked implementation was found in the inspected refs.
- **RECOMMENDATION:** proposed future state; not implemented.

The checked-out worktree is `main`; remote refs were inspected with `git show`, `git ls-tree`, and read-only diffs without changing branches. No Azure DevOps portal settings, external application repository contents, live AKS/ACR state, pipeline run history, or runtime monitoring was available. No deployment, mutating Azure CLI, `kubectl`, installation, commit, push, or implementation refactor was performed.

## 3. Complete Repository Inventory

| Area | Files/locations | Purpose | Ref coverage | Assessment |
|---|---|---|---|---|
| Documentation | `README.md` | Project/setup/build documentation | All inspected refs, with branch changes | CONFIRMED placeholder TODO content; operational documentation incomplete |
| Standard service pipelines | `deploy/azurepipeline/pipeline-actions.yaml` through `pipeline-users.yaml` | Per-service build/push/deploy | `main`, QA, UAT, PROD variants | Repeated service implementations; no common template references |
| Frontend pipeline | `deploy/azurepipeline/pipeline-frontend.yaml` | Frontend build/push/deploy | `main`, QA, UAT, PROD variants | Environment-specific variants differ materially |
| Inventory pipeline | `pipeline-inventory.yaml` | Inventory build/deploy plus reindex/diagnostics | All refs | Special operational coupling and drift |
| Secret injection | `inject-secrets.yaml`, UAT `prod/inject-secrets.yaml` | Create/update Kubernetes secrets and restart workloads | Main/QA/UAT/PROD variants | Multiple secret mechanisms; active registration UNKNOWN |
| Diagnostics | `pipeline-diagnostics.yaml`, `pipeline-diagnostics-ES.yaml` | Logs, pod/config/runtime diagnostics | QA/UAT/PROD refs | Operational tooling is branch-specific and frequently modified |
| Rollback | `rollback-prod.yml` | Production rollback/inspection | QA/UAT/PROD refs | Rollback is not present in checked-out main; branch naming is inconsistent |
| Kubernetes services | `k8s/backend/*.yaml`, `k8s/frontend/frontend.yaml` | Deployments and Services | All refs, with modifications | Direct-manifest deployment; substantial branch drift |
| Shared Kubernetes | `k8s/namespace.yaml`, `ingress.yaml`, `kind-config.yaml` | Namespace, ingress, local cluster config | All refs, with variants | Ingress and environment values differ by ref |
| Private ingress | `k8s/ingress-private.yaml` | Private/Application Gateway ingress | UAT/PROD refs | Not present in main; activation UNKNOWN |
| Certificates | `k8s/letsencrypt-prod.yaml` | Certificate/issuer resources | QA/UAT/PROD refs | Production-named certificate file appears outside PROD too |
| Configuration | `k8s/backend/configmap.yaml`, `configmap-aks.yaml` | Runtime non-secret and secret-like config | All refs | ConfigMap/Secret ownership conflicts |
| Secrets | `k8s/backend/secrets.yaml`, QA `new-secret.yaml` | Secret template or values | Branch-specific | Multiple competing secret definitions |
| Network policy | `k8s/backend/network-policy.yaml` | Pod network restrictions | QA/UAT/PROD refs; not main | Security control drift |
| Docker | `deploy/docker/backend/dockerfile`, frontend counterpart | Intended container entry points | Empty in main; branch-specific frontend content | Not active in checked-out service flow |
| Helm | `helm/backend/*`, `helm/forntend/*` | Intended chart/value placeholders | All refs | Empty, misspelled frontend directory, not referenced |
| Scripts | `k8s/aks-deploy.sh`, `aks-push-images.sh`, `deploy.sh`, plus `k8s/prod/` copies | Manual build/deploy workflows | UAT ref and its production subtree | Separate shell delivery path from Azure pipelines |
| IaC | Terraform/Bicep/ARM | Azure/platform provisioning | All inspected refs | NOT FOUND IN REPOSITORY |
| GitOps | Argo CD/Flux definitions | Reconciliation | All inspected refs | NOT FOUND IN REPOSITORY |

Files are not assumed active merely because they exist. Pipeline registration, trigger enablement, and environment association are UNKNOWN.

## 4. Branch/Ref Inventory

| Branch/ref | Commit observation | Pipelines | Environment signal | ACR/AKS signal | Kubernetes/supporting assets | Scripts | Rollback |
|---|---|---|---|---|---|---|---|
| `main` | `ac56b9a`, 2026-05-13 | 15 service + secret pipeline | Mostly UAT; one `main` trigger exception | UAT ACR/AKS values | Standard manifests, public/staging ingress, no network policy | NOT FOUND | NOT FOUND |
| `origin/qa` | `aec88a5`, 2026-09-11 | Service pipelines, diagnostics, ingress, secret, rollback | `qa` refs but DEV-named runtime | `adpgampdevuaencr`; `az-automotivemarketplace-dev-uaen-aks`; `subs-maritimecluster-nonprod` | Network policy, `new-secret`, disabled/public ingress, cert file | NOT FOUND | `rollback-prod.yml` present |
| `origin/uat` | `6a6e2a9`, 2026-07-08 | UAT service/diagnostic/private ingress plus nested `prod/` pipelines | UAT top level; PROD subtree | UAT top-level; production values in nested subtree | UAT and `k8s/prod/` copies, network policy, private ingress, cert | `k8s/*.sh` and `k8s/prod/*.sh` | `rollback-prod.yml` present |
| `origin/prod` | `270c535`, 2026-09-02 | Production-oriented service/diagnostic/private ingress/secret/rollback | Production | `subs-maritimecluster-prod`, production resource names in pipeline | Private ingress, certs, network policy, modified production manifests | NOT FOUND | `rollback-prod.yml` present |

**CONFIRMED:** all four refs have materially different file content.  
**UNKNOWN:** which ref is authoritative for each registered Azure DevOps pipeline and whether older files remain active.

## 5. Environment Inventory

| Environment | Trigger branch | Application repo/ref | Pipeline/service connection | Subscription/resource/ACR/AKS | Namespace | Image selector | Config/secret | Ingress/policy/Helm/GitOps | Approval/rollback/smoke |
|---|---|---|---|---|---|---|---|---|---|
| DEV | NOT FOUND as dedicated environment | NOT FOUND as dedicated environment | NOT FOUND as dedicated environment | DEV-named target appears from QA ref: `adpgampdevuaencr`, DEV AKS, nonprod connection | `adp` in QA-target manifests | Usually `latest` plus Build ID push | QA-ref ConfigMaps/Secrets; exact active source UNKNOWN | QA-ref ingress/network policy; Helm/GitOps NOT FOUND | Rollback file exists on QA ref; portal approval UNKNOWN; smoke not formalized |
| QA | `qa` in `origin/qa` service pipelines | BE/FE refs `qa` in QA pipelines | `subs-maritimecluster-nonprod` | `adpgampdevuaencr`; `az-automotivemarketplace-dev-uaen-aks` | `adp` | Build ID and `latest`; digest not shown | `configmap*`, `secrets.yaml`, `new-secret.yaml`, injection pipeline | Network policy and ingress variants; Helm/GitOps NOT FOUND | Manual validation appears in some QA-ref pipelines; rollback file present; formal smoke UNKNOWN |
| UAT | `uat` in main/UAT service pipelines | BE/FE refs `uat` | `subs-adports-prod` label in UAT files; exact scope UNKNOWN | `adpgampuatuaencr`, UAT resource group/AKS | `adp` | Build ID and `latest`; digest not shown | UAT ConfigMaps, Secret template, injection | Public/private ingress variants; network policy; Helm/GitOps NOT FOUND | No manual validation in simple UAT service pipelines; rollback file on UAT ref; smoke mostly ad hoc |
| PROD | Production pipeline variants are triggerless/manual-looking in files; registration UNKNOWN | Production pipelines use BE/FE `prod` refs | `subs-maritimecluster-prod` in `origin/prod`; nested UAT `prod/` files also use prod-oriented values | Production-named subscription/resource/AKS/ACR variables; exact values vary by file | `adp` | Build ID and `latest`; production deployment may patch manifest to Build ID | Production config/secret application from variable groups/inline values | Private ingress, TLS/cert, network policy; Helm/GitOps NOT FOUND | `ManualValidation`, health/auto rollback in production variants; smoke/diagnostics partly ad hoc |
| STAGING | NOT FOUND as dedicated branch/pipeline | NOT FOUND as dedicated application ref | NOT FOUND | NOT FOUND as dedicated registry/cluster | NOT FOUND | NOT FOUND | Staging hostname/config values exist | Staging host appears in ingress/frontend values | NOT FOUND as independent controls |

This matrix intentionally separates confirmed evidence from missing or unknown runtime configuration. QA’s DEV-named resources do not prove that QA and DEV are the same environment; that relationship requires Azure verification.

## 6. Current-State Architecture

### `main` / UAT-oriented path

```text
Deployment repo branch/path trigger
  -> service YAML
  -> self checkout + external BE/FE checkout at uat
  -> Java 21 + Maven package (-DskipTests)
  -> inline/external Dockerfile
  -> az acr build to UAT-named ACR (BuildId + latest)
  -> az aks command invoke against UAT-named AKS
  -> kubectl apply one manifest + rollout restart/status
```

### QA ref path

```text
qa branch/path trigger
  -> service YAML on origin/qa
  -> self checkout + external BE/FE checkout at qa
  -> Java 21 + Maven package (tests commonly skipped)
  -> ACR build to DEV-named ACR
  -> az aks command invoke against DEV-named AKS
  -> manifest/config/policy application and rollout/diagnostic commands
```

### PROD path

```text
manual/registered production pipeline (trigger status UNKNOWN)
  -> external BE/FE checkout at prod
  -> Java 21 + Maven package (tests commonly skipped)
  -> generated Dockerfile + ACR build (BuildId + latest)
  -> ManualValidation production approval
  -> production ConfigMap + variable-group Secret creation
  -> patch/apply production manifest and rollout status (300s)
  -> pod health/diagnostics; some variants rollout undo on failure
```

### Other path

UAT ref shell scripts provide an independent manual flow: local Docker build/push of all images using `latest`, in-place `sed` image patching, namespace/config/infrastructure apply, discovery-first ordering, backend/frontend apply, ingress apply, and pod/ingress listing.

## 7. QA CI/CD Flow

**CONFIRMED from `origin/qa`:** service pipelines trigger from `qa` and check out external application repositories at `refs/heads/qa`. Representative service files define `adpgampdevuaencr`, `az-emb-dev-np-uaen-automarketplace-aks-rg`, `az-automotivemarketplace-dev-uaen-aks`, and `subs-maritimecluster-nonprod`. The common sequence is Java 21, Maven module build, usually `-DskipTests`, optional/disabled Checkmarx, ACR build with Build ID and `latest`, remote manifest apply, rollout/status, and extra diagnostics in selected variants.

**CONFIRMED:** QA ref adds `pipeline-diagnostics.yaml`, `pipeline-diagnostics-ES.yaml`, `pipeline-ingress.yaml`, `network-policy.yaml`, `new-secret.yaml`, `letsencrypt-prod.yaml`, and `rollback-prod.yml`. Frontend and inventory pipelines are materially more diagnostic-heavy than the simple service files. Some commands use `kubectl get`, logs, annotations, and best-effort commands.

**UNKNOWN:** whether QA means the DEV-named AKS environment, whether these files are registered, whether approvals are portal-configured, and whether QA images are reused by UAT. No artifact promotion evidence was found.

**Assessment:** QA is a distinct branch implementation but not a clearly distinct infrastructure environment in repository naming. It appears to be a QA-to-DEV-targeted rebuild path, which must be clarified before standardization.

## 8. UAT CI/CD Flow

**CONFIRMED from `main` and top-level `origin/uat`:** most service pipelines trigger from `uat`, checkout BE/FE `uat`, use UAT-named ACR/resource group/AKS values, build Java 21 Maven modules, skip tests, generally have Checkmarx disabled, push Build ID and `latest`, apply one manifest using `az aks command invoke`, and restart/wait for rollout. `communication` in checked-out main triggers from `main`; `inventory` in main has a `main`/`uat` trigger exception. These are branch-specific confirmed mismatches, not generalized UAT behavior.

**CONFIRMED:** top-level UAT ref adds private ingress, network policy, diagnostics, rollback, and shell deployment scripts. It also contains a nested `deploy/azurepipeline/prod/` pipeline set and `k8s/prod/` tree, which is production material physically stored under the UAT ref.

**UNKNOWN:** whether top-level UAT and nested production pipelines are active simultaneously, whether UAT has environment approvals, and whether UAT is a release gate for PROD.

## 9. PROD CI/CD Flow

**CONFIRMED from `origin/prod`:** production service pipelines use production-oriented service connection text (`subs-maritimecluster-prod`), external repository refs `prod`, production variables, and production manifests. Representative production pipelines include Java 21, Maven `clean package` with `-DskipTests`, optional/disabled Checkmarx, generated Dockerfile, ACR build with Build ID and `latest`, `ManualValidation@0` with a 1440-minute timeout, production ConfigMap/Secret application, production manifest application, and rollout status with a 300-second timeout. Frontend and selected service variants include extensive diagnostics and runtime fixes.

**CONFIRMED from production pipeline content:** production secret application creates `adp-secrets` from pipeline variables such as JWT, Redis, Mongo, Service Bus, ACS, Blob, and Elasticsearch values. The deployment may patch a production manifest from a mutable image reference to `$(Build.BuildId)`. Health checks inspect pods for `CrashLoopBackOff`, `Error`, or `OOMKilled` and some variants execute `kubectl rollout undo` on failure.

**CONFIRMED from `origin/uat`:** a second production family exists under `deploy/azurepipeline/prod/`, with parameterized `SERVICE_NAME`, `MAVEN_MODULE`, `CONTAINER_PORT`, production approval, production secret/config application, production manifest deployment, and 300-second rollout/health logic. It is not safe to treat this as identical to `origin/prod` because the files, variables, diagnostics, and manifest paths differ.

**UNKNOWN:** production pipeline registration, actual production ACR/AKS IDs, approval enforcement outside `ManualValidation`, current live image digests, and whether either production family is authoritative.

## 10. Other Environment Flows

### DEV

**NOT FOUND IN REPOSITORY:** no dedicated DEV branch/ref, DEV-only pipeline family, DEV-specific application checkout contract, or independent DEV release/approval model was found.  
**CONFIRMED:** DEV-named ACR/AKS values are used by `origin/qa`.  
**UNKNOWN:** whether that is the intended DEV environment or merely legacy naming.

### STAGING

**NOT FOUND IN REPOSITORY:** no dedicated STAGING pipeline, branch, ACR, AKS, namespace, or promotion flow was found.  
**CONFIRMED:** `stg.globalautotrade.com` appears in ingress/frontend configuration and build arguments.  
**UNKNOWN:** whether staging is an alias for UAT or a separately managed runtime.

### Diagnostics and operational flows

**CONFIRMED:** diagnostics pipelines exist only in QA/UAT/PROD refs and contain direct pod/log/config inspection. These are operational workflows, not proof of a formal environment promotion gate.

## 11. Pipeline Inventory

| Family | QA (`origin/qa`) | UAT (`main`/`origin/uat`) | PROD (`origin/prod` and UAT `prod/`) | Material difference |
|---|---|---|---|---|
| Service triggers | `qa` path filters | Mostly `uat`; main/inventory exceptions in main | Often triggerless/manual-looking; registration UNKNOWN | Branch and trigger policy differ |
| Application ref | BE/FE `qa` | BE/FE `uat` | BE/FE `prod` | Rebuild from different branch per environment |
| Java/Maven | Java 21; module build; tests commonly skipped | Java 21; module build; tests skipped | Java 21; module build; tests skipped in inspected production variants | No evidence of environment-specific runtime version, but gate behavior differs |
| Security | Checkmarx present/disabled in common services; diagnostics added | Checkmarx present/disabled in common services | Checkmarx present/disabled in production variants; manual approval added | Security consistency is not established |
| Registry | DEV-named ACR | UAT-named ACR | Production-named variables/ACR | Environment resource split is visible but exact IDs vary |
| Image tags | Build ID + `latest` | Build ID + `latest` | Build ID + `latest`, sometimes patched into manifest | No digest promotion |
| Deployment | Remote `kubectl`; diagnostics/policy variants | Remote `kubectl`; service restart/status | Remote `kubectl`; secrets/config, approval, 300s health/rollback | PROD has stronger operational sequence but more inline coupling |
| Rollback | File present; actual registration UNKNOWN | File present in UAT ref; simple UAT files lack rollback | `rollout undo` in some pipelines and rollback file | Rollback is not a single standard |
| Smoke/verification | Diagnostics and pod checks; no formal common smoke gate | Rollout status; inventory ad hoc reindex or diagnostics | Pod health and manual test instructions in some files | Verification is inconsistent |
| Helm/GitOps | NOT FOUND | NOT FOUND | NOT FOUND | No common deployment abstraction |

The 14 common backend pipelines are structurally copied within each ref, but cross-ref files are not identical and must not be collapsed into one assumed implementation.

## 12. Cross-Environment Comparison

| Area | QA | UAT | PROD | Difference and risk |
|---|---|---|---|---|
| Source branch | `qa` | `uat` | `prod` | Branch-based rebuilds can produce different bytes |
| Runtime naming | DEV-named | UAT-named | Production-named | QA/DEV ownership is ambiguous |
| Trigger | CI `qa` | CI `uat` and exceptions | Manual/triggerless-looking | Promotion boundary is inconsistent |
| Tests | Commonly skipped | Common Maven options skip tests | Inspected production variants skip tests | No environment has a consistently visible test gate |
| SAST | Checkmarx present/disabled in common files | Same | Present/disabled variants | Security drift and false assurance |
| Approval | Some manual validation evidence in QA ref; registration UNKNOWN | No common service approval in main | Explicit production ManualValidation in variants | PROD has stronger human gate, but is duplicated |
| Artifact | Build ID + `latest` | Build ID + `latest` | Build ID + `latest`, sometimes patched into manifest | No build-once/promotion evidence |
| Secrets | Multiple QA files (`new-secret`, secrets, injection) | ConfigMap + template/injection | Variable-driven production Secret creation | Secret ownership differs by environment |
| Ingress | Public/disabled/private variants | Public/private variants | Private ingress/TLS | Environment exposure and certificate behavior drift |
| Network policy | Present in ref | Present in UAT ref | Present in PROD ref | Missing from main and branch activation UNKNOWN |
| Rollback | File and selected logic | File, top-level service flow weak | Health-triggered undo in some files | Rollback reliability varies |
| Diagnostics | Added extensively | Added extensively | Extensive, sometimes embedded in release pipelines | Operational debugging is coupled to individual pipelines |
| Helm/GitOps | NOT FOUND | NOT FOUND | NOT FOUND | No common deployment abstraction |

## 13. Environment Drift

| File/ref | Environment | Current behavior | Risk | Recommendation |
|---|---|---|---|---|
| QA service pipelines, `origin/qa` | QA/DEV | `qa` source deploys to DEV-named ACR/AKS | QA may not represent a separate QA environment | Confirm and document DEV/QA mapping before changes |
| Main `pipeline-communication.yaml` | UAT/main | `main` trigger with UAT checkout/target | Main changes can deploy to UAT unexpectedly | Explicit trigger-to-environment contract and approval |
| Main `pipeline-inventory.yaml` | UAT/main | Both `main` and `uat` trigger UAT deployment | Cross-environment accidental deployment | Separate CI from CD and restrict deployment refs |
| `origin/uat/deploy/azurepipeline/prod/*` | UAT/PROD | Production pipeline family stored under UAT ref | Wrong branch can expose production delivery assets | Establish one production source of truth |
| `origin/qa` vs `main` | QA/UAT | Different service connections, ACR, AKS, refs, diagnostics | Fixes and controls diverge | Compare through a generated environment contract, not copied files |
| `origin/prod` vs UAT `prod/` | PROD | Two production pipeline families with differing paths/logic | Unknown authoritative production behavior | Inventory registrations and retire/govern one family |
| `k8s/backend/network-policy.yaml` | QA/UAT/PROD | Present in remote environment refs, absent in main | Security posture depends on ref | Make policy part of a tested release contract |
| `k8s/ingress-private.yaml` / certs | UAT/PROD | Private ingress/cert assets differ by branch | Traffic exposure and TLS behavior drift | Define environment ingress ownership and validation |
| `secrets.yaml`, `new-secret.yaml`, inline secret creation | All | Multiple secret schemas and injection styles | Partial or conflicting runtime configuration | One secret store/schema with environment-specific references |

## 14. Source-of-Truth Analysis

| Configuration | Current source of truth | Other copies | Drift risk |
|---|---|---|---|
| Application code | External BE/FE repositories at branch refs | Deployment pipeline resource declaration | Pipeline may build moving branch content; source commit not packaged |
| Service build behavior | Each service pipeline YAML and external Dockerfile/module | Inline generated Dockerfiles, shell scripts | Different build contexts and runtime behavior |
| Environment target | Inline pipeline variables | Manifest image/host/config values, branch names, service connection names | Trigger may not match target |
| Image selector | Kubernetes manifest or pipeline `sed` patch | ACR `latest` and Build ID tags | Tag/digest mismatch and mutable deployment |
| Runtime config | ConfigMaps, Secrets, variable groups, inline commands | Branch-specific copies | Conflicting configuration ownership |
| Kubernetes desired state | Individual manifests applied remotely | Shell scripts, branch copies, diagnostics fixes | Partial application and branch drift |
| Ingress/TLS | `k8s/ingress*.yaml` and certificate files | Pipeline ingress files and branch copies | Exposure/certificate divergence |
| Rollback | Production pipeline inline `rollout undo`, rollback YAML, AKS revision history | Manual operator behavior | Recovery path differs by branch/service |
| Approvals | Production `ManualValidation`; portal settings UNKNOWN | Human instructions in files | Manual control is duplicated and not uniformly governed |
| Cluster/platform provisioning | NOT FOUND IN REPOSITORY | Azure/platform team or manual setup UNKNOWN | Prerequisites may be invisible to release process |

## 15. Artifact & Promotion Model

**CONFIRMED:** each environment pipeline checks out its environment branch and runs its own Maven/container build. ACR receives a Build ID tag and `latest`. Kubernetes references `latest` in many manifests; production variants sometimes patch a manifest to Build ID. No digest promotion, SBOM, provenance, signing, or Azure artifact publication is visible.

**Determination:** the current model is best classified as **C. Branch-based rebuild**, with some production pipeline-specific tag patching. It is not confirmed as A build separately per environment in every registration because portal state is UNKNOWN, but the repository implementation independently builds per branch/ref. It is not B build once/promote, D pipeline-to-pipeline promotion, or E GitOps based on repository evidence.

```text
QA branch -> QA/DEV-named rebuild -> QA/DEV-named ACR -> QA/DEV-named AKS
UAT branch -> UAT rebuild -> UAT ACR -> UAT AKS
PROD branch/pipeline -> PROD rebuild -> PROD ACR -> PROD AKS
```

**Risk:** QA-tested bytes are not demonstrably the bytes deployed to UAT or PROD; `latest` weakens traceability and rollback.  
**RECOMMENDATION:** build once from a pinned application commit, publish immutable digest plus SBOM/provenance/signature, and promote that digest through environment approvals.

## 16. Security Assessment

| Control/finding | QA | UAT | PROD | Gap/risk |
|---|---|---|---|---|
| Plaintext credential-like ConfigMap values | QA branch ConfigMap variants; exact values differ | `configmap-aks.yaml` contains Redis/JWT values | Production config/secret paths differ; inline variable use | Secrets are exposed or inconsistently represented |
| Kubernetes Secret mechanism | `secrets.yaml`, `new-secret.yaml`, injection variants | Secret template plus injection pipeline | Pipeline creates Secret from variables | No single controlled secret lifecycle |
| Key Vault/CSI/workload identity | UNKNOWN | UNKNOWN | UNKNOWN | Requires Azure/AKS verification |
| Service connection | Nonprod `subs-maritimecluster-nonprod` | `subs-adports-prod` label in UAT files | `subs-maritimecluster-prod` in prod ref; nested prod differs | Scope and least privilege UNKNOWN; names/IDs drift |
| SAST | Checkmarx present/disabled in common files | Same | Present/disabled variants | No consistently blocking SAST |
| SCA/dependency scan | NOT FOUND in repository pipeline | NOT FOUND | Threshold text may mention SCA; execution/state UNKNOWN | Dependency risk not consistently gated |
| Container/IaC/secret scanning | NOT FOUND | NOT FOUND | NOT FOUND | Supply-chain coverage missing |
| SBOM/signing/provenance | NOT FOUND | NOT FOUND | NOT FOUND | Artifact authenticity/audit gap |
| NetworkPolicy | Present in QA ref | Present in UAT ref | Present in PROD ref | Not in main; activation/order UNKNOWN |
| Ingress/TLS exposure | Multiple public/disabled variants | Public/private variants | Private ingress/TLS variants | Environment exposure is branch-dependent |
| RBAC/admission | UNKNOWN | UNKNOWN | UNKNOWN | Requires cluster verification |

### Severity findings

| ID | Severity | Evidence | Risk | Recommendation |
|---|---|---|---|---|
| SEC-01 | Critical | UAT/main `k8s/backend/configmap-aks.yaml` has `REDIS_PASSWORD` and `JWT_SECRET` in ConfigMap data | Credential theft or JWT forgery; repository history may retain values | Treat as exposed, rotate through incident/change process, remove secret material from ConfigMaps, use approved secret store |
| SEC-02 | High | QA/UAT/PROD pipelines create secrets through different mechanisms and partial restart lists | Services can run with inconsistent or stale credentials | Define one environment-specific secret contract and verify all consumers |
| SEC-03 | High | Checkmarx task is disabled in common service pipelines; scans not consistently visible across refs | Vulnerable code may pass release | Define blocking SAST/SCA/container/IaC policy with managed exceptions |
| SEC-04 | High | Inline subscription/resource/cluster/service-connection values and production secret command arguments | Cross-environment deployment or secret leakage | Scope service connections and use secure secret references/variables |
| SEC-05 | Medium | `latest` tool/image use in scripts and diagnostic pods | Mutable supply-chain input | Pin image digests and approved tool images |

## 17. Kubernetes Assessment

**CONFIRMED across refs:** service manifests use namespace `adp`; many Deployments use one replica, `imagePullPolicy: Always`, and `latest`; readiness is frequently TCP-based with long startup delays; liveness is inconsistent; resource requests/limits are repeated but not policy-enforced in repository code.

**QA:** `origin/qa` adds `network-policy.yaml`, `new-secret.yaml`, disabled/public ingress variants, and certificate files. This is more control-rich than main but may represent QA/DEV ambiguity.  
**UAT:** top-level UAT has standard and private ingress, network policy, and separate `k8s/prod` copies under the same ref.  
**PROD:** `origin/prod` has private ingress, certificate configuration, network policy, and changed production manifests; UAT’s nested production tree is a second implementation.

**NOT FOUND IN REPOSITORY:** consistent HPA, PDB, ServiceAccount/RBAC, or securityContext definitions across all refs.  
**UNKNOWN:** live replica overrides, Pod Security admission, autoscaling, PDBs applied externally, ingress controller state, and actual runtime image digests.

Differences may be intentional, such as replica count or public/private ingress, but they need an environment contract. Differences in policy presence, secret schema, probes, and image selectors appear to be implementation drift unless owners confirm otherwise.

## 18. Docker Assessment

**CONFIRMED:** main’s committed `deploy/docker/backend/dockerfile` and frontend Dockerfile are empty. Service pipelines generate Dockerfiles inline or consume Dockerfiles from the external application repository. UAT shell scripts use application-repository Dockerfiles and push `latest` locally. Production pipeline variants generate a Java 21 runtime Dockerfile and build from a broader backend context.

**Environment differences:** QA/UAT/PROD pipeline copies use different build contexts, generated Dockerfile conventions, and diagnostic/runtime fix steps. Exact external Dockerfile contents are UNKNOWN.

**Gaps:** base image digest pinning, multi-stage builds, non-root execution, labels, reproducibility, vulnerability scanning, SBOM, signature, and build-context consistency are not established by this repository.

**RECOMMENDATION:** define Dockerfile ownership per application, pin approved bases, standardize runtime/security metadata, generate SBOM/signatures, and make image digest the release interface.

## 19. Helm/GitOps Assessment

**NOT FOUND IN REPOSITORY:** active Helm charts, Argo CD, Flux, or GitOps configuration. The Helm files are empty placeholders and the frontend directory is misspelled `forntend`. No pipeline references Helm.

**CONFIRMED:** current deployment mechanisms are direct remote `kubectl` through Azure CLI and independent shell scripts in UAT refs.

**RECOMMENDATION:** consider Helm for packaging repeated service/environment defaults or GitOps for desired-state audit/reconciliation. Migration risks include live-state drift, secret integration, ordering, ingress compatibility, rollback ownership, and duplicate release controllers. Do not migrate until source-of-truth and environment ownership are resolved.

## 20. Quality Gates

| Gate | QA | UAT | PROD | Assessment |
|---|---|---|---|---|
| Unit tests | Common Maven options skip tests | Common Maven options skip tests | Inspected production variants skip tests | Missing as blocking release gate |
| Integration/contract tests | NOT FOUND in deployment repo | NOT FOUND | NOT FOUND | External application/portal UNKNOWN |
| Coverage | NOT FOUND | NOT FOUND | NOT FOUND | No threshold/publication visible |
| SAST | Checkmarx present/disabled in common files | Checkmarx present/disabled in common files | Present/disabled variants | Inconsistent/disabled |
| SCA | NOT FOUND as executable task | NOT FOUND | Threshold text may mention SCA; execution UNKNOWN | Not demonstrably blocking |
| Container/IaC/secret scan | NOT FOUND | NOT FOUND | NOT FOUND | Missing |
| Kubernetes validation | NOT FOUND | NOT FOUND | NOT FOUND | Direct apply risk |
| Rollout status | Present; selected diagnostics | Present | Present, generally 300s in production variants | Narrow availability check |
| Smoke test | No common blocking smoke | Ad hoc inventory/reindex or diagnostics | Manual scenario instructions and pod checks | Not standardized |
| Approval | Some QA-ref ManualValidation evidence; registration UNKNOWN | No common service approval in main | Explicit production ManualValidation in variants | Environment governance differs |

## 21. Rollback & Failure Handling

| Environment | Rollback mechanism | Evidence | Reliability concern |
|---|---|---|---|
| DEV/QA target | `rollback-prod.yml` exists on QA ref; selected pipeline undo/check logic | `origin/qa` rollback and diagnostic files | File name says prod; active registration and target safety UNKNOWN |
| UAT | Rollback file on UAT ref; simple top-level services mainly rollout status only | `origin/uat/deploy/azurepipeline/rollback-prod.yml` and service YAML | No uniform automatic rollback; mutable tags weaken revision identity |
| PROD | `kubectl rollout undo` in some production health checks plus rollback YAML | `origin/prod` production pipelines and UAT `prod/` pipelines | Varies by service/family; may undo to prior Deployment revision, not a verified digest |

Failure scenarios confirmed by design include image push succeeding while apply fails; apply succeeding while rollout fails; rollout success while business health fails; secrets updating only some services; and diagnostic/reindex commands suppressing errors with `|| true`. Database migration behavior is NOT FOUND IN REPOSITORY and must be assessed separately.

**RECOMMENDATION:** record prior and target digests, make verification blocking, retain a verified release manifest, use a tested rollback procedure per environment, and separate diagnostic commands from release success criteria.

## 22. Observability

**QA:** diagnostics pipelines and pod/log/config inspection are present in `origin/qa`.  
**UAT:** diagnostics and ad hoc service checks are present in `origin/uat`; inventory has reindex/diagnostic coupling.  
**PROD:** diagnostics are more extensive and are embedded in some production pipelines, including logs, pod status, environment fixes, and manual test instructions.

**NOT FOUND IN REPOSITORY:** standardized deployment annotations, release correlation IDs, metrics/alerts/traces, dashboard links, or an environment-wide post-deployment observability contract.  
**UNKNOWN:** live logging, monitoring, alerting, retention, health endpoints, and on-call ownership.

**RECOMMENDATION:** standardize release metadata, deployment events, health checks, logs/metrics/traces links, alert ownership, and rollback signals. Keep diagnostic pipelines separate from release gates unless explicitly designed as blocking verification.

## 23. Governance

**CONFIRMED:** production pipeline variants include `ManualValidation@0`; no common environment declaration is visible in YAML. Branch-specific pipeline and rollback files are present in multiple refs.

**UNKNOWN — requires Azure DevOps/Azure verification:** registered pipelines, branch policies, required reviewers, environment approvals/checks, variable-group permissions, service-connection scope, workload identity, AKS RBAC, admission policies, audit retention, and separation of duties.

**RECOMMENDATION:** protect deployment branches, require review for pipeline/manifests/secrets, scope service connections per environment, map CD stages to Azure DevOps environments, and assign ownership for templates, charts, secrets, and rollback.

## 24. Standardization Opportunities

### Common CI logic

These patterns are repeated and should be candidates for shared templates after environment mapping is verified:

- external repository checkout and commit capture;
- Java/toolchain setup;
- Maven module selection and dependency caching;
- unit/integration/contract tests and result publication;
- SAST, SCA, secret, container, and Kubernetes/IaC scans;
- Docker build context and metadata;
- SBOM, signature, provenance, and immutable artifact publication.

### Environment-specific CD logic

These should remain parameterized policy, not copied whole pipelines:

- subscription, service connection, resource group, ACR, AKS, namespace;
- environment branch/approval/check policy;
- replica/resource/probe/ingress values;
- secret references and Key Vault integration;
- deployment strategy, smoke tests, rollback timeout, and observability.

**RECOMMENDATION:** use thin service metadata files plus reusable `build`, `test`, `security`, `container`, `publish`, `deploy`, and `verify` templates. Do not extract an abstraction that hides real QA/UAT/PROD differences before those differences are approved.

## 25. Target Architecture Options

### Option A: Azure DevOps multi-stage pipeline with reusable templates

```text
Application PR -> PR validation -> Build/Test/Security -> immutable image
  -> QA environment -> approval/check -> UAT -> approval/check -> PROD
  -> verification -> rollback/observability
```

Benefits: smallest conceptual change from current Azure DevOps ownership; central stages and approvals; reusable templates. Trade-offs: Azure DevOps YAML can become complex; deployment desired state remains pipeline-driven; requires careful environment parameters. Migration complexity: medium. Prerequisites: branch/pipeline inventory, registry digest policy, service connections, tests, secret model. Risks: template over-parameterization and continued direct `kubectl` coupling.

### Option B: Azure DevOps CI/CD plus Helm

CI builds/tests/scans/signs once; CD packages environment values through a versioned Helm chart and promotes the same digest. Benefits: standard release values, rollback revisions, reusable Kubernetes defaults. Trade-offs: chart abstraction and values complexity; Helm is not automatically GitOps. Migration complexity: medium/high. Prerequisites: live-state inventory, chart ownership, secret integration, environment values. Risks: chart migration changes resource behavior or ordering.

### Option C: Azure DevOps CI plus GitOps CD

Azure DevOps publishes the immutable artifact; a deployment repository change or promotion controller updates environment desired state; Argo CD/Flux reconciles QA/UAT/PROD. Benefits: auditability, drift detection, cluster pull/reconciliation, clear CD separation. Trade-offs: new platform component, operational ownership, secret/bootstrap design, and different incident workflow. Migration complexity: high. Prerequisites: GitOps platform, repository/access model, secret integration, admission policy, rollback/runbook. Risks: competing controllers and unclear ownership during migration.

No single option is selected here. Selection depends on governance, platform ownership, audit requirements, cluster operations, and application release cadence.

## 26. Modernization Roadmap

| Phase | Objective | Activities | Dependencies/risks | Expected outcome | Do not change yet |
|---|---|---|---|---|---|
| 0. Discovery & safety | Establish facts and prevent wrong-environment delivery | Portal registration inventory; live ACR/AKS/digest capture; external repo commits; secret exposure/rotation process; environment ownership | Requires app/platform/security teams; live changes need controlled process | Authoritative environment map and known-good releases | Do not merge branches or migrate controllers |
| 1. Pipeline standardization | Remove duplicated CI logic | Shared templates, pinned tools, commit metadata, validation, service metadata | Preserve real service/build differences | Consistent CI behavior and lower drift | Do not hide environment differences in generic parameters |
| 2. Quality/security gates | Make quality policy explicit | Enable tests; SAST/SCA/container/IaC/secret scans; coverage; SBOM/sign/provenance; exceptions | Existing defects and false positives may block builds | Auditable release gates | Do not make untriaged legacy findings an immediate universal blocker |
| 3. Immutable release | Build once/promote same artifact | Digest tags, registry retention, release manifest, promotion stages, approvals | Requires portal permissions and rollback data model | Traceable cross-environment promotion | Do not remove compatibility tags before consumers migrate |
| 4. Kubernetes standardization | Establish desired-state contract | Choose templates/Helm/GitOps; converge ingress, secrets, policy, probes, resources; validate live state | Highest migration risk; controller ownership | Repeatable environment delivery | Do not bulk-convert or rename resources without canary testing |
| 5. Governance/observability | Operate and audit reliably | Protected branches, environment checks, least privilege, admission, telemetry, alerts, recovery tests | Requires organizational ownership | Measurable, recoverable platform | Do not claim control effectiveness without runtime evidence |

## 27. Prioritized Technical Findings

| ID | Finding | Category | Severity | Evidence | Risk | Recommendation | Effort |
|---|---|---|---|---|---|---|---|
| F-01 | QA branch targets DEV-named runtime | Environment Management | Critical | `origin/qa` pipeline variables | QA validation may not represent a separate QA environment | Confirm and document DEV/QA mapping before changes | Medium |
| F-02 | Production assets exist under UAT ref | Branching | High | `origin/uat/deploy/azurepipeline/prod/`, `k8s/prod/` | Wrong branch can expose/deploy production logic | Establish one production source of truth | High |
| F-03 | Secrets in ConfigMap | Security | Critical | UAT/main `k8s/backend/configmap-aks.yaml` | Credential/JWT exposure | Rotate and migrate to approved secret delivery | Medium |
| F-04 | Two production pipeline families | Maintainability/Governance | High | `origin/prod/deploy/azurepipeline/*`, UAT `prod/*` | Unknown authoritative production behavior | Inventory registrations and retire/govern one family | Medium |
| F-05 | `latest` and rebuild-per-environment | Artifact Management | High | All service pipeline families and manifests | Poor traceability and rollback; QA bytes may differ from PROD | Build once, promote immutable digest | Medium |
| F-06 | Tests skipped across environments | CI/CD | High | Maven `-DskipTests` in service variants | Defects reach all environments | Restore and publish blocking test gates | Low/Medium |
| F-07 | Security controls inconsistent/disabled | Security | High | Checkmarx disabled; scans absent | Vulnerability and supply-chain risk | Standardize blocking layered scans | Medium |
| F-08 | Direct remote `kubectl` and partial application | Kubernetes | High | `az aks command invoke`, per-service apply/restart | Partial success and drift | Controlled release unit with validation/rollback | Medium/High |
| F-09 | Multiple secret/config sources | Security/Reliability | High | ConfigMaps, Secret templates, injection pipelines, inline prod creation | Stale or conflicting runtime settings | One environment-specific secret contract | Medium |
| F-10 | Network policy/ingress/cert drift | Kubernetes | High | QA/UAT/PROD branch-only files | Different exposure/security posture | Define and test environment policy contract | Medium |
| F-11 | No dedicated DEV/STAGING implementation | Environment Management | Medium | Ref/file inventory | Ambiguous testing and promotion topology | Confirm whether aliases exist in Azure/platform | Low |
| F-12 | Rollback differs by branch/service | Reliability | Medium | Rollback YAML, inline `rollout undo`, simple rollout-only files | Recovery may not be repeatable | Standardize digest-based rollback and test it | Medium |
| F-13 | Helm/GitOps placeholders only | Maintainability | Medium | Empty `helm/*`; no GitOps files | Desired-state abstraction absent | Choose controlled migration option after discovery | High |
| F-14 | Operational diagnostics embedded in release pipelines | Operability | Medium | QA/UAT/PROD diagnostics and runtime fixes | Release behavior becomes non-deterministic | Separate diagnostics and formalize remediation changes | Medium |

## 28. Quick Wins

Recommendations only:

- Create a branch/ref-to-environment ownership table and require explicit approval for mismatches.
- Inventory registered pipelines and disable/retire ambiguous duplicate production definitions through the governed Azure DevOps process.
- Scan repository history and rotate exposed credential-like values.
- Add read-only YAML/Kubernetes validation and secret detection to PR checks.
- Record image digest and source commit in every release; stop using `latest` as the deployment selector.
- Make rollout and smoke verification blocking, and remove silent `|| true` behavior from release success criteria through an approved change.
- Document the current QA-to-DEV naming relationship and whether STAGING is an alias.

## 29. Long-Term Recommendations

- Separate common CI from environment-specific CD.
- Build and test once; promote the same signed digest across QA/UAT/PROD.
- Use environment-scoped approvals, service connections, variable groups, and secret-store integration.
- Standardize Kubernetes resource policy, probes, replicas, security context, network policy, ingress, and rollback.
- Adopt Helm or GitOps only after source-of-truth and live-state discovery.
- Establish SBOM, provenance, signing, admission, observability, recovery testing, and release ownership.

## 30. Evidence Gaps

The repository cannot confirm:

- registered/enabled Azure DevOps pipeline names and YAML paths;
- portal trigger overrides, PR policies, branch protection, environments, approvals/checks;
- variable-group values, masking, Key Vault linkage, and permissions;
- service-connection identity, subscription scope, and RBAC;
- external BE/FE contents and exact commit resolved by each run;
- live ACR repositories, retention, tags, digests, and immutability;
- live AKS manifests, replicas, policies, RBAC, admission, and ingress;
- active secret store/workload identity integration;
- current production runtime configuration and deployed release;
- pipeline run, failure, approval, rollback, and recovery history;
- monitoring, alerting, logs, traces, and health endpoint behavior;
- database migration and index/reindex ownership.

Each item is **UNKNOWN — requires Azure DevOps/Azure/external repository/runtime verification**.

## 31. Questions for Azure DevOps / Platform / Application Teams

1. Which pipeline definition is authoritative for QA, UAT, and PROD, especially the two production families?
2. Is QA intentionally deployed to DEV-named ACR/AKS, or is that naming drift?
3. Does a dedicated DEV or STAGING runtime exist outside this repository?
4. Are UAT and PROD promoted from the same image digest or rebuilt from branches?
5. What exact external repository commit does each pipeline build?
6. Which Azure DevOps environments, approvals, checks, branch policies, and service-connection scopes are enforced?
7. Which secret store is authoritative, and why are ConfigMaps, Secret templates, variable groups, and inline creation all present?
8. Are branch-only network policies, ingress, certificates, and rollback files active in the cluster?
9. What is the tested rollback procedure, and is it digest-based?
10. Which tests, scans, smoke checks, migrations, and monitoring gates run outside these YAML files?
11. Which team owns application Dockerfiles, Kubernetes manifests, Helm placeholders, and production release approval?

## 32. What NOT to Change Immediately

Do not immediately merge environment branches, rename QA/DEV resources, delete one production pipeline family, bulk-convert manifests to Helm/GitOps, replace direct AKS access, or change ingress/resource names. The live platform and Azure DevOps registrations are not represented completely, and these actions could break traffic, secret consumers, or rollback paths.

First capture current deployed digests/manifests, verify pipeline registrations, confirm environment ownership, identify secret consumers, and test any new contract in a non-production environment. Preserve resource names and public routes during the first migration. Treat branch consolidation, production pipeline selection, Helm/GitOps adoption, and security-policy changes as staged decisions with recovery checkpoints.

## 33. Final Recommendation

The repository should be treated as a multi-ref, multi-implementation CI/CD ecosystem, not as a UAT pipeline with environment values. The immediate architecture activity should produce an authoritative environment and pipeline map, resolve QA-versus-DEV and UAT-versus-PROD ownership, identify the active production pipeline family, and verify Azure/AKS controls.

After that baseline, standardize the common CI contract, introduce complete tests and security gates, publish immutable signed artifacts, and promote the same digest through explicit QA, UAT, and PROD approvals. Keep CD differences as governed environment policy rather than copied pipelines. Finally, choose Azure DevOps multi-stage, Helm, or GitOps based on platform ownership and audit requirements, and migrate Kubernetes delivery incrementally with verified rollback and observability.
