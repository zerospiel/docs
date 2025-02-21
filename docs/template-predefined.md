## Remove Templates shipped with k0rdent

If you need to limit the templates that exist in your k0rdent installation, follow the instructions below:

1. Get the list of `ProviderTemplate`, `ClusterTemplate` or `ServiceTemplate` objects shipped with k0rdent. For example,
for `ClusterTemplate` objects, run:

    ```bash
    kubectl get clustertemplates -n kcm-system -l helm.toolkit.fluxcd.io/name=kcm-templates
    ```
    ```console
    NAMESPACE    NAME                            VALID
    kcm-system   adopted-cluster-{{{ extra.docsVersionInfo.k0rdentVersion }}}           true
    kcm-system   aws-eks-{{{ extra.docsVersionInfo.k0rdentVersion }}}                   true
    kcm-system   aws-hosted-cp-{{{ extra.docsVersionInfo.k0rdentVersion }}}             true
    kcm-system   aws-standalone-cp-{{{ extra.docsVersionInfo.k0rdentVersion }}}         true
    kcm-system   azure-aks-{{{ extra.docsVersionInfo.k0rdentVersion }}}                 true
    kcm-system   azure-hosted-cp-{{{ extra.docsVersionInfo.k0rdentVersion }}}           true
    kcm-system   azure-standalone-cp-{{{ extra.docsVersionInfo.k0rdentVersion }}}       true
    kcm-system   openstack-standalone-cp-{{{ extra.docsVersionInfo.k0rdentVersion }}}   true
    kcm-system   vsphere-hosted-cp-{{{ extra.docsVersionInfo.k0rdentVersion }}}         true
    kcm-system   vsphere-standalone-cp-{{{ extra.docsVersionInfo.k0rdentVersion }}}     true
    ```

2. Remove the template from the list using `kubectl delete`, as in:

    ```bash
    kubectl delete clustertemplate -n kcm-system <template-name>
    ```