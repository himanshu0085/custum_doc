# CI/CD Modernization and Standardization Assessment

**Assessment date:** 2026-09-17  
**Mode:** Read-only repository assessment  
**Scope:** Checked-out `main` plus locally available `origin/prod`, `origin/qa`, and `origin/uat` refs

## 1. Executive Summary

The repository is a deployment/configuration repository, not the application source repository. It contains 15 service deployment pipelines, one secret-injection pipeline, Kubernetes manifests, empty Docker/Helm placeholders, and no application source, Maven POM, tests, dependency manifests, Terraform, Bicep, ARM, Compose, or active deployment scripts in checked-out `main`.

The current model is a collection of largely copied single-job pipelines. A service pipeline checks out this repository and an external application repository at `refs/heads/uat`, builds one Maven module with Java 21, skips tests, invokes an optional disabled Checkmarx task, builds and pushes an image to ACR, then uses `az aks command invoke` to apply one Kubernetes manifest and restart the Deployment.

The highest technical risks are:

- UAT-triggered pipelines use production-named Azure service connection material and hardcoded UAT resources; `communication` triggers from `main` while targeting UAT, and `inventory` triggers from both `main` and `uat` while targeting UAT.
- Kubernetes manifests deploy mutable `latest` tags rather than immutable digests. ACR receives a build ID tag, but the deployed manifest does not select it.
- Maven tests are skipped and Checkmarx is disabled. No repository evidence shows SCA, container scanning, IaC scanning, SBOM, signing, provenance, coverage, or functional smoke gates.
- `k8s/backend/configmap-aks.yaml` contains credential-like Redis and JWT values in a ConfigMap. The separate Secret template and secret-injection pipeline are incomplete and inconsistent.
- There are no reusable pipeline templates, published release artifacts, explicit deployment stages, repository-defined Azure DevOps environments, or confirmed automated rollback.
- Remote branch refs contain substantial and undocumented pipeline, ingress, network-policy, diagnostics, secret, and rollback drift.

The practical modernization direction is: establish environment and secret ownership, standardize the repeated build/deploy contract, build and test once, publish an immutable signed artifact with SBOM/provenance, promote the same digest through controlled environments, and move Kubernetes deployment toward a versioned Helm or GitOps contract in stages.

## 2. Scope & Evidence

### Evidence labels

- **CONFIRMED:** directly visible in a tracked file or read-only Git ref/diff.
- **UNKNOWN:** cannot be determined from repository contents and requires Azure DevOps, external repository, ACR, AKS, or runtime verification.
- **RECOMMENDATION:** proposed future state; not implemented by this assessment.

### Repository and environment limits

The checked-out ref is `main`, tracking `origin/main`. Local refs `origin/prod`, `origin/qa`, and `origin/uat` were inspected without checkout. Azure DevOps portal configuration, external application repositories, live cluster state, ACR state, pipeline run history, and monitoring systems were not available. No deployment, `kubectl`, mutating Azure CLI, dependency installation, commit, push, branch change, or implementation change was performed.

## 3. Repository Inventory

| Area | Inventory and evidence | Status / assessment |
|---|---|---|
| Documentation | `README.md` | CONFIRMED placeholder TODO content; does not document release or operations |
| Pipelines | `deploy/azurepipeline/pipeline-*.yaml` (15 service files) | CONFIRMED active-looking direct deployment definitions; no template references |
| Secret pipeline | `deploy/azurepipeline/inject-secrets.yaml` | CONFIRMED manual/triggerless-looking injection flow using variable group `keyvault`; active registration UNKNOWN |
| Pipeline templates | No `template:` references or template directory in checked-out `main` | CONFIRMED no repository-referenced reusable templates |
| Variable templates | No variable template references | CONFIRMED environment values are repeated inline |
| Backend application | External repo `Automotive marketplace - EMB/AutomotiveMarketplace_BE`, checked out by pipelines | CONFIRMED dependency; source contents and commit at run time UNKNOWN |
| Frontend application | External repo `Automotive marketplace - EMB/AutomotiveMarketplace_FE`, checked out by frontend pipeline | CONFIRMED dependency; source contents and commit at run time UNKNOWN |
| Kubernetes | `k8s/backend/*.yaml`, `k8s/frontend/frontend.yaml`, `k8s/ingress.yaml`, `namespace.yaml`, `kind-config.yaml` | CONFIRMED deployment assets; service pipelines apply only selected service files |
| Configuration | `k8s/backend/configmap.yaml`, `configmap-aks.yaml` | CONFIRMED dev/AKS configuration; duplicated environment data |
| Secrets | `k8s/backend/secrets.yaml` | CONFIRMED placeholder Secret template; not a complete secret lifecycle |
| Docker | `deploy/docker/backend/dockerfile`, `deploy/docker/frontend/dockerfile` | CONFIRMED empty files in `main`; service pipelines generate/use Dockerfiles elsewhere |
| Helm | `helm/backend/chart.yaml`, `values.yaml`, `helm/forntend/chart.yaml`, `values.yaml` | CONFIRMED empty placeholders; misspelled `forntend`; not referenced by checked-out pipelines |
| Scripts | No checked-out `main` deployment scripts | CONFIRMED none in `main`; UAT ref adds `k8s/aks-deploy.sh`, `aks-push-images.sh`, `deploy.sh` |
| IaC | No Terraform, Bicep, ARM files | CONFIRMED absent from inspected tracked trees |
| Tests/build config | No application POM, test, package/dependency manifest in this repository | CONFIRMED externalized; test implementation UNKNOWN |
| Rollback | No rollback file in checked-out `main`; `rollback-prod.yml` exists in remote refs | CONFIRMED branch-specific rollback drift |
| Network policy | No network policy in checked-out `main`; added in `origin/prod`, `origin/qa`, `origin/uat` | CONFIRMED branch drift; active use UNKNOWN |

