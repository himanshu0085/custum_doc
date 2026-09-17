# CI/CD Modernization and Standardization Assessment

## 1. Scope and Evidence

Assessment date: 2026-09-17.

This is a read-only assessment of the checked-out `main` tree and the locally available remote refs `origin/prod`, `origin/qa`, and `origin/uat`. No Azure DevOps project settings, service-connection permissions, environment checks, variable-group values, external application repositories, cluster state, or pipeline run history are available in this repository. Those items are marked **UNKNOWN** rather than assumed.

The checked-out repository is a deployment/configuration repository. It contains 16 Azure Pipeline YAML files, Kubernetes manifests, empty Dockerfile/Helm placeholders, and no application source, Maven POM, tests, scripts, Terraform, Bicep, ARM, Docker Compose, or dependency manifests.

## 2. Executive Summary

The current implementation is a set of service-specific, single-job pipelines. Each service pipeline checks out this repository and an external application repository, builds one Maven module with Java 21, builds and pushes an image directly in Azure Container Registry, applies one Kubernetes manifest through `az aks command invoke`, and restarts the deployment. There is no reusable pipeline template, multi-stage promotion flow, published Azure DevOps artifact, or repository-defined approval gate.

The largest modernization blockers are:

1. **Environment safety:** most pipelines trigger from `uat` but use an Azure service connection named `subs-adports-prod`, a hardcoded subscription, UAT ACR, UAT resource group, UAT AKS cluster, and UAT application checkout. `communication` triggers from `main` while building UAT; `inventory` triggers from both `uat` and `main` while deploying UAT.
2. **Release integrity:** images are pushed with both `$(Build.BuildId)` and mutable `latest`, while manifests reference `latest`. The build output is not published as an immutable artifact and is not promoted between environments.
3. **Quality and security gates:** Maven is invoked with `-DskipTests`; Checkmarx is present but disabled in the service pipelines; no repository-defined SCA, container, IaC, signing, coverage, or smoke-test gate is present.
4. **Secret exposure and drift:** `configmap-aks.yaml` contains a Redis password and JWT secret in a ConfigMap, while `secrets.yaml` and `inject-secrets.yaml` represent a separate, incomplete secret path. The secret injection pipeline updates only `MONGODB_URI` and restarts only a subset of workloads.
5. **Operational coupling:** deployment is performed through inline shell commands and remote `kubectl`, with manifests applied one service at a time. Cluster prerequisites, shared configuration, ingress, and secret lifecycle are not part of the normal service deployment path.
6. **Maintainability:** approximately identical YAML is copied across 15 service pipelines, with service-specific inline Dockerfile generation and inconsistent build contexts. Branches have substantial undocumented drift and additional production/diagnostic/rollback files.

Recommended target state: build and test once, produce a signed immutable image, publish provenance and deployment metadata, promote the same digest through QA/UAT/production using explicit environments and approvals, and deploy through a versioned Helm or GitOps contract with centralized secret delivery and health validation.

## 3. Current Logical Dependency Map

```text
Azure DevOps pipeline YAML
  -> checkout self as $(Pipeline.Workspace)/devops
  -> checkout external AutomotiveMarketplace_BE or AutomotiveMarketplace_FE at refs/heads/uat
  -> JavaToolInstaller@0 (Java 21)
  -> Maven@4 clean package for one module, usually -am and -DskipTests
  -> inline/generated Dockerfile in the external checkout
  -> AzureCLI@2: az acr build to ACR with Build.BuildId and latest tags
  -> AzureCLI@2: az aks command invoke
       -> kubectl apply one service manifest from this repository
       -> kubectl rollout restart and rollout status
  -> optional inventory reindex or separate secret injection
```

Deployment assets are in [k8s](k8s) and pipeline definitions are in [deploy/azurepipeline](deploy/azurepipeline). No pipeline in the checked-out tree references a YAML template, variable template, deployment template, script file, Helm chart, or committed Dockerfile as an active build input.

## 4. Pipeline Inventory

All service pipelines use the same broad shape: top-level `steps`, `ubuntu-latest`, two repository checkouts, Java 21, Maven, an optional disabled Checkmarx task, ACR build/push, and AKS apply/restart. No `pr`, `schedules`, `stages`, `jobs`, `deployment` jobs, `environment`, `dependsOn`, published artifact, or explicit condition is defined in the checked-out service files.

