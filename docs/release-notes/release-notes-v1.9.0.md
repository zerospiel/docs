# k0rdent 1.9.0 Release Notes

**Release date:** April 27, 2026

## Components Versions

| Provider Name                        | Version                     |
|--------------------------------------|-----------------------------|
| Cluster API                          | v1.12.5                     |
| Cluster API Provider AWS             | v2.10.0                     |
| Cluster API Provider Azure           | v1.23.0                     |
| Cluster API Provider Docker          | v1.12.5                     |
| Cluster API Provider GCP             | v1.11.1                     |
| Cluster API Provider Infoblox        | v0.1.0                      |
| Cluster API Provider IPAM            | v1.1.0-rc.1                 |
| Cluster API Provider k0smotron       | v1.10.6                     |
| Cluster API Provider Kubevirt        | v0.11.0                     |
| Cluster API Provider OpenStack (ORC) | v0.13.0-mirantis.0 (v2.1.0) |
| Cluster API Provider vSphere         | v1.15.1                     |
| Projectsveltos                       | v1.1.1                      |
| k0s (control plane runtime)          | v1.32.8                     |
| cert-manager (charts)                | v1.19.3                     |

---

## Highlights

* **Gateway API Adoption for Modern Kubernetes Networking**: {{{ docsVersionInfo.k0rdentName }}} replaces NGINX Ingress with the Kubernetes-native Gateway API powered by Envoy Gateway, moving away from an annotation-heavy, controller-specific model to a standardized, role-based approach. This cleanly separates infrastructure (Gateways) from application routing (Routes), giving users more consistent and portable configurations while enabling operators to manage multi-tenant and multi-cluster traffic more reliably. It also aligns the platform with the future direction of Kubernetes networking, reducing long-term migration risk.

* **Improved Deployment Stability and Reconciliation Behavior**: Fixes to `ClusterDeployment` conditions, `ServiceSet` reconciliation, and controller behavior reduce cases where deployments become stuck, flap between states, or require manual intervention. The system now converges more predictably toward the desired state, which means operators spend less time diagnosing ambiguous deployment status and forcing reconciles.

* **More Resilient Failure Handling and Recovery**: The system now better handles failure scenarios, including preventing service loss during dependency resolution failures and improving the reliability of components like the drift detection manager. Failures are less likely to leave the system in partially broken states, making recovery more straightforward and reducing the operational risk associated with complex deployments.

* **More Reliable Configuration and Integration Workflows**: Improvements to Helm repository handling (including `certSecretRef`) and fixes to template validation, defaults, and image pull secret handling reduce configuration-related friction. These changes eliminate a class of issues where valid configurations failed due to platform inconsistencies, resulting in smoother integrations and less time spent debugging setup problems.

* **Enhanced Observability and Deployment Predictability**: Improvements such as dry-run release notifications and dynamic release naming give operators better visibility into what changes will occur before deployment and what has been deployed afterward. This strengthens auditability and allows for more controlled rollouts, especially in production environments.

* **Improved Upgrade Safety and Lifecycle Management**: Fixes to CRD installation and upgrade policies reduce the likelihood of failed or inconsistent upgrades. Lifecycle operations are now more reliable, making upgrades less error-prone and easier to execute without introducing instability into the system.

* **Corrected Cluster Networking Data Handling**: Updates to how cluster networking data is sourced and interpreted fix issues that could lead to incorrect assumptions about cluster topology. This removes a class of subtle, hard-to-diagnose connectivity problems and improves overall system reliability.

* **Reduced Operational Noise and Manual Intervention**: Across the release, multiple small but impactful fixes reduce unnecessary churn—fewer reconcile loops, fewer false error states, and more stable controllers. While not visible as features, the cumulative effect is significant: operators spend less time reacting to the system and more time managing actual workloads. This effectively lowers the operational overhead of running the platform.

---


## Upgrade Notes

### k0rdent Observability an Finance (KOF)

KOF Enterprise v1.3.1 replaces Nginx Ingress with Envoy Gateway by introducing Gateway API support, requiring users to update their Helm values to disable ingress-nginx and enable envoy-gateway before upgrading. This change may cause temporary downtime due to DNS propagation unless a parallel regional cluster migration is used to avoid disruption. The upgrade process involves exporting current Helm values, modifying them for the new gateway configuration, handling DNS updates (automatically or manually), and then performing a Helm upgrade with the updated configuration and registry settings.

For full information on [upgrading to KOF 1.8.1, see the full documentation](../admin/kof/kof-upgrade.md).

---

## Changelog


Here’s the structured conversion with proper categorization (and deduping where needed):

---

### New Features