No file is assumed active merely because it exists. Azure DevOps pipeline registration, path, branch, and enablement must be verified in the portal.

## 4. Current CI/CD Flow

### Confirmed backend flow

```text
Commit affecting deployment-repository trigger/path
  -> Azure DevOps service pipeline
  -> checkout self as Pipeline.Workspace/devops
  -> checkout AutomotiveMarketplace_BE at refs/heads/uat
  -> JavaToolInstaller@0, Java 21, preinstalled agent JDK
  -> Maven@4: clean package, module-specific, usually -am -DskipTests
  -> use or generate module Dockerfile
  -> AzureCLI@2: az account set; az acr build
       -> push service:Build.BuildId and service:latest to ACR
  -> AzureCLI@2: az aks command invoke
       -> kubectl apply one k8s/backend/<service>.yaml
       -> kubectl rollout restart deployment/<service>
       -> kubectl rollout status, timeout 180s
```

### Confirmed frontend flow

The frontend pipeline checks out this repository and the external frontend repository at UAT, passes staging domain values as Docker build arguments, builds/pushes `adp-frontend` to the configured ACR, applies `k8s/frontend/frontend.yaml`, restarts `adp-frontend`, and waits for rollout status.

### Confirmed exceptions

- `pipeline-inventory.yaml` also invokes an in-cluster curl pod after a fixed 30-second sleep to call the inventory reindex endpoint. The command ends with `|| true`, so a failed reindex does not fail the pipeline.
- `inject-secrets.yaml` uses variable group `keyvault`, creates/updates only `MONGODB_URI` in the checked-out version, and restarts only a subset of services.
- `pipeline-chat.yaml`, `pipeline-discovery.yaml`, and `pipeline-communication.yaml` use module/repository Dockerfile contexts differently from the common generated-Dockerfile pattern.

### Unknowns in the flow

PR triggers, Azure DevOps pipeline registration, approval/check configuration, service-connection scope, variable-group permissions, actual external repository commit resolution, and live AKS behavior are UNKNOWN. The YAML does not prove that no portal controls exist.

## 5. Pipeline Inventory

