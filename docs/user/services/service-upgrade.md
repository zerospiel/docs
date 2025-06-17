# Upgrading and rolling back deployed services

## Service Version Upgrade

When you deploy a service to a cluster, you can specify a `ServiceTemplateChain` which will be used to define available upgrade path for the service. 

> INFO:
> Before you begin, make sure all templates, you're going to add to `ServiceTemplateChain`, exist in system namespace.
> Templates can be propagated to other namespaces using [Template Life Cycle Management](../../reference/template/index.md).

First, you need to create a `ServiceTemplateChain` object:

```yaml
apiVersion: k0rdent.mirantis.com/v1alpha1
kind: ServiceTemplateChain
metadata:
  name: ingress-nginx-chain
  namespace: kcm-system
spec:
  supportedTemplates:
    - name: ingress-nginx-4-11-3
      availableUpgrades:
      - name: ingress-nginx-4-11-5
    - name: ingress-nginx-4-11-5
```

This object defines a chain of templates that can be used to upgrade the service.

> WARNING:
> The `ServiceTemplateChain` has immutable spec. You can't change it after it's created.

After `ServiceTemplateChain` is created you can use it in `ClusterDeployment` or `MutliClusterService` object to define available upgrade path for the service:

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: ClusterDeployment
metadata:
  name: my-cluster-deployment
  namespace: tenant42
spec:
  config:
    clusterLabels: {}
  template: aws-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.awsStandaloneCpCluster }}}
  credential: aws-credential
  serviceSpec:
    services:
      - template: ingress-nginx-4-11-3
        templateChain: ingress-nginx-chain
        name: ingress-nginx
        namespace: tenant42
    priority: 100
```

> WARNING:
> If no `templateChain` is specified for the service, the service upgrade will be unavailable.
> You'll observe the following error message in the logs:
> `service ingress-nginx/ingress-nginx can't be upgraded from ingress-nginx-4-11-3 to ingress-nginx-4-11-5`
> in case you'll try to change the service template.

After `ClusterDeployment` or `MultiClusterService` will be reconciled, you can observe available upgrade paths for the service in the status:

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: ClusterDeployment
metadata:
  name: my-cluster-deployment
  namespace: tenant42
spec:
  ...
status:
  servicesUpgradePaths:
    - availableUpgrades:
        - upgradePaths:
            - ingress-nginx-4-11-5
      name: ingress-nginx
      namespace: ingress-nginx
      template: ingress-nginx-4-11-3
```

Now you can update the `ClusterDeployment` or `MultiClusterService` object to upgrade the service to the available version:

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: ClusterDeployment
metadata:
  name: my-cluster-deployment
  namespace: tenant42
spec:
  config:
    clusterLabels: {}
  template: aws-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.awsStandaloneCpCluster }}}
  credential: aws-credential
  serviceSpec:
    services:
      - template: ingress-nginx-4-11-5 # <-- upgrade to the latest version
        templateChain: ingress-nginx-chain
        name: ingress-nginx
        namespace: tenant42
    priority: 100
```

## Service Version Rollback

In general, the process of upgrading a service is the same as the process of rolling back to the previous version. You'll need to create separate `ServiceTemplateChain` which defines the reversed upgrade path:

```yaml
apiVersion: k0rdent.mirantis.com/v1alpha1
kind: ServiceTemplateChain
metadata:
  name: ingress-nginx-chain
  namespace: kcm-system
spec:
  supportedTemplates:
    - name: ingress-nginx-4-11-3
    - name: ingress-nginx-4-11-5
      availableUpgrades:
        - name: ingress-nginx-4-11-3
```

After `ServiceTemplateChain` is created you can use it in `ClusterDeployment` or `MutliClusterService` object to define available rollback path for the service:

1. update the `ClusterDeployment` or `MultiClusterService` object with rollback `ServiceTempalteChain`
2. wait for the `ClusterDeployment` or `MultiClusterService` to be reconciled
3. update the `ClusterDeployment` or `MultiClusterService` object with the previous version of the service
