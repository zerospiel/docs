# k0rdent 1.3.1 Release Notes
**Release date:** 2025-09-03

## Components Versions

| Provider Name | Version |
|-------------------|-------------|
| Cluster API | v1.10.4 |
| Cluster API Provider AWS | v2.8.4 |
| Cluster API Provider Azure | v1.20.2 |
| Cluster API Provider Docker | v1.9.6 |
| Cluster API Provider GCP | v1.10.0 |
| Cluster API Provider Infoblox | v0.1.0-alpha.8 |
| Cluster API Provider IPAM | v1.0.2 |
| Cluster API Provider k0smotron | v1.6.2 |
| Cluster API Provider OpenStack (ORC) | v0.12.4 (v2.1.0) |
| Cluster API Provider vSphere | v1.13.1 |
| Projectsveltos | v0.57.2 |

---

## Changelog

KSM has received several critical fixes in the v1.3.1 release:

### Notable Fixes

- [b809ba8](https://github.com/k0rdent/kcm/commit/b809ba8a1c84d1e6b7b70ee9c5fe35349f8612d4): fix: collecting ksm service statuses ([#1952](https://github.com/k0rdent/kcm/pull/1952)) by [@BROngineer](https://github.com/BROngineer)
- [66af56f](https://github.com/k0rdent/kcm/commit/66af56f2c902384beed953141cadae0e1b452210): fix: enable multiclusterservice requeue on status update ([#1946](https://github.com/k0rdent/kcm/pull/1946)) by [@BROngineer](https://github.com/BROngineer)
- [030ea58](https://github.com/k0rdent/kcm/commit/030ea5898cede11324eab08123797bf7bde08132): fix: handle adopted cluster in ksm ([#1948](https://github.com/k0rdent/kcm/pull/1948)) by [@BROngineer](https://github.com/BROngineer)
- [c66bbe2](https://github.com/k0rdent/kcm/commit/c66bbe22405bcf75d1f519f5686d876267b0b072): fix: multiclusterservice cleanup ([#1947](https://github.com/k0rdent/kcm/pull/1948)) by [@BROngineer](https://github.com/BROngineer)

## References
- [Compare KCM v1.3.0...v1.3.1](https://github.com/k0rdent/kcm/compare/v1.3.0...v1.3.1)  