* **feat:** support for registry with auth for child clusters ([#2521](https://github.com/k0rdent/kcm/pull/2521)) by @a13x5
* **feat:** add ingress support to the openstack hosted template ([#2533](https://github.com/k0rdent/kcm/pull/2533)) by @eromanova
* **feat(templates):** add auto-reload annotation ([#2592](https://github.com/k0rdent/kcm/pull/2592)) by @zerospiel
* **feat(cld):** notify release change on dry-run ([#2633](https://github.com/k0rdent/kcm/pull/2633)) by @zerospiel
* **feat(templates):** upgrade EKS to 1.35 ([#2497](https://github.com/k0rdent/kcm/pull/2497)) by @zerospiel

---

### Notable Fixes

* **fix:** remove reading from cluster network field and use node data instead by @kylewuolle
* **fix:** use certSecretRef in Flux HelmRepository ([#2486](https://github.com/k0rdent/kcm/pull/2486)) by @eromanova
* **fix:** define provider CRDs install/upgrade policy ([#2484](https://github.com/k0rdent/kcm/pull/2484)) by @surabujin
* **fix:** incorrect ASO version ([#2502](https://github.com/k0rdent/kcm/pull/2502)) by @a13x5
* **fix(cld):** limit CAPI cluster by namespace for conditions ([#2503](https://github.com/k0rdent/kcm/pull/2503)) by @zerospiel
* **fix:** servicetemplate spec.resources.path ([#2511](https://github.com/k0rdent/kcm/pull/2511)) by @BROngineer
* **fix(templates):** allow empty GKE locationPolicy ([#2514](https://github.com/k0rdent/kcm/pull/2514)) by @zerospiel
* **fix:** quote Kubernetes API server port in OpenStack CCM ([#2535](https://github.com/k0rdent/kcm/pull/2535)) by @eromanova
* **fix(template):** override kube API server only when ingress enabled ([#2540](https://github.com/k0rdent/kcm/pull/2540)) by @eromanova
* **fix:** dependency cycle validation bug ([#2527](https://github.com/k0rdent/kcm/pull/2527)) by @wahabmk
* **fix:** specify Helm chart order in filename ([#2543](https://github.com/k0rdent/kcm/pull/2543)) by @eromanova
* **fix:** check CredSecretName before passing to globals ([#2544](https://github.com/k0rdent/kcm/pull/2544)) by @a13x5
* **fix:** capo-orc-fetch indentation ([#2551](https://github.com/k0rdent/kcm/pull/2551)) by @a13x5
* **fix(api):** CLD nullable propagateCredentials ([#2555](https://github.com/k0rdent/kcm/pull/2555)) by @zerospiel
* **fix(templates):** Helm extensions caFile configuration ([#2556](https://github.com/k0rdent/kcm/pull/2556)) by @eromanova
* **fix:** return error for invalid templates ([#2562](https://github.com/k0rdent/kcm/pull/2562)) by @BROngineer
* **fix:** ClusterDeployment conditions handling improvements ([#2563](https://github.com/k0rdent/kcm/pull/2563)) by @eromanova
* **fix:** refer secrets and configmaps in target namespace for localSourceRef ([#2574](https://github.com/k0rdent/kcm/pull/2574)) by @BROngineer
* **fix:** don’t overwrite error in defer func in MCS controller ([#2582](https://github.com/k0rdent/kcm/pull/2582)) by @wahabmk
* **fix:** proper ORC rolebindings namespace ([#2595](https://github.com/k0rdent/kcm/pull/2595)) by @a13x5
* **fix:** preserve deployed services when dependency fails ([#2593](https://github.com/k0rdent/kcm/pull/2593)) by @BROngineer
* **fix:** global variables handling ([#2600](https://github.com/k0rdent/kcm/pull/2600)) by @zerospiel
* **fix:** support dynamic KCM release name ([#2586](https://github.com/k0rdent/kcm/pull/2586)) by @eromanova
* **fix:** cluster deployment reconciler improvements ([#2620](https://github.com/k0rdent/kcm/pull/2620)) by @eromanova
* **fix:** templatechain available upgrades version optional by @BROngineer
* **fix:** don’t use empty selector in MCS controller ([#2630](https://github.com/k0rdent/kcm/pull/2630)) by @wahabmk
* **fix(ksm):** serviceset processing causing continuous reconcile ([#2640](https://github.com/k0rdent/kcm/pull/2640)) by @BROngineer
* **fix(templates):** incorrect default values ([#2652](https://github.com/k0rdent/kcm/pull/2652)) by @zerospiel
* **fix(templates):** nullify imagePullSecrets and reload ([#2664](https://github.com/k0rdent/kcm/pull/2664)) by @zerospiel
* **fix:** ensure drift detection manager deployed on install by @kylewuolle

---

### Platform & Dependency Updates

* **chore(deps):** bump golang.org/x/net from 0.50.0 to 0.51.0 ([#2455](https://github.com/k0rdent/kcm/pull/2455)) by @dependabot
* **chore(deps):** bump github.com/cert-manager/cert-manager ([#2442](https://github.com/k0rdent/kcm/pull/2442)) by @dependabot
* **chore(bump):** cert-manager to v1.19.4 ([#2457](https://github.com/k0rdent/kcm/pull/2457)) by @Kshatrix
* **chore(deps):** bump actions/upload-artifact from 6 to 7 ([#2462](https://github.com/k0rdent/kcm/pull/2462)) by @dependabot
* **chore(deps):** bump k8s.io/kubectl from 0.35.1 to 0.35.2 ([#2463](https://github.com/k0rdent/kcm/pull/2463)) by @dependabot
* **chore(deps):** bump fluxcd/helm-controller/api ([#2465](https://github.com/k0rdent/kcm/pull/2465)) by @dependabot
* **chore(deps):** bump k8s.io/apiserver ([#2464](https://github.com/k0rdent/kcm/pull/2464)) by @dependabot
* **chore(deps):** bump docker/login-action ([#2479](https://github.com/k0rdent/kcm/pull/2479)) by @dependabot
* **chore(deps):** bump docker/setup-buildx-action ([#2483](https://github.com/k0rdent/kcm/pull/2483)) by @dependabot
* **chore(deps):** bump cluster-api-operator by @dependabot
* **chore(deps):** bump velero and chart ([#2494](https://github.com/k0rdent/kcm/pull/2494)) by @zerospiel
* **chore(bump):** AWS provider to v2.10.2 ([#2493](https://github.com/k0rdent/kcm/pull/2493)) by @Kshatrix
* **chore:** bump k0s to v1.35.1+k0s.1 ([#2501](https://github.com/k0rdent/kcm/pull/2501)) by @a13x5
* **chore(deps):** multiple Go, Flux, and Kubernetes dependency bumps (0.35.x series, Flux APIs, crypto, net, text) by @dependabot
* **chore:** bump projectsveltos (1.1.1 → 1.6.x → 1.8.0) by @BROngineer / @wahabmk
* **chore(bump):** Azure and GCP providers updates by @Kshatrix
* **chore(deps):** Helm v3 updates (3.20.x) by @dependabot
* **chore(deps):** kubevirt.io/api updates (1.7.x → 1.8.x) by @dependabot
* **chore(deps):** CAPA provider update ([#2648](https://github.com/k0rdent/kcm/pull/2648)) by @zerospiel
* **chore(deps):** cluster-api updates (v1.12.x series) by @dependabot
* **chore(deps):** GitHub Actions, CEL, crypto, grpc, and other ecosystem bumps by @dependabot

---

### Other Changes (CI, Tests, Refactors)

* **ci:** increase e2e test timeout to 9h ([#2447](https://github.com/k0rdent/kcm/pull/2447)) by @zerospiel
* **test:** add release controller integration tests ([#2458](https://github.com/k0rdent/kcm/pull/2458)) by @eromanova
* **test(envtest):** fix kube-apiserver error ([#2522](https://github.com/k0rdent/kcm/pull/2522)) by @zerospiel
* **test:** increase multicluster service coverage by @kylewuolle
* **test:** fix flapping templatechain test ([#2583](https://github.com/k0rdent/kcm/pull/2583)) by @BROngineer
* **ci:** telemetry workflow fixes ([#2496](https://github.com/k0rdent/kcm/pull/2496)) by @zerospiel
* **ci(cache):** remove custom cache actions ([#2621](https://github.com/k0rdent/kcm/pull/2621)) by @zerospiel
* **ci:** expand conventional commit regex ([#2643](https://github.com/k0rdent/kcm/pull/2643)) by @zerospiel
* **ci:** staging workflow fixes by @zerospiel
* **refactor:** rearrange make targets ([#2569](https://github.com/k0rdent/kcm/pull/2569)) by @zerospiel
* **chore:** serviceSet refactor ([#2482](https://github.com/k0rdent/kcm/pull/2482)) by @BROngineer

---

## References

* [Compare KCM v1.8.0...v1.9.0](https://github.com/k0rdent/kcm/compare/v1.8.0...v1.9.0)
* [Compare KOF v1.8.0...v1.9.0](https://github.com/k0rdent/kof/compare/v1.8.0...v1.9.0)