| Pipeline | Trigger source / branch / paths | External repo/ref | Build/tests/security | Image / registry | Deployment / environment | Rollback |
|---|---|---|---|---|---|---|
| `pipeline-actions.yaml` | CI `uat`; `adp.actions/*`, `adp.commons/*` | BE / `uat` | Java 21; Maven `clean package`; `-DskipTests`; Checkmarx disabled | `adp-actions:BuildId`, `:latest`; `adpgampuatuaencr` | Apply `actions.yaml`; restart `adp-actions`; UAT AKS | Not in file |
| `pipeline-analytics.yaml` | CI `uat`; analytics/common paths | BE / `uat` | Same common pattern | `adp-analytics`; UAT ACR | Apply/restart analytics; UAT AKS | Not in file |
| `pipeline-authentication.yaml` | CI `uat`; authentication/common paths | BE / `uat` | Same common pattern | `adp-authentication`; UAT ACR | Apply/restart authentication; UAT AKS | Not in file |
| `pipeline-chat.yaml` | CI `uat`; chat/common paths | BE / `uat` | Same gates; module Dockerfile context | `adp-chat`; UAT ACR | Apply/restart chat; UAT AKS | Not in file |
| `pipeline-cms.yaml` | CI `uat`; CMS/common paths | BE / `uat` | Same common pattern | `adp-cms`; UAT ACR | Apply/restart CMS; UAT AKS | Not in file |
| `pipeline-communication.yaml` | **CI `main`**; communication/common paths | BE / `uat` | Same common pattern | `adp-communication`; UAT ACR | Apply/restart communication; UAT AKS | Not in file |
| `pipeline-configuration.yaml` | CI `uat`; configuration/common paths | BE / `uat` | Same common pattern | `adp-configuration`; UAT ACR | Apply/restart configuration; UAT AKS | Not in file |
| `pipeline-discovery.yaml` | CI `uat`; discovery/common paths | BE / `uat` | Same gates; module Dockerfile context | `adp-discovery`; UAT ACR | Apply/restart discovery; UAT AKS | Not in file |
| `pipeline-enquiry.yaml` | CI `uat`; enquiry/common paths | BE / `uat` | Same common pattern | `adp-enquiry`; UAT ACR | Apply/restart enquiry; UAT AKS | Not in file |
| `pipeline-frontend.yaml` | CI `uat`; wildcard path `*` | FE / `uat` | Docker build; tests/scans not visible | `adp-frontend`; staging build args; UAT ACR | Apply/restart frontend; UAT AKS | Not in file |
| `pipeline-inventory.yaml` | **CI `uat` and `main`**; inventory/common paths | BE / `uat` | Same gates plus best-effort reindex | `adp-inventory`; UAT ACR | Apply/restart inventory; reindex; UAT AKS | Not in file |
| `pipeline-masters.yaml` | CI `uat`; masters/common paths | BE / `uat` | Same common pattern | `adp-masters`; UAT ACR | Apply/restart masters; UAT AKS | Not in file |
| `pipeline-orders.yaml` | CI `uat`; orders/common paths | BE / `uat` | Same common pattern | `adp-orders`; UAT ACR | Apply/restart orders; UAT AKS | Not in file |
| `pipeline-statemachine.yaml` | CI `uat`; statemachine/common paths | BE / `uat` | Same common pattern | `adp-statemachine`; UAT ACR | Apply/restart statemachine; UAT AKS | Not in file |
| `pipeline-users.yaml` | CI `uat`; users/common paths | BE / `uat` | Same common pattern | `adp-users`; UAT ACR | Apply/restart users; UAT AKS | Not in file |
| `inject-secrets.yaml` | No trigger shown | None | Variable group `keyvault`; no build | N/A | Secret injection/restarts; UAT-looking subscription/cluster | Not in file |

### Copy similarity

The 14 standard backend service files are effectively copies with service/module/path/image/port substitutions. The frontend and inventory files are variants, and secret injection is a separate operational pipeline. A realistic first extraction would centralize roughly 70-85% of the repeated checkout, tool setup, Maven, scan, ACR, AKS, rollout, and variable logic. This is an estimate based on visible duplication, not a generated code metric.

## 6. Pipeline-by-Pipeline Findings

### Common findings for the 14 standard backend pipelines

**CONFIRMED:** top-level `steps` are used; no explicit `stages`, `jobs`, deployment jobs, `environment`, `dependsOn`, `pr`, or `schedules` appear in the checked-out service definitions. The service resource is checked out from external repo `refs/heads/uat`; Java is 21; Maven runs `clean package` with `-DskipTests`; Checkmarx AST is declared `enabled: false`; `az acr build` pushes Build ID and `latest`; `az aks command invoke` applies one service manifest and restarts the deployment.

**UNKNOWN:** whether portal-level PR policies, schedules, approvals, checks, or permissions supplement the YAML.

**Risk:** a deployment can be triggered from a deployment-repository change without a visible promotion boundary, while the external application source changes independently and is not represented by an immutable application commit in the release artifact.

### `pipeline-communication.yaml`

**CONFIRMED:** trigger branch is `main`, but external application ref, ACR, resource group, AKS cluster, staging configuration, and manifest pattern remain UAT-oriented.  
**Risk:** a main-branch change can deploy UAT-targeted output or create a false impression of production delivery.  
**UNKNOWN:** whether this is an intentional release convention or an accidental branch mismatch.

### `pipeline-inventory.yaml`

**CONFIRMED:** accepts both `main` and `uat`; performs a fixed sleep, creates a `curlimages/curl:latest` pod, calls reindex, and suppresses failure with `|| true`.  
**Risk:** a successful pipeline can leave search/index state stale; mutable diagnostic image and unbounded pod cleanup create operational drift.  
**UNKNOWN:** whether the endpoint is idempotent and whether another scheduled index process exists.

### `pipeline-frontend.yaml`

**CONFIRMED:** wildcard path trigger and hardcoded staging domain build arguments; deploys the frontend manifest and restarts the frontend.  
**Risk:** unrelated deployment-repository changes can trigger a frontend build; configuration is coupled to a staging hostname.  
**UNKNOWN:** whether frontend tests/security checks run in the external repository or another pipeline.

### `inject-secrets.yaml`

**CONFIRMED:** variable group `keyvault` is referenced; the pipeline injects MongoDB URI and restarts only listed services. No trigger is shown in the file.  
**Risk:** partial secret propagation, configuration inconsistency, and service restarts outside a release record.  
**UNKNOWN:** variable-group masking, Key Vault linkage, approvals, and actual pipeline registration.