| Pipeline | Trigger/path filter | External checkout | Module/image | Manifest and notable behavior |
|---|---|---|---|---|
| [pipeline-actions.yaml](deploy/azurepipeline/pipeline-actions.yaml) | `uat`; `adp.actions/*`, `adp.commons/*` | Backend `uat` | `adp.actions` / `adp-actions` | `k8s/backend/actions.yaml` |
| [pipeline-analytics.yaml](deploy/azurepipeline/pipeline-analytics.yaml) | `uat`; `adp.analytics/*`, `adp.commons/*` | Backend `uat` | `adp.analytics` / `adp-analytics` | `analytics.yaml` |
| [pipeline-authentication.yaml](deploy/azurepipeline/pipeline-authentication.yaml) | `uat`; `adp.authentication/*`, `adp.commons/*` | Backend `uat` | `adp.authentication` / `adp-authentication` | Gateway/authentication deployment |
| [pipeline-chat.yaml](deploy/azurepipeline/pipeline-chat.yaml) | `uat`; `adp.chat/*`, `adp.commons/*` | Backend `uat` | `adp.chat` / `adp-chat` | Uses module Dockerfile/build context |
| [pipeline-cms.yaml](deploy/azurepipeline/pipeline-cms.yaml) | `uat`; `adp.cms/*`, `adp.commons/*` | Backend `uat` | `adp.cms` / `adp-cms` | `cms.yaml` |
| [pipeline-communication.yaml](deploy/azurepipeline/pipeline-communication.yaml) | **`main`**; `adp.communication/*`, `adp.commons/*` | Backend `uat` | `adp.communication` / `adp-communication` | Main-to-UAT mismatch |
| [pipeline-configuration.yaml](deploy/azurepipeline/pipeline-configuration.yaml) | `uat`; `adp.configuration/*`, `adp.commons/*` | Backend `uat` | `adp.configuration` / `adp-configuration` | `configuration.yaml` |
| [pipeline-discovery.yaml](deploy/azurepipeline/pipeline-discovery.yaml) | `uat`; `adp.discovery/*`, `adp.commons/*` | Backend `uat` | `adp.discovery` / `adp-discovery` | Uses module Dockerfile/build context |
| [pipeline-enquiry.yaml](deploy/azurepipeline/pipeline-enquiry.yaml) | `uat`; `adp.enquiry/*`, `adp.commons/*` | Backend `uat` | `adp.enquiry` / `adp-enquiry` | `enquiry.yaml` |
| [pipeline-frontend.yaml](deploy/azurepipeline/pipeline-frontend.yaml) | `uat`; wildcard `*` | Frontend `uat` | frontend / `adp-frontend` | Builds with staging domain arguments |
| [pipeline-inventory.yaml](deploy/azurepipeline/pipeline-inventory.yaml) | **`uat` and `main`**; inventory/common paths | Backend `uat` | `adp.inventory` / `adp-inventory` | Applies manifest, then best-effort reindex call |
| [pipeline-masters.yaml](deploy/azurepipeline/pipeline-masters.yaml) | `uat`; `adp.masters/*`, `adp.commons/*` | Backend `uat` | `adp.masters` / `adp-masters` | `masters.yaml` |
| [pipeline-orders.yaml](deploy/azurepipeline/pipeline-orders.yaml) | `uat`; `adp.orders/*`, `adp.commons/*` | Backend `uat` | `adp.orders` / `adp-orders` | `orders.yaml` |
| [pipeline-statemachine.yaml](deploy/azurepipeline/pipeline-statemachine.yaml) | `uat`; `adp.statemachine/*`, `adp.commons/*` | Backend `uat` | `adp.statemachine` / `adp-statemachine` | `statemachine.yaml` |
| [pipeline-users.yaml](deploy/azurepipeline/pipeline-users.yaml) | `uat`; `adp.users/*`, `adp.commons/*` | Backend `uat` | `adp.users` / `adp-users` | `users.yaml` |
| [inject-secrets.yaml](deploy/azurepipeline/inject-secrets.yaml) | No repository trigger shown | None | N/A | Variable group `keyvault`; updates Mongo URI only |

### Per-pipeline assessment

The following applies to each of the 15 service pipelines unless an exception is stated above:

