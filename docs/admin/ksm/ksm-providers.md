# KSM Providers

{{{ docsVersionInfo.k0rdentName }}} provides API to make underlying CD software pluggable and leverage it for KSM purposes.
There are two crucial API types: `StateManagementProvider` and `ServiceSet`.

## StateManagementProvider

The goal of `StateManagementProvider` is to ensure that the `ServiceSet` objects produced by {{{ docsVersionInfo.k0rdentName }}} controllers
will be reconciled and transformed to provider-specific objects. Aside from that, the controller which reconciles `StateManagementProvider` objects,
will validate that all components - [adapter](#adapter), [provisioner](#provisioner) and [required custom resources](#provisioner-crds) - are ready to be used. 
User-defined CEL rules will be used for this purpose.

It is also possible to [Build Your Own KSM Provider](ksm-byo-provider.md)

Here is an example of built-in `StateManagementProvider` object:

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: StateManagementProvider
metadata:
  name: ksm-projectsveltos
spec:
  adapter:
    apiVersion: apps/v1
    kind: Deployment
    name: kcm-controller-manager
    namespace: kcm-system
    readinessRule: |-
      self.status.availableReplicas == self.status.replicas &&
      self.status.availableReplicas == self.status.updatedReplicas &&
      self.status.availableReplicas == self.status.readyReplicas
  provisioner:
  - apiVersion: apps/v1
    kind: Deployment
    name: addon-controller
    namespace: projectsveltos
    readinessRule: |-
      self.status.availableReplicas == self.status.replicas &&
      self.status.availableReplicas == self.status.updatedReplicas &&
      self.status.availableReplicas == self.status.readyReplicas
  provisionerCRDs:
  - group: config.projectsveltos.io
    resources:
    - profiles
    - clustersummaries
    version: v1beta1
  selector:
    matchLabels:
      ksm.k0rdent.mirantis.com/adapter: kcm-controller-manager
  suspend: false
status:
  conditions:
  - lastTransitionTime: "2025-08-26T10:58:26Z"
    message: Successfully ensured RBAC
    observedGeneration: 1
    reason: RBACEnsuredSuccessfully
    status: "True"
    type: RBACReady
  - lastTransitionTime: "2025-08-27T21:29:49Z"
    message: Successfully ensured adapter
    observedGeneration: 1
    reason: AdapterEnsuredSuccessfully
    status: "True"
    type: AdapterReady
  - lastTransitionTime: "2025-08-26T10:59:37Z"
    message: Successfully ensured provisioner
    observedGeneration: 1
    reason: ProvisionerEnsuredSuccessfully
    status: "True"
    type: ProvisionerReady
  - lastTransitionTime: "2025-08-26T10:59:17Z"
    message: Successfully ensured provisioner CRDs
    observedGeneration: 1
    reason: ProvisionerCRDsEnsuredSuccessfully
    status: "True"
  ready: true
```

### Adapter

This is some workload deployed in management cluster which will reconcile `ServiceSet` objects. Built-in adapter comes bundled as a part of `kcm-controller-manager` binary.
Adapter is expected to convert `ServiceSpec` object being reconciled to provider-specific object and collect status of defined services.

### Provisioner

This is a set of or a single workload which will reconcile provider-specific objects produced by the adapter. For built-in provider it's a `addon-controller` provided by ProjectSveltos.

### Provisioner CRDs

This is a set of CRDs which are required by adapter and provisioners to operate normally.

> NOTE: `StateManagementProvider` controller does not deploy any workloads or CRDs to the management cluster.
> The only goal of this controller is to ensure that all components, required to deploy services, exist and
> operate normally.

## ServiceSet

The goal of `ServiceSet` is to act as a Man-in-The-Middle between {{{ docsVersionInfo.k0rdentName }}} API and provider-specific objects.

Here is an example of `ClusterDeployment` object with defined service to be deployed and the `ServiceSet` object produced out of `ClusterDeployment` spec:

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: ClusterDeployment
metadata:
  name: sample-cluster
  namespace: kcm-system
spec:
  # spec fields related to cluster configuration are omitted
  serviceSpec:
    provider:
      name: kcm-projectsveltos
      config:
        continueOnError: false
        priority: 100
        stopOnConflict: false
        syncMode: Continuous
    services:
      - template: ingress-nginx-4-11-0
        name: managed-ingress
```

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: ServiceSet
metadata:
  name: sample-cluster
  namespace: kcm-system
  labels:
    ksm.k0rdent.mirantis.com/adapter: kcm-controller-manager
spec:
  cluster: kcm-system
  provider:
    name: kcm-projectsveltos
    config:
      continueOnError: false
      priority: 100
      stopOnConflict: false
      syncMode: Continuous
  services:
    - template: ingress-nginx-4-11-0
      name: managed-ingress
      namespace: default
```