## 7. Environment Analysis

### Confirmed mapping in checked-out `main`

| Signal | Observed value |
|---|---|
| Pipeline branches | Mostly `uat`; exceptions `main` for communication and `main` + `uat` for inventory |
| External application refs | `refs/heads/uat` |
| ACR | `adpgampuatuaencr` |
| Resource group | `az-emb-uat-np-uaen-automarketplace-aks-rg` |
| AKS | `az-automotivemarketplace-uat-uaen-aks` |
| Namespace | `adp` |
| Host/configuration | `stg.globalautotrade.com` and UAT/staging values |
| Azure service connection text | `subs-adports-prod (...)` |
| Inline subscription | Hardcoded GUID differs from the service-connection label's visible GUID |

**CONFIRMED:** UAT values are repeated throughout files; environment selection is not parameterized through a visible environment contract.  
**UNKNOWN:** whether `main` is the deployment repository's UAT branch, whether the service connection intentionally spans environments, and whether portal approvals isolate production.

### Environment mechanisms observed

- Branches and repository refs: CONFIRMED.
- Inline pipeline variables: CONFIRMED.
- Variable group: `keyvault` in secret pipeline, CONFIRMED.
- Azure DevOps environments/approvals/checks: UNKNOWN; no YAML declaration visible.
- Separate clusters/ACRs for QA/UAT/PROD: UNKNOWN from the checked-out tree.
- Environment overlays: branch-specific files and folders exist in remote refs, CONFIRMED, but intended lifecycle is UNKNOWN.

## 8. Branch Strategy & Drift

### Confirmed ref differences

- `origin/prod` adds diagnostic pipelines, private-ingress pipeline, `rollback-prod.yml`, network policy, private ingress, and production certificate configuration; it modifies nearly every pipeline and manifest.
- `origin/qa` adds diagnostics, ingress pipeline, rollback, network policy, `new-secret.yaml`, disabled ingress, and a frontend Dockerfile; it also modifies most assets.
- `origin/uat` adds a `prod/` pipeline directory with production-named pipelines, `k8s/aks-deploy.sh`, `aks-push-images.sh`, `deploy.sh`, private ingress, rollback, and network policy.

**Risk:** the same logical deployment asset has multiple branch-specific implementations, so a fix, security control, rollback path, or ingress behavior may exist in one environment and not another. Production material under the UAT ref is especially confusing. Whether all branch files are registered/active is UNKNOWN.

### Viable operating models

| Model | Advantages | Disadvantages / migration complexity | Fit observations |
|---|---|---|---|
| A. Environment branches | Familiar; environment-specific manifests can be reviewed in branch; low conceptual change | Drift and cherry-pick risk remain; branch is often mistaken for promotion; fixes diverge; high long-term maintenance | Compatible with current shape but directly preserves the observed drift risk |
| B. Trunk-based CI plus artifact promotion | Clear build-once model; one source revision; strong traceability; environment approvals are explicit | Requires separating CI/CD, artifact registry discipline, environment configuration, and portal governance; moderate migration | Strong fit for repeated service pipelines and immutable ACR releases |
| C. GitOps deployment repository | Desired state and audit trail in Git; cluster reconciler handles deployment; environment changes are reviewable | Requires Argo CD/Flux, repository design, secret integration, reconciliation ownership, and rollback training; highest platform change | Strong fit for many services/manifests after contracts and secrets are stabilized |

The architecture team should choose based on operating model, audit requirements, cluster ownership, and external application release cadence. This assessment does not select a winner.

## 9. Artifact & Release Integrity

**CONFIRMED:** `az acr build` pushes both `service:$(Build.BuildId)` and `service:latest`; manifests use `latest`; no Azure DevOps artifact, SBOM, provenance, signature, or digest pin is visible. The external source is branch-pinned to `uat`, not a commit-pinned release input.

**UNKNOWN:** whether ACR content trust, retention, immutability, signing, or external release metadata is configured in the service.

**Risks:** rebuilt or overwritten `latest` can change what a restart runs; rollback cannot reliably identify prior bytes; source-to-image-to-deployment provenance is weak; environments may rebuild instead of promote.

**RECOMMENDATION:** build once; record source repository and commit, Maven/dependency inputs, image digest, SBOM, scan results, provenance, and release metadata; push immutable tags and deploy by digest; promote the same digest through QA/UAT/production.

## 10. Security Assessment

