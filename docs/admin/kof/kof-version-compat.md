# Version Compatibility

| Component       | Version  | Notes                         |
|-----------------|----------|-------------------------------|
| {{{ docsVersionInfo.k0rdentName }}}         | ≥ 0.0.7  | Required for template support |
| Kubernetes      | ≥ 1.32   | Earlier versions untested     |
| OpenTelemetry   | ≥ 0.75   | Recommended minimum           |
| VictoriaMetrics | ≥ 0.40   | Required for clustering       |

Detailed:

- [kof-mothership](https://github.com/k0rdent/kof/blob/v{{{ extra.docsVersionInfo.kofVersions.kofDotVersion }}}/charts/kof-mothership/Chart.yaml)
- [kof-storage](https://github.com/k0rdent/kof/blob/v{{{ extra.docsVersionInfo.kofVersions.kofDotVersion }}}/charts/kof-storage/Chart.yaml)
- [kof-operators](https://github.com/k0rdent/kof/blob/v{{{ extra.docsVersionInfo.kofVersions.kofDotVersion }}}/charts/kof-operators/Chart.yaml)
- [kof-collectors](https://github.com/k0rdent/kof/blob/v{{{ extra.docsVersionInfo.kofVersions.kofDotVersion }}}/charts/kof-collectors/Chart.yaml)
