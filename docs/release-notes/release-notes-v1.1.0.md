
# k0rdent OSS v1.1.0 Release Notes

Released: June 30, 2025

k0rdent OSS is the upstream, community-driven version of the k0rdent platform, released under the Apache 2.0 license. It delivers the core functionality of k0rdent as an open, composable control plane for managing Kubernetes clusters, services, and observability across public clouds, private data centers, and edge environments.

## Component & Provider Versions

| Component / Provider          | Version          |
|-------------------------------|------------------|
| Cluster API                   | v1.10.3          |
| CAPI Provider AWS             | v2.8.2           |
| CAPI Provider Azure           | v1.19.4          |
| CAPI Provider Docker          | v1.9.6           |
| CAPI Provider GCP             | v1.9.0           |
| CAPI Provider Infoblox        | v0.1.0-alpha.8   |
| CAPI Provider IPAM            | v1.0.2           |
| CAPI Provider k0smotron       | v1.5.4           |
| CAPI Provider OpenStack (ORC) | v0.12.3 / v2.1.0 |
| CAPI Provider vSphere         | v1.13.0          |
| Project Sveltos               | v0.54.0          |

### 🚀 New Features 🚀

* Pass certificate secret to the default helm repository ([#1558](https://github.com/k0rdent/kcm/pull/1558))
* Ability to set k0s dl url arch ([#1682](https://github.com/k0rdent/kcm/pull/1682))
* K0s cp/worker flags ([#1681](https://github.com/k0rdent/kcm/pull/1681))
* Adapt openstack template to mount CA cert ([#1624](https://github.com/k0rdent/kcm/pull/1624))
* Add support for GitRepository via fluxcd source. ([#1631](https://github.com/k0rdent/kcm/pull/1631))
* Add support for mounting registry and k0s URL certificates ([#1595](https://github.com/k0rdent/kcm/pull/1595))
* Mount registry certificate secret in CAPI operator ([#1580](https://github.com/k0rdent/kcm/pull/1580))
* Pass registry CA cert to hosted CP components ([#1659](https://github.com/k0rdent/kcm/pull/1659))
* StateManagementProvider API and controller ([#1489](https://github.com/k0rdent/kcm/pull/1489))
* Add non-root volumes to AWS Templates ([#1567](https://github.com/k0rdent/kcm/pull/1567))
* Automatically update stuck sveltos tokens ([#1588](https://github.com/k0rdent/kcm/pull/1588))
* Disable v1alpha1 api ([#1678](https://github.com/k0rdent/kcm/pull/1678))
* Propagate registry-credentials across cld ([#1598](https://github.com/k0rdent/kcm/pull/1598))
* Switching to upstream PrometheusRules at promxy and regional with patches for all/specific clusters ([#248](https://github.com/k0rdent/kof/pull/248))
* Add server to kof-operator for prometheus observability ([#275](https://github.com/k0rdent/kof/pull/275))
* Add configurable UI port setting ([#314](https://github.com/k0rdent/kof/pull/314))
* ContainerHighMemoryUsage alert for CAPI Operator and others ([#317](https://github.com/k0rdent/kof/pull/317))
* Configure Grafana SSO using Dex ([#319](https://github.com/k0rdent/kof/pull/319))
* Add autoinstrumentation to kof operator to collect metrics and traces ([#344](https://github.com/k0rdent/kof/pull/344))
* Custom image registries PRs and resolved conflicts ([#348](https://github.com/k0rdent/kof/pull/348))
* Sync kof operator resources when cluster annotation changes ([#340](https://github.com/k0rdent/kof/pull/340))

## ✨ Improvements ✨

* Add tests for kof operator UI ([#299](https://github.com/k0rdent/kof/pull/299))
* Bump `vite` from 6.2.0 to 6.2.7 ([#306](https://github.com/k0rdent/kof/pull/306))
* Use latest KCM release for CI ([#308](https://github.com/k0rdent/kof/pull/308))
* Add adopted regional cluster deployment ([#309](https://github.com/k0rdent/kof/pull/309))
* Add kof-ui docs ([#313](https://github.com/k0rdent/kof/pull/313))
* Align chart versions; sveltos dashboard bump to 0.54.0 ([#316](https://github.com/k0rdent/kof/pull/316), [#318](https://github.com/k0rdent/kof/pull/318))
* Add adopted child cluster deployment ([#321](https://github.com/k0rdent/kof/pull/321))
* Replace collector event receiver ([#327](https://github.com/k0rdent/kof/pull/327))

## ❗ Upgrade Notes ❗

* After upgrading KOF, please run:

  ```bash
  kubectl apply --server-side --force-conflicts \
    -f https://github.com/grafana/grafana-operator/releases/download/v5.18.0/crds.yaml
  ```

* And run the same for each regional cluster:

  ```bash
  kubectl get secret -n kcm-system $REGIONAL_CLUSTER_NAME-kubeconfig \
    -o=jsonpath={.data.value} | base64 -d > regional-kubeconfig

  KUBECONFIG=regional-kubeconfig kubectl apply --server-side --force-conflicts \
    -f https://github.com/grafana/grafana-operator/releases/download/v5.18.0/crds.yaml
  ```

This is noted as required in the [grafana-operator release notes](https://github.com/grafana/grafana-operator/releases/tag/v5.18.0).

### ✨ Notable Changes ✨

* Remove namespace from AWSClusterStaticIdentity reference ([#1585](https://github.com/k0rdent/kcm/pull/1585))
* Align vsphere-standalone tpl, fn names, logs, packages ([#1683](https://github.com/k0rdent/kcm/pull/1683))
* Auto-update all configs ([#1650](https://github.com/k0rdent/kcm/pull/1650))

## 🐛 Notable Fixes 🐛

* Flapping delete conditions ([#1653](https://github.com/k0rdent/kcm/pull/1653))
* CAPI operator registry cert mount volume mismatch ([#1621](https://github.com/k0rdent/kcm/pull/1621))
* Correct the k0s version of aws-standalone-cp ([#1623](https://github.com/k0rdent/kcm/pull/1623))
* Ensure services are removed before ClusterDeployment removal ([#1584](https://github.com/k0rdent/kcm/pull/1584))
* IPAM claim inline processing ([#1652](https://github.com/k0rdent/kcm/pull/1652))
* Rework the helm-push Makefile target ([#1520](https://github.com/k0rdent/kcm/pull/1520), [#1524](https://github.com/k0rdent/kcm/pull/1524))
* Set configure-cloud-routes=false in gcp ccm parameters ([#1540](https://github.com/k0rdent/kcm/pull/1540))
* Add CertSecretRef to registry configuration ([#1630](https://github.com/k0rdent/kcm/pull/1630))
* Check if services from self are running before deleting ClusterDeployment ([#1648](https://github.com/k0rdent/kcm/pull/1648))
* Don't expect operator provider in ProviderTemplates ([#1638](https://github.com/k0rdent/kcm/pull/1638))
* Global values for azure, capi, k0smotron providers ([#1593](https://github.com/k0rdent/kcm/pull/1593))
* Helm values for servicetemplates not showing in status.config ([#1544](https://github.com/k0rdent/kcm/pull/1544))
* Image paths inaccuracies ([#1615](https://github.com/k0rdent/kcm/pull/1615))
* Properly preserve meta during copying ([#1604](https://github.com/k0rdent/kcm/pull/1604))
* Use global.registry in orc templates ([#1566](https://github.com/k0rdent/kcm/pull/1566))
* Addon-controller ServiceMonitor watches wrong namespace ([#290](https://github.com/k0rdent/kof/pull/290))
* Filter out projectsveltos\_\* metrics from kcm cm ([#291](https://github.com/k0rdent/kof/pull/291))
* Errors on upgrade of kof ServiceTemplates ([#295](https://github.com/k0rdent/kof/pull/295))
* Logs sorting order ([#301](https://github.com/k0rdent/kof/pull/301))
* Remove shadcn add command causing unwanted file updates ([#307](https://github.com/k0rdent/kof/pull/307))
* Temporary adaptation of new alerts to current metrics ([#310](https://github.com/k0rdent/kof/pull/310))
* Workaround for `generatorURL` in alerts and "See source" in Grafana ([#312](https://github.com/k0rdent/kof/pull/312))
* Too many open files in `sveltos-dashboard` ([#320](https://github.com/k0rdent/kof/pull/320))
* Use correct namespace name for MCS ([#332](https://github.com/k0rdent/kof/pull/332))
* Update grafana to fix CVE-2025-4123 ([#339](https://github.com/k0rdent/kof/pull/339))
* Allow to parametrise operators and ingress-nginx values ([#337](https://github.com/k0rdent/kof/pull/337))
* Make Sveltos follow cluster updates ([#333](https://github.com/k0rdent/kof/pull/333))
* Pinned versions of Grafana plugins and Promxy as part of aig-gap solution ([#349](https://github.com/k0rdent/kof/pull/349))
* Duplicate `version` field in Grafana ([#350](https://github.com/k0rdent/kof/pull/350))
* Pattern dist not found on `kof-operator-release` ([#353](https://github.com/k0rdent/kof/pull/353), [#354](https://github.com/k0rdent/kof/pull/354))

## Deprecations and Removals

- All `v1alpha1` APIs are have previously been deprecated v1.0.0 and have now been removed.

## Known Issues

- None
    
## Release Metadata

| Key                   | Value                              |
| --------------------- | ---------------------------------- |
| Helm Charts           | kcm: 1.1.0, kof: 1.1.0, ksm: 1.1.0 |
| OCI Registry          | ghcr.io/k0rdent                    |
| SBOM                  | Not included in OSS                |
| OCI Signature Support | Enterprise only                    |
| Release Tags          | v1.1.0 across all components       |

## Contributors

Huge thanks to the following contributors for making this release possible:  
[@gmlexx](https://github.com/gmlexx), [@denis-ryzhkov](https://github.com/denis-ryzhkov), [@aglarendil](https://github.com/aglarendil), [@kylewuolle](https://github.com/kylewuolle), [@a13x5](https://github.com/a13x5), [@eromanova](https://github.com/eromanova), [@zerospiel](https://github.com/zerospiel), [@BROngineer](https://github.com/BROngineer), [@Kshatrix](https://github.com/Kshatrix), [@dis-xcom](https://github.com/dis-xcom), [@wahabmk](https://github.com/wahabmk), [@AndrejsPon00](https://github.com/AndrejsPon00)

## Resources

-   [Documentation](https://k0rdent.github.io/docs)
-   [GitHub Repositories](https://github.com/k0rdent)
-   CNCF Slack Channels: #k0rdent

## Try It Out

QuickStart guide: [https://docs.k0rdent.io/1.0.0/quickstarts/](https://docs.k0rdent.io/1.0.0/quickstarts/)