- **Trigger:** explicit CI branch/path trigger, but no PR validation, schedule, path exclusion, or release-only guard is defined in the file. The pipeline is a direct deployment pipeline, so a matching commit can deploy without a repository-defined promotion boundary.
- **Build:** Maven `clean package` uses Java 21 and a moving `ubuntu-latest` agent label. Maven/tool/plugin versions and dependency cache policy are not pinned here. The external repository is pinned to the `uat` branch rather than to a commit or pipeline resource version.
- **Testing:** Maven tests are explicitly skipped. No test result, coverage, integration, contract, smoke, or quality-gate publication is visible.
- **Security:** Checkmarx AST is present but `enabled: false` in the repeated service definitions. No SCA, container scan, IaC scan, SBOM, image signing, or provenance attestation is visible. Service connection and subscription identifiers are hardcoded in YAML; actual permissions are unknown.
- **Artifact:** `az acr build` pushes `service:$(Build.BuildId)` and `service:latest`. No Azure DevOps artifact is published, and the Kubernetes manifests use `latest`, so the deployed bytes are not selected by the build ID.
- **Deployment:** `az aks command invoke` sends `kubectl apply` for one service manifest, then force-restarts the Deployment and waits up to 180 seconds for rollout. Namespace/shared resource reconciliation is not part of the service flow.
- **Approvals/gates:** No environment, approval, check, or gate is declared in YAML. Azure DevOps UI approvals/checks are **UNKNOWN**.
- **Failure/recovery:** rollout status gives a bounded wait, but there is no automated rollback, previous digest capture, failed-release quarantine, or post-deployment functional test. A restart can cause unnecessary disruption even when only a manifest or configuration change is involved.

## 5. Confirmed Current Flow

For a matching backend commit, the confirmed sequence is:

1. Azure DevOps starts a service pipeline when the deployment repository branch/path trigger matches.
2. The pipeline checks out this repository into `devops` and the external backend repository at `refs/heads/uat` into `backend`. The frontend pipeline does the equivalent for the frontend repository.
3. Java 21 is selected from the agent's preinstalled toolchain.
4. Maven builds the selected module and required modules with `clean package -DskipTests`.
5. The pipeline creates a Dockerfile if the external module does not have one, or uses an external-repository/module Dockerfile for selected services.
6. Azure Container Registry builds and pushes two tags: an immutable-looking build ID tag and mutable `latest`.
7. Azure CLI invokes `kubectl apply` remotely against AKS with the corresponding manifest from this repository.
8. The deployment is restarted and its rollout status is checked for up to 180 seconds.

The frontend pipeline additionally applies the frontend manifest and uses hardcoded staging URLs as build arguments. The inventory pipeline sleeps 30 seconds and starts a temporary `curlimages/curl:latest` pod to call the reindex endpoint; its command ends with `|| true`, so a failed reindex does not fail the pipeline. The secret pipeline creates/updates `adp-secrets` from the `keyvault` variable group and restarts only the services listed in that file.

## 6. Environments, Branches, and Promotion

### Confirmed

- Checked-out `main` has service triggers mostly on `uat` and deployment variables for UAT ACR, UAT resource group, and UAT AKS.
- Backend and frontend repository resources are all pinned to `refs/heads/uat` in the checked-out pipelines.
- `communication` uses a `main` trigger but still builds the UAT checkout and deploys to UAT.
- `inventory` accepts both `uat` and `main` but deploys the same UAT target.
- Kubernetes manifests use namespace `adp`, image names such as `adpgampuatuaencr.azurecr.io/adp-users:latest`, and staging hostname values.

### Branch-ref assessment

The local refs are substantially divergent from checked-out `main`:

- `origin/prod` adds diagnostic pipelines, a private-ingress pipeline, `rollback-prod.yml`, a network policy, private ingress, and production certificate configuration; it modifies nearly every pipeline and manifest.
- `origin/qa` adds diagnostic and ingress pipelines, rollback, network policy, `new-secret.yaml`, disabled ingress, and a frontend Dockerfile; it also modifies most deployment assets.
- `origin/uat` adds a `prod/` pipeline directory containing many production pipelines, `k8s/aks-deploy.sh`, `k8s/aks-push-images.sh`, `k8s/deploy.sh`, private ingress, rollback, and network policy.

This is **CONFIRMED** from remote-ref trees and diffs. The intended source-of-truth relationship among these refs is **UNKNOWN**. The presence of production files under the UAT ref and environment-specific files in multiple branches indicates high branch drift and a material risk of deploying the wrong environment.

There is no confirmed build-once/promote-same-artifact flow. Rebuild versus promotion behavior across Azure DevOps pipelines is **UNKNOWN** because the branch-specific pipeline contents and Azure DevOps pipeline registrations were not exhaustively run or inspected through the service UI.

## 7. Configuration and Secret Handling

