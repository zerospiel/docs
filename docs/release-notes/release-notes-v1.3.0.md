# k0rdent 1.3 Release Notes
**Release date:** 2025-08-28

k0rdent 1.3 brings major improvements across the platform, with a new provider-agnostic service orchestration API, stronger observability, and stability fixes across cluster management.

---

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

## Known Issues
- No critical issues reported at release. Track [GitHub issues](https://github.com/k0rdent/kcm/issues) for updates.

---

## Contributors
Thank you to everyone who contributed PRs, issues, and testing for this release.

## References
- [Compare KCM v1.2.0...v1.3.0](https://github.com/k0rdent/kcm/compare/v1.2.0...v1.3.0)  
- [Compare KOF v1.2.0...v1.3.0](https://github.com/k0rdent/kof/compare/v1.2.0...v1.3.0)  