| ID | Severity | Evidence | Risk | Recommendation |
|---|---|---|---|---|
| SEC-01 | Critical | **CONFIRMED:** `k8s/backend/configmap-aks.yaml`, `REDIS_PASSWORD` and `JWT_SECRET` data entries | Anyone with repository/read or ConfigMap access may obtain credentials/signing material; JWT forgery or data access may result | Treat values as exposed, rotate, remove from Git/ConfigMap, use Key Vault/CSI or workload identity, and verify historical exposure |
| SEC-02 | High | **CONFIRMED:** `k8s/backend/configmap.yaml` contains a JWT value and development profile/configuration | Shared signing material may be reused or accidentally deployed; environment confusion | Separate non-secret config from secrets and enforce environment-specific secret references |
| SEC-03 | High | **CONFIRMED:** `deploy/azurepipeline/inject-secrets.yaml` injects only Mongo URI and restarts a subset of services | Services may run with inconsistent credentials/configuration; secret rotation may not propagate | Define one secret ownership and synchronization model with complete dependency mapping and rollout verification |
| SEC-04 | High | **CONFIRMED:** service connection text, subscription, resource group, cluster, ACR, and staging host are repeated inline | Hardcoded infrastructure increases cross-environment deployment and least-privilege failure risk | Use environment-scoped service connections and non-secret parameter/variable templates; verify scope in portal |
| SEC-05 | High | **CONFIRMED:** Checkmarx task is disabled in repeated pipelines | Code vulnerabilities may reach build/release without a blocking gate | Define SAST/SCA policy, severity thresholds, exception ownership, and blocking behavior |
| SEC-06 | Medium | **CONFIRMED:** no repository evidence of container, IaC, secret scanning, SBOM, signing, provenance, or admission policy | Supply-chain and deployment misconfiguration risks are not systematically detected | Add layered scanning and signed-artifact admission controls |
| SEC-07 | Medium | **CONFIRMED:** inventory uses `curlimages/curl:latest` and a remote command with endpoint invocation | Mutable tool image and ad hoc in-cluster execution weaken auditability and policy control | Use a versioned, authorized job/task with explicit identity, timeout, cleanup, and failure semantics |

Actual service-connection permissions, Key Vault linkage, variable masking, RBAC, and admission controls are **UNKNOWN — requires Azure DevOps/AKS portal verification**.

## 11. Kubernetes Assessment

**CONFIRMED:** manifests define 15 backend services plus frontend, Services, namespace, ingress, ConfigMaps, and a Secret template. Most Deployments use one replica, `imagePullPolicy: Always`, and `latest`; readiness probes are commonly TCP with long initial delays; liveness probes are inconsistent; resource requests/limits are repeated but policy validation is not visible.

**CONFIRMED:** checked-out `main` contains no NetworkPolicy, PodDisruptionBudget, HorizontalPodAutoscaler, ServiceAccount/RBAC, or security-context standard. These are added in some remote refs, so branch drift exists.

**CONFIRMED:** `k8s/ingress.yaml` routes `/api/` to authentication and `/` to frontend with Application Gateway annotations and staging TLS hostname; `k8s/frontend/frontend.yaml` contains a hardcoded staging host alias.

**Risks:** one replica limits availability; `Always` plus `latest` obscures version selection; TCP readiness may not prove application/dependency health; force restart creates avoidable disruption; shared resource ordering is not controlled; branch-specific policies may not be deployed with workloads.

**UNKNOWN:** live replica counts, admission policy, Pod Security settings, RBAC, autoscaling, actual ingress controller state, and whether external platform automation supplies missing resources.

## 12. Docker/Container Assessment

**CONFIRMED:** checked-out Dockerfiles `deploy/docker/backend/dockerfile` and `deploy/docker/frontend/dockerfile` are empty. Several pipelines generate Dockerfiles inline when absent; others use external module Dockerfiles or different build contexts. The visible generated backend pattern uses `eclipse-temurin:21-jre-alpine`, copies a JAR, exposes a service port, and starts Java with JVM options.

**Gaps:** Dockerfile ownership is external/implicit; build context differs by service; base image digest is not pinned in the deployment repository; non-root execution, multi-stage build, image labels, SBOM, vulnerability scanning, and reproducible build policy are not confirmed.

**RECOMMENDATION:** define reviewed Dockerfile ownership, pinned base images, minimal runtime/non-root defaults, consistent labels, deterministic build inputs, image scan policy, SBOM, signature, and digest-based release metadata.

## 13. Helm/GitOps Assessment

**CONFIRMED:** Helm files are empty and not referenced by checked-out pipelines. No Argo CD or Flux configuration is present. Current deployment is direct manifest application through `az aks command invoke`.

**RECOMMENDATION:** Helm can package repeated service defaults, environment values, probes, resources, image digest, and shared dependencies. GitOps can make desired state, approvals, drift, and rollback auditable. Migration risks include chart/template correctness, secret integration, ordering, ingress compatibility, release ownership, and divergence from live cluster state. First inventory live state and establish a known-good manifest/digest baseline.

## 14. Quality Gates