### Confirmed risks

- [configmap-aks.yaml](k8s/backend/configmap-aks.yaml) contains `REDIS_PASSWORD` and `JWT_SECRET` as ordinary ConfigMap data. The file comments acknowledge this is a temporary compatibility arrangement.
- [configmap.yaml](k8s/backend/configmap.yaml) contains a development JWT value and development profile values.
- [secrets.yaml](k8s/backend/secrets.yaml) is a placeholder Secret template and warns against committing real values, but it is not the mechanism used by the normal service pipelines.
- [inject-secrets.yaml](deploy/azurepipeline/inject-secrets.yaml) consumes variable group `keyvault` but injects only `MONGODB_URI` in the checked-out version.
- Secret injection restarts only a subset of services, so configuration propagation is incomplete and implicit.
- Environment-specific values are duplicated in pipeline variables, manifests, and inline commands instead of being selected from one environment contract.

The committed credential-like values should be treated as exposed and rotated through the appropriate incident/change process. This recommendation is remediation guidance only; no changes were made.

### Unknown

Whether the values are active, masked in Azure DevOps, overridden by other manifests, or already rotated cannot be established from Git. Azure Key Vault, CSI driver, managed identity, service-connection scope, and cluster RBAC configuration are not present in the repository.

## 8. Kubernetes, Docker, and Helm Assessment

- There are 15 service Deployments plus Services and shared resources in namespace `adp`.
- Most workloads use one replica, `imagePullPolicy: Always`, and `latest`; this weakens availability, rollback, and provenance.
- Readiness probes are mostly TCP probes with long initial delays. Liveness probes are inconsistent. End-to-end readiness and dependency health are not established by these probes.
- Resource requests/limits are broadly repeated but there is no policy or validation visible.
- [ingress.yaml](k8s/ingress.yaml) routes `/api/` to authentication and `/` to frontend, uses an Azure Application Gateway class, and has staging TLS/domain values.
- [frontend.yaml](k8s/frontend/frontend.yaml) includes a hardcoded host alias for the staging hostname.
- The committed Dockerfiles under [deploy/docker](deploy/docker) are empty. The Helm files under [helm](helm) are empty, and `forntend` is misspelled. These are not active deployment abstractions in the checked-out pipelines.
- Effective Dockerfiles are generated inline or taken from the external application repository. This makes container build behavior difficult to review, reproduce, and scan.
- Pipelines do not apply the namespace, ingress, ConfigMaps, Secret template, or all service resources as a release unit. Their lifecycle and ordering are therefore external or manual.

## 9. Standardization Assessment

| Standard | Current state | Assessment |
|---|---|---|
| Naming | Service names, image names, paths, and `forntend` directory are inconsistent | Standardize canonical service/environment/resource names |
| Branch strategy | UAT triggers and production-named assets are mixed across refs | Define protected promotion branches or release tags with one source of truth |
| Trigger convention | Similar path filters are copied; main/UAT exceptions exist | Centralize trigger policy and add PR validation |
| Stages/jobs | Top-level steps only | Use Build, Security, Publish, Deploy, Verify stages with dependencies |
| Templates | No active reusable templates | Extract common checkout/build/scan/publish/deploy templates |
| Parameters | Service values are embedded in each file | Use typed service/environment parameters and approved defaults |
| Variables | UAT infrastructure is hardcoded repeatedly | Use environment-scoped variable groups/templates with non-secret values only |
| Secrets | ConfigMap credentials plus partial variable-group injection | Use Key Vault/CSI or workload identity and one documented ownership model |
| Agents/tooling | `ubuntu-latest`, Java 21 preinstalled, unpinned external build | Pin supported toolchain and containerize build where practical |
| Artifacts | ACR tags but no pipeline artifact or digest contract | Publish immutable image digest, SBOM, provenance, and release metadata |
| Approvals | No YAML-declared environments/checks | Map deploy stages to Azure DevOps environments and required checks |
| Quality/security | Tests skipped; Checkmarx disabled | Make tests/scans policy gates with documented exceptions |
| Deployment | Direct remote `kubectl` per service | Use Helm/GitOps or a controlled deployment job with validation |
| Rollback | No checked-out main rollback path | Roll back by image digest and versioned manifest/chart, then verify |
| Observability | Rollout status only; ad hoc diagnostics in other refs | Add standard smoke checks, deployment annotations, logs/metrics links |
| Documentation | README is a template and does not describe operations | Document ownership, release flow, environments, recovery, and prerequisites |

