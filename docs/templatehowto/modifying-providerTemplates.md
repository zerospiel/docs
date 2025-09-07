# Modifying ProviderTemplates

1. The `ProviderTemplate` resource has dedicated fields to set compatible `CAPI` contract versions along
with CRDs contract versions supported by the provider.
Given contract versions will then be set accordingly in the `.status` field.
Compatibility contract versions are key-value pairs, where the key is **the core `CAPI` contract version**,
and the value is an underscore-delimited (_) list of provider contract versions supported by the core `CAPI`.
For the core `CAPI` Template values should be empty.

    Example with the `.spec`:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1beta1
    kind: ProviderTemplate
    # ...
    spec:
      providers:
      - infrastructure-aws
      capiContracts:
        # commented is the example exclusively for the core CAPI Template
        # v1alpha3: ""
        # v1alpha4: ""
        # v1beta1: ""
        v1alpha3: v1alpha3
        v1alpha4: v1alpha4
        v1beta1: v1beta1_v1beta2
    ```

    Example with the `annotations` in the `Chart.yaml` with the same logic
    as in the `.spec`:

    ```yaml
    annotations:
      cluster.x-k8s.io/provider: infrastructure-aws
      cluster.x-k8s.io/v1alpha3: v1alpha3
      cluster.x-k8s.io/v1alpha4: v1alpha4
      cluster.x-k8s.io/v1beta1: v1beta1_v1beta2
    ```

1. The `ClusterTemplate` resource has dedicated fields to set an exact compatible Kubernetes version
in the Semantic Version format and required contract versions per each provider to match against
the related `ProviderTemplate` objects.
Given compatibility attributes will be then set accordingly in the `.status` field.
Compatibility contract versions are key-value pairs, where the key is **the name of the provider**,
and the value is the provider contract version required to be supported by the provider.

    Example with the `.spec`:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1beta1
    kind: ClusterTemplate
    # ...
    spec:
      k8sVersion: 1.30.0 # only exact semantic version is applicable
      providers:
      - bootstrap-k0sproject-k0smotron
      - control-plane-k0sproject-k0smotron
      - infrastructure-aws
      providerContracts:
        bootstrap-k0sproject-k0smotron: v1beta1 # only a single contract version is applicable
        control-plane-k0sproject-k0smotron: v1beta1
        infrastructure-aws: v1beta2
    ```

    Example with the `.annotations` in the `Chart.yaml`:

    ```yaml
    annotations:
      cluster.x-k8s.io/provider: infrastructure-aws, control-plane-k0sproject-k0smotron, bootstrap-k0sproject-k0smotron
      cluster.x-k8s.io/bootstrap-k0sproject-k0smotron: v1beta1
      cluster.x-k8s.io/control-plane-k0sproject-k0smotron: v1beta1
      cluster.x-k8s.io/infrastructure-aws: v1beta2
      k0rdent.mirantis.com/k8s-version: 1.30.0
    ```

1. The `ServiceTemplate` resource has dedicated fields to set an compatibility constrained
Kubernetes version to match against the related `ClusterTemplate` objects.
Given compatibility values will be then set accordingly in the `.status` field.

    Example with the `.spec`:

    ```yaml
    apiVersion: k0rdent.mirantis.com/v1beta1
    kind: ServiceTemplate
    # ...
    spec:
      k8sConstraint: "^1.30.0" # only semantic version constraints are applicable
    ```

    Example with the `annotations` in the `Chart.yaml`:

    ```yaml
    k0rdent.mirantis.com/k8s-version-constraint: ^1.30.0
    ```

### Compatibility attributes enforcement

The aforedescribed attributes are checked for compliance with the following rules:

* Both the exact and constraint version of the same type (for example `k8sVersion` and `k8sConstraint`) must
be set, otherwise no check is performed;
* If a `ClusterTemplate` object's provider's contract version does not satisfy contract versions
from the related `ProviderTemplate` object, updates to the `ClusterDeployment` object will be blocked;
* If a `ProviderTemplate` object's `CAPI` contract version
(for example, in a `v1beta1: v1beta1_v1beta2` key-value pair, the key `v1beta1` is the core `CAPI` contract version)
is not listed in the core `CAPI` `ProviderTemplate` object, the updates to the `Management` object will be blocked;
* If a `ClusterTemplate` object's exact kubernetes version does not satisfy the kubernetes version
constraint from the related `ServiceTemplate` object, the updates to the `ClusterDeployment` object will be blocked.