| Gate | State | Evidence |
|---|---|---|
| Unit tests | Missing/disabled in these pipelines | Maven options include `-DskipTests` |
| Integration/contract tests | Missing in repository pipeline definitions | No visible task or external test contract |
| Coverage | Missing | No publication/threshold task |
| SAST | Disabled | Checkmarx AST task has `enabled: false` |
| SCA/dependency scan | Missing from checked-out pipelines | No visible task |
| Container scan | Missing | No visible task |
| Secret scan | Missing | No visible task |
| IaC/Kubernetes validation | Missing | No visible validation stage/task |
| SBOM/signing/provenance | Missing | No visible generation or verification |
| Rollout verification | Present but narrow | `kubectl rollout status`, 180-second timeout |
| Smoke/functional validation | Missing except inventory best-effort endpoint call | Inventory call is not a blocking, general smoke gate |
| Azure DevOps approvals/checks | UNKNOWN | Requires portal verification |

## 15. Failure & Rollback

**CONFIRMED:** rollout status has a 180-second timeout; there is no rollback command or prior digest capture in checked-out service pipelines. Image push, manifest apply, restart, and verification are separate operations. Inventory post-deployment reindex failure is explicitly ignored.

Partial-success scenarios include: image pushed but manifest apply fails; manifest applied but rollout fails; rollout succeeds while application dependencies or business health are broken; `latest` changes between build and restart; secret injection updates only some services; reindex fails while pipeline reports success.

**UNKNOWN:** whether Azure DevOps automatically retries tasks, whether AKS Deployment revision history is retained, whether external rollback pipelines are registered, and whether database migrations are handled elsewhere.

**RECOMMENDATION:** record immutable image digest and release revision, validate before rollout, use controlled progressive or transactional deployment where appropriate, run blocking smoke/health checks, retain a known-good release, and roll back by digest/release revision. Database migration compatibility must be separately designed; it cannot be inferred here.

## 16. Operability

**CONFIRMED:** rollout status is the primary visible deployment verification. Other remote refs add diagnostics pipelines, but those are branch-specific and not proven active. No standard deployment annotations, release links, metrics, alerting, or post-deployment dashboard integration is visible in checked-out `main`.

**UNKNOWN:** application logging, centralized metrics/tracing, alert rules, health endpoint quality, log retention, and on-call ownership.

**RECOMMENDATION:** standardize release annotations, deployment correlation IDs, service health/smoke checks, rollout duration/error metrics, links to logs/metrics/traces, and alert ownership. Keep diagnostic workflows separate from release success criteria unless they are formal gates.

## 17. Governance

**CONFIRMED:** no `CODEOWNERS`, repository-defined PR policy, Azure DevOps environment declaration, approval declaration, or service-connection authorization is visible in this repository.

**UNKNOWN — requires Azure DevOps portal verification:** branch protection, required reviewers, PR validation, environment approvals/checks, variable-group permissions, service-connection scope, pipeline authorization, separation of duties, audit retention, AKS RBAC, workload identity, and admission controls.

**RECOMMENDATION:** use protected branches and required reviewers for deployment assets; map deployment stages to Azure DevOps environments; scope service connections and variable groups per environment; require authorized release identities; establish ownership for pipeline templates, manifests, secrets, and rollback.

## 18. Current-State Architecture

```text
Developer / deployment-repository change
        |
        v
Azure DevOps service-specific YAML pipeline
        |
        +--> checkout deployment repository as devops
        +--> checkout external BE/FE repository at UAT branch
        |
        v
JavaToolInstaller 21 -> Maven clean package (-DskipTests)
        |
        v
Inline or external Dockerfile -> az acr build
        |
        v
UAT-oriented Azure Container Registry
  tags: service:BuildId and service:latest
        |
        v
az aks command invoke -> remote kubectl apply one manifest
        |
        v
AKS namespace adp -> rollout restart -> rollout status
        |
        +--> inventory: best-effort curl/reindex
        +--> separate secret pipeline: variable group -> partial secret update/restarts
```

This is the **current-state** view reconstructed from repository evidence, not a claim about live Azure topology.

## 19. Recommended Target Architecture

```text
Developer
  -> PR validation: YAML/Kubernetes validation, tests, policy checks
  -> reusable CI pipeline
       -> checkout pinned application commit + deployment revision
       -> build and test
       -> SAST/SCA/secret/IaC/container scans
       -> SBOM + provenance + signed immutable image
       -> publish image digest and release metadata
  -> CD promotion pipeline / GitOps change
       -> QA environment approval and deploy exact digest
       -> automated smoke/health/contract verification
       -> UAT approval and deploy exact digest
       -> production approval/checks and deploy exact digest
       -> post-deployment verification and observability
  -> rollback to prior verified digest/release revision

Kubernetes delivery:
  versioned Helm chart or GitOps desired-state repository
    -> environment values/overlays
    -> Key Vault/CSI or workload identity secret references
    -> policy validation and signed-image admission
    -> AKS reconciliation/deployment
```