## 10. Modernization Recommendations

### Phase 0: Safety and discovery

1. Establish ownership and inventory for the deployment repo, external application repos, ACRs, clusters, Azure DevOps pipelines, service connections, variable groups, environments, and branch policies.
2. Rotate any credential-like values committed to Git and verify historical exposure. Confirm the active source of every secret and remove parallel injection paths.
3. Freeze or explicitly approve the current main/UAT/prod trigger behavior until environment mapping is documented.
4. Add a pipeline validation job that parses YAML and validates Kubernetes manifests without deploying.

### Phase 1: Standard build contract

1. Create one parameterized service pipeline template for checkout, toolchain setup, dependency caching, test execution, packaging, scanning, image build, and metadata publication.
2. Keep thin service entrypoints containing only service name, module, image, manifest/chart path, and trigger policy.
3. Define a versioned build contract: commit SHA, build ID, image digest, source repository/commit, dependency lock state, SBOM, and scan results.
4. Build images with explicit Dockerfiles in the application repository or a reviewed centralized build context. Eliminate generated inline Dockerfiles.
5. Push immutable tags and deploy by digest. Treat `latest` as prohibited for environment workloads.

### Phase 2: Promotion and deployment

1. Separate CI from CD. CI builds once and publishes the immutable artifact; CD promotes that exact digest through QA, UAT, and production.
2. Define Azure DevOps environments for QA/UAT/prod with required approvals, branch controls, service-health checks, and least-privilege service connections.
3. Replace direct per-service remote command strings with a versioned Helm chart or GitOps repository. The deployment contract should include shared prerequisites, ConfigMaps, secret references, network policies, ingress, workload resources, and ordering.
4. Add pre-deployment validation, readiness checks, smoke/contract tests, and post-deployment verification. Make reindexing an explicit idempotent job with observable success/failure rather than a best-effort curl pod.
5. Implement rollback by prior image digest and known-good release revision. Test rollback and dependency ordering in each environment.

### Phase 3: Platform controls

1. Enforce PR validation, protected branches, required reviewers, path ownership, and policy checks.
2. Add SAST, dependency/SCA, container, Kubernetes/IaC, secret-detection, SBOM, image signing, and provenance verification gates with severity policy.
3. Use managed identity/workload identity and Key Vault/CSI or an equivalent approved secret store; remove credentials from ConfigMaps and pipeline command arguments.
4. Add admission policies for non-`latest` images, required probes/resources, namespace boundaries, approved registries, and signed artifacts.
5. Add deployment telemetry: release annotations, correlation IDs, change links, rollout duration/failure metrics, and standard diagnostic links.

## 11. What Not to Change Immediately

Do not begin by renaming every service, moving branches, replacing AKS access, or converting all manifests to Helm in one release. The external application repositories, live cluster state, service names, ingress behavior, and production branch files are coupled and not represented completely here.

Before structural changes, capture a known-good deployment manifest/image digest for each service, confirm live dependencies and secret consumers, inventory Azure DevOps UI configuration, and test the deployment contract in a disposable or non-production environment. Preserve service resource names and public routes during the first migration unless a compatibility plan exists. Treat branch consolidation, Helm conversion, and GitOps adoption as staged migrations with rollback checkpoints.

## 12. Evidence Gaps and Required Follow-up

The repository cannot answer the following:

- Which Azure DevOps pipeline definitions are registered, enabled, scheduled, or authorized.
- Actual PR policies, environment approvals/checks, variable-group permissions, service-connection scopes, and agent capabilities.
- The contents and commit identity of the external backend/frontend repositories at pipeline execution time.
- Current AKS objects, image digests, replica health, rollout history, RBAC, admission policy, ingress controller, and secret store integration.
- Whether branch-specific pipelines under remote refs are active or historical.
- Test, deployment, rollback, and security scan outcomes from real pipeline runs.

These should be collected before implementation planning is baselined. The current repository evidence is sufficient to prioritize environment isolation, secret remediation, immutable artifacts, test/security gates, and template standardization, but not to claim production readiness or confirm the actual live release topology.

## 13. Assessment Conclusion

The implementation functions as a collection of direct UAT-oriented deployment scripts encoded in YAML, not as a standardized multi-environment CI/CD platform. The safest modernization sequence is to establish environment and secret ownership first, then introduce a shared build/release contract and immutable image promotion, followed by controlled deployment and policy gates. Immediate broad refactoring would carry avoidable risk because the deployment repository, external application repo
