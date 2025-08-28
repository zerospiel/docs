# k0rdent 1.3 Release Notes
**Release date:** 2025-08-28

## Highlights
- **ServiceSet API & Sveltos provider (KSM/KCM):** 
    - New abstraction for managing services with explicit dependencies and ordered upgrades. Enables smoother rollouts and future multi-provider support.

- **Observability (KOF):**  
    - New kube-state-metrics dashboards for k0rdent CRDs and alerting.  
    - "Raw metrics" tab in the UI for collectors/logs.  
    - Promxy tuned for heavy queries to avoid OOMs.

- **Platform & Dependency Updates:**  
    - k0smotron upgraded to **v1.6.2**  
    - Cluster API Azure provider upgraded to **v1.20.2**  
    - Go toolchain moved to **1.24.6**, with Helm and testing libraries updated.

- **Stability & UX:** 
    - Multiple fixes to credential propagation, service status collection, hosted cluster templates, Helm repos, and cloud provider templates (OpenStack, Azure).

---

## Upgrade Notes
- **Helm charts:**  
   - Some standalone chart network settings are now defined in `values.yaml`. Review overrides before upgrading.
- **Providers:**  
   - Align pinned dependencies with new versions (k0smotron 1.6.2, CAPI Azure 1.20.2).  
   - Build/test environments should use Go 1.24.6.
- **Observability:**  
   - After upgrading KOF, apply the new dashboards and validate Promxy resource settings against your workload size.
- **KSM/KCM APIs:**  
   - The new `ServiceSet` API is additive. Existing Sveltos configurations continue to work, but you should begin testing with ServiceSets for future upgrades.

---

## Changelog

### New Features
- bdb9707: feat: child telemetry tracker (#1783) by @zerospiel
- 1355a18: feat: expose services.policyRef in CD and MCS spec (#1725) by @wahabmk
- 82d3e06: feat: introduce local telemetry collector (#1845) by @zerospiel
- f5fe7bb: feat: provider-agnostic KSM with built-in provider (#1670) by @BROngineer
- b15d226: feat: telemetry local storage configuration (#1857) by @zerospiel

### Notable Fixes
- d61ff20: fix: Mount CA cert as a volume to flux components (#1844) by @eromanova
- e11fe9e: fix: Mount CA secret to flux before installing kcm-templates (#1847) by @eromanova
- efb50cd: fix: add default storage class for openstack standalone templates (#1871) by @bnallapeta
- 6a2ae2e: fix: allow configuration of the default HelmRelease timeout (#1830) by @eromanova
- fe394a8: fix: continue e2e tests after deployment failure (#1840) by @eromanova
- cfcd997: fix: credentials propagation (#1886) by @BROngineer
- 44c9e2e: fix: drop subpath from the registry in hosted helm repositories (#1890) by @eromanova
- 76e1c79: fix: install yq before generating release.yaml (#1894) by @eromanova
- 2322cbe: fix: k0s image url for azure-hosted-cp (#1856) by @a13x5
- 6e49c90: fix: multiclusterservice requeue (#1899) by @BROngineer
- 97af4ed: fix: redundant servicetemplates fetching (#1832) by @zerospiel
- f930e2: fix: remote-cluster (hosted) respects useSudo (#1880) by @zerospiel
- 103b439: fix: service statuses (#1888) by @BROngineer
- 55d963e: fix: services deployed & clusters matched in MCS kubectl output (#1779) by @wahabmk
- af5feae: fix: set kcm version in release.yaml before making the release (#1877) by @eromanova

## Notable Changes 
- b3e2247: fix(os-tpls): correct identity name in identityref (#1901) by @zerospiel

---

## Known Issues
- No critical issues reported at release. Track [GitHub issues](https://github.com/k0rdent/kcm/issues) for updates.

---

## Contributors
Thank you to everyone who contributed PRs, issues, and testing for this release.

## References
- [Compare KCM v1.2.0...v1.3.0](https://github.com/k0rdent/kcm/compare/v1.2.0...v1.3.0)  
- [Compare KOF v1.2.0...v1.3.0](https://github.com/k0rdent/kof/compare/v1.2.0...v1.3.0)  