This is a **RECOMMENDATION**, not an implementation or assertion that these controls currently exist.

## 20. Standardization Recommendations

**RECOMMENDATION:** use thin service entrypoints and shared templates such as `build.yml`, `test.yml`, `security.yml`, `container.yml`, `deploy.yml`, and `verify.yml`; keep service metadata (module, image, manifest/chart, ports, path filters) declarative.

Standardize:

- stage names: Build, Test, Security, Publish, Deploy, Verify;
- typed service/environment parameters and canonical names;
- environment-scoped non-secret variables and secret references;
- pinned tool/runtime versions and caching policy;
- immutable image naming and digest promotion;
- required probes/resources/security-context policy;
- rollout, smoke-test, retry, timeout, and rollback semantics;
- artifact metadata, release annotations, and audit links.

Do not duplicate environment values across pipelines and manifests. Do not make a template abstraction until external application checkout, Dockerfile ownership, and deployment ownership are agreed.

## 21. Modernization Roadmap

| Phase | Objective and activities | Dependencies / risks | Expected outcome | Do not change yet |
|---|---|---|---|---|
| 0. Discovery & Safety | Verify portal registrations, live AKS/ACR state, external repos, secret consumers, branch ownership; rotate exposed values through approved process; capture known-good digests | Requires platform/app/security owners; risk of disrupting undocumented consumers | Trusted baseline and environment map | Do not consolidate branches or migrate deployment technology |
| 1. Pipeline Standardization | Extract common checkout/build/publish/deploy behavior; add YAML/Kubernetes validation; preserve service entrypoints | Must preserve service-specific build contexts and ports | Lower duplication and consistent execution | Do not change public routes or resource names |
| 2. Security & Quality Gates | Re-enable/standardize tests, SAST, SCA, secret, container, IaC scans; define thresholds/exceptions | Existing defects may block releases; need ownership and false-positive process | Repeatable blocking quality/security policy | Do not make every historical finding a release blocker without triage |
| 3. Immutable Release Model | Build once; publish digest, SBOM, provenance, signature; promote through environments | Requires registry policy, approvals, rollback data model | Traceable release and reliable rollback | Do not remove compatibility tags until consumers migrate |
| 4. Kubernetes / Helm or GitOps | Choose Helm/GitOps after live-state inventory; model shared resources, secrets, policies, ingress, probes, and overlays | Highest structural migration risk; reconciliation/ownership must be explicit | Versioned desired state and controlled deployment | Do not bulk-convert manifests without canary validation |
| 5. Governance / Observability / Platform Controls | Protected branches, environments/checks, least privilege, admission policies, release telemetry, alerts, runbooks | Requires Azure DevOps/AKS governance decisions | Auditable and operable platform | Do not claim compliance until portal/runtime evidence exists |

## 22. Prioritized Findings

| ID | Finding | Category | Severity | Evidence | Risk | Recommendation | Effort |
|---|---|---|---|---|---|---|---|
| F-01 | Credentials in ConfigMap | Security | Critical | `k8s/backend/configmap-aks.yaml`, `REDIS_PASSWORD`/`JWT_SECRET` | Credential theft, JWT forgery, data access | Rotate and migrate to approved secret store; verify history | Medium |
| F-02 | UAT/main/prod target ambiguity | Environment Management | High | Pipeline triggers/refs/variables; communication and inventory exceptions | Wrong environment deployment | Establish explicit environment contract and portal checks | Medium |
| F-03 | Mutable `latest` deployment | Artifact Management | High | Service manifests and `az acr build` tags | Non-reproducible releases and weak rollback | Deploy immutable digest; promote same artifact | Medium |
| F-04 | Tests skipped | CI/CD | High | Maven `-DskipTests` in service pipelines | Defects reach deployment | Restore tests and publish blocking results | Low/Medium |
| F-05 | Security scan disabled/missing | Security | High | Checkmarx `enabled: false`; no visible SCA/container/IaC/SBOM | Vulnerable code/images/config reach environments | Define layered blocking scan policy | Medium |
| F-06 | Pipeline duplication | Maintainability | High | 14 near-identical service YAMLs | Fixes drift and inconsistent controls | Shared parameterized templates | Medium |
| F-07 | Partial secret injection | Security / Reliability | High | `inject-secrets.yaml` only updates Mongo URI/subset restarts | Inconsistent runtime configuration | Define complete secret ownership/propagation | Medium |
| F-08 | Direct remote kubectl and force restart | Kubernetes / CI/CD | High | `az aks command invoke`, apply + rollout restart | Partial deployment and avoidable disruption | Controlled deployment contract with pre/post validation | Medium/High |
| F-09 | Branch drift | Branching | High | Remote ref diffs; production assets under UAT | Environment behavior diverges | Select and document branch/promotion model | High |
| F-10 | Empty Helm/Docker placeholders | Maintainability | Medium | `deploy/docker/*`, `helm/*` are empty | False source-of-truth and drift | Decide ownership; remove/complete only through approved migration | Low/Medium |
| F-11 | Weak health verification | Kubernetes / Operability | Medium | TCP/readiness inconsistency; rollout only | Bad application can be considered healthy | Application-level probes and smoke tests | Medium |
| F-12 | Best-effort reindex | Reliability | Medium | Inventory fixed sleep, curl `latest`, `|| true` | Stale search state with green pipeline | Idempotent observable job and blocking policy | Medium |
| F-13 | Governance unknown | Governance | Medium | No repository evidence of environments/checks/policies | Unauthorized or unreviewed deployment | Verify portal controls and document ownership | Low/Medium |

## 23. Quick Wins

These are recommendations only, not implemented:

- Document the actual pipeline-to-environment map and mark `main`/UAT exceptions for explicit approval.
- Add read-only YAML/Kubernetes validation and pipeline linting to PR validation.
- Stop treating `latest` as the release selector; at minimum record and verify the built digest before deployment.
- Re-enable tests and establish a temporary documented exception process for existing failures.
- Enable secret detection and scan the repository/history; rotate exposed values through the approved process.
- Centralize repeated non-secret variables and service connection references.
- Add blocking smoke/health verification after rollout and make inventory reindex failure observable.
- Create an operational README/runbook covering ownership, dependencies, prerequisites, deployment, and recovery.

## 24. Long-Term Recommendations

- Separate application CI from deployment CD and promote one immutable artifact.
- Adopt a shared pipeline template library with thin service metadata files.
- Use environment approvals/checks and least-privilege service connections.
- Move secrets to Key Vault/CSI or workload identity-backed delivery.
- Choose Helm or GitOps after live-state and ownership discovery; migrate incrementally.
- Add signed images, SBOM/provenance, admission controls, network policies, resource/probe standards, autoscaling and disruption controls where workload requirements justify them.
- Establish release telemetry, change correlation, rollback runbooks, and regular recovery tests.

## 25. Evidence Gaps / Questions for Azure DevOps Team

| Required information | Why it matters |
|---|---|
| Registered/enabled pipeline list and YAML path per pipeline | Determines what is actually executable |
| PR policies, protected branches, required reviewers, CODEOWNERS equivalent | Determines whether changes are reviewed and gated |
| Azure DevOps environment approvals/checks | Determines whether direct deployment is portal-gated |
| Service connection identities, scopes, and approvals | Determines blast radius and environment isolation |
| Variable groups, masking, Key Vault linkage, permissions | Determines whether secrets are protected and complete |
| External BE/FE repo commits and branch protections | Determines source reproducibility and trigger coupling |
| ACR repositories, retention, immutability, current digests | Determines artifact traceability and rollback feasibility |
| AKS live objects, rollout history, RBAC, admission policies | Determines runtime security and actual desired state |
| Actual deployed image digests per environment | Determines what is running, independent of tags |
| Production runtime config, secrets, ingress, network policy | Determines whether branch files match production |
| Pipeline run history, failures, rollback history | Determines real reliability and operational workarounds |
| Monitoring, alerting, logs, tracing, health endpoints | Determines post-deployment detection and recovery |
| Database migrations and reindex ownership | Determines safe ordering and rollback constraints |

Each item is **UNKNOWN — requires Azure DevOps portal, Azure/AKS, external repository, or runtime verification**.

## 26. What NOT to Change Immediately

Do not immediately rename all services, consolidate branches, replace `az aks command invoke`, bulk-convert manifests to Helm, or move to GitOps in a single release. The live cluster, external application repositories, ingress routes, secrets, service names, and branch-specific pipeline registrations are not fully represented here.

First capture known-good manifests and image digests, verify live dependencies and secret consumers, inventory portal controls, and test proposed contracts in a non-production environment. Preserve resource names and public routes during the first migration unless compatibility is proven. Treat branch consolidation, Helm/GitOps adoption, and runtime security changes as staged migrations with rollback checkpoints.

## 27. Final Recommendation

The repository should be modernized incrementally, beginning with evidence and safety rather than a wholesale rewrite. The first implementation workstream should isolate environments and remove/rotate exposed secrets. The second should extract the repeated pipeline behavior while preserving service-specific build details. The third should introduce build-once/promote-same-artifact with immutable image digests, tests, security gates, SBOM/provenance, and controlled Azure DevOps environments. Only after those contracts are stable should Kubernetes delivery move toward Helm or GitOps.

This sequence reduces production risk while improving traceability, quality, rollback, security, and maintainability. It also leaves room for the architecture team to choose the appropriate branch and deployment model after the missing Azure DevOps and runtime evidence is collected.
